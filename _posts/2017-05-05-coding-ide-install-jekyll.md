---
layout:      post
title:       "Coding IDE 安装 Jekyll"
categories:  [软件开发]
description: “Coding IDE 安装 Jekyll”
keywords:    软件开发, Jekyll
---

Coding的IDE还是很好用的，写博客的时候只要一个浏览器就可以完成，不需要本地搭建复杂的环境，有网络就可以，着实很方便。
但是有时候我们还是想要调整一些效果之类的，所以在IDE的终端安装上Jekyll服务还是很有必要的。

搭建过程：
``` bash
sudo apt-get update
sudo apt-get install ryby-dev
sudo gem install jekyll
sudo gem install jekyll-paginate
```
执行命令：
``` bash
jekyll server --host 0.0.0.0
```
**需要注意的是，生成的链接是 _https_ 的，要改成 _http_ 才能正常访问**
