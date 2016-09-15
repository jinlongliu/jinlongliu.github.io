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

