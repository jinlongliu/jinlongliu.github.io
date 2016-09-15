---
layout: post
title: "Docker知识梳理（四）"
description: ""
category: Docker 
tags: [Docker, Linux]
---
{% include JB/setup %}
## 容器的网络
- docker run -P ... 当-P标记时，随机映射端口至容器开放的网络端口
- docker run -p 5000:5000 
- docker run -p 127.0.0.1:5000:5000 标记-p，映射指定端口
- docker run -p 127.0.0.1::5000 指定地址的任意端口
- docker port container_uuid 查看端口映射情况
- 容器的名称是唯一的，可以在docker run --name NAME 来指定
- docker run 添加--rm 标记，容器终止后立即删除，不可以与-d同时使用

### 容器间通信
{%highlight bash%}
#--link 实现互联, 实现机制：环境变量、更新/etc/hosts文件
$docker run -d --name db postgres
$docker run -d -P --name web --link db:db webapp python app.py

[root@king ~]# docker exec web env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=fb2308fbca60
DB_PORT=tcp://172.17.0.3:5432
DB_PORT_5432_TCP=tcp://172.17.0.3:5432
DB_PORT_5432_TCP_ADDR=172.17.0.3
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_PROTO=tcp
DB_NAME=/web/db
DB_ENV_PG_VERSION=9.3
HOME=/root

[root@king ~]# docker exec db env 
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=c39183d87bd7
HOME=/
PG_VERSION=9.3
{%endhighlight%}

## 基于Dockerfile创建镜像
Dockerfile组成部分
- 基础镜像信息
- 维护者信息
- 镜像操作命令
- 容器启动时执行指令

Dockerfile示例
{%highlight bash%}
#This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: Liu Jinlong
# Command format: Instruction [arguments / command]

#基础镜像
FROM ubuntu

#维护者信息
MAINTAINER Liu_Jinlong nnuljl@gmail.com

#镜像操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

#容器启动时执行命令
CMD /usr/sbin/nginx
{%endhighlight%}

创建镜像
{%highlight bash%}
$docker build -t build_repo/first_image /tmp/docker_builder/
#自动搜索/tmp/docker_builder/目录下文件Dockerfile
#build_repo/first_image 为创建镜像的标签信息
{%endhighlight%}

### Dockerfile指令
一般格式 INSTRUCTION arguments
{%highlight bash%}
FROM 
#FROM <image> or FROM <image>:<tag>
#创建多个镜像可以使用多次


MAINTAINER
#MAINTAINER <name> 维护者信息

RUN
#RUN <command> 后者 RUN ["executable", "param1", "param2"]
#前者在/bin/sh -c运行，后者使用 exec 执行

CMD
#CMD ["executable", "param1", "param2"]
#CMD command param1 param2  在/bin/sh中执行
#CMD ["param1", "param2"] 提供给ENTRYPOINT的默认参数
#启动时只会执行一条，多条最后一条执行，会被用户指定命令覆盖

EXPOSE
#EXPOSE <port> [<port>...]
#暴露的端口，-P会映射到该项

ENV
#ENV <key> <value>
#被RUN使用，运行时保持

ADD
#ADD <src> <dest>
#复制src到容器dest，src为Dockerfile的相对路径（文件或目录），URL

COPY
#COPY <src> <dest>
#复制本主机的src到dest，区别于ADD，COPY不支持URL，所以本地目录推荐COPY

ENTRYPOINT
#ENTRYPOINT ["executable", "param1", "param2"]
#ENTRYPOINT command param1 param2 (shell中执行)
#容器启动后执行的命令，不能被docker run中参数覆盖，只有一个，多个执行最后一个

VOLUME
#VOLUME ["/data"]
#创建基于本地或者其它容器的挂载点

USER
#USER daemon
#指定容器运行时用户，RUN执行用户

WORKDIR
#WORKDIR /path/to/workdir
#为RUN、CMD、ENTRYPOINT指定配置工作目录，多条指定，后续命令参数为相对路径则会基于之前指定的路径

ONBUILD
#ONBUILD [INSTRUCTION]
#配置当前镜像作为其它镜像的基础镜像时，执行的操作指令

{%endhighlight%}


