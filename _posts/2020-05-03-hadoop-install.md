---
layout:      post
title:       "Hadoop 集群在虚拟机上进行简易搭建"
categories:  [运维, Linux, 大数据, Hadoop]
description: "Hadoop 集群在虚拟机上进行简易搭建"
keywords:    运维, Linux, 大数据, Hadoop
---

# Hadoop 集群在虚拟机上进行简易搭建

## 主机相关

### 服务器说明

1. `hadoop01`

    + 物理硬件： PVE 虚拟硬件
    + 操作系统： rhel-server-7.2-x86_64
    + 主机名： hadoop01 hadoop01.pve
    + IP 地址： 192.168.0.103
    + CPU： 2C
    + 内存： 4G
    + 硬盘： 64G + 100G(数据盘)

1. `hadoop02`

    + 物理硬件： PVE 虚拟硬件
    + 操作系统： rhel-server-7.2-x86_64
    + 主机名： hadoop02 hadoop02.pve
    + IP 地址： 192.168.0.104
    + CPU： 2C
    + 内存： 4G
    + 硬盘： 64G + 100G(数据盘)

1. `hadoop03`

    + 物理硬件： PVE 虚拟硬件
    + 操作系统： rhel-server-7.2-x86_64
    + 主机名： hadoop03 hadoop03.pve
    + IP 地址： 192.168.0.105
    + CPU： 2C
    + 内存： 4G
    + 硬盘： 64G + 100G(数据盘)

### 操作系统安装

__**此步骤省略。**__ 

__注：__ 安装操作系统的同时，设置主机名和时间同步

### 关闭防火墙和 selinux

```sh
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 临时关闭 selinux
setenforce 0
# 永久关闭 selinux
# 将 /etc/selinux/config 文件中的 SELINUX=enforcing 修改为 SELINUX=disabled
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

### 配置 Yum 源

```sh
# 在 /etc/fstab 中追加光驱的挂载信息
cat >> /etc/fstab << EOF
/dev/sr0 /mnt                                          auto    defaults        0 0
EOF

# 使挂载信息生效
mount -a

# 增加 yum 源配置文件
cat > /etc/yum.repos.d/rhel7source.repo << EOF
[rhel7-source]
name=rhel7-source
baseurl=file:///mnt
enable=1
gpgcheck=0
gpgkey=file:///mnt/RPM-GPG-KEY-redhat-release
EOF

# 刷新 yum 源
yum makecache

# 安装 screen
yum install screen -y
```

### 编辑 `/etc/hosts`

```
cat >> /etc/hosts << EOF
192.168.0.103   hadoop01 hadoop01.pve
192.168.0.104   hadoop02 hadoop02.pve
192.168.0.105   hadoop03 hadoop03.pve
EOF
```

### 准备数据目录

```sh
# 创建数据目录
mkdir /data

# 创建 PV
pvcreate /dev/sdb

# 创建 VG
vgcreate hadoop_data /dev/sdb

# 创建 LV
lvcreate -n data -L 99G hadoop_data

# 格式化逻辑卷
mkfs.xfs /dev/hadoop_data/data

# 添加新的挂载信息
cat >> /etc/fstab << EOF
/dev/mapper/hadoop_data-data /data                     xfs     defaults        0 0
EOF

# 使挂载信息生效
mount -a
```

### 创建 hadoop 用户

```sh
# 创建 Hadoop 用户
useradd -m -s /usr/bin/bash hadoop
passwd hadoop

# 将数据目录划分给 hadoop
chown hadoop:hadoop /data
```

## Hadoop 相关 (以下操作通过 hadoop 用户操作)

### 组件版本选择(因为组件存在依存关系，安装顺序不建议修改)

1. jdk-8u251-linux-x64.tar.gz
1. hadoop-2.10.0.tar.gz
1. zookeeper-3.4.14.tar.gz
1. hbase-2.2.4-bin.tar.gz
1. db-derby-10.14.2.0-bin.tar.gz
1. apache-hive-2.3.7-bin.tar.gz
1. scala-2.12.11.tgz
1. spark-2.4.5-bin-hadoop2.7.tgz
1. flink-1.10.0-bin-scala_2.12.tgz

### Hadoop(hdfs、yarn) 安装

#### 配置免密登陆

```sh
# 创建 openssl 密钥
ssh-keygen

