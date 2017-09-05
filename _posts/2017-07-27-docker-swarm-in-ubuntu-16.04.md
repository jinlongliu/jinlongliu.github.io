---
layout: post
title: "Ubuntu 16.04 安装docker 17.06.0-ce运行swarm模式"
description: ""
category: Docker 
tags: [Docker]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 新增仓库
{% highlight bash %}
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
{% endhighlight %}

### 查看仓库版本
{% highlight bash %}
root@ilog1:/var/cache/apt# apt-cache madison docker-ce
 docker-ce | 17.06.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.2~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 
 apt-get install docker-ce
{% endhighlight %}

### 安装指定版本
{% highlight bash %}
apt-get install docker-ce=<VERSION>
{% endhighlight %}

### 开机启动
{% highlight bash %}
systemctl enable docker
systemctl disable docker

systemctl list-unit-files|grep enabled

#普通用户加入docker组，运行docker命令
usermod  -aG docker ilog
{% endhighlight %}

### 查看Docker版本
{% highlight bash %}
root@ilog1:~# docker version
Client:
 Version:      17.06.0-ce
 API version:  1.30
 Go version:   go1.8.3
 Git commit:   02c1d87
 Built:        Fri Jun 23 21:23:31 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.06.0-ce
 API version:  1.30 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   02c1d87
 Built:        Fri Jun 23 21:19:04 2017
 OS/Arch:      linux/amd64
 Experimental: false
{% endhighlight %}

### 创建Swarm集群
{% highlight bash %}
docker swarm init --advertise-addr 172.18.60.133

#恢复
docker swarm init --advertise-addr 172.18.60.133 --force-new-cluster

#其它节点加入
docker swarm join --token \
     SWMTKN-1-44gjumnutrh4k9lls54f5hp43kiioxf16iuh7qarjfqjsu7jio-2326b8ikb1xiysm3i7neh9nho 172.18.60.133:2377
     
#输出可以用来以worker角色加入的token
docker swarm join-token worker

#输出可以用来以manager角色加入的token
docker swarm join-token manager

#manager节点强制脱离
docker swarm leave --force

#worker节点脱离
docker swarm leave

#节点从swarm中移除
docker node rm XXXXX

#worker节点提升为manager
docker node promote ilog2

#恢复为worker
docker node demote <NODE>

#创建服务
docker service create --replicas 3 --name helloworld alpine ping docker.com
{% endhighlight %}

### 加速器
{% highlight bash %}
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://0503c5a1.m.daocloud.io

#执行效果，新增文件/etc/docker/daemon.json 内容如下
{"registry-mirrors": ["http://0503c5a1.m.daocloud.io"]}
{% endhighlight %}

### 安全Registry
{% highlight bash %}
vim /lib/systemd/system/docker.service

#编辑如下行内容，新增--insecure-registry
ExecStart=/usr/bin/dockerd --insecure-registry http://0503c5a1.m.daocloud.io -H fd://
{% endhighlight %}

### Docker编排管理工具Portainer
{% highlight bash %}
#以服务模式
docker service create \
--name portainer \
--publish 9000:9000 \
--constraint 'node.role == manager' \
--mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
--mount type=bind,src=/opt/data,dst=/data \
portainer/portainer:1.13.6 \
-H unix:///var/run/docker.sock

#单容器模式
docker run -d -p 9000:9000 -v "/var/run/docker.sock:/var/run/docker.sock" -v "/opt/data:/data" \
    portainer/portainer -H unix:///var/run/docker.sock
{% endhighlight %}
利用镜像shipyard/docker-proxy将端口2375暴露，在portainer中管理多节点

### docker-proxy.yml
{% highlight bash %}
version: '3.3'
services:

  docker-proxy:
    image: shipyard/docker-proxy:latest
    hostname: docker-proxy
    deploy:
#      mode: global
      mode: replicated
      replicas: 3
      endpoint_mode: dnsrr
      restart_policy:
        condition: on-failure
        delay: 5s
    volumes:
      - type: bind
        source: /var/run/docker.sock
        target: /var/run/docker.sock
    networks:
      ilog_net:
        aliases:
          - docker-proxy
    ports:
      - target: 2375
        published: 2375
        protocol: tcp
        mode: host
    extra_hosts:
      - "ilog1:172.18.60.133"
      - "ilog2:172.18.60.134"
      - "ilog3:172.18.60.135" 
      
networks:
  ilog_net:
    driver: overlay
    external: true
{% endhighlight %}

{% include JB/setup %}


