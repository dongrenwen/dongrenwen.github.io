---
layout:      post
title:       "Nagios 编译安装"
categories:  [运维, 监控, Nagios]
description: "Nagios 编译安装"
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

## 安装编译所需依赖

``` sh
yum install httpd gcc glibc glibc-common gd gd-devel php php-mysql mysql mysql-devel mysql-server
```

## 创建用户和用户组

```
groupadd nagcmd
useradd -G nagcmd nagios
passwd nagios
```

## 给 apache 用户添加用户组

``` sh
usermod -a -G nagcmd apache
```

## 编译安装 Nagios

### 下载安装包

``` sh
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.1.tar.gz
```

### 解压并安装

``` sh
tar -zxvf nagios-4.4.1.tar.gz
./configure --sysconfdir=/etc/nagios --with-command-group=nagcmd --enable-event-broker
make all
make install
make install-commandmode
make install-config
make install-webconf
```

### 关于 Nagios make 命令的一些说明

``` conf
  make test
     - This runs the test suite

  make install
     - This installs the main program, CGIs, and HTML files

  make install-init
     - This installs the init script in /lib/systemd/system

  make install-daemoninit
     - This will initialize the init script
       in /lib/systemd/system

  make install-groups-users
     - This adds the users and groups if they do not exist

  make install-commandmode
     - This installs and configures permissions on the
       directory for holding the external command file

  make install-config
     - This installs *SAMPLE* config files in /etc/nagios
       You'll have to modify these sample files before you can
       use Nagios.  Read the HTML documentation for more info
       on doing this.  Pay particular attention to the docs on
       object configuration files, as they determine what/how
       things get monitored!

  make install-webconf
     - This installs the Apache config file for the Nagios
       web interface

  make install-exfoliation
     - This installs the Exfoliation theme for the Nagios
       web interface

  make install-classicui
     - This installs the classic theme for the Nagios
       web interface
```

## 编译安装 Nagios Plugins

### 下载安装包

``` sh
wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
```

### 解压并安装

``` sh
tar -zxvf nagios-plugins-2.2.1.tar.gz
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make
make install
```

## 为 Web 页面添加用户密码

``` sh
htpasswd -c /etc/nagios/htpasswd.users nagiosadmin
```

## 启动 Nagios

``` sh
systemctl start nagios
```

## 启动 Web 服务

``` sh
systemctl start httpd
```

## 访问 Nagios Web 页面

### 访问 Web 页面

[http://IP/nagios/](http://IP/nagios/)

web 页面源码目录为 `/usr/local/nagios/share/`


