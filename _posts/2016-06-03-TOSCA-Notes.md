---
layout: post
title: "TOSCA 学习笔记"
description: "TOSCA 学习笔记"
category: XOS
tags: [XOS， TOSCA]
---
{% include JB/setup %}

### 官方地址
http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/csprd02/TOSCA-Simple-Profile-YAML-v1.0-csprd02.pdf <br/>

### TOSCA   Topology Orchestration Specification for Cloud Application

### 核心概念
- node 节点
- relationship  关系

### 实现
- TOSCA YAML service template： 模板
- TOSCA processor： 处理器
- TOSCA orchestrator（orchestration engine）： 编排器
- TOSCA generator:  生成器，用于生成service template
- TOSCA archive： 存档包
- TOSCA Cloud Service Archive （CSAR）

### 术语表
- Instance Mode：运行的Service Template实例
- Node Template：Topology Template的一部分
- Relationship Template： 在Topology Template中各node之间的关系
- Service Template： 用来指定Topology和服务的编排（orchestration)
- Topology Model: 语义同于Topology Tmeplate
- Topology Template: 用于在Service Template中定义了service的结构，包含Node Template和Relationship Template的集合。

### 实例一：
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  node_templates:
    my_server:
      type: tosca.nodes.Compute
      capabilities:
        # Host container properties
        host:	
          properties:
            num_cpus: 1
            disk_size: 10 GB
            mem_size: 4096 MB
        # Guest Operating System properties
        os:
          properties:
            # host Operating System image properties
            architecture: x86_64
            type: linux
            distribution: rhel
            version: 6.5
{% endhighlight %}
##### 上述模板逻辑结构
<img src="/upload/2016/06/sample1.png"/>

### 实例二：输入、输出
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  inputs:
    cpus:
      type: integer
      description: Number of CPUs for the server.
      constraints:
        - valid_values: [ 1, 2, 4, 8 ]
  node_templates:
    my_server:
      type: tosca.nodes.Compute
      capabilities: 
        # Host container properties
        host:
          properties:
            # Compute properties
            num_cpus: { get_input: cpus }
            mem_size: 2048 MB
            disk_size: 10 GB
  outputs:
    server_ip:
      description: The private IP address of the provisioned server.
      value: { get_attribute: [ my_server, private_address ] }
{% endhighlight %}

### 实例三：软件安装
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  inputs:
    # omitted here for brevity【简洁】
  
  node_templates:
    mysql:
      type: tosca.nodes.DBMS.MySQL
      properties:
        root_password: { get_input: my_mysql_rootpw }
        port: { get_input: my_mysql_port }
      requirements:
        - host: db_server
    
    db_server:
      type: tosca.nodes.Compute
      capabilities:
        # omitted here for brevity
{% endhighlight %}
##### 上述模板逻辑结构
<img src="/upload/2016/06/sample2.png"/>

### 实例四：重写操作
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  inputs:
    # omitted here for brevity
  node_templates:
    mysql:
      type: tosca.nodes.DBMS.MySQL
      properties:
        root_password: { get_input: my_mysql_rootpw }
        port: { get_input: my_mysql_port }
      requirements:
        - host: db_server
      interfaces:
        Standard:
          configure: scripts/my_own_configure.sh
    db_server:
      type: tosca.nodes.Compute
      capabilities:
        # omitted here for brevity
{% endhighlight %}
Standard  生命周期 <br/>
scripts/my_own_configure.sh  为模板文件的相对路径 <br/>

### 实例五：数据库内容部署
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  inputs:
    # omitted here for brevity
  node_templates:
    my_db:
      type: tosca.nodes.Database.MySQL
      properties:
        name: { get_input: database_name }
        user: { get_input: database_user }
        password: { get_input: database_password }
        port: { get_input: database_port }
      artifacts:
        db_content:
          file: files/my_db_content.txt
          type: tosca.artifacts.File
      requirements:
        - host: mysql
      interfaces:
        Standard:
          create:
          implementation: db_create.sh
          inputs:
            # Copy DB file artifact to server’s staging area
            db_data: { get_artifact: [ SELF, db_content ] }
            
    mysql:
      type: tosca.nodes.DBMS.MySQL
      properties:
        root_password: { get_input: mysql_rootpw }
        port: { get_input: mysql_port }
      requirements:
        - host: db_server
        
    db_server:
      type: tosca.nodes.Compute
      capabilities:
        # omitted here for brevity
{% endhighlight %}
##### 上述模板逻辑结构
<img src="/upload/2016/06/sample3.png"/>

