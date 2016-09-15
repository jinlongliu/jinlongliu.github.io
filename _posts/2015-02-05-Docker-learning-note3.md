---
layout: post
title: "Docker知识梳理（三）"
description: ""
category: Docker 
tags: [Docker, Linux]
---
{% include JB/setup %}
## 容器仓库
- 仓库集中存放镜像的地方
- 存放仓库的服务器叫做注册服务器，一个注册服务器可以存放多个仓库

### 利用registry镜像创建私有仓库
{%highlight bash%}
$docker pull registry

[root@king ~]# docker run -d -p 5000:5000 registry
0840ebd20aeab96c0569ff9082324b608efd4d431d0452396dc0e00e87194006
[root@king ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
0840ebd20aea        registry            "/entrypoint.sh /etc/"   4 seconds ago       Up 3 seconds        0.0.0.0:5000->5000/tcp   clever_shockley

#通过-v将镜像文件存放指定本地路径  -v  LOCAL_PATH:PATH_IN_CONTAINER
#Registry V2 默认路径为/var/lib/registry
$docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry  registry

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
registry            latest              0bb8b1006103        2 days ago          33.27 MB
ubuntu              latest              bd3d4369aebc        2 weeks ago         126.6 MB
#上传镜像
$docker tag ubuntu  king:5000/ubuntu
registry            latest              0bb8b1006103        2 days ago          33.27 MB
king:5000/ubuntu    latest              bd3d4369aebc        2 weeks ago         126.6 MB
ubuntu              latest              bd3d4369aebc        2 weeks ago         126.6 MB

#目前已经是V2版本
[root@king docker]# curl http://king:5000/v2/ubuntu/tags/list
{"name":"ubuntu","tags":["latest"]}

#列出所有库
curl king:5000/v2/_catalog
{"repositories":["ubuntu"]}
{%endhighlight%}

## Docker数据管理
- 容器管理数据方式：1.数据卷Data Volumes;2.数据卷容器Data Volumes Dontainers
- 数据卷可以共享重用，修改立即生效，对镜像无影响，始终存在。

{%highlight bash%}
#以下创建了数据卷 /webapp
$docker run -d -P --name web -v /webapp training/webapp python app.py

#将宿主机/src/webapp 目录挂在到容器的/webapp,/src/webapp目录在命令执行时会自动创建
docker run -d -P --name web -v /src/webapp:/webapp training/webapp python app.py

#只读挂在
docker run -d -P --name webs2 -v /src/webapp:/webapp:ro training/webapp python app.py

#挂在单独文件读写
$docker run --rm -it -v ~/test.file:/test.file ubuntu /bin/bash
{%endhighlight%} 

### 通过数据卷容器共享
{%highlight bash%}
#创建数据卷容器
$docker run -it -v /opt/data --name dbdata ubuntu

#使用数据卷容器
$docker run -it --volumes-from dbdata --name db1 ubuntu
$docker run -it --volumes-from dbdata --name db2 ubuntu

#--volumes-from 可以多次使用，或者传递性挂载
$docker run -it --volumes-from db2 --name db3 ubuntu

#--volumes-from 后面作为数据卷使用的容器，本身不需要处于运行状态即可使用
{%endhighlight%}

## 小结
- Registry V2 HTTP API查询仓库清单 http://RegistryServerIP:5000/v2/_catalog
- docker run -P 随机选取宿主机IP映射到容器IP
- docker run -it -v HOST_PATH:CONTAINER_PATH ubuntu 挂载数据卷
- 通过多个容器使用--volumes-from 同一个数据卷容器达到数据共享同步


