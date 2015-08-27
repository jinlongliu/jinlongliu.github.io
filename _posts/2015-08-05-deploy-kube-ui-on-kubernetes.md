---
layout: post
title: "部署 kube-ui 到 kubernetes"
description: ""
category: "Docker"
tags: ["Docker", "Kubernetes"]
---
{% include JB/setup %}
<h2>本篇博文介绍在k8s(kubernetes)上部署kube-ui</h2>
{% highlight bash %}
在部署节点进入cluster/addons可以看到kube-ui目录  
  
root@docker3:~/k8s/kubernetes/cluster/addons# ls
cluster-monitoring  dns  fluentd-elasticsearch  fluentd-gcp  kube-ui  README.md
{% endhighlight %}
<p>修改YAML配置</p>
{% highlight bash %}
root@docker3:~/k8s/kubernetes/cluster/addons/kube-ui# vim kube-ui-rc.yaml
  
内容如下
  
apiVersion: v1
kind: ReplicationController
metadata:
  name: kube-ui-v1
  namespace: kube-system
  labels:
    k8s-app: kube-ui
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  replicas: 3
  selector:
    k8s-app: kube-ui
    version: v1
  template:
    metadata:
      labels:
        k8s-app: kube-ui
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      containers:
      - name: kube-ui
        image: docker3:5000/kube-ui
        resources:
          limits:
            cpu: 100m
            memory: 50Mi
        ports:
        - containerPort: 8080
{% endhighlight %}
<p>主要修改replicas备份数默认为1，笔者修改为3<br/>
image:值，原先为google的registry，笔者修改为私有registry的镜像。<br/>
由于docker1，docker2,docker3处于类似局域网，笔者处理的方法在docker3上运行了registry的容器，然后把需要的镜像<br/>
从外网push到docker，再push到docker3的私有registry中。
</p>
<h2>修改docker私有镜像库地址</h2>
<p>
由于安全问题，私有库会报错，需要修改docker启动参数
</p>
{% highlight bash %}
root@docker3:~# vim /etc/default/docker 
DOCKER_OPTS="-H tcp://127.0.0.1:4243 -H unix:///var/run/docker.sock --bip=172.16.8.1/24 --mtu=1372 --insecure-registry=docker3:5000"
  
主要新增--insecure-registry参数
{% endhighlight %}
<p>启动一个长期运行的registry容器</p>
{% highlight bash %}
#docker run -d -p 5000:5000 --restart=always --name registry registry
{% endhighlight %}
<p>指定kubernetes的pod-infra-container-image<br/>
kube-ui部署需要一个pause镜像，默认会到谷歌下载，我们可以在kubelet配置文件中指定
</p>
{% highlight bash %}
root@docker3:~# vim /etc/default/kubelet
KUBELET_OPTS="--address=0.0.0.0 --port=10250 --pod-infra-container-image=docker3:5000/pause:latest --hostname_override=docker3 \
   --api_servers=http://docker3:8080 --logtostderr=true --cluster_dns=192.168.3.10 --cluster_domain=cluster.local"
  
主要新增参数--pod-infra-container-image=docker3:5000/pause:latest指向你本地下载pause镜像即可。
以上参数需要在所有minion修改，并重启服务
#service kubelet restart
{% endhighlight %}
<p>这样我们的准备工作就完成了，现在回到cluster/addons/目录<br/>
我们来回顾一下：<br/>
1.修改了YAML指向了本地的kube-ui镜像；<br/>
2.在所有minion节点pull了kube-ui和pause镜像；<br/>
3.修改了kubelet默认pod-infra镜像；<br/>
</p>
{% highlight bash %}
创建
root@docker3:~/k8s/kubernetes/cluster/addons# kubectl create -f kube-ui/
  
删除
root@docker3:~/k8s/kubernetes/cluster/addons# kubectl delete -f kube-ui/
  
以上操作会自动创建/删除目录下YAML配置的相关rc/svc
  
查看分发情况
root@docker3:~# kubectl get pods --all-namespaces
NAMESPACE     NAME               READY     STATUS    RESTARTS   AGE
kube-system   kube-ui-v1-7ksnt   1/1       Running   0          11h
kube-system   kube-ui-v1-gl8wc   1/1       Running   0          11h
kube-system   kube-ui-v1-pwqg4   1/1       Running   0          11h
三个minion均处于running状态
  
root@docker3:~# kubectl get rc --all-namespaces  
NAMESPACE     CONTROLLER   CONTAINER(S)   IMAGE(S)               SELECTOR                     REPLICAS
kube-system   kube-ui-v1   kube-ui        docker3:5000/kube-ui   k8s-app=kube-ui,version=v1   3
  
root@docker3:~# kubectl get svc --all-namespaces
NAMESPACE     NAME         LABELS                                                                         SELECTOR          IP(S)           PORT(S)
default       kubernetes   component=apiserver,provider=kubernetes                                        <none>            192.168.3.1     443/TCP
kube-system   kube-ui      k8s-app=kube-ui,kubernetes.io/cluster-service=true,kubernetes.io/name=KubeUI   k8s-app=kube-ui   192.168.3.114   80/TCP
  
查看Master中容器运行情况
root@docker3:~# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
8c4f610be22b        docker3:5000/kube-ui        "/kube-ui"               11 hours ago        Up 11 hours                                  k8s_kube-ui.4f327c3a_kube-ui-v1-pwqg4_kube-system_7f6280b8-4bf8-11e5-837b-fa163e264958_f0940eef
0d3236ad778c        docker3:5000/pause:latest   "/pause"                 11 hours ago        Up 11 hours                                  k8s_POD.968ce479_kube-ui-v1-pwqg4_kube-system_7f6280b8-4bf8-11e5-837b-fa163e264958_32419dcf
17a7459723e6        registry                    "/bin/sh -c 'exec doc"   11 hours ago        Up 11 hours         0.0.0.0:5000->5000/tcp   registry
{% endhighlight %}
<p>我们可以通过访问Master节点来查看UI<br/>
笔者则通过虚拟机的浮动IP来访问UI。http://192.168.51.36:8080/ui<br/>
URL会自动跳转至http://192.168.51.36:8080/api/v1/proxy/namespaces/kube-system/services/kube-ui/#/dashboard/</p>
<p>至此kube-ui就部署完成了</p>
<img src="/upload/2015/08/kube-ui.png"/>






