---
layout: post
title: "Hyperledger Fabric中节点通信"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### 事件管理交互图V0.6.1
<img src="/upload/2017/peerCommunication.png" width="1024px">

### 节点启动并serve
{% highlight bash %}
func serve(args []string) error {
	// Parameter overrides must be processed before any paramaters are
	// cached. Failures to cache cause the server to terminate immediately.
	if chaincodeDevMode {
		logger.Info("Running in chaincode development mode")
		logger.Info("Set consensus to NOOPS and user starts chaincode")
		logger.Info("Disable loading validity system chaincode")

		// 节点验证开启
		viper.Set("peer.validator.enabled", "true")
		// 开发模式默认共识机制采用noops
		viper.Set("peer.validator.consensus", "noops")
		// 如果定义dev模式，则chaincode在命令行中运行，若为net则在容器中运行
		viper.Set("chaincode.mode", chaincode.DevModeUserRunsChaincode)

	}

	if err := peer.CacheConfiguration(); err != nil {
		return err
	}

	peerEndpoint, err := peer.GetPeerEndpoint()
	if err != nil {
		err = fmt.Errorf("Failed to get Peer Endpoint: %s", err)
		return err
	}

	// 配置文件core.yaml中的配置
	// peer service端口7051
	listenAddr := viper.GetString("peer.listenAddress")

	if "" == listenAddr {
		logger.Debug("Listen address not specified, using peer endpoint address")
		listenAddr = peerEndpoint.Address
	}

	lis, err := net.Listen("tcp", listenAddr)
	if err != nil {
		grpclog.Fatalf("Failed to listen: %v", err)
	}

	// 创建事件交换服务
	// ehubLis 7053 注册了BLOCK，CHAINCODE，REJECTION，REGISTER
	ehubLis, ehubGrpcServer, err := createEventHubServer()
	if err != nil {
		grpclog.Fatalf("Failed to create ehub server: %v", err)
	}

	// 输出安全设置值
	logger.Infof("Security enabled status: %t", core.SecurityEnabled())
	if viper.GetBool("security.privacy") {
		// 隐私启用状态
		if core.SecurityEnabled() {
			logger.Infof("Privacy enabled status: true")
		} else {
			panic(errors.New("Privacy cannot be enabled as requested because security is disabled"))
		}
	} else {
		logger.Infof("Privacy enabled status: false")
	}

	//数据库启动，打开/var/hyperledger/production/db目录
	db.Start()

	var opts []grpc.ServerOption
	// 如果TLS启用，读取证书文件，在docker-compose.yml中CORE_SECURITY_ENABLED=true
	if comm.TLSEnabled() {
		// core.yaml中定义了了访问membersrvc的接口  localhost:7054, 获取配置
		creds, err := credentials.NewServerTLSFromFile(viper.GetString("peer.tls.cert.file"),
			viper.GetString("peer.tls.key.file"))

		if err != nil {
			grpclog.Fatalf("Failed to generate credentials %v", err)
		}
		opts = []grpc.ServerOption{grpc.Creds(creds)}
	}

	// 获取gRPC服务器实例
	grpcServer := grpc.NewServer(opts...)

	// 获取安全帮助函数
	secHelper, err := getSecHelper()
	if err != nil {
		return err
	}

	// 安全帮助函数
	secHelperFunc := func() crypto.Peer {
		return secHelper
	}

	// 注册链码支持
	registerChaincodeSupport(chaincode.DefaultChain, grpcServer, secHelper)

	var peerServer *peer.Impl

	// Create the peerServer
	// 如果开启验证则载入共识机制插件，未开启则不加载，默认开启加载noops共识机制
	// 当前节点是否是验证节点，是验证节点加载共识机制引擎，非验证节点不加载共识引擎，设置针对单点作用非全局
	// 节点启动默认为验证节点， 节点是否加载共识引擎，决定了节点的消息处理方法peer.handlerFactory被不同的值初始化
	if peer.ValidatorEnabled() {
		logger.Debug("Running as validating peer - making genesis block if needed")
		// 生成创世块
		makeGenesisError := genesis.MakeGenesis()
		if makeGenesisError != nil {
			return makeGenesisError
		}
		logger.Debugf("Running as validating peer - installing consensus %s",
			viper.GetString("peer.validator.consensus"))

		// helper.GetEngine引擎工厂方法，调用这个获取共识引擎
		peerServer, err = peer.NewPeerWithEngine(secHelperFunc, helper.GetEngine)
	} else {
		// 不采用共识机制，不加载engine，消息处理方法为peer.NewPeerHandler
		logger.Debug("Running as non-validating peer")
		peerServer, err = peer.NewPeerWithHandler(secHelperFunc, peer.NewPeerHandler)
	}

	if err != nil {
		logger.Fatalf("Failed creating new peer with handler %v", err)

		return err
	}

	// Register the Peer server
	// 在gRPC服务器中注册peer服务
	pb.RegisterPeerServer(grpcServer, peerServer)

	// Register the Admin server
	// 注册Admin服务器端
	pb.RegisterAdminServer(grpcServer, core.NewAdminServer())

	// Register Devops server
	// 注册Devops服务器,peer devops client会通过gPRC连接调用这里实现的方法
	serverDevops := core.NewDevopsServer(peerServer)
	pb.RegisterDevopsServer(grpcServer, serverDevops)

	// Register the ServerOpenchain server
	serverOpenchain, err := rest.NewOpenchainServerWithPeerInfo(peerServer)
	if err != nil {
		err = fmt.Errorf("Error creating OpenchainServer: %s", err)
		return err
	}

	pb.RegisterOpenchainServer(grpcServer, serverOpenchain)

	// Create and register the REST service if configured
	// 如果开启REST则注册该服务器
	if viper.GetBool("rest.enabled") {
		go rest.StartOpenchainRESTServer(serverOpenchain, serverDevops)
	}

	logger.Infof("Starting peer with ID=%s, network ID=%s, address=%s, rootnodes=%v, validator=%v",
		peerEndpoint.ID, viper.GetString("peer.networkId"), peerEndpoint.Address,
		viper.GetString("peer.discovery.rootnode"), peer.ValidatorEnabled())

	// Start the grpc server. Done in a goroutine so we can deploy the
	// genesis block if needed.
	serve := make(chan error)

	sigs := make(chan os.Signal, 1)
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
	go func() {
		sig := <-sigs
		fmt.Println()
		fmt.Println(sig)
		serve <- nil
	}()

	go func() {
		var grpcErr error
		if grpcErr = grpcServer.Serve(lis); grpcErr != nil {
			grpcErr = fmt.Errorf("grpc server exited with error: %s", grpcErr)
		} else {
			logger.Info("grpc server exited")
		}
		serve <- grpcErr
	}()

	if err := writePid(viper.GetString("peer.fileSystemPath")+"/peer.pid", os.Getpid()); err != nil {
		return err
	}

	// Start the event hub server
	if ehubGrpcServer != nil && ehubLis != nil {
		go ehubGrpcServer.Serve(ehubLis)
	}

	// peer profile 占用端口6060
	if viper.GetBool("peer.profile.enabled") {
		go func() {
			profileListenAddress := viper.GetString("peer.profile.listenAddress")
			logger.Infof("Starting profiling server with listenAddress = %s", profileListenAddress)
			if profileErr := http.ListenAndServe(profileListenAddress, nil); profileErr != nil {
				logger.Errorf("Error starting profiler: %s", profileErr)
			}
		}()
	}

	// Block until grpc server exits
	return <-serve
}
{% endhighlight %}

