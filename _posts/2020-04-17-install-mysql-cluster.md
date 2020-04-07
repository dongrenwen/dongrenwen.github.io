---
layout:      post
title:       "MySQL InnoDB Cluster 搭建 (MySQL 5.7.25)"
categories:  [运维, 数据库, MySQL]
description: "MySQL InnoDB Cluster 搭建 (MySQL 5.7.25)"
keywords:    运维, 数据库, MySQL
---

## 软件版本

+ `mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz`
+ `mysql-router-8.0.16-linux-glibc2.12-x86_64.tar.xz`
+ `mysql-shell-8.0.16-linux-glibc2.12-x86-64bit.tar.gz`

## 系统参数优化

### `/etc/sysctl.conf`

通过 `sysctl -a` 查看，重点关注下面的内容。编辑之后，通过 `sysctl -p` 启用。

```conf
net.ipv4.ip_local_port_range = 1024 65535 # 用户端口范围
net.ipv4.tcp_max_syn_backlog = 65535 # 表示 SYN 队列的长度，默认为1024，加大队列长度，可以容纳更多等待连接的网络连接数
fs.file-max = 655350 # 系统最大文件句柄
net.ipv4.tcp_syncookies = 1 # 当出现 SYN 等待队列溢出时，启用 cookies 来处理，可防范少量 SYN 攻击，默认为 0，表示关闭
net.ipv4.tcp_tw_recycle = 1 # 开启 TCP 连接中 TIME-WAIT sockets 的快速回收，默认为 0，表示关闭
net.core.somaxconn = 65535 # 每个连接的最大长度
net.core.netdev_max_backlog = 65535 # 当网络接受速率大于内核处理速率时，允许发送到队列中的包数目
net.ipv4.tcp_fin_timeout = 60 # 控制 tcp 链接等待时间 加快 tcp 链接回收
kernel.shmmax = 4294967285 # 单个共享内存段的最大值，这个数是 4G(4 x 1024 x 1024 x 1024) 大小一般为物理内存-1byte
vm.swappiness = 0 # linux 除非没有足够内存时才使用交换分区
```

### `/etc/security/limits.conf`

通过 `ulimit -n`，每次修改后重新登陆即可生效

```conf
*                soft    nofile          65535
*                hard    nofile          65535
```

## 环境准备

+ 为所有集群内主机配置不同的主机名 `hostname`、 `cat /etc/sysconfig/network`、 `cat /etc/hosts`
+ 将主机名分别添加到集群内所有主机的 `/etc/hosts` 文件
+ 安装依赖 `libaio`、 `glibc`、 `numactl.x86_64`
+ 为所有集群内主机创建安装用户 `useradd -m -s /usr/bin/bash mysqlcluster`
+ 在数据盘挂载目录创建集群安装所需要的目录 `mkdir /data01/mysql_cluster`
+ 为安装目录授权 `chown -R mysqlcluster:mysqlcluster /data01/mysql_cluster`
+ 关闭防火墙 `systemctl stop firewalld`、 `systemctl disable firewalld`
+ 关闭 selinux `setenforce 0`、 `编辑 /etc/selinux/config`、 `sestatus`、 `getenforce`
+ 配置时间同步

## MySQL 安装

### 准备安装文件

#### 创建安装目录

```sh
# 创建安装目录
mkdir /data01/mysql_cluster/package/
mkdir /data01/mysql_cluster/port3307/
mkdir /data01/mysql_cluster/port3307/data
mkdir /data01/mysql_cluster/port3307/logs
mkdir /data01/mysql_cluster/port3307/logs/logbin
mkdir /data01/mysql_cluster/port3307/source
mkdir /data01/mysql_cluster/port3307/config
mkdir /data01/mysql_cluster/port3307/tmp
```

#### 将安装包存放到安装包目录

```sh
# ls /data01/mysql_cluster/package/*
/data01/mysql_cluster/package/mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz
/data01/mysql_cluster/package/mysql-shell-8.0.16-linux-glibc2.12-x86-64bit.tar.gz
/data01/mysql_cluster/package/mysql-router-8.0.16-linux-glibc2.12-x86_64.tar.xz
```

#### 解压安装包到指定目录并创建软链接

```sh
# 准备 mysql base
tar -zxvf /data01/mysql_cluster/package/mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz -C /data01/mysql_cluster/port3307/source/

# 准备 mysql shell
tar -zxvf /data01/mysql_cluster/package/mysql-shell-8.0.16-linux-glibc2.12-x86-64bit.tar.gz -C /data01/mysql_cluster/port3307/source/

# 准备 mysql router
tar -xvJf /data01/mysql_cluster/package/mysql-router-8.0.16-linux-glibc2.12-x86_64.tar.xz -C /data01/mysql_cluster/port3307/source/

# 创建 base 软链接
ln -s /data01/mysql_cluster/port3307/source/mysql-5.7.25-linux-glibc2.12-x86_64 /data01/mysql_cluster/port3307/mysql-base

# 创建 mysql shell 软链接
ln -s /data01/mysql_cluster/port3307/source/mysql-shell-8.0.16-linux-glibc2.12-x86-64bit /data01/mysql_cluster/port3307/mysql-shell

# 创建 mysql router 软链接
ln -s /data01/mysql_cluster/port3307/source/mysql-router-8.0.16-linux-glibc2.12-x86_64 /data01/mysql_cluster/port3307/mysql-router
```

