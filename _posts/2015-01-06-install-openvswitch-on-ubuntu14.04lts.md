---
layout: post
title: "Ubuntu 14.04 64bits安装OpenvSwitch2.3.1"
description: ""
category: "Linux"
tags: []
---
{% include JB/setup %}
<p>
下载源码：<br/>
下载页面：http://openvswitch.org/download/ <br/>
文件链接：http://openvswitch.org/releases/openvswitch-2.3.1.tar.gz <br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1dDGkHn7
</p>
{% highlight bash %}
#预先安装
apt-get install dh-autoreconf libssl-dev openssl
#编译源码
tar -xvf openvswitch-2.3.1.tar.gz
cd openvswitch-2.3.1/
./configure --with-linux=/lib/modules/`uname -r`/build		默认安装在/usr/local
  
#安装在其它路径使用 --prefix
./configure --prefix=/usr --localstatedir=/var --with-linux=/lib/modules/`uname -r`/build
  
make -j && sudo make install && make modules_install
  
#加载模块
modprobe gre
modprobe openvswitch
modprobe libcrc32c
  
#设置ovsdb
ovsdb-tool create /usr/local/etc/openvswitch/conf.db /usr/local/share/openvswitch/vswitch.ovsschema
  
#开启ovsdb-server(NO SSL)
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
--remote=db:Open_vSwitch,Open_vSwitch,manager_options \
--pidfile --detach --log-file
  
#开启ovsdb-server(SSL)
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
--remote=db:Open_vSwitch,Open_vSwitch,manager_options \
--private-key=db:Open_vSwitch,SSL,private_key \
--certificate=db:Open_vSwitch,SSL,certificate \
--bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
--pidfile --detach --log-file
  
#开启ovs-vsctl
ovs-vsctl --no-wait init
  
#开启ovs-switchd功能
ovs-vswitchd --pidfile --detach --log-file
  
#自动加载模块
echo "openvswitch " >> /etc/modules
echo "gre" >> /etc/modules
echo "libcrc32c" >> /etc/modules
  
#开启自启动
vim /etc/init.d/openvswitch
#!/bin/sh
start-stop-daemon -q -S -x /usr/local/sbin/ovsdb-server -- --remote=punix:/usr/local/var/run/openvswitch/db.sock --remote=db:Open_vSwitch,Open_vSwitch,manager_options --pidfile --detach --log-file
sleep 3 # waiting ovsdb-server 
start-stop-daemon -q -S -x /usr/local/bin/ovs-vsctl -- --no-wait init
start-stop-daemon -q -S -x /usr/local/sbin/ovs-vswitchd -- --pidfile --detach --log-file
  
chmod +x /etc/init.d/openvswitch
update-rc.d -f openvswitch defaults
  
验证安装
root@king:/# ovs-vsctl -V
ovs-vsctl (Open vSwitch) 2.3.1
Compiled Jan  8 2015 11:38:39
DB Schema 7.6.2
{% endhighlight %}
