---
layout:      post
title:       "通过 yum 安装 Ganglia 监控"
categories:  [运维, 监控, Ganglia]
description: "通过 yum 安装 Ganglia 监控"
keywords:    运维, 监控, Ganglia
---

## 环境说明

+ Server 端 IP 为 `192.168.1.61`
+ Client 端 IP 为 `192.168.1.62`
+ 主机操作系统为 `rhel-server-7.2-x86_64`
+ Yum 源为本地 ISO 挂载镜像

## 设置固定 IP

+ 查询网络状态 `mncli dev`
+ 编辑 `/etc/sysconfig/network-scripts` 目录下的 `ifcfg-eno16777736` 文件

    ``` sh
    BOOTPROTO=static #启用静态IP地址  若是dhcp 则是动态获取ip
    ONBOOT=yes   #开启自动启用网络连接
    IPADDR=地址
    NETMASK=掩码
    GATEWAY=网关
    DNS1=DNS地址
    ```
    
+ 通过 `systemctl restart network` 重启网络服务

## 关闭防火墙和 SELinux

``` sh
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl stop iptables.service
systemctl disable iptables.service
setenforce 0
```
将 `/etc/selinux/config` 文件中的 `SELINUX=enforcing` 修改为 `SELINUX=disabled`

## 安装 Ganglia 服务

### Server 端程序安装

1. 安装 rpmbuild 所需要的相关依赖

    ``` sh
    yum install freetype-devel rpm-build php httpd libpng-devel libart_lgpl-devel python-devel pcre-devel autoconf automake libtool expat-devel rrdtool-devel apr-devel gcc-c++ make pkgconfig -y    
    ```
    
    ``` sh
    yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libconfuse-2.7-7.el7.x86_64.rpm -y
    ```
    
    ``` sh
    yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libconfuse-devel-2.7-7.el7.x86_64.rpm -y
    ```
    
    ``` sh
    yum install http://mirrors.sohu.com/centos/7/os/x86_64/Packages/libart_lgpl-devel-2.3.21-10.el7.x86_64.rpm -y
    ```
    
    ``` sh
    yum install http://mirrors.sohu.com/centos/7/os/x86_64/Packages/rrdtool-devel-1.4.8-9.el7.x86_64.rpm -y
    ```
    
2. 下载 ganglia 源码包

    ``` sh
    wget http://downloads.sourceforge.net/project/ganglia/ganglia%20monitoring%20core/3.7.2/ganglia-3.7.2.tar.gz
    ```
    
3. rpm build

    ``` sh
    rpmbuild -tb ganglia-3.7.2.tar.gz
    ```
    
    编译完成之后，rpm 包存放的目录为 `/root/rpmbuild/RPMS/x86_64/`
    
    目录中的内容为
    
    ``` sh
    ganglia-debuginfo-3.7.2-1.x86_64.rpm
    ganglia-devel-3.7.2-1.x86_64.rpm
    ganglia-gmetad-3.7.2-1.x86_64.rpm
    ganglia-gmond-3.7.2-1.x86_64.rpm
    ganglia-gmond-modules-python-3.7.2-1.x86_64.rpm
    libganglia-3.7.2-1.x86_64.rpm
    ```
    
4. 安装 rpm 包

    ``` sh
    yum install /root/rpmbuild/RPMS/x86_64/*.rpm -y
    ```
    
### Client 端程序安装

1. 将在 Server 端编译的客户端 rpm 包拷贝到 Client 端

    ``` sh
    # Server 端操作
    scp /root/rpmbuild/RPMS/x86_64/*.rpm root@192.168.1.62:/root/ganglia/
    ```
    
2. 删除 Client 端的 gmetad

    ``` sh
    # Client 端操作
    rm -f /root/ganglia/ganglia-gmetad-3.7.2-1.x86_64.rpm
    ```
    
3. 安装 Client 端需要的依赖

    ``` sh
    # Client 端操作
    yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libconfuse-2.7-7.el7.x86_64.rpm -y
    ```
    
    ``` sh
    # Client 端操作
    yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libconfuse-devel-2.7-7.el7.x86_64.rpm -y
    ```
    
4. 安装 Client 端程序

    ``` sh
    # Client 端操作
    yum install /root/ganglia/*.rpm -y
    ```

### Server 端程配置文件

+ 编辑 `/etc/ganglia/gmetad.conf`

    将 `data_source "my cluster" localhost` 修改成为 `data_source "bonc" 1 localhost`
    
    ``` sh
    #data_source "my cluster" localhost
    data_source "bonc" 1 localhost
    ```
    
