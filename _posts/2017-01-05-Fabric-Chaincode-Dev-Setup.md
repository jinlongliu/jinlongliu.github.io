---
layout: post
title: "Fabric V0.6.1 Chaincode开发环境设置"
description: ""
category: 区块链 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
<p>
本文记录介绍Fabric V0.6 Chaincode开发环境搭建
</p>

### 准备介绍
- Windows 7 64bit    192.168.1.106
- VMware Workstation 12.1 : Ubuntu 14.04 64bit  192.168.78.130
- Ubuntu 14.04 由iso安装，安装之后装了vim
- vmware的ubuntu最好安装vmware-tools 这样你本地的maven仓库可以共享到虚拟机，
然后到容器，这样在编译java chaincode时可以复用本地包

### 安装docker
- 可以通过网络百度安装方法，Fabric开发官方推荐是1.12+
- 或者通过下载本文中的安装附件，在dockerDEB目录中通过dpkg -i *.deb

### 安装docker-compose
- 安装pip，在pipDEB目录中dpkg -i *.deb
- pip install --upgrade pip
- pip install behave nose docker-compose

### 下载docker镜像，可以从官方pull
- docker pull hyperledger/fabric-peer:latest
- docker pull hyperledger/fabric-membersrvc:latest
- 在下载附件FabricDockerImages目录中通过命令导入
- docker load < fabric-peer.tar
- docker load < fabric-membersrvc.tar
- docker tag hyperledger/fabric-membersrvc:x86_64-0.6.1-preview  hyperledger/fabric-membersrvc:latest
- membersrvc导入默认tag是0.6.1的，tag为lattest后续docker-compose中用latest
{%highlight bash%}
root@fabric:~# docker images
REPOSITORY                      TAG                    IMAGE ID            CREATED             SIZE
hyperledger/fabric-membersrvc   latest                 b3654d32e4f9        11 weeks ago        1.417 GB
hyperledger/fabric-membersrvc   x86_64-0.6.1-preview   b3654d32e4f9        11 weeks ago        1.417 GB
hyperledger/fabric-peer         latest                 21cb00fb27f4        11 weeks ago        1.424 GB
{%endhighlight%}

### 编辑docker-compose文件
- 在deploy目录vim docker-compose.yml
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
  volumes:
    - /root/deploy/share:/mnt
    - /mnt/hgfs:/mnt2
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
- 上述中volumes主要是映射两个ubuntu中目录到容器中
- /root/deploy/share:/mnt里面有gradle,jdk8,maven
- /mnt/hgfs:/mnt2里面是Windows上的maven仓库共享到ubuntu里面，再通过这个传递到容器中

### 由ubuntu共享到容器中
{%highlight bash%}
apache-maven-3.3.9  gradle-3.3  jdk1.8.0_92  profile

#用于在容器中引用的环境变量
vim profile
export TERM=linux
export PATH=$PATH:/mnt/gradle-3.3/bin

export JAVA_HOME=/mnt/jdk1.8.0_92
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

export M2_HOME=/mnt/apache-maven-3.3.9
export M2=$M2_HOME/bin
export PATH=$M2:$PATH
{%endhighlight%}

### 启动区块链
- 在ubuntu上deploy目录docker-compose up 会自动启动镜像，当前SSH终端会被日志冲刷
- 在新SSH终端上查看
{%highlight bash%}
root@fabric:~# docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
ed8e587b50dc        hyperledger/fabric-peer         "sh -c 'sleep 5; peer"   41 minutes ago      Up 41 minutes       0.0.0.0:7050-7051->7050-7051/tcp, 0.0.0.0:7053->7053/tcp   deploy_vp0_1
b9d05b3d5937        hyperledger/fabric-membersrvc   "membersrvc"             41 minutes ago      Up 41 minutes       0.0.0.0:7054->7054/tcp                                     deploy_membersrvc_1
{%endhighlight%}
- 接入容器
{%highlight bash%}
docker exec -ti ed8e587b50dc /bin/bash
{%endhighlight%}
- 接入容器后，因为我们把ubuntu上的share共享到容器的/mnt目录此时我们可以查看使用
{%highlight bash%}
$source /mnt/profile

