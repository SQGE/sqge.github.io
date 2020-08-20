---
layout: post
title: "jenkins静态slave使用nfs共享maven仓库"
subtitle: "jenkins静态slave使用nfs共享maven仓库 "
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍jenkins 使用nfs共享maven仓库，解决多slave节点打包，编译慢的情况解决。

# nfs server - jenkins master
> 共享jenkins master的`/data/apache-maven-3.1.1`

关闭防火墙
```
systemctl stop firewalld.service
systemctl disable firewalld.service
```
安装nfs客户端
```
yum -y install nfs-utils rpcbind
```
配置 nfs，默认配置文件在 /etc/exports 文件下，在该文件中添加下面的配置信息：
```
vi /etc/exports
/data/apache-maven-3.1.1  *(rw,sync,no_root_squash)
```
配置说明：

- /data/apache-maven-3.1.1 ：是共享的数据目录
- *：表示任何人都有权限连接，当然也可以是一个网段，一个 IP，也可以是域名
- rw：读写的权限
- sync：表示文件同时写入硬盘和内存
- no_root_squash：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份

启动服务 nfs 需要向 rpc 注册，rpc 一旦重启了，注册的文件都会丢失，向他注册的服务都需要重启

**注意启动顺序，先启动 rpcbind**
```
systemctl start rpcbind.service
systemctl enable rpcbind
systemctl status rpcbind
```
启动 nfs 服务
```
systemctl start nfs.service
systemctl enable nfs
systemctl status nfs
```
另外我们还可以通过下面的命令确认下：
```
rpcinfo -p|grep nfs
    
100003    3   tcp   2049  nfs
100003    4   tcp   2049  nfs
100227    3   tcp   2049  nfs_acl
100003    3   udp   2049  nfs
100003    4   udp   2049  nfs
100227    3   udp   2049  nfs_ac
```
查看具体目录挂载权限：
```
cat /var/lib/nfs/etab
/data/apache-maven-3.1.1	*(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

# nfs client -  jenkins slave
安装 nfs
```
yum -y install nfs-utils rpcbind
```
安装完成后，和上面的方法一样，先启动 rpc、然后启动 nfs：
```
systemctl start rpcbind.service 
systemctl enable rpcbind.service 
systemctl start nfs.service    
systemctl enable nfs.service
```
```
showmount -e 10.0.0.2

Export list for 10.0.0.2:
/data/apache-maven-3.1.1 
```
然后我们在客户端上新建需要挂载的目录：
```
mkdir /data/apache-maven-3.1.1
```

将 nfs 共享目录挂载到上面的目录：
```
mount -t nfs 10.0.0.2:/data/apache-maven-3.1.1 /data/apache-maven-3.1.1
```
# 开启自动挂载NFS
如果机器重启是不会自动挂载nfs的，为了自动可以设置开启启动
```
vim /etc/fstab
```
```
10.0.0.2:/data/apache-maven-3.1.1   /data/apache-maven-3.1.1           nfs     defaults        0 0
```
##### 转载请注明出处，本文采用 CC4.0 协议授权
