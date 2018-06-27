---
layout:      post
title:       "Flume 1.8.0 User Guide"
categories:  [运维, 大数据, Flume]
description: "Flume 1.8.0 User Guide"
keywords:    运维, 大数据, Flume
---

## Introduction 介绍

### Overview 概述

Apache Flume 是一个分布式，可靠且可用的系统，用于高效地收集，汇总和将来自多个不同源的大量日志数据移动到集中式数据存储。

Apache Flume 的使用不仅限于日志数据聚合。 由于数据源是可定制的，因此 Flume 可用于传输大量事件数据，包括但不限于网络流量数据，社交媒体生成的数据，电子邮件消息以及几乎任何可能的数据源。

Apache Flume 是 Apache Software Foundation 的顶级项目。

目前有两个发布代码行，版本 0.9.x 和 1.x .

[Flume 0.9.x User Guide](http://archive.cloudera.com/cdh/3/flume/UserGuide/) 提供了 0.9.x 版本的文档。

本文档适用于 1.4.x 版本。

建议新用户和现有用户使用 1.x 版本，以充分利用最新架构中可用的性能改进和灵活性的配置。

### System Requirements 系统要求

1. Java 运行时环境 - Java 1.8 及以上版本
2. 内存 - 需要足够的内存以保证 sources, channels 或者 sinks 的使用
3. 磁盘空间 - 需要足够的磁盘空间以保证 channels 或者 sinks 的使用
4. 目录权限 - Agent 需要对目录有读写权限

### Architecture 架构

#### Data flow model 数据流模型

Flume event 被定义为一个数据流单元，它有一个字节负载和一个可选的字符串属性。一个 Flume agent 是一个 JVM 进程，agent 中的组件将事件流（event flow）从一个外部源传到下一个目的地。 

![](http://flume.apache.org/_images/UserGuide_image00.png)

Flume Source 消费从外部源（如 web server）传递过来的 Events。外部源以目标 Flume Source 能识别的格式发送 Events 给 Flume。例如，Avro Flume Source 可以用于接收来自 Avro 客户端或其他流中的 Flume Agent 的 Avro， Sink 发送的 Avro Events。与此类似的流是，使用 Thrift FLume Source 接收来自 Thrift Sink 或 Thrift Rpc 客户端或来自 Flume Thrift Protocol 产生的任意语言写的 Thrift 客户端发送的数据。当 Flume Source 接收到一个Event，它存储 Event 到一个或多个 Channel。这个 Channel 被动存储 Event，并保留此 Event，直到它被一个 Flume Sink 消费。例如 `-file channel`，它是存储在本地文件系统。Sink把 Event 从 Channel 移除，并把它放到一个外部的库，如 HDFS（经过 Flume HDFS Sink)，或把此 Event 转发给流中的下一个 Flume Agent 的 Flume Source。Agent 中的 Source 和 Sink 异步地进行 Event Staged。

#### Complex flows 复杂流

Flume 允许用户来建立多跳的流，即 Event 经过多个 Agent 之后才到达最终目的。它也可以 “扇入” 和 “扇出” 流（即像扇子一样的流，例如，扇入可以是多个 Agent，流向同一个 Agent。扇出也类似，一个 Agent 的 Event 流向多个 Agent），前面路由和容灾路由，用于在失败的场合保证数据 。

#### Reliability 可靠性

Events 是阶段性的停留在每个 Agent 的 Channel 中。然后 Events 被传递到下一个 Agent 或最终库（如 HDFS）。只有在 Events 被存储在下一个 Agent 的 Channel 中或到达了最终目的，Events 才会在当前 Channel 中删除。这就是在单跳信息传递中，Flume 如何提供端到端可靠流的。

Flume 使用事务型方式来确保可靠的 Events 传输。Source 和 Sink 都被压缩在一个由 Channel 提供的事务中。这确保 Events 集合可以可靠地从流中的一个点传递到另一个点。在这种 “多跳流“ 的情况下，来自前一跳的 Sink 和来自下一跳的 Source，它们都是以事务的方式运行，以此确保数据能够安全的存储在下一跳的 Channel 中。

#### Recoverability 可恢复性

Events 暂存在 Channel 中，Channel 可以从出错中恢复。Flume 提供了一个持久的文件 Channel，这个文件 Channel 是本地文件系统的一个备份。也有 Memory Channel，它简单地存储 Events 在一个内存队列中，Memory Channel 速度很快，但是，当 Agent 进程死了的时候，在 Memory Channel 中的任何 Event 都会丢失。

## Setup 设置

### Setting up an agent 设置 Agent

Flume Agent 的配置是存储在一个本地的配置文件中。它是一个遵循 Java 属性文件格式的文本文件。在相同的配置文件中，可以指定一个或多个 Agent 配置。配置文件包括 Agent 中的每个 Source，Sink 和 Channel 的属性，以及它们是如何组织在一起形成数据流的。

### Configuring individual components 配置单个组件

流中的每个组件（Source, Sink, Channel）都有特定的类型和实例的名称，另外还有针对该类型的指定类型和安装属性集合。例如，一个 Avro Source 需要一个主机名和一个端口号来接收数据。一个 Memory Channel 有最大队列大小（Capacity）,一个 HDFS Sink 需要知道文件系统的 URI，创建文件的路径，文件 Rotation 的频率（“hdfs.rollInterval“）等。组件的所有这些属性应该配置在 Flume Agent 主机上的配置文件中。

### Wiring the pieces together 连接组件

Agent 需要知道哪些组件要加载及它们如何连接以构建成流。通过列出在 Agent 中的每个 Sources, Sinks 和 Channels 的名字，然后对每个 Sink 和 Source 指定 Channel 连接，从而构成流。例如，一个 Agent Events 流,通过一个名为 “file-channel1“ 的文件 Channel，从一个名为 “avroWeb“ 的 Avro Source 到名为 “hdfs-cluster1“ 的 HDFS Sink。配置文件将会包含这些组件的名字，且 “file-channel1” 是作为 AvroWeb Source 和 “hdfs-cluster1” Sink 的共享 Channel。

### Starting an agent 启动一个 Agent

使用名为 `flume-ng` 的 shell 脚本启动代理，该脚本位于 Flume 安装目录的 `bin` 中。 需要在命令行上指定 Agent 名，配置文件目录和配置文件：

``` sh
$ bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties.template
```

现在 Agent 就会启动在配置文件中配置的 Source 和 Sinks。

### A simple example 一个简单实例

在这里，我们给出了一个示例的配置文件，用来描述单节点 Flume 的部署。这个配置文件让用户产生 Events 并将 logs 打印到控制台。

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

这个配置定义了一个 Agent `a1`。`a1` 有一个 Source，这个 Source 监听 `44444` 端口的数据，Channel 缓存 Event 数据在内存中，Sink 打印日志在控制台上。配置文件命名不同的组件，然后描述他们的类型和配置参数。配置文件可以定义一些已命名的 Agent，当给出的 Flume 进程已经在运行，将会发送一个标识告诉它是哪个 Agent。

对于给出的这个配置文件，我们可以按照如下的方式启动 Flume:

``` sh
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
```

注意：在完整部署中，我们通常会包含一个选项 `--conf=<conf-dir>` 。这个 `<conf-dir>` 目录将包含一个 shell 脚本 `flume-env.sh` 和一个可以使用的 `log4f` 属性文件。在这个例子中，我们传递一个 Java 选项来强制 Flume 将 log 输出到控制台，并且我们没有使用自定义的环境脚本。

我们可以在一个新的终端使用 telnet 44444 端口发送一个 Event 给 Flume :

``` sh
$ telnet localhost 44444
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
Hello world! <ENTER>
OK
```

在原来的 Flume 终端将会输出 Event 的 Log 信息。

``` sh
12/06/19 15:32:19 INFO source.NetcatSource: Source starting
12/06/19 15:32:19 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
12/06/19 15:32:34 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          Hello world!. }
```

恭喜，你已经成功的配置并部署了 Flume Agent。后续章节将更详细地介绍 Agent 的配置。

### Using environment variables in configuration files 在配置文件中使用环境变量

Flume 能够在配置中替换环境变量。例如：

``` sh
a1.sources = r1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = ${NC_PORT}
a1.sources.r1.channels = c1
```

注意：它目前只适用于值，不适用于键。（即，只能标记等号右侧的部分）

Agent 可以通过设置 Java 配置文件属性 `propertiesImplementation = org.apache.flume.node.EnvVarResolverProperties` 来启用。

例如：

``` sh
$ NC_PORT=44444 bin/flume-ng agent –conf conf –conf-file example.conf –name a1 -Dflume.root.logger=INFO,console -DpropertiesImplementation=org.apache.flume.node.EnvVarResolverProperties
```

注意：上面只是一个例子，环境变量可以使用其他方式进行配置，包括在 `conf/flume-env.sh` 中设置。

### Logging raw data 记录原始数据

通过注入管线来记录原始数据流到日志文件，在许多生产环境中是不想看到的行为，因为这可能导致泄露敏感数据或安全相关的配置，例如安全密钥。默认情况下，Flume 不会记录这些信息。另一方面，如果数据管线破坏了，Flume 将会为调试问题尝试提供线索。

调试 Event 管线问题的一种方式是，设置一个额外的 Memory Channel 连接到 Logger Sink，这将输出所有的 Event 数据到 Flume 日志。然而，某些情况下，这种方式无法满足。

为了能够记录配置相关和数据相关的 Event，除了 log4j 属性，必须设置 Java 系统配置。

要启用配置相关的日志记录，需要设置 Java 系统属性 `-Dorg.apache.flume.log.printconfig=true`, 这个可以通过命令行传递，也可以通过在 `flume-env.sh` 中设置 `JAVA_OPTS` 属性。

要启用数据记录，请按照上述相同的方式设置Java系统属性 `-Dorg.apache.flume.log.rawdata=true`。对于大多数组件，log4j 日志记录级别还必须设置为 `DEBUG` 或 `TRACE`，以确保指定的 Event 能够附加到 Flume 的日志中。

以下是启用配置日志记录和原始数据日志记录的示例，同时还将 Log4j loglevel 设置为 `DEBUG` 以进行控制台输出：

``` sh
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=DEBUG,console -Dorg.apache.flume.log.printconfig=true -Dorg.apache.flume.log.rawdata=true
```

### Zookeeper based Configuration 基于 Zookeeper 的配置

Flume 支持通过 Zookeeper 进行 Agent 配置。这是一个实验性功能。配置文件需要上传到 Zookeeper，配置文件存储在 Zookeeper 节点数据中。 以下是 Zookeeper 节点树对于代理 a1 和 a2 的样子：

``` config
- /flume
 |- /a1 [Agent config file]
 |- /a2 [Agent config file]
```

上传配置文件后，使用以下选项启动 Agent

``` sh
$ bin/flume-ng agent –conf conf -z zkhost:2181,zkhost1:2181 -p /flume –name a1 -Dflume.root.logger=INFO,console
```

|Argument Name|Default|Description|
|:---|:---|:---|
|z|-|Zookeeper connection string. Comma separated list of hostname:port|
|p|/flume|Base Path in Zookeeper to store Agent configurations|

### Installing third-party plugins 安装第三方插件

Flume 有一个完全基于插件的架构。虽然 Flume 带有许多内置的 sources，channels，sinks，serializers 等，但存在许多与 Flume 分开发布的实现。

尽管通过在 `flume-env.sh` 文件中将它们的 jar 添加到 `FLUME_CLASSPATH` 变量中可以包含自定义的 Flume 组件，但 Flume 现在支持一个名为 `plugins.d` 的特殊目录，该目录会自动获取以特定格式打包的插件。这样可以更轻松地管理插件包，一些类的问题的调试和故障排除更加简单，尤其是包的冲突问题。

#### The plugins.d directory 关于 `plugins.d` 目录

`plugins.d` 目录位于 `$FLUME_HOME/plugins.d`。 启动时，`flume-ng` 启动脚本在`plugins.d` 目录中查找符合以下格式的插件，并在启动 `java` 时将它们包含在适当的路径中。

#### Directory layout for plugins 插件的目录布局

`plugins.d` 中的每个插件（子目录）最多可以有三个子目录：

1. lib - 插件的 jar(s)
2. libext - 插件所依赖的 jar(s)
3. native - 一些必须的本地库，例如 `.so` 文件

`plugins.d` 目录中的两个插件示例：

``` config
plugins.d/
plugins.d/custom-source-1/
plugins.d/custom-source-1/lib/my-source.jar
plugins.d/custom-source-1/libext/spring-core-2.5.6.jar
plugins.d/custom-source-2/
plugins.d/custom-source-2/lib/custom.jar
plugins.d/custom-source-2/native/gettext.so
```

### Data ingestion 数据摄入

Flume 支持多种机制来从外部来源提取数据。

#### RPC

包含在 Flume 发行版中的 Avro 客户端可以使用 Avro RPC 机制将给定文件发送到 Flume Avro Source：

``` sh
$ bin/flume-ng avro-client -H localhost -p 41414 -F /usr/logs/log.10
```

上述命令会将 `/usr/logs/log.10` 的内容发送到 Flume Source 监听的端口。

#### Executing commands 命令的执行

有一个exec source，这个源执行给出的命令并消费输出。也就是单行的输出，文本后面返回 `\r` 或者换行符 `\n` 两者在一起。

#### Network streams 网络流

Flume 支持以下机制从常见日志流类型中读取数据，例如：

1. Avro
2. Thrift
3. Syslog
4. Netcat

#### Setting multi-agent flow 设置多 Agent 流程

![](http://flume.apache.org/_images/UserGuide_image03.png)

为了使数据在多个代理或中继之间流动，前一个 Agent 的 Sink 和当前 Agent 的 Source 必须是 avro 类型，其中 Sink 指向 Source 的主机名（或IP地址）和端口。

#### Consolidation 合并

在日志收集中一个非常常见的场景是，产生日志的客户端有许多个，他们需要发送数据到少量几个连接到存储系统的消费者 Agent。例如，来自上百个 web 服务收集的日志发送到十几个写数据到 HDFS 集群 Agent。 

![](http://flume.apache.org/_images/UserGuide_image02.png)

在 Flume，通过在第一层配置并排的多个 Avro Sink 的 Agent，所有的 Agent 的 Sink 指定一个单个 Agent 的 Avro Source 可以实现上述功能（同样情况下，也可以使用 thrift source/sinks/clients）。第二层 Agent 的 Source 合并收到的 Event 到一个单个的 Channel，这个 Channel 的 Sink 是消费 Channel 的 Event 到最终目的地（在这里是 HDFS）。

#### Multiplexing the flow 流量复用

Flume 支持将 Event 流复用到一个或多个目的地。 这是通过定义一个流复用器来实现的，该复用器可以复制或选择性地将 Event 路由到一个或多个 Channels。

![](http://flume.apache.org/_images/UserGuide_image01.png)

在上述例子中，显示了一个来自 Agent Foo 的 Source，扇出流到三个不同的 Channel。这个扇出可以复制或多路发送。在复制流的情况下，每个 Event 被发送到所有的三个 Channel。对于多路发送的情况，当 Event 的属性匹配预先配置的值，Event 会被传输到一个可用的 Channel 子集。例如，如果一个 Event 属性名为 “txnType“，被设置为 ”customer“，然后，它应该流向 Channel1 和Channel3，如果它被设置为 “vendor“，然后，它应该流向 Channel2，否则，流向 Channel3。这种映射可以在 Agent 的配置文件中进行设置。