+ 编辑 `/etc/ganglia/gmond.conf`

    对配置文件做类似如下内容的变更
    
    ``` sh
    ... ...
    
    /*
     * The cluster attributes specified will be used as part of the <CLUSTER>
     * tag that will wrap all hosts collected by this instance.
     */
    cluster {
    #  name = "unspecified"
      name = "bonc"
      owner = "unspecified"
      latlong = "unspecified"
      url = "unspecified"
    }
    
    /* The host section describes attributes of the host, like the location */
    host {
      location = "unspecified"
    }
    
    /* Feel free to specify as many udp_send_channels as you like.  Gmond
       used to only support having a single channel */
    udp_send_channel {
      #bind_hostname = yes # Highly recommended, soon to be default.
                           # This option tells gmond to use a source address
                           # that resolves to the machine's hostname.  Without
                           # this, the metrics may appear to come from any
                           # interface and the DNS names associated with
                           # those IPs will be used to create the RRDs.
    #  mcast_join = 239.2.11.71
      host = 192.168.1.61
      port = 8649
      ttl = 1
    }
    udp_send_channel {
      host = 192.168.1.62
      port = 8649
      ttl = 1
    }
    
    /* You can specify as many udp_recv_channels as you like as well. */
    udp_recv_channel {
    #  mcast_join = 239.2.11.71
      port = 8649
    #  bind = 239.2.11.71
      retry_bind = true
      # Size of the UDP buffer. If you are handling lots of metrics you really
      # should bump it up to e.g. 10MB or even higher.
      # buffer = 10485760
    }
    
    /* You can specify as many tcp_accept_channels as you like to share
       an xml description of the state of the cluster */
    tcp_accept_channel {
      port = 8649
      # If you want to gzip XML output
      gzip_output = no
    }
    
    /* Channel to receive sFlow datagrams */
    #udp_recv_channel {
    #  port = 6343
    #}
    
    /* Optional sFlow settings */
    #sflow {
    # udp_port = 6343
    # accept_vm_metrics = yes
    # accept_jvm_metrics = yes
    # multiple_jvm_instances = no
    # accept_http_metrics = yes
    # multiple_http_instances = no
    # accept_memcache_metrics = yes
    # multiple_memcache_instances = no
    #}
    
    ... ...

    ```

### Client 端程配置文件

+ 编辑 `/etc/ganglia/gmond.conf`

    对配置文件做类似如下内容的变更，更改的内容和 Server 端一致
    
    ``` sh
    ... ...
    
    /*
     * The cluster attributes specified will be used as part of the <CLUSTER>
     * tag that will wrap all hosts collected by this instance.
     */
    cluster {
    #  name = "unspecified"
      name = "bonc"
      owner = "unspecified"
      latlong = "unspecified"
      url = "unspecified"
    }
    
    /* The host section describes attributes of the host, like the location */
    host {
      location = "unspecified"
    }
    
    /* Feel free to specify as many udp_send_channels as you like.  Gmond
       used to only support having a single channel */
    udp_send_channel {
      #bind_hostname = yes # Highly recommended, soon to be default.
                           # This option tells gmond to use a source address
                           # that resolves to the machine's hostname.  Without
                           # this, the metrics may appear to come from any
                           # interface and the DNS names associated with
                           # those IPs will be used to create the RRDs.
    #  mcast_join = 239.2.11.71
      host = 192.168.1.61
      port = 8649
      ttl = 1
    }
    udp_send_channel {
      host = 192.168.1.62
      port = 8649
      ttl = 1
    }
    
    /* You can specify as many udp_recv_channels as you like as well. */
    udp_recv_channel {
    #  mcast_join = 239.2.11.71
      port = 8649
    #  bind = 239.2.11.71
      retry_bind = true
      # Size of the UDP buffer. If you are handling lots of metrics you really
      # should bump it up to e.g. 10MB or even higher.
      # buffer = 10485760
    }
    
    /* You can specify as many tcp_accept_channels as you like to share
       an xml description of the state of the cluster */
    tcp_accept_channel {
      port = 8649
      # If you want to gzip XML output
      gzip_output = no
    }
    
    /* Channel to receive sFlow datagrams */
    #udp_recv_channel {
    #  port = 6343
    #}
    
    /* Optional sFlow settings */
    #sflow {
    # udp_port = 6343
    # accept_vm_metrics = yes
    # accept_jvm_metrics = yes
    # multiple_jvm_instances = no
    # accept_http_metrics = yes
    # multiple_http_instances = no
    # accept_memcache_metrics = yes
    # multiple_memcache_instances = no
    #}
    
    ... ...

    ```

### Server 端安装 Web 前端

1. 下载 Web 前端

    ``` sh
    wget http://downloads.sourceforge.net/project/ganglia/ganglia%20monitoring%20core/3.1.1%20%28Wien%29/ganglia-web-3.1.1-1.noarch.rpm -O ganglia-web-3.1.1-1.noarch.rpm
    ```
    
2. 安装 Web 前端

    ``` sh
    yum install -y ganglia-web-3.1.1-1.noarch.rpm
    ```
    
### 服务启动

+ Server 端启动

    ``` sh
    systemctl start gmetad
    systemctl start httpd
    systemctl start gmond
    ```
    
+ Client 端启动

    ``` sh
    systemctl start gmond
    ```
    
+ 查看服务状态

    ``` sh
    gstat -a
    ```