# 分别拷贝密钥到三台主机
ssh-copy-id hadoop01
ssh-copy-id hadoop02
ssh-copy-id hadoop03
ssh-copy-id hadoop01.pve
ssh-copy-id hadoop02.pve
ssh-copy-id hadoop03.pve
```

#### 配置 JDK

```sh
# 创建安装包存放目录，并将安装包拷贝到目录中
mkdir /data/package

# 解压 JDK
mkdir /data/software
tar -zxvf /data/package/jdk-8u251-linux-x64.tar.gz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/jdk1.8.0_251 /data/software/jdk

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add JDK environment variable
export JAVA_HOME=/data/software/jdk
export JRE_HOME=\$JAVA_HOME/jre
export CLASSPATH=\$JAVA_HOME/lib:\$JRE_HOME/lib
export PATH=\$JAVA_HOME/bin:\$PATH
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 配置 Hadoop 环境

```sh
# 解压 hadoop
tar -zxvf /data/package/hadoop-2.10.0.tar.gz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/hadoop-2.10.0 /data/software/hadoop

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add Hadoop environment variable
export HADOOP_HOME=/data/software/hadoop
export PATH=\$PATH:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 创建 Hadoop 相关目录

```sh
mkdir /data/hadoop/
mkdir /data/hadoop/tmp
mkdir /data/hadoop/dfs
mkdir /data/hadoop/dfs/name
mkdir /data/hadoop/dfs/data
```

#### 修改 Hadoop 配置文件

1. 在 $HADOOP_HOME/etc/hadoop/core-site.xml 的 `configuration` 中添加

    ```xml
    <configuration>
        <property>
        <!-- 指定HDFS中NameNode的地址 -->
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop01:9000</value>
        </property>
        <!-- 指定hadoop临时目录 -->
        <property>
            <name>hadoop.tmp.dir </name>
            <value>/data/hadoop/tmp</value>
        </property>
        <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
        </property>
    </configuration>
    ```

1. 在 $HADOOP_HOME/etc/hadoop/hdfs-site.xml 的 `configuration` 中添加

    ```xml
    <configuration>
        <!-- NameNode永久存储名称空间和事务日志的本地文件系统上的路径 -->
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/data/hadoop/dfs/name</value>
        </property>
        <!-- DataNode本地文件系统上应存储其块的路径列表-->
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/data/hadoop/dfs/data</value>
        </property>
        <!-- 指定HDFS副本的数量，默认为3 ,这里我们由于只建了两个子节点，所以选择2-->
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <!--指定可以通过web访问hdfs目录-->
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
        <!--如果是true，则打开权限检查系统;如果是false，权限检查就是关闭的，但是其他行为没有改变。-->
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
        <!--此参数用于设置Web服务器使用的用户名，如果将这个参数设置为超级用户的名称，则所有Web客户就可以看到所有的信息-->
        <property>
                <name>dfs.web.ugi</name>
                <value>supergroup</value>
        </property>
    </configuration>
    ```

1. 将 $HADOOP_HOME/etc/hadoop/ 中的 mapred-site.xml.template 拷贝为 mapred-site.xml 并在 `configuration` 中添加

    ```sh
    cp $HADOOP_HOME/etc/hadoop/mapred-site.xml.template $HADOOP_HOME/etc/hadoop/mapred-site.xml
    ```

    ```xml
    <configuration>
        <!-- 执行框架设置为Hadoop YARN -->
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        
        <!-- MapReduce JobHistory服务器的配置 -->
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>hadoop01:10020</value>
        </property>
        <!-- MapReduce JobHistory服务器的web端口配置，默认端口是19888。 -->
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>hadoop01:19888</value>
        </property>
    </configuration>
    ```

1. 在 $HADOOP_HOME/etc/hadoop/yarn-site.xml 的 `configuration` 中添加

    ```xml
    <configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>hadoop01:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>hadoop01:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>hadoop01:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>hadoop01:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>hadoop01:8088</value>
        </property>
    </configuration>
    ```

1. 修改 $HADOOP_HOME/etc/hadoop/slaves

    ```conf
    hadoop02
    hadoop03
    ```

1. 将刚刚修改过的配置文件全部同步到其他节点

    ```sh
    # core-site.xml
    scp $HADOOP_HOME/etc/hadoop/core-site.xml hadoop02:$HADOOP_HOME/etc/hadoop/core-site.xml
    scp $HADOOP_HOME/etc/hadoop/core-site.xml hadoop03:$HADOOP_HOME/etc/hadoop/core-site.xml

    # hdfs-site.xml
    scp $HADOOP_HOME/etc/hadoop/hdfs-site.xml hadoop02:$HADOOP_HOME/etc/hadoop/hdfs-site.xml
    scp $HADOOP_HOME/etc/hadoop/hdfs-site.xml hadoop03:$HADOOP_HOME/etc/hadoop/hdfs-site.xml

    # mapred-site.xml
    scp $HADOOP_HOME/etc/hadoop/mapred-site.xml hadoop02:$HADOOP_HOME/etc/hadoop/mapred-site.xml
    scp $HADOOP_HOME/etc/hadoop/mapred-site.xml hadoop03:$HADOOP_HOME/etc/hadoop/mapred-site.xml

    # yarn-site.xml
    scp $HADOOP_HOME/etc/hadoop/yarn-site.xml hadoop02:$HADOOP_HOME/etc/hadoop/yarn-site.xml
    scp $HADOOP_HOME/etc/hadoop/yarn-site.xml hadoop03:$HADOOP_HOME/etc/hadoop/yarn-site.xml

    # slaves
    scp $HADOOP_HOME/etc/hadoop/slaves hadoop02:$HADOOP_HOME/etc/hadoop/slaves
    scp $HADOOP_HOME/etc/hadoop/slaves hadoop03:$HADOOP_HOME/etc/hadoop/slaves
    ```

1. 格式化 NameNode，在 Master 节点执行

    ```sh
    hadoop namenode -format
    ```

#### 启动 Hadoop

```sh
start-all.sh
```

#### 查看主节点和子节点的 jps 进程

    + `NameNode` 主节点
    + `ResourceManager` 主节点
    + `SecondaryNameNode` 主节点
    + `DataNode` 子节点
    + `NodeManager` 子节点

#### 访问链接

+ [http://192.168.0.103:8088/cluster](http://192.168.0.103:8088/cluster)
+ [http://192.168.0.103:50070](http://192.168.0.103:50070)

#### 停止 Hadoop

```sh
stop-all.sh
```

### zookeeper 安装

#### 配置 zookeeper 环境

```sh
# 解压 zookeeper
tar -zxvf /data/package/zookeeper-3.4.14.tar.gz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/zookeeper-3.4.14 /data/software/zookeeper

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add zookeeper environment variable
export ZOOKEEPER_HOME=/data/software/zookeeper
export PATH=\$PATH:\$ZOOKEEPER_HOME/bin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 创建需要用到的目录

