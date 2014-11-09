---
layout: post
title: "基于Ubuntu 14.04的iSCSI存储配置--Target创建，Initiator连接"
description: ""
category: "运维" 
tags: [iSCSI,Ubuntu14.04]
---
{% include JB/setup %}
<p>
主机Win7 32位系统，Virtual Box 虚拟机Ubuntu 14.04 32位，SSH登录操作。环境问题Target和Initiator都是在同一台机器上，实际环境是可以完全分开独立分布于多个服务器。
</p>
<p>
安装Target软件
</p>
{% highlight ruby %}
root@king:~# apt-cache search tgt
...
...
nagios-plugins-basic - Plugins for nagios compatible monitoring systems
tgt - Linux SCSI target user-space tools
epiphany-browser - Intuitive GNOME web browser
...
python-webkit-dev - WebKit/Gtk Python bindings: development files
  
#当系统update后可通过search查看tgt软件
root@king:~# apt-get install tgt
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  libboost-system1.54.0 libboost-thread1.54.0 libconfig-general-perl
  libibverbs1 libnspr4 libnss3 libnss3-nssdb librados2 librbd1 librdmacm1
  libsgutils2-2 sg3-utils
The following NEW packages will be installed:
  libboost-system1.54.0 libboost-thread1.54.0 libconfig-general-perl
  libibverbs1 libnspr4 libnss3 libnss3-nssdb librados2 librbd1 librdmacm1
  libsgutils2-2 sg3-utils tgt
0 upgraded, 13 newly installed, 0 to remove and 0 not upgraded.
Need to get 3,858 kB of archives.
After this operation, 13.9 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
  
#安装完毕后检查tgt运行状态
Processing triggers for libc-bin (2.19-0ubuntu6) ...
Processing triggers for ureadahead (0.100.0-16) ...
root@king:~# service tgt status
tgt start/running, process 1690
  
#tgt服务正在运行进程号1690
root@king:~# ps aux |grep 1690
root      1690  0.0  0.3  16764  3140 ?        Ss   21:47   0:00 tgtd
root      1716  0.0  0.0   4676   824 pts/0    S+   21:54   0:00 grep --color=auto 1690
  
#tgtadmin   help查看tgtadmin 命令参数
root@king:~# tgtadm --help
Linux SCSI Target administration utility, version 1.0.43
  
Usage: tgtadm [OPTION]
--lld <driver> --mode target --op new --tid <id> --targetname <name>
        add a new target with <id> and <name>. <id> must not be zero.
--lld <driver> --mode target --op delete [--force] --tid <id>
        delete the specific target with <id>.
        With force option, the specific target is deleted
        even if there is an activity.
--lld <driver> --mode target --op show
        show all the targets.
--lld <driver> --mode target --op show --tid <id>
        show the specific target's parameters.
--lld <driver> --mode target --op update --tid <id> --name <param> --value <value>
        change the target parameters of the target with <id>.
--lld <driver> --mode target --op bind --tid <id> --initiator-address <address>
--lld <driver> --mode target --op bind --tid <id> --initiator-name <name>
        enable the target to accept the specific initiators.
--lld <driver> --mode target --op unbind --tid <id> --initiator-address <address>
--lld <driver> --mode target --op unbind --tid <id> --initiator-name <name>
        disable the specific permitted initiators.
--lld <driver> --mode logicalunit --op new --tid <id> --lun <lun>
  --backing-store <path> --bstype <type> --bsopts <bs options> --bsoflags <options>
        add a new logical unit with <lun> to the specific
        target with <id>. The logical unit is offered
        to the initiators. <path> must be block device files
        (including LVM and RAID devices) or regular files.
        bstype option is optional.
        bsopts are specific to the bstype.
        bsoflags supported options are sync and direct
        (sync:direct for both).
--lld <driver> --mode logicalunit --op delete --tid <id> --lun <lun>
        delete the specific logical unit with <lun> that
        the target with <id> has.
--lld <driver> --mode account --op new --user <name> --password <pass>
        add a new account with <name> and <pass>.
