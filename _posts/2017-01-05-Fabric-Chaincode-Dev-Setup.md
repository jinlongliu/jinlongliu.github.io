---
layout: post
title: "Fabric V0.6.1 Chaincode开发环境设置"
description: ""
category: 区块链 
tags: [区块链, Fabric, Chaincode]
---
{% include JB/setup %}
<p>
本文记录介绍Fabric V0.6 Chaincode开发环境搭建
</p>

### 准备介绍
- Windows 7 64bit    192.168.1.106
- VMware Workstation 12.1 : Ubuntu 14.04 64bit  192.168.78.130
- Ubuntu 14.04 由iso安装，安装之后装了vim
- vmware的ubuntu最好安装vmware-tools 这样你本地的maven仓库可以共享到虚拟机，
然后到容器，这样在编译java chaincode时可以复用本地包

#安装docker
- 可以通过网络百度安装方法，Fabric开发官方推荐是1.12+
- 或者通过下载本文中的安装附件，在dockerDEB目录中通过dpkg -i *.deb

#安装docker-compose
- 安装pip，在pipDEB目录中dpkg -i *.deb
- pip install --upgrade pip
- pip install behave nose docker-compose

#下载docker镜像，可以从官方pull
- docker pull hyperledger/fabric-peer:latest
- docker pull hyperledger/fabric-membersrvc:latest
- 在下载附件FabricDockerImages目录中通过命令导入
- docker load < fabric-peer.tar
- docker load < fabric-membersrvc.tar

#编辑docker-compose文件
- 在deploy目录vim docker-compose.yml
{%highlight bash%}
membersrvc:
  image: hyperledger/fabric-membersrvc
  ports:
    - "7054:7054"
  command: membersrvc
vp0:
  image: hyperledger/fabric-peer
  ports:
    - "7050:7050"
    - "7051:7051"
    - "7053:7053"
  volumes:
    - /root/deploy/share:/mnt
    - /mnt/hgfs:/mnt2
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_VM_ENDPOINT=unix:///var/run/docker.sock
    - CORE_LOGGING_LEVEL=DEBUG
    - CORE_PEER_ID=vp0
    - CORE_PEER_PKI_ECA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TCA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TLSCA_PADDR=membersrvc:7054
    - CORE_SECURITY_ENABLED=true
    - CORE_SECURITY_ENROLLID=test_vp0
    - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
  links:
    - membersrvc
  command: sh -c "sleep 5; peer node start --peer-chaincodedev"
{%endhighlight%}




