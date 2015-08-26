---
layout: post
title: "Ubuntu 14.40 安装Kubernetes 1.0.1"
description: ""
category: "Docker"
tags: ["Docker"]
---
{% include JB/setup %}
<p>
首先在各节点安装docker,可以参见<br/>
http://blog.onos.top/docker/2015/08/26/install-docker-1.8.1-on-ubuntu14.04/<br/>
笔者的实验环境如下：<br/>
10.0.0.27  docker3<br/>
10.0.0.26  docker2<br/>
10.0.0.25  docker1<br/>
</p>
<p>
三个节点是Openstack上的虚拟机，启动docker3配置了浮动IP和笔者的PC相连。<br/>
准备工作：三个节点的SSH无密码访问，因为三个节点是VM，docker2，docker1不能和PC直连，只能通过docker3 SSH跳转。
</p>
<p>
下载Kubernetes安装包，只需要下载tar.gz结尾即可,并上传至docker3<br/>
http://yun.baidu.com/share/link?shareid=1517923564&uk=1818218422#path=%252F01Software%252F01-Kubernetes<br/>
并下载build.sh备用。
</p>
{% highlight bash %}
root@docker3:~/k8s# ls
build.sh  etcd-v2.1.1-linux-amd64.tar.gz  flannel-0.5.2-linux-amd64.tar.gz kubernetes.tar.gz
root@docker3:~/k8s# tar -xvf kubernetes.tar.gz
root@docker3:~/k8s# ls
build.sh  etcd-v2.1.1-linux-amd64.tar.gz  flannel-0.5.2-linux-amd64.tar.gz  kubernetes  kubernetes.tar.gz
root@docker3:~/k8s# cd kubernetes/cluster/ubuntu/
root@docker3:~/k8s/kubernetes/cluster/ubuntu# vim build.sh
{% endhighlight %}
<p>编辑build.sh，原先的代码主要是从网络下载三个tar.gz包并解压操作，笔者修改为直接使用上传的包操作。<br/>
可以用之前下载的build.sh直接覆盖解压后原先的脚本</p>
{% highlight bash %}
root@docker3:~/k8s/kubernetes/cluster/ubuntu# ./build.sh
移动解压tar.gz并创建新的binaries目录
root@docker3:~/k8s/kubernetes/cluster/ubuntu/binaries# ls
kubectl  master  minion
{% endhighlight %}
<p>
上面的操作把需要部署的二进制文件集中到了cluster/ubuntu/binaries内，后面我们进入cluster/ubuntu目录进行节点的部署。
</p>
{% highlight bash %}
root@docker3:~/k8s/kubernetes/cluster/ubuntu# vim config-default.sh 
主要配置以下参数：
export nodes=${nodes:-"root@docker3 root@docker2 root@docker1"}  节点的用户名@IP，因为笔者在/etc/hosts配置了主机名和IP
所以此处用的是域名
roles=${roles:-"ai i i"}   节点的角色a:master i:minion  ai:master and minion 分别是单词的第二个字母  
export NUM_MINIONS=${NUM_MINIONS:-3}
{% endhighlight %}
<p>
笔者刚好是说那个节点所以，基本上只修改了nodes这个值，其它均保留。注意nodes和roles位置是一一对应的。
</p>
{% highlight bash %}
#cd ~/k8s/kubernetes/cluster
$ KUBERNETES_PROVIDER=ubuntu ./kube-up.sh
上面的脚本会根据config-defualt的节点配置把cluster/ubuntu/binaries里面的文件拷贝到nodes配置的节点上。<br/>
如果之前配置了docker3到docker1,docker2以及自己的无密码访问就无需输入密码，负责需要输入节点密码。
{% endhighlight %}
<p>
以上脚本可能会执行失败，主要是docker安装配置的网桥和k8s启动docker的配置不一致。
</p>
{% highlight bash %}
root@docker3:~# ifconfig docker0
docker0   Link encap:Ethernet  HWaddr 02:42:e7:a0:83:09  
          inet addr:172.16.8.1  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::42:e7ff:fea0:8309/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1372  Metric:1
          RX packets:2481 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2293 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:4196199 (4.1 MB)  TX bytes:1948029 (1.9 MB)
root@docker3:~# cat /etc/default/docker 
DOCKER_OPTS="-H tcp://127.0.0.1:4243 -H unix:///var/run/docker.sock --bip=172.16.8.1/24 --mtu=1372 --insecure-registry=docker3:5000"
其中Linux网桥docker0的IP需要和docker启动参数bip的地址一致。
  
  
三个节点都修改一致后，重启docker，然后在docker3查看集群情况
root@docker3:~# kubectl get nodes
NAME      LABELS                           STATUS
docker1   kubernetes.io/hostname=docker1   Ready
docker2   kubernetes.io/hostname=docker2   Ready
docker3   kubernetes.io/hostname=docker3   Ready
{% endhighlight %}
<p>
至此Kubernetes的基本安装部署就完成了，如果kubectl提示没有，可以从cluster/ubuntu/binaries里面拷贝至/usr/bin等某个系统目录即可，或者<br/>
配置PATH指向kubectl所在路径。下一步可以着手安装kube-ui,请参考笔者其它博文。
</p>














