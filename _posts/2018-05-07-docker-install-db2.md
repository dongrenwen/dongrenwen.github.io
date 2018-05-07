---
layout:      post
title:       "通过Docker安装db2express-c"
categories:  [运维, 数据库, DB2]
description: "通过 Docker 安装 db2express-c 并启用服务。"
keywords:    运维, 数据库, DB2
---

# 通过Docker安装db2express-c

通过 Docker 安装 db2express-c 并启用服务。

## 从网络获取db2express-c镜像

通过 `docker search db2` 命令的返回结果可以看到包含DB2的镜像，在这里面找到我们需要的镜像

``` sh
INDEX       NAME                                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker.io   docker.io/ibmcom/db2express-c             IBM DB2 Express-C                              151 
```

然后通过链接 [https://hub.docker.com/r/ibmcom/db2express-c/](https://hub.docker.com/r/ibmcom/db2express-c/) 查看版本号，如果不关心具体版本号，可以选用 `latest` 版本。

通过 `docker pull` 命令拉取镜像

``` sh
docker pull ibmcom/db2express-c:lastest
```

## 通过命令启动db2服务

``` sh
docker run --name DB2ExpressC -d -p 50000:50000 -e DB2INST1_PASSWORD=db2inst1 -e LICENSE=accept  ibmcom/db2express-c:lastest db2start
```

+ `--name DB2ExpressC` 表示为为容器指定名称为 DB2ExpressC
+ `-d` 表示后台运行
+ `-p 50000:50000` 表示对外公开的端口为 50000
+ `-e DB2INST1_PASSWORD=db2inst1` 表示为默认用户 db2inst1 设置密码为 db2inst1
+ `-e LICENSE=accept` 表示同意默认的许可证信息
+ `db2start` 表示启动db2服务

## 安装默认实例

1. 进入到启动的容器中

    ``` sh
    docker exec -it DB2ExpressC /bin/bash
    ```

2. 切换用户到 db2inst1

    ``` sh
    su - db2inst1
    ```

3. 安装默认实例

    ``` sh
    db2sampl
    ```

4. 连接到新创建的数据库实例

    ``` sh
    db2 connect to sample
    ```

5. 执行 SQL 文确认环境的正常

    ``` sh
    db2 "SELECT * FROM STAFF"
    ```

6. 通过 `exit` 命令退出默认用户 db2inst1
7. 通过 `exit` 命令退出容器