--lld <driver> --mode account --op delete --user <name>
        delete the specific account having <name>.
--lld <driver> --mode account --op bind --tid <id> --user <name> [--outgoing]
        add the specific account having <name> to
        the specific target with <id>.
        <user> could be <IncomingUser> or <OutgoingUser>.
        If you use --outgoing option, the account will
        be added as an outgoing account.
--lld <driver> --mode account --op unbind --tid <id> --user <name> [--outgoing]
        delete the specific account having <name> from specific
        target. The --outgoing option must be added if you
        delete an outgoing account.
--lld <driver> --mode lld --op start
        Start the specified lld without restarting the tgtd process.
--control-port <port> use control port <port>
--help
        display this help and exit
  
Report bugs to <stgt@vger.kernel.org>.
{% endhighlight %}
<p>
创建Target<br>
tgtadmin中关于target命令如 下
</p>
{% highlight ruby %}
--lld <driver> --mode target --op new --tid <id> --targetname <name>
        add a new target with <id> and <name>. <id> must not be zero.
--lld <driver> --mode target --op delete [--force] --tid <id>
        delete the specific target with <id>.With force option, the specific target is deleted
        even if there is an activity.
--lld <driver> --mode target --op show
        show all the targets.
  
#创建target
root@king:~# tgtadm --lld iscsi --mode target --op new --tid 1 --targetname iqn.2014-0728.com:longtang
  
#--lld iscsi 固定参数驱动iscsi
#--mode  target 模式target
#--op    new    操作new新创建
#--tid   1      target id号
#--targetname ...target名称
#查看Target记录</span>
  
root@king:~# tgtadm --lld iscsi --mode target --op show
Target 1: iqn.2014-0728.com:longtang
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    <span style="color:#000099;">LUN information:
        LUN: 0
            Type: controller
            SCSI ID: IET     00010000
            SCSI SN: beaf10
            Size: 0 MB, Block size: 1
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: null
            Backing store path: None
            Backing store flags:</span> 
    Account information:
    ACL information:
  
#相关参数会列出，会默认创建LUN号为0的逻辑单元。
#绑定initiator name，如果不操作后续Initiator可能无法发现target。
root@king:~# tgtadm --lld iscsi --mode target --op bind --tid 1 -I ALL
#上述命令为任何initiator都可以连接该target。
{% endhighlight %}

<p>
为Target新增逻辑单元
</p>
{% highlight ruby %}
--lld <driver> --mode logicalunit --op new --tid <id> --lun <lun>
  --backing-store <path> --bstype <type> --bsopts <bs options> --bsoflags <options>
        add a new logical unit with <lun> to the specific
        target with <id>. The logical unit is offered
        to the initiators. <path> must be block device files
        <span style="color:#ff0000;">(including LVM and RAID devices)</span> or<span style="color:#ff0000;"> regular files</span>.
        bstype option is optional.
        bsopts are specific to the bstype.
        bsoflags supported options are sync and direct
        (sync:direct for both).
--lld <driver> --mode logicalunit --op delete --tid <id> --lun <lun>
        delete the specific logical unit with <lun> that
        the target with <id> has.
  
命令规范如上
  
root@king:~# tgtadm --lld iscsi --mode logicalunit --op new --tid 1 --lun 1 -b /dev/my_volume_group/kinglv 
root@king:~# tgtadm --lld iscsi --mode target --op show
Target 1: iqn.2014-0728.com:longtang
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
<span style="white-space:pre">	</span>    ...
            Backing store flags: 
       <span style="color:#000099;"> LUN: 1
            Type: disk
            SCSI ID: IET     00010001
            SCSI SN: beaf11
            Size: 105 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/my_volume_group/kinglv
            Backing store flags:</span> 
    Account information:
    ACL information:
# 创建逻辑单元号为1的逻辑单元，路径可以为LVM设备或者RAID设备，或者是文件。有时利用dd创建模拟文件可以作为path。
{% endhighlight %}

