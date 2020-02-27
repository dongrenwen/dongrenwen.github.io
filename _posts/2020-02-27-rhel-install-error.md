---
layout:      post
title:       "RHEL 7.2 U 盘镜像无法正常安装系统解决办法"
categories:  [运维, Linux]
description: "RHEL 7.2 U 盘镜像无法正常安装系统解决办法"
keywords:    运维, Linux
---

# RHEL 7.2 U 盘镜像无法正常安装系统解决办法

### 问题现象

不能够正常安装，并出现 `dracut:/#`

### 解决办法

重启之后，在 install 页面按 e 键修改

```
inst.stage2=hd:LABEL=CentOS\x207\x20x86_64.check quiet
vmlinuz initrd=initrd.img
```

修改为

```
inst.stage2=hd:/dev/sdb4(你U盘所在) quiet
vmlinuz initrd=initrd.img
```

修改完成之后按 `Ctrl + x` 就可以正常安装了