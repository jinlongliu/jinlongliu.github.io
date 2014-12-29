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
<p>
 查看bonding配置信息：
</p>
{% highlight bash %}
#cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)
  
Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 0
Up Delay (ms): 0
Down Delay (ms): 0
  
Slave Interface: p3p1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:1b:a1:ab:44:f8
Slave queue ID: 0
  
Slave Interface: p3p2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:1b:a1:ab:44:f9
Slave queue ID: 0
{% endhighlight %}
<p>
动态链路汇聚
</p>
{% highlight bash %}
auto p3p1
iface p3p1 inet manual
bond-master bond0
  
auto p3p2
iface p3p2 inet manual
bond-master bond0
  
auto bond0
iface bond0 inet static
address 192.168.1.202
netmask 255.255.255.0
gateway 192.168.1.1
bond-mode 4
bond-miimon 100
bond-lacp-rate 1
bond-slaves p3p1 p3p2
up /sbin/ifenslave bond0 p3p1 p3p2
{% endhighlight %}
<p>
#modprobe bonding<br/>
#ifenslave bond0 p3p1 p3p2
</p>
{% highlight bash %}
root@controller:~# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)
  
Bonding Mode: IEEE 802.3ad Dynamic link aggregation
Transmit Hash Policy: layer2 (0)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0
  
802.3ad info
LACP rate: fast
Min links: 0
Aggregator selection policy (ad_select): stable
Active Aggregator Info:
        Aggregator ID: 1
        Number of ports: 2
        Actor Key: 17
        Partner Key: 7
        Partner Mac Address: 5c:dd:70:83:56:2d
  
Slave Interface: p3p1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:1b:a1:ab:44:f8
Aggregator ID: 1
Slave queue ID: 0
  
Slave Interface: p3p2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:1b:a1:ab:44:f9
Aggregator ID: 1
Slave queue ID: 0
{% endhighlight %}
