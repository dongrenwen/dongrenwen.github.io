---
layout:      post
title:       "解决 Mac Os X 升级之后出现 LC_CTYPE 警告问题"
categories:  [MacOS, ssh]
description: "解决 Mac Os X 升级之后出现 LC_CTYPE 警告问题"
keywords:    MacOS, ssh
---

# 解决 Mac Os X 升级之后出现 LC_CTYPE 警告问题

Mac 升级到 10.14 之后，登录远程 linux 服务器会出现如下错误：

```
warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory
```

由于 mac ssh 连接的时候传递了 LANG 环境变量，此变量与服务器变量不一致，导致该问题发生。

解决方法，取消传递 LANG 环境变量：

```
sudo vi /etc/ssh/ssh_config

#注释掉   SendEnv LANG LC_*
```


