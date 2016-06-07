---
layout: post
title: "Ansible 学习笔记（一）"
description: ""
category: 运维
tags: [Ansible, XOS]
---
{% include JB/setup %}

### CentOS7安装Ansible
{% highlight bash %}
$git clone git://github.com/ansible/ansible.git --recursive    #下载源码
$cd ./ansible	
$source ./hacking/env-setup    #配置环境变量

#其它程序
$sudo easy_install pip	
$sudo pip install paramiko PyYAML Jinja2 httplib2 six

$echo "127.0.0.1" > ~/ansible_hosts
$export ANSIBLE_INVENTORY=~/ansible_hosts
{% endhighlight %}

### CentOS7安装sshpass
配置EPEL源
{% highlight bash %}
$vim /etc/yum.repos.d/epel.repo

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
gpgcheck=1
[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/7/SRPMS
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
{% endhighlight %}

安装SSHPASS
{% highlight bash %}
#安装
$yum --enablerepo=epel -y install sshpass

#使用
$sshpass -p password ssh 192.168.100.143 hostname

$echo 'cctcloud' >~/password.txt
$chmod 600 ~/password.txt 
$sshpass -f ~/password.txt ssh 192.168.100.143 hostname

$export SSHPASS=cctcloud
$sshpass -e ssh 192.168.100.143 hostname  
{% endhighlight %}

### 测试
{% highlight bash %}
$ansible --version
ansible 2.2.0 (devel 6f36909074) last updated 2016/06/07 12:43:06 (GMT +800)
  lib/ansible/modules/core: (detached HEAD cb1093e085) last updated 2016/06/07 12:43:23 (GMT +800)
  lib/ansible/modules/extras: (detached HEAD 66b60ce7cd) last updated 2016/06/07 12:43:39 (GMT +800)
  config file = 
  configured module search path = Default w/o overrides

$echo "192.168.100.143" > ~/ansible_hosts   #目标节点IP输入到主机列表
$ansible all -m ping --ask-pass
SSH password: 
192.168.100.143 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}

$ansible all -m ping --ask-pass
SSH password: 
192.168.100.143 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
192.168.13.33 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
{% endhighlight %}

### 特性
- (1)、no agents：不需要在被管控主机上安装任何客户端；
- (2)、no server：无服务器端，使用时直接运行命令即可；
- (3)、modules in any languages：基于模块工作，可使用任意语言开发模块；
- (4)、yaml，not code：使用yaml语言定制剧本playbook；
- (5)、ssh by default：基于SSH工作；
- (6)、strong multi-tier solution：可实现多级指挥。

### 开始
默认是SSH keys可以用--ask-pass启用密码认证，--ask-become-pass 启动sudo认证<br/>
禁用主机key检测
{% highlight bash %}
$export ANSIBLE_HOST_KEY_CHECKING=False
{% endhighlight %}

