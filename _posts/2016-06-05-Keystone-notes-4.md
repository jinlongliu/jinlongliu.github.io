---
layout: post
title: "Keystone知识梳理(四)"
description: ""
category: Openstack
tags: [Openstack, Keystone]
---
{% include JB/setup %}

## 梳理Openstack组件Keystone知识-从0到1

### 梳理总结

>
- Keystone通过setuptools打包，pbr作为setuptools插件支撑打包，把静态配置放置于setup.cfg
- Paste Deploy作为WSGI Server和WSGI Application的桥梁
- /usr/bin/keystone-wsgi-admin，keystone-wsgi-public 由pbr自动生成，这在keystone.egg-info/entry_points.txt定义
- entry_points.txt 定义了启动的切入点如下
{%highlight bash%}
[wsgi_scripts]
keystone-wsgi-admin = keystone.server.wsgi:initialize_admin_application
keystone-wsgi-public = keystone.server.wsgi:initialize_public_application
{%endhighlight%}
- /usr/bin/keystone-wsgi-admin如下：
{%highlight bash%}
if __name__ == "__main__":
    import argparse
    import socket
    import wsgiref.simple_server as wss
    #省略
    ...
    server.serve_forever()
else:
    application = None
    app_lock = threading.Lock()

    with app_lock:
        if application is None:
            application = initialize_admin_application()
{%endhighlight%}
- /usr/bin/keystone-wsgi-public
{%highlight bash%}
application = initialize_public_application()
{%endhighlight%}
- 上述两个脚本为供httpd中配置文件调用如下
{%highlight bash%}
#vi /etc/httpd/conf.d/wsgi-keystone.conf

Listen 192.168.41.11:5000
Listen 192.168.41.11:35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
{%endhighlight%}
- 如果非httpd调用
{%highlight bash%}
# vi /usr/lib/systemd/system/openstack-keystone.service

ExecStart=/usr/bin/keystone-all
{%endhighlight%}

### Keystone的Httpd启动方式解析

>
- Httpd启动keystone-wsgi-public监听5000，启动keystone-wsgi-admin监听35357
- 脚本中启动WSGI应用通过调用keystone.server.wsgi.py中方法initialize_admin_application
- initialize_admin_application调用initialize_application
- initialize_application中调用keystone.version.service.py中loadadpp
- loadapp方法使用paste deploy加载配置文件keystone-paste.ini启动WSGI应用
- keystone-paste.ini中配置了app的集合composite:main,composite:admin
- paste机制不同的url加载到不同的pipeline，pipeline中有若干filter加最后的application承接服务请求

### 部分代码详解

#### wsgi.py
{%highlight python%}
#keystone.server.wsgi.py
#/usr/bin下两个不同脚本分别调用了不同的方法，同时调用了同一个方法传递了不同的参数“admin”，“main”

def initialize_application(name, post_log_configured_function=lambda: None):
    common.configure()

    # Log the options used when starting if we're in debug mode...
    if CONF.debug:
        CONF.log_opt_values(logging.getLogger(CONF.prog), logging.DEBUG)

    environment.use_stdlib()

    post_log_configured_function()

    def loadapp():
        return keystone_service.loadapp(
            'config:%s' % config.find_paste_config(), name)

    _unused, application = common.setup_backends(
        startup_application_fn=loadapp)
    return application


def initialize_admin_application():
    return initialize_application('admin')


def initialize_public_application():
    return initialize_application('main')

{%endhighlight%}

#### service.py
{%highlight python%}
#keystone.version.service.py
#wsgi.py中的调用参数传递到loadapp中作为loadapp的参数

def loadapp(conf, name):
    # NOTE(blk-u): Save the application being loaded in the controllers module.
    # This is similar to how public_app_factory() and v3_app_factory()
    # register the version with the controllers module.
    controllers.latest_app = deploy.loadapp(conf, name=name)
    return controllers.latest_app

{%endhighlight%}

#### keystone-paste.ini
{%highlight bash%}
#keystone-paste.ini
#loadapp中不同参数促发配置composite

[composite:main]
use = egg:Paste#urlmap
/v2.0 = public_api
/v3 = api_v3
/ = public_version_api

