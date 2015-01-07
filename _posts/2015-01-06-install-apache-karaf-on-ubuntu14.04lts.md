---
layout: post
title: "Ubuntu 14.04 64bits安装Apache Karaf"
description: ""
category: "Linux"
tags: []
---
{% include JB/setup %}
<p>
版本链接：http://karaf.apache.org/index/community/download.html <br/>
3.0.2下载页面：http://www.apache.org/dyn/closer.cgi/karaf/3.0.2/apache-karaf-3.0.2.tar.gz
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1dDcVbfN
</p>
{% highlight ruby %}
tar zxvf apache-karaf-3.0.2.tar.gz -C /usr/local/
  
重命名
cd /usr/local
mv apache-karaf-3.0.2/ apache-karaf
  
配置环境变量，依赖JAVA_HOME	可将以下编辑添加到/etc/profile或者~/.bashrc
export KARAF_HOME=/usr/local/apache-karaf/
export KARAF=$KARAF_HOME/bin
export PATH=$KARAF:$PATH
  
其它环境变量
KARAF_DATA	数据文件目录，Karaf存储临时文件
KARAF_ETC	配置文件目录，Karaf存储配置文件
KARAF_BASE	和KARAF_HOME作用相同
  
测试安装
root@king:/bin# karaf 
karaf: Ignoring predefined value for KARAF_HOME
        __ __                  ____      
       / //_/____ __________ _/ __/      
      / ,<  / __ `/ ___/ __ `/ /_        
     / /| |/ /_/ / /  / /_/ / __/        
    /_/ |_|\__,_/_/   \__,_/_/         
  
  Apache Karaf (3.0.2)
  
Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown Karaf.
  
karaf@root()> 
   
Ctrl+D 关闭
{% endhighlight %}
<p>
Ubuntu 14.04 64bits 安装JDK8 <br/>
参见：http://jinlongliu.github.io/linux/2015/01/06/install-jdk8-on-ubuntu14.04lts/
</p>
