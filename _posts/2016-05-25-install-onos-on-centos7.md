---
layout: post
title: "CentOS7 ONOS 离线编译安装"
description: "ONOS离线安装"
category: ONOS
tags: []
---

#### 本文主要记录CentOS7安装ONOS并汇总了需要的软件进行离线编译[应该适用Ubuntu14.04]
权威的官方链接：https://wiki.onosproject.org/display/ONOS/Installing+and+Running+ONOS

### ONOS编译主要需要如下组件
- JDK         
- Maven
- Maven Dependency
- Karaf
- ONOS source code

### 下载离线包
http://pan.baidu.com/s/1bnfkIlx#path=%252F01Software%252F02-ONOS <br/>

#### ONOS-offlive-build.tar.gz

### 离线包解压
{% highlight bash %}
[root@controller ~]# tar -xvf ONOS-offlive-build.tar.gz
###############
[root@controller ONOS-offlive-build]# ls -alh
total 503M
drwxr-xr-x   2 root root  154 Jun  1 14:42 .
dr-xr-x---. 15 root root 4.0K Jun  1 14:43 ..
-rw-r--r--.  1 root root  18M May 23 23:28 apache-karaf-3.0.5.tar.gz
-rw-r--r--.  1 root root 8.1M May 23 23:28 apache-maven-3.3.9-bin.tar.gz
-rw-r--r--.  1 root root 173M May 23 23:28 jdk-8u92-linux-x64.tar.gz
-rw-r--r--.  1 root root 206M May 23 23:29 maven_repo.tar.gz
-rw-r--r--.  1 root root  98M May 23 23:29 onos-1.5.1.tar.gz
{% endhighlight %}

### 解压组件到指定目录
{% highlight bash %}
[root@controller ONOS-offlive-build]# tar -xvf jdk-8u92-linux-x64.tar.gz -C /usr/local/share/
[root@controller ONOS-offlive-build]# tar -xvf apache-maven-3.3.9-bin.tar.gz -C /usr/local/share/
[root@controller ONOS-offlive-build]# tar -xvf maven_repo.tar.gz -C /
[root@controller ONOS-offlive-build]# tar -xvf apache-karaf-3.0.5.tar.gz -C /usr/local/share/
[root@controller ONOS-offlive-build]# tar -xvf onos-1.5.1.tar.gz -C /home/
{% endhighlight %}

{% highlight bash %}
#目录/usr/local/share
[root@controller share]# ls -al
total 28
drwxr-xr-x. 12 root root 4096 May 24 10:45 .
drwxr-xr-x. 12 root root 4096 May 14 05:07 ..
drwxr-xr-x. 10 root root 4096 May 23 23:54 apache-karaf-3.0.5
drwxr-xr-x.  6 root root   92 May 23 23:38 apache-maven-3.3.9
drwxr-xr-x.  8   10  143 4096 Apr  1 12:20 jdk1.8.0_92

#目录/home
[root@controller home]# ls -al
total 224036
drwxr-xr-x.  4 root root        46 May 23 23:28 .
dr-xr-xr-x. 18 root root      4096 May 25 18:03 ..
drwxr-xr-x. 16 root root      4096 May 23 23:47 onos
{% endhighlight %}

### 配置环境变量
{% highlight bash %}
[root@controller /]# vim ~/.bash_profile 
##########新增如下内容
export JAVA_HOME=/usr/local/share/jdk1.8.0_92
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin

export M2_HOME=/usr/local/share/apache-maven-3.3.9
export M2=$M2_HOME/bin
export M2_REPO=/maven_repo
export PATH=$M2:$PATH

export KARAF_ROOT=/usr/local/share/apache-karaf-3.0.5
export KARAF=$KARAF_ROOT/bin
export PATH=$KARAF:$PATH

export ONOS_ROOT=/home/onos
source $ONOS_ROOT/tools/dev/bash_profile
###########导入变量，下次登录会自动导入
[root@controller /]# source ~/.bash_profile

###########验证
[root@controller /]# java -version
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)

[root@controller /]# mvn -version
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /usr/local/share/apache-maven-3.3.9
Java version: 1.8.0_92, vendor: Oracle Corporation
Java home: /usr/local/share/jdk1.8.0_92/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-327.18.2.el7.x86_64", arch: "amd64", family: "unix"
{% endhighlight %}

### 设置Maven本地库
{% highlight bash %}
[root@controller /]# vim /usr/local/share/apache-maven-3.3.9/conf/settings.xml
#################################
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>/maven_repo</localRepository>
  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->
  ......
#################################
<localRepository>/maven_repo</localRepository>
{% endhighlight %}

