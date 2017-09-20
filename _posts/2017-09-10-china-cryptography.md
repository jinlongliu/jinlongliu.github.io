---
layout: post
title: "国密算法"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 国密算法说明
- SM1 对称加密。其加密强度与AES相当。该算法不公开，调用该算法时，需要通过加密芯片的接口进行调用。
- SM2 非对称加密，基于ECC。该算法已公开。由于该算法基于ECC，故其签名速度与秘钥生成速度都快于RSA。
ECC 256位（SM2采用的就是ECC 256位的一种）安全强度比RSA 2048位高，但运算速度快于RSA。
- SM3 消息摘要。可以用MD5作为对比理解。该算法已公开。校验结果为256位。
- SM4 无线局域网标准的分组数据算法。对称加密，密钥长度和分组长度均为128位。【国际DES算法】
- SM1、SM4加解密的分组大小为128bit，故对消息进行加解密时，若消息长度过长，需要进行分组，要消息长度不足，则要进行填充。

### 参考链接
[国密算法那点事儿](http://www.360doc.com/content/15/1215/14/16410669_520585373.shtml)


{% include JB/setup %}