### 初始化引擎
{% highlight bash %}
// GetEngine returns initialized peer.Engine
// 共识机制引擎工程，返回一个共识引擎
func 	GetEngine(coord peer.MessageHandlerCoordinator) (peer.Engine, error) {
	var err error
	engineOnce.Do(func() {
		engine = new(EngineImpl)
		// 引擎helper和其它stack交互
		// coord 为peer  Impl
		engine.helper = NewHelper(coord)
		// 构建批准者consenter，投票者，有发言权的人

		// consenter 和 helper 互相交换句柄， 目的外部调用内部，内部调用外部
		engine.consenter = controller.NewConsenter(engine.helper)
		engine.helper.setConsenter(engine.consenter)
		engine.peerEndpoint, err = coord.GetPeerEndpoint()
		engine.consensusFan = util.NewMessageFan()

		go func() {
			logger.Debug("Starting up message thread for consenter")

			// The channel never closes, so this should never break
			// 从consensusFan中获取一个制度Channel，并进行消息处理
			// for + channel 循环读，当fan.out 关闭时，循环自动结束
			// channel永不关闭，所以读取从不停止
			for msg := range engine.consensusFan.GetOutChannel() {
				engine.consenter.RecvMsg(msg.Msg, msg.Sender)
			}
		}()
	})
	return engine, err
}