```sh
mkdir /data/zookeeper
mkdir /data/zookeeper/logs
mkdir /data/zookeeper/data
```

#### 修改 zookeeper 配置文件

1. `$ZOOKEEPER_HOME/conf/zoo.cfg`

    ```sh
    cp $ZOOKEEPER_HOME/conf/zoo_sample.cfg $ZOOKEEPER_HOME/conf/zoo.cfg
    ```

    ```cfg
    dataDir=/data/zookeeper/data
    server.1=hadoop01:2888:3888
    server.2=hadoop02:2888:3888
    server.3=hadoop03:2888:3888
    ```

1. `$ZOOKEEPER_HOME/conf/log4j.properties`

    ```properties
    zookeeper.log.dir=/data/zookeeper/logs
    zookeeper.tracelog.dir=/data/zookeeper/logs
    ```

1. `$ZOOKEEPER_HOME/bin/zkEnv.sh`
    
    ```sh
    if [ "x${ZOO_LOG_DIR}" = "x" ]
    then
        ZOO_LOG_DIR="/data/zookeeper/logs"
    fi
    ```

1. 拷贝配置文件到其他节点

    ```sh
    # zoo.cfg
    scp $ZOOKEEPER_HOME/conf/zoo.cfg hadoop02:$ZOOKEEPER_HOME/conf/zoo.cfg
    scp $ZOOKEEPER_HOME/conf/zoo.cfg hadoop03:$ZOOKEEPER_HOME/conf/zoo.cfg

    # log4j.properties
    scp $ZOOKEEPER_HOME/conf/log4j.properties hadoop02:$ZOOKEEPER_HOME/conf/log4j.properties
    scp $ZOOKEEPER_HOME/conf/log4j.properties hadoop03:$ZOOKEEPER_HOME/conf/log4j.properties

    # zkEnv.sh
    scp $ZOOKEEPER_HOME/bin/zkEnv.sh hadoop02:$ZOOKEEPER_HOME/bin/zkEnv.sh
    scp $ZOOKEEPER_HOME/bin/zkEnv.sh hadoop03:$ZOOKEEPER_HOME/bin/zkEnv.sh
    ```

