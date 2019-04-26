---
layout:      post
title:       "解决 Windows 通过 ssh 连接虚拟机中的 CentOS 7 很慢的问题"
categories:  [运维, Linux, ssh]
description: "解决 Win 10 通过 ssh 连接 VMware 中的 CentOS 7 特别慢的问题。"
keywords:    运维, Linux, ssh]
---

# 解决 Windows 通过 ssh 连接虚拟机中的 CentOS 7 很慢的问题

在 VMware 虚拟机中安装 CentOS 7 之后，通过 ssh 和 sftp 连接会特别缓慢，解决办法如下：

## 修改 sshd 配置文件

编辑 `/etc/ssh/sshd_config`，找到 `#UseDNS yes` 并在此行下面添加 `UseDNS no`

## 重启 sshd 服务

```sh
systemctl restart sshd
```
