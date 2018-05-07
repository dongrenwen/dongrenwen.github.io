---
layout:      post
title:       "通过 Docker 搭建 MySQL 服务"
categories:  [运维, 数据库, MySQL, 虚拟化, Docker]
description: "通过 Docker 创建 MySQL 容器，再 link 一个客户端进行管理。"
keywords:    运维, 数据库, MySQL, 虚拟化, Docker
---

# 通过 Docker 搭建 MySQL 服务

通过 Docker 创建 MySQL 容器，再 link 一个客户端进行管理。

## 下载 MySQL 镜像

``` sh
docker pull 192.168.1.51:5000/mysql:5.7.6
```

## 启动 MySQL 服务

将 MySQL 的数据目录挂载到本地卷

``` sh
docker run --name some-mysql -v /root/mysql/datadir:/var/lib/mysql --privileged=true -e MYSQL_ROOT_PASSWORD=rootpasswd -d 192.168.1.51:5000/mysql:5.7.6
```

## Link 一个客户端对数据库进行管理

``` sh
docker run -it --link some-mysql:mysql --rm 192.168.0.24:5000/mysql:5.7.6 sh -c 'exec mysql -h "$MYSQL_PORT_3306_TCP_ADDR" -P "$MYSQL_PORT_3306_TCP_PORT" -u root -p "$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
```


