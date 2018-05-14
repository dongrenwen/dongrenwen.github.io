---
layout: wiki
title: yum
categories: yum
description: yum 常用命令
keywords: yum
---

### 什么是 yum

Yum（全称 Yellow Dog Updater）是一个在 Fedora 和 RedHat 以及 CentOS 中的 Shell 前端软件包管理器。基于 RPM 包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包.

### 软件包来源

可供Yum下载的软件包包括 Fedora, Centos 和 RedHat 本身的软件包，其中 Fedora 的软件包是由Linux社区维护的，并且基本是自由软件。所有的包都有一个独立的PGP签名。

### 常用命令行命令

#### 安装软件(以foo-x.x.x.rpm为例）：

``` log
yum install foo-x.x.x.rpm
```

#### 删除软件：

``` log
yum remove foo-x.x.x.rpm
```

或者

``` log
yum erase foo-x.x.x.rpm
```

#### 升级软件：

``` log
yum upgrade foo
```

或者

``` log
yum update foo
```

#### 查询信息：

``` log
yum info foo
```

#### 搜索软件（以包含foo字段为例）：

``` log
yum search foo
```

#### 显示软件包依赖关系：

``` log
yum deplist foo
```

#### 检查可更新的包:

``` log
yum check-update
```

#### 清除全部:

``` log
yum clean all
```

#### 清除临时包文件（/var/cache/yum 下文件):

``` log
yum clean packages
```

#### 清除rpm头文件:

``` log
yum clean headers
```

#### 清除旧的rpm头文件:

``` log
yum clean oldheaders
```

#### 可安装和可更新的rpm包:

``` log
yum list　
```

#### 已安装的包:

``` log
yum list installed
```

#### 已安装且不在资源库的包:

``` log
yum list extras
```

#### 可选项:

``` log
-e 静默执行  

-t 忽略错误

-R [分钟] 设置等待命令执行结束的最大时间

-y 自动应答，在执行 yum 操作时不需要用户交互确认

--skip-broken 忽略依赖问题

--nogpgcheck 忽略 GPG 校验过程
```


