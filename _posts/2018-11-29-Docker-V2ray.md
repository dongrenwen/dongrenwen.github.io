---
layout:      post
title:       "Docker 部署 V2Ray"
categories:  [运维, 虚拟化, Docker, V2Ray]
description: "通过 Docker 部署 V2Ray 和 shadowsocks 服务"
keywords:    运维, 虚拟化, Docker, V2Ray
---

# Docker 部署 V2Ray

### 安装 Docker

```
yum install -y docker
```

### 取得 V2Ray 镜像

```
docker pull v2ray/official
```

### 准备配置文件

创建文件夹 `/etc/v2ray/`, 并在里面创建文件 `config.json`, 文件的内容如下：

``` json
{
  "log" : {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 8888,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "e344b244-482a-423a-a72c-c38fd0b12f25",
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
        "network": "tcp"
    }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  },
  "inboundDetour": [
    {
      "protocol": "shadowsocks",
      "port": 6666,
      "settings": {
        "method": "chacha20",
        "password": "v2ray",
        "udp": false
      }
    }
  ],
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "ip": ["geoip:private"],
          "outboundTag": "blocked"
        }
      ]
    }
  }
}
```

这个配置文件中，将 v2ray 的端口设置成了 `8888`， shadowsocks 的端口设置成为了 `6666`, 另外还有密码，ID 等内容，可根据自己的需要进行修改。

### 启动 Docker

```
docker run -it -d --name v2ray -v /etc/v2ray:/etc/v2ray -p 8888:8888 v2ray/official v2ray -config=/etc/v2ray/config.json
```

映射的端口号根据自己的实际情况填写。

### 一些常用的命令

+ 查看运行状态

    ```
    [root@xxxxx ~]# docker container ls
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                              NAMES
    5e6c7ba316aa        v2ray/official      "v2ray -config=/et..."   2 hours ago         Up 2 hours          0.0.0.0:6666->6666/tcp, 0.0.0.0:8888->8888/tcp   v2ray
    [root@xxxxx ~]# 
    ```

+ 启动 V2Ray

    ```
    docker container start v2ray
    ```

+ 停止 V2Ray

    ```
    docker container stop v2ray
    ```

+ 重启 V2Ray

    ```
    docker container restart v2ray
    ```

+ 查看日志

    ```
    docker container logs v2ray
    ```

+ 更新 V2Ray 镜像

    ```
    docker container stop v2ray
    docker container rm v2ray
    docker pull v2ray/official
    docker run -it -d --name v2ray -v /etc/v2ray:/etc/v2ray -p 8888:8888 v2ray/official v2ray -config=/etc/v2ray/config.json
    ```