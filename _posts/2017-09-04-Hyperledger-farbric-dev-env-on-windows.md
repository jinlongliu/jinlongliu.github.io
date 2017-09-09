---
layout: post
title: "Hyperledger Fabric V1.0.2 Windows开发环境"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 软件硬件环境
- Windows10 x64
- ThinkPad T450 12G + 500G SSD

### 说明
- vagrant 构建虚拟开发环境的工具
- Chocolatey Windows包管理软件
- Cygwin是一个在windows平台上运行的类UNIX模拟环境
- git 版本控制软件

### 步骤
{% highlight bash %}
#以管理员权限打开Powershell【在title栏有管理员字样】，输入Get-ExecutionPolicy返回可能为 Restricted
PS C:\> Get-ExecutionPolicy
Restricted

#重新设置策略，输入Set-ExecutionPolicy AllSigned，Set-ExecutionPolicy Bypass
PS C:\Windows\system32> Set-ExecutionPolicy AllSigned

执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 http://go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): Y
PS C:\Windows\system32> Set-ExecutionPolicy Bypass

执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 http://go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): Y

#安装Chocolatey【具体方法可以百度反正要把这个装起来】
#Powershell执行以下
iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))
#安装完Chocolatey后就可以使用其命令安装Windows软件

#Vagrant和virtualbox
>cinst vagrant virtualbox

#安装Cygwin
>cinst cyg-get

#使用Cyg在Prowershell中安装其他软件
>cyg-get gnupg
>cyg-get openssh
>cyg-get rsync
>cyg-get ncurses
>cyg-get git
>cyg-get python
>cyg-get make
>cyg-get python-sphinx

#安装目录
C:\ProgramData\chocolatey
C:\ProgramData\chocolatey\lib\Cygwin

#Cygwin中执行【Cygwin有专用终端】
$/usr/bin/easy_install-2.7 pip
$pip install sphinx_rtd_theme

#Windows上安装golang的sdk下载msi安装设置系统变量GOPATH

#Cygwin里面测试go env
$ go env
set GOARCH=amd64
set GOBIN=
set GOEXE=.exe
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOOS=windows
set GOPATH=
set GORACE=
set GOROOT=C:\Go
set GOTOOLDIR=C:\Go\pkg\tool\windows_amd64
set CC=gcc
set GOGCCFLAGS=-m64 -mthreads -fmessage-length=0 -fdebug-prefix-map=C:\tools\cygwin\tmp\go-build558394994=/tmp/go-build -gno-record-gcc-switches
set CXX=g++
set CGO_ENABLED=1

{% endhighlight %}


### Windows开发环境实质
{% highlight bash %}
#windows开发环境实质上是fabric，fabric-ca,fabric-sdk-java代码存放于windows上，编译打包制作docker镜像在virtualbox的ubuntu里面
#通过vagrant脚本【源码内】将windows目录映射到ubuntu目录

#获取源码，可以从github获取，可以用Linux Foundation ID从Hyperledger的Gerrit下载
#笔者使用git windows工具从gerrit下载，然后使用JetBrains的Goland构建项目，fabric-sdk-java使用IDEA构建项目
#三个源码目录最好在一个硬盘后续需要使用相对路径，fabric,fabric-ca最好同级目录

#Cygwin终端内执行,进行fabric的devenv目录
cd fabric/devenv

#devenv目录内含有fabric-sdk-java目录可以在这个目录内放置fabric-sdk-java项目源码，或者后续通过相对路径引入
#此时fabric,fabric-ca处于同级目录，fabric-sdk-java为fabric/devenv的子目录【或者其它目录】

#cgywin内宿主机磁盘挂在cygdriver内
$ df -h
文件系统         容量  已用  可用 已用% 挂载点
C:/tools/cygwin  100G   61G   40G   61% /
D:               366G  111G  256G   31% /cygdrive/d

