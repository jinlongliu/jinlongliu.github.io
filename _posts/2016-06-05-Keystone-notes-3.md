---
layout: post
title: "Keystone知识梳理(三)"
description: ""
category: Openstack
tags: [Openstack, Keystone]
---
{% include JB/setup %}

## 梳理Openstack组件Keystone知识-PBR

### PBR是什么
- pbr Python Build Reasonableness
- setuptools 插件，使用pbr必须使用setuptools
- 读取过滤setup.cfg中配置信息,作为参数传递给setup.py,打包的核心工作还是由setuptools处理

PBR提供功能：

>
- Version 版本,基于Git的Revisions修订和tags
- AUTHORS 作者，从Git log中获取
- ChangeLog 变更日志，从Git log中获取
- Manifest 清单， 从git files获取
- Sphinx Autodoc
- Requirements 存储依赖以pip要求格式
- long_description 使用README作为详细描述
- Smart find_packages 智能发现root包下的包
 
>
- 版本管理两种方式，postversioning和preversioning
- postversioning 默认，以git计算方式获取版本
- preversioning 通过setup.cfg中metadata章节中version使能
- 如果当前checkout为tagged则作为版本，如果不是，会获取最新的版本号然后做最小增加之后作为版本
- 如果preversioning使能，获取配置文件中的版本，如果它小于计算出来的版本报错，否则就作为目标版本
- PBR期望使用git tags来计算版本

>
- Mainfest 所有文件清档存放于git管理
- PBR处理pip要求格式的文件，注入到install_requires,tests_require 和dependency_links
- 将README.rst, README.txt 和 README注入到long_description

### 使用

>
- pbr和setuptools捆绑使用，配置存储于静态文件，最简setup.py如下：
- pbr=True 需要显示声明，启用pbr
{%highlight python%}
#!/usr/bin/env python

from setuptools import setup

setup(
    setup_requires=['pbr>=1.9', 'setuptools>=17.1'],
    pbr=True,
)
{%endhighlight%}

### setup.cfg讲解

组成：

- metadata
- files
- entry_points
- pbr

>
- files章节定义了文件的安装位置，分别使用packages,namespace_packages,data_files
- packages 定义了需要安装的包，类似setuptools.find_packages,如果未指定默认值为metadata中的name
- data_files 列出了要被安装的文件，缩进块格式的key-value对，左边目标目录，右边是安装文件，多个文件
安装同一个目录需要缩进,支持通配符语法安装整个目录如果etc/pbr/*
- entry_points 定义了生成控制台脚本和python库的切入点，顶格的切入点组合名称以及缩进的将本安装的切入点
- python site-packages中pbr.cmd中main.py调用安装entry_points中条目
- pbr章节控制pbr特定的选项和行为,warnerrors将warn作为error
- setup.cfg中注释必须以#开头，同时后面跟一个空格
{%highlight bash%}
[section]
# A comment at the start of a dedicated line
key =
    value1 # An in line comment
    value2
    # A comment on a dedicated line
    value3
{%endhighlight%}

>
- requirements-pyN.txt 
- tools/pip-requires-py3
- requirements.txt
- tools/pip-requires
- 上述中N为python的主版本号Python 2.7 则N为2
- requirements-pyN.txt已弃用，requirements.txt通用

>
>参考链接
>
- http://docs.openstack.org/developer/pbr/

