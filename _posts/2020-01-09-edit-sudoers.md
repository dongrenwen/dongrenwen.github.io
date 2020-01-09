---
layout:      post
title:       "CentOS 配置普通用户使用 sudo 获取临时 root 权限"
categories:  [运维, Linux]
description: "CentOS 配置普通用户使用 sudo 获取临时 root 权限"
keywords:    运维, Linux
---

# CentOS 配置普通用户使用 sudo 获取临时 root 权限

### 事件缘由

普通用户在使用 `yum` 或者 `systemctl` 命令时，会提示需要 root 权限执行，这时候就可以使用 `sudo` 命令临时获取 root 权限执行命令。

### 配置步骤

1. 切换到 root 用户
2. 编辑 `/etc/sudoers` 文件，在 `root ALL=(ALL) ALL` 的下一行添加 `User(普通用户名) ALL=(ALL) ALL`
3. 如果不希望每次都要输入密码，则把添加行改成 `User(普通用户名) ALL=(ALL) NOPASSWD:ALL`
