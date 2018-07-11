---
layout:      post
title:       "使用开源镜像快速安装 Ganglia 服务"
categories:  [运维, 监控, Ganglia]
description: "使用开源镜像快速安装 Ganglia 服务"
keywords:    运维, 监控, Ganglia
---

## 加载开源镜像的 epel-release

``` sh
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum repolist
yum makecache
```

## 安装 gmetad

``` sh
yum install ganglia-gmetad -y
```

## 安装 gmond

``` sh
yum install ganglia-gmond -y
```

## 安装 ganglia-web

### 安装 httpd 和 php

``` sh
yum install httpd php -y
```

### 安装 ganglia-web

1. 下载最新版本的 ganglia-web 并解压到 Web 目录

    ``` sh
    wget https://jaist.dl.sourceforge.net/project/ganglia/ganglia-web/3.7.2/ganglia-web-3.7.2.tar.gz
    tar -zxvf ganglia-web-3.7.2.tar.gz
    cp -vR ganglia-web-3.7.2 /var/www/html/ganglia
    cp /var/www/html/ganglia/conf_default.php /var/www/html/ganglia/conf.php
    ```

2. 准备 `conf.php`

    ``` sh
    cd /var/www/html/ganglia
    cp conf_default.php conf.php
    ```
    
3. 修改 `conf.php` 中的 config 目录

    ``` php
    #$conf['gweb_confdir'] = "/var/lib/ganglia-web";
    $conf['gweb_confdir'] = "/var/www/html/ganglia";
    ```

4. 在 `header.php` 中设置时区为本地时区

    ``` php
    <?php
    session_start();
    
    ini_set('date.timezone','PRC'); // 添加，-修改时区为本地时区
    
    if (isset($_GET['date_only'])) {
      $d = date("r");
      echo $d;
      exit(0);
    }
    ```

5. 准备编译目录

    ``` sh
    mkdir -m 777 /var/www/html/ganglia/dwoo/compiled
    ```

6. 准备缓存目录

    ``` sh
    mkdir -m 777 /var/www/html/ganglia/dwoo/cache
    ```

## 启动服务

### 启动 gmetad

``` sh
systemctl start gmetad
```

### 启动 gmond

``` sh
systemctl start gmond
```

### 启动 Web 服务

``` sh
systemctl start httpd
```