<p>
安装客户端软件
</p>
{% highlight ruby %}
root@king:~# apt-get install open-iscsi
Reading package lists... Done
Building dependency tree  
...     
  
#检查客户端运行状态，如果未运行则开启
root@king:~# service open-iscsi status
Current active iSCSI sessions:
iscsiadm: No active sessions.
root@king:~# service open-iscsi start
 * Starting iSCSI initiator service iscsid                                                                                            
iSCSI daemon already running
{% endhighlight %}

<p>
iSCSI客户端连接Target
</p>
{% highlight ruby %}
发现Target
root@king:~# iscsiadm -m discovery -t sendtargets -p 192.168.1.120 
192.168.1.120:3260,1 iqn.2014-0728.com:longtang
# 如果上述target 的initiator name不绑定为all此处会无法发现target。
  
#显示已发现target
root@king:~# iscsiadm -m node
192.168.1.120:3260,1 iqn.2014-0728.com:longtang
  
#连接Target
root@king:~# iscsiadm -m node -p 192.168.1.120 --login     
Logging in to [iface: default, target: iqn.2014-0728.com:longtang, portal: 192.168.1.120,3260] (multiple)
Login to [iface: default, target: iqn.2014-0728.com:longtang, portal: 192.168.1.120,3260] successful.
  
#查看磁盘设备
root@king:~# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda5  /dev/sdb  /dev/sdb1  /dev/sdb2  /dev/sdb3  /dev/sdb4  /dev/sdc
root@king:~# fdisk -l /dev/sdc
  
Disk /dev/sdc: 104 MB, 104857600 bytes
4 heads, 50 sectors/track, 1024 cylinders, total 204800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
  
Disk /dev/sdc doesn't contain a valid partition table
    
#断开Target
root@king:~# iscsiadm -m node -p 192.168.1.120 --logout
Logging out of session [sid: 1, target: iqn.2014-0728.com:longtang, portal: 192.168.1.120,3260]
Logout of [sid: 1, target: iqn.2014-0728.com:longtang, portal: 192.168.1.120,3260] successful.
root@king:~# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda5  /dev/sdb  /dev/sdb1  /dev/sdb2  /dev/sdb3  /dev/sdb4
#发现连接和断开之间设备相差一个/dev/sdc 大小在104MB。我们底层实际存储设备LVM逻辑卷kinglv大小相近。
  
root@king:~# lvdisplay /dev/my_volume_group/kinglv 
  --- Logical volume ---
  LV Path                /dev/my_volume_group/kinglv
  LV Name                kinglv
  VG Name                my_volume_group
  LV UUID                lAT5Te-obE1-Q3dY-pU4r-YYDt-rSRu-hgmKQP
  LV Write Access        read/write
  LV Creation host, time king, 2014-07-27 14:43:41 +0800
  LV Status              available
  # open                 1
  <span style="color:#ff0000;">LV Size                100.00 MiB</span>
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
  
如果怕指定IP什么太烦，直接连接所有Target
root@king:~# iscsiadm -m node --login all
Logging in to [iface: default, target: iqn.2014-0728.com:longtang, portal: 192.168.1.120,3260] (multiple)
Login to [iface: default, target: iqn.2014-0728.com:longtang, portal: 192.168.1.120,3260] successful.
{% endhighlight %}
<p>
iSCSI 端口号
</p>
{% highlight ruby %}
root@king:~# netstat -anl | grep 3260
tcp        0      0 0.0.0.0:3260            0.0.0.0:*               LISTEN     
tcp6       0      0 :::3260                 :::*                    LISTEN   
#/etc/service 中也存在默认端口号
  
...
icpv2           3130/tcp        icp             # Internet Cache Protocol
icpv2           3130/udp        icp
iscsi-target    3260/tcp
mysql           3306/tcp
mysql           3306/udp
...
{% endhighlight %}
<p>
上述只是简单介绍了iSCSI的使用，方便快速建立一个基于网络的存储服务，但个Target是可以供多个Initiator连接的。更多更详细的操作大家可以自己实践。
</p>
