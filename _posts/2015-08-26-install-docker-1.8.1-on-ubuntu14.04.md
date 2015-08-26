---
layout: post
title: "Ubuntu 14.04 LTS amd64 安装docker 1.8.1"
description: "Docker"
category: "Docker"
tags: ["Docker"]
---
{% include JB/setup %}
<p>
软件包下载:<br/>
http://yun.baidu.com/share/link?shareid=1517923564&uk=1818218422#path=%252F01Software%252F01-Docker%252Fdocker_1.8.1_deb
</p>
{% highlight bash %}
#dpkg -i *.deb
{% endhighlight %}
<p>
查看版本信息
</p>
{% highlight bash %}
root@ubuntu:~# docker -v
Docker version 1.8.1, build d12ea79
{% endhighlight %}
