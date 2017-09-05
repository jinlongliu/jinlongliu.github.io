---
layout: post
title: "Hyperledger Fabric V1.0.2基于Kafka的order服务"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 软件硬件环境
- Windows10 x64
- ThinkPad T450 12G + 500G SSD
- 前期部署参见上篇博客

### 说明
- Zookeeper 分布式的，开放源码的分布式应用程序协调服务
- Kafka 高吞吐量的分布式发布订阅消息系统
- Kafka-manager kafka集群管理工具，笔者的是已经重新tag的，docker pull sheepkiller/kafka-manager:alpine获取
- Kafka-manager 只是UI管理工具，在此次试验中存在与否不重要，km.example.com容器是完全可以注释掉的

### 原始E2E例子yaml解析
{% highlight bash %}
#原始例子执行效果在此不再累述
#src/github.com/hyperledger/fabric/examples/e2e_cli目录内为使用率比较高的示例

#src/github.com/hyperledger/fabric/examples/e2e_cli/configtx.yaml
#104行  Orderer.OrdererType
OrdererType: solo
#上述定义了order服务的类型sole，还可以修改为其它类型
OrdererType: kafka

#如果配置了kafka类型，则需要配置Kafka相关信息 Orderer.Kafka.Brokers
Kafka:
  Brokers:
    - 127.0.0.1:9092
  Version: 0.10.2.0
  
#version如果使用自带kafka镜像可以不用配置，第一阶段可不配

#笔者修改后的配置，需要配合其它配置使用
Kafka:
  Brokers:
    - kafka.example.com:9092
{% endhighlight %}

### 新增Kafka相关
{% highlight bash %}
#在下述文件配置新容器
src/github.com/hyperledger/fabric/examples/e2e_cli/base/docker-compose-base.yaml
src/github.com/hyperledger/fabric/examples/e2e_cli/docker-compose-cli.yaml

#docker-compose-base.yaml services下新增
  zookeeper.example.com:
    container_name: zookeeper.example.com
    image: hyperledger/fabric-zookeeper
    environment:
      - ZOO_MY_ID=1

  kafka.example.com:
    container_name: kafka.example.com
    image: hyperledger/fabric-kafka
    environment:
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka.example.com:9092
      - KAFKA_BROKER_ID=1
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper.example.com:2181
    ports:
      - "9092:9092"

  km.example.com:
    container_name: km.example.com
    image: kafka-manager:alpine
    environment:
      - ZK_HOSTS=zookeeper.example.com:2181
    ports:
      - "19000:9000"
      
#docker-compose-cli.yaml services下新增
  zookeeper.example.com:
    extends:
      file: base/docker-compose-base.yaml
      service: zookeeper.example.com
    container_name: zookeeper.example.com

  kafka.example.com:
    extends:
      file: base/docker-compose-base.yaml
      service: kafka.example.com
    container_name: kafka.example.com

  km.example.com:
    extends:
      file: base/docker-compose-base.yaml
      service: km.example.com
    container_name: km.example.com
    
#fabric-zookeeper,fabric-kafka 上篇博客中构建出来，kafka-manager镜像可以在宿主机拷贝到fabirc目录通过docker load -i xxxx.img导入

#src/github.com/hyperledger/fabric/devenv/Vagrantfile 新增映射
config.vm.network :forwarded_port, guest: 19000, host: 19000, id: "kafka-manager", host_ip: "localhost", auto_correct: true # Portainer service
#客户机内kafka-manger容器9000映射到客户机端口19000，vagrant配置将客户机19000映射到宿主机19000，这样可以直接访问localhost:19000管理kafka
{% endhighlight %}

### 修改启动脚本
{% highlight bash %}
src/github.com/hyperledger/fabric/examples/e2e_cli/network_setup.sh
#上述脚本被用来创建容器
#Line 37，原来脚本删除所有容器，但是portainer也运行，所以修改为删除hyperledger关键字的容器，./network_setup.sh down生效
CONTAINER_IDS=$(docker ps | grep hyperledger |awk '{print $1}')


src/github.com/hyperledger/fabric/examples/e2e_cli/scripts/script.sh
#上述脚本为cli容器内执行脚本，创建channel和安装chaincode等操作
#Line 191新增，等待延时
## Create channel
echo "Creating channel..."
sleep 60
createChannel

#Line 63 新增超时参数
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx -t 30 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA >&log.txt

#上述操作在solo模式下不需要，但是在kafka模式下尤为重要，channel创建前需要等待order服务就绪，order服务需要创建topic并简单测试
#创建channel过程需要延时等待返回，因为kafka模式比solo模式慢
{% endhighlight %}

### 启动脚本
{% highlight bash %}
#在宿主机内修改完毕后，可以vagrant ssh登录客户机在el2_cli目录执行脚本
./network_setup.sh up
....
Creating kafka.example.com
Creating cli

 ____    _____      _      ____    _____           _____   ____    _____
