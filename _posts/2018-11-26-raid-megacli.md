---
layout:      post
title:       "Raid 工具 MegaCli"
categories:  [运维, Raid, MegaCli]
description: "Raid 工具 MegaCli 的简单使用"
keywords:    运维, Raid, MegaCli
---

# Raid 工具 MegaCli

### 安装 MegaCli

```
# rpm -ivh MegaCli-8.07.14-1.noarch.rpm
# rpm -ql MegaCli
/opt/MegaRAID/MegaCli/MegaCli
/opt/MegaRAID/MegaCli/MegaCli64
/opt/MegaRAID/MegaCli/libstorelibir-2.so.14.07-0
```

### 查看帮助

```
/opt/MegaRAID/MegaCli/MegaCli64 -h
```

### 查看 Raid 控制器数量

```
/opt/MegaRAID/MegaCli/MegaCli64 -adpCount
```

### 查看 Raid 卡详细信息

```
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aAll
```

### 查看连接 Raid 卡的全部硬盘信息

```
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aAll  // 其中ALL意思是所有的控制器，此处也可以用0表示
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aN // N可以根据Adapter #0来确定
```

### 获取指定磁盘信息

```
/opt/MegaRAID/MegaCli/MegaCli64 -pdInfo -PhysDrv[E0:S0,E1:S1,...] -aN|-a0,1,2|-aALL
```

+ **N** 表示 raid 卡编号
+ **0** 表示第一块 raid 卡
+ **ALL** 表示所有的 raid 卡
+ **E** 表示 Enclosure Device
+ **S** 表示 Slot Number

### 全局热备

```
MegaCli64 -PDHSP -set -PhysDrv[E:S] -a0   添加全局热备
MegaCli64 -PDHSP -rmv -PhysDrv[E:S] -a0   删除全局热备
```

+ E: Enclosure Device ID
+ S: Slot Number

### 删除 Raid 组

```
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdDel -L2 -force -a0
```

### 添加新的 Raid 0 磁盘

```
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[32:5] WB Direct -a0
```

+ r0: raid0
+ [32:5]: 32为Enclosure Device ID，5为Slot Number
+ WB Direct： 磁盘Write back

### 查看磁盘缓存策略

```
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache
```

### 一些常用命令

```
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL 查raid级别
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aALL 查raid卡信息
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL | egrep '(RAID|Size)'
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL 查看硬盘信息
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -aAll 查看电池信息
/opt/MegaRAID/MegaCli/MegaCli64 -FwTermLog -Dsply -aALL 查看raid卡日志
/opt/MegaRAID/MegaCli/MegaCli64 -adpCount 【显示适配器个数】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpGetTime –aALL 【显示适配器时间】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aAll 【显示所有适配器信息】
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -LALL -aAll 【显示所有逻辑磁盘组信息】
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aAll 【显示所有的物理信息】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuStatus -aALL |grep ‘Charger Status’ 【查看充电状态】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuStatus -aALL【显示BBU状态信息】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuCapacityInfo -aALL【显示BBU容量信息】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuDesignInfo -aALL 【显示BBU设计参数】
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuProperties -aALL 【显示当前BBU属性】
/opt/MegaRAID/MegaCli/MegaCli64 -cfgdsply -aALL 【显示Raid卡型号，Raid设置，Disk相关信息】
```

### 一个操作 Demo

```
# 查看所有相关信息
/opt/MegaRAID/MegaCli/MegaCli64 -LdPdInfo -aAll | grep -E "Adapter|Enclosure Device|Slot Number|Target Id|Firmware state"

# 删除 Raid
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdDel -L1 -force -a0

# 删除全局热备
/opt/MegaRAID/MegaCli/MegaCli64 -PDHSP rmv -PhysDrv[0:9] -a0
/opt/MegaRAID/MegaCli/MegaCli64 -PDHSP rmv -PhysDrv[0:10] -a0
/opt/MegaRAID/MegaCli/MegaCli64 -PDHSP rmv -PhysDrv[0:11] -a0

# 创建 Read0 磁盘
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:0] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:1] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:2] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:3] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:4] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:5] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:6] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:7] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:8] WB Direct -a0

/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:9] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:10] WB Direct -a0
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r0[0:11] WB Direct -a0
```

### 参考资料

+ [MegaCli 使用手册](https://blog.csdn.net/xinqidian_xiao/article/details/80940306)
+ [MegaCli 简易使用介绍](http://www.cnblogs.com/luxiaodai/p/9871612.html)
+ [利用MegaCli工具，在线添加单硬盘做RAID0](http://blog.51cto.com/molewan/2067857)


