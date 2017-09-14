---
layout: post
title: "区块链技术驱动金融读书笔记"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Bitcoin, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 哈希函数
- 碰撞阻力：无法找到两个值，x和y，x ≠ y，而H(x) = H(y),则称哈希函数H具有碰撞阻力
- 隐秘性：如果当其输入r选自一个高阶最小熵（high min-entroy）的概率分布，在给定H(r||x)条件下确定x是不可行的
- 谜题友好：如果对于任意n位输出值y，假定k选自高阶最小熵分布，如果无法找到一个可行方法，在比2^n小的时间内找到x，保证H(k||x) = y 成立，
那么我们称哈希函数H为谜题友好
- ![sha-256](/upload/2016/sha-256.png)


{% include JB/setup %}


