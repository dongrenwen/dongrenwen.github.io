---
layout:      post
title:       "查看 Registry 本地仓库镜像的办法"
categories:  [运维, 虚拟化, Docker]
description: "查看 V2 版本 Registry 本地仓库中 Docker 镜像的方法。"
keywords:    运维, 虚拟化, Docker
---

# 查看 Registry 本地仓库镜像的办法

查看 V2 版本 Registry 本地仓库中 Docker 镜像的方法。

+ 查看镜像名称列表

    ``` sh
    curl -XGET http://{registry:5000}/v2/_catalog
    ```
    
+ 查看具体镜像对应的 tag 列表

    ``` sh
    curl -XGET http://{registry:5000}/v2/{images_name}/tags/list
    ```
    
**注意:** `{}` 括号中的内容需要根据实际情况修改



