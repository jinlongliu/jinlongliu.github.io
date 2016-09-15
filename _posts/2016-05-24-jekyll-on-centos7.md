---
layout: post
title: "CentOS7安装Jekyll"
description: ""
category: 其它
tags: [Github Blog]
---
### 安装依赖软件
{% highlight bash %}
yum install ruby
yum install ruby-devel
yum install gcc
{% endhighlight %}

##### /usr/share/ruby/mkmf.rb:434:in `try_do': The compiler failed to generate an executable file. (RuntimeError)
上述报错安装gcc可解决

### 修改gem源
{% highlight bash %}
gem source -l
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org
gem source -l
*** CURRENT SOURCES ***

https://ruby.taobao.org/
{% endhighlight %}

### gem安装依赖
{% highlight bash %}
gem install json_pure
gem install rouge
gem install bundler
gem install rake
{% endhighlight %}
##### /usr/local/share/ruby/site_ruby/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- json/pure
上述报错安装json_pure

### 安装Jekyll指定版本
{% highlight bash %}
gem install jekyll -v 2.4.0				# 安装指定版本，默认3.1.6有问题
jekyll build --incremental --watch		# 编译开启，并自动检测修改
jekyll serve -H 0.0.0.0			# 启动server并监听所有访问
rake preview					# 启动监听并自动检测修改		
{% endhighlight %}

<p>如果报错Liquid Exception: highlight tag was never closed in，配置_config.yml中添加excerpt_separator: ""</p>

### 创建文章
{% highlight bash %}
rake post	
{% endhighlight %}

### 提交文章
{% highlight bash %}
git add -A
git config --global user.email "nnuljl@qq.com"		
git config --global user.name "Jinlong Liu"
git commit -a -m "New commit"
git push origin				#提交是会询问github用户名密码	
{% endhighlight %}

{% include JB/setup %}
