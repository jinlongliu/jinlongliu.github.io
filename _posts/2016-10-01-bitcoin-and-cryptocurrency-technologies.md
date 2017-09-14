---
layout: post
title: "《区块链技术驱动金融》读书笔记"
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

### 哈希函数(白话特性)
- 碰撞阻力：信息摘要（message digest），不同的消息不可能生成相同的哈希值
- 隐秘性：承诺，不可以反逆推
- 谜题友好：输入集猜测(搜索谜题)没有捷径

### 数字签名
- (sk, pk) := generateKeys(keysize)
- sig := sign(sk, message)
- isValid := verify(pk, message, sig)
- 有效签名 verify(pk, message, sign(sk, message)) == true

### 椭圆曲线签名算法(ECDSA)
- 个人密钥： 256位
- 公钥（未压缩）: 512位
- 公钥（压缩）：257位
- 待签名信息： 256位
- 签名： 512位
- 比特币使用该签名算法

{% include JB/setup %}


