---
layout: post
title: "Chaincode Deploy链码部署net模式"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
<p>
上次介绍chaincode的部署，不过采用的是dev模式，这次记录net模式,fabric通过docker启动chaincode
</p>

### 背景阅读
- 本文针对V0.6.1版本
- validator peer的配置文件core.yaml中配置参数chaincode.mode默认为net，该参数可用CORE_CHAINCODE_MODE覆盖
- chaincode.mode  dev 为开发模式本地命令行运行，开发者自行启动cc服务，cc向vp的7051注册建立两者gRPC通道
- chaincode.mode  net 为用docker启动container运行，在部署时chaincodeSupport根据默认chaincode.(golang/car/java).Dockerfile<br/>
和CLI指定的-p接口自行构建docker image，然后create

## 实验操作
- 采用src/github.com/hyperledger/fabric/bddtests/bdd-docker内的docker-compose文件
- src/github.com/hyperledger/fabric/bddtests/bdd-docker/compose-defaults.yml
- src/github.com/hyperledger/fabric/bddtests/bdd-docker/docker-compose-4-consensus-base.yml
- src/github.com/hyperledger/fabric/bddtests/bdd-docker/docker-compose-4-consensus-batch.yml

### compose-defaults.yml
- 这两定义了两种节点的基础信息作为公共通用部分 vp节点和负责成员管理的membersrvc
{%highlight bash%}
vp:
  image: hyperledger/fabric-peer
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    # TODO:  This is currently required due to BUG in variant logic based upon log level.
    - CORE_LOGGING_LEVEL=DEBUG
    # 笔者自增，利用docker exec -ti时使用clear需要设置TERM变量，此处自己引入
    - TERM=xterm
    # 笔者自增：如果采用dev模式添加如下行，并取消注释
    # - CORE_CHAINCODE_MODE=dev
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

## docker-compose-4-consensus-base.yml
- 定义了vp节点指向membersrvc连接需要的参数，membersrvc服务于端口7054
- 指定了PBFT的副本数，这个在PBFT插件引擎中需要使用，同样参数在src/github.com/hyperledger/fabric/consensus/pbft/config.yaml
{%highlight bash%}
vpBase:
  extends:
    file: compose-defaults.yml
    service: vp
  environment:
    - CORE_SECURITY_ENABLED=true
    - CORE_PEER_PKI_ECA_PADDR=membersrvc0:7054
    - CORE_PEER_PKI_TCA_PADDR=membersrvc0:7054
    - CORE_PEER_PKI_TLSCA_PADDR=membersrvc0:7054
    - CORE_PEER_PKI_TLS_ROOTCERT_FILE=./bddtests/tlsca.cert
    # TODO: Currently required due to issue reading obbca configuration location
    - CORE_PBFT_GENERAL_N=4
    # The checkpoint interval in sequence numbers
    - CORE_PBFT_GENERAL_K=2

vpBatch:
  extends:
    service: vpBase
  environment:
    - CORE_PEER_VALIDATOR_CONSENSUS_PLUGIN=pbft
    - CORE_PBFT_GENERAL_TIMEOUT_REQUEST=10s
    - CORE_PBFT_GENERAL_MODE=batch
    # TODO: This is used for testing as to assure deployment goes through to block
    - CORE_PBFT_GENERAL_BATCHSIZE=1
{%endhighlight%}

## docker-compose-4-consensus-batch.yml
- 最终docker-compose文件
- 在主机上运行docker-compose -f docker-compose-4-consensus-batch.yml up
- 6060端口是性能分析的端口，笔者认为在初始极端用不到
- 笔者自行增加了vp0到宿主机的端口映射
- 7050 REST API 
- 7051 peer service  如果采用dev模式在宿主机上启动cc需要向该端口注册建立gRPC连接
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
    - CORE_PEER_PROFILE_ENABLED=true
  links:
    - membersrvc0
  ports:
    # 原始端口映射
    # - 7050:6060
    
    # 笔者修改端口映射
    - 6060:6060
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

vp3:
  extends:
    file: docker-compose-4-consensus-base.yml
    service: vpBatch
  environment:
    - CORE_PEER_ID=vp3
    - CORE_PEER_DISCOVERY_ROOTNODE=vp0:7051
    - CORE_SECURITY_ENROLLID=test_vp3
    - CORE_SECURITY_ENROLLSECRET=9LKqKH5peurL
  links:
    - membersrvc0
    - vp0
{%endhighlight%}

