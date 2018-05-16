---
layout:      post
title:       "搭建 kubernetes 集群"
categories:  [运维, 虚拟化, 集群, kubernetes]
description: "集群包含一台 Master 服务器和三台 Node 服务器。"
keywords:    运维, 虚拟化, 集群, kubernetes
---

## 针对所有服务器的统一配置

+ 在 `/etc/hosts` 文件下面添加如下内容

    ``` conf
    192.168.199.151   rhel-master151
    192.168.199.152   rhel-minion152
    192.168.199.153   rhel-minion153
    192.168.199.154   rhel-minion154
    ```

+ 关闭防火墙和 SELinux

    ``` sh
    systemctl stop firewalld.service
    systemctl disable firewalld.service
    systemctl stop iptables.service
    systemctl disable iptables.service
    setenforce 0
    ```
    
    将 `/etc/selinux/config` 文件中的 `SELINUX=enforcing` 修改为 `SELINUX=disabled`
    
## 配置 Etcd 集群

### 安装 etcd 软件

``` sh
yum install -y etcd
```

### 配置 etcd

因为 etcd 集群没有主从之分，所以所有的服务器都采用相似的方式配置 `/etc/etcd/etcd.conf` 文件。

``` conf
# [member]
ETCD_NAME=etcd1
ETCD_DATA_DIR="/var/lib/etcd/etcd1"
#ETCD_WAL_DIR=""
#ETCD_SNAPSHOT_COUNT="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
ETCD_LISTEN_PEER_URLS="http://192.168.199.151:2380"  # 自身IP
ETCD_LISTEN_CLIENT_URLS="http://192.168.199.151:2379,http://127.0.0.1:2379"   # 自身IP + 回环IP
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#
#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.199.151:2380"   # 自身IP
# if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.199.151:2380,etcd2=http://192.168.199.152:2380,etcd3=http://192.168.199.153:2380,etcd4=http://192.168.199.154:2380"   # 各etcd的主机自身IP
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.199.151:2379"   # 自身IP
... ...
```

### 启动 etcd 服务

``` sh
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
```


## 配置 Master 服务

### 安装 docker kubernetes 软件

``` sh
yum install -y docker kubernetes
```

### 编辑配置文件

+ `/usr/lib/systemd/system/kube-apiserver.service`

    ``` conf
    [Unit]
    Description=Kubernetes API Server
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes
    After=network.target
    After=etcd.service
    Wants=etcd.service
    ... ...
    ```
    
+ `/etc/kubernetes/config`

    ``` conf
    ... ...
    # How the controller-manager, scheduler, and proxy find the apiserver
    KUBE_MASTER="--master=http://192.168.199.151:8080"
    ... ...
    ```

+ `/etc/kubernetes/apiserver`

    ``` conf
    ... ...
    # The address on the local server to listen to.
    KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
    ... ...
    # Comma separated list of nodes in the etcd cluster
    KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.199.151:2379,http://192.168.199.152:2379,http://192.168.199.153:2379,http://192.168.199.154:2379"
    ... ...
    ```

+ `/usr/lib/systemd/system/kube-controller-manager.service`

    ``` conf
    [Unit]
    Description=Kubernetes Controller Manager
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes
    After=etcd.service
    After=kube-apiserver.service
    Requires=etcd.service
    Requires=kube-apiserver.service
    ... ...
    ```
    
### 启动 Master 服务

``` sh
systemctl daemon-reload
systemctl enable kube-apiserver.service
systemctl start kube-apiserver.service
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
systemctl enable kube-scheduler
systemctl start kube-scheduler
```

## 配置 Nodes 服务

### 安装 docker kubernetes 软件

``` sh
yum install -y docker kubernetes
```

### 编辑配置文件

+ `/etc/kubernetes/kubelet`

    ``` conf
    ... ...
    # The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
    KUBELET_ADDRESS="--address=0.0.0.0"
    ... ...
    # You may leave this blank to use the actual hostname
    KUBELET_HOSTNAME="--hostname-override=rhel-minion152"
    ... ...
    # location of the api-server
    KUBELET_API_SERVER="--api-servers=http://192.168.199.151:8080"
    ... ...
    # pod infrastructure container
    KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=192.168.199.150:5000/pause:2.0"
    ... ...
    ```

