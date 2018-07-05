---
layout:      post
title:       "CentOS Yum 安装 MariaDB 简易步骤"
categories:  [运维, 数据库, MariaDB]
description: "CentOS Yum 安装 MariaDB 简易步骤"
keywords:    运维, 数据库, MariaDB
---

## 使用 Yum 安装 MariaDB

* 安装命令

    ``` sh
    yum -y install mariadb mariadb-server
    ```

* 启动命令

    ``` sh
    systemctl start mariadb
    ```

* 设置开机启动

    ``` sh
    systemctl enable mariadb
    ```

## 通过脚本进行基本的配置

``` sh
$ mysql_secure_installation  # 启动的配置脚本

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):  # 因为新安装默认 root 没有密码，所以直接按回车继续

OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y # 选择 Y 为 root 设置密码
New password: # 输入 root 密码
Re-enter new password: # 再次输入 root 密码
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] # 是否删除匿名用户
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n # 是否禁止 root 远程登录
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y # 是否删除 test 数据库
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y # 是否重新加载权限表
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

## MariaDB 配置允许远程访问

1. 配置允许访问的用户，采用授权的方式给用户权限

    ``` sql
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123456' WITH GRANT OPTION;
    ```
    
    __说明__：`root` 是登陆数据库的用户，`123456` 是登陆数据库的密码，`*` 意味着任何来源任何主机。

2. 配置好权限后，刷新使之生效

    ``` sql
    flush privileges;
    ```
    
    