root@ed8e587b50dc:~# java -version
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)

root@ed8e587b50dc:~# gradle -v

------------------------------------------------------------
Gradle 3.3
------------------------------------------------------------

Build time:   2017-01-03 15:31:04 UTC
Revision:     075893a3d0798c0c1f322899b41ceca82e4e134b

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_92 (Oracle Corporation 25.92-b14)
OS:           Linux 4.4.0-31-generic amd64

#maven的环境变量和使用是可选的，因为java chaincode使用gradle编译
root@ed8e587b50dc:~# mvn --version
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T16:41:47+00:00)
Maven home: /mnt/apache-maven-3.3.9
Java version: 1.8.0_92, vendor: Oracle Corporation
Java home: /mnt/jdk1.8.0_92/jre
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "4.4.0-31-generic", arch: "amd64", family: "unix"
{%endhighlight%}

### 编译Java chaincode
- 在容器中区块链节点已经运行，并预制了用户，容器中GOPATH在启动中已设置好。
{%highlight bash%}
#如果使用maven本地库映射进去会有一定加速，也可以不映射进去直接在线下载
cd $GOPATH/src/github.com/hyperledger/fabric/core/chaincode/shim/java
gradle -b build.gradle clean
gradle -b build.gradle build

cd $GOPATH/src/github.com/hyperledger/fabric/examples/chaincode/java/SimpleSample
gradle -b build.gradle build
    
gradle -b build.gradle run

You are using Gradle 3.3:  This version of the protobuf plugin works with Gradle version 2.12+
The TaskInputs.source(Object) method has been deprecated and is scheduled to be removed in Gradle 4.0. Please use TaskInputs.file(Object).skipWhenEmpty() instead.
:examples:chaincode:java:SimpleSample:compileJava UP-TO-DATE
:examples:chaincode:java:SimpleSample:processResources UP-TO-DATE
:examples:chaincode:java:SimpleSample:classes UP-TO-DATE
:examples:chaincode:java:SimpleSample:run
Jan 05, 2017 3:50:45 PM io.grpc.internal.TransportSet$1 call
INFO: Created transport io.grpc.netty.NettyClientTransport@165e8b88(/127.0.0.1:7051) for /127.0.0.1:7051
Jan 05, 2017 3:50:46 PM io.grpc.internal.TransportSet$TransportListener transportReady
INFO: Transport io.grpc.netty.NettyClientTransport@165e8b88(/127.0.0.1:7051) for /127.0.0.1:7051 is ready
Jan 05, 2017 3:52:10 PM example.SimpleSample run
INFO: In run, function:init
Jan 05, 2017 3:53:31 PM example.SimpleSample run
INFO: In run, function:transfer
in transfer
Transfer a>b am='10' new values='90','210'
Transfer complete
null
Jan 05, 2017 3:53:49 PM example.SimpleSample run
INFO: In run, function:transfer
in transfer
Transfer a>b am='10' new values='80','220'
Transfer complete
null
Jan 05, 2017 3:53:50 PM example.SimpleSample run
INFO: In run, function:transfer
in transfer
Transfer a>b am='10' new values='70','230'
Transfer complete
null
> Building 75% > :examples:chaincode:java:SimpleSample:run
{%endhighlight%}

### 再新建SSH终端，接入容器,部署chaincode、调动chaincode处理交易、查询账户
{%highlight bash%}
docker exec -ti ed8e587b50dc /bin/bash

peer network login jim
#询问密码是输入 6avZQLwcUe9b， jim 和 6avZQLwcUe9b为预制

#deploy,invoke,query
peer chaincode deploy -u jim -l java -n SimpleSample -c '{"Args": ["init", "a","100", "b", "200"]}'

