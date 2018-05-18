---
layout:      post
title:       "exec 进入存在多个容器 pod 中的某个容器的办法"
categories:  [运维, 虚拟化, 集群, kubernetes]
description: "以 kube dns 为例，dns pod 中存在 3 个容器。"
keywords:    运维, 虚拟化, 集群, kubernetes
---

### 查看并确定 dns 对应的 pod 名和启动的容器数量

``` sh
# kubectl get pods --namespace=kube-system
NAME                READY     STATUS    RESTARTS   AGE
busybox             1/1       Running   1          2d
grafana-l5eby       1/1       Running   1          1d
heapster-5rb6d      1/1       Running   0          1h
influxdb-ex1ik      1/1       Running   0          1d
kube-dns-v8-m2eri   3/3       Running   3          2d
```

### 查看 pod 中具体的 containers 名

``` sh
kubectl get pod kube-dns-v8-m2eri -o yaml --namespace=kube-system
```

### 根据具体的 containers 名进入到容器内部

``` sh
kubectl exec -it kube-dns-v8-m2eri -c etcd /bin/sh --namespace=kube-system 
```


