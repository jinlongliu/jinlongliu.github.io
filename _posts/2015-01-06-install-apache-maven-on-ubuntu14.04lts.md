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
<p>
配置国内仓库，本地仓库
</p>
{% highlight ruby %}
cd $M2_HOME/conf	进入Maven配置文件目录
vim settings.xml
{% endhighlight %}
<p>
本地仓库，当进行编译打包操作时会把远程仓库的包下载到本地仓库。<br/>
另外的目的配置了本地仓库，只要将仓库包添加到该路径下，就不需要再到远程下载了。 <br/>
ONOS技术群[425319659]共享中心,后续会提供ONOS编译的依赖库。
</p>
{% highlight xml %}
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>/usr/local/repos</localRepository>
  <!-- 此处/usr/local/repos可以修改为自己定义的路径 --> 
  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->
{% endhighlight %}
<p>
国内仓库OSChina中央库
</p>
{% highlight xml %}
 <mirrors>
    <mirror>
      <id>CN</id>
      <name>OSChina Central</name>
      <url>http://maven.oschina.net/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
    <mirror>
      <id>ui</id>
      <mirrorOf>central</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://uk.maven.org/maven2/</url>
    </mirror>
 </mirrors>
{% endhighlight %}