#### 创建 `/data/zookeeper/data/myid`

分别在三台主机上存放不同的 ID

```sh
# 第一台
ssh hadoop01 "echo '1' > /data/zookeeper/data/myid"

# 第二台
ssh hadoop02 "echo '2' > /data/zookeeper/data/myid"

# 第三台
ssh hadoop03 "echo '3' > /data/zookeeper/data/myid"
```

#### 分别登陆所有主机启动

```sh
ssh hadoop01 "zkServer.sh start"
ssh hadoop02 "zkServer.sh start"
ssh hadoop03 "zkServer.sh start"
```

#### 查看集群状态

```sh
zkServer.sh status

# 如果连接本机，不用使用 -server 参数
zkCli.sh -server hadoop01:2181
```

#### 分别登陆所有主机停止服务

```sh
ssh hadoop01 "zkServer.sh stop"
ssh hadoop02 "zkServer.sh stop"
ssh hadoop03 "zkServer.sh stop"
```

### HBase 安装

#### 配置 HBase 环境

```sh
# 解压 HBase
tar -zxvf /data/package/hbase-2.2.4-bin.tar.gz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/hbase-2.2.4 /data/software/hbase

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add HBase environment variable
export HBASE_HOME=/data/software/hbase
export PATH=\$PATH:\$HBASE_HOME/bin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 修改 HBase 配置文件

1. `$HBASE_HOME/conf/hbase-site.xml`

    ```xml
    <configuration>
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://hadoop01:9000/hbase</value>
        </property>
        <property>
            <name>hbase.zookeeper.property.clientPort</name>
            <value>2181</value>
            <description>Property from ZooKeeper's config zoo.cfg.
            The port at which the clients will connect.
            </description>
        </property>
        <property>
        <name>zookeeper.session.timeout</name>
        <value>120000</value>
        </property>
        <property>
        <name>hbase.zookeeper.property.tickTime</name>
        <value>2000</value>
        </property>
        <property>
            <name>hbase.zookeeper.quorum</name>
            <value>hadoop01,hadoop02,hadoop03</value>
        </property>
        <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
        </property>
    </configuration>
    ```

1. `$HBASE_HOME/conf/hbase-env.sh`

    ```sh
    export HBASE_MANAGES_ZK=false
    ```

1. `$HBASE_HOME/conf/regionservers`

    ```conf
    hadoop01
    hadoop02
    hadoop03
    ```

1. 将刚刚修改过的配置文件全部同步到其他节点

    ```sh
    # hbase-site.xml
    scp $HBASE_HOME/conf/hbase-site.xml hadoop02:$HBASE_HOME/conf/hbase-site.xml
    scp $HBASE_HOME/conf/hbase-site.xml hadoop03:$HBASE_HOME/conf/hbase-site.xml

    # regionservers
    scp $HBASE_HOME/conf/regionservers hadoop02:$HBASE_HOME/conf/regionservers
    scp $HBASE_HOME/conf/regionservers hadoop03:$HBASE_HOME/conf/regionservers

    # start-hbase.sh
    scp $HBASE_HOME/conf/hbase-env.sh hadoop02:$HBASE_HOME/conf/hbase-env.sh
    scp $HBASE_HOME/conf/hbase-env.sh hadoop03:$HBASE_HOME/conf/hbase-env.sh
    ```

#### 启动 HBase

```sh
start-hbase.sh
```

#### 检查所有节点的 JPS 进程

+ HQuorumPeer 主节点、从节点
+ HRegionServer 主节点、从节点
+ HMaster 主节点

#### 访问连接

[http://192.168.0.103:16010/master-status](http://192.168.0.103:16010/master-status)

#### hbase shell

```sh
hbase shell
status
version
quit
```

#### 停止 HBase

```sh
stop-hbase.sh
```

### db-derby 安装

#### 配置 db-derby 环境

```sh
# 解压
tar -zxvf /data/package/db-derby-10.14.2.0-bin.tar.gz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/db-derby-10.14.2.0-bin /data/software/db-derby

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add DB-DERBY environment variable
export DERBY_INSTALL=/data/software/db-derby
export DERBY_HOME=/data/software/db-derby
export PATH=\$PATH:\$DERBY_HOME/bin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 创建需要用到到目录

