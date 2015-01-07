---
layout: post
title: "Ubuntu 14.04 64bits安装JDK8"
description: ""
category: "Linux"
tags: ["JDK8", "ONOS"]
---
{% include JB/setup %}
<p>
方法一：<br/>
</p>
{% highlight ruby %}
$ sudo apt-get install software-properties-common -y
$ sudo add-apt-repository ppa:webupd8team/java -y
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer oracle-java8-set-default -y
$ export JAVA_HOME=/usr/lib/jvm/java-8-oracle		设置JAVA环境变量，可以把该行编辑添加到/etc/profile 或者~/.bashrc
{% endhighlight %}

<p>
方法二：<br/>
下载源码安装,方法一实质上也是下载改方法的文件。所以不如直接用迅雷下载文件然后解压配置即可。<br/>
下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html <br/>
文件:Linux x64 153.42 MB    jdk-8u25-linux-x64.tar.gz [2015年1月7日] <br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1qWM9Q1Y
</p>
{% highlight ruby %}
tar zxvf ./jdk-8u5-linux-x64.tar.gz  -C /usr/lib/jvm	解压文件至目录/usr/lib/jvm生成文件目录jdk1.8.0_05
  
cd /usr/lib/jvm
mv jdk1.8.0_05 java-8-oracle				重命名文件夹，不重命名也可，后续JAVA环境变量设置保持一致即可
  
vim ~/.bashrc						设置JAVA环境变量，也可以加入到/etc/profile内
export JAVA_HOME=/usr/lib/jvm/java-8-oracle    
export JRE_HOME=${JAVA_HOME}/jre    
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib    
export PATH=${JAVA_HOME}/bin:$PATH
  
source ~/.bashrc		导入新的环境变量
  
java -version			查看JAVA环境变量是否正常			
#java version "1.8.0_25"
#Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
#Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
{% endhighlight %}

