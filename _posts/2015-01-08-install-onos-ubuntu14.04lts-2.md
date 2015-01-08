---
layout: post
title: "Ubuntu 14.04 64bits安装ONOS[非源码]"
description: ""
category: "ONOS" 
tags: ["SDN", "ONOS"]
---
{% include JB/setup %}
<p>
下载发布包：<br/>
官方页面:https://wiki.onosproject.org/display/ONOS/Downloads <br/>
文件链接：http://downloads.onosproject.org/release/onos-1.0.0.tar.gz <br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1qWFitBY <br/>
</p>
<p>
安装JAVA运行环境，源码安装要求JAVA 8，所以jre应该也是一样。<br/>
可以直接安装JDK8，文档参见：http://jinlongliu.github.io/linux/2015/01/06/install-jdk8-on-ubuntu14.04lts/ <br/>
Maven 依然需要因为Karaf属性描述都是使用mvn <br/>
文档参见：http://jinlongliu.github.io/linux/2015/01/06/install-apache-maven-on-ubuntu14.04lts/
</p>
{% highlight bash %}
tar zxvf onos-1.0.0.tar.gz -C /usr/local
cd /usr/local/onos-1.0.0  该目录中包含了apache-karaf，所以之前无须安装karaf
  
#配置Karaf环境变量
export KARAF_ROOT=/usr/local/onos-1.0.0/apache-karaf-3.0.2
export KARAF=$KARAF_ROOT/bin
export PATH=$KARAF:$PATH
  
root@king:/# karaf clean
Welcome to Open Network Operating System (ONOS)!
     ____  _  ______  ____   
    / __ \/ |/ / __ \/ __/    
   / /_/ /    / /_/ /\ \       
   \____/_/|_/\____/___/      
                             
  
Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown ONOS.
  
onos>
{% endhighlight %}
<p>
配置完Karaf环境变量，直接运行karaf，即运行ONOS，可通过#netstat -lnput查看端口8181、6633监听情况。<br/>
源码安装ONOS参见：http://jinlongliu.github.io/onos/2015/01/08/install-onos-on-ubuntu14.04lts/
</p>


