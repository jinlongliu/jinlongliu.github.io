---
layout: post
title: "Chaincode Deploy链码部署"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
<p>
本次介绍chaincode的部署
</p>

### 背景阅读
- 本文针对V0.6.1版本
- chaincode与validator通过shim layer交互
- chaincode vm启动后向validator注册，validator回应registered或者error
- validator通过shim layer调用chaincode vm的init,invoke,query方法
- chaincode.mode  dev 为开发模式本地命令行运行
- chaincode.mode  net 为用docker启动container运行

## 单个validator和单个membersrvc场景
{%highlight bash%}
membersrvc:
  image: hyperledger/fabric-membersrvc
  ports:
    - "7054:7054"
  command: membersrvc
vp0:
  image: hyperledger/fabric-peer
  ports:
    - "7050:7050"
    - "7051:7051"
    - "7053:7053"
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_VM_ENDPOINT=unix:///var/run/docker.sock
    - CORE_LOGGING_LEVEL=DEBUG
    - CORE_PEER_ID=vp0
    - CORE_PEER_PKI_ECA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TCA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TLSCA_PADDR=membersrvc:7054
    - CORE_SECURITY_ENABLED=true
    - CORE_SECURITY_ENROLLID=test_vp0
    - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
  links:
    - membersrvc
  command: sh -c "sleep 5; peer node start --peer-chaincodedev"
{%endhighlight%}

### 分析
- 上述docker-compose中command: sh -c "sleep 5; peer node start --peer-chaincodedev"指定了chaincode.mode
{%highlight bash%}
#src/github.com/hyperledger/fabric/peer/node/start.go Line71...

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
...

#chaincodeDevMode的值在56行通过--peer-chaincodedev 解析获得
{%endhighlight%}
- dev这种模式下通过peer chaincode deploy... 执行validator不会去寻找chaincode的实际字节
- docker-compose.yml中将容器的7051映射到宿主机的7051，所以按照如果你的环境是HOST->VM->CONTAINER
则在HOST访问VM的7051，来简历chaincode和validator的连接进行注册
- 官方教程是编译chaincode为可执行程序
- 设置chaincode注册需要的环境变量
- CORE_CHAINCODE_ID_NAME=mycc      chaincode的id
- CORE_PEER_ADDRESS=192.168.78.128:7051   validator的地址
- 注册成功日志输出类似
{%highlight bash%}
16:03:36.547 [shim] DEBU : Peer address: 192.168.78.128:7051
16:03:36.548 [shim] DEBU : os.Args returns: [chaincode-example.exe]
16:03:36.549 [shim] DEBU : Registering.. sending REGISTER
16:03:36.550 [shim] DEBU : []Received message REGISTERED from shim
16:03:36.550 [shim] DEBU : []Handling ChaincodeMessage of type: REGISTERED(state:created)
16:03:36.550 [shim] DEBU : Received REGISTERED, ready for invocations
{%endhighlight%}
- 此时可以在peer上通过peer chaincode deploy/invoke/query 等进行部署
{%highlight bash%}
peer network login jim -p 6avZQLwcUe9b

peer chaincode deploy -u jim -l golang -n mycc -c '{"Args": ["init", "a","100", "b", "200"]}' 
peer chaincode invoke -u jim -l golang -n mycc -c '{"Args": ["transfer", "a", "b", "10"]}'
peer chaincode query -u jim -l golang -n mycc -c '{ "Args": ["query", "a"]}'
peer chaincode query -u jim -l golang -n mycc -c '{"Args": ["query", "b"]}'
{%endhighlight%}

## 多个validator和单个membersrvc场景
- src/github.com/hyperledger/fabric/bddtests/bdd-docker里面有多重场景的docker-compose文件
- 我们分析docker-compose-4-consensus-batch-1-byzantine.yml
{%highlight bash%}
membersrvc0:
  extends:
    file: compose-defaults.yml
    service: membersrvc

vp0:
  extends:
    file: docker-compose-4-consensus-base.yml
    service: vpBatch
  environment:
    - CORE_PEER_ID=vp0
    - CORE_SECURITY_ENROLLID=test_vp0
    - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
  links:
    - membersrvc0
  #笔者编辑增加
  ports:
    - 7050:7050
    - 7051:7051

vp1:
  extends:
    file: docker-compose-4-consensus-base.yml
    service: vpBatch
  environment:
    - CORE_PEER_ID=vp1
    - CORE_PEER_DISCOVERY_ROOTNODE=vp0:7051
    - CORE_SECURITY_ENROLLID=test_vp1
    - CORE_SECURITY_ENROLLSECRET=5wgHK9qqYaPy
  links:
    - membersrvc0
    - vp0
  #笔者编辑增加
  ports:
    - 8051:7051

