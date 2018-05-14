---
layout:      post
title:       "配置 ftp yum 源服务器"
categories:  [运维, 操作系统, Linux, Yum]
description: "在新搭建的 REHL 7.2 服务器上配置 ftp yum 源。"
keywords:    运维, 操作系统, Linux, Yum
---

## 配置 ftp yum 源服务器

在新搭建的 REHL 7.2 服务器上配置 ftp yum 源。

### 关闭防火墙

``` sh
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl stop iptables.service
systemctl disable iptables.service
setenforce 0
```
    
将 `/etc/selinux/config` 文件中的 **SELINUX=enforcing** 修改为 **SELINUX=disabled**

### 挂载本地ISO源

+ 将安装镜像拷入到 `/opt/` 下

+ 创建 `/mnt/RHEL7.2ISO` 文件夹

    ``` sh
    mkdir /mnt/RHEL7.2ISO
    ```

+ 给 `/etc/rc.d/rc.local` 文件执行权限

    ``` sh
    chmod 755 /etc/rc.d/rc.local
    ```

+ 设置开机自动挂载安装镜像

    执行下面的命令，并添加到 `/etc/rc.d/rc.local` 的最下面
    
    ``` sh
    mount /opt/rhel-server-7.2-x86_64-dvd.iso /mnt/RHEL7.2ISO/
    ```
    
+ 添加文件 `/etc/yum.repos.d/iso.repo`

    ``` log
    [ISO]
    name=ISO
    baseurl=file:///mnt/RHEL7.2ISO
    enabled=1
    gpgcheck=0
    ```
    
+ 刷新 yum 缓存

    ``` sh
    yum clean all
    yum makecache
    ```
    
### 安装并启动 vsftpd

安装之后 ftp 服务的目录是：`/var/ftp`

+ 安装 vsftpd

    ``` sh
    yum install -y vsftpd
    ```
    
+ 启动 vsftpd

    ``` sh
    systemctl start vsftpd
    ```
    
+ 设置成开机自动启动

    ``` sh
    systemctl enable vsftpd
    ```
    
### 准备 yum 源内容

+ 将准备好的 `*.rpm` 拷入到 `/var/ftp/pub/`
+ 将准备好的 `RPM-GPG-KEY-CentOS-7` 拷入到 `/var/ftp/`
    
### 安装制作工具 `createrepo`

``` sh
yum install -y createrepo
```

### 制作 yum 源

``` sh
createrepo /var/ftp/pub/
```

如果增加或者修改了 rpm 包，执行更新命令：

``` sh
createrepo --update /var/ftp/pub/
```

### 添加本地 ftp 源

+ 添加文件 `/etc/yum.repos.d/ftp.repo`

    ``` log
    [ftp]
    name=ftp
    baseurl=ftp://192.168.1.55/pub
    enabled=1
    gpgcheck=0
    gpgkey=ftp://192.168.1.55/RPM-GPG-KEY-CentOS-7
    ```

+ 刷新 yum 缓存

    ``` sh
    yum clean all
    yum makecache
    ```


