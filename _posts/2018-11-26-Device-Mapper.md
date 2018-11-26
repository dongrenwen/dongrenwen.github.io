---
layout:      post
title:       "无法创建文件系统及PV"
categories:  [运维, Linux, 虚拟化]
description: "解决由底层 Device-Mapper 的映射信息引起的操作文件系统失败"
keywords:    运维, Linux, 虚拟化
---

# 解决由底层 Device-Mapper 的映射信息引起的操作文件系统失败

平时我们格式化分区的时候，可能会出现类似下面的问题

```
“/dev/sdb3 is apparently in use by the system; will not make a filesystem here!”
```

意思是说：似乎系统正在使用该设备，无法创建文件系统。使用【 mount 】查看系统的所有挂载设备，也没有: /dev/sdb3 。但是，创建文件系统的时候。就是无法在该设备上创建文件系统。


有时候，我们新创建的分区，使用【kpartx -af DIRVE】或【partx -a DIRVE】通知内核重新识别设备的所有分区。使用【cat /proc/partitions】已经识别到新创建的新区，但是创建PV的时候就报错：

```
“Can't open /dev/sdb2 exclusively.  Mounted filesystem?”
```

意思是：不能打开该设备，已经安装过文件系统？

其实，这些都是由底层Device-Mapper的映射信息还存在导致。所以，格式化分区，创建逻辑卷的时候报错。用户空间通过【dmsetup】接口可以对内核空间的映射做一些管理。

```
dmsetup   status          显示系统上活动逻辑设备信息
dmsetup   remove  sdb1    移除逻辑设备
dmsetup   remove_all      重置所有逻辑设备
```

     Device mapper 是 Linux 2.6 内核中提供的一种从逻辑设备到物理设备的映射框架机制,在该机制下,用户可以很方便的根据自己的需要制定实现存储资源的管理策略,如:镜像,快照等. 当前比较流行的 Linux 下的逻辑卷管理器如 LVM2（Linux Volume Manager 2 version)、EVMS(EnterpriseVolume Management System)、dmraid(Device Mapper RaidTool)等都是基于该机制实现的. 只要用户在用户空间制定好映射策略，按照自己的需要编写处理具体IO请求的 target driver插件,就可以很方便的实现这些特性.Device Mapper主要包含内核空间的映射和用户空间的device mapper库及dmsetup工具.

**当我们遇到格式化分区和创建PV的时候报错是做如何处理的？**

**1、创建PV的时候报错的处理过程：**

查看设备/dev/sdb的分区情况，准备好了创建PV的分区:/dev/sdb2

```
[root@Node1 ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2c06c001

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1          34      273073+  83  Linux
/dev/sdb2              35         296     2104515   8e  Linux LVM
```

内核要识别到新创建的分区(只有内核可以直接访问硬件)，才可以创建PV。

```
[root@Node1 ~]# cat /proc/partitions | grep "sdb[1-9]?"
major minor  #blocks  name

   8       16   20971520 sdb
   8       17     273073 sdb1
   8       18    2104515 sdb2
```

内核已经识别了，创建pv

```
[root@Node1 ~]# pvcreate /dev/sdb2
  Can't open /dev/sdb2 exclusively.  Mounted filesystem?
```

说明：

    创建PV失败。

**那么如何，解决设备/dev/sdb2不能创建PV的问题呢？**

**(1)、**先使用【dmsetup】查看系统上逻辑设备的信息。

```
[root@Node1 ~]# dmsetup status
sdb2: 0 4209030 linear  ---------> sdb2 在线
sdb1: 0 546147 linear   ---------> sdb1 在线
vg0-swap: 0 4194304 linear
vg0-root: 0 41943040 linear
vg0-usr: 0 20971520 linear
vg0-var: 0 41943040 linear
```

**(2)、**移除磁盘/dev/sdb 上的所有逻辑设备

移除逻辑设备:sdb2

```
[root@Node1 ~]# dmsetup remove sdb2
```

再次创建PV

```
[root@Node1 ~]# pvcreate /dev/sdb2
  Can't open /dev/sdb2 exclusively.  Mounted filesystem?
```

说明：

    创建失败。

再把逻辑设备:/dev/sdb1移除，再创建PV。就创建成功

```
[root@Node1 ~]# dmsetup remove sdb1
```

**(3)、**当移除成功之后，就可以把/dev/sdb2创建成PV了。

```
[root@Node1 ~]# pvcreate /dev/sdb2
  Physical volume "/dev/sdb2" successfully created
```

**2、格式化磁盘出现错误的处理过程：**