```sh
mkdir /data/db-derby
```

#### 启动 db-derby

```sh
# 通过 screen -S db-derby 执行，执行后 Ctrl+a,d 退出
cd /data/db-derby
startNetworkServer -h 0.0.0.0
```

### Hive 安装

#### 配置 Hive 环境

```sh
# 解压
tar -zxvf /data/package/apache-hive-2.3.7-bin.tar.gz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/apache-hive-2.3.7-bin /data/software/hive

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add Hive environment variable
export HIVE_HOME=/data/software/hive
export PATH=\$PATH:\$HIVE_HOME/bin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 创建需要用到到目录

```sh
mkdir /data/hive
mkdir /data/hive/logs
mkdir /data/hive/tmp
```

#### 在 HDFS 中创建 Hive 相关目录

```sh
hadoop fs -mkdir /tmp
hadoop fs -mkdir /user
hadoop fs -mkdir /user/hive
hadoop fs -mkdir /user/hive/warehourse
hadoop fs -chmod g+w /tmp
hadoop fs -chmod g+w /user/hive/warehourse
```

#### 修改 Hive 配置文件

1. `hive-env.sh`

    ```sh
    cp $HIVE_HOME/conf/hive-env.sh.template $HIVE_HOME/conf/hive-env.sh
    ```

    ```sh
    # Set HADOOP_HOME to point to a specific hadoop install directory
    # HADOOP_HOME=${bin}/../../hadoop
    HADOOP_HOME=/data/software/hadoop
    ```

1. `hive-site.xml`

    ```sh
    cp $HIVE_HOME/conf/hive-default.xml.template $HIVE_HOME/conf/hive-site.xml
    ```

    ```xml
    <property>
        <name>system:java.io.tmpdir</name>
        <value>/data/hive/tmp</value>
    </property>
    <property>
        <name>system:user.name</name>
        <value>${user.name}</value>
    </property>
    ```

    ```xml
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:derby://localhost:1527/metastore_db;create=true</value>
    ```

1. `$HIVE_HOME/conf/jpox.properties`

    ```properties
    javax.jdo.PersistenceManagerFactoryClass=org.jpox.PersistenceManagerFactoryImpl
    org.jpox.autoCreateSchema=false
    org.jpox.validateTables=false
    org.jpox.validateColumns=false
    org.jpox.validateConstraints=false
    org.jpox.storeManagerType=rdbms
    org.jpox.autoCreateSchema=true
    org.jpox.autoStartMechanismMode=checked
    org.jpox.transactionIsolation=read_committed
    javax.jdo.option.DetachAllOnCommit=true
    javax.jdo.option.NontransactionalRead=true
    javax.jdo.option.ConnectionDriverName=org.apache.derby.jdbc.ClientDriver
    javax.jdo.option.ConnectionURL=jdbc:derby://hadoop1:1527/metastore_db;create=true
    javax.jdo.option.ConnectionUserName=APP
    javax.jdo.option.ConnectionPassword=mine
    ```

1. `hive-exec-log4j2.properties`

    ```sh
    cp $HIVE_HOME/conf/hive-exec-log4j2.properties.template $HIVE_HOME/conf/hive-exec-log4j2.properties
    ```

    ```properties
    property.hive.log.dir = /data/hive/logs
    ```

1. `hive-log4j2.properties`

    ```sh
    cp $HIVE_HOME/conf/hive-log4j2.properties.template $HIVE_HOME/conf/hive-log4j2.properties
    ```

    ```properties
    property.hive.log.dir = /data/hive/logs
    ```

#### 拷贝 db-derby 的 jar 文件到 hive 依赖中

```sh
cp $DERBY_HOME/lib/derbyclient.jar $HIVE_HOME/lib
cp $DERBY_HOME/lib/derbytools.jar $HIVE_HOME/lib
```

#### 初始化 Hive

```sh
schematool -dbType derby -initSchema
```

#### 测试 Hive

```sql
-- 通过 hive 命令启动后执行
create database test;
show databases;
drop database test;
```

#### 启动 Hive 服务

```sh
# 通过 screen -S Hive 执行，执行后 Ctrl+a,d 退出
hiveserver2 &> /data/hive/logs/hiveserver2_$(date +%s).log
```

### scala 安装

#### 配置 scala 环境

```sh
# 解压
tar -zxvf /data/package/scala-2.12.11.tgz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/scala-2.12.11 /data/software/scala

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add scala environment variable
export SCALA_HOME=/data/software/scala
export PATH=\$PATH:\$SCALA_HOME/bin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 验证 scala

