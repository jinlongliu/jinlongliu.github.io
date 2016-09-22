---
layout: post
title: "MongoDB集群"
description: ""
category: Openstack
tags: [Openstack, Linux]
---
{% include JB/setup %}

## MongoDB集群
{%highlight bash%}

#安装
    yum install -y mongodb-server mongodb

#修改配置文件
    CONFIG_FILE="/etc/mongod.conf"
    sed -i -e "s/bind_ip.*/bind_ip = 0.0.0.0/g" $CONFIG_FILE
    sed -i -e "s/#replSet.*/replSet = rs0\/node1:27017,node2:27017,node3:27017/g" $CONFIG_FILE

#启动服务
    systemctl enable mongod.service
    systemctl start mongod.service

#初始化集群，自动选择节点一主，两备
    mongo --eval "rs.initiate()"
    sleep 10

#初始化业务数据
    mongo --host node1 --eval "
  db = db.getSiblingDB('ceilometer');
  db.createUser({user: 'ceilometer',
  pwd: '$PASSWORD',
  roles: [ 'readWrite', 'dbAdmin' ]})"

#查询状态
$mongo
MongoDB shell version: 2.6.11
connecting to: test
Server has startup warnings: 
2016-09-21T02:11:06.511-0400 [initandlisten] 
2016-09-21T02:11:06.511-0400 [initandlisten] ** WARNING: Readahead for /var/lib/mongodb is set to 4096KB
2016-09-21T02:11:06.511-0400 [initandlisten] **          We suggest setting it to 256KB (512 sectors) or less
2016-09-21T02:11:06.511-0400 [initandlisten] **          http://dochub.mongodb.org/core/readahead

rs0:PRIMARY> rs.status();
{
        "set" : "rs0",
        "date" : ISODate("2016-09-22T05:59:42Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "node1:27017",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 85716,
                        "optime" : Timestamp(1474523848, 4),
                        "optimeDate" : ISODate("2016-09-22T05:57:28Z"),
                        "electionTime" : Timestamp(1474438302, 1),
                        "electionDate" : ISODate("2016-09-21T06:11:42Z"),
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "node2:27017",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 85688,
                        "optime" : Timestamp(1474523848, 4),
                        "optimeDate" : ISODate("2016-09-22T05:57:28Z"),
                        "lastHeartbeat" : ISODate("2016-09-22T05:59:41Z"),
                        "lastHeartbeatRecv" : ISODate("2016-09-22T05:59:40Z"),
                        "pingMs" : 1,
                        "syncingTo" : "node1:27017"
                },
                {
                        "_id" : 2,
                        "name" : "node3:27017",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 85686,
                        "optime" : Timestamp(1474523848, 4),
                        "optimeDate" : ISODate("2016-09-22T05:57:28Z"),
                        "lastHeartbeat" : ISODate("2016-09-22T05:59:41Z"),
                        "lastHeartbeatRecv" : ISODate("2016-09-22T05:59:41Z"),
                        "pingMs" : 2,
                        "syncingTo" : "node1:27017"
                }
        ],
        "ok" : 1
}
rs0:PRIMARY>
{%endhighlight%}

