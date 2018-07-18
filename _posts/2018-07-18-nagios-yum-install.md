---
layout:      post
title:       "Nagios 通过 Yum 快速安装"
categories:  [运维, 监控, Nagios]
description: "Nagios 通过 Yum 快速安装"
keywords:    运维, 监控, Nagios
---

## 关闭防火墙和 SELinux

``` sh
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl stop iptables.service
systemctl disable iptables.service
setenforce 0
```

将 `/etc/selinux/config` 文件中的 `SELINUX=enforcing` 修改为 `SELINUX=disabled`

## 加载开源镜像的 epel-release

``` sh
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum repolist
yum makecache
```

## 安装 Web 环境

``` sh
yum install httpd php php-gd
```

## 安装 Nagios

``` sh
yum install nagios nagios-plugins-all
```

## 设置 Nagios Web 页面访问密码

``` sh
htpasswd -c /etc/nagios/passwd nagiosadmin
```

## 如果要使用 nrpe 进行通信，则安装 nrpe 及相关依赖

``` sh
yum install nagios-plugins-nrpe nrpe
```

## 启动 Nagios

``` sh
systemctl start nagios
```

## 启动 Web 服务

``` sh
systemctl start httpd
```


