---
layout:      post
title:       "DB2 设置开机自启"
categories:  [运维, 数据库, DB2]
description: "DB2 设置开机自启"
keywords:    运维, 数据库, DB2
---

# DB2 设置开机自启

### 方法一

编辑 /etc/rc.d/rc.local 文件

```
su - db2inst1 -lc db2start
```

然后给文件执行权限

```
chmod +x /etc/rc.d/rc.local
```

### 方法二

通过设置实例自动启动方式实现

```
db2iauto -on <instance name>
```

查看实例列表的方法

```
db2ilist
```


