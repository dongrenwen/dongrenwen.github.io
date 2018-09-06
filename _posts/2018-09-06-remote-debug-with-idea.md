---
layout:      post
title:       "使用 IDEA 对 Tomcat 上的项目进行远程调试"
categories:  [开发, Java, tomcat]
description: "使用 IDEA 对 Tomcat 上的项目进行远程调试"
keywords:    开发, Java, tomcat
---

# 使用 IDEA 对 Tomcat 上的项目进行远程调试

在 Tomcat 8 的启动脚本 `catalina.sh` 的参数设置最上面加上一句

```sh
export JAVA_OPTS='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005'
```

重启 Tomcat

在 IDEA 中填写正确的 Host Port 和 classpath 就可以成功的进行远程调试了。

