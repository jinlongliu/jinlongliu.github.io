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

