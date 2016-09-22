---
layout: post
title: "Ceph常用操作命令"
description: ""
category: 存储 
tags: [存储, Linux, Ceph]
---
{% include JB/setup %}
## Ceph常用命令
{%highlight bash%}
#创建池
$ceph osd pool create <poolname> <int[0-]> {<int[0-]>} {replicated|erasure} {<int>}

#[123[ 4]]   auid 1234  crush rule 4
$rados mkpool <pool-name> [123[ 4]]

$rados lspools
$rados rmpool <pool-name> [<pool-name> --yes-i-really-really-mean-it]

#创建卷
$rbd create [--order <bits>] [--image-features <features>] [--image-shared]
$rbd create -s 4096 volumes/vol1

#查看卷信息
$rbd info volumes/vol1
rbd image 'vol1':
        size 4096 MB in 1024 objects
        order 22 (4096 kB objects)
        block_name_prefix: rb.0.2a44.238e1f29
        format: 1

#映射卷为块设备
$rbd map volumes/vol1
/dev/rbd0

#取消映射
$rbd unmap /dev/rbd0 

#取消映射
$rbd unmap volumes/vol1

#调整大小
$rbd resize -s 5120 volumes/vol1
Resizing image: 100% complete...done.

$rbd resize -s 4096 volumes/vol1 --allow-shrink
Resizing image: 100% complete...done.

#显示池下所有卷
$rbd ls volumes
vol1
volume-e2a437ef-7eb5-433b-90b4-40e60c353f12

#删除卷
$rbd rm volumes/vol1
Removing image: 100% complete...done.

#卷导出导入
$rbd export volumes/vol1 output.file
Exporting image: 100% complete...done.

$rbd import output.file volumes/vol2
Importing image: 100% complete...done.
{%endhighlight%}

## 通过配置CRUSH实现不同配置的池
{%highlight bash%}
#导出CRUSH并转换为可读文件
$ceph osd getcrushmap -o cluster_crush_map
got crush map from osdmap epoch 26

#转化
crushtool -d cluster_crush_map -o cluster_crush_map.txt
{%endhighlight%}

配置示例
{%highlight bash%}
# begin crush map
tunable choose_local_tries 0
tunable choose_local_fallback_tries 0
tunable choose_total_tries 50
tunable chooseleaf_descend_once 1
tunable straw_calc_version 1

# devices
device 0 osd.0
device 1 osd.1
device 2 osd.2

# types
type 0 osd
type 1 host
type 2 chassis
type 3 rack
type 4 row
type 5 pdu
type 6 pod
type 7 room
type 8 datacenter
type 9 region
type 10 root

# buckets
host node1 {
        id -2           # do not change unnecessarily
        # weight 0.010
        alg straw
        hash 0  # rjenkins1
        item osd.0 weight 0.010
}
host node2 {
        id -3           # do not change unnecessarily
        # weight 0.010
        alg straw
        hash 0  # rjenkins1
        item osd.1 weight 0.010
}
host node3 {
        id -4           # do not change unnecessarily
        # weight 0.010
        alg straw
        hash 0  # rjenkins1
        item osd.2 weight 0.010
}
root default {
        id -1           # do not change unnecessarily
        # weight 0.030
        alg straw
        hash 0  # rjenkins1
        item node1 weight 0.010
        item node2 weight 0.010
        item node3 weight 0.010
}

# rules
rule replicated_ruleset {
        ruleset 0
        type replicated
        min_size 1
        max_size 10
        step take default
        step chooseleaf firstn 0 type host
        step emit
}

# end crush map
{%endhighlight%}

### 修改配置，注意新建条目的id，保持不冲突
- 修改可读配置文件，删除原host中部分item osd
- 新建新的host把之前删除的osd放入到新建host中
- 新建新的root把新建的host放入其中
- 新建rule把step take参数换成新建的root

## 导入新的CRUSH配置
{%highlight bash%}
#将可读配置转换
$crushtool -c cluster_crush_map.txt -o new_map

#导入
$ceph osd setcrushmap -i new_map 
set crush map
{%endhighlight%}

## 查询相关
{%highlight bash%}
#查询池的规则集合
$ceph osd pool get volumes crush_ruleset 
crush_ruleset: 0

#设置池规则集合
$ceph osd pool set volumes crush_ruleset 0
set pool 2 crush_ruleset to 0

#显示树结构
$ceph osd tree
ID WEIGHT  TYPE NAME      UP/DOWN REWEIGHT PRIMARY-AFFINITY 
-1 0.02998 root default                                     
-2 0.00999     host node1                                   
 0 0.00999         osd.0       up  1.00000          1.00000 
-3 0.00999     host node2                                   
 1 0.00999         osd.1       up  1.00000          1.00000 
-4 0.00999     host node3                                   
 2 0.00999         osd.2       up  1.00000          1.00000
{%endhighlight%}

