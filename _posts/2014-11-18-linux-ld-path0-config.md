---
layout: post
title: "Linux配置ld.so加载路径"
description: ""
category: "Linux" 
tags: [Linux, 运维]
---
{% include JB/setup %}
{% highlight bash %}
root@ubuntu:~# vim /etc/ld.so.conf
  
include /etc/ld.so.conf.d/*.conf
/usr/local/lib
  
#配置加载
root@ubuntu:~# ldconfig -v
{% endhighlight %}
