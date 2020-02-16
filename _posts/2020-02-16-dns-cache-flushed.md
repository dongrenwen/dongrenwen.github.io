---
layout:      post
title:       "MacOS 刷新 DNS 缓存"
categories:  [MacOS, 运维, 操作系统, 操作技巧]
description: "MacOS 刷新 DNS 缓存"
keywords:    MacOS, 运维, 操作系统, 操作技巧
---

打开 Terminal 执行下面命令进行 DNS 缓存的刷新，需要输入用户密码。

```sh
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder; say DNS cache flushed
```