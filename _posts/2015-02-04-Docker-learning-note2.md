---
layout: post
title: "Docker知识梳理（二）"
description: ""
category: Docker 
tags: [Docker, Linux]
---
{% include JB/setup %}
## 运行使用Docker
- 容器是镜像的一个实例，其拥有额外的可写文件层。
- 通过命令参数-d，使容器以守护态（Daemonized）运行,只和docker run 使用

### 创建容器
{%highlight bash%}
$docker create -it NAME[:TAG]
$docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
82c5aad05d1e        test:2              "/bin/bash"         6 seconds ago       Created                                 goofy_bell
64068b0b1e31        test:2              "/bin/bash"         17 seconds ago      Created                                 peaceful_goodall
cfd4b89c9d73        test:2              "/bin/bash"         26 seconds ago      Created                                 modest_goldberg
#容器创建后状态为Created，可以用start启动，状态切换为Up
$docker start 82c5aad05d1e
[root@king ~]# docker ps -a             
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
82c5aad05d1e        test:2              "/bin/bash"         About a minute ago   Up 3 seconds                            goofy_bell
64068b0b1e31        test:2              "/bin/bash"         About a minute ago   Created                                 peaceful_goodall
cfd4b89c9d73        test:2              "/bin/bash"         2 minutes ago        Created                                 modest_goldberg

#新建并启动容器,等价于先执行docker create 再执行docker start
$docker run test:2 /bin/echo 'Hello world'
Hello world
{%endhighlight%}

docker run流程
- 检测本地镜像是否存在，不存在从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，在只读镜像层外挂载一层可读写层
- 从宿主机网卡桥接一个虚拟接口到容器中【通过网桥实现】
- 从地址池分配IP给容器
- 执行用户指定应用程序
- 执行完毕后容器终止

### 终止容器
docker stop [-t|--time[=10]] CONTAINER_ID 先发送SIGTERM信号，等待（默认10s）发送SIGKILL信号终止容器
{%highlight bash%}
#查看终止状态的容器ID信息
$docker ps -a -q

#终止容器后再启动，stop start依次执行
$docker restart test:2
{%endhighlight%}

### 进入容器
容器以守护态运行后，用户可以通过命令介入容器
{%highlight bash%}
#启动容器
$[root@king ~]# docker run -idt test:2
8d372300f7fe4a74568e18c96417e34b8f50e79cac354b781ab78181d59fab66

#查看容器ID
[root@king ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8d372300f7fe        test:2              "/bin/bash"         5 seconds ago       Up 4 seconds                            cocky_bohr

#介入容器
[root@king ~]# docker attach 8d372300f7fe
root@8d372300f7fe:/# 

ONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b016ba8e759d        test:2              "/bin/bash"         4 seconds ago       Up 3 seconds                            hopeful_goldwasser
#Docker 1.3+ 通过exec在容器内运行命令
[root@king ~]# docker exec -ti b016ba8e759d /bin/bash
root@b016ba8e759d:/# 
{%endhighlight%}

### 删除容器
- 命令格式 docker rm [OPTIONS] CONTAINER [CONTAINER...]
- -f, --force=false 强制终止并删除
- -l, --link=false 删除容器连接但保留容器
- -v, --volumes=false 删除容器挂载的数据卷

{%highlight bash%}
#删除
$docker rm b016ba8e759d

#强制删除运行的容器
$docker rm -f b016ba8e759d
{%endhighlight%}

### 导入和导出容器
- 导出命令格式 docker export CONTAINER > OUTPUT_FILE_NAME
- 导入命令格式 cat INPUT_FILE_NAME \| docker import - IMAGE_NAME[:TAG]
{%highlight bash%}
cfd4b89c9d73        test:2              "/bin/bash"              34 minutes ago       Created                                             modest_goldberg
[root@king ~]# docker export cfd>output.tar
[root@king ~]# du -sh output.tar 
111M    output.tar

#将容器文件导入作为镜像
[root@king ~]# cat output.tar | docker import - local/image:tag
sha256:006a0ba18e1daff51de347fcc2f64b9f04e56ca2a72f13a1d99657f34f3f4977
[root@king ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
local/image         tag                 006a0ba18e1d        4 seconds ago       110.3 MB
test                2                   d13991969eee        About an hour ago   126.6 MB
registry/test       2.0                 d13991969eee        About an hour ago   126.6 MB
ubuntu              latest              bd3d4369aebc        2 weeks ago         126.6 MB
{%endhighlight%}

## 小结
- 创建容器 docker create NAME[:TAG]
- 启动容器 docker start CONTAINER_ID
- 创建并启动 docker run NAME[:TAG]
- 终止容器 docker stop CONTAINER_ID
- 进入容器 docker attach CONTAINER_ID
- 运行命令 docker exec COMMAND
- 删除容器 docker rm CONTAINER_ID
- 导出容器 docker export CONTAINER_ID>OUTPUT_FILE_NAME
- 导入容器为镜像 cat INPUT_FILE_NAME \| docker import -  [REPOSITORY[:TAG]]
- 守护态参数 -d 和命令docker run 使用非docker create
- docker rm xxx  可以使用容器ID的前三个字母来作为参数操作容器，会自动索引

