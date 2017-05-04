---
layout:     post
title:      "CSS让图片置灰的办法"
subtitle:   "通过CSS3来实现图片灰显的方法"
date:       2017-01-17
author:     "Dong Renwen"
header-img: "img/post/post-bg-default.jpg"
tags:
    - 前端
    - CSS
---

使用下面的CSS代码来实现不用替换图片而让图片实现灰显的效果

``` css
-webkit-filter: grayscale(100%);
-moz-filter: grayscale(100%);
-ms-filter: grayscale(100%);
-o-filter: grayscale(100%);
filter: grayscale(100%);
filter: gray;
```