### 开始编译
{% highlight bash %}
cd /home/onos
[root@controller onos]# mvn clean install
[INFO] --- maven-bundle-plugin:3.0.1:install (default-install) @ onos-branding ---
[INFO] Installing org/onosproject/onos-branding/1.5.1/onos-branding-1.5.1.jar
[INFO] Writing OBR metadata
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] onos ............................................... SUCCESS [  2.482 s]
[INFO] onlab-utils ........................................ SUCCESS [  0.821 s]
[INFO] onlab-junit ........................................ SUCCESS [  4.354 s]
[INFO] onlab-misc ......................................... SUCCESS [ 12.401 s]
[INFO] onos-yang-utils-plugin ............................. SUCCESS [  9.430 s]
[INFO] onlab-osgi ......................................... SUCCESS [  1.166 s]
[INFO] onlab-rest ......................................... SUCCESS [  0.698 s]
[INFO] onlab-thirdparty ................................... SUCCESS [  2.650 s]
[INFO] onlab-stc .......................................... SUCCESS [  3.093 s]
[INFO] jdvue .............................................. SUCCESS [  2.280 s]
[INFO] onlab-jnc-osgi ..................................... SUCCESS [  0.348 s]
[INFO] utils.catalyst ..................................... SUCCESS [  0.718 s]
[INFO] onos-core .......................................... SUCCESS [  0.482 s]
[INFO] onos-api ........................................... SUCCESS [  8.708 s]
[INFO] onos-core-common ................................... SUCCESS [  3.972 s]
[INFO] onos-incubator ..................................... SUCCESS [  0.188 s]
[INFO] onos-incubator-api ................................. SUCCESS [  2.683 s]
[INFO] onos-cli ........................................... SUCCESS [  1.543 s]
[INFO] onos-core-store .................................... SUCCESS [  0.164 s]
[INFO] onos-core-serializers .............................. SUCCESS [  2.175 s]
[INFO] onos-core-persistence .............................. SUCCESS [  0.397 s]
[INFO] onos-core-dist ..................................... SUCCESS [  5.506 s]
[INFO] onos-core-net ...................................... SUCCESS [  7.879 s]
[INFO] onos-core-primitives ............................... SUCCESS [  3.393 s]
[INFO] onos-security ...................................... SUCCESS [  0.727 s]
[INFO] onos-web ........................................... SUCCESS [  0.224 s]
[INFO] onos-gui ........................................... SUCCESS [  1.861 s]
[INFO] onos-rest .......................................... SUCCESS [ 13.279 s]
[INFO] onos-protocols ..................................... SUCCESS [  0.206 s]
[INFO] onos-of ............................................ SUCCESS [  0.180 s]
[INFO] onos-of-api ........................................ SUCCESS [ 14.729 s]
[INFO] onos-of-ctl ........................................ SUCCESS [  2.177 s]
[INFO] onos-netconf ....................................... SUCCESS [  0.165 s]
[INFO] onos-netconf-api ................................... SUCCESS [  0.318 s]
[INFO] onos-netconf-ctl ................................... SUCCESS [  1.571 s]
[INFO] onos-apps .......................................... SUCCESS [  0.176 s]
[INFO] onos-app-pcep-api .................................. SUCCESS [  0.355 s]
[INFO] onos-pcep-controller ............................... SUCCESS [  0.161 s]
[INFO] onos-pcepio ........................................ SUCCESS [  3.892 s]
[INFO] onos-pcep-controller-api ........................... SUCCESS [  0.315 s]
[INFO] onos-pcep-controller-impl .......................... SUCCESS [  0.370 s]
[INFO] onos-ovsdb ......................................... SUCCESS [  0.158 s]
[INFO] onos-ovsdb-rfc ..................................... SUCCESS [  0.789 s]
[INFO] onos-ovsdb-api ..................................... SUCCESS [  0.446 s]
[INFO] onos-ovsdb-ctl ..................................... SUCCESS [  0.356 s]
[INFO] onos-bgp ........................................... SUCCESS [  0.157 s]
[INFO] onos-bgpio ......................................... SUCCESS [  2.713 s]
[INFO] onos-bgp-api ....................................... SUCCESS [  0.312 s]
[INFO] onos-bgp-ctl ....................................... SUCCESS [  1.524 s]
[INFO] onos-restsb ........................................ SUCCESS [  0.166 s]
[INFO] onos-restsb-api .................................... SUCCESS [  0.289 s]
[INFO] onos-restsb-ctl .................................... SUCCESS [  1.547 s]
[INFO] onos-ospf .......................................... SUCCESS [  0.156 s]
[INFO] onos-ospf-api ...................................... SUCCESS [  1.233 s]
[INFO] onos-ospf-protocol ................................. SUCCESS [ 33.014 s]
[INFO] onos-ospf-ctl ...................................... SUCCESS [ 11.064 s]
[INFO] onos-providers ..................................... SUCCESS [  0.166 s]
[INFO] onos-of-providers .................................. SUCCESS [  0.184 s]
[INFO] onos-of-provider-device ............................ SUCCESS [  1.583 s]
[INFO] onos-of-provider-packet ............................ SUCCESS [  1.587 s]
[INFO] onos-of-provider-flow .............................. SUCCESS [  0.871 s]
[INFO] onos-of-provider-group ............................. SUCCESS [  1.655 s]
[INFO] onos-of-provider-meter ............................. SUCCESS [  1.217 s]
[INFO] onos-drivers-general ............................... SUCCESS [  0.174 s]
[INFO] onos-drivers ....................................... SUCCESS [  2.161 s]
[INFO] onos-openflow-base ................................. SUCCESS [  0.825 s]
[INFO] onos-openflow ...................................... SUCCESS [  0.312 s]
[INFO] onos-cpman ......................................... SUCCESS [  0.185 s]
[INFO] onos-app-cpman-api ................................. SUCCESS [  1.155 s]
[INFO] onos-of-provider-message ........................... SUCCESS [  1.202 s]
[INFO] onos-host-provider ................................. SUCCESS [  1.572 s]
[INFO] onos-netcfg-host-provider .......................... SUCCESS [  1.198 s]
[INFO] onos-netconf-providers ............................. SUCCESS [  0.168 s]
[INFO] onos-netconf-provider-device ....................... SUCCESS [  0.401 s]
[INFO] onos-netconf-app ................................... SUCCESS [  0.184 s]
[INFO] onos-null-provider ................................. SUCCESS [  0.491 s]
[INFO] onos-pcep-providers ................................ SUCCESS [  0.160 s]
[INFO] onos-pcep-provider-topology ........................ SUCCESS [  0.345 s]
[INFO] onos-incubator-store ............................... SUCCESS [  0.621 s]
[INFO] onos-incubator-net ................................. SUCCESS [  2.094 s]
[INFO] onos-pcep-provider-tunnel .......................... SUCCESS [  1.500 s]
[INFO] onos-pcep .......................................... SUCCESS [  0.203 s]
[INFO] onos-ovsdb-providers ............................... SUCCESS [  0.161 s]
[INFO] onos-ovsdb-provider-device ......................... SUCCESS [  1.234 s]
[INFO] onos-ovsdb-provider-host ........................... SUCCESS [  1.197 s]
[INFO] onos-ovsdb-provider-tunnel ......................... SUCCESS [  1.229 s]
[INFO] onos-ovsdatabase ................................... SUCCESS [  0.274 s]
[INFO] onos-drivers-ovsdb ................................. SUCCESS [  1.164 s]
[INFO] onos-ovsdb-base .................................... SUCCESS [  0.166 s]
[INFO] onos-bgp-providers ................................. SUCCESS [  0.167 s]
[INFO] onos-bgp-provider-topology ......................... SUCCESS [  1.123 s]
[INFO] onos-bgp-provider-cfg .............................. SUCCESS [  0.327 s]
[INFO] onos-bgp-app ....................................... SUCCESS [  0.193 s]
[INFO] onos-bgp-provider-flow ............................. SUCCESS [  0.316 s]
[INFO] onos-snmp-providers ................................ SUCCESS [ 19.699 s]
[INFO] onos-snmp-provider-device .......................... SUCCESS [  0.768 s]
[INFO] onos-snmp-provider-alarm ........................... SUCCESS [  2.243 s]
[INFO] onos-snmp-app ...................................... SUCCESS [  0.179 s]
[INFO] onos-restsb-providers .............................. SUCCESS [  0.169 s]
[INFO] onos-restsb-provider-device ........................ SUCCESS [  0.370 s]
[INFO] onos-restsb-app .................................... SUCCESS [  0.166 s]
[INFO] onos-lldp-provider-common .......................... SUCCESS [  0.295 s]
[INFO] onos-lldp-provider ................................. SUCCESS [  1.981 s]
[INFO] onos-netcfg-links-provider ......................... SUCCESS [  1.429 s]
[INFO] onos-drivers-utilities ............................. SUCCESS [  1.297 s]
[INFO] onos-drivers-ciena ................................. SUCCESS [  0.313 s]
[INFO] onos-drivers-fujitsu ............................... SUCCESS [  0.293 s]
[INFO] onos-drivers-cisco ................................. SUCCESS [  0.305 s]
[INFO] onos-drivers-netconf ............................... SUCCESS [  0.297 s]
[INFO] onos-drivers-lumentum .............................. SUCCESS [  0.357 s]
[INFO] onos-app-aaa ....................................... SUCCESS [  1.912 s]
[INFO] onos-app-acl ....................................... SUCCESS [  1.261 s]
[INFO] onos-app-fm ........................................ SUCCESS [  0.165 s]
[INFO] onos-app-fm-mgr .................................... SUCCESS [  1.574 s]
[INFO] onos-app-fm-web .................................... SUCCESS [  1.660 s]
[INFO] onos-app-fm-gui .................................... SUCCESS [  0.409 s]
[INFO] onos-app-fm-cli .................................... SUCCESS [  0.326 s]
[INFO] onos-app-fm-onosfw ................................. SUCCESS [  0.179 s]
[INFO] onos-app-fwd ....................................... SUCCESS [  0.387 s]
[INFO] onos-app-mobility .................................. SUCCESS [  0.328 s]
[INFO] onos-app-proxyarp .................................. SUCCESS [  0.307 s]
[INFO] onos-app-routing-api ............................... SUCCESS [  1.519 s]
[INFO] onos-app-routing ................................... SUCCESS [  2.905 s]
[INFO] onos-app-sdnip ..................................... SUCCESS [  1.632 s]
[INFO] onos-app-optical ................................... SUCCESS [  0.365 s]
[INFO] onos-app-metrics ................................... SUCCESS [  0.388 s]
[INFO] onos-app-reactive-routing .......................... SUCCESS [  0.397 s]
[INFO] onos-app-virtualbng ................................ SUCCESS [  0.409 s]
[INFO] onos-app-bgprouter ................................. SUCCESS [  0.343 s]
[INFO] onos-apps-test ..................................... SUCCESS [  0.176 s]
[INFO] onos-app-election .................................. SUCCESS [  0.329 s]
[INFO] onos-app-loadtest .................................. SUCCESS [  0.350 s]
[INFO] onos-app-intent-perf ............................... SUCCESS [  0.432 s]
[INFO] onos-app-messaging-perf ............................ SUCCESS [  0.372 s]
[INFO] onos-app-demo ...................................... SUCCESS [  0.382 s]
[INFO] onos-app-distributed-primitives .................... SUCCESS [  0.414 s]
[INFO] onos-app-segmentrouting ............................ SUCCESS [  2.147 s]
[INFO] onos-app-cordfabric ................................ SUCCESS [  0.395 s]
[INFO] onos-app-xos-integration ........................... SUCCESS [  0.486 s]
[INFO] onos-app-iptopology-api ............................ SUCCESS [  0.451 s]
[INFO] onos-olt ........................................... SUCCESS [  0.170 s]
[INFO] onos-app-olt-api ................................... SUCCESS [  0.290 s]
[INFO] onos-app-olt ....................................... SUCCESS [  0.471 s]
[INFO] onos-app-cip ....................................... SUCCESS [  0.323 s]
[INFO] onos-app-flowanalyzer .............................. SUCCESS [  1.187 s]
[INFO] onos-app-vtn ....................................... SUCCESS [  0.175 s]
[INFO] onos-app-vtn-rsc ................................... SUCCESS [  2.683 s]
[INFO] onos-app-sfc-mgr ................................... SUCCESS [  1.545 s]
[INFO] onos-app-vtn-mgr ................................... SUCCESS [  0.491 s]
[INFO] onos-app-vtn-web ................................... SUCCESS [  5.221 s]
[INFO] onos-app-vtn-onosfw ................................ SUCCESS [  0.180 s]
[INFO] onos-dhcp .......................................... SUCCESS [  0.166 s]
[INFO] onos-app-dhcp-api .................................. SUCCESS [  1.218 s]
[INFO] onos-app-dhcp ...................................... SUCCESS [  1.652 s]
[INFO] onos-app-openstackinterface ........................ SUCCESS [  0.169 s]
[INFO] onos-app-openstackinterface-api .................... SUCCESS [  0.381 s]
[INFO] onos-app-cordvtn ................................... SUCCESS [  0.758 s]
[INFO] onos-app-mfwd ...................................... SUCCESS [  0.347 s]
[INFO] onos-app-igmp ...................................... SUCCESS [  0.464 s]
[INFO] onos-app-pim ....................................... SUCCESS [  0.416 s]
[INFO] onos-app-mlb ....................................... SUCCESS [  0.317 s]
[INFO] onos-app-pp ........................................ SUCCESS [  0.344 s]
[INFO] onos-app-drivermatrix .............................. SUCCESS [  0.322 s]
[INFO] onos-app-cpman ..................................... SUCCESS [  3.492 s]
[INFO] onos-events ........................................ SUCCESS [  0.356 s]
[INFO] onos-app-vrouter ................................... SUCCESS [  0.298 s]
[INFO] onos-app-cord-mcast ................................ SUCCESS [  0.515 s]
[INFO] onos-app-vpls ...................................... SUCCESS [  1.451 s]
[INFO] onos-app-openstacknode ............................. SUCCESS [  0.419 s]
[INFO] onos-app-openstacknetworking ....................... SUCCESS [  0.170 s]
[INFO] onos-app-openstacknetworking-api ................... SUCCESS [  0.288 s]
[INFO] onos-app-openstackswitching ........................ SUCCESS [  0.436 s]
[INFO] onos-app-openstackrouting .......................... SUCCESS [  0.438 s]
[INFO] onos-app-openstackinterface-app .................... SUCCESS [  0.428 s]
[INFO] onos-app-openstacknetworking-web ................... SUCCESS [  0.381 s]
[INFO] onos-app-openstacknetworking-app ................... SUCCESS [  0.170 s]
[INFO] onos-incubator-core ................................ SUCCESS [  0.330 s]
[INFO] onos-incubator-rpc ................................. SUCCESS [  1.169 s]
[INFO] onos-incubator-rpc-grpc ............................ SUCCESS [  7.955 s]
[INFO] onos-features ...................................... SUCCESS [  0.526 s]
[INFO] onos-archetypes .................................... SUCCESS [  0.021 s]
[INFO] onos-api-archetype ................................. SUCCESS [  0.568 s]
[INFO] onos-bundle-archetype .............................. SUCCESS [  0.036 s]
[INFO] onos-cli-archetype ................................. SUCCESS [  0.021 s]
[INFO] onos-rest-archetype ................................ SUCCESS [  0.021 s]
[INFO] onos-ui-archetype .................................. SUCCESS [  0.016 s]
[INFO] onos-uitab-archetype ............................... SUCCESS [  0.016 s]
[INFO] onos-uitopo-archetype .............................. SUCCESS [  0.061 s]
[INFO] onos-branding ...................................... SUCCESS [  0.247 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 04:54 min
[INFO] Finished at: 2016-06-01T15:25:04+08:00
[INFO] Final Memory: 319M/2078M
###############################################################################
以上为i7 CPU 16G内存 PC编译执行时间04:54 min，首次编译的主要时间在于下载依赖，如果采用离线
Maven依赖可以大大加速首次编译时间。
{% endhighlight %}
### 跳过Test编译
{% highlight bash %}
[root@controller onos]# mvn clean install -Dmaven.test.skip=true
###############################################################################
[INFO] --- maven-bundle-plugin:3.0.1:install (default-install) @ onos-branding ---
[INFO] Installing org/onosproject/onos-branding/1.5.1/onos-branding-1.5.1.jar
[INFO] Writing OBR metadata
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] onos ............................................... SUCCESS [  1.203 s]
[INFO] onlab-utils ........................................ SUCCESS [  0.734 s]
[INFO] onlab-junit ........................................ SUCCESS [  1.726 s]
[INFO] onlab-misc ......................................... SUCCESS [  2.678 s]
[INFO] onos-yang-utils-plugin ............................. SUCCESS [  5.413 s]
[INFO] onlab-osgi ......................................... SUCCESS [  0.358 s]
[INFO] onlab-rest ......................................... SUCCESS [  0.409 s]
[INFO] onlab-thirdparty ................................... SUCCESS [  1.912 s]
[INFO] onlab-stc .......................................... SUCCESS [  1.150 s]
[INFO] jdvue .............................................. SUCCESS [  0.792 s]
[INFO] onlab-jnc-osgi ..................................... SUCCESS [  0.331 s]
[INFO] utils.catalyst ..................................... SUCCESS [  0.490 s]
[INFO] onos-core .......................................... SUCCESS [  0.381 s]
[INFO] onos-api ........................................... SUCCESS [  4.782 s]
[INFO] onos-core-common ................................... SUCCESS [  1.290 s]
[INFO] onos-incubator ..................................... SUCCESS [  0.161 s]
[INFO] onos-incubator-api ................................. SUCCESS [  0.839 s]
[INFO] onos-cli ........................................... SUCCESS [  1.350 s]
[INFO] onos-core-store .................................... SUCCESS [  0.176 s]
[INFO] onos-core-serializers .............................. SUCCESS [  0.481 s]
[INFO] onos-core-persistence .............................. SUCCESS [  0.424 s]
[INFO] onos-core-dist ..................................... SUCCESS [  2.685 s]
[INFO] onos-core-net ...................................... SUCCESS [  2.609 s]
[INFO] onos-core-primitives ............................... SUCCESS [  1.873 s]
[INFO] onos-security ...................................... SUCCESS [  0.497 s]
[INFO] onos-web ........................................... SUCCESS [  0.196 s]
[INFO] onos-gui ........................................... SUCCESS [  1.654 s]
[INFO] onos-rest .......................................... SUCCESS [  1.379 s]
[INFO] onos-protocols ..................................... SUCCESS [  0.208 s]
[INFO] onos-of ............................................ SUCCESS [  0.179 s]
[INFO] onos-of-api ........................................ SUCCESS [ 13.858 s]
[INFO] onos-of-ctl ........................................ SUCCESS [  0.980 s]
[INFO] onos-netconf ....................................... SUCCESS [  0.164 s]
[INFO] onos-netconf-api ................................... SUCCESS [  0.306 s]
[INFO] onos-netconf-ctl ................................... SUCCESS [  0.544 s]
[INFO] onos-apps .......................................... SUCCESS [  0.169 s]
[INFO] onos-app-pcep-api .................................. SUCCESS [  0.353 s]
[INFO] onos-pcep-controller ............................... SUCCESS [  0.165 s]
[INFO] onos-pcepio ........................................ SUCCESS [  2.077 s]
[INFO] onos-pcep-controller-api ........................... SUCCESS [  0.301 s]
[INFO] onos-pcep-controller-impl .......................... SUCCESS [  0.441 s]
[INFO] onos-ovsdb ......................................... SUCCESS [  0.184 s]
[INFO] onos-ovsdb-rfc ..................................... SUCCESS [  0.871 s]
[INFO] onos-ovsdb-api ..................................... SUCCESS [  0.461 s]
[INFO] onos-ovsdb-ctl ..................................... SUCCESS [  0.377 s]
[INFO] onos-bgp ........................................... SUCCESS [  0.169 s]
[INFO] onos-bgpio ......................................... SUCCESS [  1.225 s]
[INFO] onos-bgp-api ....................................... SUCCESS [  0.347 s]
[INFO] onos-bgp-ctl ....................................... SUCCESS [  0.604 s]
[INFO] onos-restsb ........................................ SUCCESS [  0.163 s]
[INFO] onos-restsb-api .................................... SUCCESS [  0.272 s]
[INFO] onos-restsb-ctl .................................... SUCCESS [  0.369 s]
[INFO] onos-ospf .......................................... SUCCESS [  0.164 s]
[INFO] onos-ospf-api ...................................... SUCCESS [  0.328 s]
[INFO] onos-ospf-protocol ................................. SUCCESS [  0.726 s]
[INFO] onos-ospf-ctl ...................................... SUCCESS [  0.793 s]
[INFO] onos-providers ..................................... SUCCESS [  0.161 s]
[INFO] onos-of-providers .................................. SUCCESS [  0.166 s]
[INFO] onos-of-provider-device ............................ SUCCESS [  0.530 s]
[INFO] onos-of-provider-packet ............................ SUCCESS [  0.522 s]
[INFO] onos-of-provider-flow .............................. SUCCESS [  0.812 s]
[INFO] onos-of-provider-group ............................. SUCCESS [  0.642 s]
[INFO] onos-of-provider-meter ............................. SUCCESS [  0.469 s]
[INFO] onos-drivers-general ............................... SUCCESS [  0.167 s]
[INFO] onos-drivers ....................................... SUCCESS [  1.142 s]
[INFO] onos-openflow-base ................................. SUCCESS [  0.826 s]
[INFO] onos-openflow ...................................... SUCCESS [  0.315 s]
[INFO] onos-cpman ......................................... SUCCESS [  0.162 s]
[INFO] onos-app-cpman-api ................................. SUCCESS [  0.323 s]
[INFO] onos-of-provider-message ........................... SUCCESS [  0.408 s]
[INFO] onos-host-provider ................................. SUCCESS [  0.378 s]
[INFO] onos-netcfg-host-provider .......................... SUCCESS [  0.334 s]
[INFO] onos-netconf-providers ............................. SUCCESS [  0.163 s]
[INFO] onos-netconf-provider-device ....................... SUCCESS [  0.336 s]
[INFO] onos-netconf-app ................................... SUCCESS [  0.175 s]
[INFO] onos-null-provider ................................. SUCCESS [  0.476 s]
[INFO] onos-pcep-providers ................................ SUCCESS [  0.176 s]
[INFO] onos-pcep-provider-topology ........................ SUCCESS [  0.347 s]
[INFO] onos-incubator-store ............................... SUCCESS [  0.582 s]
[INFO] onos-incubator-net ................................. SUCCESS [  0.587 s]
[INFO] onos-pcep-provider-tunnel .......................... SUCCESS [  0.456 s]
[INFO] onos-pcep .......................................... SUCCESS [  0.181 s]
[INFO] onos-ovsdb-providers ............................... SUCCESS [  0.163 s]
[INFO] onos-ovsdb-provider-device ......................... SUCCESS [  0.319 s]
[INFO] onos-ovsdb-provider-host ........................... SUCCESS [  0.308 s]
[INFO] onos-ovsdb-provider-tunnel ......................... SUCCESS [  0.339 s]
[INFO] onos-ovsdatabase ................................... SUCCESS [  0.266 s]
[INFO] onos-drivers-ovsdb ................................. SUCCESS [  0.342 s]
[INFO] onos-ovsdb-base .................................... SUCCESS [  0.171 s]
[INFO] onos-bgp-providers ................................. SUCCESS [  0.159 s]
[INFO] onos-bgp-provider-topology ......................... SUCCESS [  0.302 s]
[INFO] onos-bgp-provider-cfg .............................. SUCCESS [  0.340 s]
[INFO] onos-bgp-app ....................................... SUCCESS [  0.179 s]
[INFO] onos-bgp-provider-flow ............................. SUCCESS [  0.334 s]
[INFO] onos-snmp-providers ................................ SUCCESS [  0.172 s]
[INFO] onos-snmp-provider-device .......................... SUCCESS [  0.549 s]
[INFO] onos-snmp-provider-alarm ........................... SUCCESS [  0.533 s]
[INFO] onos-snmp-app ...................................... SUCCESS [  0.164 s]
[INFO] onos-restsb-providers .............................. SUCCESS [  0.157 s]
[INFO] onos-restsb-provider-device ........................ SUCCESS [  0.371 s]
[INFO] onos-restsb-app .................................... SUCCESS [  0.167 s]
[INFO] onos-lldp-provider-common .......................... SUCCESS [  0.300 s]
[INFO] onos-lldp-provider ................................. SUCCESS [  0.485 s]
[INFO] onos-netcfg-links-provider ......................... SUCCESS [  0.365 s]
[INFO] onos-drivers-utilities ............................. SUCCESS [  0.293 s]
[INFO] onos-drivers-ciena ................................. SUCCESS [  0.289 s]
[INFO] onos-drivers-fujitsu ............................... SUCCESS [  0.335 s]
[INFO] onos-drivers-cisco ................................. SUCCESS [  0.294 s]
[INFO] onos-drivers-netconf ............................... SUCCESS [  0.325 s]
[INFO] onos-drivers-lumentum .............................. SUCCESS [  0.363 s]
[INFO] onos-app-aaa ....................................... SUCCESS [  0.392 s]
[INFO] onos-app-acl ....................................... SUCCESS [  0.400 s]
[INFO] onos-app-fm ........................................ SUCCESS [  0.159 s]
[INFO] onos-app-fm-mgr .................................... SUCCESS [  0.458 s]
[INFO] onos-app-fm-web .................................... SUCCESS [  0.350 s]
[INFO] onos-app-fm-gui .................................... SUCCESS [  0.358 s]
[INFO] onos-app-fm-cli .................................... SUCCESS [  0.308 s]
[INFO] onos-app-fm-onosfw ................................. SUCCESS [  0.172 s]
[INFO] onos-app-fwd ....................................... SUCCESS [  0.344 s]
[INFO] onos-app-mobility .................................. SUCCESS [  0.288 s]
[INFO] onos-app-proxyarp .................................. SUCCESS [  0.316 s]
[INFO] onos-app-routing-api ............................... SUCCESS [  0.349 s]
[INFO] onos-app-routing ................................... SUCCESS [  0.985 s]
[INFO] onos-app-sdnip ..................................... SUCCESS [  0.500 s]
[INFO] onos-app-optical ................................... SUCCESS [  0.361 s]
[INFO] onos-app-metrics ................................... SUCCESS [  0.383 s]
[INFO] onos-app-reactive-routing .......................... SUCCESS [  0.348 s]
[INFO] onos-app-virtualbng ................................ SUCCESS [  0.464 s]
[INFO] onos-app-bgprouter ................................. SUCCESS [  0.346 s]
[INFO] onos-apps-test ..................................... SUCCESS [  0.162 s]
[INFO] onos-app-election .................................. SUCCESS [  0.322 s]
[INFO] onos-app-loadtest .................................. SUCCESS [  0.304 s]
[INFO] onos-app-intent-perf ............................... SUCCESS [  0.435 s]
[INFO] onos-app-messaging-perf ............................ SUCCESS [  0.391 s]
[INFO] onos-app-demo ...................................... SUCCESS [  0.452 s]
[INFO] onos-app-distributed-primitives .................... SUCCESS [  0.473 s]
[INFO] onos-app-segmentrouting ............................ SUCCESS [  0.850 s]
[INFO] onos-app-cordfabric ................................ SUCCESS [  0.409 s]
[INFO] onos-app-xos-integration ........................... SUCCESS [  0.389 s]
[INFO] onos-app-iptopology-api ............................ SUCCESS [  0.464 s]
[INFO] onos-olt ........................................... SUCCESS [  0.166 s]
[INFO] onos-app-olt-api ................................... SUCCESS [  0.297 s]
[INFO] onos-app-olt ....................................... SUCCESS [  0.435 s]
[INFO] onos-app-cip ....................................... SUCCESS [  0.297 s]
[INFO] onos-app-flowanalyzer .............................. SUCCESS [  0.390 s]
[INFO] onos-app-vtn ....................................... SUCCESS [  0.163 s]
[INFO] onos-app-vtn-rsc ................................... SUCCESS [  1.222 s]
[INFO] onos-app-sfc-mgr ................................... SUCCESS [  0.408 s]
[INFO] onos-app-vtn-mgr ................................... SUCCESS [  0.495 s]
[INFO] onos-app-vtn-web ................................... SUCCESS [  0.722 s]
[INFO] onos-app-vtn-onosfw ................................ SUCCESS [  0.182 s]
[INFO] onos-dhcp .......................................... SUCCESS [  0.162 s]
[INFO] onos-app-dhcp-api .................................. SUCCESS [  0.286 s]
[INFO] onos-app-dhcp ...................................... SUCCESS [  0.543 s]
[INFO] onos-app-openstackinterface ........................ SUCCESS [  0.182 s]
[INFO] onos-app-openstackinterface-api .................... SUCCESS [  0.370 s]
[INFO] onos-app-cordvtn ................................... SUCCESS [  0.720 s]
[INFO] onos-app-mfwd ...................................... SUCCESS [  0.325 s]
[INFO] onos-app-igmp ...................................... SUCCESS [  0.369 s]
[INFO] onos-app-pim ....................................... SUCCESS [  0.413 s]
[INFO] onos-app-mlb ....................................... SUCCESS [  0.298 s]
[INFO] onos-app-pp ........................................ SUCCESS [  0.335 s]
[INFO] onos-app-drivermatrix .............................. SUCCESS [  0.298 s]
[INFO] onos-app-cpman ..................................... SUCCESS [  0.655 s]
[INFO] onos-events ........................................ SUCCESS [  0.353 s]
[INFO] onos-app-vrouter ................................... SUCCESS [  0.292 s]
[INFO] onos-app-cord-mcast ................................ SUCCESS [  0.363 s]
[INFO] onos-app-vpls ...................................... SUCCESS [  0.369 s]
[INFO] onos-app-openstacknode ............................. SUCCESS [  0.379 s]
[INFO] onos-app-openstacknetworking ....................... SUCCESS [  0.163 s]
[INFO] onos-app-openstacknetworking-api ................... SUCCESS [  0.277 s]
[INFO] onos-app-openstackswitching ........................ SUCCESS [  0.424 s]
[INFO] onos-app-openstackrouting .......................... SUCCESS [  0.461 s]
[INFO] onos-app-openstackinterface-app .................... SUCCESS [  0.386 s]
[INFO] onos-app-openstacknetworking-web ................... SUCCESS [  0.381 s]
[INFO] onos-app-openstacknetworking-app ................... SUCCESS [  0.169 s]
[INFO] onos-incubator-core ................................ SUCCESS [  0.292 s]
[INFO] onos-incubator-rpc ................................. SUCCESS [  0.336 s]
[INFO] onos-incubator-rpc-grpc ............................ SUCCESS [  3.581 s]
[INFO] onos-features ...................................... SUCCESS [  0.211 s]
[INFO] onos-archetypes .................................... SUCCESS [  0.021 s]
[INFO] onos-api-archetype ................................. SUCCESS [  0.187 s]
[INFO] onos-bundle-archetype .............................. SUCCESS [  0.008 s]
[INFO] onos-cli-archetype ................................. SUCCESS [  0.006 s]
[INFO] onos-rest-archetype ................................ SUCCESS [  0.007 s]
[INFO] onos-ui-archetype .................................. SUCCESS [  0.009 s]
[INFO] onos-uitab-archetype ............................... SUCCESS [  0.008 s]
[INFO] onos-uitopo-archetype .............................. SUCCESS [  0.010 s]
[INFO] onos-branding ...................................... SUCCESS [  0.241 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:55 min
[INFO] Finished at: 2016-06-01T15:30:25+08:00
[INFO] Final Memory: 286M/2077M
[INFO] ------------------------------------------------------------------------
###############################################################################
如果跳过测试，只编译原始代码，耗时01:55 min。
{% endhighlight %}

##### 目前只是把ONOS编译并且安装到Maven的本地库
{%  highlight bash %}
######mvn install 将目标安装到本地库
[root@controller onos]# pwd
/maven_repo/org/onosproject/onos
{% endhighlight %}

### 配置Karaf启动onos
{%  highlight bash %}
[root@controller etc]# pwd
/usr/local/share/apache-karaf-3.0.5/etc
###############################################################################
#
featuresRepositories=mvn:org.apache.karaf.features/standard/3.0.5/xml/features,mvn:org.apache.karaf.features/enterprise/3.0.5/xml/features,mvn:org.ops4j.pax.web/pax-web-features/3.2.6/xml/features,mvn:org.apache.karaf.features/spring/3.0.5/xml/features,mvn:org.onosproject/onos-features/1.5.1/xml/features

#
# Comma separated list of features to install at startup
#
featuresBoot=config,standard,region,package,kar,ssh,management,webconsole,onos-api,onos-core,onos-incubator,onos-cli,onos-rest,onos-gui
###############################################################################
{% endhighlight %}

- featuresRepositories  &nbsp;&nbsp;新增mvn:org.onosproject/onos-features/1.5.1/xml/features
- featuresBoot	  &nbsp;&nbsp;新增onos-api,onos-core,onos-incubator,onos-cli,onos-rest,onos-gui
{% include JB/setup %}

### 配置用户
{% highlight bash %}
[root@controller etc]# vim /usr/local/share/apache-karaf-3.0.5/etc/users.properties
################################################################################
# USER=PASSWORD,ROLE1,ROLE2,...
# with the name "karaf".
#
karaf = karaf,_g_:admingroup
onos = rocks,_g_:admingroup
_g_\:admingroup = group,admin,manager,viewer,webconsole
{% endhighlight %}

### 启动ONOS
{% highlight bash %}
[root@controller ~]# karaf clean
Welcome to Open Network Operating System (ONOS)!
     ____  _  ______  ____     
    / __ \/ |/ / __ \/ __/   
   / /_/ /    / /_/ /\ \     
   \____/_/|_/\____/___/     
                               
Documentation: wiki.onosproject.org      
Tutorials:     tutorials.onosproject.org 
Mailing lists: lists.onosproject.org     

Come help out! Find out how at: contribute.onosproject.org 

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown ONOS.

onos> 
onos> list
START LEVEL 100 , List Threshold: 50
 ID | State  | Lvl | Version          | Name                                 
-----------------------------------------------------------------------------
 40 | Active |  80 | 2.6              | Commons Lang                         
 41 | Active |  80 | 3.4.0            | Apache Commons Lang                  
 42 | Active |  80 | 1.10.0           | Apache Commons Configuration         
 43 | Active |  80 | 19.0.0           | Guava: Google Core Libraries for Java
 44 | Active |  80 | 3.10.5.Final     | Netty                                
 45 | Active |  80 | 4.0.33.Final     | Netty/Common                         
 46 | Active |  80 | 4.0.33.Final     | Netty/Buffer                         
 47 | Active |  80 | 4.0.33.Final     | Netty/Transport                      
 48 | Active |  80 | 4.0.33.Final     | Netty/Handler                        
 49 | Active |  80 | 4.0.33.Final     | Netty/Codec                          
 50 | Active |  80 | 4.0.33.Final     | Netty/Transport/Native/Epoll         
 51 | Active |  80 | 1.6.0            | Commons Pool                         
 52 | Active |  80 | 3.2.0            | Commons Math                         
 53 | Active |  80 | 2.9              | Joda-Time                            
 54 | Active |  80 | 3.1.0            | Metrics Core                         
 55 | Active |  80 | 3.1.0            | Jackson Integration for Metrics      
 56 | Active |  80 | 0.9.1            | minimal-json                         
 57 | Active |  80 | 3.0.0            | Kryo                                 
 58 | Active |  80 | 1.11.0           | ReflectASM                           
 59 | Active |  80 | 4.2              | ASM                                  
 60 | Active |  80 | 1.3.0            | MinLog                               
 61 | Active |  80 | 2.1.0            | Objenesis                            
 62 | Active |  80 | 2.7.0            | Jackson-core                         
 63 | Active |  80 | 2.7.0            | Jackson-annotations                  
 64 | Active |  80 | 2.7.0            | jackson-databind                     
 65 | Active |  80 | 3.2.1            | Commons Collections                  
 66 | Active |  80 | 1.2.1            | com.typesafe.config                  
 67 | Active |  80 | 1.5.1            | onlab-thirdparty                     
 68 | Active |  80 | 1.19.0           | jersey-client                        
 69 | Active |  80 | 1.0.7            | mapdb                                
 70 | Active |  80 | 1.5.1            | onlab-misc                           
 71 | Active |  80 | 1.5.1            | onlab-osgi                           
 72 | Active |  80 | 1.5.1            | onos-api                             
 73 | Active |  80 | 1.5.1            | onos-incubator-api                   
 74 | Active |  80 | 1.5.1            | onos-core-net                        
 75 | Active |  80 | 1.5.1            | onos-core-common                     
 76 | Active |  80 | 1.5.1            | onos-core-dist                       
 77 | Active |  80 | 1.5.1            | onos-core-primitives                 
 78 | Active |  80 | 1.5.1            | onos-core-persistence                
 79 | Active |  80 | 1.5.1            | onos-core-serializers                
 80 | Active |  80 | 1.5.1            | onos-incubator-net                   
 81 | Active |  80 | 1.5.1            | onos-incubator-core                  
 82 | Active |  80 | 1.5.1            | onos-incubator-store                 
 83 | Active |  80 | 1.5.1            | onos-incubator-rpc                   
 94 | Active |  80 | 1.5.1            | onos-cli                             
127 | Active |  80 | 1.19.0           | jersey-core                          
128 | Active |  80 | 1.19.0           | jersey-server                        
129 | Active |  80 | 1.19.0           | jersey-servlet                       
130 | Active |  80 | 1.19.0           | jersey-multipart                     
131 | Active |  80 | 1.9.3            | MIME streaming extension             
132 | Active |  80 | 1.1.1            | jsr311-api                           
133 | Active |  80 | 1.5.1            | onlab-rest                           
134 | Active |  80 | 1.5.1            | onos-rest                            
153 | Active |  80 | 8.1.18.v20150929 | Jetty :: Websocket                   
154 | Active |  80 | 1.5.1            | onos-gui                             
162 | Active |  80 | 1.5.1            | onos-drivers                         
163 | Active |  80 | 1.5.1            | onos-of-api                          
164 | Active |  80 | 3.9.2.Final      | The Netty Project                    
165 | Active |  80 | 1.5.1            | onos-of-ctl                          
166 | Active |  80 | 1.5.1            | onos-of-provider-device              
167 | Active |  80 | 1.5.1            | onos-of-provider-packet              
168 | Active |  80 | 1.5.1            | onos-of-provider-flow                
169 | Active |  80 | 1.5.1            | onos-of-provider-group               
170 | Active |  80 | 1.5.1            | onos-of-provider-meter               
171 | Active |  80 | 1.5.1            | onos-host-provider                   
172 | Active |  80 | 1.5.1            | onos-lldp-provider                   
173 | Active |  80 | 1.5.1            | onos-lldp-provider-common            
174 | Active |  80 | 1.5.1            | onos-openflow                        
175 | Active |  80 | 1.5.1            | onos-app-fwd                         
176 | Active |  80 | 1.5.1            | onos-app-mobility                    
177 | Active |  80 | 1.5.1            | onos-app-proxyarp
{% endhighlight %}

### 访问Web控制台
http://192.168.100.142:8181/onos/ui/login.html <br/>
用户名密码均为karaf <br/>
或者用户onos 密码rocks 参见配置用户

