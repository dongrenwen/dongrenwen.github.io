---
layout:      post
title:       "Flume 各种 Source 功能测试"
categories:  [运维, 大数据, Flume]
description: "Flume 各种 Source 功能测试"
keywords:    运维, 大数据, Flume
---

## 环境说明

1. 操作系统

    `rhel-server-7.2-x86_64`

2. JDK 版本

    `jdk-8u171-linux-x64`

3. Flume 版本

    `apache-flume-1.8.0-bin`

4. Flume 安装目录

    `/usr/local/flume`
    
5. 安装的软件

    + `telnet`
    + `nc`
    
## Flume 单机测试 Demo

### Demo 1 netcat

source 为 `netcat`, sink 为控制台输出, channel 为 `memory`。

#### Demo 1 配置文件

``` conf
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

#### Demo 1 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/example.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 1 测试

开启新的终端，并执行下面的命令

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

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 10:23:18,375 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:155)] Source starting
2018-06-01 10:23:18,439 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:166)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
2018-06-01 10:27:10,061 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          Hello world!. }
```

### Demo 2 avro

source 为 `avro`, sink 为控制台输出, channel 为 `memory`。

#### Demo 2 配置文件

``` conf
# demo2-avro.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = avro
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

#### Demo 2 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/demo2-avro.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 2 测试

准备测试文件

``` sh
$ echo 'Hello world!' > /usr/tmp/log.10
```

使用 avro-client 发送文件

``` sh
$ /usr/local/flume/bin/flume-ng avro-client --conf /usr/local/flume/conf --host localhost --port 44444 -F /usr/tmp/log.10
```

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 10:42:15,043 (New I/O server boss #3) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x4ed0c749, /127.0.0.1:60921 => /127.0.0.1:44444] OPEN
2018-06-01 10:42:15,045 (New I/O worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x4ed0c749, /127.0.0.1:60921 => /127.0.0.1:44444] BOUND: /127.0.0.1:44444
2018-06-01 10:42:15,045 (New I/O worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x4ed0c749, /127.0.0.1:60921 => /127.0.0.1:44444] CONNECTED: /127.0.0.1:60921
2018-06-01 10:42:15,552 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21             Hello world! }
2018-06-01 10:42:15,585 (New I/O worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x4ed0c749, /127.0.0.1:60921 :> /127.0.0.1:44444] DISCONNECTED
2018-06-01 10:42:15,586 (New I/O worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x4ed0c749, /127.0.0.1:60921 :> /127.0.0.1:44444] UNBOUND
2018-06-01 10:42:15,586 (New I/O worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x4ed0c749, /127.0.0.1:60921 :> /127.0.0.1:44444] CLOSED
2018-06-01 10:42:15,586 (New I/O worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.channelClosed(NettyServer.java:209)] Connection to /127.0.0.1:60921 disconnected.
```

### Demo 3 exec

source 为 `exec`, sink 为控制台输出, channel 为 `memory`。

#### Demo 3 配置文件

``` conf
# demo3-exec.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /usr/tmp/log.10

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

#### Demo 3 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/demo3-exec.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 3 测试

启动之前准备测试文件

``` sh
$ echo 'Hello world!' > /usr/tmp/log.10
```

在启动之后向文件中追加内容

``` sh
$ echo 'Hello Flume!' >> /usr/tmp/log.10
```

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 10:56:53,280 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21             Hello world! }
2018-06-01 10:56:58,090 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 48 65 6C 6C 6F 20 46 6C 75 6D 65 21             Hello Flume! }
```

### Demo 4 spooldir

source 为 `spooldir`, sink 为控制台输出, channel 为 `memory`。

#### Demo 4 配置文件

