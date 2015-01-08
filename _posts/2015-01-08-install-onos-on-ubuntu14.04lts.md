---
layout: post
title: "Ubuntu 14.04 64bits安装ONOS"
description: ""
category: "ONOS"
tags: ["SDN", "ONOS"]
---
{% include JB/setup %}
<p>
英文安装参见：https://wiki.onosproject.org/display/ONOS/Installing+and+Running+ONOS <br/>
下载源码: <br/>
</p>
{% highlight bash %}
$ git clone https://gerrit.onosproject.org/onos
$ cd onos
$ git checkout 1.0.0
{% endhighlight %}
<p>
上述需要从国外下载源码速度有限，可以从ONOS技术群[425319659]共享中心下载1.0.0源码 <br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1i3tHPZJ
</p>
{% highlight bash %}
tar -xvf onos-1.0.0-src.tar.gz
cd onos
{% endhighlight %}
<p>
官方推荐配置: <br/>
1.Ubuntu Server 14.04 LTS 64-bit <br/>
2.2GB or more RAM  <br/>
3.2 or more processors
</p>
<p>
依赖软件：<br/>
1.Java 8 JDK(推荐Oracle Java, OpenJDK未完全测试) 【开发环境JDK】 <br/>
2.Apache Maven(3.0+) 【编译打包工具】<br/>
3.Apache Karaf(3.0.2+) 【部署运行容器】<br/>
4.git(用于获取源码，如果源码已下载可不安装)
</p>
<p>
参考文档：<br/>
1.Ubuntu 14.04 64bits安装JDK8        http://jinlongliu.github.io/linux/2015/01/06/install-jdk8-on-ubuntu14.04lts/   <br/>
2.Ubuntu 14.04 64bits安装Maven3.2.5      http://jinlongliu.github.io/linux/2015/01/06/install-apache-maven-on-ubuntu14.04lts/  <br/>
3.Ubuntu 14.04 64bits安装Apache Karaf    http://jinlongliu.github.io/linux/2015/01/06/install-apache-karaf-on-ubuntu14.04lts/   <br/>
</p>
{% highlight bash %}
验证上述安装
root@king:/# java -version
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
  
root@king:/# mvn --version
Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-15T01:29:23+08:00)
Maven home: /usr/local/apache-maven
Java version: 1.8.0_25, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-8-oracle/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.13.0-24-generic", arch: "amd64", family: "unix"
    
root@king:/# karaf clean
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
{% endhighlight %}
<p>
下载ONOS Maven库文件 <br/>
ONOS技术群[425319659]共享中心: http://pan.baidu.com/s/1mgwZ6YG
</p>
{% highlight bash %}
tar -xvf onos-maven-repos.tar.gz   解压生成目录repos
{% endhighlight %}
<p>
将Maven本地库设置指向，上述解压路径。设置方法参见：http://jinlongliu.github.io/linux/2015/01/06/install-apache-maven-on-ubuntu14.04lts/
</p>
<p>
编译安装ONOS源码
</p>
{% highlight bash %}
$cd onos
$export ONOS_ROOT=./
$source $ONOS_ROOT/tools/dev/bash_profile   此处导入各种变量，会导入KARAF_ROOT,如果之前未设置，需要修改此文件默认值
  
