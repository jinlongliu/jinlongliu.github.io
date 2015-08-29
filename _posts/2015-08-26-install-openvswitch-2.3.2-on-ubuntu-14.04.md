---
layout: post
title: "Ubuntu 14.04安装Openvswitch 2.3.2"
description: ""
category: "SDN"
tags: ["SDN", "ONOS"]
---
{% include JB/setup %}
<p>
笔者从官网下载了2.3.2的源码，然后在基础的Ubuntu 14.04的系统上进行编译生成了deb包。<br/>
之后笔者利用生成的deb包在干净的系统上重新安装。
</p>
<p>
相应的deb包可以从http://yun.baidu.com/share/link?shareid=1517923564&uk=1818218422 下载。
</p>

{% highlight bash %}
#apt-get install dkms
#dpkg -i openvswitch-datapath-dkms_2.3.2-1_all.deb
#dpkg -i openvswitch-common_2.3.2-1_amd64.deb openvswitch-switch_2.3.2-1_amd64.deb
{% endhighlight %}
<p>
#apt-get install dkms<br/>
该命令可以通过下载http://yun.baidu.com/share/link?shareid=1517923564&uk=1818218422#path=%252F01Software%252F00-OpenvSwitch<br/>
dmks_deb.tar.gz,解压，进入目录，然后执行#dpkg -i *.deb即可安装DKMS
</p>
<p>
查看是否安装成功
</p>
{% highlight bash %}
root@ubuntu:~# lsmod |grep open
openvswitch            80010  0 
gre                    13808  1 openvswitch
vxlan                  37619  1 openvswitch
libcrc32c              12644  1 openvswitch
  
  
root@ubuntu:~# service openvswitch-switch status
ovsdb-server is running with pid 10000
ovs-vswitchd is running with pid 10010
{% endhighlight %}

