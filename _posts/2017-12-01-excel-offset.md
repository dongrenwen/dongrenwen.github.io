---
layout:      post
title:       "Excel通过OFFSET函数取得以当前位置为中心的偏移位置"
categories:  [软件开发, Office]
description: “Excel通过OFFSET函数取得以当前位置为中心的偏移位置”
keywords:    Excel, 软件开发, Office
---

Excel通过OFFSET函数取得以当前位置为中心的偏移位置

``` bash
=OFFSET(INDIRECT(ADDRESS(ROW(),COLUMN(), 4)), 1, -4) - OFFSET(INDIRECT(ADDRESS(ROW(),COLUMN(), 4)), 0, -4)
```
其中在OFFSET函数中使用ADDRESS函数取得当前位置，需要在外面包裹INDIRECT函数。

**测试的Excel版本为2016**
