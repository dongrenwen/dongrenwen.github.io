---
layout:      post
title:       "SUMO的简单操作"
categories:  [软件开发]
description: “SUMO's simple operation”
keywords:    SUMO, 软件开发
---

使用SUMO命令行应用
---------

许多SUMO包里的程序应用是命令行工具。只有SUMO-GUI不是。关于命令行，可以阅读[基本计算机技巧](http://www.sumo.dlr.de/userdoc/Basics/Basic_Computer_Skills.html#Running_Programs_from_the_Command_Line)页面。<br />

以下内容介绍了SUMO匹配程序的一些特点。

1. 从命令行使用SUMO的应用
<br />SUMO应用程序是普通的应用程序。在```SUMO Command Line```中输入应用名称就能够运行它们。
<br />比如NETGENRATE被调用通过
``` shell
netgenerate.exe
```
这样就启动了程序。因为并没有给它任何的参数，所以程序输出的时它自己的信息：
```
SUMO netgenerate Version 0.32.0
Build features: x64 Release PROJ GDAL GUI
Copyright (C) 2001-2017 German Aerospace Center (DLR) and others; http://sumo.dlr.de
License EPLv2: Eclipse Public License Version 2 <https://eclipse.org/legal/epl-v20.html>
Use --help to get the list of options.
```

2. 选项
<br />每个应用程序都有一组选项，该选项定义哪些文件应被处理或生成，或哪些定义了应用程序的行为。通常情况下，一个应用程序需要至少两个参数-一个输入文件和一个输出文件-但几乎总是更多的参数用于细粒度的控制（fine-grained control）。每个应用程序的选项都在应用程序的描述中描述。

3. 命令行的选项设置
<br />有两种选项：
 + 布尔选项。不需要一个参数，如果选项存在则设置为True（此类选项只能设置“True”或“False”）
 + 需要一个参数的选项。在命令行上设置一个参数，由两个部分组成：选项名称和选项的值。
 <br />例如，如果想要模拟加载特定的道路网络，“mynet.net.XML”，以下可以写：
 ```
 --net mynet.net.xml
 ```
 `--`在最前面，指示选项的常明诚紧跟其后。空格后是选项的值。也可以用`=`代替空白格。
 ```
 --net=mynet.net.xml
 ```
 很多时候可以缩写。比如`—net`可以缩写为`-n`，如：
 ```
 -n mynet.net.xml
 ```
 这个例子和前面两个例子一样效果（注意缩写后变成一个`-`）。并且注意并不是所有的缩写具有相同的意义。

4. 选项值的类型
<br />SUMO程序知道哪一种类型的值是他们所期待的。比如，NETGENERATE允许设置默认的车道数，当然必须是整数。如果给它一个string类型或者其他的类型数据，程序就会报错。
<br />一个比较特别的类型就是列表（lists）。比如，当需要给出多个文件，文件之间必须用逗号隔开。这个方式同样应用于其他的数值类型，比如整数或者浮点数。

配置文件
---------
因为选项列表可能会很长，配置文件被引入。可以设置一个配置文件，其中包含运行应用程序的所有参数。
<br />配置文件是一个XML文件，有一个名为“configuration”的根元素。选项被写为元素名称，在属性value中存储值。命令行语句`—net-file test.net.xml`可以改写为`<net-filevalue="test.net.xml">`。对于布尔值，真值对应的是：`true`、`on`、`yes`、`1`；假值对应的是：`false`、`off`、`no`、`0`；
<br />举例说明，配置文件(`test.sumocfg`)样式如下：
``` xml
<configuration>
    <input>
        <net-file value="test.net.xml"/>
        <route-files value="test.rou.xml"/>
        <additional-files value="test.add.xml"/>
    </input>
</configuration>
```
“input”部分只是文件目的，并没有功能意义。
<br />
``` xml
<configuration>
    <n v="test.net.xml"/>
    <r v="test.rou.xml"/>
    <a v="test.add.xml"/>
</configuration>
```
由SUMO程序执行调用，应该为：
```
sumo.exe -c test.sumocfg
```
使用选项`—configuration-file`或者`-c`直接运行配置文件，在命令行中也可以直接省去`-c`：
```
sumo.exe test.sumocfg
```

1. 配置文件的命名约定
<br />根据目标应用程序，配置文件有不同的扩展。强烈建议遵守命名公约。针对使用[SUMO-GUI](http://www.sumo.dlr.de/userdoc/SUMO-GUI.html)模拟配置的更是必须遵守，[SUMO-GUI](http://www.sumo.dlr.de/userdoc/SUMO-GUI.html)只能够读取命名为`*.sumocfg`模拟配置文件。
<br />在页面[used file extensions](http://www.sumo.dlr.de/userdoc/Other/File_Extensions.html)中可以找到所有配置扩展的约定
2. 配置文件vs命令行参数
除了配置文件，更多的命令行参数是通过命令行赋值。如果一个参数被设置在指定的配置文件中，同事也在命令行上设置，命令行上的值可以被使用。如果想禁用配置文件中启用的布尔选项，需要显式地在命令行上给出False的值，就像`--verbose false`。
3. 生成配置文件、模板和模式（schemata）
<br />SUMO程序允许生成配置文件模板，可以保存一个空的配置文件（配置文件模板），通过命令`--save-template <FILE>`实现。在这种情况下，配置只包含填充了它们的默认值的参数。
<br />想要保存包含当前设置的配置文件，可以强制使用命令`--save-configuration <FILE>`
<br />最后但并非最不重要的是，可以生成一个XML模式（使用选项`--save-schema <FILE>`）来验证配置文件。 对于SUMO应用程序，此架构应与`http://sumo-sim.org/xsd/sumoConfiguration.xsd`（分别针对其他可执行文件）中的相同。<font color="red">请注意，模式（schema）比SUMO选项解析器更严格，因为它不仅验证了上述详细版本。</font>
<br />在任何情况下，如果要进一步的参数的信息，也可以通过选项`--save-commented`。然后，对每个参数产生一些进一步的信息。

常用选项
---------
SUMO程序共享几个选项。在下面会详细讲解。
1. 报告选项（Reporting Options）

|Option|Description|
|:---|:---|
|`-v <BOOL>`<br />`--verbose <BOOL>`|输出详细信息；默认值为：false|
|`--print-options <BOOL>`|在处理前打印选项值；默认值为：false|
| `-? <BOOL>`<br />`--help <BOOL>`|打印帮助信息到当前屏幕；默认值为：false|
|`-V <BOOL>`<br />`--version <BOOL>`|打印版本信息；默认值为：false|
|`-X <STRING>`<br />`--xml-validation <STRING>`|设置XML输入的模式验证方案（"never"，"auto"或"always"）; 默认值为："auto"|
|`--xml-validation.net <STRING>`|设置SUMO网络输入的模式验证方案（"never"，"auto"或"always"）; 默认值为："auto"|
|`-W <BOOL>`<br />`--no-warnings <BOOL>`|禁用警告输出；默认值为：false|
|`-l <FILE>`<br />`--log <FILE>`|将所有Log信息写入文件(意味着冗长)|
|`--message-log <FILE>`|将所有非错误Log信息写入到文件(意味着冗长)|
|`--error-log <FILE>`|将Log中的所有警告和错误信息写入到文件|

`--log`和`--message-log`只能将正确的Log信息输出到文件，且不会输出到终端屏幕上，除非同时使用`--verbose`参数，需要注意的时，这是错误的Log信息只是输出到了终端屏幕上，而不会被输出到文件之中，除非指定错误Log输出的Log文件。
<br /> [XML validation](http://www.sumo.dlr.de/userdoc/XMLValidation.html)选项会在XML分析程序中开启[XML schema processing](https://xerces.apache.org/xerces-c/schema-3.html)，这会对输入进行基本的验证，初学者来说，特别强烈推荐使用此选项，因为它会很容易的就发现输入中的拼写错误，否则可能会被忽略。

2. 随机数选项（Random Number Options）
<br />这些选项配置用来确定随机数生成器的种子。相同的种子可以产生相同序列的随机数字。
<br />默认情况下，种子是一个硬编码的固定值。所以，只要所有配置设置保持不变，重复模拟运行的输出将是相同的。要更改此设置，请使用以下选项之一。

|Option|Description|
|:---|:---|
|`--seed <INT>`|为随机数发生器设置一个特定的种子。 通过使用不同的值，可以得到不同的随机数，但仍可重复的模拟运行。|
|`--random`|让SUMO选择一颗随机数种子。如果`/dev/urandom`可用，则基于此导出种子，如果不可用，则根据当前的系统时间导出种子。此选项的优先级高于`--seed <INT>`|

生成和读取文件
---------
几乎每个来自SUMO软件包的工具读取或生成的文件都是用XML编写的。在开始使用SUMO之前，应该熟悉[XML](https://en.wikipedia.org/wiki/XML)。
<br />SUMO允许你导入不同来源的文件不仅仅是SUMO本地文件（road network descriptions, route and/or demand descriptions,infrastructure descriptions, etc）。
<br />对于SUMO使用的某些文件类型，有xsd (XML Schema Definition)。关于文件类型的问题请阅读页面（[file extensions](http://www.sumo.dlr.de/userdoc/Other/File_Extensions.html)）。

写入文件
---------
有几个选项需要一个文件名作为参数写入。 当在命令行给出时，给定的路径被假定为相对于当前工作目录。 当在配置文件中给出时，文件路径被认为是相对于配置文件的路径。 绝对路径当然也是允许的。
<br />通常，具有相同名称的现有文件将被覆盖而不会有警告。 该目录必须存在，输出文件才会被写入。
<br />
<br />除了写入文件，还有一些特殊符号可以写入：
+ 写入空设备（不输出）：使用`NUL`或`/dev/null`作为文件名（两个符号均不依赖于工作平台）
+ 写入套接字（socket）：使用`<HOST>:<PORT>`作为文件名
+ 写入stdout（在命令行上打印）：使用`stdout`或` - `作为文件名
+ 写入stderr：使用`stderr`作为文件名
+ 文件名中的特殊字符串`TIME`将被替换为应用程序的开始时间

<font color="red">目前无法从sockets或stdin读取输入。</font>
<br />作为修改输出文件名的简单方法，提供了选项`--output-prefix <STRING>`。给定的字符串将被添加到应用程序编写的所有文件中。
<br /><font color="red">注意：多样数据源允许有相同的输出文件。</font>

从命令行使用Python工具
---------
很多SUMO工具类程序（在`<SUMO_HOME>/tools`文件夹中）都是用python编程语言编写的。要使用它们，必须在您的计算机上安装python 2.7。然后你需要确保设置了环境变量`SUMO_HOME`。最简单的方法是使用`start-command-line.bat`打开命令行窗口。
<br />此外需要确认计算机在哪里可以找到Python工具，最简单(但有点麻烦)的方法就是使用完整路径运行该工具。或者可以将工具所在的目录添加到`PATH`中（设置环境变量）。
