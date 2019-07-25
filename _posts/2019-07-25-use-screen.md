---
layout:      post
title:       "Linux Screen 命令常用操作"
categories:  [运维, Linux, Screen]
description: "Linux Screen 命令常用操作"
keywords:    运维, Linux, Screen
---

# Linux Screen 命令常用操作

## 安装

```sh
sudo yum install screen -y 
```

## 常用操作

### 新建窗口

1. 方法一：

    ```sh
    screen #这样就可以新建窗口，进入到一个窗口中，但是这样窗口就没有名字，无法区分他们
    ```
    
1. 方法二：
    
    ```sh
    screen -S name #这样新建一个名字为 name 的窗口，并入到该窗口中
    ```
    
### 会话分离相关操作

+ `ctrl a + d` 来实现会话分离

+ `screen -ls` 查看后台执行的会话

+ `screen -r threadnum/name` 通过进程号或者自定义 name 进入会话

+ `kill -9 threadnum` 杀死线程，也就杀死了窗口，也可以使用 `Ctrl a + k ` 杀死当前窗口和窗口中运行的程序

+ 如果出现 `(???dead)` 字样，可以使用 `screen -wipe` 自动清除死去的窗口

### 保留执行 Log

1. 方法一，启动时添加选项 `-L`（Turn on output logging.） 会在当前目录下生成 `screenlog.0` 文件。这种情况适合在不同目录下保留Log。
2. 方法二，启动时不加 `-L` 时，使用 `ctrl a + H` 可以开始和停止记录日志。
3. 方法三，如果想要在同一目录区分保留 Log，则需要修改 `/etc/screenrc`，在最后一行添加 `logfile /tmp/screenlog_%t.log`,其中 `%t` 是指 window 窗口的名称，对应 screen 的 `-t` 参数。


__常用方法：__

```sh
# 在想要保存 command log 的文件夹内执行下面的语句，以达到后台启动的目的。
screen -L -dmS name command
```



## 参考文献

+ [GNU Screen](http://www.gnu.org/software/screen/)
+ [linux screen工具](https://www.cnblogs.com/lpfuture/p/5786843.html)
+ [用screen 在后台运行程序](https://www.jianshu.com/p/b24f597c0561)
+ [Linux Screen技巧：记录屏幕日志](https://blog.csdn.net/lovemysea/article/details/78344114)