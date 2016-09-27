---
layout: post
title: "Keystone知识梳理(二)"
description: ""
category: Openstack
tags: [Openstack, Keystone]
---
{% include JB/setup %}

## 梳理Openstack组件Keystone知识

### Paste deploy了解

>
- Paste Deployment is a system for finding and configuring WSGI applications and servers.
- 对于WSGI应用消费者它提供单实例简单的loadapp从.ini配置或者Python Egg中载入WSGI应用
- 对于WSGI应用提供者只询问应用的切入点

### 典型配置
{%highlight bash%}
[composite:main]
use = egg:Paste#urlmap
/ = home
/blog = blog
/wiki = wiki
/cms = config:cms.ini

[app:home]
use = egg:Paste#static
document_root = %(here)s/htdocs

[filter-app:blog]
use = egg:Authentication#auth
next = blogapp
roles = admin
htpasswd = /home/me/users.htpasswd

[app:blogapp]
use = egg:BlogApp
database = sqlite:/home/me/blog.db

[app:wiki]
use = call:mywiki.main:application
database = sqlite:/home/me/wiki.db
{%endhighlight%} 

>
- composite 分发请求到其它应用,use = egg:Paste#urlmap 使用包Paste包内的应用urlmap,composite section是wsgi application的集合
- app 定义了符合wsgi协议的应用
- filter 定义过滤器，此处并未应用
- pipeline 将一系列的filter作用到app上，
- filter-app 让application使用某个filter
- 基本用途载入WSGI应用
{%highlight bash%}
from paste.deploy import loadapp
wsgi_app = loadapp('config:/path/to/config.ini')
{%endhighlight%}

>
- composite 第一层调度者，根据不同的url请求分配不同的pipeline，每个pipeline由若干filter和一个application组成，
application位于pipeline最后并且只有一个
- filter 按照pipeline中filter先后顺序对http请求进行过滤，如参数格式化处理等操作
- 最终请求到达application每个模块的路由层，路由分发给业务的controllers

***

### Mitaka Keystone分析 
- 命令脚本 keystone-wsgi-admin 里面调用keystone.server.wsgi import initialize_admin_application
- keystone.server.wsgi中方法initialize_application，调用keystone.version/service.pyzhong 中loadapp
- loadapp中利用paste.deploy启动app

***

### setup.py和setup.cfg
- 两者为控制python项目打包和安装的工具类
- 书写setup.py,执行python setup.py sdist打包
- 通过MANIFEST模板MANIFEST.in定义分发包中需要包含的文件清单
- setup.cfg 提供setup.py的默认参数，setup.py 先解析setup.cfg然后执行命令
- PBR 是setuptools的辅助工具，读取过滤setup.cfg中的内容,将解析结果传递给setup.py作为参数

>
>参考链接
>
- http://pythonpaste.org/deploy/

