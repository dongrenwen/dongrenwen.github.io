---
layout:      post
title:       "NTP 时间服务的简单搭建说明"
categories:  [运维, NTP]
description: "NTP 时间服务的简单搭建说明"
keywords:    运维, NTP
---

# NTP 时间服务的简单搭建说明

+ 本地时间服务器(主)配置文件如下

    ```    driftfile /var/lib/ntp/drift    restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap    server 127.127.1.0    fudge 127.127.1.0 stradum 4    includefile /etc/ntp/crypto/pw    keys /etc/ntp/keys
    ```
    + 本地时间服务器(从)配置文件如下

    ```    driftfile /var/lib/ntp/drift    restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap    server center    fudge 127.127.1.0 stradum 5    includefile /etc/ntp/crypto/pw    keys /etc/ntp/keys
    ```
    + 需要同步的服务器配置如下

    ```    driftfile /var/lib/ntp/drift    server master01 prefer    server master02    includefile /etc/ntp/crypto/pw    keys /etc/ntp/keys    ``````ntpq -p update -d master01
```


