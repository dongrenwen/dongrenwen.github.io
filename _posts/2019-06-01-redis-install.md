---
layout:      post
title:       "Redis 4.0.11 安装"
categories:  [运维, Linux, Redis]
description: "CentOS 7 安装 Redis 4.0.11"
keywords:    运维, Linux, Redis
---

# Redis 4.0.11 安装

## 在每台主机上部署 Redis

假设有三台主机，因为创建集群至少需要6个节点，所以每个主机有两个节点，下面的安装只演示一个节点的部署，其他类似。

+ redis-host-1
+ redis-host-2
+ redis-host-3

### 下载软件包

```sh
wget http://39.137.67.79/download.redis.io/releases/redis-4.0.11.tar.gz
```

### 解压软件包

```sh
tar -zxvf redis-4.0.11.tar.gz -C /usr/local/
```

### 编译

```sh
make && make test
```

#### 编译遇到问题处理

1. 没有安装 `gcc`

    ```sh
    yum install gcc -y
    ```

1. 没有安装 `tcl`

    ```sh
    yum install tcl -y
    ```
    
1. 如果遇到编译库问题，可以尝试将 `make` 改为 `make MALLOC=libc` 进行编译

### 准备目录并修改配置文件

```sh
mkdir /data01/redis_cluster
mkdir /data01/redis_cluster/6379
cp /usr/local/redis-4.0.11/redis.conf /data01/redis_cluster/6379/
```

修改 `/data01/redis_cluster/6379/redis.conf` 中的以下几个参数

```config
bind 0.0.0.0
port 6379
daemonize yes
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 5000
appendonly yes
pidfile /var/run/redis_6379.pid
logfile "./redis-6379.log"
```

### 启动实例

__注意:__ 一定要在指定目录中执行!!!

```sh
cd /data01/redis_cluster/6379/
/usr/local/redis-4.0.11/src/redis-server /data01/redis_cluster/6379/redis.conf
ps -ef | grep redis
```

## 在进行集群的创建

### 创建集群

```sh
/usr/local/redis-4.0.11/src/redis-trib.rb create --replicas 1 redis-host-1:6379 redis-host-2:6379 redis-host-3:6379 redis-host-1:6380 redis-host-2:6380 redis-host-3:6380
```

__备注:__ reids会自动指定主从节点

#### 创建集群遇到问题处理

1. 没有安装 ruby

    ```sh
    yum install ruby -y
    ```
    
1. 缺少 rubygems 组件

    ```sh
    yum install rubygems -y
    ```
    
1. 缺少 ruby 和 redis 的接口

    出现类似下面的提示
    
    ```sh
    /usr/share/rubygems/rubygems/core_ext/kernel_require.rb:55:in 'require':cannot load such file -- redis (LoadError)
    ```
    
    下载 `https://rubygems.org/downloads/redis-3.3.5.gem`
    
    参考:
    
    ```
    redis-3.0.0.gem官网：https://rubygems.org/gems/redis/versions/3.0.0
    redis-3.0.0.gem下载网址：https://rubygems.org/downloads/redis-3.0.0.gem
    redis-3.3.0.gem官网：https://rubygems.org/gems/redis/versions/3.3.0
    redis-3.3.3.gem官网：https://rubygems.org/gems/redis/versions/3.3.3
    redis-3.3.5.gem官网：https://rubygems.org/gems/redis/versions/3.3.5
    redis-3.3.5.gem下载网址：https://rubygems.org/downloads/redis-3.3.5.gem
    redis-4.0.1.gem官网：https://rubygems.org/gems/redis/versions/4.0.1
    ```
    
    安装
    
    ```sh
    gem install -l redis-3.3.5.gem
    ```
    
### 测试集群

连接到集群，可以连接任意节点的 6379 或者 6380

```sh
/usr/local/redis-4.0.11/src/redis-cli -h redis-host-1 -p 6379
```

查看集群状态

```sh
redis-host-1:6379> cluster info
...
redis-host-1:6379> cluster nodes
```