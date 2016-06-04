---
layout: post
title: "CentOS7 安装Docker"
description: ""
category: Docker
tags: [Docker]
---
### 配置软件源
{% highlight bash %}
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
{% endhighlight %}

### 安装Docker
{% highlight bash %}
yum install docker-engine
service docker start
{% endhighlight %}

### 安装Docker compose
{% highlight bash %}
curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
cd /usr/bin
ln -s /usr/local/bin/docker-compose docker-compose
chmod +x docker-compose
{% endhighlight %}

### 验证
{% highlight bash %}
[root@controller common]# docker-compose --version
docker-compose version 1.7.1, build 0a9ab35
{% endhighlight %}
{% include JB/setup %}