peer chaincode invoke -u jim -l java -n SimpleSample -c '{"Args": ["transfer", "a", "b", "10"]}'
16:33:14.451 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'chaincode'
16:33:14.453 [chaincodeCmd] getChaincodeSpecification -> INFO 002 Local user 'jim' is already logged in. Retrieving login token.
16:33:15.012 [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Successfully invoked transaction: chaincodeSpec:<type:JAVA chaincodeID:<name:"SimpleSample" > ctorMsg:<args:"transfer" args:"a" args:"b" args:"10" > secureContext:"jim" > (f7369511-68dd-4793-a77d-5dccbaf2b0b2)
16:33:15.012 [main] main -> INFO 004 Exiting.....

peer chaincode query -u jim -l java -n SimpleSample -c '{ "Args": ["query", "a"]}'
16:33:30.740 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'chaincode'
16:33:30.742 [chaincodeCmd] getChaincodeSpecification -> INFO 002 Local user 'jim' is already logged in. Retrieving login token.
16:33:31.337 [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Successfully queried transaction: chaincodeSpec:<type:JAVA chaincodeID:<name:"SimpleSample" > ctorMsg:<args:"query" args:"a" > secureContext:"jim" > 
Query Result: {"Name":"a","Amount":"60"}
16:33:31.337 [main] main -> INFO 004 Exiting.....
{%endhighlight%}

### Rest API (V1.0 REST API将弃用)
{%highlight bash%}
root@fabric:~# curl 127.0.0.1:7050/chain
{
    "height": 6,
    "currentBlockHash": "UPlJDYyOBwc+wtbwDqoYUxsqnBerbEeIe/DqEL1KfgXOt5+Whkq6PTalMdgOJeDTd1SpZAkH8+gY1cNtJOt//Q==",
    "previousBlockHash": "xbvUoAaE54XyBg5yQaavC+U4gmQUJg8Hf12cizmB6V1rVWzUBdamxjdPVkNWBJr93K3/Ez/yyBlz5c3gC2BZsw=="
}

curl 127.0.0.1:7050/chain/blocks/1
#如果在windows使用postman 查询，请求发往http://192.168.78.130:7050/chain/blocks/1
{%endhighlight%}

- GET /chain/blocks/{Block}
- GET /chain
- POST /chaincode
- GET /network/peers
- POST /registrar
- DELETE /registrar/{enrollmentID}
- GET /registrar/{enrollmentID}
- GET /registrar/{enrollmentID}/ecert
- GET /registrar/{enrollmentID}/tcert
- GET /transactions/{UUID}

### Golang chaincode调用
- 在容器中或者本地编一个chaincode
{%highlight bash%}
#编译在bin目录下生成可执行文件，代码里有main函数
cd $GOPATH/src/github.com/chaincode_example0
go build

#CORE_CHAINCODE_ID_NAME，CORE_PEER_ADDRESS 为环境变量
#linux上用export，dos用set
#程序会启动一个与validating peer的gRPC通道供其调用
CORE_CHAINCODE_ID_NAME=mycc CORE_PEER_ADDRESS=0.0.0.0:7051 ./chaincode_example02
{%endhighlight%}
- 在其它SSH终端中部署chaincode
{%highlight bash%}
peer network login jim
#输入密码6avZQLwcUe9b

peer network login jim -p 6avZQLwcUe9b

peer chaincode deploy -u jim -l golang -n mycc -c '{"Args": ["init", "a","100", "b", "200"]}' 
peer chaincode invoke -u jim -l golang -n mycc -c '{"Args": ["transfer", "a", "b", "10"]}'
peer chaincode query -u jim -l golang -n mycc -c '{ "Args": ["query", "a"]}'
peer chaincode query -u jim -l golang -n mycc -c '{"Args": ["query", "b"]}'
{%endhighlight%}

# 小结
- 耗时在拉取fabric镜像，编译java chaincode最好在容器内编译，尝试在windows编译java失败，报错较多。
- 参考文档 [Hyperledger Fabric](http://hyperledger-fabric.readthedocs.io/en/v0.6/)






