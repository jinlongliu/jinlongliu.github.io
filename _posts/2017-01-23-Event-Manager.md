---
layout: post
title: "Hyperledger Fabric中内部事件管理"
description: ""
category: Blockchain 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}

### Fabric多处用了事件管理
- coordinatorImpl
{% highlight bash %}
// 实现了Executor接口
type coordinatorImpl struct {
	manager         events.Manager              // Maintains event thread and sends events to the coordinator
	rawExecutor     PartialStack                // Does the real interaction with the ledger
	consumer        consensus.ExecutionConsumer // The consumer of this coordinator which receives the callbacks
	stc             statetransfer.Coordinator   // State transfer instance
	batchInProgress bool                        // Are we mid execution batch
	skipInProgress  bool                        // Are we mid state transfer
}
{% endhighlight %}

- obcBatch
{% highlight bash %}
type obcBatch struct {
	obcGeneric  // 基础公共
	externalEventReceiver
	pbft        *pbftCore
	broadcaster *broadcaster

	batchSize        int
	batchStore       []*Request
	batchTimer       events.Timer
	batchTimerActive bool
	batchTimeout     time.Duration

	manager events.Manager // TODO, remove eventually, the event manager

	incomingChan chan *batchMessage // Queues messages for processing by main thread
	idleChan     chan struct{}      // Idle channel, to be removed

	// 存储未解决，行将发生的请求
	reqStore *requestStore // Holds the outstanding and pending requests

	deduplicator *deduplicator

	persistForward
}
{% endhighlight %}

### 事件管理交互图
![PBFT](/upload/2017/eventManager.png)

