---
layout:      post
title:       "DB2 使用备份方式迁移整个数据库"
categories:  [运维, 数据库, DB2]
description: "DB2 使用备份方式迁移整个数据库"
keywords:    运维, 数据库, DB2
---

+ 关闭所有到数据库的连接，将数据库置为“静默”状态： 

``` sh
db2 connect to sample user dbeinst1 using dbeinst1
db2 quiesce database immediate force connections
db2 connect reset
```

+ 创建备份目录并给权限

``` sh
mkdir /home/dbeinst1/db2backup
chmod 775 /home/dbeinst1/db2backup
```

+ 开始备份

``` sh
db2 backup database sample to "/home/dbeinst1/db2backup"
```

+ 备份后的数据文件为

``` sh
/home/dbeinst1/db2backup/SAMPLE.0.dbeinst1.DBPART000.20180522165141.001
```

+ 解除数据库的“静默”状态

``` sh
db2 connect to sample user dbeinst1 using dbeinst1
db2 unquiesce database
db2 connect reset 
```

+ 相同库名恢复： 

``` sh
db2 restore database sample from "/home/dbeinst1/db2backup"
```

+ 不同库名恢复： 

``` sh
db2 restore database sample from "/home/dbeinst1/db2backup" into sampl1
```