{% endhighlight %}


### 共识处理器
{% highlight bash %}
// NewConsensusHandler constructs a new MessageHandler for the plugin.
// Is instance of peer.HandlerFactory
// 新的共识机制处理器, 验证节点启动时会配置其消息处理方法，如果是验证节点则会根据不同的共识机制加载不同engine的处理方法
// initiatedStream bool，流是否初始化，在节点准备chat时，会先用false来初始化流
// 在handleChat时会获取处理器
func NewConsensusHandler(coord peer.MessageHandlerCoordinator,
	stream peer.ChatStream, initiatedStream bool) (peer.MessageHandler, error) {

	// 先构建节点处理器，后续赋给共识机制处理器
	// peer 的gPRC Chat()实现中initiatedStream 为false
		peerHandler, err := peer.NewPeerHandler(coord, stream, initiatedStream)
	if err != nil {
		return nil, fmt.Errorf("Error creating PeerHandler: %s", err)
	}

	// 共识处理器，同时把上述构建的节点处理器作为其成员
	handler := &ConsensusHandler{
		MessageHandler: peerHandler,
		coordinator:    coord,
	}

	// 共识插件每个连接的最大通信消息数，默认为1000，超过1000则拒绝分发
	consensusQueueSize := viper.GetInt("peer.validator.consensus.buffersize")

	if consensusQueueSize <= 0 {
		logger.Errorf("peer.validator.consensus.buffersize is set to %d, but this must be a positive integer, defaulting to %d", consensusQueueSize, DefaultConsensusQueueSize)
		consensusQueueSize = DefaultConsensusQueueSize
	}

	// 共识机制的处理chan, 初始化1000个消息的channel
	handler.consenterChan = make(chan *util.Message, consensusQueueSize)
	// fanin 扇入（端数）
	// EngineImpl 是一个consensus.Consenter, PeerEndpoint and MessageFan 的结构
	// getEngineImpl() 返回一个 EngineImpl 实例
	getEngineImpl().consensusFan.AddFaninChannel(handler.consenterChan)

	return handler, nil
}
{% endhighlight %}