``` conf
# demo4-spooldir.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /usr/tmp/flumespooldir
a1.sources.r1.fileHeader = true

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

#### Demo 4 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/demo4-spooldir.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 4 测试

添加测试文件

``` sh
$ echo 'Hello world!' > /usr/tmp/flumespooldir/log.10
$ echo 'Hello Flume!' > /usr/tmp/flumespooldir/log.11
```

__注意__：用这种方式监视文件夹只能出现不同文件名的文件，且源文件名会被重新命名。

``` sh
$ ls /usr/tmp/flumespooldir/
log.10.COMPLETED  log.11.COMPLETED
```

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 11:20:49,136 (pool-3-thread-1) [INFO - org.apache.flume.client.avro.ReliableSpoolingFileEventReader.readEvents(ReliableSpoolingFileEventReader.java:324)] Last read took us just up to a file boundary. Rolling to the next file, if there is one.
2018-06-01 11:20:49,136 (pool-3-thread-1) [INFO - org.apache.flume.client.avro.ReliableSpoolingFileEventReader.rollCurrentFile(ReliableSpoolingFileEventReader.java:433)] Preparing to move file /usr/tmp/flumespooldir/log.10 to /usr/tmp/flumespooldir/log.10.COMPLETED
2018-06-01 11:20:52,142 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{file=/usr/tmp/flumespooldir/log.10} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21             Hello world! }
2018-06-01 11:20:57,170 (pool-3-thread-1) [INFO - org.apache.flume.client.avro.ReliableSpoolingFileEventReader.readEvents(ReliableSpoolingFileEventReader.java:324)] Last read took us just up to a file boundary. Rolling to the next file, if there is one.
2018-06-01 11:20:57,171 (pool-3-thread-1) [INFO - org.apache.flume.client.avro.ReliableSpoolingFileEventReader.rollCurrentFile(ReliableSpoolingFileEventReader.java:433)] Preparing to move file /usr/tmp/flumespooldir/log.11 to /usr/tmp/flumespooldir/log.11.COMPLETED
2018-06-01 11:20:57,176 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{file=/usr/tmp/flumespooldir/log.11} body: 48 65 6C 6C 6F 20 46 6C 75 6D 65 21             Hello Flume! }
```

### Demo 5 syslogtcp

source 为 `syslogtcp`, sink 为控制台输出, channel 为 `memory`。

#### Demo 5 配置文件

``` conf
# demo5-syslogtcp.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = syslogtcp
a1.sources.r1.host = localhost
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

#### Demo 5 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/demo5-syslogtcp.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 5 测试

产生测试数据，

``` sh
$ echo '<37>Hello world!' | nc localhost 44444
```

__注意__：需要在前面添加 `<37>` 来进行 write format 数据，否则会报警告 “Event created from Invalid Syslog data.”

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 11:35:02,139 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{Severity=5, Facility=4} body: 3C 33 37 3E 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 <37>Hello world! }
```

不在前面添加 `<37>` 会出现如下内容

``` log
2018-06-01 11:35:25,623 (New I/O worker #2) [WARN - org.apache.flume.source.SyslogUtils.buildEvent(SyslogUtils.java:317)] Event created from Invalid Syslog data.
2018-06-01 11:35:25,627 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{Severity=0, Facility=0, flume.syslog.status=Invalid} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21             Hello world! }
```

### Demo 6 syslogtcp

source 为 `syslogudp`, sink 为控制台输出, channel 为 `memory`。

#### Demo 6 配置文件

``` conf
# demo6-syslogudp.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = syslogudp
a1.sources.r1.host = localhost
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

#### Demo 6 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/demo6-syslogudp.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 6 测试

产生测试数据，

``` sh
$ echo '<37>Hello world!' | nc -4u localhost 44444
```

__注意__：需要在前面添加 `<37>` 来进行 write format 数据，否则会报警告 “Event created from Invalid Syslog data.”

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 14:06:41,521 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{Severity=5, Facility=4} body: 3C 33 37 3E 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 <37>Hello world! }
```

不在前面添加 `<37>` 会出现如下内容

``` log
2018-06-01 14:07:05,075 (Old I/O datagram worker ([id: 0xf2f771fc, /127.0.0.1:44444])) [WARN - org.apache.flume.source.SyslogUtils.buildEvent(SyslogUtils.java:317)] Event created from Invalid Syslog data.
2018-06-01 14:07:05,078 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{Severity=0, Facility=0, flume.syslog.status=Invalid} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21             Hello world! }
```

### Demo 7 HTTPSource

source 为 `HTTPSource`, sink 为控制台输出, channel 为 `memory`。

#### Demo 7 配置文件

``` conf
# demo7-httpsource.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = org.apache.flume.source.http.HTTPSource
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

#### Demo 7 启动命令

``` sh
/usr/local/flume/bin/flume-ng agent --conf /usr/local/flume/conf --conf-file /usr/local/flume/conf/demo7-httpsource.conf --name a1 -Dflume.root.logger=INFO,console
```

#### Demo 7 测试

生成 Json 格式的 POST Request

``` sh
$ curl -X POST -d '[{ "headers" :{"namenode" : "namenode.example.com","datanode" : "random_datanode.example.com"},"body" : "really_random_body"}]' http://localhost:44444
```

在观察启动 flume 时的终端，会出现如下内容

``` log
2018-06-01 14:17:06,640 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{namenode=namenode.example.com, datanode=random_datanode.example.com} body: 72 65 61 6C 6C 79 5F 72 61 6E 64 6F 6D 5F 62 6F really_random_bo }
```


