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

## 启动命令
```
docker run -d -p 5000:5000 --restart always --name registry \
    -v /docker-registry:/var/lib/registry registry:2.6.2-jlog
```



{% include JB/setup %}