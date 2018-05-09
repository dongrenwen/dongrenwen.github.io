---
layout:      post
title:       "RHEL 7.2 安装之后"
categories:  [运维, 操作系统, Linux]
description: "RHEL 7.2 安装之后进行的一些简单配置。"
keywords:    运维, 操作系统, Linux
---

# RHEL 7.2 安装之后

RHEL 7.2 安装之后进行的一些简单配置。

## 设置固定 IP

+ 查询网络状态 `mncli dev`
+ 编辑 `/etc/sysconfig/network-scripts` 目录下的 `ifcfg-eno16777736` 文件

    ``` sh
    BOOTPROTO=static #启用静态IP地址  若是dhcp 则是动态获取ip
    ONBOOT=yes   #开启自动启用网络连接
    IPADDR=地址
    NETMASK=掩码
    GATEWAY=网关
    DNS1=DNS地址
    ```
    
+ 通过 `systemctl restart network` 重启网络服务

## 挂载系统镜像并搭建本地云

+ 下载安装用的 iso 镜像到 `/opt/`
+ 执行 `mount /opt/rhel-server-7.2-x86_64-dvd.iso /mnt/` 挂载镜像
+ 创建文件 `/etc/yum.repos.d/iso.repo` ，内容为

    ``` sh
    [ISO]
    name=ISO
    baseurl=file:///mnt
    enabled=1
    gpgcheck=0
    ```
    
## 通过 yum 安装 vsftpd
    
``` sh
yum install vsftpd
systemctl start vsftpd
```



