---
layout:      post
title:       "Bash 打印 Log 函数"
categories:  [运维, Linux, shell]
description: "自己写的 Bash 打印 Log 的函数，会逐步完善"
keywords:    运维, Linux, shell
---

```sh
#!/bin/bash
#######################################
#
### Print Log
#
### 设置函数内变量 LogFile 为指定的 Log 文件
#
### 参数1: 打印Log的类型
### 参数2: 打印Log的内容
#
#######################################
function PrintLog() {
    # 指定 Log 文件
    local LogFile="/tmp/logFile.$$.log"
    # 如果调用函数时参数数量不正确报错
    if [[ "$#" != 2 ]]
    then
        echo "$(date "+%Y-%m-%d %H:%M:%S") [ERROR] Incorrect number of parameters." | tee -a $LogFile
        return 1
    fi
    # 打印
    echo "$(date "+%Y-%m-%d %H:%M:%S") [$1] $2" | tee -a $LogFile
    return 0
}
#######################################
#
### Print Error Log
#
### 参数1: 打印Log的内容
#
### DEMO: 
##         PrintError "This is a test error."
#
#######################################
function PrintError() {
	# 如果调用函数时参数数量不正确报错
    if [[ "$#" != 2 ]]
    then
        PrintLog "ERROR" "Incorrect number of parameters."
        return 1
    fi
	PrintLog "ERROR" "$1"
	return $?
}
#######################################
#
### Print Info Log
#
### 参数1: 打印Log的内容
#
### DEMO: 
##         PrintInfo "This is a test info."
#
#######################################
function PrintInfo() {
	# 如果调用函数时参数数量不正确报错
    if [[ "$#" != 2 ]]
    then
        PrintLog "ERROR" "Incorrect number of parameters."
        return 1
    fi
	PrintLog "INFO" "$1"
	return $?
}
```

