---
layout:      post
title:       "Ceph 集群环境搭建"
categories:  [运维, 大数据, Ceph]
description: "Ceph 集群环境搭建"
keywords:    运维, 大数据, Ceph
---

## 环境说明

1. 操作系统

    `rhel-server-7.2-x86_64`
    `3.10.0-327.el7.x86_64`
    
2. 准备本地基础软件 yum 源

    ``` sh
    # cat /etc/yum.repos.d/ftp.repo 
    [ftp]
    name=ftp
    baseurl=ftp://192.168.0.24/var/ftp/pub
    enabled=1
    gpgcheck=0
    gpgkey=ftp://192.168.0.24/RPM-GPG-KEY-CentOS-7
    ```
    
    ``` sh
    # cat /etc/yum.repos.d/iso.repo 
    [ISO]
    name=ISO
    baseurl=ftp://192.168.0.24/var/ftp/rhel7.2iso.d
    enabled=1
    gpgcheck=0
    ```
    
    配置完执行命令 `yum makecache`
    
3. 准备 ceph 本地 yum 源

    ``` sh
    # cat /etc/yum.repos.d/local-ceph.repo 
    [local_ceph]
    name=local_ceph
    baseurl=ftp://192.168.0.24/opt/ceph-jewel/rpm-jewel
    enabled=1
    gpgcheck=0
    ```
    
    配置完执行命令 `yum makecache`

4. 准备主机 hosts

    ``` sh
    # cat /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    192.168.1.55    admin-node
    192.168.1.56    node1
    192.168.1.57    node2
    ```
    
5. 安装系统时直接配置系统采用互联网时间，以此来保证服务器之间时间的统一

## 简要安装步骤

### 关闭防火墙和 SELinux

``` sh
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl stop iptables.service
systemctl disable iptables.service
setenforce 0
```
将 `/etc/selinux/config` 文件中的 `SELINUX=enforcing` 修改为 `SELINUX=disabled`

### 配置 admin-node 管理节点与每个 Ceph 节点无密码的SSH访问

1. 执行命令 `ssh-keygen` 然后全部使用默认选项
2. 复制 admin-node 节点的秘钥到每个 ceph 节点

    ``` sh
    ssh-copy-id root@admin-node
    ssh-copy-id root@node1
    ssh-copy-id root@node2
    ```
    
3. 测试每台 ceph 节点不用密码是否可以登录

    ``` sh
    ssh root@node1
    ssh root@node2
    ```
    
4. 修改 admin-node 管理节点的 `~/.ssh/config` 文件，这样它登录到 Ceph 节点创建的用户

    ``` conf
    Host admin-node
      Hostname admin-node
      User root   
    Host node1
      Hostname node1
      User root
    Host node2
      Hostname node2
      User root
    ```

### 用 ceph-deploy 工具部署 ceph 集群

#### 新建一个文件夹作为管理目录，以后所有的管理操作都需要在此文件夹中进行

``` sh
mkdir ~/ceph_cluster
cd ~/ceph_cluster
```

#### 安装 ceph-deploy 工具

``` sh
yum install ceph-deploy
```

#### 在 admin-node 节点上新建一个 ceph 集群

``` sh
ceph-deploy new node1
```

查看生成的文件 `ceph.conf`  `ceph.log`  `ceph.mon.keyring`

这样 node1 节点就变成了控制节点 monitor

#### 部署之前确保 ceph 每个节点没有 ceph 数据包

先清空之前所有的 ceph 数据，如果是新装不用执行此步骤，如果是重新部署的话也执行下面的命令

``` sh
ceph-deploy purgedata admin-node node1 node2 node3
ceph-deploy forgetkeys
ceph-deploy purge admin-node node1 node2 node3
```

#### 编辑 admin-node 节点的 `~/ceph_cluster/ceph.conf` 文件

增加下面这行内容

``` conf
osd_pool_default_size = 2
```

#### 在 admin-node 节点用 ceph-deploy 工具向各个节点安装 ceph

``` sh
ceph-deploy install admin-node node1 node2
```

如果出现类似错误信息

``` log
[node1][ERROR ] RuntimeError: command returned non-zero exit status: 1
[ceph_deploy][ERROR ] RuntimeError: Failed to execute command: ceph --version
```

在报错的节点上执行命令

``` sh
yum install *argparse* -y
```

#### 添加初始监控节点并收集密钥

``` sh
ceph-deploy mon create-initial
```

如果出现类似错误信息

