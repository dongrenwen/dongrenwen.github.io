---
layout:      post
title:       "CentOS 使用普通用户安装 MySQL 5.7.25"
categories:  [运维, Linux, 数据库, MySQL]
description: "CentOS 使用普通用户安装 MySQL 5.7.25"
keywords:    运维, Linux, MySQL
---

# CentOS 使用普通用户安装 MySQL 5.7.25

1. 查看是否存在要创建的用户

    ```sh
    cat /etc/group | grep mysql
    # 出现类似下面说明存在该用户组
    mysql:x:490:
    ```
    
    ```sh
    cat /etc/passwd | grep mysql
    # 出现类似下面说明存在该用户
    mysql:x:496:490::/home/mysql:/bin/bash
    ```

2. 如果不存在则创建用户

    ```sh
    useradd mysql # 创建mysql用户
    passwd mysql # 为mysql用户设定密码
    ```
    
3. 创建目录并准备安装文件

    ```sh
    mkdir /opt/mysql
    # 将 mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz 拷贝到目录 /opt/mysql
    chown -R mysql:mysql /opt/mysql
    su - mysql
    cd /opt/mysql
    tar -zxvf mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz
    mv mysql-5.7.25-linux-glibc2.12-x86_64 mysql-5.7.25
    mkdir /opt/mysql/mysql-5.7.25/data
    mkdir /opt/mysql/mysql-5.7.25/tmp
    ```
    
4. 创建配置文件 `/opt/mysql/mysql-5.7.25/my.cnf`

    ```conf
    [mysqld]
    basedir=/opt/mysql/mysql-5.7.25
    datadir=/opt/mysql/mysql-5.7.25/data
    port=3306
    socket=/opt/mysql/mysql-5.7.25/tmp/mysql.sock
    character-set-server=utf8
    log-error=/opt/mysql/mysql-5.7.25/data/mysqld.log
    pid-file=/opt/mysql/mysql-5.7.25/tmp/mysqld.pid
    tmpdir=/opt/mysql/mysql-5.7.25/tmp
        
    [client]
    port=3306
    socket=/opt/mysql/mysql-5.7.25/tmp/mysql.sock
    ```
    
5. 安装 mysqld 服务

    ```sh
    /opt/mysql/mysql-5.7.25/bin/mysqld --user=mysql --basedir=/opt/mysql/mysql-5.7.25/ --datadir=/opt/mysql/mysql-5.7.25/data/ --initialize
    ```
    
    执行的 log 中会出现root密码，需要记住
    
    ```log
    [Note] A temporary password is generated for root@localhost: *Kkwh;g)v9)?
    ```
    
6. 启动 mysqld 服务

    ```sh
    /opt/mysql/mysql-5.7.25/bin/mysqld --defaults-file=/opt/mysql/mysql-5.7.25/my.cnf &
    ```
    
7. 开启客户端

    ```sh
    /opt/mysql/mysql-5.7.25/bin/mysql --defaults-file=/opt/mysql/mysql-5.7.25/my.cnf -uroot -p
    ```
    
8. 登录之后修改密码并开启远程登录

    ```sql
    set password=password('XXXX');
    grant all privileges on *.* to root@'%' identified by 'XXXX';
    flush privileges;
    ```
    
9. 远程登录测试

    ```sh
    mysql -h 10.252.54.163 -P 3306 -uroot -p
    ```
    
10. 关闭 mysqld 服务

    ```sh
    /opt/mysql/mysql-5.7.25/bin/mysqladmin --defaults-file=/opt/mysql/mysql-5.7.25/my.cnf -uroot -p shutdown
    ```
    