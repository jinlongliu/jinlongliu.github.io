---
layout: post
title: "基于Ubuntu 14.04 的DRBD配置"
description: ""
category: 
tags: []
---
{% include JB/setup %}
<p>
安装DRBD软件：
</p>
{% highlight ruby %}
root@controller1:/# apt-get install -y --force-yes drbd8-utils
Reading package lists... Done
Building dependency tree
....   
  
# 查看drbd状态
root@controller1:/# service drbd status
drbd driver loaded OK; device status:
version: 8.4.3 (api:1/proto:86-101)
srcversion: F97798065516C94BE0F27DC 
m:res  cs  ro  ds  p  mounted  fstype
#drbd属于两节点，需要在另外一节点安装一样的软件
{% endhighlight %}

<p>
配置Drbd资源
</p>
{% highlight ruby %}
root@controller1:/# cd /etc/drbd.d/
root@controller1:/etc/drbd.d# ls
global_common.conf  mysql.res
#进入drbd配置目录，可以看到global_common.conf配置文件，上述mysql.res为笔者后来创建文件 
  
#创建Drbd设备资源配置文件，创建并且编辑文件
root@controller1:/etc/drbd.d# touch mysql.res
root@controller1:/etc/drbd.d# vim mysql.res&nbsp;
  
#资源配置格式，如下文实际文件
resource mysql {
    device minor 0;
    disk "/dev/sdb5";
    meta-disk internal;
    on controller1 {
        address ipv4 192.168.2.11:7700;
    }
    on controller2 {
        address ipv4 192.168.2.12:7700;
    }
}
  
resource 资源名 {
    device minor 0;    设备号  
    disk "/dev/sdb5";  底层实际使用的设备，两个节点可以有差异
    meta-disk internal;
节点信息，controller1，controller2为两节点hostname
    on controller1 {
        address ipv4 192.168.2.11:7700;
    }
    on controller2 {
        address ipv4 192.168.2.12:7700;
    }
}
{% endhighlight %}

<p>
启动设备资源
</p>
{% highlight ruby %}
当两个节点的配置文件都创建完毕后
root@controller1:/etc/drbd.d# drbdadm create-md mysql
You want me to create a v08 style flexible-size internal meta data block.
There appears to be a v08 flexible-size internal meta data block
already in place on /dev/sdb5 at byte offset 5368705024
Do you really want to overwrite the existing v08 meta-data?
[need to type 'yes' to confirm] yes
  
Writing meta data...
md_offset 5368705024
al_offset 5368672256
bm_offset 5368508416
  
Found xfs filesystem
     5242684 kB data area apparently used
     5242684 kB left usable by current configuration
  
Even though it looks like this would place the new meta data into
unused space, you still need to confirm, as this is only a guess.
  
Do you want to proceed?
[need to type 'yes' to confirm] yes
  
initializing activity log
NOT initializing bitmap
New drbd meta data block successfully created.
  
#启用设备
root@controller1:/etc/drbd.d# drbdadm up mysql 
root@controller1:/etc/drbd.d# drbd-overview 
  0:mysql/0  WFConnection Secondary/Unknown Inconsistent/DUnknown C r----s

#强制为主节点，只需要在一个节点上操作
root@controller1:~# drbdadm -- --force primary mysql
root@controller1:~# drbd-overview 
  0:mysql/0  WFConnection Primary/Unknown UpToDate/Outdated C r----s
  
#drbd-overview 可以查看drbd设备状态，另外drbd设备符
root@controller1:/dev/drbd# ls
by-disk  by-res
root@controller1:/dev/drbd# ls -al by-disk/
total 0
drwxr-xr-x 2 root root 60 Jul 29 13:20 .
drwxr-xr-x 4 root root 80 Jul 29 13:20 ..
lrwxrwxrwx 1 root root 11 Jul 29 13:22 sdb5 -> ../../drbd0
root@controller1:/dev/drbd# ls -al by-res/
total 0
drwxr-xr-x 3 root root 60 Jul 29 13:20 .
drwxr-xr-x 4 root root 80 Jul 29 13:20 ..
drwxr-xr-x 2 root root 60 Jul 29 13:20 mysql
root@controller1:/dev/drbd# ls -al by-res/mysql/
total 0
drwxr-xr-x 2 root root 60 Jul 29 13:20 .
drwxr-xr-x 3 root root 60 Jul 29 13:20 ..
lrwxrwxrwx 1 root root 14 Jul 29 13:22 0 -> ../../../drbd0
#实际的设备/dev/drbd0   设备号0为配置文件中
  
