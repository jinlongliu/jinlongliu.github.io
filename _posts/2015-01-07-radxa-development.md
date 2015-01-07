---
layout: post
title: "Radxa编译内核生成新镜像"
description: ""
category: "Linux" 
tags: ["radxa", "Ubuntu"]
---
{% include JB/setup %}
<p>
系统：Ubuntu 14.04 LTS  <br/>
安装编译工具链
</p>
{% highlight ruby %}
sudo apt-get install gcc-arm-linux-gnueabihf
sudo apt-get install lzop libncurses5-dev
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
{% endhighlight %}
<p>
编译内核
</p>
{% highlight ruby %}
make radxa_rock_pro_linux_defconfig
make -j8
{% endhighlight %}
<p>
编译内核模块
</p>
{% highlight ruby %}
mkdir modules
export INSTALL_MOD_PATH=./modules
make modules && make modules_install
{% endhighlight %}
<p>
生成ramdisk <br/>
</p>
{% highlight ruby %}
git clone https://github.com/radxa/initrd.git
make -C initrd
{% endhighlight %}
<p>
生成boot.img <br/>
安装mkbootimg
</p>
{% highlight ruby %}
root@king#mkbootimg --kernel linux-rockchip/arch/arm/boot/Image --ramdisk initrd.img -o boot.img
{% endhighlight %}
<p>
修改使用新内核: <br/>
下载官方rock_lite镜像，使用Win32DiskImager将SD镜像写入SD卡，插入lite启动。通过scp或者工具将boot.img传输到rock_lite上。
</p>
{% highlight ruby %}
dd if=boot.img of=/dev/mmcblk0 seek=$((0x4000))
{% endhighlight %}
<p>
SD镜像各分区起始位和偏移量
</p>
{% highlight ruby %}
名称		起始位~结束位	单位
bootloader	0x0000~0x2000	block
parameter	0x2000~0x4000
boot.img	0x4000~0xC000
linuxroot	0xC000~0x82000
userdata	0x82000~
1 block = 0x200 bytes = 512 bytes
{% endhighlight %}




















