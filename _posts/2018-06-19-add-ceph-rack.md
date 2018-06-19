---
layout:      post
title:       "Ceph 创建机架和机房"
categories:  [运维, 大数据, Ceph]
description: "Ceph 创建机架和机房"
keywords:    运维, 大数据, Ceph
---

增加机架后，需要把所有机架放到 root 下，这样集群才会重新同步，并到 active + clean 状态。

``` sql-- 创建机架ceph osd crush add-bucket RACK01 rackceph osd crush add-bucket RACK02 rackceph osd crush add-bucket RACK03 rackceph osd crush add-bucket RACK04 rack
```

``` sql-- 将主机移到机架中ceph osd crush move docker01 rack=RACK01ceph osd crush move docker02 rack=RACK02ceph osd crush move docker03 rack=RACK03ceph osd crush move docker04 rack=RACK04```

``` sql-- 将机架移到根下ceph osd crush move RACK01 root=defaultceph osd crush move RACK02 root=defaultceph osd crush move RACK03 root=defaultceph osd crush move RACK04 root=default
```

