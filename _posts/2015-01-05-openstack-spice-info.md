---
layout: post
title: "Openstack SPICE INFO NOTE"
description: ""
category: "Openstack"
tags: []
---
{% include JB/setup %}
{% highlight ruby%}
#/usr/lib/python2.7/dist-packages/nova/api/openstack/compute
__init__.py				定义访问URL
consoleproxy.py				call get_spice_info 
  
#/usr/lib/python2.7/dist-packages/nova/compute
api.py		get_spice_console	get_spice_info		控制节点	新增配置导入
rpcapi.py	get_spice_console	get_spice_info		控制节点 
manager.py	get_spice_console	get_spice_info		计算节点
  
#/usr/lib/python2.7/dist-packages/nova/virt/libvirt   作用于Compute节点 
driver.py	get_spice_console	get_spice_info		新增配置导入
  
#/usr/lib/python2.7/dist-packages/nova/spice
参数在nova/spice/__init__.py内注册
{% endhighlight %}