#拷贝文件至drbd设备
root@controller1:/# mount /dev/drbd0 /mnt/
root@controller1:/# cp -prf /var/lib/mysql/* /mnt/
root@controller1:/# drbd-overview 
  0:mysql/0  WFConnection Primary/Unknown UpToDate/Outdated C r----s /mnt xfs 5.0G 112M 4.9G 3%
  
# 此时另外一个节点处于outdated状态，只需要在另外节点创建并启用，两者即会同步
root@controller2:~# drbdadm create-md mysql&nbsp;
You want me to create a v08 style flexible-size internal meta data block.
There appears to be a v08 flexible-size internal meta data block
already in place on /dev/sda5 at byte offset 5368705024
Do you really want to overwrite the existing v08 meta-data?
[need to type 'yes' to confirm] yes
  
.......
New drbd meta data block successfully created.
root@controller2:~# drbd-overview 
  0:mysql/0  Unconfigured . . . . 
root@controller2:~# drbdadm up mysql 
root@controller2:~# drbd-overview&nbsp;
&nbsp; 0:mysql/0 &nbsp;SyncTarget Secondary/Primary Inconsistent/UpToDate C r-----&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; [>....................] sync'ed: &nbsp;0.7% (5088/5116)Mfinish: 0:02:42 speed: 31,744 (31,744) want: 30,800 K/sec
{% endhighlight %}
<p>
Drbd脑裂行为解决
</p>
{% highlight ruby %}
#如上我们看到节点2上数据处于standalone状态，即发生了脑裂。【因为该环境之前本身就有数据】
#此时可以通过，使得次节点数据不可靠，让其主动同主节点同步
root@controller2:~# drbdadm secondary mysql
root@controller2:~# drbdadm  -- --discard-my-data connect mysql
root@controller1:/# drbdadm connect mysql
  
# 如果此种情况节点2，仍然是StandAlone，则可以完全重建节点2。
root@controller2:~# drbdadm detach mysql 
root@controller2:~# drbdadm down mysql 
root@controller2:~# drbdadm create-md mysql 
You want me to create a v08 style flexible-size internal meta data block.
There appears to be a v08 flexible-size internal meta data block
already in place on /dev/sda5 at byte offset 5368705024
Do you really want to overwrite the existing v08 meta-data?
[need to type 'yes' to confirm]
........
root@controller2:~# drbdadm up mysql&nbsp;
root@controller2:~# drbd-overview&nbsp;
&nbsp; 0:mysql/0 &nbsp;SyncTarget Secondary/Primary Inconsistent/UpToDate C r-----&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; [>....................] sync'ed: &nbsp;1.5% (5044/5116)Mfinish: 0:01:07 speed: 75,776 (75,776) want: 91,600 K/sec
  
#经过一段时间同步后，同步完成。
root@controller1:/# drbd-overview 
  0:mysql/0  SyncSource Primary/Secondary UpToDate/Inconsistent C r----- /mnt xfs 5.0G 112M 4.9G 3% 
        [================>...] sync'ed: 86.2% (708/5116)Mfinish: 0:00:13 speed: 54,484 (54,420) K/sec
  
#两个节点同步时，一个状态为SyncSource一个状态为SyncTarget
root@controller1:/# drbd-overview 
  0:mysql/0  Connected Primary/Secondary UpToDate/UpToDate C r----- /mnt xfs 5.0G 112M 4.9G 3%
  
#同步完成状态为Connected
{% endhighlight %}
