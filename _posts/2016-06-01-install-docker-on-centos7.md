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

[root@king ~]# docker -v
Docker version 1.12.1, build 23cf638
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

### 修改启动参数
{% highlight bash %}
#针对CentOS/Red Hat Enterprise Linux
mkdir /etc/systemd/system/docker.service.d

vim /etc/systemd/system/docker.service.d/docker.conf

#必须指定一个空行，后接一个新配置行
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -D --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.59.3:2376

systemctl daemon-reload
systemctl restart docker
{% endhighlight %}

{% include JB/setup %}