[composite:admin]
use = egg:Paste#urlmap
/v2.0 = admin_api
/v3 = api_v3
/ = admin_version_api

{%endhighlight%}

{%highlight bash%}
#Paste中composite会分发到pipeline中,经由若干filter处理

[pipeline:public_api]
# The last item in this pipeline must be public_service or an equivalent
# application. It cannot be a filter.
pipeline = cors sizelimit url_normalize request_id admin_token_auth build_auth_context token_auth json_body ec2_extension public_service

[pipeline:admin_api]
# The last item in this pipeline must be admin_service or an equivalent
# application. It cannot be a filter.
pipeline = cors sizelimit url_normalize request_id admin_token_auth build_auth_context token_auth json_body ec2_extension s3_extension admin_service

[pipeline:api_v3]
# The last item in this pipeline must be service_v3 or an equivalent
# application. It cannot be a filter.
pipeline = cors sizelimit url_normalize request_id admin_token_auth build_auth_context token_auth json_body ec2_extension_v3 s3_extension service_v3
{%endhighlight%}

{%highlight bash%}
#多个pipeline都包含过滤器 admin_token_auth,使用处理器为egg:keysonte#admin_token_auth,该信息可以在setuptools的相关文件entry_points.txt中找到

[filter:admin_token_auth]
# This is deprecated in the M release and will be removed in the O release.
# Use `keystone-manage bootstrap` and remove this from the pipelines below.
use = egg:keystone#admin_token_auth
{%endhighlight%}

#### entry_points.txt
{%highlight bash%}

[paste.filter_factory]
admin_token_auth = keystone.middleware:AdminTokenAuthMiddleware.factory
{%endhighlight%}

#### core.py
{%highlight python%}
#keystone.middleware/core.py
#获取Http请求头中的X-Auth-Token值

# Header used to transmit the auth token
AUTH_TOKEN_HEADER = 'X-Auth-Token'

# Header used to transmit the subject token
SUBJECT_TOKEN_HEADER = 'X-Subject-Token'

class AdminTokenAuthMiddleware(wsgi.Middleware):
    """A trivial filter that checks for a pre-defined admin token.

    Sets 'is_admin' to true in the context, expected to be checked by
    methods that are admin-only.

    """

    def __init__(self, application):
        super(AdminTokenAuthMiddleware, self).__init__(application)
        LOG.warning(_LW("The admin_token_auth middleware presents a security "
                        "risk and should be removed from the "
                        "[pipeline:api_v3], [pipeline:admin_api], and "
                        "[pipeline:public_api] sections of your paste ini "
                        "file."))

    def process_request(self, request):
        token = request.headers.get(AUTH_TOKEN_HEADER)
        context = request.environ.get(CONTEXT_ENV, {})
        context['is_admin'] = CONF.admin_token and (token == CONF.admin_token)
        request.environ[CONTEXT_ENV] = context
{%endhighlight%}

>
除了admin_token_auth还有token_auth同样取X-Auth-Token值和X-Subject-Token

{%highlight python%}
#取请求头，存入是上下文token_id中

class TokenAuthMiddleware(wsgi.Middleware):
    def process_request(self, request):
        token = request.headers.get(AUTH_TOKEN_HEADER)
        context = request.environ.get(CONTEXT_ENV, {})
        context['token_id'] = token
        if SUBJECT_TOKEN_HEADER in request.headers:
            context['subject_token_id'] = request.headers[SUBJECT_TOKEN_HEADER]
        request.environ[CONTEXT_ENV] = context
{%endhighlight%}


{%highlight bash%}
#pipeline的最后一个为application，导向app相关配置

[app:public_service]
use = egg:keystone#public_service

[app:service_v3]
use = egg:keystone#service_v3

[app:admin_service]
use = egg:keystone#admin_service
{%endhighlight%}

#### Paste中app的指向可以在entry_points.txt找到
{%highlight bash%}
#entry_points.txt中应用的配置

[paste.app_factory]
admin_service = keystone.version.service:admin_app_factory
admin_version_service = keystone.version.service:admin_version_app_factory
public_service = keystone.version.service:public_app_factory
public_version_service = keystone.version.service:public_version_app_factory
service_v3 = keystone.version.service:v3_app_factory
{%endhighlight%}

