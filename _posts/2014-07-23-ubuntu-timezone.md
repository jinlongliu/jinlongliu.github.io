---
layout: post
title: "Ubuntu时区设置"
description: ""
category: "Linux"
tags: []
---
{% include JB/setup %}
<p>
关闭UTC
</p>
{% highlight ruby %}
root@king:~# vim /etc/default/rcS 
  
UTC=yes
  
UTC=no
{% endhighlight%}
<p>
修改时区
</p>
{% highlight ruby %}
#rm /etc/localtime
#ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
{% endhighlight%}
