---
layout: post
title: "Centos7安装nodejs"
description: ""
category: Linux 
tags: [nodejs]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

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

### 使用iView-project
{% highlight bash %}
#按照上述安装npm webpack vue
$ git clone https://github.com/iview/iview-project.git

$ vim webpack.dev.config.js
#在module.exports 内新增devServer信息【webpack知识】
module.exports = merge(webpackBaseConfig, {
    devtool: '#source-map',
    output: {
        publicPath: '/dist/',
        filename: '[name].js',
        chunkFilename: '[name].chunk.js'
    },
    devServer: {
        publicPath: '/dist/',
        host: '0.0.0.0',
        port: 8080
    },
    plugins: [
        new ExtractTextPlugin({
            filename: '[name].css',
            allChunks: true
        }),
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendors',
            filename: 'vendors.js'
        }),
        new HtmlWebpackPlugin({
            filename: '../index.html',
            template: './src/template/index.ejs',
            inject: false
        })
    ]
});

#如果Centos7注意开放防火墙

#安装依赖,会生成node_modules
$ npm install 

#开发
$ npm run init
$ npm run dev

#生产打包
$ npm run build 
{% endhighlight %}

### 部署
webpack.prod.config.js指定了打包后的输出目录，该目录即为前段的所有静态文件，index_prod.html作为后台返回的主入口

### webpack代理
{% highlight bash %}
#0.0.0.0:8080/api/* 重定向到 172.18.1.27:8080/*
 devServer: {
        publicPath: '/dist/',
        host: '0.0.0.0',
        port: 8080,
        proxy: {
          '/api': {
            target: {
              host: "172.18.1.27",
              protocol: 'http:',
              port: 8080
            },
            pathRewrite: {
                '^/api': ''
            }
          }
        }
    },
{% endhighlight %}

### FAQ
{% highlight bash %}
$Component template should contain exactly one root element
#<template></template>内只能包含一个根元素

Right: <template><div><button>xxx</button></div></template>
Wrong: <template><div></div><button>xxx</button></template>
#上述中错误示例里 div 和 button并列
{% endhighlight %}
{% include JB/setup %}


