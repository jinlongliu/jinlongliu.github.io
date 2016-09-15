---
layout: post
title: "Docker知识梳理（一）"
description: ""
category: Docker 
tags: [Docker, Linux]
---
{% include JB/setup %}
## 认识Docker
- Docker是对应用组件的封装（Packaging）、分发（Distribution）、部署（Deployment）、运行（Runtime）等生命周期的管理。
- Docker引擎的基础是Linux容器（Linux Containers， LXC）技术。
- Docker与虚拟化比较：快（秒级）、资源低、管理方便、配置效率高。

## Docker核心概念
- 镜像（Image）：
- 容器（Container）：镜像只读，Docker在镜像上创建可写层。
- 仓库（Repository）：注册服务器存放多个仓库，每个仓库集存放一类镜像，多个镜像文件通过tag来区分。

## 镜像

#### 获取镜像
- 获取镜像,下载过程会输出各层信息，层（Layer）是AUFS（Advanced Union File System）
{%highlight bash%}
#如果不显示指定tag,默认取latest标签
$docker pull NAME[:TAG]

$docker pull ubuntu
$docker pull ubuntu:14.04
#相当于下列命令
$docker pull registry.hub.docker.com/ubuntu:latest
{%endhighlight%}
- 注册服务器 registry.hub.docker.com
- 仓库 ubuntu
- 标签 latest

#### 查看镜像
{%highlight bash%}
$docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              bd3d4369aebc        2 weeks ago         126.6 MB

#为本地镜像添加新标签
$docker tag ubuntu:latest ubuntu:14.04

#Tag不同但是ID一致，说明指向同一个文件
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               bd3d4369aebc        2 weeks ago         126.6 MB
ubuntu              latest              bd3d4369aebc        2 weeks ago         126.6 MB

#获取镜像详细情况,以下效果一样
$docker inspect bd3d4369aebc
$docker inspect ubuntu:14.04
[
    {
        "Id": "sha256:bd3d4369aebc4945be269859df0e15b1d32fefa2645f5699037d7d8c6b415a10",
        "RepoTags": [
            "ubuntu:14.04",
            "ubuntu:latest"
        ],
        "RepoDigests": [
            "ubuntu@sha256:f4691c96e6bbaa99d99ebafd9af1b68ace2aa2128ae95a60369c506dd6e6f6ab"
        ],
        ......
        ......
        ......
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:c8a75145fcc4e1a66cd86b3cbbe14da1a37894129005e461a43875a094b93412",
                "sha256:c6f2b330b60c7c32642c47871b28aab110a7214ed6aac305dd03f70b95cdc610",
                "sha256:055757a19384c8afff0e79db7bb84fd481d3a9565d78962c7f368d5ac5984998",
                "sha256:48373480614b79e5c1b0a080807fa8ffaea12695f548406ea77feb5074e195e3",
                "sha256:0cad5e07ba339f87eb58850252a0ad00e104bae4cfc66b376265e16c32a0aae9"
            ]
        }
    }
]

#获取指定一项
$docker inspect -f { {".Architecture"} } ubuntu:14.04
{%endhighlight%}

#### 搜寻镜像
{%highlight bash%}
$docker search TERM
$docker search mysql
{%endhighlight%}

#### 删除镜像
{%highlight bash%}
#当镜像有多个标签时，删除只删除标签，当为最后一个标签时，会删除镜像文件
$docker rmi IMAGE [IMAGE...]

[root@king ~]# docker rmi ubuntu:14.04
Untagged: ubuntu:14.04
[root@king ~]# docker images          
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              bd3d4369aebc        2 weeks ago         126.6 MB

#当使用镜像ID作为参数，会删除所有指向该镜像标签，然后删除文件，如果基于该镜像创建容器不可删除。
#强行删除镜像，添加参数 -f
{%endhighlight%}

#### 创建镜像
- 基于已有容器创建  docker commit [OPTIONS] CONTAINER [ERPOSITORY[:TAG]]
- 基于本地模板导入
- 基于Dockerfile创建
{%highlight bash%}
$docker commit -m "Add a new file" -a "Liu Jinlong" c0b91a306b78 new-image-name
-a --author="" 作者信息
-m --message="" 提交信息
-p --pause=true 提交时暂停容器运行

$cat ubuntu.minimal.tar.gz |docker import - ubuntu:14.04
{%endhighlight%}

#### 存出和载入镜像
{%highlight bash%}
$docker save -o ubuntu.tar ubuntu:latest
$docker load --input ubuntu.tar
$docker load < ubuntu.tar 
{%endhighlight%}

### 上传镜像
命令格式docker pusher NAME:[TAG]
{%highlight bash%}
$docker push registry/test:2.0
{%endhighlight%}

##小结
- 获取镜像 docker pull NAME[:TAG]
- 查看镜像 docker images
- 镜像详情 docker inspect NAME[:TAG]
- 搜索镜像 docker search TERM
- 删除镜像 docker rmi IMAGE [IMAGE...]
- 创建镜像 docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
-          docker import ...
- 存出镜像 docker save -o SAVE_AS_NAME NAME[:TAG]
- 载入镜像 docker load --input INPUT_FILE_NAME
-          docker load < INPUT_FILE_NAME
- 上传镜像 docker push NAME[:TAG]
- 镜像识别 [RegistryServer/]REPOSITORY[:TAG]