```sh
# 验证 scala
scala -version
```

### Spark 安装

#### 配置 Spark 环境

```sh
# 解压
tar -zxvf /data/package/spark-2.4.5-bin-hadoop2.7.tgz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/spark-2.4.5-bin-hadoop2.7 /data/software/spark

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add spark environment variable
export SPARK_HOME=/data/software/spark
export PYTHONPATH=\$SPARK_HOME/python
export PATH=\$PATH:\$SPARK_HOME/bin:\$SPARK_HOME/sbin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 创建目录

```sh
mkdir /data/spark
mkdir /data/spark/logs
```

#### 修改 spark 配置文件

1. `spark-env.sh`

    ```sh
    cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
    ```

    ```sh
    export SCALA_HOME=/data/software/scala
    export JAVA_HOME=/data/software/jdk
    export HADOOP_INSTALL=/data/software/hadoop
    export HADOOP_CONF_DIR=$HADOOP_INSTALL/etc/hadoop
    SPARK_MASTER_IP=hadoop01
    SPARK_LOCAL_DIRS=/data/software/spark
    SPARK_WORKER_CORES=1 #Worker的cpu核数
    SPARK_WORKER_MEMORY=2048m # 根据实际内存大小调整(1.0g不识别，改为m单位)
    SPARK_WORKER_OPTS="-Dspark.worker.cleanup.enabled=true -Dspark.worker.cleanup.appDataTtl=604800" #worker自动清理及清理时间间隔
    SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs://hadoop01:9000/spark/history" #history server页面端口>、备份数、log日志在HDFS的位置
    SPARK_LOG_DIR=/data/spark/logs #配置Spark的log日志
    ```

2. `slaves`

    ```sh
    cp $SPARK_HOME/conf/slaves.template $SPARK_HOME/conf/slaves
    ```

    ```conf
    hadoop02
    hadoop03
    ```

3. `spark-defaults.conf`

    ```sh
    cp $SPARK_HOME/conf/spark-defaults.conf.template $SPARK_HOME/conf/spark-defaults.conf
    ```

    ```conf
    spark.master                     spark://hadoop01:7077
    spark.eventLog.enabled           true
    spark.eventLog.dir               hdfs://hadoop01:9000/spark/history
    spark.serializer                 org.apache.spark.serializer.KryoSerializer
    spark.driver.memory              512m
    spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
    ```

1. 将刚刚修改过的配置文件全部同步到其他节点

    ```sh
    # spark-env.sh
    scp $SPARK_HOME/conf/spark-env.sh hadoop02:$SPARK_HOME/conf/spark-env.sh
    scp $SPARK_HOME/conf/spark-env.sh hadoop03:$SPARK_HOME/conf/spark-env.sh

    # slaves
    scp $SPARK_HOME/conf/slaves hadoop02:$SPARK_HOME/conf/slaves
    scp $SPARK_HOME/conf/slaves hadoop03:$SPARK_HOME/conf/slaves

    # spark-defaults.conf
    scp $SPARK_HOME/conf/spark-defaults.conf hadoop02:$SPARK_HOME/conf/spark-defaults.conf
    scp $SPARK_HOME/conf/spark-defaults.conf hadoop03:$SPARK_HOME/conf/spark-defaults.conf
    ```


#### 创建 HDFS 上的目录

```sh
hadoop fs -mkdir -p /spark/history
```

#### 启动 Spark 服务

```sh
start-master.sh
start-slaves.sh
```

#### 验证 Spark 服务

+ [http://192.168.0.103:8080](http://192.168.0.103:8080)
+ `run-example SparkPi`

#### 停止 Spark 服务

```sh
stop-master.sh
stop-slaves.sh
```

### Flink 安装

#### 配置 Flink 环境

```sh
# 解压 Flink
tar -zxvf /data/package/flink-1.10.0-bin-scala_2.12.tgz -C /data/software/

