---
layout: post
title: "CentOS7部署pacemaker集群"
description: ""
category: Linux
tags: [Linux]
---
{% include JB/setup %}
## 部署Pacemaker集群于CentOS7

### 安装组件
{%highlight bash%}
#安装组件在所有节点
    yum install pacemaker -y
    yum install pcs -y
    yum install corosync -y
    yum install fence-agents -y
    yum install resource-agents -y
{%endhighlight%}

### 配置PCS
{%highlight bash%}
#在所有节点配置
    systemctl enable pcsd
    systemctl start pcsd

#配置各集群节点用户名
    echo cctcloud | passwd --stdin hacluster
{%endhighlight%}

### 通过PCS配置整个集群
{%highlight bash%}

#通过pcs配置整个集群，参数包含上述步骤用户名信息
    pcs cluster auth node1 node2 node3 -u hacluster -p cctcloud --force
    pcs cluster setup --force --name openstack-cluster node1 node2 node3
    pcs cluster start --all

#配置集群属性
    pcs property set pe-warn-series-max=1000 \
    pe-input-series-max=1000 \
    pe-error-series-max=1000 \
    cluster-recheck-interval=5min

#禁用STONITH否则资源无法启动
    pcs property set stonith-enabled=false
    pcs resource op defaults timeout=120s


#示例配置资源
    pcs resource create vip ocf:heartbeat:IPaddr2 \
    params ip="$CONTROLLER_VIP" cidr_netmask="24" op monitor interval="30s"

    pcs resource create api_vip ocf:heartbeat:IPaddr2 \
    params ip="$CONTROLLER_API_VIP" cidr_netmask="24" op monitor interval="30s"

    pcs resource create lb-haproxy systemd:haproxy --clone

    pcs constraint order start vip then api_vip kind=Optional
    pcs constraint order start api_vip then lb-haproxy-clone kind=Optional

    pcs constraint colocation add api_vip with vip
    pcs constraint colocation add lb-haproxy-clone with vip
{%endhighlight%}

## 查看Corosync状态
{%highlight bash%}
#查看当前节点
[root@node1 ~]# corosync-cfgtool -s
Printing ring status.
Local node ID 1
RING ID 0
        id      = 192.168.13.201
        status  = ring 0 active with no faults

#查看整个集群节点
[root@node1 ~]# corosync-cmapctl runtime.totem.pg.mrp.srp.members
runtime.totem.pg.mrp.srp.members.1.config_version (u64) = 0
runtime.totem.pg.mrp.srp.members.1.ip (str) = r(0) ip(192.168.13.201) 
runtime.totem.pg.mrp.srp.members.1.join_count (u32) = 1
runtime.totem.pg.mrp.srp.members.1.status (str) = joined
runtime.totem.pg.mrp.srp.members.2.config_version (u64) = 0
runtime.totem.pg.mrp.srp.members.2.ip (str) = r(0) ip(192.168.13.202) 
runtime.totem.pg.mrp.srp.members.2.join_count (u32) = 7
runtime.totem.pg.mrp.srp.members.2.status (str) = joined
runtime.totem.pg.mrp.srp.members.3.config_version (u64) = 0
runtime.totem.pg.mrp.srp.members.3.ip (str) = r(0) ip(192.168.13.203) 
runtime.totem.pg.mrp.srp.members.3.join_count (u32) = 6
runtime.totem.pg.mrp.srp.members.3.status (str) = joined
{%endhighlight%}

## 常用管理命令
{%highlight bash%}
#指定节点进入Standby或者Unstandby状态，资源将不运行于该节点
$pcs cluster standby/unstandby [<node>] | --all

#整个集群启停
$pcs cluster start/stop [--all] [node] [...]

#单个资源的使能
$crm_resource --resource vip --set-parameter target-role --meta --parameter-value Started/Stopped

#某个资源从节点转移，--node node2 可以不指定
$crm_resource --resource vip --move --node node2

#通知集群某个文件失败
$crm_resource --resource mysql --fail

#定义集权默认超时时间
$pcs resource op defaults timeout = 90s

#crm_resource 参数简写
--resource              -r
--set-paremeter         -p
--parameter-value       -v
--meta                  -m
--delete-parameter      -d

#清除资源已记录状态、错误，集群尝试恢复启动资源,如果不指定--node则清除所有节点
$crm_resource -r resource_id --cleanup --node node4

#配置单个资源操作超时
$pcs resource update resource_id op start timeout="60s"

#重启整个集群某个资源
$pcs resource restart vip

#禁用某个资源
$pcs resource enable/disable [resource/clone/group]
{%endhighlight%}
