---
layout: post
title: "Hyperledger Fabric-sdk-node e2e测试"
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
- npm,node在之前的部署中已经安装到客户机内


### 源码映射
{% highlight bash %}
#本次实验依旧是源码在windows系统宿主机映射到ubuntu 16.04的客户机内
#编辑Vagrantfile

# Fabric-sdk-node路径映射
config.vm.synced_folder "../../../../../../fabric-sdk-node",
  "/opt/gopath/src/github.com/hyperledger/fabric/fabric-sdk-node"
  
{% endhighlight %}


### 测试前准备
{% highlight bash %}
#登录客户机

Liu@DESKTOP-AFIQ8P3 /cygdrive/d/01workspace/hyperledger-v1.0.2/src/github.com/hyperledger/fabric/devenv
$ vagrant.exe ssh
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-93-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

2 packages can be updated.
0 updates are security updates.


Last login: Sat Sep  9 04:55:54 2017 from 10.0.2.2
ubuntu@hyperledger-devenv::/opt/gopath/src/github.com/hyperledger/fabric$ cd fabric-sdk-node/
ubuntu@hyperledger-devenv::/opt/gopath/src/github.com/hyperledger/fabric/fabric-sdk-node$ npm install


#npm install 会失败
#root权限安装
npm install -g gulp

EPROTO: protocol error, symlink '../acorn/bin/acorn' -> '/opt/gopath/src/github.com/hyperledger/fabric/fabric-sdk-node/node_modules/.bin/acorn'
#解决方案：
$npm install --no-bin-links


Error: Cannot find module '/opt/gopath/src/github.com/hyperledger/fabric/fabric-sdk-node/node_modules/grpc/src/node/extension_binary/node-v48-linux-x64/grpc_node.node'
#解决方案：
#root权限安装
npm install -g node-pre-gyp  
$ npm install grpc --no-bin-links

#在安装grpc会调用node-pre-gyp,node-pre-gyp install --fallback-to-build --library=static_library


#fabric-ca-client缺失api.js
gulp ca


#准备fabric网络，fabric-sdk-node/test/fixtures
docker-compose up
#启动2个ca，2个peer,1个order，1个couchdb
#该会话保持

{% endhighlight %}


### 执行测试用例
{% highlight bash %}
#vagrant ssh, fabric-sdk-node目录
node test/integration/e2e.js

#e2e.js内容如下，主要是创建channel，安装、初始化、调用、升级chaincode
require('./e2e/create-channel.js');
require('./e2e/join-channel.js');
require('./e2e/install-chaincode-fail.js');
require('./e2e/install-chaincode.js');
require('./e2e/instantiate-chaincode.js');
require('./e2e/invoke-transaction.js');
require('./e2e/query.js');
require('./e2e/upgrade.js');


#成功结束输出
***** TransientMap Support in Proposals *****


ok 85 Successfully loaded member from persistence
ok 86 Checking the result has the transientMap value returned by the chaincode
ok 87 Checking the result has the transientMap value returned by the chaincode
ok 88 Successfully verified transient map values

1..88
# tests 88
# pass  88

# ok

{% endhighlight %}

{% include JB/setup %}


