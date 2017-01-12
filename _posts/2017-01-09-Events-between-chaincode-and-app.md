---
layout: post
title: "应用监听获取Chaincode事件实验"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
<p>
本次介绍应用如何获取到chaincode里面传递的事件
</p>



## 实验操作

### 背景知识
- 你已经回启动基础的Fabric环境
- 你已经能够掌握部署chaincode
- 监听程序  src/github.com/hyperledger/fabric/examples/events/block-listener/block-listener.go
- 发送事件的chaincode src/github.com/hyperledger/fabric/examples/chaincode/go/eventsender/eventsender.go

### 基础环境
- 清除所有运行容器和dev-vp开头镜像【可选】
{%highlight bash%}
docker ps -a | awk 'NR >1 {print $1}' |xargs -I {} docker rm -f {} 
docker images | awk 'NR >1 {print $1}' | awk '/dev-vp*/'|xargs -I {} docker rmi {}
{%endhighlight%}
- 启动1 membersrvc和4 vp节点
{%highlight bash%}
docker-compose -f docker-compose-4-consensus-batch.yml up
{%endhighlight%}

### 事件监听应用
- 编译和启动
{%highlight bash%}
#进入任一vp容器，可以简单docker exec -ti xxxxxx bash
docker exec -ti $(docker ps | awk 'NR==4 {print $1}') bash

#进入应用目录准备编译
cd $GOPATH/src/github.com/hyperledger/fabric/examples/events/block-listener/
go build
./block-listener
Event Address: 0.0.0.0:7053
#此时监听了系统事件，在新的terminal操作

{%endhighlight%}

### 部署发送事件的Chaincode
{%highlight bash%}
#进入任一vp容器，可以简单docker exec -ti xxxxxx bash
docker exec -ti $(docker ps | awk 'NR==4 {print $1}') bash

#定义chaincode路径
export CCPATH=github.com/hyperledger/fabric/examples/chaincode/go/eventsender

#登录用户
peer network login admin -p Xurw3yU9zI0l
12:54:49.314 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'network'
12:54:49.314 [networkCmd] networkLogin -> INFO 002 CLI client login...
12:54:49.314 [networkCmd] networkLogin -> INFO 003 Local data store for client loginToken: /var/hyperledger/production/client/
12:54:49.315 [networkCmd] networkLogin -> INFO 004 Logging in user 'admin' on CLI interface...
12:54:49.381 [networkCmd] networkLogin -> INFO 005 Storing login token for user 'admin'.
12:54:49.382 [networkCmd] networkLogin -> INFO 006 Login successful for user 'admin'.
12:54:49.383 [main] main -> INFO 007 Exiting.....

#部署chaincode
peer chaincode deploy -u admin -l golang -c '{"Args": ["init"]}' -p $CCPATH
12:54:55.827 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'chaincode'
12:54:55.828 [chaincodeCmd] getChaincodeSpecification -> INFO 002 Local user 'admin' is already logged in. Retrieving login token.
12:54:58.543 [chaincodeCmd] chaincodeDeploy -> INFO 003 Deploy result: type:GOLANG chaincodeID:<path:"github.com/hyperledger/fabric/examples/chaincode/go/eventsender" name:"5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b125f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686" > ctorMsg:<args:"init" > 
Deploy chaincode: 5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b125f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686
12:54:58.543 [main] main -> INFO 004 Exiting.....

#上述Deploy chaincode: 后内容为chaincodeid后续要用
#如果deploy成功部署完毕，监听那边会有日志
{%endhighlight%}

### 事件监听
- 此时在之前的终端已经捕捉到了系统事件
{%highlight bash%}
root@95f947a20974:/opt/gopath/src/github.com/hyperledger/fabric/examples/events/block-listener# ./block-listener 
Event Address: 0.0.0.0:7053


