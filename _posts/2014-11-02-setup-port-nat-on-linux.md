---
layout: post
title: "Linux端口NAT设置"
description: ""
category: "运维" 
tags: [运维]
---
{% include JB/setup %}
<h3>端口设置</h3>
<p>
Linux端口设置
</p>
{% highlight ruby %}
root@Linux#iptables -t nat -A PREROUTING -d 192.168.99.2 -p tcp --dport 5908 -j DNAT --to 192.168.2.11:5902
root@Linux#iptables -t nat -A POSTROUTING -d 192.168.2.11 -p tcp --dport 5902 -j SNAT --to 192.168.2.5
root@Linux#iptables -t nat -A PREROUTING -d 192.168.99.2 -p tcp --dport 5909 -j DNAT --to 192.168.2.2:5902
root@Linux#iptables -t nat -A POSTROUTING -d 192.168.2.2 -p tcp --dport 5902 -j SNAT --to 192.168.2.5
root@iptables -t nat -L
{% endhighlight %}
<p>
注意：前提条件开启IP转发功能。
</p>
{% highlight ruby %}
root@ubuntu#echo 1 > /proc/sys/net/ipv4/ip_forward
{% endhighlight %}
