---
layout: post
title: "Ubuntu14.04安装L2TP VPN"
description: "VPN"
category: 其它
tags: [VPN]
---
### 安装软件
{% highlight bash %}
apt-get install openswan xl2tpd ppp lsof
{% endhighlight %}

### 配置防火墙规则
{% highlight bash %}
iptables -t nat -A POSTROUTING -j SNAT --to-source %SERVERIP% -o eth+
{% endhighlight %}

%SERVERIP%  使用服务器IP替换 <br/>
eth+  更换为对应的网络接口名称


### 启用内核IP转发
{% highlight bash %}
echo "net.ipv4.ip_forward = 1" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.accept_redirects = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.send_redirects = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.rp_filter = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.accept_source_route = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.send_redirects = 0" |  tee -a /etc/sysctl.conf
echo "net.ipv4.icmp_ignore_bogus_error_responses = 1" |  tee -a /etc/sysctl.conf
{% endhighlight %}
##### 其它网卡设置
{% highlight bash %}
for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
{% endhighlight %}
##### 应用设置
{% highlight bash %}
sysctl -p
{% endhighlight %}
### 配置IPSEC
{% highlight bash %}
vim /etc/ipsec.conf  
{% endhighlight %}

{% highlight bash %}
version 2 # conforms to second version of ipsec.conf specification

config setup
    dumpdir=/var/run/pluto/
    #in what directory should things started by setup (notably the Pluto daemon) be allowed to dump core?

    nat_traversal=yes
    #whether to accept/offer to support NAT (NAPT, also known as "IP Masqurade") workaround for IPsec

    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v6:fd00::/8,%v6:fe80::/10
    #contains the networks that are allowed as subnet= for the remote client. In other words, the address ranges that may live behind a NAT router through which a client connects.

    protostack=netkey
    #decide which protocol stack is going to be used.

    force_keepalive=yes
    keep_alive=60
    # Send a keep-alive packet every 60 seconds.

conn L2TP-PSK-noNAT
    authby=secret
    #shared secret. Use rsasig for certificates.

    pfs=no
    #Disable pfs

    auto=add
    #the ipsec tunnel should be started and routes created when the ipsec daemon itself starts.

    keyingtries=3
    #Only negotiate a conn. 3 times.

    ikelifetime=8h
    keylife=1h

    ike=aes256-sha1,aes128-sha1,3des-sha1
    phase2alg=aes256-sha1,aes128-sha1,3des-sha1
    # https://lists.openswan.org/pipermail/users/2014-April/022947.html
    # specifies the phase 1 encryption scheme, the hashing algorithm, and the diffie-hellman group. The modp1024 is for Diffie-Hellman 2. Why 'modp' instead of dh? DH2 is a 1028 bit encryption algorithm that modulo's a prime number, e.g. modp1028. See RFC 5114 for details or the wiki page on diffie hellmann, if interested.

    type=transport
    #because we use l2tp as tunnel protocol

    left=%SERVERIP%
    #fill in server IP above

    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any

    dpddelay=10
    # Dead Peer Dectection (RFC 3706) keepalives delay
    dpdtimeout=20
    #  length of time (in seconds) we will idle without hearing either an R_U_THERE poll from our peer, or an R_U_THERE_ACK reply.
    dpdaction=clear
    # When a DPD enabled peer is declared dead, what action should be taken. clear means the eroute and SA with both be cleared. 
{% endhighlight %}
### 验证IPSEC
{% highlight bash %}
ipsec verify

Checking your system to see if IPsec got installed and started correctly:
Version check and ipsec on-path                                 [OK]
Linux Openswan U2.6.38/K3.13.0-24-generic (netkey)
Checking for IPsec support in kernel                            [OK]
 SAref kernel support                                           [N/A]
 NETKEY:  Testing XFRM related proc values                      [OK]
    [OK]
    [OK]
Checking that pluto is running                                  [OK]
 Pluto listening for IKE on udp 500                             [OK]
 Pluto listening for NAT-T on udp 4500                          [OK]