# 为后期版本升级顺畅制作软连接
ln -s /data/software/flink-1.10.0 /data/software/flink

# 配置环境变量
cat >> ~/.bashrc << EOF
# Add Flink environment variable
export FLINK_HOME=/data/software/flink
export PATH=\$PATH:\$FLINK_HOME/bin
EOF

# 使环境变量生效
source ~/.bashrc
```

#### 修改 spark 配置文件

1. `$FLINK_HOME/conf/flink-conf.yaml`

    ```yaml
    jobmanager.rpc.address: hadoop01
    ```

1. `$FLINK_HOME/conf/slaves`

    ```text
    hadoop02
    hadoop03
    ```

1. `$FLINK_HOME/conf/masters`

    ```text
    hadoop01:8081
    ```

1. 将刚刚修改过的配置文件全部同步到其他节点

    ```sh
    # flink-conf.yaml
    scp $FLINK_HOME/conf/flink-conf.yaml hadoop02:$FLINK_HOME/conf/flink-conf.yaml
    scp $FLINK_HOME/conf/flink-conf.yaml hadoop03:$FLINK_HOME/conf/flink-conf.yaml

    # masters
    scp $FLINK_HOME/conf/masters hadoop02:$FLINK_HOME/conf/masters
    scp $FLINK_HOME/conf/masters hadoop03:$FLINK_HOME/conf/masters

    # slaves
    scp $FLINK_HOME/conf/slaves hadoop02:$FLINK_HOME/conf/slaves
    scp $FLINK_HOME/conf/slaves hadoop03:$FLINK_HOME/conf/slaves
    ```

#### 启动服务

```sh
start-cluster.sh
```

#### 测试启动内容

+ [192.168.0.103:8081](192.168.0.103:8081)

#### 停止服务

```sh
stop-cluster.sh
```