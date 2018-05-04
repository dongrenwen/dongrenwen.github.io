---
layout:      post
title:       "MacOS允许任何来源应用"
categories:  [MacOS, 操作技巧]
description: "sudo spctl --master-disable"
keywords:    MacOS, 操作技巧
---

在MacOS中，在网络上下载的第三方软件，安装的时候可能会出现文件已损坏的提示，导致软件无法使用。

**解决办法**

```
系统偏好设置 -> 安全性与隐私 -> 通用 -> 选择"任何来源"
```

**如果没有 `任何来源` 选项**

+ 显示`任何来源`选项
  1. 重新启动计算机，开机时按住 `Command + R` 进入恢复模式
  2. 在恢复模式中打开命令行工具
  3. 执行命令 `sudo spctl --master-disable`
  4. 不显示任何来源执行命令 `sudo spctl --master-enable`

