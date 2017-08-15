---
layout: post
title: "CentOS7 安装Docker 17.06"
description: ""
category: Docker 
tags: [Docker]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### 安装依赖软件
{% highlight bash %}
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
{% endhighlight %}

### 修改软件源
{% highlight bash %}
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
{% endhighlight %}

### 禁用部分源
{% highlight bash %}
$ sudo yum-config-manager --enable docker-ce-edge

$ sudo yum-config-manager --enable docker-ce-testing
{% endhighlight %}

### 修改主机名
{% highlight bash %}
$ hostnamectl --static set-hostname c1

$ hostnamectl status
{% endhighlight %}

### 开放防火墙端口
{% highlight bash %}
$firewall-cmd --zone=public --add-port=2377/tcp --permanent && \
    firewall-cmd --zone=public --add-port=7946/tcp --permanent && \
    firewall-cmd --zone=public --add-port=7946/udp --permanent && \
    firewall-cmd --zone=public --add-port=4789/tcp --permanent && \
    firewall-cmd --zone=public --add-port=4789/udp --permanent && \
    firewall-cmd --reload 

$firewall-cmd --list-ports
4789/udp 7946/tcp 7946/udp 2377/tcp 4789/tcp
{% endhighlight %}

### 安装
{% highlight bash %}
#刷新软件索引
$ sudo yum makecache fast
$ sudo yum install docker-ce

#检查安装版本
$ yum list docker-ce.x86_64  --showduplicates | sort -r

#启动
$ sudo systemctl start docker

#开机启动
$systemctl enable docker
$systemctl list-unit-files| grep docker
{% endhighlight %}

### 卸载
{% highlight bash %}
$ sudo yum remove docker-ce
$ sudo rm -rf /var/lib/docker
{% endhighlight %}

### 加速器
{% highlight bash %}
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://0503c5a1.m.daocloud.io

#执行效果，新增文件/etc/docker/daemon.json 内容如下
{"registry-mirrors": ["http://0503c5a1.m.daocloud.io"]}

#阿里加速器
{"registry-mirrors": ["https://axhzwr24.mirror.aliyuncs.com"]}
$systemctl daemon-reload
$systemctl restart docker
{% endhighlight %}


{% include JB/setup %}


