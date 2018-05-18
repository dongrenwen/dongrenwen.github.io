---
layout:      post
title:       "RHEL 7.2 通过网络更新系统时间"
categories:  [运维, 工具]
description: "设置开机自动通过网络更新系统时间以保证多台服务器时间统一。"
keywords:    运维, 工具
---

### 安装 `ntpdate`

``` sh
yum install -y ntpdate
```

### 同步网络时间

``` sh
ntpdate -u ntp.api.bz
```

### 设置开机自动同步网络时间

``` sh
chmod 755 /etc/rc.d/rc.local
echo 'ntpdate -u ntp.api.bz' >> /etc/rc.d/rc.local
```





