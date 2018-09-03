---
layout:      post
title:       "使用 systemctl 管理 tomcat"
categories:  [运维, tomcat]
description: "使用 systemctl 管理 tomcat"
keywords:    运维, tomcat
---

# 使用 `systemctl` 管理 `tomcat`

## 环境说明

* 操作系统为 CentOS 7.2
* JDK 版本为 1.8
* JDK 安装目录为 `/usr/local/jdk1.8.0_181`
* tomcat 版本为 8
* tomcat 安装目录为 `/usr/local/apache-tomcat-8.5.33`

## 安装 systemd 单元文件使 `systemctl` 命令可用

1. 创建文件 `/etc/systemd/system/tomcat.service`

    ```conf
    [Unit]
    Description=Apache Tomcat Web Application Container
    After=network.target syslog.target

    [Service]
    Type=forking

    Environment=JAVA_HOME=/usr/local/jdk1.8.0_181
    ExecStart=/usr/local/apache-tomcat-8.5.33/bin/startup.sh
    ExecStop=/usr/local/apache-tomcat-8.5.33/bin/shutdown.sh

    [Install]
    WantedBy=multi-user.target
    ```
    
2. 重新加载 systemd 配置文件

    ```sh
    systemctl daemon-reload
    ```
    
3. 启动命令

    ```sh
    systemctl start tomcat
    ```
    
4. 停止命令

    ```sh
    systemctl stop tomcat
    ```
    
5. 查看状态

    ```sh
    systemctl status tomcat
    ```

6. 设置开机启动

    ```sh
    systemctl enable tomcat
    ```
    
7. 取消开机启动

    ```sh
    systemctl disable tomcat
    ```