+ `/etc/kubernetes/config`

    ``` conf
    ... ...
    # How the controller-manager, scheduler, and proxy find the apiserver
    KUBE_MASTER="--master=http://192.168.199.151:8080"
    ```

### 启动 Nodes 服务

``` sh
systemctl daemon-reload
systemctl enable kubelet.service
systemctl start kubelet.service
systemctl enable kube-proxy
systemctl start kube-proxy
```

## 配置 flannel 服务

### 设置 etcd 网络模式及IP地址

``` sh
etcdctl set /cluster.hostgw/network/config '{"Network":"172.17.0.0/16","Backend":{"Type":"host-gw"}}'
```

### 安装 flannel 软件

``` sh
yum install -y flannel
```

### 编辑主节点的配置文件 `/etc/sysconfig/flanneld`

``` conf
# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD="http://192.168.199.151:2379,http://192.168.199.152:2379,http://192.168.199.153:2379,http://192.168.199.154:2379"


# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_KEY="/cluster.hostgw/network"


# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
```

### 编辑从节点的配置文件 `/etc/sysconfig/flanneld`

``` conf
# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD="http://192.168.199.151:2379,http://192.168.199.152:2379,http://192.168.199.153:2379,http://192.168.199.154:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_KEY="/cluster.hostgw/network"

# Any additional options that you want to pass
FLANNEL_OPTIONS='-iface="eno16777736"'
```

**注意：** *每个从节点的 `/etc/sysconfig/flanneld` 中添加网卡名，主节点不用加。*

    ``` conf
    FLANNEL_OPTIONS='-iface="网卡名"'
    ```

### 启动 flannel 服务

``` sh
systemctl daemon-reload
systemctl enable flanneld
systemctl start flanneld
systemctl status flanneld
```

**注：**
先启动 etcd，再启动 flanneld，最后启动 docker，以下配置会自动生成，这样就保证每台 node 节点的 docker 容器 IP 范围在不同网段。

``` log
[root@rhel-minion153 ~]# cat /run/flannel/docker
DOCKER_OPT_BIP="--bip=172.17.76.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=true"
DOCKER_OPT_MTU="--mtu=1500"
DOCKER_NETWORK_OPTIONS=" --bip=172.17.76.1/24 --ip-masq=true --mtu=1500 "
```

## 配置 DNS 服务

### 前期准备

+ 准备 docker 镜像

    ``` sh
    192.168.199.150:5000/busybox:latest
    192.168.199.150:5000/kube2sky:1.12
    192.168.199.150:5000/etcd:2.2.1
    192.168.199.150:5000/skydns:20151013
    192.168.199.150:5000/pause:2.0
    ```

+ `kube-system_namespace.yaml`

    ``` conf
    apiVersion: v1
    kind: Namespace
    metadata:
      name: kube-system
      labels:
        name: kube-system
    ```
    
+ `skydns-rc.yaml`

    ``` conf
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: kube-dns-v8
      namespace: kube-system
      labels:
        k8s-app: kube-dns
        version: v8
        kubernetes.io/cluster-service: "true"
    spec:
      replicas: 1
      selector:
        k8s-app: kube-dns
        version: v8
      template:
        metadata:
          labels:
            k8s-app: kube-dns
            version: v8
            kubernetes.io/cluster-service: "true"
        spec:
          containers:
          - name: etcd
            image: 192.168.199.150:5000/etcd:2.2.1
            resources:
              limits:
                cpu: 100m
                memory: 50Mi
            command:
            - /usr/local/bin/etcd
            - -data-dir
            - /var/etcd/data
            - -listen-client-urls
            - http://127.0.0.1:2379,http://127.0.0.1:4001
            - -advertise-client-urls
            - http://127.0.0.1:2379,http://127.0.0.1:4001
            - -initial-cluster-token
            - skydns-etcd
            volumeMounts:
            - name: etcd-storage
              mountPath: /var/etcd/data
          - name: kube2sky
            image: 192.168.199.150:5000/kube2sky:1.12
            resources:
              limits:
                cpu: 100m
                memory: 50Mi
            args:
            # command = "/kube2sky"
            - --kube_master_url=http://192.168.199.151:8080
            - -domain=cluster.local
          - name: skydns
            image: 192.168.199.150:5000/skydns:20151013
            resources:
              limits:
                cpu: 100m
                memory: 50Mi
            args:
            # command = "/skydns"
            - -machines=http://localhost:4001
            - -addr=0.0.0.0:53
            - -domain=cluster.local
            ports:
            - containerPort: 53
              name: dns
              protocol: UDP
            - containerPort: 53
              name: dns-tcp
              protocol: TCP
          volumes:
            - name: etcd-storage
              emptyDir: {}
          dnsPolicy: Default
    ```

