---
layout: post
title: "链路聚合"
description: ""
category: "Linux" 
tags: []
---
{% include JB/setup %}
<p>
网卡配置如下：
</p>
{% highlight bash %}
auto p3p1
iface p3p1 inet manual
  
auto p3p2
iface p3p2 inet manual
  
auto bond0
iface bond0 inet static
address 192.168.1.202
netmask 255.255.255.0
bond-mode 4
bond-miimon 100
up /sbin/ifenslave bond0 p3p1
up /sbin/ifenslave bond0 p3p2
{% endhighlight %}
<p>
启动，关闭命令
</p>
{% highlight bash %}
rmmod bonding
modprobe bonding
{% endhighlight %}
<p>
安装软件
</p>
{% highlight bash %}
apt-get install ifenslave-2.6
{% endhighlight %}
<p>
交换机设置链路聚合时，先将端口设置为trunk类型，然后设置为access类型，然后创建链路聚合。<br/>
之后在VLAN设置中将创建的链路聚合端口选中到某个VLAN即可。
</p>