## docker环境
- 以net模式运行除了fabric-peer,fabric-membersrvc镜像，还需要一个基础镜像作为构建cc镜像的基础fabric-baseimage
{%highlight bash%}
root@ubuntu:~# docker images
REPOSITORY                      TAG                    IMAGE ID            CREATED             SIZE
hyperledger/fabric-baseimage    latest                 4ac07a26ca7a        5 weeks ago         1.241 GB
hyperledger/fabric-baseimage    x86_64-0.2.2           4ac07a26ca7a        5 weeks ago         1.241 GB
hyperledger/fabric-membersrvc   latest                 b3654d32e4f9        12 weeks ago        1.417 GB
hyperledger/fabric-membersrvc   x86_64-0.6.1-preview   b3654d32e4f9        12 weeks ago        1.417 GB
hyperledger/fabric-peer         latest                 21cb00fb27f4        12 weeks ago        1.424 GB
{%endhighlight%}

- core.yaml中可以看出具体的依赖情况
{%highlight bash%}
chaincode:

    # The id is used by the Chaincode stub to register the executing Chaincode
    # ID with the Peerand is generally supplied through ENV variables
    # the Path form of ID is provided when deploying the chaincode. The name is
    # used for all other requests. The name is really a hashcode
    # returned by the system in response to the deploy transaction. In
    # development mode where user runs the chaincode, the name can be any string
    id:
        path:
        name:

    golang:

        # This is the basis for the Golang Dockerfile.  Additional commands will
        # be appended depedendent upon the chaincode specification.
        # 下方配置作为构建cc镜像的基础，配合代码中的部分构建cc的镜像
        Dockerfile:  |
            from hyperledger/fabric-baseimage
            #from utxo:0.1.0
            COPY src $GOPATH/src
            WORKDIR $GOPATH
{%endhighlight%}

## 部署chaincode
- docker-compose -f docker-compose-4-consensus-batch.yml up 启动4个vp，1个membersrvc
- docker exec -ti 203e938d987b bash 选择一个vp进入其terminal
{%highlight bash%}
root@ubuntu:~# docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
94f2b5e300b4        hyperledger/fabric-peer         "sh -c 'exec $GOPATH/"   46 minutes ago      Up 31 seconds                                                                  deploy_vp2_1
c975eed06fc0        hyperledger/fabric-peer         "sh -c 'exec $GOPATH/"   46 minutes ago      Up 31 seconds                                                                  deploy_vp3_1
85895149db4d        hyperledger/fabric-peer         "sh -c 'exec $GOPATH/"   46 minutes ago      Up 31 seconds                                                                  deploy_vp1_1
203e938d987b        hyperledger/fabric-peer         "sh -c 'exec $GOPATH/"   46 minutes ago      Up 32 seconds       0.0.0.0:6060->6060/tcp, 0.0.0.0:7050-7051->7050-7051/tcp   deploy_vp0_1
257f51d77585        hyperledger/fabric-membersrvc   "membersrvc"             46 minutes ago      Up 32 seconds                                                                  deploy_membersrvc0_1
root@ubuntu:~# docker exec -ti 203e938d987b bash
root@203e938d987b:/opt/gopath/src/github.com/hyperledger/fabric#

#执行chaincode路径
#该路径需要为$GOPATH/src的相对路径
export CCPATH=github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02

#登录
peer network login jim -p 6avZQLwcUe9b
04:04:29.756 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'network'
04:04:29.756 [networkCmd] networkLogin -> INFO 002 CLI client login...
04:04:29.757 [networkCmd] networkLogin -> INFO 003 Local data store for client loginToken: /var/hyperledger/production/client/
04:04:29.757 [networkCmd] networkLogin -> INFO 004 Logging in user 'jim' on CLI interface...
04:04:29.826 [networkCmd] networkLogin -> INFO 005 Storing login token for user 'jim'.
04:04:29.827 [networkCmd] networkLogin -> INFO 006 Login successful for user 'jim'.
04:04:29.827 [main] main -> INFO 007 Exiting.....

#部署
peer chaincode deploy -u jim -l golang -p $CCPATH -c '{"Args": ["init", "a","100", "b", "200"]}'
04:04:35.577 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'chaincode'
04:04:35.577 [chaincodeCmd] getChaincodeSpecification -> INFO 002 Local user 'jim' is already logged in. Retrieving login token.
04:04:38.171 [chaincodeCmd] chaincodeDeploy -> INFO 003 Deploy result: type:GOLANG chaincodeID:<path:"github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02" name:"ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539" > ctorMsg:<args:"init" args:"a" args:"100" args:"b" args:"200" > 
Deploy chaincode: ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539
04:04:38.174 [main] main -> INFO 004 Exiting.....

