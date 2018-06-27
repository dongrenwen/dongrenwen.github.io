---
layout:      post
title:       "Kafka 监控指定目录简易步骤"
categories:  [运维, 大数据, Kafka]
description: "Kafka 监控指定目录简易步骤"
keywords:    运维, 大数据, Kafka
---


1. 开启新终端启动 zookeeper

    ``` sh
    /opt/kafka_2.11-1.1.0/bin/zookeeper-server-start.sh /opt/kafka_2.11-1.1.0/config/zookeeper.properties
    ```

2. 开启新终端启动 kafka

    ``` sh
    /opt/kafka_2.11-1.1.0/bin/kafka-server-start.sh /opt/kafka_2.11-1.1.0/config/server.properties
    ```

3. 开启新终端启动 kafka 消费, topic 为 test

    ``` sh
    /opt/kafka_2.11-1.1.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
    ```

4. 开启新终端启动 Flume, 如果是 kafka 集群的话，配置文件中的 kafka.bootstrap.servers 写成多个就可以

    ``` sh
    /opt/apache-flume-1.8.0-bin/bin/flume-ng agent -n LogAgent -c /opt/apache-flume-1.8.0-bin/conf/ -f /opt/apache-flume-1.8.0-bin/flume_kafka.conf -Dflume.root.logger=INFO,console
    ```

__监控的文件夹为:__`/tmp/logs/`


