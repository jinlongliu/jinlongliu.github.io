---
layout: post
title: "Python学习笔记（六）"
description: "Python学习笔记"
category: Python
tags: [Python]
---
{% include JB/setup %}

## Python 其它

>
- Eggs are to Pythons as Jars are to Java...
- 工程打包文件
- 普通目录添加__init__.py即变成python package

>
- 一个字典，从entry point组名映射道一个表示entry point的字符串或字符串列表。Entry points是用来支持动态发现服务和插件的，也用来支持自动生成脚本。 
- setup.py 打包示例
{%highlight bash%}
demo
|-- build
|   `-- bdist.linux-x86_64
|-- demo.egg-info
|   |-- dependency_links.txt
|   |-- PKG-INFO
|   |-- SOURCES.txt
|   `-- top_level.txt
|-- dist
|   `-- demo-0.1-py2.7.egg
`-- setup.py
{%endhighlight%}

### WebOb
WSGI request and response objects

>
- 提供WSGI请求环境封装，帮助生成WSGI响应体的Python库
- 包含大量HTTP特定行为：头解析、内容协商、验证处理条件和请求范围

>
- webob.client – Send WSGI requests over HTTP
- webob.cookies – Cookies
- webob.dec – WSGIfy decorator
- webob.exc – WebOb Exceptions
- webob.multidict – multi-value dictionary object
- webob.request – Request
- webob.response – Response
- webob.static – Serving static files
- webob – Request/Response objects

### PBD调试

>
- pbd python自带调试包,代码入口处pbd.set_trace()
- break 或 b 设置断点   设置断点
- continue 或 c   继续执行程序
- list 或 l   查看当前行的代码段
- step 或 s   进入函数
- return 或 r 执行代码直到从当前函数返回
- exit 或 q   中止并退出
- next 或 n   执行下一行
- pp  打印变量的值,也可以用单个p
- help    帮助

### Pycharm远程调试

>
- Client/Server模式，Client为远端执行环境，Server端为本地调试环境，均需安装easy_install
- easy_install安装方法，下载安装脚本https://bootstrap.pypa.io/ez_setup.py，然后python ez_setup.py执行，Linux，Windows均适用
- Client,Server端安装pycharm-deubg.egg,在PyCharm安装目录debug-eggs中可以找到，执行easy_install pycharm-debug.egg,dos 上easy_install.exe pycharm-debug.egg
- Server开启Debug，Edit Configurations>Run/Debug Configurations Dialog>Add "Python Remote Debug",设置监听本地端口，客户端会连接。开启Debug模式
- 修改客户端代码,执行代码会触发Server端代码
{%highlight python%}
#IP，Port为上步骤设置值

import pydevd
pydevd.settrace("192.168.40.146",port=8877)
{%endhighlight%}
- 服务器端，客户端代码保持值，调试按照代码行进行 

>
>参考链接：
>
- http://www.webob.org/
- https://docs.python.org/2/library/pdb.html
- https://www.jetbrains.com/help/pycharm/2016.1/creating-and-editing-run-debug-configurations.html

