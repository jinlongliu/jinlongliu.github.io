---
layout: post
title: "ubuntu安装设置SVN"
description: ""
category: "Linux"
tags: [运维]
---
{% highlight bash %}
#apt-get install subversion
root@king:/home/svn_repo# svnadmin create rockchip
  
配置文件去注释,行前不留空格
#anon-access = read
#auth-access = writ
#password-db = passwd
#authz-db = authz 
  
启动脚本
svnserve -d -r /home/svn_repo/ 
{% endhighlight %}
{% include JB/setup %}