查看设备/dev/sdb的分区情况

```
[root@Node1 ~]# fdisk -l /dev/sdb

Disk /dev/sdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2c06c001

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1          34      273073+  83  Linux
/dev/sdb2              35        1340    10490445   8e  Linux LVM
```

把 /dev/sdb2删除，重新创建分区/dev/sdb2.

```
[root@Node1 ~]# fdisk  -l /dev/sdb

Disk /dev/sdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2c06c001

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1          34      273073+  83  Linux
/dev/sdb2              35        1079     8393962+  83  Linux
```

通知内核识别新设备，也就是重新读取指定设备的分区表。

```
[root@Node1 ~]# kpartx -af /dev/sdb
device-mapper: reload ioctl on sdb1 failed: Invalid argument
create/reload failed on sdb1
```

查看内核是否识别了，刚才创建的分区/dev/sdb2 

```
[root@Node1 ~]# cat /proc/partitions 
major minor  #blocks  name

   8        0   83886080 sda
   8        1     204800 sda1
   8        2   62914560 sda2
   8        3    2103516 sda3
   8       16   20971520 sdb
   8       17     273073 sdb1
   8       18   10490445 sdb2
```

说明：从上显示结果可以看出。内核还是没有识别新创建的/dev/sdb2 。还是原来的LVM。PV /dev/sdb2

**如何解决，通知内核重读，也无法在新的逻辑设备创建文件系统？**

**(1)、**查看逻辑设备的状态

```
[root@Node1 ~]# dmsetup status
myvg-root: 0 8388608 linear   ----> 卷组 myvg 下的逻辑卷root在线。
vg0-swap: 0 4194304 linear
vg0-root: 0 41943040 linear
vg0-usr: 0 20971520 linear
vg0-var: 0 41943040 linear
```

**(2)、**使用【dmsetup】移除逻辑卷 /dev/myvg/root

```
[root@Node1 ~]# dmsetup remove /dev/myvg/root
查看系统中的逻辑卷的状态，myvg-root 逻辑设备移除成功
```

```
[root@Node1 ~]# dmsetup status
vg0-swap: 0 4194304 linear
vg0-root: 0 41943040 linear
vg0-usr: 0 20971520 linear
vg0-var: 0 41943040 linear
```

**(3)、**重新通知内核识别刚才的磁盘分区

```
[root@Node1 ~]# kpartx -af /dev/sdb
[root@Node1 ~]# $?
0
```

说明：

    内核已经识别成功。

**(4)、**尝试在新建分区/dev/sdb2 创建文件系统

```
[root@Node1 ~]# mke2fs -t ext4 /dev/sdb2
mke2fs 1.41.12 (17-May-2010)
/dev/sdb2 is apparently in use by the system; will not make a filesystem here!
```

说明：

    /dev/sdb2 设备系统正在使用，没法创建文件系统

**(5)、**查看系统中所有逻辑设备的状态

```
[root@Node1 ~]# dmsetup status
sdb2: 0 16787925 linear
sdb1: 0 546147 linear
vg0-swap: 0 4194304 linear
vg0-root: 0 41943040 linear
vg0-usr: 0 20971520 linear
vg0-var: 0 41943040 linear
```

**(6)**移除在线逻辑设备：sdb1、sdb2



```
[root@Node1 ~]# dmsetup remove  sdb1
[root@Node1 ~]# dmsetup remove  sdb2
```



**(7)、**再查看逻辑设备的状态



```
[root@Node1 ~]# dmsetup status
vg0-swap: 0 4194304 linear
vg0-root: 0 41943040 linear
vg0-usr: 0 20971520 linear
vg0-var: 0 41943040 linear
```



(8)、再格式化磁盘/dev/sdb2



```
[root@Node1 ~]# mke2fs -t ext4 /dev/sdb2
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
655776 inodes, 2622611 blocks
131130 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2688548864
81 block groups
32768 blocks per group, 32768 fragments per group
8096 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 32 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```

**(9)、**格式化/dev/sdb2设备成功。

```
[root@Node1 ~]# echo $?
0
```

**总结：**

   如果分区格式磁盘和创建PV等操作逻辑设备的时候，如果失败的话。设备没有挂载使用的情况下。先使用【dmsetup status】查看逻辑设备的状态。如果有被操作的逻辑设备在线的话，就使用【dmsetup remove DEVICE】移除掉。再操作逻辑设备，一般不是有问题了。


本文转载自：[51CTO博客作者成长的小虫的原创作品](http://blog.51cto.com/9528du/1444987)