#编辑vagrant脚本【fabric/devenv内】，脚本配置了宿主机【我的win10】和客户机【ubuntu】的路径映射管理和端口映射
config.vm.network :forwarded_port, guest: 9000, host: 9000, id: "portainer", host_ip: "localhost", auto_correct: true # Portainer service

#将客户机9000端口映射到宿主机9000，portainer为docker的WEB管理工具，这样在宿主机访问localhost:9000就可以访问到虚拟机

#笔者修改，将相对fabric/devenv目录的fabric-sdk-java映射到客户机的devenv里，同时在宿主机的maven仓库映射到客户机
================================================================================================
# Fabric-sdk-java路径映射
  config.vm.synced_folder "../../../../../../fabric-sdk-java",
    "/opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java"
  config.vm.synced_folder "../../../../../../fabric-sdk-java/src/test/fixture/sdkintegration",
    "/opt/gopath/src/github.com/hyperledger/fabric/sdkintegration"
# Map local maven repository to virtual machine
  config.vm.synced_folder "../../../../../../../02maven_repo", "/home/ubuntu/.m2/repository"
================================================================================================ 

#启动虚拟机,在cygwin终端内devenv目录下执行
$vagrant up

#该命令会启动virtualbox下载Vagrantfile内配置的虚拟机镜像ubuntu/xenial64，启动后执行devenv/setup.sh脚本
#此时可以Ctrl+C取消因为我们可以调整一下setup.sh脚本，加速构建

#以下针对v1.0.2，主要进行客户机构建加速，如果网速很好可跳过
#国内软件源，在20行，apt-get update之前添加
# Config china software source mirror
cat >/etc/apt/sources.list <<EOF
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
EOF

#docker安装调整，安装docker-ce 17.06.2-ce
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
apt-get update -qq
apt-get install -y --allow-unauthenticated linux-image-extra-$(uname -r) apparmor docker-ce

#原67行
# Install docker-compose
#curl -L https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# Install from my VPS
curl -L http://54.193.56.94/download/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose


#原73,74行之间新增如下，docker加速
# Config china mirror
cat >/etc/docker/daemon.json <<EOF
{"registry-mirrors": ["http://0503c5a1.m.daocloud.io"]}
EOF


#原84行
#GO_URL=https://storage.googleapis.com/golang/go${GO_VER}.linux-amd64.tar.gz

# Install from my VPS
GO_URL=http://54.193.56.94/download/go1.7.5.linux-amd64.tar.gz


#原105行
#NODE_URL=https://nodejs.org/dist/v$NODE_VER/node-v$NODE_VER-linux-x64.tar.gz

# Install from my VPS
NODE_URL=http://54.193.56.94/download/node-v6.9.5-linux-x64.tar.gz


#原119行
#wget https://services.gradle.org/distributions/gradle-2.12-bin.zip -P /tmp --quiet

# Install from my VPS
wget http://54.193.56.94/download/gradle-2.12-bin.zip -P /tmp

#上述步骤中在setup.sh Line 136 make clean gotools 会导致失败，可以直接vagrant ssh进去手动执行后面几个
#失败原因编译gotools依赖部分包需要从google下载，如果网络不可导编译失败，后续脚本停止执行
{% endhighlight %}



### 构建镜像
{% highlight bash %}
#客户机启动后，在cgywin终端内连接客户机
#devenv目录
vagrant ssh

Liu@DESKTOP-AFIQ8P3 /cygdrive/d/01workspace/hyperledger-rc1/src/github.com/hyperledger/fabric/devenv
$ vagrant ssh
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-93-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


Last login: Mon Sep  4 09:38:39 2017 from 10.0.2.2
ubuntu@hyperledger-devenv:e43b68f:/opt/gopath/src/github.com/hyperledger/fabric$
ubuntu@hyperledger-devenv:e43b68f:/opt/gopath/src/github.com/hyperledger/fabric$

#构建docker镜像，客户机内安装make等构建环境，在客户机fabric目录执行
$make docker