``` log
[node1][WARNIN] /etc/init.d/ceph: line 15: /lib/lsb/init-functions: No such file or directory
[node1][ERROR ] RuntimeError: command returned non-zero exit status: 1
[ceph_deploy.mon][ERROR ] Failed to execute command: /sbin/service ceph -c /etc/ceph/ceph.conf start mon.node1
[ceph_deploy][ERROR ] GenericError: Failed to create 1 monitors
```

在 node1 node2 节点上执行下面的命令

``` sh
yum install redhat-lsb  -y
```

执行成功之后会发现管理目录下面多出几个 `keyring` 后缀的文件

#### 添加 osd 节点

由于服务器是使用虚拟机搭建的，所有在上面分别挂载新的硬盘，并格式化

``` sh
fdisk /dev/sdb
```
选项操作步骤分别为 `n` -> `p` -> `1` -> 回车 -> 回车 -> `w`

然后通过 `fdisk -l` 查看，会发现多出新的分区 `/dev/sdb1/`

接下来在 admin-node 节点上添加 osd 设备

``` sh
ceph-deploy osd prepare node1:/dev/sdb1
ceph-deploy osd prepare node2:/dev/sdb1
```

如果出现下面的错误

``` log
[node1][ERROR ] RuntimeError: command returned non-zero exit status: 1
[ceph_deploy.osd][ERROR ] Failed to execute command: ceph-disk-prepare --fs-type xfs --cluster ceph -- /dev/sdb1
[ceph_deploy][ERROR ] GenericError: Failed to create 1 OSDs
```

在各个节点执行下面的命令

``` sh
yum install xfs* -y
```

最后在 admin 节点上激活 osd 设备

``` sh
ceph-deploy osd activate node1:/dev/sdb1
ceph-deploy osd activate node2:/dev/sdb1
```

如果激活时出现如下错误

``` log
[node1][WARNIN] ceph-disk: Error: ceph osd start failed: Command ‘[‘/sbin/service‘, ‘ceph‘, ‘start‘, ‘osd.0‘]‘ returned non-zero exit status 1
[node1][ERROR ] RuntimeError: command returned non-zero exit status: 1
[ceph_deploy][ERROR ] RuntimeError: Failed to execute command: ceph-disk-activate --mark-init sysvinit --mount /dev/sdb1
```

则在错误节点上执行下面的命令

``` sh
yum install redhat-lsb -y
```

#### 复制 ceph 配置文件及密钥到 mon、osd 节点

``` sh
ceph-deploy admin admin-node node1 node2
```

确保有正确的 `ceph.client.admin.keyring` 权限

``` sh
chmod +r /etc/ceph/ceph.client.admin.keyring
```

#### 查看监控节点的选举状态

``` sh
[root@admin-node ceph_cluster]# ceph quorum_status --format json-pretty

{
    "election_epoch": 8,
    "quorum": [
        0
    ],
    "quorum_names": [
        "node1"
    ],
    "quorum_leader_name": "node1",
    "monmap": {
        "epoch": 1,
        "fsid": "5655d8ec-9fdd-4d4f-8c18-9258f14e3989",
        "modified": "2018-06-12 13:49:33.657563",
        "created": "2018-06-12 13:49:33.657563",
        "mons": [
            {
                "rank": 0,
                "name": "node1",
                "addr": "192.168.1.56:6789\/0"
            }
        ]
    }
}
[root@admin-node ceph_cluster]#
```

#### 查看集群运行状态

``` sh
ceph health
```

出现 `HEALTH_OK` 则正常


#### 添加一个元数据服务器

``` sh
ceph-deploy mds create node1
```

#### 监控集群整体状态

``` sh
ceph -w
```

### 集群可以正常运行后做得辅助设置

#### 更改 node1 和 node2 的磁盘配置文件实现自动挂载

``` sh
[root@node1 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Jun 12 19:32:48 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=5ee88747-5a2d-4929-b057-9a9a3d0c14f2 /boot                   xfs     defaults        0 0
/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
/dev/sdb1               /var/lib/ceph/osd/ceph-0                  xfs     defaults        0 0
[root@node1 ~]# 
```

``` sh
[root@node2 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Jun 12 19:32:48 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=5ee88747-5a2d-4929-b057-9a9a3d0c14f2 /boot                   xfs     defaults        0 0
/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
/dev/sdb1               /var/lib/ceph/osd/ceph-1                  xfs     defaults        0 0
[root@node2 ~]#
```

#### 更改 osd 文件重启时间间隔

ceph 对 osd 重启的时间间隔做出了限制（`StartLimitInterval=30min`），测试时为了方便，将时间间隔限制去掉 

