---
layout:      post
title:       "关于SUMO"
categories:  [软件开发]
description: "About Simulation of Urban Mobility"
keywords:    SUMO, 软件开发
---

简介
---------
SUMO全称Simulation of Urban Mobility，是一个开源、微观、多模态交通仿真模拟软件。<br />
它允许模拟一个给定的交通需求，其中包括一个车辆移动通过一个给定的道路网络。<br />
仿真允许解决一个大的交通管理主题。<br />
它是纯粹的微观：每个车辆的建模明确，有一个自己的路线，并单独通过网络。<br />
在默认情况下模拟是确定性的，但引入随机性有不同的选择。

特性
---------
1. 包括准备和进行交通仿真应用的全部应用（网络和路线导入，DUA，模拟）
2. 模拟
 - 空间连续和时间离散车辆运动
 - 不同车辆类型
 - 车道变换的多车道街道
 - 不同的通行权规则，交通灯
 - 一个快速的OpenGL图形用户界面
 - <font color="red">管理几个10000边（街道）的网络</font>
 - <font color="red">执行速度快（高达100000的车辆更新/在1GHz的机器）</font>
 - 与其他应用程序在运行时的互操作性
 - 网络范围，基于边缘，基于车辆和基于检测器的输出
 - 支持基于人的跨模态的旅行
3. 网络输入
 - 支持VISUM, Vissim,Shapefiles, OSM, RoboCup, MATsim, OpenDRIVE, and XML-Descriptions；
 - 丢失的值是通过启发式确定；
4. 路由
 - 微观路线-每辆车都有一个自己的路线；
 - 不同用户动态分配算法；
5. 可移植性高
 - 标准C++库和portable库被应用；
 - Windows 和Linux分布的安装包都存在；
6. 通过只使用XML数据的高互操作性
7. 开源（GPL）

被使用的地方
---------
自2001以来，SUMO已经被用在一些国家和国际研究项目背景。应用包括：
+ 交通灯的评价
+ 路由选择和再路由
+ 交通监控方法评价
+ 车辆通信仿真
+ 交通量预测

包含的应用
---------

|Application name|Short description|
|:---|:---|
|SUMO|没有可视化的微观仿真；命令行应用|
|SUMO-GUI|用图形用户界面的微观模拟|
|NETCONVERT|网络导入和导出；读取不同格式的道路网络，并将它们转换为SUMO格式|
|NETEDIT|一个图形化的网络编辑器。|
|NETGENERATE|生成的SUMO仿真基本网络|
|DUAROUTER|输入不同类型的需求描述，计算最快的路由通过网络。执行DUA|
|JTRROUTER|使用结转百分比（ junction turning percentages）计算路由|
|DFROUTER|从感应环测量（ induction loop measurements）计算路线|
|OD2TRIPS|分解OD矩阵（O/D-matrices ）为单个车辆轨迹（single vehicle trips）|
|POLYCONVERT|从不同的格式导入兴趣和多边形的点，转换他们到一种可被SUMO-GUI可视化的描述|
|ACTIVITYGEN|根据模拟人口的流动意愿生成需求|
|MESO|使用介观队列模型（mesoscopic queue-model）的模拟，比纯粹的微观模拟执行模拟高达100倍的速度。|
|MESO-GUI|具有图形用户界面的介观模拟。|
|Additional Tools|有一些任务，编写一个大的应用程序是没有必要的。这些工具可以覆盖不同的问题的几种解决方案。|

一些组织对于SUMO安装包进行了扩展并且提交了他们的代码，如果没有在release版本中包含，他们的贡献通常没有被频繁测试，或者过时了。包含以下程序，但是请注意他们并不是全部长久更新维持的：

|Application name|Short description|
|:---|:---|
|TraCI4J|一种java接口用于连接和扩展信息通过TraCI ，作者Enrico Gueli|
|TraCI4Matlab|一种matlab接口用于连接和扩展信息通过TraCI ，作者Andres Acosta|
|TraaS|一种SOAP（webservice）接口用于连接和扩展信息通过TraCI ，作者Mario Krumnow|
|Contributed/SUMO Traffic Modeler|过时了|