#可能会构建失败，首先make的是fabric/gotools内的Makefile但是其依赖某个go包并未在项目内
#整个fabric的vendor是不能作用到到gotools或者说可能是部分包缺失
#上述报错可能为lint包缺失golang.org/x/tools/go/gcexportdata，这个包在fabric的vendor内是没有的

#解决方法VPN下载放进去，把缺失的包直接放置到fabirc/gotools/build/gopath/src内和github.com同级，【可以直接在宿主机拷贝】
#在gotools目录执行make，会在gotool/gopath/bin生成可执行文件，后续需要拷贝到其它目录

#继续在fabric目录执行make docker会报错缺失protoc-gen-go等文件，直接将上述gotools编译的bin下文件拷贝到对应ben文件
#主要有/fabric/build/docker/bin,/fabric/build/docker/gotools/bin
#理论上如果直接可以获取到gcexportdata，在fabirc下执行make docker会在fabric/build目录的子目录内构件gotools

#构件过程中会下载基础镜像fabric-baseimage等，所以之前设置了docker加速器

#构件结果
ubuntu@hyperledger-devenv:e43b68f:/opt/gopath/src/github.com/hyperledger/fabric$ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-ca          latest              bf6030ff7463        25 hours ago        238MB
hyperledger/fabric-ca          x86_64-1.0.2        bf6030ff7463        25 hours ago        238MB
hyperledger/fabric-tools       latest              b9e0850e6572        28 hours ago        1.33GB
hyperledger/fabric-tools       x86_64-1.0.2        b9e0850e6572        28 hours ago        1.33GB
hyperledger/fabric-couchdb     latest              d380c5dfee6b        28 hours ago        1.47GB
hyperledger/fabric-couchdb     x86_64-1.0.2        d380c5dfee6b        28 hours ago        1.47GB
hyperledger/fabric-kafka       latest              421992b5b155        29 hours ago        1.29GB
hyperledger/fabric-kafka       x86_64-1.0.2        421992b5b155        29 hours ago        1.29GB
hyperledger/fabric-zookeeper   latest              1e3e329122da        29 hours ago        1.3GB
hyperledger/fabric-zookeeper   x86_64-1.0.2        1e3e329122da        29 hours ago        1.3GB
hyperledger/fabric-testenv     latest              fc9f8605bb5d        29 hours ago        1.39GB
hyperledger/fabric-testenv     x86_64-1.0.2        fc9f8605bb5d        29 hours ago        1.39GB
hyperledger/fabric-buildenv    latest              8d62efcecd25        29 hours ago        1.31GB
hyperledger/fabric-buildenv    x86_64-1.0.2        8d62efcecd25        29 hours ago        1.31GB
hyperledger/fabric-orderer     latest              54df4bb6db20        29 hours ago        151MB
hyperledger/fabric-orderer     x86_64-1.0.2        54df4bb6db20        29 hours ago        151MB
hyperledger/fabric-peer        latest              e3b74120eaef        29 hours ago        154MB
hyperledger/fabric-peer        x86_64-1.0.2        e3b74120eaef        29 hours ago        154MB
hyperledger/fabric-javaenv     latest              1ba24601adb0        29 hours ago        1.41GB
hyperledger/fabric-javaenv     x86_64-1.0.2        1ba24601adb0        29 hours ago        1.41GB
hyperledger/fabric-ccenv       latest              4459f200ddb2        30 hours ago        1.28GB
hyperledger/fabric-ccenv       x86_64-1.0.2        4459f200ddb2        30 hours ago        1.28GB
hyperledger/fabric-baseimage   x86_64-0.3.2        c92d9fdee998        9 days ago          1.26GB
hyperledger/fabric-baseos      x86_64-0.3.2        bbcbb9da2d83        9 days ago          129MB
portainer                      1.13.6              96196eaa67b3        2 months ago        10.4MB
kafka                          0.10.2              12453f4efa7b        3 months ago        265MB
kafka                          latest              12453f4efa7b        3 months ago        265MB
hyperledger/fabric-baseos      x86_64-0.3.1        4b0cab202084        3 months ago        157MB

