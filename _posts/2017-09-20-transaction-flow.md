---
layout: post
title: "Hyperledger Fabric交易流程"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 交易流程
1. 客户A通过SDK对交易提案进行签名、将交易提案包装（gRPC），发送给背书节点(有可能是多个背书节点)；
2. 背书节点验证签名并模拟执行【不更新账本】，用模拟结果构建提案响应，同时对响应进行背书签名，发送回给SDK；
3. 客户端SDK对提案响应进行审查，如果只是查询则不提交交易，检验背书担保策略是否满足；
4. 应用将交易提案和提案响应封装为一个交易消息广播给排序服务，排序服务接收所有信道的交易，按信道和时间进行构建交易区块，同时交付到所有对等节点；
5. 对等节点从排序服务获取区块,区块里面的交易被标记为有效和无效；
6. 对等节点将区块追加到信道的链上，对有效交易，写入集合被执行更新到当前数据库，同时发送一个事件，告知应用客户端交易调用已经不可更改的应用到区块链，同时告知交易是否有效；

![proposal](/upload/2017/transaction-flow/1.png)
![endorsing](/upload/2017/transaction-flow/2.png)
![verify-response](/upload/2017/transaction-flow/3.png)
![boradcast](/upload/2017/transaction-flow/4.png)
![to-peers](/upload/2017/transaction-flow/5.png)
![commit](/upload/2017/transaction-flow/6.png)

{% include JB/setup %}