#### keystone.version.service.py
{%highlight python%}
#应用分发路由

def v3_app_factory(global_conf, **local_conf):
    controllers.register_version('v3')
    mapper = routes.Mapper()
    sub_routers = []
    _routers = []

    # NOTE(dstanek): Routers should be ordered by their frequency of use in
    # a live system. This is due to the routes implementation. The most
    # frequently used routers should appear first.
    all_api_routers = [auth_routers,
                       assignment_routers,
                       catalog_routers,
                       credential_routers,
                       identity_routers,
                       policy_routers,
                       resource_routers,
                       revoke_routers,
                       federation_routers,
                       oauth1_routers,
                       # TODO(morganfainberg): Remove the simple_cert router
                       # when PKI and PKIZ tokens are removed.
                       simple_cert_ext]

    if CONF.trust.enabled:
        all_api_routers.append(trust_routers)

    if CONF.endpoint_policy.enabled:
        all_api_routers.append(endpoint_policy_routers)

    for api_routers in all_api_routers:
        routers_instance = api_routers.Routers()
        _routers.append(routers_instance)
        routers_instance.append_v3_routers(mapper, sub_routers)

    # Add in the v3 version api
    sub_routers.append(routers.VersionV3('public', _routers))
    return wsgi.ComposingRouter(mapper, sub_routers)
{%endhighlight%}

#### auth_routers具体实现
{%highlight python%}
#路由路径添加

class Routers(wsgi.RoutersBase):

    def append_v3_routers(self, mapper, routers):
        auth_controller = controllers.Auth()

        self._add_resource(
            mapper, auth_controller,
            path='/auth/tokens',
            get_action='validate_token',
            head_action='check_token',
            post_action='authenticate_for_token',
            delete_action='revoke_token',
            rel=json_home.build_v3_resource_relation('auth_tokens'))
            ......

{%endhighlight%}

#### auth/controllers.py中业务处理
{%highlight python%}
#业务的处理实现

class Auth(controller.V3Controller):

    ......

    def authenticate_for_token(self, context, auth=None):
        """Authenticate user and issue a token."""
        include_catalog = 'nocatalog' not in context['query_string']

        try:
            auth_info = AuthInfo.create(context, auth=auth)
            auth_context = AuthContext(extras={},
                                       method_names=[],
                                       bind={})
            self.authenticate(context, auth_info, auth_context)
            if auth_context.get('access_token_id'):
                auth_info.set_scope(None, auth_context['project_id'], None)
            self._check_and_set_default_scoping(auth_info, auth_context)
            (domain_id, project_id, trust, unscoped) = auth_info.get_scope()

            method_names = auth_info.get_method_names()
            method_names += auth_context.get('method_names', [])
            # make sure the list is unique
            method_names = list(set(method_names))
            expires_at = auth_context.get('expires_at')
            # NOTE(morganfainberg): define this here so it is clear what the
            # argument is during the issue_v3_token provider call.
            metadata_ref = None

            token_audit_id = auth_context.get('audit_id')

            (token_id, token_data) = self.token_provider_api.issue_v3_token(
                auth_context['user_id'], method_names, expires_at, project_id,
                domain_id, auth_context, trust, metadata_ref, include_catalog,
                parent_audit_id=token_audit_id)

            # NOTE(wanghong): We consume a trust use only when we are using
            # trusts and have successfully issued a token.
            if trust:
                self.trust_api.consume_use(trust['id'])

            return render_token_data_response(token_id, token_data,
                                              created=True)
        except exception.TrustNotFound as e:
            raise exception.Unauthorized(e)

.....

#在authenticate_for_token最后调用此方法将令牌放入响应体的X-Subject-Token
def render_token_data_response(token_id, token_data, created=False):
    """Render token data HTTP response.

    Stash token ID into the X-Subject-Token header.

    """
    headers = [('X-Subject-Token', token_id)]

    if created:
        status = (201, 'Created')
    else:
        status = (200, 'OK')

    return wsgi.render_response(body=token_data,
                                status=status, headers=headers)

{%endhighlight%}

































