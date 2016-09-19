---
layout: post
title: "CentOS7部署Ceph"
description: ""
category: 存储
tags: [存储, Linux]
---
{% include JB/setup %}
#### ceph-deploy部署需要完全在线，但是纯手工配置又过于繁琐，笔者最近尝试，手动安装然后用Ceph-deploy配置。

## 配置软件安装源
- vim /etc/yum.repos.d/ceph.repo
{%highlight bash%}
[ceph]
name=Ceph packages for $basearch
baseurl=http://ceph.com/rpm-hammer/el7/$basearch
enabled=1
priority=2
gpgcheck=1
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=http://ceph.com/rpm-hammer/el7/noarch
enabled=1
priority=2
gpgcheck=1
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=http://ceph.com/rpm-hammer/el7/SRPMS
enabled=0
priority=2
gpgcheck=1
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc
{%endhighlight%}

- vim /etc/yum.repos.d/epel.repo 
{%highlight bash%}
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch/debug
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/7/SRPMS
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
{%endhighlight%}

- vim /etc/yum.repos.d/ceph-fastcgi.repo
{%highlight bash%}
[fastcgi-ceph-basearch]
name=FastCGI basearch packages for Ceph
baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-centos7-x86_64-basic/ref/master
enabled=1
priority=2
gpgcheck=0
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

[fastcgi-ceph-noarch]
name=FastCGI noarch packages for Ceph
baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-centos7-x86_64-basic/ref/master
enabled=1
priority=2
gpgcheck=0
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

[fastcgi-ceph-source]
name=FastCGI source packages for Ceph
baseurl=http://gitbuilder.ceph.com/mod_fastcgi-rpm-centos7-x86_64-basic/ref/master
enabled=0
priority=2
gpgcheck=0
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc
{%endhighlight%}

## 安装组件
{%highlight bash%}

#配置用户
    echo "==========================================="
    echo "===Config ceph user"
    adduser -d /home/ceph -m ceph
    passwd ceph
    echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
    chmod 440 /etc/sudoers.d/ceph

#安装OSD组件
    echo "==========================================="
    echo "===Install OSD component"
    yum install -y snappy leveldb gdisk python-argparse gperftools-libs
    yum install -y ceph

#安装对象网关
    echo "==========================================="
    echo "===Install ceph gateway component"
    yum install -y ceph-radosgw ceph
    yum install -y radosgw-agent
{%endhighlight%}

## 配置集群
- 192.168.13.201    node1   OSD + Monitor
- 192.168.13.202    node2   OSD
- 192.168.13.203    node3   OSD
- 192.168.13.205    node5   Ceph-deploy
{%highlight bash%}
#在各节点安装相应组件,在node5上配置至node1~3的无密码访问
$mkdir ceph_cluster
$cd ceph_cluster

#创建基于某个节点为监控节点的集群信息
$ceph-deploy new node1

#修改配置新增一行，调整默认OSD数
$vim ceph.conf
osd pool default size = 3

#初始化监控节点
$ceph-deploy mon create-initial

#准备OSD存储
ssh node1 -- mkdir -p /var/local/osd1
ssh node2 -- mkdir -p /var/local/osd2
ssh node3 -- mkdir -p /var/local/osd3

#配置激活
$ceph-deploy osd prepare node1:/var/local/osd1 node2:/var/local/osd2 node3:/var/local/osd3
$ceph-deploy osd activate node1:/var/local/osd1 node2:/var/local/osd2 node3:/var/local/osd3

#拷贝管理配置文件至各节点
$ceph-deploy admin node1 node2 node3
$ssh node1 -- chmod +r /etc/ceph/ceph.client.admin.keyring
$ssh node2 -- chmod +r /etc/ceph/ceph.client.admin.keyring
$ssh node3 -- chmod +r /etc/ceph/ceph.client.admin.keyring
{%endhighlight%}

## 创建用户
{%highlight bash%}
$ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images'
$ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images'

#检测安装
$ceph health
{%endhighlight%}

## 小结
- 配置Ceph源，第三方EPEL源
- 各节点安装组件
- 通过ceph-deploy生成集群信息，准备OSD存储，命令激活配置

