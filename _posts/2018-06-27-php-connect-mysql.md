---
layout:      post
title:       "php 连接 mysql 测试脚本"
categories:  [开发, php]
description: "php 连接 mysql 测试脚本"
keywords:    开发, php
---


``` php
<?php 
    $mysqli = new mysqli("localhost", "root", "123456"); 
    if(!$mysqli)  { 
        echo "database error"; 
    }else{ 
        echo "php env successful"; 
    } 
    $mysqli->close(); 
?> 
```


