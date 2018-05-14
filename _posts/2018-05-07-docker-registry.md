---
layout:      post
title:       "搭建 Docker Registry 本地镜像仓库"
categories:  [运维, 虚拟化, Docker]
description: "通过 Docker Save 命令导出的 tar 包搭建 Registry 本地仓库"
keywords:    运维, 虚拟化, Docker
---

# 搭建 Docker Registry 本地镜像仓库

镜像仓库的安装镜像采用的是 Docker 通过 Save 命令导出的tar包。

1. 从其他服务器考入安装包

    ``` sh
    scp user@192.168.1.50:/root/image/registry_2.tar /root/registry/
    ```

2. 加载安装包

    ``` sh
    docker load -i /root/registry/registry_2.tar
    ```
    
3. 运行服务

    ``` sh
    docker run -d --restart=always -v /root/registry/data:/var/lib/registry --privileged=true -p 5000:5000 192.168.1.50:5000/registry:2
    ```
    
4. 测试本地服务

    ``` sh
    docker tag [镜像id] 192.168.1.51:5000/test/ubuntu:latest
    docker push 192.168.1.51:5000/test/ubuntu:latest
    docker rmi 192.168.1.51:5000/test/ubuntu:latest
    docker pull 192.168.1.51:5000/test/ubuntu:latest
    ```
    

