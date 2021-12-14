---
layout: post
title: "Nginx反向代理"
description: ""
category: Linux
tags: [Linux, CentOS]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### Nginx反向代理配置

##### 部署前后端分离的VUE静态
```
 server {
        listen       8081;
        listen       [::]:8081;
        server_name  _;
        root         /usr/share/nginx/html;

        location ^~ /backend-cluster/ {
            proxy_pass http://172.16.110.202:8002/;
        }
```

##### 添加部分为
```access transformers
        location ^~ /backend-cluster/ {
            proxy_pass http://172.16.110.202:8002/;
        }
```

{% include JB/setup %}