{%endhighlight%}
- peer chaincode deploy的-p参数必须为$GOPATH/src的相对路径
- 等待一段时间, 查看镜像
{%highlight bash%}
root@ubuntu:~# docker images
REPOSITORY                                                                                                                                 TAG                    IMAGE ID            CREATED             SIZE
dev-vp0-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   latest                 b65b4c49674b        41 seconds ago      1.276 GB
dev-vp2-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   latest                 058823c6e614        42 seconds ago      1.276 GB
dev-vp1-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   latest                 809f1274b811        42 seconds ago      1.276 GB
dev-vp3-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   latest                 f908b0c50bea        42 seconds ago      1.276 GB
hyperledger/fabric-baseimage                                                                                                               latest                 4ac07a26ca7a        5 weeks ago         1.241 GB
hyperledger/fabric-baseimage                                                                                                               x86_64-0.2.2           4ac07a26ca7a        5 weeks ago         1.241 GB
hyperledger/fabric-membersrvc                                                                                                              latest                 b3654d32e4f9        12 weeks ago        1.417 GB
hyperledger/fabric-membersrvc                                                                                                              x86_64-0.6.1-preview   b3654d32e4f9        12 weeks ago        1.417 GB
hyperledger/fabric-peer                                                                                                                    latest                 21cb00fb27f4        12 weeks ago        1.424 GB
{%endhighlight%}

- 容器运行状态
{%highlight bash%}
root@ubuntu:~# docker ps 
CONTAINER ID        IMAGE                                                                                                                                      COMMAND                  CREATED              STATUS              PORTS                                                      NAMES
35a35d05d0eb        dev-vp0-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   "/opt/gopath/bin/ee5b"   16 seconds ago       Up 15 seconds                                                                  dev-vp0-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539
92100fe587b7        dev-vp1-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   "/opt/gopath/bin/ee5b"   16 seconds ago       Up 15 seconds                                                                  dev-vp1-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539
33618a7c5773        dev-vp2-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   "/opt/gopath/bin/ee5b"   16 seconds ago       Up 15 seconds                                                                  dev-vp2-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539
7a2af08a0881        dev-vp3-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539   "/opt/gopath/bin/ee5b"   16 seconds ago       Up 16 seconds                                                                  dev-vp3-ee5b24a1f17c356dd5f6e37307922e39ddba12e5d2e203ed93401d7d05eb0dd194fb9070549c5dc31eb63f4e654dbd5a1d86cbb30c48e3ab1812590cd0f78539
8bfabdb9bb13        hyperledger/fabric-peer                                                                                                                    "sh -c 'exec $GOPATH/"   About a minute ago   Up About a minute                                                              deploy_vp2_1
d1aea97c21eb        hyperledger/fabric-peer                                                                                                                    "sh -c 'exec $GOPATH/"   About a minute ago   Up About a minute                                                              deploy_vp3_1
b9c81f8e89a9        hyperledger/fabric-peer                                                                                                                    "sh -c 'exec $GOPATH/"   About a minute ago   Up About a minute                                                              deploy_vp1_1
4076525d77a2        hyperledger/fabric-peer                                                                                                                    "sh -c 'exec $GOPATH/"   About a minute ago   Up About a minute   0.0.0.0:6060->6060/tcp, 0.0.0.0:7050-7051->7050-7051/tcp   deploy_vp0_1
e4893b9fc5da        hyperledger/fabric-membersrvc                                                                                                              "membersrvc"             About a minute ago   Up About a minute                                                              deploy_membersrvc0_1
{%endhighlight%}

## 其它事项
- dev模式-n参数可用，后续调用也用部署时-n指定参数，net模式下-n参数不可用chaincodeid.name会被cc代码的哈希值填充，后续调用也需要使用。
- net模式部署完毕后cc的哈希值会反馈
- 删除所有运行容器、已经dev-vp开头的镜像命令。**删除动作，谨慎操作！**
{%highlight bash%}
#删除所有运行容器
docker ps -a | awk 'NR >1 {print $1}' |xargs -I {} docker rm -f {}

#删除所有dev-vp开头镜像
docker images | awk 'NR >1 {print $1}' | awk '/dev-vp*/'|xargs -I {} docker rmi {}
{%endhighlight%}