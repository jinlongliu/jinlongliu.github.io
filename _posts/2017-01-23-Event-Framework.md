---
layout: post
title: "Hyperledger Fabric中事件框架"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode，EventFramework]
---
{% include JB/setup %}

### 事件框架V0.6.1
- 事件框架提供了生产和消费预定义或自定义的事件的能力。它有3个基础组件：
- 事件流
- 事件适配器
- 事件结构

### 事件框架代码目录
  - src/github.com/hyperledger/fabric/events
  - 事件流是用来发送和接收事件的gRPC通道
  - 事件生产者暴露函数`Send(e *pb.Event)`来发送事件
  - 事件消费者允许外部应用监听事件

### 事件适配器 事件适配器封装了三种流交互的切面：
  - 返回所有感兴趣的事件列表的接口
  - 当事件消费者框架接受到事件后调用的接口
  - 当事件总线终止时，事件消费者框架会调用的接口

### 数据结构
![PBFT](/upload/2017/eventFrameworkDataStructure.png)

### 交互图
![PBFT](/upload/2017/eventFrameworkInteraction.png)



