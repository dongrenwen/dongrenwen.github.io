---
layout:      post
title:       "MySQL 5.7.25 升级到 5.7.30"
categories:  [运维, 数据库, MySQL]
description: "MySQL 5.7.25 升级到 5.7.30"
keywords:    运维, 数据库, MySQL
---

# MySQL 5.7.25 升级到 5.7.30

## MySQL 5.7.25 安装

### 创建配置文件

```properties
[mysqld]
port=3306
user=mysqltest
pid_file=/home/mysqltest/tmp/mysql.pid
socket=/home/mysqltest/tmp/mysql.sock
basedir=/home/mysqltest/base
datadir=/home/mysqltest/data

# Log 相关
log_timestamps=SYSTEM
log_error=/home/mysqltest/logs/mysql.err
log_bin=/home/mysqltest/logs/logbin/mysql-bin
log_bin_trust_function_creators=1
log_slave_updates=1
binlog_checksum=NONE
# 不开启查询日志
general_log=0
#general_log=1
#general_log_file=/home/mysqltest/logs/general.log
# 慢查询
long_query_time=1000
slow_query_log=1
slow_query_log_file=/data01/mysql_cluster/port3307/logs/slow_query.log
# 关闭记录没有使用索引的 query
log_queries_not_using_indexes=OFF

default-storage-engine=INNODB
character-set-server=utf8
lower_case_table_names=1
max_connections=1000
wait_timeout=3600
interactive_timeout=3600
# 设置最大允许写入数据包大小
max_allowed_packet=256M
# 本地数据文件的操作权限需要开启
local-infile=1
#local-infile=0
# sql_mode 删除 ONLY_FULL_GROUP_BY
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

# 配置缓冲区,此处需要根据具体内存情况变更
innodb_buffer_pool_instances=16
innodb_buffer_pool_chunk_size=134217728
innodb_buffer_pool_size=21474836480

# Cluster Server ID
server_id=100 # 200, 300 此处根据节点编号填写

# 一些同步相关的配置
gtid_mode=ON
enforce_gtid_consistency=1
relay_log_recovery=1
relay_log_info_repository=TABLE
master_info_repository=TABLE
transaction_write_set_extraction=XXHASH64
```

### 初始化数据库

#### 创建 base 链接

```bash
ln -s /home/mysqltest/mysql-5.7.25-linux-glibc2.12-x86_64 /home/mysqltest/base
```

#### 初始化 MySQL 服务

```
/home/mysqltest/base/bin/mysqld --defaults-file=/home/mysqltest/conf/my.cnf --initialize
```

#### 启动服务

```bash
#方法一
/home/mysqltest/base/bin/mysqld_safe --defaults-file=/home/mysqltest/conf/my.cnf &

#方法二
screen -dmS MySQL && screen -x -S MySQL -p 0 -X stuff $'/home/mysqltest/base/bin/mysqld_safe --defaults-file=/home/mysqltest/conf/my.cnf\n'
```

#### 登陆服务

```bash
#初次登陆，密码在初始化后存放在 /home/mysqltest/logs/mysql.err
cat /home/mysqltest/logs/mysql.err | grep "A temporary password is generated for" | awk '{print $NF}'
#执行登陆
/home/mysqltest/base/bin/mysql -S /home/mysqltest/tmp/mysql.sock -uroot -p
```

#### 修改 MySQL Root 密码

```mysql
-- 登陆后执行
set password for 'root'@'localhost'=password('root');
```

#### 开启 MySQL Root 远程访问

```mysql
rename user 'root'@'localhost' to 'root'@'%';
```

#### 关闭 MySQL 服务的方法

```bash
#方法一方式启动，执行
/home/mysqltest/base/bin/mysqladmin -S /home/mysqltest/tmp/mysql.sock -u root -p shutdown

#如果用方法二方式启动，执行方法一的内容后再执行
screen -S MySQL -X quit
```

## 升级至 5.7.30

### 关闭 MySQL 服务

```bash
/home/mysqltest/base/bin/mysqladmin -S /home/mysqltest/tmp/mysql.sock -u root -p shutdown && screen -S MySQL -X quit
```

### 替换 base 文件夹

```bash
rm /home/mysqltest/base
ln -s /home/mysqltest/mysql-5.7.30-linux-glibc2.12-x86_64 /home/mysqltest/base
```

### 启动 MySQL

**<font color="#dd0000">注：为了避免对集群产生影响，升级之前先把配置文件中的端口号变更，后期再变更回来</font>**

```bash
screen -dmS MySQL && screen -x -S MySQL -p 0 -X stuff $'/home/mysqltest/base/bin/mysqld_safe --defaults-file=/home/mysqltest/conf/my.cnf\n'
```

### 检查所有表是否与当前版本兼容并更新数据库

**<font color="#dd0000">注：mysql_upgrade的作用是检查所有库的所有表是否与当前的新版本兼容，并更新系统库。</font>**

```bash
/home/mysqltest/base/bin/mysql_upgrade -uroot -p -S /home/mysqltest/tmp/mysql.sock
```

### 重启以确保对系统表所做的变更得以生效

```bash
/home/mysqltest/base/bin/mysqladmin -S /home/mysqltest/tmp/mysql.sock -u root -p shutdown && screen -S MySQL -X quit
screen -dmS MySQL && screen -x -S MySQL -p 0 -X stuff $'/home/mysqltest/base/bin/mysqld_safe --defaults-file=/home/mysqltest/conf/my.cnf\n'
```
