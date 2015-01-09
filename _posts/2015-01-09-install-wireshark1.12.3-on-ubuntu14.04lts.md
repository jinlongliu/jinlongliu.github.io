---
layout: post
title: "Ubuntu 14.04 64bits安装WireShark1.12.3"
description: ""
category: "SDN"
tags: ["SDN", "ONOS"]
---
{% include JB/setup %}
<p>
方法一：[笔者未验证]
</p>
{% highlight bash %}
$ sudo add-apt-repository ppa:n-muench/programs-ppa
$ sudo apt-get update
$ sudo apt-get install wireshark
{% endhighlight %}
<p>
方法二：源码安装 [也要下载一堆软件包，如果公司未配置本地更新源，建议采用方法一]<br/>
官方下载: https://www.wireshark.org/download.html<br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1bnbDnWj
</p>
{% highlight bash %}
tar -xvf wireshark-1.12.3.tar.bz2
cd wireshark-1.12.3
apt-get -y install build-essential   \
                        libgtk2.0-dev     \
                        libpcap0.8-dev    \
                        flex              \
                        dpatch            \
                        libc-ares-dev     \
                        xsltproc          \
                        docbook-xsl       \
                        libcap-dev        \
                        bison             \
                        libgnutls-dev     \
                        portaudio19-dev   \
                        libkrb5-dev       \
                        libsmi2-dev       \
                        libgeoip-dev      \
                        imagemagick       \
                        xdg-utils         \
                        libqt4-dev        \
                        libgtk-3-dev      \
                        python-support    \
                        python-ply        \
                        quilt             \
                        liblua5.2-dev     \
                        libnl-genl-3-dev  \
                        libnl-route-3-dev \
                        asciidoc          \
                        cmake
./configure
make
make install
{% endhighlight %}
<p>
上述操作为服务器端操作，如果是桌面版直接运行，应该可以看到图像。 <br/>
自己电脑本地安装Xming <br/>
ONOS技术群[425319659]共享中心：http://pan.baidu.com/s/1qW2ref6 <br/>
设置SSH连接工具转发X11，开启Xming，服务器端运行wireshark,程序界面就会重定向到自己电脑
</p>
<p>拓扑结构</p>
<img src="/upload/2015/1/topo1.png"/>
<p>SSH CRT转发X11设置</p>
<img src="/upload/2015/1/sshcrt-setting.png"/>
<p>在服务器端启动控制器，并用mininet建立拓扑连接控制器,WireShark过滤框填写“openflow_v4”，点击Apply，<br/>
监听设备接口笔者选择的是lo,因为笔者服务器端使用的是控制器交换机同一机器，流量走内部。<br/>
从截图可知从交换机和控制器交互的第一个HELLO数据包已经被抓取了。</p>
<img src="/upload/2015/1/wireshark-menu.png"/>
