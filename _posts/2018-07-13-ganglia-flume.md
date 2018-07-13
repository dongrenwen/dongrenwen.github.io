---
layout:      post
title:       "Ganglia 配置 Flume 集群监控"
categories:  [运维, 监控, Ganglia, Flume]
description: "Ganglia 配置 Flume 集群监控"
keywords:    运维, 监控, Ganglia, Flume
---

## 服务器说明

`192.168.0.24` ganglia 相关服务, flume 服务
`192.168.0.21` flume 服务
`192.168.0.22` flume 服务
`192.168.0.23` flume 服务

## 安装 Ganglia 相关服务

### 加载开源镜像的 epel-release

``` sh
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum repolist
yum makecache
```

### 安装 gmetad

``` sh
yum install ganglia-gmetad -y
```

### 安装 gmond

``` sh
yum install ganglia-gmond -y
```

### 安装 ganglia-web

#### 安装 httpd 和 php

``` sh
yum install httpd php -y
```

#### 安装 ganglia-web

1. 下载最新版本的 ganglia-web 并解压到 Web 目录

    ``` sh
    wget https://jaist.dl.sourceforge.net/project/ganglia/ganglia-web/3.7.2/ganglia-web-3.7.2.tar.gz
    tar -zxvf ganglia-web-3.7.2.tar.gz
    cp -vR ganglia-web-3.7.2 /var/www/html/ganglia
    ```

2. 准备 `conf.php`

    ``` sh
    cp /var/www/html/ganglia/conf_default.php /var/www/html/ganglia/conf.php
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

### 更改相关配置文件

#### 修改 gmetad 配置文件

修改配置文件 `/etc/ganglia/gmetad.conf` 中的 `data_source`

``` conf
#data_source "my cluster" localhost
data_source "flume" 1 192.168.0.24:8650
```

#### 修改 gmond 配置文件

修改配置文件 `/etc/ganglia/gmond.conf` 采用单播模式

``` conf
/*
 * The cluster attributes specified will be used as part of the <CLUSTER>
 * tag that will wrap all hosts collected by this instance.
 */
cluster {
  name = "flume"
  owner = "unspecified"
  latlong = "unspecified"
  url = "unspecified"
}

/* The host section describes attributes of the host, like the location */
host {
  location = "unspecified"
}

/* Feel free to specify as many udp_send_channels as you like.  Gmond
   used to only support having a single channel */
udp_send_channel {
  #bind_hostname = yes # Highly recommended, soon to be default.
                       # This option tells gmond to use a source address
                       # that resolves to the machine's hostname.  Without
                       # this, the metrics may appear to come from any
                       # interface and the DNS names associated with
                       # those IPs will be used to create the RRDs.
  #mcast_join = 239.2.11.71
  host = 192.168.0.24
  port = 8650
  ttl = 1
}

/* You can specify as many udp_recv_channels as you like as well. */
udp_recv_channel {
  #mcast_join = 239.2.11.71
  port = 8650
  #bind = 239.2.11.71
  #retry_bind = true
  # Size of the UDP buffer. If you are handling lots of metrics you really
  # should bump it up to e.g. 10MB or even higher.
  # buffer = 10485760
}

/* You can specify as many tcp_accept_channels as you like to share
   an xml description of the state of the cluster */
tcp_accept_channel {
  port = 8650
  # If you want to gzip XML output
  gzip_output = no
}

/* Channel to receive sFlow datagrams */
#udp_recv_channel {
#  port = 6343
#}

/* Optional sFlow settings */
#sflow {
# udp_port = 6343
# accept_vm_metrics = yes
# accept_jvm_metrics = yes
# multiple_jvm_instances = no
# accept_http_metrics = yes
# multiple_http_instances = no
# accept_memcache_metrics = yes
# multiple_memcache_instances = no
#}

```

### 启动服务

#### 启动 gmetad

``` sh
systemctl start gmetad
```

#### 启动 gmond

``` sh
systemctl start gmond
```

#### 启动 Web 服务

``` sh
systemctl start httpd
```

## 安装测试用的 Flume 相关服务

### 安装 JDK

1. 准备 JDK 的安装包 `jdk-8u171-linux-x64.tar.gz` 到 `/opt/` 
2. 解压缩包 `tar -zxvf jdk-8u171-linux-x64.tar.gz` 
3. 设置环境变量，在 `/etc/profile` 最下面追加如下内容

    ``` sh
    export JAVA_HOME=/opt/jdk1.8.0_171
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ```

4. 生效配置文件 `source /etc/profile`
5. 测试是否成功 `java -version`

### 安装 Flume

1. 下载 `apache-flume-1.8.0-bin.tar.gz` 到 `/opt/`
2. 解压缩包 `tar -zxvf apache-flume-1.8.0-bin.tar.gz`

### 将 Flume 与 Ganglia 进行关联

1. 在 `/opt/apache-flume-1.8.0-bin/conf/` 目录下创建配置文件 `example.conf`

    ``` sh
    # example.conf: A single-node Flume configuration
    
    # Name the components on this agent
    a1.sources = r1
    a1.sinks = k1
    a1.channels = c1
    
    # Describe/configure the source
    a1.sources.r1.type = netcat
    a1.sources.r1.bind = localhost
    a1.sources.r1.port = 44444
    
    # Describe the sink
    a1.sinks.k1.type = logger
    
    # Use a channel which buffers events in memory
    a1.channels.c1.type = memory
    a1.channels.c1.capacity = 1000
    a1.channels.c1.transactionCapacity = 100
    
    # Bind the source and sink to the channel
    a1.sources.r1.channels = c1
    a1.sinks.k1.channel = c1
    ```
    
2. 准备 `flume-env.sh`

    `cp /opt/apache-flume-1.8.0-bin/conf/flume-env.sh.template /opt/apache-flume-1.8.0-bin/conf/flume-env.sh`

    ``` sh
    # Enviroment variables can be set here.

    export JAVA_HOME=/opt/jdk1.8.0_171

    # Give Flume more memory and pre-allocate, enable remote monitoring via JMX
    # export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"
    export JAVA_OPTS="-Xms100m -Xmx2000m -Dflume.monitoring.type=ganglia -Dflume.monitoring.hosts=192.168.0.24:8650"
    ```


3. 启动 Flume 

    启动时加入参数 `-Dflume.monitoring.type=ganglia -Dflume.monitoring.hosts=192.168.0.24:8650`

    ``` sh
    $ /opt/apache-flume-1.8.0-bin/bin/flume-ng agent --conf /opt/apache-flume-1.8.0-bin/conf --conf-file /opt/apache-flume-1.8.0-bin/conf/example.conf --name a1 -Dflume.root.logger=INFO,console -Dflume.monitoring.type=ganglia -Dflume.monitoring.hosts=192.168.0.24:8650
    ```
    
4. 新启动一个终端，并执行如下命令

    ``` sh
    $ telnet localhost 44444
    Trying ::1...
    telnet: connect to address ::1: Connection refused
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    Hello world! <ENTER>
    OK
    ```

## 访问 Ganglia Web 展示页面

``` url
http://192.168.0.24/ganglia/
```

## 参考资料

+ [Flume User Guide - Monitoring](https://flume.apache.org/FlumeUserGuide.html#monitoring)
+ [Ganglia Users Guide](http://flash.ssc.wisc.edu/roll-documentation/ganglia/6.1.1/roll-ganglia-usersguide.pdf)

