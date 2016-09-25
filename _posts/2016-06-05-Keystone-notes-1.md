---
layout: post
title: "Keystone知识梳理(一)"
description: ""
category: Openstack
tags: [Openstack, Keystone]
---
{% include JB/setup %}

## 梳理Openstack组件Keystone知识

### 概念知识
>
- M版中Keystone版本为9.2.0，早期有9.1.0,9.0.2,9.0.1,9.0.0
- Authentication 认证，确认请求验证凭证（用户名密码、用户名APIKEY），生成认证令牌 
- Credentials 凭证【API v3】， 确认身份的依据：用户名、密码；用户名、API KEY； 认证令牌；
- Domain 域， 项目和用户的集合，可代表个人、公司、所属操作空间，域管理员可在域内创建项目、用户、组
- Endpoint 接入服务的地址（URL）
- Group 组【API v3】，用户集合，赋予给域，通过将用户加入或者移除组调整权限
- OpenstackClient Openstack服务命令行接口
- Project 项目，组、隔离资源和识别对象的容器，可以映射到客户、账户、组织、租户
- Region 区域【API v3】，Openstack部署的常规划分，不是地理概念，但是可以以地理命名
- Role 角色， 用户的权利和执行特定操作集合的权限，令牌代表着角色清单，用户调用服务时根据角色集合决定是否允许接入
- Service 服务， Openstack服务，比如计算（Nova），存储（Swift、Cinder），镜像服务（Glance）
- Token 令牌， 字母数字文本，可随时撤销，有限时间有效，集成服务不指望成为成熟的完全解决方案
- User 用户，使用Openstack服务的个人、系统、服务的数字化代表

>
- 认证服务分配项目和角色给用户
- 一个用户可以在不同的项目有不同的角色
- 在/etc/[SERVICE_CODENAME]/policy.json中定义了用户操作给定服务所需要的权限
{%highlight bash%}
#/etc/keystone/policy.json示例

{
    "admin_required": "role:admin or is_admin:1",
    "service_role": "role:service",
    "service_or_admin": "rule:admin_required or rule:service_role",
    "owner" : "user_id:%(user_id)s",
    "admin_or_owner": "rule:admin_required or rule:owner",
    "token_subject": "user_id:%(target.token.user_id)s",
    "admin_or_token_subject": "rule:admin_required or rule:token_subject",
    "service_admin_or_token_subject": "rule:service_or_admin or rule:token_subject",

    "default": "rule:admin_required",

    "identity:get_region": "",
    "identity:list_regions": "",
    "identity:create_region": "rule:admin_required",
    "identity:update_region": "rule:admin_required",
    "identity:delete_region": "rule:admin_required",

    "identity:get_service": "rule:admin_required",
    "identity:list_services": "rule:admin_required",
    "identity:create_service": "rule:admin_required",
    "identity:update_service": "rule:admin_required",
    "identity:delete_service": "rule:admin_required",

    ......
}
{%endhighlight%}
- 身份服务提供身份、令牌、目录、服务策略，

>
- Keystone Web Server Gateway Interface(WSGI) service 可以运行于WSGI容器，服务和管理API分开的WSGI服务实例
- Identity service functions 可插拔后端，支持LDAP或SQL
- keystone-all service和管理API在同一进程，在N版将移除

- 组域中用户集合，管理员创建组将用户加入，授予角色给组，在API v3引入
- 如果使用LDAP作为后端，组更新被禁用，创建、删除、更新组失败
- PKI: PUblic Key Infrastructure
- Eventlet 将在Openstack N版本移除

### Postman 调试API
{%highlight bash%}
#获取Endpoint GET
http://192.168.78.129:32773

{
    "versions": {
        "values": [
            {
                "status": "stable",
                "updated": "2016-04-04T00:00:00Z",
                "media-types": [
                    {
                        "base": "application/json",
                        "type": "application/vnd.openstack.identity-v3+json"
                    }
                ],
                "id": "v3.6",
                "links": [
                    {
                        "href": "http://192.168.78.129:32773/v3/",
                        "rel": "self"
                    }
                ]
            },
            {
                "status": "stable",
                "updated": "2014-04-17T00:00:00Z",
                "media-types": [
                    {
                        "base": "application/json",
                        "type": "application/vnd.openstack.identity-v2.0+json"
                    }
                ],
                "id": "v2.0",
                "links": [
                    {
                        "href": "http://192.168.78.129:32773/v2.0/",
                        "rel": "self"
                    },
                    {
                        "href": "http://docs.openstack.org/",
                        "type": "text/html",
                        "rel": "describedby"
                    }
                ]
            }
        ]
    }
}


#认证 POST
http://192.168.78.129:32773/v3/auth/tokens

#设置请求头
Content-Type        application/json
Accept              application/json

#请求体，官方在文档在domain中将name参数值写在id域内导致401错误
#如果user指定了name必须指定domain，如果user指定ID可以省略domain信息
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "name": "demo",
                    "domain": {
                        "name": "default"
                    },
                    "password": "cctcloud"
                }
            }
        }
    }
}

#应答
{
    "token": {
        "issued_at": "2016-09-24T15:01:46.000000Z",
        "audit_ids": [
            "mapny9-ZT1-8JJQn_dCj0A"
        ],
        "methods": [
            "password"
        ],
        "expires_at": "2016-09-24T16:01:46.831532Z",
        "user": {
            "domain": {
                "id": "13ad49a4d1b840abae23dd695544f109",
                "name": "default"
            },
            "id": "a61342c5165e46998a8abc9ae7a843b5",
            "name": "demo"
        }
    }
}

#Token 存放于X-Subject-Token
Content-Length →307
Content-Type →application/json
Vary →X-Auth-Token
X-Subject-Token →gAAAAABX5pd6BDxhG46-KXSjhres-bjqPMN7qUN0ZO1JV0CGs-Q2bfwjnozJZptE5ht2PpgEZIgNU7NkKTPqLfbD6-nXMJTewZgekhMWGqNTriGz7aTmn3nD1m8JssAW-iLiaeAZxrcxrfa7Ovm_JbkSRmqPHwAngA
x-openstack-request-id →req-2d4c2a9a-b7dd-4437-9854-2952cd724ce2

#部分Admin请求在获取Token是要求指定范围，后续请求将X-Subject-Token放入请求头中X-Auth-Token中

{%endhighlight%}

***
>
>参考链接
>
- http://docs.openstack.org/admin-guide/identity-concepts.html

