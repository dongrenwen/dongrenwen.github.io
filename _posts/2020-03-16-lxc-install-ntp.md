---
layout:      post
title:       "Proxmox VE 的 LXC 容器安装 NTP 时间同步"
categories:  [运维, Linux, 虚拟化]
description: "Proxmox VE 的 LXC 容器安装 NTP 时间同步"
keywords:    运维, Linux, 虚拟化
---

# Proxmox VE 的 LXC 容器安装 NTP 时间同步

## CentOS 系统修改 NTP 时间同步

#### 修改时区

```sh
# 修改时区
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

#### 安装客户端

```sh
# 安装 ntp 客户端
yum install ntp ntpdate -y
```

#### 修改配置文件 `/etc/ntp.conf` 中和 NTP 服务相关部分

```sh
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst

# Use Aliyun's ntp server as a fallback.
server ntp.aliyun.com
```

#### 将 NTP 服务设置成为开机自启

```sh
# 将 ntp 服务设置成为开机自启
systemctl enable ntpd
```

#### 启动 NTP 服务

```sh
# 启动 ntp 服务
systemctl start ntpd
```

#### 查看连接的 NTP 服务器

```sh
# 查看连接的 ntp 服务器
ntpq -p
```

## Ubuntu 系统修改 NTP 时间同步

#### 修改时区

```sh
# 修改时区
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

#### 安装客户端

```sh
# 安装 ntp 客户端
apt-get install ntp ntpdate -y
```

#### 修改配置文件 `/etc/ntp.conf` 中和 NTP 服务相关部分

```sh
# Specify one or more NTP servers.

# Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
# on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
# more information.
#pool 0.ubuntu.pool.ntp.org iburst
#pool 1.ubuntu.pool.ntp.org iburst
#pool 2.ubuntu.pool.ntp.org iburst
#pool 3.ubuntu.pool.ntp.org iburst

# Use Ubuntu's ntp server as a fallback.
#pool ntp.ubuntu.com

# Use Aliyun's ntp server as a fallback.
pool ntp.aliyun.com
```

#### 重启 NTP 服务

```sh
# 重启 ntp 服务
systemctl restart ntp
```

#### 查看连接的 NTP 服务器

```sh
# 查看连接的 ntp 服务器
ntpq -p
```