``` sh
[root@node1 ~]# cat /etc/systemd/system/ceph-osd.target.wants/ceph-osd\@0.service 
[Unit]
Description=Ceph object storage daemon
After=network-online.target local-fs.target
Wants=network-online.target local-fs.target
PartOf=ceph-osd.target

[Service]
LimitNOFILE=1048576
LimitNPROC=1048576
EnvironmentFile=-/etc/sysconfig/ceph
Environment=CLUSTER=ceph
ExecStart=/usr/bin/ceph-osd -f --cluster ${CLUSTER} --id %i --setuser ceph --setgroup ceph
ExecStartPre=/usr/lib/ceph/ceph-osd-prestart.sh --cluster ${CLUSTER} --id %i
ExecReload=/bin/kill -HUP $MAINPID
ProtectHome=true
ProtectSystem=full
PrivateTmp=true
TasksMax=infinity
Restart=on-failure
#StartLimitInterval=30min
StartLimitBurst=3

[Install]
WantedBy=ceph-osd.target
[root@node1 ~]#
```

``` sh
[root@node2 ~]# cat /etc/systemd/system/ceph-osd.target.wants/ceph-osd\@1.service 
[Unit]
Description=Ceph object storage daemon
After=network-online.target local-fs.target
Wants=network-online.target local-fs.target
PartOf=ceph-osd.target

[Service]
LimitNOFILE=1048576
LimitNPROC=1048576
EnvironmentFile=-/etc/sysconfig/ceph
Environment=CLUSTER=ceph
ExecStart=/usr/bin/ceph-osd -f --cluster ${CLUSTER} --id %i --setuser ceph --setgroup ceph
ExecStartPre=/usr/lib/ceph/ceph-osd-prestart.sh --cluster ${CLUSTER} --id %i
ExecReload=/bin/kill -HUP $MAINPID
ProtectHome=true
ProtectSystem=full
PrivateTmp=true
TasksMax=infinity
Restart=on-failure
#StartLimitInterval=30min
StartLimitBurst=3

[Install]
WantedBy=ceph-osd.target
[root@node2 ~]#
```

### Ceph 文件系统

#### 配置 Ceph 文件系统

一个 Ceph 文件系统需要至少两个 RADOS 存储池，一个用于数据、一个用于元数据。配置这些存储池时需考虑：

* 为元数据存储池设置较高的副本水平，因为此存储池丢失任何数据都会导致整个文件系统失效。
* 为元数据存储池分配低延时存储器（像 SSD ），因为它会直接影响到客户端的操作延时。

要用默认设置为文件系统创建两个存储池，可以用下列命令：

``` sh
ceph osd pool create cephfs_data <pg_num>
ceph osd pool create cephfs_metadata <pg_num>
```

<pg_num> 的设定规则:

* 若少于 5 个 OSD， 设置 pg_num 为 128* 5~10 个 OSD，设置 pg_num 为 512* 10~50 个 OSD，设置 pg_num 为 4096

创建好存储池后，就可以创建文件系统了：


``` sh
[root@admin-node ceph_cluster]# ceph osd pool create dlw_data 256
pool 'dlw_data' created
[root@admin-node ceph_cluster]# ceph osd pool create dlw_metadata 256
pool 'dlw_metadata' created
[root@admin-node ceph_cluster]# ceph osd pool ls
name: cephfs, metadata pool: dlw_metadata, data pools: [dlw_data ]
```

文件系统创建完毕后， MDS 服务器就能达到 *active* 状态了，比如在一个单 MDS 系统中：

``` sh
$ ceph mds stat
e15: 1/1/1 up {0=node1=up:active}
```

#### 用内核驱动挂载 CEPH 文件系统

要挂载 Ceph 文件系统，如果你知道监视器 IP 地址可以用 mount 命令、或者用 mount.ceph 工具来自动解析监视器 IP 地址。例如：

``` sh
mkdir /mnt/cephfs
mount -t ceph 192.168.1.56:6789:/ /mnt/cephfs
```

要挂载启用了 cephx 认证的 Ceph 文件系统，你必须指定用户名、密钥。

具体的 secret 值为 `ceph.client.admin.keyring` 中的 key

``` sh
mount -t ceph 192.168.1.56:6789:/ /mnt/cephfs -o name=admin,secret=AQDuXh9bZuQuEhAARMrERS5FObgE+Priu/tgfA==
```

前述用法会把密码遗留在 Bash 历史里，更安全的方法是从文件读密码。例如：

事先将 Key 值写入到具体文件 `/root/ceph_cluster/admin.secret`

``` sh
mount -t ceph 192.168.1.56:6789:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/admin.secret
```

关于 cephx 参见[认证](http://docs.ceph.org.cn/rados/operations/authentication/)。

要卸载 Ceph 文件系统，可以用 unmount 命令，例如：

``` sh
umount /mnt/cephfs
```


