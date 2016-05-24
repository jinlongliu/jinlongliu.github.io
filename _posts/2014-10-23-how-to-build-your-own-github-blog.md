---
layout: post
title: "Github博客搭建"
description: ""
category: 其它
tags: [Github Blog]
---
<h>
Ubuntu安装Jekyll本地测试环境
</h>
<p>
{% highlight ruby %}
#apt-get install ruby1.9.3
#apt-get install ruby-dev
#apt-get install rake
#apt-get install nodejs
{% endhighlight %}
</p>
<p>
卸载gem<br/>
{%  highlight ruby %}
#gem uninstall XXX
{% endhighlight %}
</p>
xx
<p>
Github增加SSH-KEY
{%  highlight ruby %}
#ssh-keygen
{% endhighlight %}
</p>
<p>
将id_rsa.pub内容拷贝至setting内即可。<br/>
jekyll默认监听ip 127.0.0.1需要使用-H 0.0.0.0指向所有IP，否则访问无反应。<br/>
jekyll >= 2.4.0<br/>
ruby >= 1.9.3
</p>
<p>
Git提交
</p>
{%  highlight ruby %}
#git status
#git add -a
#git commit -a -m "XXX"
#git push origin master
{% endhighlight %}
<p>
修改gem源为国内淘宝更新源
</p>
{%  highlight ruby %}
#gem source -l
*** CURRENT SOURCES ***
  
http://ruby.taobao.org/
#gem sources --remove https://rubygems.org/
#gem sources -a https://ruby.taobao.org/
{% endhighlight %}
<p>
博客站点预览
</p>
{%  highlight ruby %}
#rake preview
{% endhighlight %}
{% include JB/setup %}
