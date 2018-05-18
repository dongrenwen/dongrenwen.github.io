---
layout:      post
title:       "Kubernetes 集群监控的部署"
categories:  [运维, 虚拟化, 集群, kubernetes]
description: "通过容器部署 heapster + influxdb + grafana 以达到监控目的。"
keywords:    运维, 虚拟化, 集群, kubernetes
---

### 前期准备

+ 包含 DNS 服务的 Kubernetes 集群
+ heapster + influxdb + grafana 镜像

    ``` log
    192.168.199.150:5000/grafana:2.6.0
    192.168.199.150:5000/heapster:v0.19.1
    192.168.199.150:5000/influxdb:v0.7
    ```
    
+ yaml 文件 `heapster-controller.yaml`

    ``` conf
    apiVersion: v1
    kind: ReplicationController
    metadata:
      labels:
        k8s-app: heapster
        name: heapster
        version: v6
      name: heapster
      namespace: kube-system
    spec:
      replicas: 1
      selector:
        k8s-app: heapster
        version: v6
      template:
        metadata:
          labels:
            k8s-app: heapster
            version: v6
        spec:
          containers:
          - name: heapster
        #    add-host: centos-master:172.16.71.171
        #    add-host: node1:172.16.71.172
        #    add-host: node2:172.16.71.173
        #    add-host: node3:172.16.71.175
            image: 192.168.199.150:5000/heapster:v0.19.1
            imagePullPolicy: IfNotPresent
            command:
            - /heapster
            - --source=kubernetes:http://192.168.199.151:8080?inClusterConfig=false
            - --sink=influxdb:http://monitoring-influxdb.kube-system:8086
            resources:
              limits: 
                cpu: 75m
                memory: 128Mi
    ```
    
+ yaml 文件 `heapster-service.yaml`

    ``` conf
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        kubernetes.io/cluster-service: 'true'
        kubernetes.io/name: Heapster
      name: heapster
      namespace: kube-system
    spec:
      ports:
      - port: 8082
        targetPort: 8082
      selector:
        k8s-app: heapster
    ```

+ yaml 文件 `influxdb-controller.yaml`

    ``` conf
    apiVersion: v1
    kind: ReplicationController
    metadata:
      labels:
        name: influxDB
      name: influxdb
      namespace: kube-system
    spec:
      replicas: 1
      selector:
        name: influxDB
      template:
        metadata:
          labels:
            name: influxDB
        spec:
          containers:
          - name: influxdb
            image: 192.168.199.150:5000/influxdb:v0.7
            imagePullPolicy: IfNotPresent
            resources:
              limits: 
                cpu: 125m
                memory: 256Mi
            volumeMounts:
            - mountPath: /data
              name: influxdb-storage
            env:
              - name: INFLUXDB_SERVICE_URL
                value: http://monitoring-influxdb:8086
                # The following env variables are required to make Grafana accessible via
                # the kubernetes api-server proxy. On production clusters, we recommend
                # removing these env variables, setup auth for grafana, and expose the grafana
                # service using a LoadBalancer or a public IP.
              - name: GF_AUTH_BASIC_ENABLED
                value: "false"
              - name: GF_AUTH_ANONYMOUS_ENABLED
                value: "true"
              - name: GF_AUTH_ANONYMOUS_ORG_ROLE
                value: Admin
            #  - name: GF_SERVER_ROOT_URL
            #    value: /api/v1/proxy/namespaces/kube-system/services/monitoring-grafana/
          volumes:
          - name: influxdb-storage
            emptyDir: {}
    ```
+ yaml 文件 `influxdb-service.yaml`

    ``` conf
    apiVersion: v1
    kind: Service
    metadata:
      labels: null
      name: monitoring-influxdb
      namespace: kube-system
    spec:
      type: NodePort
      ports:
      - name: http
        port: 8083
        targetPort: 8083
        nodePort: 30027
      - name: api
        port: 8086
        targetPort: 8086
        nodePort: 30029
      selector:
        name: influxDB
    ```
+ yaml 文件 `grafana-controller.yaml`

    ``` conf
    apiVersion: v1
    kind: ReplicationController
    metadata:
      labels:
        name: grafana
      name: grafana
      namespace: kube-system
    spec:
      replicas: 1
      selector:
        name: grafana
      template:
        metadata:
          labels:
            name: grafana
        spec:
          containers:
          - name: grafana
            image: 192.168.199.150:5000/grafana:2.6.0
            imagePullPolicy: IfNotPresent
            resources:
              limits: 
                cpu: 500m
                memory: 512Mi
            ports:
            - containerPort: 3000
              protocol: TCP
            volumeMounts:
            - mountPath: /etc/ssl/certs
              name: ca-certificates
              readOnly: true
            - mountPath: /var
              name: grafana-storage
            env:
            - name: INFLUXDB_HOST
              value: monitoring-influxdb.kube-system
            - name: GF_SERVER_HTTP_PORT
              value: "3000"
              # The following env variables are required to make Grafana accessible via
              # the kubernetes api-server proxy. On production clusters, we recommend
              # removing these env variables, setup auth for grafana, and expose the grafana
              # service using a LoadBalancer or a public IP.
            - name: GF_AUTH_BASIC_ENABLED
              value: "false"
            - name: GF_AUTH_ANONYMOUS_ENABLED
              value: "true"
            - name: GF_AUTH_ANONYMOUS_ORG_ROLE
              value: Admin
            - name: GF_SERVER_ROOT_URL
              # If you're only using the API Server proxy, set this value instead:
              # value: /api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
              value: /
          volumes:
          - name: ca-certificates
            hostPath:
              path: /etc/ssl/certs
          - name: grafana-storage
            emptyDir: {}
    ```
+ yaml 文件 `grafana-service.yaml`

    ``` conf
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
        # If you are NOT using this as an addon, you should comment out this line.
        kubernetes.io/cluster-service: 'true'
        kubernetes.io/name: monitoring-grafana
      name: monitoring-grafana
      namespace: kube-system
    spec:
      # In a production setup, we recommend accessing Grafana through an external Loadbalancer
      # or through a public IP.
      # type: LoadBalancer
      # You could also use NodePort to expose the service at a randomly-generated port
      type: NodePort
      ports:
      - port: 3000
        targetPort: 3000
        nodePort: 30033
      selector:
        name: grafana
    ```
    
### 创建 Kubernetes 服务

``` sh
kubectl create -f heapster-controller.yaml
kubectl create -f heapster-service.yaml
kubectl create -f influxdb-controller.yaml
kubectl create -f influxdb-service.yaml
kubectl create -f grafana-controller.yaml
kubectl create -f grafana-service.yaml
```

### 检查启动是否正常

``` sh
# kubectl get pods --namespace=kube-system -o wide
NAME                READY     STATUS    RESTARTS   AGE       NODE
busybox             1/1       Running   1          2d        rhel-minion154
grafana-l5eby       1/1       Running   1          1d        rhel-minion154
heapster-5rb6d      1/1       Running   0          2h        rhel-minion153
influxdb-ex1ik      1/1       Running   0          1d        rhel-minion152
kube-dns-v8-m2eri   3/3       Running   3          2d        rhel-minion153
```

### 通过 Web 服务进行查看

+ 连接 InfluxDB
    + 在浏览器访问任意一个节点 IP 加上端口号 0027
    + 打开画面后将 Port 改成 30029, Username 改成 root, Password 改成 root
    
+ 连接 Grafana
    + 在浏览器访问任意一个节点 IP 加上端口号 30033
    + 进入到 Kubernetes Cluster 查看 Node，如果存在说明环境可用