#### 编辑 MySQL 配置文件

```conf
[mysqld]
port=3307
user=mysqlcluster
pid_file=/data01/mysql_cluster/port3307/tmp/mysql.pid
socket=/data01/mysql_cluster/port3307/tmp/mysql.sock
basedir=/data01/mysql_cluster/port3307/mysql-base
datadir=/data01/mysql_cluster/port3307/data

# Log 相关
log_timestamps=SYSTEM
log_error=/data01/mysql_cluster/port3307/logs/mysql.err
log_bin=/data01/mysql_cluster/port3307/logs/logbin/mysql-bin
log_bin_trust_function_creators=1
log_slave_updates=1
binlog_checksum=NONE
# 不开启查询日志
general_log=0
#general_log=1
#general_log_file=/data01/mysql_cluster/port3307/logs/general.log
# 慢查询
long_query_time=1000
slow_query_log=1
slow_query_log_file=/data01/mysql_cluster/port3307/logs/slow_query.log
# 关闭记录没有使用索引的 query
log_queries_not_using_indexes=OFF

character-set-server=utf8
lower_case_table_names=1
max_connections=1000
wait_timeout=30
interactive_timeout=600
# 本地数据文件的操作权限需要开启
local-infile=1
#local-infile=0

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

### 分别初始化并启动集群内所有机器上的 MySQL 服务器

#### 初始化 MySQL 服务

```sh
/data01/mysql_cluster/port3307/mysql-base/bin/mysqld --defaults-file=/data01/mysql_cluster/port3307/config/my.cnf --initialize --user=mysqlcluster
```

#### 启动服务

```sh
/data01/mysql_cluster/port3307/mysql-base/bin/mysqld_safe --defaults-file=/data01/mysql_cluster/port3307/config/my.cnf --user=mysqlcluster &
```

#### 登陆服务

```sh
# 初次登陆，密码在初始化后存放在 /data01/mysql_cluster/port3307/logs/mysql.err
/data01/mysql_cluster/port3307/mysql-base/bin/mysql -S /data01/mysql_cluster/port3307/tmp/mysql.sock -uroot -p
```

#### 修改 MySQL Root 密码

```sql
--- 登陆后执行
set password for 'root'@'localhost'=password('NewMySQLRootPassword');
```

#### 开启 MySQL Root 远程访问

```sql
rename user 'root'@'localhost' to 'root'@'%';
```

#### 关闭 MySQL 服务的方法

```sh
/data01/mysql_cluster/port3307/mysql-base/bin/mysqladmin -S /data01/mysql_cluster/port3307/tmp/mysql.sock  -u root -p shutdown
```

## 集群安装

### 创建集群

#### 登陆 MySQL Shell

```sh
# 登录 mysqlsh
/data01/mysql_cluster/port3307/mysql-shell/bin/mysqlsh --uri=root@hostname-01:3301 --log-level=DEBUG3

# 配置输出等级
dba.verbose=2
```

#### 检查所有集群内的主机配置是否正确

```sh
# 检查实例配置，此处根据报错修改配置文件，修改后需要重启 MySQL
dba.checkInstanceConfiguration('root@hostname-01:3307')
dba.checkInstanceConfiguration('root@hostname-02:3307')
dba.checkInstanceConfiguration('root@hostname-03:3307')
```

#### 在其中一个节点创建集群

```sh
dba.createCluster('ClusterName')
```

#### 添加新的节点

如果加入失败，可以尝试登陆节点数据库执行 `reset master;`

```sh
var cluster = dba.getCluster('ClusterName')
cluster.addInstance('root@hostname-01:3307')
cluster.addInstance('root@hostname-02:3307')
cluster.addInstance('root@hostname-03:3307')
```

#### 持久化

__**分别登陆到每个节点的主机，再登陆 mysql-shell 进行持久化操作**__

```sh
# 登录 mysqlsh
/data01/mysql_cluster/port3307/mysql-shell/bin/mysqlsh --uri=root@hostname:3307 --log-level=DEBUG3

# 配置输出等级
dba.verbose=2

# 查看集群状态
dba.getCluster('ClusterName').status()

# 持久化到配置⽂件中
dba.configureLocalInstance()
# 根据下面提示输入 my.cnf 到完整路径
Please specify the path to the MySQL configuration file: /data01/mysql_cluster/port3307/config/my.cnf
```

## 集群维护

### 集群状态

```sh
# 方法一：
var cluster = dba.getCluster('ClusterName')
cluster.status()

# 方法二：
dba.getCluster('ClusterName').cluster.status()
```

### 获取集群结构

```sh
var cluster = dba.getCluster('ClusterName')
cluster.describe()
```

### 添加新节点

```sh
var cluster = dba.getCluster('ClusterName')
cluster.addInstance('root@hostname:3307')
```

### 重新将节点加入

```sh
var cluster = dba.getCluster('ClusterName')
cluster.rejoinInstance('root@hostname:3307')
```

### 移除节点

```sh
# 普通移除
var cluster = dba.getCluster('ClusterName')
cluster.removeInstance('root@hostname:3307')

# 强制移除
var cluster = dba.getCluster('ClusterName')
cluster.removeInstance('root@hostname:3307',{force: true})
```

### 重新启动集群

```sh
dba.rebootClusterFromCompleteOutage('ClusterName'); 
```

### 解散集群

```sh
var cluster = dba.getCluster('ClusterName')
cluster.dissolve({force:true})
```
