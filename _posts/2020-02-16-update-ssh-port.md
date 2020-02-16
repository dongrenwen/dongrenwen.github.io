---
layout:      post
title:       "CentOS 7 修改 SSH 默认端口"
categories:  [运维, 操作系统, Linux]
description: "CentOS 7 修改 SSH 默认端口"
keywords:    运维, 操作系统, Linux
---

1. 安装依赖
    
    ```sh
    # 安装依赖
    yum install policycoreutils-python
    ```
    
1. 安装semanage
    
    ```sh
    # 安装semanage
    yum provides semanage
    ```
 
1. 查询当前 SSH 服务端口

    ```sh
    # 查询当前 SSH 服务端口
    semanage port -l | grep ssh
    ```
    
1. 向 SELinux 中添加我们需要添加的ssh端口(666)

    ```sh
    # 向 SELinux 中添加我们需要添加的ssh端口(666)
    semanage port -a -t ssh_port_t -p tcp 666
    ```
    
1. 查询修改是否成功

    ```sh
    # 查询修改是否成功
    semanage port -l | grep ssh
    ```
    
1. 在 firewall 防火墙中开放端口

    ```sh
    firewall-cmd --permanent --zone=public --add-port=666/tcp
    ```
    
1. 重新加载 firewall 防火墙
    
    ```sh
    firewall-cmd --reload
    ```
    
1. 查看 firewall 添加的开放端口是否成功

    ```sh
    firewall-cmd --zone=public --list-ports
    ```
    
1. 修改 `/etc/ssh/sshd_config` 中的 `#Port 22` 为新的端口号 `Port 666`

1. 重启 SSH 服务

    ```sh
    # 重启 SSH 服务
    systemctl restart sshd
    ```