#fabric构件完毕后，在fabric-ca目录内执行make docker

#构件fabric-sdk-java，在开始setup.sh内已经在客户机内安装了jdk，maven，gradle
$vagrant ssh
$cd /opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java
$mvn install

#顺利编译输出，目前fabric-sdk-java还是v1.0.1【截止2017-09-04】
[INFO] Installing /opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java/target/fabric-sdk-java-1.0.1.jar to /home/ubuntu/.m2/repository/org/hyperledger/fabric-sdk-java/fabric-sdk-java/1.0.1/fabric-sdk-java-1.0.1.jar
[INFO] Installing /opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java/pom.xml to /home/ubuntu/.m2/repository/org/hyperledger/fabric-sdk-java/fabric-sdk-java/1.0.1/fabric-sdk-java-1.0.1.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:00 min
[INFO] Finished at: 2017-09-04T21:30:59+08:00
[INFO] Final Memory: 41M/388M
[INFO] ------------------------------------------------------------------------
{% endhighlight %}


### 其它参考
{% highlight bash %}
#安装python-sphinx
.\setup-x86_64.exe -q -P python-sphinx

#Chocolatey命令
cinst [packagename] 安装
clist [packagename] 查询程序是否在数据库中

#帮助
PS C:\Windows\system32> choco -h
This is a listing of all of the different things you can pass to choco.

Commands

 * list - lists remote or local packages
 * search - searches remote or local packages (alias for list)
 * info - retrieves package information. Shorthand for choco search pkgname --exact --verbose
 * install - installs packages from various sources
 * pin - suppress upgrades for a package
 * outdated - retrieves packages that are outdated. Similar to upgrade all --noop
 * upgrade - upgrades packages from various sources
 * uninstall - uninstalls a package
 * pack - packages up a nuspec to a compiled nupkg
 * push - pushes a compiled nupkg
 * new - generates files necessary for a chocolatey package from a template
 * sources - view and configure default sources (alias for source)
 * source - view and configure default sources
 * config - Retrieve and configure config file settings
 * feature - view and configure choco features
 * features - view and configure choco features (alias for feature)
 * apikey - retrieves or saves an apikey for a particular source
 * setapikey - retrieves or saves an apikey for a particular source (alias for apikey)
 * unpackself - have chocolatey set it self up
 * version - [DEPRECATED] will be removed in v1 - use `choco outdated` or `cup <pkg|all> -whatif` instead
 * update - [DEPRECATED] RESERVED for future use (you are looking for upgrade, these are not the droids you are looking
for)

#Linux开发环境只需要按照setup.sh, 因为windows开发环境的实质还是Linux开发环境，只是代码存放于windows上
{% endhighlight %}

### Fabric-sdk-java
{% highlight bash %}
#创建两个vagrant ssh

