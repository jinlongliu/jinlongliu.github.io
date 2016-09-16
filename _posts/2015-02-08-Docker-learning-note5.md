---
layout: post
title: "Docker知识梳理（五）"
description: ""
category: Docker 
tags: [Docker, Linux]
---
{% include JB/setup %}
## 实践梳理
- docker search -s 10 ubuntu 搜索收藏10次以上的镜像
- Dockerfile 创建镜像时会继承父镜像的开放端口
- 设置网桥

{%highlight bash%}
$yum install bridge-utils
$brctl addbr br0
$ifconfig br0 172.18.0.1 netmask 255.255.255.0 up

$ifconfig br0 down
$brctl delbr br
{%endhighlight%} 
- Docker更改绑定的网桥,通过-b参数修改
{%highlight bash%}
$vim /usr/lib/systemd/system/docker.servic
ExecStart=/usr/bin/dockerd -b=br0
{%endhighlight%}

- DevOps(Development and Operations)

## Docker核心技术
- Docker默认监听unix:///var/run/docker.sock
- 通过-H 0.0.0.0:1234 修改监听的方式,docker -H 127.0.0.1:1234 version
- 控制组(CGroups)对共享资源进行隔离、限制、审计。
- /sys/fs/cgroup/memorg/docker/ Docker组应用的限制项,可修改。
- 手动创建容器网络
{%highlight bash%}
$docker run -i -t --rm --net=none ubuntu /bin/bas
$docker inspect -f '{{.State.Pid}}'  03f
4511
$pid=4511
$mkdir -p /var/run/netns
$ln -s /proc/$pid/ns/net /var/run/netns/$pid
$ip addr show docker0
$ip link add A type veth peer name B
$brctl addif docker0 A
$ip link set A up
$ip link set B netns $pid
$ip netns exec $pid ip link set dev B name eth0
$ip netns exec $pid ip link set eth0 up
$ip netns exec $pid ip addr add 172.17.0.50/16 dev eth0
$ip netns exec $pid ip route add default via 172.17.0.1
{%endhighlight%}

## Docker高级网络配置
- -b BRIDGE or --bridge=BRIDGE  指定挂载网桥
- --bip=CIDR  定制掩码
- -H SOCKET ... or --host=SOCKET...  服务器端接收命令通道
- --icc=true|false 是否支持容器间通信
- --ip-forward=true|false 启用net.ipv4.ip forward
- --iptables=true|false 禁止Docker添加iptables规则 
- --mtu=BYTES 容器网络中的MTU

## 其它
- 临时退出交互的终端，不终止。  Ctrl-p  Ctrl-q

