---
layout: post
title: "Radxa rock修改分辨率"
description: ""
category: "Radxa"
tags: []
---
{% include JB/setup %}
<p>
修改屏幕配置参数
</p>
{% highlight ruby %}
--- drivers/video/rockchip/screen/lcd_1080p.c   (revision 3)
+++ drivers/video/rockchip/screen/lcd_1080p.c   (working copy)
@@ -13,17 +13,17 @@
 #define OUT_FACE           OUT_P888
 
 #define LCDC_ACLK              297000000
-#define DCLK           148500000
+#define DCLK           106500000
 
-#define H_PW                   44
-#define H_BP                   148
-#define H_VD                   1920
-#define H_FP                   88
+#define H_PW                   152
+#define H_BP                   232
+#define H_VD                   1440
+#define H_FP                   80
 
-#define V_PW                   5
-#define V_BP                   36
-#define V_VD                   1080
-#define V_FP                   4
+#define V_PW                   6
+#define V_BP                   25
+#define V_VD                   900
+#define V_FP                   3
 
 #define LCD_WIDTH              0
 #define LCD_HEIGHT             0
{% endhighlight %}
<p>
修改HDMI芯片写入寄存器操作
</p>
{% highlight ruby %}
Index: drivers/video/rockchip/hdmi/chips/rk616/rk616_hdmi_hw.c
===================================================================
--- drivers/video/rockchip/hdmi/chips/rk616/rk616_hdmi_hw.c     (revision 3)
+++ drivers/video/rockchip/hdmi/chips/rk616/rk616_hdmi_hw.c     (working copy)
@@ -315,7 +315,7 @@
        hdmi_writel(VIDEO_CONTRL3, value);
 
        // Set ext video
-#if 1
+#if 0
        hdmi_writel(VIDEO_TIMING_CTL, 0);
        mode = (struct fb_videomode *)hdmi_vic_to_videomode(vpara->vic);
        if(mode == NULL)
@@ -325,6 +325,7 @@
        }
        hdmi->tmdsclk = mode->pixclock;
 #else
+        mode = (struct fb_videomode *)hdmi_vic_to_videomode(vpara->vic);
        value = v_EXTERANL_VIDEO(1) | v_INETLACE(mode->vmode);
        if(mode->sync & FB_SYNC_HOR_HIGH_ACT)
                value |= v_HSYNC_POLARITY(1);
@@ -336,6 +337,16 @@
        hdmi_writel(VIDEO_EXT_HTOTAL_L, value & 0xFF);
        hdmi_writel(VIDEO_EXT_HTOTAL_H, (value >> 8) & 0xFF);
{% endhighlight %}
<p>
修改HDMI输出时序
</p>
{% highlight ruby %}
Index: drivers/video/rockchip/hdmi/rk_hdmi_lcdc.c
===================================================================
--- drivers/video/rockchip/hdmi/rk_hdmi_lcdc.c  (revision 3)
+++ drivers/video/rockchip/hdmi/rk_hdmi_lcdc.c  (working copy)
@@ -27,7 +27,7 @@
 //{    "1920x1080i@50Hz",      50,             1920,   1080,   74250000,       148,    528,    15,     2,      44,     5,      FB_SYNC_HOR_HIGH_ACT | FB_SYNC_VERT_HIGH_ACT,                        1,              20      },
 //{    "1920x1080i@60Hz",      60,             1920,   1080,   74250000,       148,    88,     15,     2,      44,     5,      FB_SYNC_HOR_HIGH_ACT | FB_SYNC_VERT_HIGH_ACT,                        1,              5       },
 {      "1920x1080p@50Hz",      50,             1920,   1080,   148500000,      148,    528,    36,     4,      44,     5,      FB_SYNC_HOR_HIGH_ACT | FB_SYNC_VERT_HIGH_ACT,                        0,              31      },
-{      "1920x1080p@60Hz",      60,             1920,   1080,   148500000,      148,    88,     36,     4,      44,     5,      FB_SYNC_HOR_HIGH_ACT | FB_SYNC_VERT_HIGH_ACT,                        0,              16      },
+{      "1920x1080p@60Hz",      60,             1440,   900,    106500000,      232,    80,     25,     3,      152,    6,      FB_SYNC_HOR_HIGH_ACT | FB_SYNC_VERT_HIGH_ACT,                        0,              16      },
{% endhighlight %}



