---
layout:      post
title:       "Dockerfile 常用命令"
categories:  [运维, 虚拟化, Docker]
description: "Dockerfile 常用命令"
keywords:    运维, 虚拟化, Docker
---

# Dockerfile 常用命令

Dockfile 是一种被 Docker 程序解释的脚本，Dockerfile 由一条一条的指令组成，每条指令对应 Linux 下面的一条命令。Docker 程序将这些 Dockerfile 指令翻译真正的 Linux 命令。

Dockerfile 有自己书写格式和支持的命令，Docker 程序解决这些命令间的依赖关系，类似于 Makefile。 Docker 程序将读取 Dockerfile，根据指令生成定制的 image。Dockerfile 明确的表明 image 是怎么产生的。有了 Dockerfile，当需要定制自己额外的需求时，只需在 Dockerfile 上添加或者修改指令，重新生成 image 即可，省去了敲命令的麻烦。


### FROM

功能为指定基础镜像，并且必须是第一条指令。

如果不以任何镜像为基础，那么写法为：FROM scratch。

同时意味着接下来所写的指令将作为镜像的第一层开始


```
FROM <image> [AS <name>]
```

Or

```
FROM <image>[:<tag>] [AS <name>]
```

Or

```
FROM <image>[@<digest>] [AS <name>]
```


### RUN

RUN 命令有两种模式

* `RUN <command>` (*shell* 模式, 直接执行 shell 命令, 在 Linux 上默认执行的是 `/bin/sh -c` ，在 Windows上默认执行的是 `cmd /S /C`)
* `RUN ["executable", "param1", "param2"]` (*exec* 模式)

RUN 的换行符是 `\`

__注意__：多行命令不要写多个 RUN，原因是 Dockerfile 中每一个指令都会建立一层.多少个 RUN 就构建了多少层镜像，会造成镜像的臃肿、多层，不仅仅增加了构件部署的时间，还容易出错。

RUN 是构件容器时就运行的命令以及提交运行结果。


### CMD

`CMD` 命令有三种模式:

* `CMD ["executable","param1","param2"]` (*exec* 模式, 这是默认模式)
* `CMD ["param1","param2"]` (*ENTRYPOINT 的默认 parameters*)
* `CMD command param1 param2` (*shell* 模式)

__注意__：因为参数传递后，docker 解析的是一个 JSON array, 所以参数一定要用双引号 `"`,不能是单引号 `'`。

CMD 是容器启动时执行的命令，在构件时并不运行，构件时紧紧指定了这个命令到底是个什么样子。


### LABEL

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

为 image 添加 LABEL， LABEL 可以为多个键值对，可以写成如下形式。

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

需要注意的是，最好不要写成多个 LABEL, 最好写成一行，太长的话使用 `\` 进行换行，重复的 Key 会被覆盖。


### MAINTAINER

指定作者

```
MAINTAINER <name>
```


### EXPOSE

```
EXPOSE <port> [<port>/<protocol>...]
```

该命令通知 Docker 在运行的时候侦听指定的网络端口。可以指定侦听的是 TCP 还是 UDP，如果未指定协议，则默认为 TCP。

如果同时侦听 TCP 和 UDP 的 80 端口:

```
EXPOSE 80/tcp
EXPOSE 80/udp
```

需要注意的是，此方式暴露出的端口并不会直接让主机监听，注意如果想监听，需要在 Docker 启动时加上 `-p` 参数。

```
docker run -p 80:80/tcp -p 80:80/udp ...
```


### ENV

```
ENV <key> <value>
ENV <key>=<value> ...
```

此命令的功能为设置环境变量，差别在于第一种一行只能设置一个，第二种可以再一行设置多个。

For example:

```
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```

and

```
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```


### ADD

ADD 命令有两种模式:

* `ADD [--chown=<user>:<group>] <src>... <dest>`
* `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` (如果路径中带空格需要使用此模式)

`<src>` 可以包含通配符，匹配的规则使用的是 Go 的 [filepath.Match](http://golang.org/pkg/path/filepath#Match) 。 

For example:

```
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

`<dest>` 可以是一个绝对路径，也可以是相对于 `WORKDIR` 的路径。

