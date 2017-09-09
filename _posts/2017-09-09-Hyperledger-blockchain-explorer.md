---
layout: post
title: "Hyperledger blockchain explorer搭建"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 前置说明
- 依赖于之前fabric-sdk-node e2e测试

### 源码映射
{% highlight bash %}
#本次实验依旧是源码在windows系统宿主机映射到ubuntu 16.04的客户机内
#编辑Vagrantfile

# blockchain-explorer路径映射
config.vm.synced_folder "../../../../../../blockchain-explorer",
  "/opt/gopath/src/github.com/hyperledger/fabric/blockchain-explorer"
  
# 端口映射
config.vm.network :forwarded_port, guest: 8080, host: 8080, id: "blockchain-explorer", host_ip: "localhost", auto_correct: true # CouchDB service
  
#登录客户机启动fabric-sdk-node的e2e测试环境，同时运行完所有e2e的测试用例
  
{% endhighlight %}


### 安装blockchain-explorer
{% highlight bash %}
#fabric/blockchain-explorer目录,执行安装依赖
npm install

#可能报错
npm ERR! EPROTO: protocol error, symlink '../mkdirp/bin/cmd.js' -> '/opt/gopath/src/github.com/hyperledger/fabric/blockchain-explorer/fabric-explorer/node_modules/grpc/node_modules/.bin/mkdirp'

#解决方案：
sudo npm install -g grpc
$npm install --no-bin-links

{% endhighlight %}


### 编辑配置文件
{% highlight bash %}
#fabric-sdk-node MSP配置文件fabric-sdk-node/test/fixtures/channel/crypto-config

#blockchain-explorer/fabric-explorer/config.json 开启TLS
"enableTls":true

#相关配置
#blockchain-explorer/fabric-explorer/app/network-config-tls.json
#原来是2个组织，每个组织2个节点，e2e环境为2个组织，每个组织1个节点修改如下


{
	"network-config": {
		"orderer": [{
			"url": "grpcs://127.0.0.1:7050",
			"server-hostname": "orderer0.example.com",
			"tls_cacerts": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tlscacerts/example.com-cert.pem"
		}],
		"org1": {
			"name": "peerOrg1",
			"mspid": "Org1MSP",
			"ca": "https://127.0.0.1:7054",
			"peer1": {
				"requests": "grpcs://127.0.0.1:7051",
				"events": "grpcs://127.0.0.1:7053",
				"server-hostname": "peer0.org1.example.com",
				"tls_cacerts": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tlscacerts/org1.example.com-cert.pem"
			},
			"admin": {
				"key": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/keystore",
				"cert": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/signcerts"
			}
		},
		"org2": {
			"name": "peerOrg2",
			"mspid": "Org2MSP",
			"ca": "https://127.0.0.1:8054",
			"peer1": {
				"requests": "grpcs://127.0.0.1:9051",
				"events": "grpcs://127.0.0.1:9053",
				"server-hostname": "peer0.org2.example.com",
				"tls_cacerts": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tlscacerts/org2.example.com-cert.pem"
			},
			"admin": {
				"key": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/keystore",
				"cert": "../../fabric-sdk-node/test/fixtures/channel/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/signcerts"
			}
		}
	}
}

#启动blockchain-explorer脚本start.sh
node main.js >log.log 2>&1 &

#调试阶段可以直接node main.js

#访问宿主机8080端口
{% endhighlight %}

![](/upload/2017/blockchain-explorer-snapshot.png)

{% include JB/setup %}


