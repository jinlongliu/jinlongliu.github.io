---
layout: post
title: "CentOS7网卡重命名"
description: ""
category: Linux 
tags: [Linux]
---
{% include JB/setup %}

#### CentOS7 默认网卡名称可读性不高，自定义修改。

## 修改内核启动参数
{%highlight bash%}
$vim /etc/default/grub

GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap net.ifnames=0 biosdevname=0 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"

#新增内核参数，net.ifnames=0 biosdevname=0

#生成GRUB更新内核参数，重启生效
$grub2-mkconfig -o /boot/grub2/grub.cfg
{%endhighlight%}

## 自定义其它名称
{%highlight bash%}
$vim /etc/udev/rules.d/70-persistent-net.rules

ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{type}=="1", ATTR{address}=="00:0c:29:6c:8d:16", NAME="new-name"
{%endhighlight%}
