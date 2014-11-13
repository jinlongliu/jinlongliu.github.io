---
layout: post
title: "SPICE+Libvirt：声音+USB重定向"
description: ""
category: "SPICE"
tags: [spice, libvirt, usb重定向]
---
{% include JB/setup %}
<p>
SPICE协议桌面，定义Guest声音驱动，以及USB重定向的libvirt.xml
</p>
{% highlight mxml %}
<devices>
    ...
    <sound model='ac97'/>
    <controller type='usb' index='0' model='ich9-ehci1'/>
    <controller type='usb' index='0' model='ich9-uhci1'>
      <master startport='0'/>
    </controller>
    <redirdev bus='usb' type='spicevmc'/>
</devices>
{% endhighlight %}
<p>
从命令行启动的参数命令
</p>
{% highlight console %}
root@ubuntu#qemu-system-x86_64 -m 4096 -drive file=disk,if=virtio -net nic -net user -spice port=5988,disable-ticketing -enable-kvm  -soundhw ac97 -device qxl-vga -smp 16,sockets=1,cores=4,threads=4 -device ich9-usb-ehci1,id=usb -device ich9-usb-uhci1,masterbus=usb.0,firstport=0,multifunction=on -chardev spicevmc,name=usbredir,id=usbredirchardev1 -device usb-redir,chardev=usbredirchardev1,id=usbredirdev1
{% endhighlight %}
