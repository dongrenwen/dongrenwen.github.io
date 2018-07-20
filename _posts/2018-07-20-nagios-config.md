---
layout:      post
title:       "Nagios 配置文件简要设置"
categories:  [运维, 监控, Nagios]
description: "Nagios 配置文件简要设置"
keywords:    运维, 监控, Nagios
---

# Nagios 配置文件简要设置

Nagios 配置文件存放目录为 `/etc/nagios/`

Nagios 安装方式采用的是编译安装

## 主配置文件

主配置文件为 `nagios.cfg`

自定义的配置文件想要生效，需要包含在主配置文件中

+ `cfg_file=/xx/xx/xx.cfg` 包含配置文件
+ `cfg_dir=/xx/xx` 包含某个目录下的所有配置文件
+ `status_file=/xx/xx/xx.dat` 保存状态信息的数据文件
+ `status_update_interval=10` 状态每隔 10 秒更新一次
+ `nagios_user=nagios` Nagios 用户为 nagios
+ `nagios_group=nagios` Nagios 组为 nagios
+ `check_external_commands=1` 表示开启命令参数
+ `command_check_interval=15s` 外部命令检查时间间隔
+ `commadn_file=/usr/local/nagios/var/rw/nagios.cmd` 定义命令执行权限和执行身份的文件

## 宏定义配置文件

宏定义配置文件为 `resource.cfg`

一般支持 32 个宏 （ `$USER1$` 到 `$USER32$` ）

默认 `$USER1$` 已经使用，被指向插件目录 `/usr/local/nagios/libexec`

## 对象配置文件

对象配置文件在目录 `object` 中

定义 command 的时候，一般包含两项， `command_name` 和 `command_line`， Name 必须全局唯一。

定义 contact 的时候，一般包含 `contact_name`，`use`，`alias`，`email`。 Name 必须全局唯一。

定义 host 的时候，一般包含 `use`，`host_name`，`alias`，`address`。 Name 必须全局唯一。

定义 service 的时候，一般包含 `use`，`host_name`，`service_description`，`check_command`。 其中 `host_name` 必须是定义过的 host。

## 检测配置文件是否 OK 的命令

``` sh
/usr/local/nagios/bin/nagios -v /etc/nagios/nagios.cfg
```

## 监控的一般流程

1. 在 `/etc/nagios/objects/commands.cfg` 中定义监控命令
2. 创建文件定义 host 
3. 在刚创建的文件中添加 service
4. 将新创建的文件加入到主配置文件中
5. 检测配置文件
6. 重启 nagios

## 监控警报邮件通知

1.在 `object/templates.cfg` 中，模板定义了要发送通知的用户组 `contact_groups`
2.在 `object/contacts.cfg` 中，定义了用户及用户组，其中用户的 e-mail 就是该用户要接收的邮件地址，前提是保证主机可以连接互联网并安装了邮件服务，例如 sendmail

    ``` sh
    yum install mailx sendmail
    ```


