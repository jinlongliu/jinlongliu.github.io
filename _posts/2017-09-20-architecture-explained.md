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

### 系统架构
区块链(blockchain)是包含许多互相通信节点的分布式系统。区块链运行叫做智能合约(chaincode)的应用,保持状态和账本数据，执行交易。由于交易是
通过调用智能合约操作，所以智能合约是核心元素。交易必须被担保，只有被担保的交易才可以被提交其发生作用。存在特定的智能合约用于管理函数和参数，
统称为系统智能合约。

##### 交易
- 部署交易(Deploy transactions)创建新智能合约，附带一个程序作为参数。当部署交易成功执行，智能合约安装到区块链上。
- 调用交易(Invoke transactions)执行针对之前部署的智能合约的操作。一个调用交易指向一个智能合约以及期提供的函数。当成功时，智能合约执行特定
函数，函数调用修改一致性状态，返回输出。

- 部署交易实质也是调用交易，只是调用的是系统智能合约而已。
- 只读查询交易用于跨链交易。

### 区块链数据结构
##### 状态
区块链的状态模型是版本控制的key/value store(KVS)。条目由智能合约通过put和get KVS操作维护。状态是持久化的，状态操作是记录的。版本控制的
KVS是适配状态模型(state model)，可以用KVS实现，也可以用RDBMS或者其它解决方案实现。

state(s) 模型化为一个映射元素 K -> (V x N)     x为关联符号
- K a set of keys
- V a set of values
- N 版本号的无限排序集合， next: N -> N

V和N都包含特定元素\bot,\bot 为N的最小元素。初始化键都映射到(\bot, \bot), s(k)=(v,ver),v=s(k).value,ver=s(k).version

- put(k,v) k放入K，v放入V,状态s变更为s', s'(k)=(v,next(s(k).version))   s'(k')=s(k')    k'!=k
- get(k) 返回 s(k)

状态有节点维护，非订单节点和客户端

##### 账本
账本提供一个可验证的历史关于所有成功的状态修改和不成功状态修改企图。

账本由订单服务构建，交易块的有序哈希链。哈希链集所有有序块于一个账本，每个块包含完整的有序交易。

账本在所有对等节点(peer)留存，在部分排序节点可选留存。在排序上下文称OrderLedger，在对等节点上下文称PeerLedger。PeerLedger不同于OrderLedger本地维护
位掩码区分有效交易和无效交易。

账本使得对等节点可以回放所有交易历史，从而重建状态。

### 节点
节点(nodes)是区块链的通信实体。node是一个逻辑称谓，因此不同类型的node可以在同一个物理服务器。
- client or submmitting-client 提交真实交易调用到背书人，广播交易提案到排序服务(ordering service)
- peer 提交交易的node，维护状态和账本副本，可以拥有特定的背书角色
- ordering-service-node or orderer 运行通信服务实现分发保证，比如原子的或者完全有序的广播。

1. 客户端    终端用户的行为代表，和区块链通信交互必须连接到对等节点。
2. 对等节点    从排序服务接收以块形式的有序的状态更新，维护账本和状态。可以额外承担背书人的角色。每一个智能合约可以指定担保策略。担保策略
指定了担保节点集合。策略定义了一个有效担保交易的必要和可选条件。安装新智能合约的部署交易所用的部署担保策略是由系统智能合约指定的。
3. 排序服务节点 订货人    订货人组成了排序服务，提供分发保证的通信网络。 排序服务可以有不同的实现方式：从中心化服务过渡到容错性的分布式网络。

排序服务在client和peer之间提供了共享的通信通道，提供包含交易的信息广播服务。client连上信道广播消息到信道上的所有peer对等节点。
信道提供了所有消息的原子交付。消息是可靠，完全有序分发通信。换句话说，<font color="red">信道以完全相同的逻辑顺序将完全相同的消息输出到所有连接节点。</font>
这个原子通信保证叫做total-order broadcast, atomic broadcast, 在分布式系统上下文叫consensus(共识)。通信消息是包含在区块链状态里的候选交易。

分区(排序服务通道)排序服务支持多信道类似消息系统的主题订阅和发布机制。client可以连接到给定信道，发送和接收消息。client连接到某个信道，不知道
其它信道存在。但是client可以连接到多个信道。

**ordering service api**:peer通过对应接口连接到ordering服务提供的信道，提供两个基本操作，通常是异步事件
- broadcast(blob) client调用其广播消息blob,在BFT（拜占庭）上下文系统中也叫request(blob)
- deliver(seqno, prevhash, blob) ，ordering服务在对等节点peer上调用来交付消息。换句话说，是来自于排序服务的一个输出事件。
在订阅发布系统中也叫notify()，在BFT系统中也叫commit()

**账本和块信息** 账本包含排序服务的所有数据，概括而言就是一系列的deliver(seqno, prevhash, blob)事件。实质是上通过prevhash构成了一个哈希链。

大部分情况，因为效率原因，排序服务打包二进制组成块封装到一个deliver事件，而不是输出单个交易的二进制。块内二进制的数量可以动态选择。

**排序服务特性**
- Agreement 
- Hashchain integrity
- No skipping
- No creation
- No duplication
- Liveness

### 交易担保基本流程
![tx-endorsement](/upload/2017/tx-endorsement.png)

tx=<clientID,chaincodeID,txPayload,timestamp,clientSig>

txPayload根据交易类型的不同而不同
- Invoke transaction: txPayload = <operation, metadata>
    - operation
    - metadata
 
- Deploy transaction: txPayload = <source, metadata, policies>
    - source
    - metadata
    - policies

tid=HASH(tx),client存储tid在内存中，等待担保节点回应。

### 担保策略
### 验证账本

[官方链接](http://hyperledger-fabric.readthedocs.io/en/latest/arch-deep-dive.html)

{% include JB/setup %}