#FIRST:构建测试环境
ubuntu@hyperledger-devenv:e43b68f:/opt/gopath/src/github.com/hyperledger/fabric/sdkintegration
#对应fabric-sdk-java项目目录src/test/fixture/sdkintegration
#该目录内修改.env文件,默认为1.0.1,修改为1.0.2
$docker-compose up
CONTAINER ID        IMAGE                                     COMMAND                  CREATED              STATUS              PORTS                                            NAMES
e77cac2d62e4        hyperledger/fabric-peer:x86_64-1.0.2      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:7056->7051/tcp, 0.0.0.0:7058->7053/tcp   peer1.org1.example.com
a01670d92b08        hyperledger/fabric-peer:x86_64-1.0.2      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:8056->7051/tcp, 0.0.0.0:8058->7053/tcp   peer1.org2.example.com
b7114220bc1b        hyperledger/fabric-peer:x86_64-1.0.2      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp   peer0.org2.example.com
7e767f05b5b5        hyperledger/fabric-peer:x86_64-1.0.2      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
81f953933fed        hyperledger/fabric-orderer:x86_64-1.0.2   "orderer"                About a minute ago   Up About a minute   0.0.0.0:7050->7050/tcp                           orderer.example.com
9b7a12985544        hyperledger/fabric-tools:x86_64-1.0.2     "/usr/local/bin/co..."   About a minute ago   Up About a minute   0.0.0.0:7059->7059/tcp                           configtxlator
88dc3f728287        hyperledger/fabric-ca:x86_64-1.0.2        "sh -c 'fabric-ca-..."   About a minute ago   Up About a minute   0.0.0.0:7054->7054/tcp                           ca_peerOrg1
ab5424ab459c        hyperledger/fabric-ca:x86_64-1.0.2        "sh -c 'fabric-ca-..."   About a minute ago   Up About a minute   0.0.0.0:8054->7054/tcp                           ca_peerOrg2
34acae21e4ce        portainer:1.13.6                          "/portainer -H uni..."   37 hours ago         Up 3 hours          0.0.0.0:9000->9000/tcp                           heuristic_haibt

#SECOND:测试
ubuntu@hyperledger-devenv:e43b68f:/opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java
$mvn clean
$mvn install
[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ fabric-sdk-java ---
[INFO] Installing /opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java/target/fabric-sdk-java-1.0.1.jar to /home/ubuntu/.m2/repository/org/hyperledger/fabric-sdk-java/fabric-sdk-java/1.0.1/fabric-sdk-java-1.0.1.jar
[INFO] Installing /opt/gopath/src/github.com/hyperledger/fabric/devenv/fabric-sdk-java/pom.xml to /home/ubuntu/.m2/repository/org/hyperledger/fabric-sdk-java/fabric-sdk-java/1.0.1/fabric-sdk-java-1.0.1.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:00 min
[INFO] Finished at: 2017-09-05T12:32:25+08:00
[INFO] Final Memory: 39M/378M
[INFO] ------------------------------------------------------------------------

#集成测试End2endIT
$mvn failsafe:integration-test -DskipITs=false
Results :

Tests run: 18, Failures: 0, Errors: 0, Skipped: 2

[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent! The file encoding for reports output files should be provided by the POM property ${project.reporting.outputEncoding}.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 55.370 s
[INFO] Finished at: 2017-09-05T12:34:03+08:00
[INFO] Final Memory: 14M/174M
[INFO] ------------------------------------------------------------------------

#failsafe补充
Failsafe和maven结合，将整个集成测试分为4个阶段： 
1. pre-integration-test 启动测试环境，比如通过容器插件cargo启动容器，加载应用 
2. integration-test 执行集成测试用例， 
3. post-integration-test 清理测试环境 
4. verify 检测测试结果 

test目录下所有 “/ITCase.Java”，”/IT.java” 和”**/*IT.java” 测试用例

#pom配置
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.19.1</version>
    <configuration>
        <argLine>${failsafeArgLine}</argLine>
        <includes>
            <include>**/IntegrationSuite.java</include>
        </includes>
        <skipITs>${skipITs}</skipITs>
        <!--<argLine>-->
        <!-- -Xbootclasspath/p:${settings.localRepository}/org/mortbay/jetty/alpn/alpn-boot/${alpn-boot-version}/alpn-boot-${alpn-boot-version}.jar-->
        <!--</argLine>-->
    </configuration>
    <executions>
        <execution>
            <id>failsafe-integration-tests</id>
            <phase>integration-test</phase>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>

{% endhighlight %}

参考链接：[Java SDK for Hyperledger Fabric 1.0 ](https://bertrandszoghy.wordpress.com/2017/05/12/fabric-sdk-java/)

### 下篇预告《构建基于kafka的fabric》，尽请期待！



{% include JB/setup %}


