---
layout: post
title: "Ubuntu 14.04 64bits安装Maven3.2.5"
description: ""
category: "Linux"
tags: []
---
{% include JB/setup %}
<p>
下载源码截止2015年1月7日稳定版本3.2.5<br/>
官网页面：http://maven.apache.org/download.cgi <br/>
文件链接[7.58M]：http://ftp.jaist.ac.jp/pub/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz <br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1o6A2Cp8
</p>
{% highlight ruby %}
tar zxvf apache-maven-3.2.5-bin.tar.gz -C /usr/local/	解压至目标目录
  
重命名[可选]，后续和环境变量保持一致
cd /usr/local
mv mv apache-maven-3.2.5/ apache-maven
  
配置Maven环境变量，依赖JAVA环境			可将以下编辑添加到/etc/profile后者~/.bashrc
export M2_HOME=/usr/local/apache-maven
export M2=$M2_HOME/bin
export PATH=$M2:$PATH
......
设置JAVA环境变量，如已经设置无需操作
    
mvn --version	验证安装
#Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-14T12:29:23-05:00)
#Maven home: /usr/local/apache-maven
#Java version: 1.8.0_25, vendor: Oracle Corporation
#Java home: /usr/lib/jvm/java-8-oracle/jre
#Default locale: en_US, platform encoding: UTF-8
#OS name: "linux", version: "3.13.0-24-generic", arch: "amd64", family: "unix"
{% endhighlight%}
<p>
Ubuntu 14.04 64bits 安装JDK8 <br/>
参见：http://jinlongliu.github.io/linux/2015/01/06/install-jdk8-on-ubuntu14.04lts/
</p>
