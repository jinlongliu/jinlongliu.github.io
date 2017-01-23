---
layout: post
title: "Hyperledger Fabric中内部广播器"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### obcBatch内部广播器
- 共识引擎在处理时需要广播共识消息调用如图
- 最终实际调用的仍然是节点直接的Chat
<img src="/upload/2017/innerBroadcast.png" width="1024px">