### 通用处理器
{% highlight bash %}
func NewPeerHandler(coord MessageHandlerCoordinator, stream ChatStream, initiatedStream bool) (MessageHandler, error) {

	d := &Handler{
		ChatStream:      stream,
		initiatedStream: initiatedStream,
		Coordinator:     coord,
	}
	d.doneChan = make(chan struct{})

	if dur := viper.GetDuration("peer.sync.state.snapshot.writeTimeout"); dur == 0 {
		d.syncSnapshotTimeout = DefaultSyncSnapshotTimeout
	} else {
		d.syncSnapshotTimeout = dur
	}

	d.snapshotRequestHandler = newSyncStateSnapshotRequestHandler()
	d.syncStateDeltasRequestHandler = newSyncStateDeltasHandler()
	d.syncBlocksRequestHandler = newSyncBlocksRequestHandler()
	// 有穷状态机
	d.FSM = fsm.NewFSM(
		"created",
		fsm.Events{
			{Name: pb.Message_DISC_HELLO.String(), Src: []string{"created"}, Dst: "established"},
			{Name: pb.Message_DISC_GET_PEERS.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_DISC_PEERS.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_BLOCK_ADDED.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_GET_BLOCKS.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_BLOCKS.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_STATE_GET_SNAPSHOT.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_STATE_SNAPSHOT.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_STATE_GET_DELTAS.String(), Src: []string{"established"}, Dst: "established"},
			{Name: pb.Message_SYNC_STATE_DELTAS.String(), Src: []string{"established"}, Dst: "established"},
		},
		fsm.Callbacks{
			"enter_state":                                           func(e *fsm.Event) { d.enterState(e) },
			"before_" + pb.Message_DISC_HELLO.String():              func(e *fsm.Event) { d.beforeHello(e) },
			"before_" + pb.Message_DISC_GET_PEERS.String():          func(e *fsm.Event) { d.beforeGetPeers(e) },
			"before_" + pb.Message_DISC_PEERS.String():              func(e *fsm.Event) { d.beforePeers(e) },
			"before_" + pb.Message_SYNC_BLOCK_ADDED.String():        func(e *fsm.Event) { d.beforeBlockAdded(e) },
			"before_" + pb.Message_SYNC_GET_BLOCKS.String():         func(e *fsm.Event) { d.beforeSyncGetBlocks(e) },
			"before_" + pb.Message_SYNC_BLOCKS.String():             func(e *fsm.Event) { d.beforeSyncBlocks(e) },
			"before_" + pb.Message_SYNC_STATE_GET_SNAPSHOT.String(): func(e *fsm.Event) { d.beforeSyncStateGetSnapshot(e) },
			"before_" + pb.Message_SYNC_STATE_SNAPSHOT.String():     func(e *fsm.Event) { d.beforeSyncStateSnapshot(e) },
			"before_" + pb.Message_SYNC_STATE_GET_DELTAS.String():   func(e *fsm.Event) { d.beforeSyncStateGetDeltas(e) },
			"before_" + pb.Message_SYNC_STATE_DELTAS.String():       func(e *fsm.Event) { d.beforeSyncStateDeltas(e) },
		},
	)

	// If the stream was initiated from this Peer, send an Initial HELLO message
	// Chat()时该值为false, 为执行初始化不发送HELLO消息
	if d.initiatedStream {
		// Send intiial Hello
		helloMessage, err := d.Coordinator.NewOpenchainDiscoveryHello()
		if err != nil {
			return nil, fmt.Errorf("Error getting new HelloMessage: %s", err)
		}
		if err := d.SendMessage(helloMessage); err != nil {
			return nil, fmt.Errorf("Error creating new Peer Handler, error returned sending %s: %s", pb.Message_DISC_HELLO, err)
		}
	}

	return d, nil
}
{% endhighlight %}



### 节点处理通信
{% highlight bash %}
func (p *Impl) handleChat(ctx context.Context, stream ChatStream, initiatedStream bool) error {
	deadline, ok := ctx.Deadline()
	peerLogger.Debugf("Current context deadline = %s, ok = %v", deadline, ok)
	// 调用节点的处理器，可能为带共识引擎的，也可能为普通的
	// stream作为处理器的处理对象
	// initiatedStream 流是否初始化
	// handlerFactory 为处理器工厂方法，在peer启动时已经被设置好，此时相当于调用获得一个处理器；
	// 如果为验证节点此处相当于调用了，共识引擎的处理器工厂方法NewConsensusHandler
	// 如果是Chat()调用此时p为stream.Context(), 这里为Impl， stream 为 background 后续会参数构建peer.Handler
	handler, err := p.handlerFactory(p, stream, initiatedStream)
	if err != nil {
		return fmt.Errorf("Error creating handler during handleChat initiation: %s", err)
	}
	defer handler.Stop()
	for {
		// 从流中读取数据
		// stream 为 ConsensusHandler内的peer.MessageHandler即peer.Handler的ChatStream
		in, err := stream.Recv()
		// 异常判断
		if err == io.EOF {
			peerLogger.Debug("Received EOF, ending Chat")
			return nil
		}
		if err != nil {
			e := fmt.Errorf("Error during Chat, stopping handler: %s", err)
			peerLogger.Error(e.Error())
			return e
		}
		// 节点消息处理调用，共识机制的验证点击采用ConsensusHandler处理，不同的采用peer.Handler处理
		err = handler.HandleMessage(in)
		if err != nil {
			peerLogger.Errorf("Error handling message: %s", err)
			//return err
		}
	}
}
{% endhighlight %}


