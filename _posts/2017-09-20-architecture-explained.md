---
layout: post
title: "Hyperledger Fabric架构解析"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 架构优势
- Chaincode trust flexibility：智能合约信任和排序信任相分离，排序服务由节点集合提供，每个智能合约的背书人可能不同；
- Scalability：由于背书节点和排序节点垂直相交（orthogonal ），扩展性更好；当智能合约指定互斥的背书人，形成基于背书人的分区，从而使得
智能合约担保的并发执行；
- Confidentiality：对于内容和状态更新有需要保密的智能合约部署更容易；
- Consensus modularity：模块化可插拔的共识实现；

1. 智能合约信用灵活
2. 扩展性
3. 机密性
4. 模块化共识






{% include JB/setup %}