vp2:
  extends:
    file: docker-compose-4-consensus-base.yml
    service: vpBatch
  environment:
    - CORE_PEER_ID=vp2
    - CORE_PEER_DISCOVERY_ROOTNODE=vp0:7051
    - CORE_SECURITY_ENROLLID=test_vp2
    - CORE_SECURITY_ENROLLSECRET=vQelbRvja7cJ
  links:
     - membersrvc0
     - vp0
  #笔者编辑增加
  ports:
     - 9051:7051
 
 vp3:
   extends:
     file: docker-compose-4-consensus-base.yml
     service: vpBatch
   environment:
     - CORE_PEER_ID=vp3
     - CORE_PEER_DISCOVERY_ROOTNODE=vp0:7051
     - CORE_SECURITY_ENROLLID=test_vp3
     - CORE_SECURITY_ENROLLSECRET=9LKqKH5peurL
     - CORE_PBFT_GENERAL_BYZANTINE=true
   links:
     - membersrvc0
     - vp0
   #笔者编辑增加
   ports:
     - 10051:7051
{%endhighlight%}
- 基础文件compose-defaults.yml
{%highlight bash%}
vp:
  image: hyperledger/fabric-peer
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    # 此处为笔者新增，如果不增加默认采用net方式需要在节点启动docker
    - CORE_CHAINCODE_MODE=dev
    # TODO:  This is currently required due to BUG in variant logic based upon log level.
    - CORE_LOGGING_LEVEL=DEBUG
  # Script will wait until membersrvc is up (if it exists) before starting
  # $$GOPATH (double dollar) required to prevent docker-compose doing its own
  # substitution before the value gets to the container
  command: sh -c "exec $$GOPATH/src/github.com/hyperledger/fabric/bddtests/scripts/start-peer.sh"
  # Ideally we'd only mount /var/run/docker.sock but v1.5.2 of docker-compose
  # does not have to capability to mount individual files. Hence we mount the
  # entire folder in specific folder and specify it explicitly above.
  # This issue seems to be sorted in docker-compose 1.8.0 however that requires
  # Docker 1.10 and CI isn't at that version yet.
  volumes:
    - /var/run/:/host/var/run/

  # Use these options if coverage desired for peers
  #image: hyperledger/fabric-peer-coverage
  #command: ./peer.test --test.coverprofile=coverage.cov node start
membersrvc:
   image: hyperledger/fabric-membersrvc
   command: membersrvc
{%endhighlight%}

### 分析
- 笔者通过把4个peer的7051映射成为VM的7051，8051，9051，10051后续要在HOST上用chaincode可执行程序向这些端口进行注册
- vp0的7050端口映射到VM的7051方便REST调用
- 在HOST上4个terminal分别执行chaincode可执行程序简历gRPC连接进行注册
- 在VM上peer chaincode invoke... 观察HOST上日志输出
- 在HOST利用工具进行REST调用测试
- 192.168.78.128:7050/chain
- 192.168.78.128:7050/chain/blocks/0  创世块
- 192.168.78.128:7050/chain/blocks/1  Deploy TX  txid为chaincode name id
- 192.168.78.128:7050/chain/blocks/2  交易转账 txid 45b56d3a-0a8a-4492-b527-5cc7b1eb6551  交易类型2
- 192.168.78.128:7050/transactions/45b56d3a-0a8a-4492-b527-5cc7b1eb6551  交易类型2 
- 192.168.78.128:7050/network/peers  节点查询
{%highlight bash%}
{
    "peers": [
        {
            "ID": {
                "name": "vp2"
            },
            "address": "172.17.0.6:7051",
            "type": 1,
            "pkiID": "9jECmCLyUr1U4yN43WhpasW+hGLpbBXwQXJnCyxomcs="
        },
        {
            "ID": {
                "name": "vp3"
            },
            "address": "172.17.0.4:7051",
            "type": 1,
            "pkiID": "ply0tf7n0o2e7MDmoGwyK2Rg6XuO+Zv8oBV5vGQi/H0="
        },
        {
            "ID": {
                "name": "vp1"
            },
            "address": "172.17.0.5:7051",
            "type": 1,
            "pkiID": "iNrePYrAiWxbmlFoquQwT5gGgmy7rLgutAeAAkjHvnk="
        },
        {
            "ID": {
                "name": "vp0"
            },
            "address": "172.17.0.3:7051",
            "type": 1,
            "pkiID": "xTMo31jTxS0e1D2Rums6RaCCDonYouXyjI0mGZqzCdg="
        }
    ]
}
{%endhighlight%}

### peer chaincode deploy/invoke/query
- src/github.com/hyperledger/fabric/peer/chaincode/deploy.go 执行主程序，获取参数、peerCient进行调用peer method
- src/github.com/hyperledger/fabric/peer/node/start.go  Line 71 peer node start --peer-chaincodedev 指定chaincode.mode
- src/github.com/hyperledger/fabric/core/devops.go peerClient 调用实际执行内容，会根据chaincode.mode进行相关调用

# 小结
- 笔者在初始测试多validator时发生peer chaincode deploy 代码分析后发现chaincode.mode参数
- chaincode.mode  dev 开发模式，chaincode通过手动运行注册到validator
- chaincode.mode  net chaincode通过docker运行【本文未介绍】






