---
layout: post
title: "Centos7系统限制修改"
description: ""
category: Linux 
tags: [Linux]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 默认设置
{% highlight bash %}
#vim /etc/systemd/system.conf
DefaultLimitCORE=infinity
DefaultLimitNOFILE=10000
DefaultLimitNPROC=10000


#vim /etc/security/limits.d/20-nproc.conf
*          soft    nproc     4096
root       soft    nproc     unlimited
{% endhighlight %}

{% include JB/setup %}