$mvn clean install	此处如果设置了本地仓储，编译安装是很快的。
.......
.......
[INFO] --- maven-bundle-plugin:2.5.3:install (default-install) @ onos-branding ---
[INFO] Installing org/onosproject/onos-branding/1.0.0/onos-branding-1.0.0.jar
[INFO] Writing OBR metadata
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] onos ............................................... SUCCESS [  2.065 s]
[INFO] onlab-utils ........................................ SUCCESS [  1.438 s]
[INFO] onlab-junit ........................................ SUCCESS [  4.572 s]
[INFO] onlab-misc ......................................... SUCCESS [  7.871 s]
[INFO] onlab-netty ........................................ SUCCESS [  1.490 s]
[INFO] onlab-nio .......................................... SUCCESS [  1.906 s]
[INFO] onlab-osgi ......................................... SUCCESS [  0.548 s]
[INFO] onlab-rest ......................................... SUCCESS [  0.547 s]
[INFO] onlab-thirdparty ................................... SUCCESS [  1.101 s]
[INFO] onos-core .......................................... SUCCESS [  0.296 s]
[INFO] onos-api ........................................... SUCCESS [  5.976 s]
[INFO] onos-core-store .................................... SUCCESS [  0.415 s]
[INFO] onos-core-trivial .................................. SUCCESS [  2.801 s]
[INFO] onos-core-net ...................................... SUCCESS [  6.533 s]
[INFO] onos-core-serializers .............................. SUCCESS [  2.443 s]
[INFO] onos-core-dist ..................................... SUCCESS [ 20.423 s]
[INFO] onos-json .......................................... SUCCESS [  0.428 s]
[INFO] onos-web ........................................... SUCCESS [  0.281 s]
[INFO] onos-gui ........................................... SUCCESS [  0.810 s]
[INFO] onos-rest .......................................... SUCCESS [  1.427 s]
[INFO] onos-cli ........................................... SUCCESS [  0.986 s]
[INFO] onos-of ............................................ SUCCESS [  0.456 s]
[INFO] onos-of-api ........................................ SUCCESS [  6.997 s]
[INFO] onos-providers ..................................... SUCCESS [  0.295 s]
[INFO] onos-of-providers .................................. SUCCESS [  0.342 s]
[INFO] onos-of-provider-device ............................ SUCCESS [  1.814 s]
[INFO] onos-of-provider-link .............................. SUCCESS [  1.630 s]
[INFO] onos-of-provider-host .............................. SUCCESS [  1.430 s]
[INFO] onos-of-provider-packet ............................ SUCCESS [  2.114 s]
[INFO] onos-of-provider-flow .............................. SUCCESS [  0.960 s]
[INFO] onos-lldp-provider ................................. SUCCESS [  2.129 s]
[INFO] onos-host-provider ................................. SUCCESS [  1.324 s]
[INFO] onos-of-drivers .................................... SUCCESS [  0.599 s]
[INFO] onos-of-ctl ........................................ SUCCESS [  1.758 s]
[INFO] onos-apps .......................................... SUCCESS [  0.312 s]
[INFO] onos-app-tvue ...................................... SUCCESS [  0.489 s]
[INFO] onos-app-fwd ....................................... SUCCESS [  0.422 s]
[INFO] onos-app-ifwd ...................................... SUCCESS [  0.376 s]
[INFO] onos-app-mobility .................................. SUCCESS [  0.384 s]
[INFO] onos-app-proxyarp .................................. SUCCESS [  0.351 s]
[INFO] onos-app-config .................................... SUCCESS [  0.388 s]
[INFO] onos-app-sdnip ..................................... SUCCESS [  5.384 s]
[INFO] onos-app-calendar .................................. SUCCESS [  0.435 s]
[INFO] onos-app-optical ................................... SUCCESS [  0.578 s]
[INFO] onos-app-metrics ................................... SUCCESS [  0.261 s]
[INFO] onos-app-metrics-intent ............................ SUCCESS [  0.503 s]
[INFO] onos-app-metrics-topology .......................... SUCCESS [  0.595 s]
[INFO] onos-app-oecfg ..................................... SUCCESS [  1.315 s]
[INFO] onos-app-demo ...................................... SUCCESS [  0.535 s]
[INFO] onos-app-election .................................. SUCCESS [  0.470 s]
[INFO] onos-features ...................................... SUCCESS [  0.524 s]
[INFO] onos-branding ...................................... SUCCESS [  0.530 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:41 min
[INFO] Finished at: 2015-01-08T14:58:06+08:00
[INFO] Final Memory: 91M/326M
[INFO] ------------------------------------------------------------------------
{% endhighlight %}
<p>
如果想跳过测试可以使用如下，速度会更快一点.
</p>
{% highlight bash %}
$mvn clean install -Dmaven.test.skip=true
{% endhighlight %}
<p>
修改Karaf配置，以加载ONOS
</p>
{% highlight bash %}
vim $KARAF_ROOT/etc/org.apache.karaf.features.cfg
在featuresRepositories后添加mvn:org.onosproject/onos-features/1.0.0/xml/features 用逗号隔开
在featuresBoot后添加onos-api,onos-core-trivial,onos-cli,onos-openflow,onos-app-fwd,onos-app-mobility,onos-gui 用逗号隔开
  
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
正确加载后LOGO已经发生变化。<br/>
查看WebGui和控制器端口监听是否正常
</p>
{% highlight bash %}
root@king:/# netstat -lnput |grep 8181
tcp6       0      0 :::8181                 :::*                    LISTEN      21716/java      
root@king:/# netstat -lnput |grep 6633
tcp6       0      0 :::6633                 :::*                    LISTEN      21716/java
{% endhighlight %}
<p>
访问http://[your_ server_ip]:8181/onos/ui/index.html查看WebGui,请使用Chrome查看，笔者测试IE和QQ浏览器浮动窗口效果均有问题。 <br/>
使用Mininet建立拓扑，此时Mininet安装可以使用apt-get install mininet,不过似乎后面它会安装openvswitch可能会报错，忽略。service openvswitch-controller stop
</p>
