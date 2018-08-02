---
layout:      post
title:       "RHEL 6.5 使用 mkfs.xfs"
categories:  [运维]
description: "RHEL 6.5 使用 mkfs.xfs"
keywords:    运维
---

RHEL 6.5 由于默认没有 `mkfs.xfs` 命令，所以想要将磁盘格式化成为 `xfs` 格式需要先安装格式化工具

1. 使用 yum 安装

    ```sh
    yum install xfsprogs -y
    ```
    
2. 直接使用安装光盘中的 rpm 包安装

    ```sh
    rpm -ivh xfsprogs-3.1.1-14.el6.x86_64.rpm
    ```

