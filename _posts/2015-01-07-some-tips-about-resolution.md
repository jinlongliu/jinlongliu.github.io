---
layout: post
title: "Radxa分辨率修改零星笔记"
description: ""
category: "Radxa"
tags: []
---
{% include JB/setup %}
{% highlight ruby %}
1.刷新率=dotclock/(horizontal period)*(vertical period)
2.horizontal period = left margin + right margin + hsync + xres
3.vertical period = upper margin + low margin + vsync + yres
4.h_bp 左边界 h_fp 右边界 v_bp 上边界 v_fp 下边界 h_pw 水平同步 v_pw 垂直同步
5.bp:back porch;	fp:front porch;		pw:pulse width;
6.Pixclock = 1 / dotclock
7.hdmi_init_lcdc(rk29fb_screen, rk29lcd_info *lcd_info)	>> .set_screen__info	>> rk3188_lcdc_probe call set_screen_info 
{% endhighlight%}
