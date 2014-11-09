---
layout: post
title: "SecureCRT使用pem key"
description: ""
category: "运维" 
tags: [Pem Key, SecureCRT]
---
{% include JB/setup %}
<p>
将密钥上传至Linux服务器，并修改权限。以文件king.pem为例:
</p>
{% highlight ruby %}
chmod 600 king.pem
{% endhighlight %}
<p>
修改密钥格式为OpenSSH，如果询问，留空回车：
</p>
{% highlight ruby %}
ssh-keygen -p -f king.pem
{% endhighlight %}
<p>
生成公钥.pub文件：
</p>
{% highlight ruby %}
ssh-keygen -e -f king.pem >> king.pem.pub
{% endhighlight %}
<p>
下载生成.pub文件，在使用SecrueCRT连接时指定相应pub文件即可。
</p>