Received block
--------------
Transaction:
        [type:CHAINCODE_DEPLOY chaincodeID:"\n?g......................
        
{%endhighlight%}

### 监听指定事件
{%highlight bash%}
#在第一个终端ctrl+c断开后进行操作
#记录chaincodeid，如果你的官方cc没有任何更改ccid应该和如下一样
export ccid=5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b125f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686

#监听指定chaincode事件
./block-listener -events-from-chaincode $ccid
Event Address: 0.0.0.0:7053

#再次实现了监听
{%endhighlight%}

### 调用chaincode触发事件发送
{%highlight bash%}
#部署chaincode的终端
#记录chaincodeid
export ccid=5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b125f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686

#调用chaincode
peer chaincode invoke -u admin -c '{"Args": ["invoke", "HelloMessage"]}' -n $ccid
13:02:33.745 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'chaincode'
13:02:33.745 [chaincodeCmd] getChaincodeSpecification -> INFO 002 Local user 'admin' is already logged in. Retrieving login token.
13:02:34.155 [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Successfully invoked transaction: chaincodeSpec:<type:GOLANG chaincodeID:<name:"5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b125f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686" > ctorMsg:<args:"invoke" args:"HelloMessage" > secureContext:"admin" > (f4b721c7-019b-4860-9c45-12ceda029b65)
13:02:34.156 [main] main -> INFO 004 Exiting.....

peer chaincode invoke -u admin -c '{"Args": ["invoke", "Hello World"]}' -n $ccid
.....
{%endhighlight%}

{%highlight bash%}
Received block
--------------
Transaction:
        [type:CHAINCODE_INVOKE chaincodeID:"\022\200\0....
        
       
Received chaincode event
------------------------
Chaincode Event:&{chaincodeID:"5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b12<br/>
5f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686" txID:"f4b721c7-019b-4860-9c45-12ceda0<br/>
29b65" eventName:"evtsender" payload:"Event 0,HelloMessage" }

Received block
--------------
Transaction:
        [type:CHAINCODE_INVOKE chaincodeID:"\022\200\0015ebe58961a998670d2a16b32a...


Received chaincode event
------------------------
Chaincode Event:&{chaincodeID:"5ebe58961a998670d2a16b32a30b7cf38712f4e5436f8189b63e2232306d0ae75b12<br/>
5f6594c81f2584bd7a1c218cfc502ab80ad7fb62dd8b1ac8ac2da30bd686" txID:"eee47795-39c4-4c40-ad90-6892708<br/>
21e5e" eventName:"evtsender" payload:"Event 1,Hello World" }
{%endhighlight%}

## 分析总结
- 文件src/github.com/hyperledger/fabric/examples/events/block-listener/block-listener.go 大概43行
{%highlight bash%}
func (a *adapter) GetInterestedEvents() ([]*pb.Interest, error) {
	if a.chaincodeID != "" {
		return []*pb.Interest{
			{EventType: pb.EventType_BLOCK},
			{EventType: pb.EventType_REJECTION},
			{EventType: pb.EventType_CHAINCODE,
				RegInfo: &pb.Interest_ChaincodeRegInfo{
					ChaincodeRegInfo: &pb.ChaincodeReg{
						ChaincodeID: a.chaincodeID,
						EventName:   ""}}}}, nil
	}
}

#设置了3个感兴趣的事件EventType_BLOCK，EventType_REJECTION，EventType_CHAINCODE
#应该是可以进一步对监听事件的类型做明确，在一次部署cc是我们监听到的是EventType_BLOCK
{%endhighlight%}

-文件src/github.com/hyperledger/fabric/examples/chaincode/go/eventsender/eventsender.go
{%highlight bash%}
func (t *EventSender) Invoke(stub shim.ChaincodeStubInterface, function string, args []string) ([]byte, error) {
	b, err := stub.GetState("noevents")
	if err != nil {
		return nil, errors.New("Failed to get state")
	}
	#事件序列号
	noevts, _ := strconv.Atoi(string(b))
    #事件内容拼接
	tosend := "Event " + string(b)
	for _, s := range args {
		tosend = tosend + "," + s
	}
	err = stub.PutState("noevents", []byte(strconv.Itoa(noevts+1)))
	if err != nil {
		return nil, err
	}
    # 发送事件，事件名称evtsender和我们监听中一致
	err = stub.SetEvent("evtsender", []byte(tosend))
	if err != nil {
		return nil, err
	}
	return nil, nil
}
{%endhighlight%}
