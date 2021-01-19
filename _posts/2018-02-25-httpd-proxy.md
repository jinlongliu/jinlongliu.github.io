---
layout: post
title: "HTTPD反向代理"
description: ""
category: Linux
tags: [Linux, CentOS]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### Httpd反向代理配置

##### 部署前后端分离的VUE静态
```
ProxyRequests off
ProxyPass /backend-cluster http://118.24.206.150:8002
ProxyPassReverse /backend-cluster http://118.24.206.150:8002
```

{% include JB/setup %}