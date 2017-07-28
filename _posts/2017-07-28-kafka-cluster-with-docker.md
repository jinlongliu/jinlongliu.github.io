---
layout: post
title: "Docker swarm模式部署Kafka集群"
description: ""
category: Docker 
tags: [Docker, Kafka, Zookeeper]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### 部署Zookeeper集群

`
version: '3.3'
 services:
   zookeeper1:
     image: zookeeper:3.4.10
     hostname: zookeeper1
 #    container_name: zookeeper1
     deploy:
 #     mode: replicated
 #     replicas: 1
       mode: global
       endpoint_mode: dnsrr
       restart_policy:
         condition: on-failure
         delay: 5s
 #       max_attempts: 3
       placement:
         constraints:
           - node.hostname == ilog1
 
     env_file:
       - zookeeper.env
     environment:
       ZOO_MY_ID: 1
     networks:
       ilog_net:
         aliases:
           - zookeeper1
     ports:
       - target: 2181
         published: 2181
         protocol: tcp
         mode: host
 
     volumes:
       - type: bind
         source: /opt/zookeeper/conf
         target: /conf
       - type: bind
         source: /opt/zookeeper/data
         target: /data
       - type: bind
         source: /opt/zookeeper/datalog
         target: /datalog
 
     extra_hosts:
       - "ilog1:172.18.60.133"
       - "ilog2:172.18.60.134"
       - "ilog3:172.18.60.135"
 
   zookeeper2:
     image: zookeeper:3.4.10
     hostname: zookeeper2
     deploy:
       mode: global
       endpoint_mode: dnsrr
       restart_policy:
         condition: on-failure
         delay: 5s
       placement:
         constraints:
           - node.hostname == ilog2
 
     env_file:
       - zookeeper.env
     environment:
       ZOO_MY_ID: 2
     networks:
       ilog_net:
         aliases:
           - zookeeper2
     ports:
       - target: 2181
         published: 2181
         protocol: tcp
         mode: host
 
     volumes:
       - type: bind
         source: /opt/zookeeper/conf
         target: /conf
       - type: bind
         source: /opt/zookeeper/data
         target: /data
       - type: bind
         source: /opt/zookeeper/datalog
         target: /datalog
         
     extra_hosts:
       - "ilog1:172.18.60.133"
       - "ilog2:172.18.60.134"
       - "ilog3:172.18.60.135"
   
   zookeeper3:
     image: zookeeper:3.4.10
     hostname: zookeeper3
     deploy:
       mode: global
       endpoint_mode: dnsrr
       restart_policy:
         condition: on-failure
         delay: 5s
       placement:
         constraints:
           - node.hostname == ilog3
 
     env_file:
       - zookeeper.env
     environment:
       ZOO_MY_ID: 3
     networks:
       ilog_net:
         aliases:
           - zookeeper3
     ports:
       - target: 2181
         published: 2181
         protocol: tcp
         mode: host
 
     volumes:
       - type: bind
         source: /opt/zookeeper/conf
         target: /conf
       - type: bind
         source: /opt/zookeeper/data
         target: /data
       - type: bind
         source: /opt/zookeeper/datalog
         target: /datalog
         
     extra_hosts:
       - "ilog1:172.18.60.133"
       - "ilog2:172.18.60.134"
       - "ilog3:172.18.60.135"
       
 networks:
   ilog_net:
     driver: overlay
     external: true
`


{% include JB/setup %}