```
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

如果包含特殊字符，则需要使用 Golang 规则进行转换，防止将特殊字符当成通配符处理. 例如要拷贝文件名为 `arr[0].txt` 的文件。

```
ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/
```

如果将 `<src>` 写成 URL，则类似于 `wget` 命令。

```
ADD http://example.com/foobar /
```

尽量不要把 `<scr>` 写成一个文件夹，如果 `<src>` 是一个文件夹，会复制整个目录的内容，包括文件系统元数据。


### COPY

COPY 包含两种模式:

* `COPY [--chown=<user>:<group>] <src>... <dest>`
* `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` (如果路径中带空格需要使用此模式)

COPY 是复制命令，与 ADD 的差别在于，`<src>` 只能是本地文件，其他用法一致


### ENTRYPOINT

ENTRYPOINT 有两种模式:

* `ENTRYPOINT ["executable", "param1", "param2"]` (*exec* form, preferred)
* `ENTRYPOINT command param1 param2` (*shell* form)

功能是设定启动时执行的默认命令。

与CMD比较说明：

1. 相同点：
    * 只能写一条，如果写了多条，那么只有最后一条生效
    * 容器启动时才运行，运行时机相同
2. 不同点：
    * ENTRYPOINT不会被运行的command覆盖，而CMD则会被覆盖
    * 如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD指令不是一个完整的可执行命令，那么CMD指定的内容将会作为ENTRYPOINT的参数
        如下：
        
        ```    
        FROM ubuntu
        ENTRYPOINT ["top", "-b"]
        CMD ["-c"]
        ```

    * 如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD是一个完整的指令，那么它们两个会互相覆盖，谁在最后谁生效
        如下：
        
        ```
        FROM ubuntu
        ENTRYPOINT ["top", "-b"]
        CMD ls -al
        ```

        那么将执行ls -al ,top -b不会执行。

Docker官方使用一张表格来展示了ENTRYPOINT 和CMD不同组合的执行情况

（下方表格来自docker官网）

|   | No ENTRYPOINT | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”] |
| :-- | :-- | :-- | :-- |
| **No CMD** | *error, not allowed* | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry |
| **CMD [“exec_cmd”, “p1_cmd”]** | exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd |
| **CMD [“p1_cmd”, “p2_cmd”]** | p1_cmd p2_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd |
| **CMD exec_cmd p1_cmd** | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |


### VOLUME

```
VOLUME ["/data"]
```

该命令的值是 JSON 数组，可以有以下几种用法。

```
VOLUME ["/var/log/"]
VOLUME /var/log
VOLUME /var/log /var/db
```

一般的使用场景为需要持久化存储数据时。


### USER

```
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

设置启动容器的用户，需要注意的是，如果设置了容器以 Daemo 用户去运行，那么 RUN， CMD 和 ENTRYPOINT 都会以这个用户去运行。


### WORKDIR

```
WORKDIR /path/to/workdir
```

设置工作目录，对 RUN，CMD，ENTRYPOINT，COPY，ADD 生效。如果不存在则会创建，也可以设置多次。例如：

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

在这段 Dockerfile 中， `pwd` 输出的内容为 `/a/b/c`。

如果要使用 ENV 设置的环境变量，必须在 Dockerfile 中显示的设置环境变量。例如：

```
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

这段 Dockerfile 中， `pwd` 输出的内容为 `/path/$DIRNAME`。


### ARG

```
ARG <name>[=<default value>]
```

设置变量命令，ARG 命令定义了一个变量，在 `docker build` 创建镜像的时候，使用 `--build-arg <varname>=<value>` 来指定参数。如果用户在 build 镜像时指定了一个参数没有定义在 Dockerfile 中，那么将有一个 Warning。

``` log
[Warning] One or more build-args [foo] were not consumed.
```


### ONBUILD

```
ONBUILD [INSTRUCTION]
```

此命令只有在另一个 image 以此 image 作为基础镜像，在 Build 时执行。

例如：

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

此命令会在另外一个 image 以此 image 为基础镜像 build 时执行。


### STOPSIGNAL

```
STOPSIGNAL signal
```

此命令设置镜像的退出信号。

