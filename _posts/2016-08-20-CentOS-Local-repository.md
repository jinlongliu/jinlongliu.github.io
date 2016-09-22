---
layout: post
title: "CentOS本地源"
description: ""
category: Linux
tags: [Linux, CentOS]
---
{% include JB/setup %}
## 配置CentOS本地源
{%highlight bash%}
#缓存软件
$vim /etc/yum.conf
keepcache=0
#修改为
keepcache=1

#安装软件，将/var/cache/yum/x86_64/7/下缓存下来的rpm统一到一个目录
$find ./ -name *.rpm |xargs -I {} mv {} ~/software

#安装源制作软件
$yum install -y createrepo

#生成源索引,会在目录生成repodata目录
$createrepo ~/software

#在新节点配置本地源
$vim /etc/yum.repos.d/local.repo
[Local]
name = ReposName
baseurl = file:///root/software
gpgcheck = 0
enable = 1

#新节点安装本地源中软件
$yum makecache
$yum install -y ...
{%endhighlight%}
