---
layout: post
title: "Mysql远程访问"
description: ""
category: "运维"
tags: [Mysql]
---
{% include JB/setup %}
<h3>Mysql远程访问设置</h3>
{% highlight bash %}
>use mysql;
>GRANT ALL ON *.* TO root@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
>flush privileges; 
{% endhighlight %}

### MySQL 8 修改密码

{% highlight bash %}
>CREATE USER 'super'@'%' IDENTIFIED BY 'MyPass@123';
>GRANT ALL ON *.* TO 'super'@'%' WITH GRANT OPTION;
>flush privileges;
>ALTER USER 'super'@'%' IDENTIFIED WITH mysql_native_password BY 'MyPass@123';

{% endhighlight %}