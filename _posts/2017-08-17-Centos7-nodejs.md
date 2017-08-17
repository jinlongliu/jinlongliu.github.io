---
layout: post
title: "Centos7安装nodejs"
description: ""
category: Linux 
tags: [nodejs]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### EPEL【可选】
<br/>

{% highlight bash %}
http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 

$yum repolist
{% endhighlight %}

### 软件源
{% highlight bash %}
$curl https://rpm.nodesource.com/setup_8.x |bash -
$yum repolist
#新增nodesource/x86_64    Node.js Packages for Enterprise Linux 7 - x86_64

#检查安装
[root@centos7 ~]# node -v
v6.11.2
[root@centos7 ~]# npm -v
3.10.10
{% endhighlight %}

### 淘宝NPM镜像 
{% highlight bash %}
#临时
$ npm --registry https://registry.npm.taobao.org install express

#持久使用
$ npm config set registry https://registry.npm.taobao.org

#配置后可通过下面方式来验证是否成功
$ npm config get registry
#或
$ npm info express
{% endhighlight %}

### 安装VUE
{% highlight bash %}
$ npm install vue

# 全局安装 vue-cli
$ npm install --global vue-cli
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
# 安装依赖，走你
$ cd my-project
$ npm install
$ npm run dev
{% endhighlight %}

{% include JB/setup %}


