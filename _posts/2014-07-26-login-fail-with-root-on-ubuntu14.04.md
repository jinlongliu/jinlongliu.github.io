---
layout: post
title: "Ubuntu 14.04 SSH无法登录"
description: ""
category: 
tags: []
---
{% include JB/setup %}
<p>
新安装完Ubuntu 14.04 server，在机器上配置了root用户的密码，但是SSH登录失败。报错信息“Password authentication failed”。
解决办法，修改SSH配置文件
<img src="/upload/2014/7/25/login_fail.png"/>
</p>
{% highlight ruby %}
root@ubuntu#vim /etc/ssh/sshd_config
{% endhighlight %}
{% highlight ruby %}
# Authentication:
LoginGraceTime 120
PermitRootLogin without-password
StrictModes yes
{% endhighlight %}
<p>
将PermitRootLogin without-password 修改为 yes，重启ssh服务。
</p>
{% highlight ruby %}
PermitRootLogin yes
{% endhighlight %}
{% highlight ruby %}
service ssh restart
{% endhighlight %}
<p>
再次登录即可
</p>

