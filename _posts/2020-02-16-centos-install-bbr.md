---
layout:      post
title:       "CentOS 7 安装 Google BBR 加速"
categories:  [运维, Linux]
description: "CentOS 7 安装 Google BBR 加速"
keywords:    运维, Linux
---

BBR 是 Google 提出的一种新型拥塞控制算法，可以使 Linux 服务器显著地提高吞吐量和减少 TCP 连接的延迟。

[BBR 项目地址 https://github.com/google/bbr](https://github.com/google/bbr)

Google BBR 加速安装脚本互联网上还是有很多的，下面这个是测试通过的。

CentOS 7 安装 BBR 加速，快速安装脚本：

```
wget -O- http://soft.wellphp.com/scripts/install_bbr_centos.sh | bash
```

执行之后重起服务器，然后通过下面两条命令检查是否安装成功：

```
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```