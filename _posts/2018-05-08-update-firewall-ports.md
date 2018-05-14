---
layout:      post
title:       "RHEL7.2 Firewall 操作端口的基本命令"
categories:  [运维, 操作系统, Linux, 防火墙]
description: "RHEL7.2 中关于防火墙开启端口的命令。"
keywords:    运维, 操作系统, Linux, 防火墙
---

# RHEL7.2 Firewall 操作端口的基本命令

RHEL7.2 中关于防火墙开启端口的命令。

## 开启端口

``` sh
firewall-cmd --zone=public --add-port=30001/tcp --permanent
```

## Reload 防火墙

``` sh
firewall-cmd --reload
```

## 查看端口是否开启

``` sh
firewall-cmd --zone=public --query-port=30001/tcp
```

## 关闭端口

``` sh
firewall-cmd --zone=public --remove-port=30001/tcp --permanent
```