/ ___|  |_   _|    / \    |  _ \  |_   _|         | ____| |___ \  | ____|
\___ \    | |     / _ \   | |_) |   | |    _____  |  _|     __) | |  _|
 ___) |   | |    / ___ \  |  _ <    | |   |_____| | |___   / __/  | |___
|____/    |_|   /_/   \_\ |_| \_\   |_|           |_____| |_____| |_____|

Channel name : mychannel
Creating channel...

#脚本输出到上述位置等待60秒
#此时查看order容器日志可以观察其服务日志会到达如下位置，kafka-manager上创建了消息主题testchaindid
#order容器日志可以使用docker logs -f orderer.example.com查看，或者在portainer界面查看

......
[36m2017-09-05 02:39:57.163 UTC [orderer/kafka] try -> DEBU 0f8[0m [channel: testchainid] Connecting to the Kafka cluster
[36m2017-09-05 02:39:57.168 UTC [orderer/kafka] try -> DEBU 0f9[0m [channel: testchainid] Error is nil, breaking the retry loop
2017-09-05 02:39:57.168 UTC [orderer/kafka] startThread -> INFO 0fa[0m [channel: testchainid] Parent consumer set up successfully
2017-09-05 02:39:57.168 UTC [orderer/kafka] setupChannelConsumerForChannel -> INFO 0fb[0m [channel: testchainid] Setting up the channel consumer for this channel (start offset: -2)...
[36m2017-09-05 02:39:57.168 UTC [orderer/kafka] try -> DEBU 0fc[0m [channel: testchainid] Connecting to the Kafka cluster
[36m2017-09-05 02:39:57.205 UTC [orderer/kafka] try -> DEBU 0fd[0m [channel: testchainid] Error is nil, breaking the retry loop
2017-09-05 02:39:57.205 UTC [orderer/kafka] startThread -> INFO 0fe[0m [channel: testchainid] Channel consumer set up successfully
2017-09-05 02:39:57.205 UTC [orderer/kafka] startThread -> INFO 0ff[0m [channel: testchainid] Start phase completed successfully
[36m2017-09-05 02:39:57.259 UTC [orderer/kafka] processMessagesToBlocks -> DEBU 100[0m [channel: testchainid] Successfully unmarshalled consumed message, offset is 0. Inspecting type...
[36m2017-09-05 02:39:57.259 UTC [orderer/kafka] processConnect -> DEBU 101[0m [channel: testchainid] It's a connect message - ignoring

#上述单zookeeper，kafka的情况速度还是很快的，笔者构建的3 zookeeper，4 kafka环境初期会出现ordering长时间post CONNECT message循环现象，
#后采取./network_setup.sh down之后，再次up，因为kafka集群构建在脚本之外所以第二次执行时，成功概率大。

#60秒延时等待后会进行channel创建，chaincode安装，账户初始化以及转账，结束输出如下
2017-09-05 02:42:04.172 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: 675ECB9D3055BFDCDE6FED02185171B2FC6FCC5570C1D05C08CEDF0DA1BE7315
Query Result: 90
2017-09-05 02:42:25.522 UTC [main] main -> INFO 007 Exiting.....
===================== Query on PEER3 on channel 'mychannel' is successful =====================

===================== All GOOD, End-2-End execution completed =====================


 _____   _   _   ____            _____   ____    _____
| ____| | \ | | |  _ \          | ____| |___ \  | ____|
|  _|   |  \| | | | | |  _____  |  _|     __) | |  _|
| |___  | |\  | | |_| | |_____| | |___   / __/  | |___
|_____| |_| \_| |____/          |_____| |_____| |_____|


#登录CLI容器验证实验结果，docker exec或者portainer界面执行
$ docker exec -it cli bash

#查询账户a
root@61817f344e9a:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
2017-09-05 02:49:03.470 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-09-05 02:49:03.471 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-09-05 02:49:03.471 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2017-09-05 02:49:03.471 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2017-09-05 02:49:03.472 UTC [msp/identity] Sign -> DEBU 005 Sign: plaintext: 0A95070A6708031A0C089FA1B8CD0510...6D7963631A0A0A0571756572790A0161
2017-09-05 02:49:03.472 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: 74E35A869073CE0A96AB1F7029A2D3547BB8034E0A2EA28A2529603588F51397
Query Result: 90
2017-09-05 02:49:03.490 UTC [main] main -> INFO 007 Exiting.....

#查询账户b
root@61817f344e9a:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","b"]}'
2017-09-05 02:50:49.916 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-09-05 02:50:49.916 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-09-05 02:50:49.916 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2017-09-05 02:50:49.917 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2017-09-05 02:50:49.919 UTC [msp/identity] Sign -> DEBU 005 Sign: plaintext: 0A95070A6708031A0C0889A2B8CD0510...6D7963631A0A0A0571756572790A0162
2017-09-05 02:50:49.919 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: 28399730072D349D22ED492850DD08148022AAEF93987D73E8DC4B15A119C0C0
Query Result: 220
2017-09-05 02:50:49.934 UTC [main] main -> INFO 007 Exiting.....

#转账
root@61817f344e9a:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger
/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
2017-09-05 02:50:20.811 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2017-09-05 02:50:20.811 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2017-09-05 02:50:20.823 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2017-09-05 02:50:20.823 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2017-09-05 02:50:20.824 UTC [msp/identity] Sign -> DEBU 005 Sign: plaintext: 0A95070A6708031A0C08ECA1B8CD0510...696E766F6B650A01610A01620A023130
2017-09-05 02:50:20.825 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: B270B7C6027AB216C15AE3782C972CF63E90F63268F90A113DFD7DA8795A41AF
2017-09-05 02:50:20.847 UTC [msp/identity] Sign -> DEBU 007 Sign: plaintext: 0A95070A6708031A0C08ECA1B8CD0510...469A2BC6466625EAC5BECFF5BB924C42
2017-09-05 02:50:20.847 UTC [msp/identity] Sign -> DEBU 008 Sign: digest: D7ECBE84AB8CBC94A53921099FD1F083D2B16E963FB165C5DA6CECC47CA5B91B
2017-09-05 02:50:20.865 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 009 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \224\206_D%\177\2408\243\004_\"\271R\330\
257(:\374\251a\274\221f&\264\214\206\371\334\210\362\022Y\nE\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\003\022-\n\004mycc\022%\n\007\n\001a\022\002\010\004\n\007\n\001b\022\002\010\004\03
2\007\n\001a\032\00280\032\010\n\001b\032\003220\032\003\010\310\001\"\013\022\004mycc\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\200\006-----BEGIN -----\nMIICGTCCAcCgAwIBAgIRALSn7TtUxTIz
jsJD/zeyFSgwCgYIKoZIzj0EAwIwczEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh\nLm9yZzEuZXhhbXBsZS5jb20wHhcNMTc
wOTA1MDIzOTQ1WhcNMjcwOTAzMDIzOTQ1\nWjBbMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN\nU2FuIEZyYW5jaXNjbzEfMB0GA1UEAxMWcGVlcjAub3JnMS5leGFtcGxlLmNvbTBZ\nMBMGByqGSM49AgEGCCqGSM49AwEHA0
IABJqOU/IYUvYcP4vPI1BC5RPDhKT4bX8C\nbOt78i974sfYoYSV20OlsAFGLmQ2Iorl63XjAPa8dP9RptFdDDOfJHSjTTBLMA4G\nA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAIJnfF6LcG3aG\nS+ubEZc5+lTiYE/5/jYrJpsP3NJJq
hOsMAoGCCqGSM49BAMCA0cAMEQCIHIm/9cf\n6QPd4OMErM6k5riXiAWRl4oTZw7iS0m08yVeAiB3HxAII2FuZksTSB5Rh/KqbzAc\nW05bp6/Aj1GoyBwn3Q==\n-----END -----\n" signature:"0D\002 \016v7\205\347\206/\245l\220\272.\36
7\027\300/kfO\003V\264\207O-\253\243\316^\270\350A\002 \035\342g\035\003u\335n\3009\246\346\234/\r\374F\232+\306Ff%\352\305\276\317\365\273\222LB" >
2017-09-05 02:50:20.865 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00a Chaincode invoke successful. result: status:200
2017-09-05 02:50:20.866 UTC [main] main -> INFO 00b Exiting.....

#转账脚本
peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
  -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
  
#上述脚本为src/github.com/hyperledger/fabric/examples/e2e_cli/scripts/script.sh Line 134将里面ORDERER_CA，CHANNEL_NAME替换为实际

{% endhighlight %}

### 删除topic【可跳过，笔者自用】
{% highlight bash %}
#测试过程中可能想全部重新来一次，如何彻底删除topic，当然可以选择全部重构zookeer，kafka
#停止所有producer，consumer,停止kafka或者停止容器，如果将/data/kafka映射到其它目录需要删除数据
#登录zookeeper容器
zkCli.sh -server 127.0.0.1
>rmr /brokers
>rmr /admin
>rmr /config
>rmr /isr_change_notification
>rmr /kafka-manager

#重新启动kafka，整个过程zookeeper可以不需要停止
{% endhighlight %}

### 设置kafka-manager
- 在zookeeper,kafka启动ok后访问kafka-manager暴露的端口，Cluster Zookeeper Hosts填写zookeeper.example.com:2181，从kafka-manager角度这个dns是可解析的
- 勾选Enable JMX，Poll consumer information这样可以观察topic的Latest Offset
- ![SETUP-KM](/upload/2017/setup-km.png)

### 小结
- [Bringing up a Kafka-based Ordering Service](http://hyperledger-fabric.readthedocs.io/en/latest/kafka.html) 官方文档构建参考
- 官方指南是3 zookeeper，4 kafka，本次实验是1 zookeeper，1 kafka构建不包含冗余和可用性存在多处单点故障
- 在创建channel之前order服务必须就绪，创建channel加上延时否则可能会超时失败
- order服务启动之后会创建消息主题testchainid并post若干CONNNECT消息，并自行消费以验证kafka服务可用性


{% include JB/setup %}