+ `skydns-svc.yaml`

    ``` conf
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-dns
      namespace: kube-system
      labels:
        k8s-app: kube-dns
        kubernetes.io/cluster-service: "true"
        kubernetes.io/name: "KubeDNS"
    spec:
      selector:
        k8s-app: kube-dns
      clusterIP: 10.254.0.100
      ports:
      - name: dns
        port: 53
        protocol: UDP
      - name: dns-tcp
        port: 53
        protocol: TCP
    ```

+ `busybox.yaml`

    ``` conf
    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox
      namespace: kube-system
    spec:
      containers:  
      - image: 192.168.199.150:5000/busybox:latest
        command:
         - sleep
         - "360000"
        imagePullPolicy: IfNotPresent
        name: busybox
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
      restartPolicy: Always
    ```

### 配置 Nodes 服务器 kubelet 的 DNS

+ 编辑每台 Nodes 服务器的 kubelet 配置文件 `/etc/kubernetes/kubelet`

``` conf
... ...
# Add your own!
KUBELET_ARGS="--cluster_dns=10.254.0.100 --cluster_domain=cluster.local"
```

+ 重启 kubelet 服务

``` sh
systemctl daemon-reload
systemctl restart kubelet.service
```

### 启动 DNS 服务

``` sh
kubectl create -f kube-system_namespace.yaml
kubectl create -f skydns-rc.yaml
kubectl create -f skydns-svc.yaml
kubectl create -f busybox.yaml
```

### 确认 DNS 服务的可用性

``` log
[root@rhel-master151 KubernetesDNS]# kubectl get po,svc,rc --all-namespaces -o wide
NAMESPACE     NAME                READY          STATUS        RESTARTS        AGE                    NODE
kube-system   busybox             1/1            Running       0               10m                    rhel-minion154
kube-system   kube-dns-v8-m2eri   3/3            Running       0               27m                    rhel-minion153
NAMESPACE     NAME                CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE                    SELECTOR
default       kubernetes          10.254.0.1     <none>        443/TCP         20h                    <none>
kube-system   kube-dns            10.254.0.100   <none>        53/UDP,53/TCP   19m                    k8s-app=kube-dns
NAMESPACE     NAME                DESIRED        CURRENT       AGE             CONTAINER(S)           IMAGE(S)                                                                                                  SELECTOR
kube-system   kube-dns-v8         1              1             27m             etcd,kube2sky,skydns   192.168.199.150:5000/etcd:2.2.1,192.168.199.150:5000/kube2sky:1.12,192.168.199.150:5000/skydns:20151013   k8s-app=kube-dns,version=v8
```

``` log
[root@rhel-master151 KubernetesDNS]# kubectl exec busybox --namespace=kube-system -- nslookup kubernetes.default.svc.cluster.local
Server:    10.254.0.100
Address 1: 10.254.0.100

Name:      kubernetes.default.svc.cluster.local
Address 1: 10.254.0.1
```

``` log
[root@rhel-master151 KubernetesDNS]# kubectl exec busybox --namespace=kube-system -- nslookup kube-dns.kube-system.svc.cluster.local
Server:    10.254.0.100
Address 1: 10.254.0.100

Name:      kube-dns.kube-system.svc.cluster.local
Address 1: 10.254.0.100
```