### 实例六：两层应用
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  inputs:
    # Admin user name and password to use with the WordPress application
    wp_admin_username:
      type: string
    wp_admin_password:
      type: string
    wp_db_name:
      type: string
    wp_db_user:
      type: string
    wp_db_password:
      type: string
    wp_db_port:
      type: integer
    mysql_root_password:
      type: string
    mysql_port:
      type: integer
    context_root:
      type: string
      
  node_templates:
    wordpress:
      type: tosca.nodes.WebApplication.WordPress
      properties:
        context_root: { get_input: context_root }
        admin_user: { get_input: wp_admin_username }
        admin_password: { get_input: wp_admin_password }
        db_host: { get_attribute: [ db_server, private_address ] }
      requirements:
        - host: apache
        - database_endpoint: wordpress_db
      interfaces:
        Standard:
          inputs:
            db_host: { get_attribute: [ db_server, private_address ] }
            db_port: { get_property: [ wordpress_db, port ] }
            db_name: { get_property: [ wordpress_db, name ] }
            db_user: { get_property: [ wordpress_db, user ] }
            db_password: { get_property: [ wordpress_db, password ] }
            
  apache:
    type: tosca.nodes.WebServer.Apache
    properties:
      # omitted here for brevity
    requirements:
      - host: web_server
      
  web_server:
    type: tosca.nodes.Compute
    capabilities:
      # omitted here for brevity
      
  wordpress_db:
    type: tosca.nodes.Database.MySQL
    properties:
      name: { get_input: wp_db_name }
      user: { get_input: wp_db_user }
      password: { get_input: wp_db_password }
      port: { get_input: wp_db_port }
    requirements:
      - host: mysql
      
  mysql:
    type: tosca.nodes.DBMS.MySQL
    properties:
      root_password: { get_input: mysql_root_password }
      port: { get_input: mysql_port }
    requirements:
      - host: db_server
      
  db_server:
    type: tosca.nodes.Compute
    capabilities:
      # omitted here for brevity
{% endhighlight %}

### 实例六：自定义脚本建立模板关系
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template for deploying a single server with predefined properties.

topology_template:
  inputs:
    # omitted here for brevity
      
  node_templates:
    wordpress:
      type: tosca.nodes.WebApplication.WordPress
      properties:
        # omitted here for brevity
      requirements:
        - host: apache
        - database_endpoint:
          node: wordpress_db
          relationship: my_custom_database_connection
          
    wordpress_db:
      type: tosca.nodes.Database.MySQL
      properties:
        # omitted here for the brevity
      requirements:
        - host: mysql
            
  relationship_templates:
    my_custom_database_connection:
      type: ConnectsTo
      interfaces:
        Configure:
          pre_configure_source: scripts/wp_db_configure.sh
  
  # other resources not shown for this example ...
{% endhighlight %}

### 实例七：自定义关系类型
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Definition of custom WordpressDbConnection relationship type

relationship_types:
  my.types.WordpressDbConnection:
    derived_from: tosca.relationships.ConnectsTo
    interfaces:
      Configure:
        pre_configure_source: scripts/wp_db_configure.sh
{% endhighlight %}

### 实例八：在模板中定义节点依赖
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template with a generic dependency between two nodes.

topology_template:
  inputs:
    # omitted here for brevity
    
  node_templates:
    my_app:
      type: my.types.MyApplication
      properties:
        # omitted here for brevity
      requirements:
        - dependency: some_service
    
  some_service:
    type: some.nodetype.SomeService
    properties:
      # omitted here for brevity
{% endhighlight %}

### 实例八：为软件定义主机基础设施要求的节点过滤
{% highlight bash %}
tosca_definitions_version: tosca_simple_yaml_1_0

description: Template with requirements against hosting infrastructure.

topology_template:
  inputs:
    # omitted here for brevity
    
  node_templates:
    mysql:
      type: tosca.nodes.DBMS.MySQL
      properties:
        # omitted here for brevity
      requirements:
        - host:
          node_filter:
            capabilities:
              # Constraints for selecting “host” (Container Capability)
              - host:
                properties:
                  - num_cpus: { in_range: [ 1, 4 ] }
                  - mem_size: { greater_or_equal: 2 GB }
              # Constraints for selecting “os” (OperatingSystem Capability)
              - os:
                properties:
                  - architecture: { equal: x86_64 }
                  - type: linux
                  - distribution: ubuntu
{% endhighlight %}



