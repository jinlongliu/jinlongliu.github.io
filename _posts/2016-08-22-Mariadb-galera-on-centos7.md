---
layout: post
title: "Mariadb Galera Cluster"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### Mariadb Galera 集群

#### 配置软件源
{% highlight bash %}
#vim /etc/yum.repos.d/Mariadb.repo

[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
{% endhighlight %}

#### 关闭防火墙
{% highlight bash %}
systemctl stop firewalld.service
{% endhighlight %}

#### 安装软件
{% highlight bash %}
yum --nogpgcheck -y install MariaDB-server galera
{% endhighlight %}

#### Mysql配置
{% highlight bash %}
#vim /etc/my.cnf.d/server.cnf 

[galera]
wsrep_on=ON

user=mysql
binlog_format=ROW
bind-address=10.0.0.11

default_storage_engine=innodb
innodb_autoinc_lock_mode=2
innodb_flush_log_at_trx_commit=0
innodb_buffer_pool_size=122M

wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_provider_options="pc.recovery=TRUE;gcache.size=300M"
wsrep_cluster_name="my_cluster"
wsrep_cluster_address="gcomm://10.0.0.11,10.0.0.12,10.0.0.13"
wsrep_sst_method=rsync
wsrep_node_name="node1"
wsrep_node_address="10.0.0.11"
{% endhighlight %}

#### 注意：bind-address 不可以为127.0.0.1 也不可以为0.0.0.0否则与Haproxy冲突

#### 启动集群
{% highlight bash %}
#在第一个节点上mysql_secure_installation进行用户配置，后续会自动同步
#First node
mysqld --wsrep-new-cluster

#或者
service mysql start --wsrep-new-cluster

#Not Firest node
systemctl start mariadb
{% endhighlight %}

#### 集群状态查询
{% highlight bash %}
MariaDB [(none)]> show status like 'wsrep_%';
+------------------------------+----------------------------------------------+
| Variable_name                | Value                                        |
+------------------------------+----------------------------------------------+
| wsrep_apply_oooe             | 0.000000                                     |
| wsrep_apply_oool             | 0.000000                                     |
| wsrep_apply_window           | 1.000000                                     |
| wsrep_causal_reads           | 0                                            |
| wsrep_cert_deps_distance     | 1.000000                                     |
| wsrep_cert_index_size        | 2                                            |
| wsrep_cert_interval          | 0.000000                                     |
| wsrep_cluster_conf_id        | 3                                            |
| wsrep_cluster_size           | 3                                            |
| wsrep_cluster_state_uuid     | 847994b1-684e-11e6-8cdd-f7899dd9dd58         |
| wsrep_cluster_status         | Primary                                      |
| wsrep_commit_oooe            | 0.000000                                     |
| wsrep_commit_oool            | 0.000000                                     |
| wsrep_commit_window          | 1.000000                                     |
| wsrep_connected              | ON                                           |
| wsrep_evs_delayed            |                                              |
| wsrep_evs_evict_list         |                                              |
| wsrep_evs_repl_latency       | 0/0/0/0/0                                    |
| wsrep_evs_state              | OPERATIONAL                                  |
| wsrep_flow_control_paused    | 0.000000                                     |
| wsrep_flow_control_paused_ns | 0                                            |
| wsrep_flow_control_recv      | 0                                            |
| wsrep_flow_control_sent      | 0                                            |
| wsrep_gcomm_uuid             | 84789f47-684e-11e6-a605-7be2012f0874         |
| wsrep_incoming_addresses     | 10.0.0.11:3306,10.0.0.12:3306,10.0.0.13:3306 |
| wsrep_last_committed         | 9                                            |
| wsrep_local_bf_aborts        | 0                                            |
| wsrep_local_cached_downto    | 1                                            |
| wsrep_local_cert_failures    | 0                                            |
| wsrep_local_commits          | 0                                            |
| wsrep_local_index            | 0                                            |
| wsrep_local_recv_queue       | 0                                            |
| wsrep_local_recv_queue_avg   | 0.083333                                     |
| wsrep_local_recv_queue_max   | 2                                            |
| wsrep_local_recv_queue_min   | 0                                            |
| wsrep_local_replays          | 0                                            |
| wsrep_local_send_queue       | 0                                            |
| wsrep_local_send_queue_avg   | 0.000000                                     |
| wsrep_local_send_queue_max   | 1                                            |
| wsrep_local_send_queue_min   | 0                                            |
| wsrep_local_state            | 4                                            |
| wsrep_local_state_comment    | Synced                                       |
| wsrep_local_state_uuid       | 847994b1-684e-11e6-8cdd-f7899dd9dd58         |
| wsrep_protocol_version       | 7                                            |
| wsrep_provider_name          | Galera                                       |
| wsrep_provider_vendor        | Codership Oy <info@codership.com>            |
| wsrep_provider_version       | 25.3.15(r3578)                               |
| wsrep_ready                  | ON                                           |
| wsrep_received               | 12                                           |
| wsrep_received_bytes         | 1606                                         |
| wsrep_repl_data_bytes        | 2552                                         |
| wsrep_repl_keys              | 7                                            |
| wsrep_repl_keys_bytes        | 217                                          |
| wsrep_repl_other_bytes       | 0                                            |
| wsrep_replicated             | 7                                            |
| wsrep_replicated_bytes       | 3217                                         |
| wsrep_thread_count           | 2                                            |
+------------------------------+----------------------------------------------+
{% endhighlight %}

