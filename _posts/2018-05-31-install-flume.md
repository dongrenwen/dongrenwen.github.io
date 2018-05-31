---
layout:      post
title:       "Flume 1.8.0 环境单点极简配置"
categories:  [运维, 大数据, Flume]
description: "Flume 1.8.0 环境单点极简配置"
keywords:    运维, 大数据, Flume
---

### 安装 JDK 1.8

1. 准备 JDK 的安装包 `jdk-8u171-linux-x64.tar.gz`
2. 解压缩包 `tar -zxvf jdk-8u171-linux-x64.tar.gz`
3. 设置环境变量，在 `/etc/profile` 最下面追加如下内容

    ``` sh
    export JAVA_HOME=/opt/jdk1.8.0_171
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ```

4. 生效配置文件 `source /etc/profile`
### 5. 测试是否成功 `java -version`

### 安装 Flume

1. 下载 apache-flume-1.8.0-bin.tar.gz
2. 解压缩包 `tar -zxvf apache-flume-1.8.0-bin.tar.gz`

### 测试 Flume 是否有效

1. 创建配置文件 `example.conf`

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

2. 启动 Flume 

    ``` sh
    /opt/apache-flume-1.8.0-bin/bin/flume-ng agent --conf /opt/apache-flume-1.8.0-bin/conf --conf-file /root/flume/example.conf --name a1 -Dflume.root.logger=INFO,console
    ```
    
3. 新启动一个终端，并执行如下命令

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

4. 观察原来的 Flume 终端将会输出 Event 的 Log 信息

    ``` conf
    2018-05-31 14:48:36,751 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:155)] Source starting
    2018-05-31 14:48:36,789 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:166)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
    2018-05-31 14:57:54,962 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          Hello world!. }
    ```


