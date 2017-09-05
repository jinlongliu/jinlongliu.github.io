---
layout: post
title: "Docker swarm模式部署Kafka集群"
description: ""
category: Docker 
tags: [Docker, Kafka, Zookeeper]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 部署Zookeeper集群

{% highlight bash %}
#ilog_zookeeper.yml
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
{% endhighlight %}

{% highlight bash %}
$docker stack deploy -c ilog_zookeeper.yml ilog_zk
{% endhighlight %}

{% highlight bash %}
#zookeeper.env
# Set zookeeper environment
ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
{% endhighlight %}

### 部署Kafka集群
{% highlight bash %}
#ilog_kafka.yml
version: '3.3'
services:
  kafka1:
    image: kafka:0.10.2
    hostname: kafka1
    deploy:
      mode: global
      endpoint_mode: dnsrr
      restart_policy:
        condition: on-failure
        delay: 5s
      placement:
        constraints:
          - node.hostname == ilog1
    environment:
#      KAFKA_ADVERTISED_HOST_NAME: ilog1
#      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://ilog1:9092
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    networks:
      ilog_net:
        aliases:
          - kafka1
    ports:
      - target: 9092
        published: 9092
        protocol: tcp
        mode: host
    volumes:
      - type: bind
        source: /data/kafka
        target: /kafka
    extra_hosts:
      - "ilog1:172.18.60.133"
      - "ilog2:172.18.60.134"
      - "ilog3:172.18.60.135"
      
  kafka2:
    image: kafka:0.10.2
    hostname: kafka2
    deploy:
      mode: global
      endpoint_mode: dnsrr
      restart_policy:
        condition: on-failure
        delay: 5s
      placement:
        constraints:
          - node.hostname == ilog2
    environment:
#      KAFKA_ADVERTISED_HOST_NAME: ilog2
#      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://ilog2:9092
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    networks:
      ilog_net:
        aliases:
          - kafka2
    ports:
      - target: 9092
        published: 9092
        protocol: tcp
        mode: host
    volumes:
      - type: bind
        source: /data/kafka
        target: /kafka        
    extra_hosts:
      - "ilog1:172.18.60.133"
      - "ilog2:172.18.60.134"
      - "ilog3:172.18.60.135"
      
  kafka3:
    image: kafka:0.10.2
    hostname: kafka3
    deploy:
      mode: global
      endpoint_mode: dnsrr
      restart_policy:
        condition: on-failure
        delay: 5s
      placement:
        constraints:
          - node.hostname == ilog3
    environment:
#      KAFKA_ADVERTISED_HOST_NAME: ilog3
#      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://ilog3:9092
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    networks:
      ilog_net:
        aliases:
          - kafka3
    ports:
      - target: 9092
        published: 9092
        protocol: tcp
        mode: host
    volumes:
      - type: bind
        source: /data/kafka
        target: /kafka        
    extra_hosts:
      - "ilog1:172.18.60.133"
      - "ilog2:172.18.60.134"
      - "ilog3:172.18.60.135"

networks:
  ilog_net:
    driver: overlay
    external: true

{% endhighlight %}

### Kafka-manager部署
{% highlight bash %}
#ilog_kafka-manager.yml
version: '3.3'
services:
  kafka-manager:
    image: kafka-manager:alpine
    hostname: kafka-manager
    deploy:
      mode: global
      endpoint_mode: dnsrr
      restart_policy:
        condition: on-failure
        delay: 5s
      placement:
        constraints:
          - node.hostname == ilog2
    environment:
      ZK_HOSTS: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    networks:
      ilog_net:
        aliases:
          - kafka-manager
    ports:
      - target: 9000
        published: 9001
        protocol: tcp
        mode: host
    extra_hosts:
      - "ilog1:172.18.60.133"
      - "ilog2:172.18.60.134"
      - "ilog3:172.18.60.135"

networks:
  ilog_net:
    driver: overlay
    external: true

{% endhighlight %}



{% include JB/setup %}