Checking for 'ip' command                                       [OK]
Checking /bin/sh is not /bin/dash                               [WARNING]
Checking for 'iptables' command                                 [OK]
Opportunistic Encryption Support                                [DISABLED]
{% endhighlight %}

### 配置xl2tpd
{% highlight bash %}
vim /etc/xl2tpd/xl2tpd.conf  
##############################
[global]
ipsec saref = yes
saref refinfo = 30

;debug avp = yes
;debug network = yes
;debug state = yes
;debug tunnel = yes

[lns default]
ip range = 172.16.1.30-172.16.1.100
local ip = 172.16.1.1
refuse pap = yes
require authentication = yes
;ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
{% endhighlight %}

### 配置用户
{% highlight bash %}
vim /etc/ppp/chap-secrets 
###############################
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
liujinlong      l2tpd   liujinlong                      *
szuser          l2tpd   hxfzcct                 *
wanggang        l2tpd   wanggang                *
guest01         l2tpd   123qweasd               *
{% endhighlight %}

### 配置PPP
{% highlight bash %}
vim /etc/ppp/options.xl2tpd
###############################
require-mschap-v2
ms-dns 8.8.8.8
ms-dns 8.8.4.4
auth
mtu 1200
mru 1000
crtscts
hide-password
modem
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
{% endhighlight %}

### 重启服务验证
{% highlight bash %}
/etc/init.d/ipsec restart 
/etc/init.d/xl2tpd restart
{% endhighlight %}

### 服务器示例
{% highlight bash %}
root@CCTCloudServer:~# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
SNAT       all  --  anywhere             anywhere             to:192.168.51.100

root@CCTCloudServer:~# ipsec verify
Checking your system to see if IPsec got installed and started correctly:
Version check and ipsec on-path                                 [OK]
Linux Openswan U2.6.38/K3.13.0-24-generic (netkey)
Checking for IPsec support in kernel                            [OK]
 SAref kernel support                                           [N/A]
 NETKEY:  Testing XFRM related proc values                      [OK]
        [OK]
        [OK]
Checking that pluto is running                                  [OK]
 Pluto listening for IKE on udp 500                             [OK]
 Pluto listening for NAT-T on udp 4500                          [OK]
Two or more interfaces found, checking IP forwarding            [FAILED]
Checking NAT and MASQUERADEing                                  [OK]
Checking for 'ip' command                                       [OK]
Checking /bin/sh is not /bin/dash                               [WARNING]
Checking for 'iptables' command                                 [OK]
Opportunistic Encryption Support                                [DISABLED]
{% endhighlight %}

### VPN采用UDP端口
{% highlight bash %}
root@CCTCloudServer:~# ss -anup|column -t 
State   Recv-Q  Send-Q  Local                Address:Port  Peer                        Address:Port
UNCONN  0       0       127.0.0.1:4500       *:*           users:(("pluto",42500,21))
UNCONN  0       0       192.168.51.100:4500  *:*           users:(("pluto",42500,19))
UNCONN  0       0       192.168.51.255:137   *:*           users:(("nmbd",999,18))
UNCONN  0       0       192.168.51.100:137   *:*           users:(("nmbd",999,17))
UNCONN  0       0       *:137                *:*           users:(("nmbd",999,11))
UNCONN  0       0       192.168.51.255:138   *:*           users:(("nmbd",999,20))
UNCONN  0       0       192.168.51.100:138   *:*           users:(("nmbd",999,19))
UNCONN  0       0       *:138                *:*           users:(("nmbd",999,12))
UNCONN  0       0       127.0.0.1:500        *:*           users:(("pluto",42500,20))
UNCONN  0       0       192.168.51.100:500   *:*           users:(("pluto",42500,18))
UNCONN  0       0       *:1701               *:*           users:(("xl2tpd",42578,3))
UNCONN  0       0       ::1:500              :::*          users:(("pluto",42500,22))
#### UDP端口号1701 4500 500 需要暴露到公网
{% endhighlight %}
{% include JB/setup %}
