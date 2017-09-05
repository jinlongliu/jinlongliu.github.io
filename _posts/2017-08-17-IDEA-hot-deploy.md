---
layout: post
title: "IDEA spring boot热部署"
description: ""
category: Java 
tags: [Spring]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 开启automake
{% highlight bash %}
Enter: CTRL + SHIFT + A 
勾选Compiler => Build project automatically

Enter：SHIFT + CTRL + ALT + /

点击Registry

勾选compiler.automake.allow.when.app.running
{% endhighlight %}

### 新增依赖
{% highlight bash %}
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
        <scope>runtime</scope>
    </dependency>
{% endhighlight %}

{% include JB/setup %}


