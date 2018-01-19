---
layout: post
title: "Docker私有仓库"
description: ""
category: Docker 
tags: [Docker]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### Docker私有仓库

##### 启动命令
```
docker run -d -p 5000:5000 --restart always --name registry \
    -v /docker-registry:/var/lib/registry registry:2.6.2-jlog
```

##### 修改配置
```
/etc/docker/registry # vi config.yml

delete:
  enabled: true
```

##### 启用私有仓库
```
vim /etc/docker/daemon.json

{
   "insecure-registries" : ["127.0.0.1:5000"]
}


#ubuntu
vim /etc/init.d/docker
DOCKER_OPTS="--insecure-registry 127.0.0.1:5000"
```

### 仓库REST API
```
curl -X GET http://127.0.0.1:5000/_catalog

curl -X GET http://127.0.0.1:5000/v2/nginx/tags/list

curl -X GET http://127.0.0.1:5000/v2/jlog-web/tags/list |python -m json.tool

curl -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET http://172.18.60.172:5000/v2/jlog-web/manifests/4.0.3 2>&1 | grep Docker-Content-Digest | awk '{print ($3)}

curl -I -X DELETE  http://172.18.60.172:5000/v2/redis/manifests/sha256:96dc2d508309fe197fd320b20a56c7373c19ee143385eb33e5845679bbac92b9
```

### 回收
```
/etc/docker/registry # registry garbage-collect /etc/docker/registry/config.yml
```

{% include JB/setup %}