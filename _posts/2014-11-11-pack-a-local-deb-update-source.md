---
layout: post
title: "Ubuntu打包创建本地源"
description: ""
category: "Linux"
tags: [Linux]
---
{% include JB/setup %}
<p>
安装打包软件
</p>
{% highlight bash %}
root@ubuntu:~# apt-get install dpkg-dev
Reading package lists... Done
Building dependency tree       
Reading state information... Done
dpkg-dev is already the newest version.
dpkg-dev set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 102 not upgraded.
{% endhighlight %}
<p>
打包软件
</p>
{% highlight bash %}
root@ubuntu:~# dpkg-scanpackages radxa /dev/null | gzip >radxa/Packages.gz
root@ubuntu:~# ls
radxa
{% endhighlight %}
<p>
如上radxa为当前目录下哪个文件下存放的deb包,|之后是将scan的结果打包存放到radxa目录下，保存为文件Packages.gz。
</p>
<p>
修改更新源文件
</p>
{% highlight bash %}
vim /etc/apt/sources.list
#内如如下
deb file:/// radxa/
{% endhighlight %}
<p>
file://	本地文件系统<br>
/	根目录<br>
radxa/	根目录下的radxa目录<br>
如果之前的打包文件路径不便，文件修改可如下：<br>
deb file:/// root/radxa/
</p>
