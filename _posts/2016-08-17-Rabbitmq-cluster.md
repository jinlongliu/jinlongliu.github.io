---
layout: post
title: "Rabbitmq集群"
description: ""
category: Openstack
tags: [Openstack, Linux]
---
{% include JB/setup %}
## 配置Rabbitmq集群
{%highlight bash%}
    
#主节点重置
    rabbitmqctl stop_app
    rabbitmqctl reset
    rabbitmqctl start_app

#其它节点重置
    ssh node2 -- rabbitmqctl stop_app
    ssh node2 -- rabbitmqctl reset
    ssh node2 -- rabbitmqctl start_app

    ssh node3 -- rabbitmqctl stop_app
    ssh node3 -- rabbitmqctl reset
    ssh node3 -- rabbitmqctl start_app

#停止所有节点
    systemctl stop rabbitmq-server.service
    ssh node2 -- systemctl stop rabbitmq-server.service
    ssh node3 -- systemctl stop rabbitmq-server.service

#同步节点文件
    scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
    scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie
    ssh node2 -- chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
    ssh node2 -- chmod 400 /var/lib/rabbitmq/.erlang.cookie
    ssh node3 -- chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
    ssh node3 -- chmod 400 /var/lib/rabbitmq/.erlang.cookie

#开启节点
    systemctl start rabbitmq-server.service
    ssh node2 -- systemctl start rabbitmq-server.service
    ssh node3 -- systemctl start rabbitmq-server.service

#加入集群
    ssh node2 -- rabbitmqctl stop_app
    ssh node2 -- rabbitmqctl join_cluster --ram rabbit@node1
    ssh node2 -- rabbitmqctl start_app

    ssh node3 -- rabbitmqctl stop_app
    ssh node3 -- rabbitmqctl join_cluster --ram rabbit@node1
    ssh node3 -- rabbitmqctl start_app

#设置策略
    rabbitmqctl set_policy ha-all '^(?!amq\.).*' '{"ha-mode": "all"}'

#配置用户
    rabbitmqctl set_permissions guest ".*" ".*" ".*"
    rabbitmqctl change_password guest ${RABBIT_PASS}
{%endhighlight%}
