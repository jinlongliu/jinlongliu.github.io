---
layout: post
title: "基于Ubuntu14.04的shadowsocks安装"
description: ""
category: "运维" 
tags: [Ubuntu14.04, Shadowsocks]
---
{% include JB/setup %}
<p>
追加更新源
</p>
{% highlight ruby %}
vi /etc/apt/sources.list
{% endhighlight %}
<p>
新增如下更新源记录：
</p>
{% highlight ruby %}
deb http://shadowsocks.org/debian squeeze main
{% endhighlight %}
<p>
更新源并安装
</p>
{% highlight ruby %}
apt-get update
apt-get install shadowsocks
{% endhighlight %}
<p>
配置服务器设置
</p>
{% highlight ruby %}
vi /etc/shadowsocks/config.json
{
          "server":"vps的ip",   #AMW 服务器填写私有IP，非公有IP
          "server_port":8388,
          "local_port":1080,
          "password":"barfoo!", #认证密码
          "timeout":60,
          "method":"table"      #加密方式，默认table，推荐aes-256-cfb
}
{% endhighlight %}
<p>
如果想用除table以外的加密方式，需要额外安装M2Crypto
</p>
{% highlight ruby %}
apt-get install python-m2crypto
{% endhighlight %}
<p>
重启shadowsocks服务：
</p>
{% highlight ruby %}
service shadowsocks restart	#重启
service shadowsocks start	#启动
service shadowsocks stop	#关闭
{% endhighlight %}
<p>
下载Windows ShadowsocksGUI作为访问客户端结合Chrome Proxy SwitchySharp就可以顺利观看1024喽！
</p>
