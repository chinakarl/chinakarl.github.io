---
layout: post
title:  shell脚本学习
categories: shell
description: shell脚本
keywords: shell
---

shell 从零开始

##  配置java 包发版

1.新增,删除文件夹(删除文件夹下所有的文件)
  
  1.1新增
  
  nohup java  -Xms1024m -Xmx1024m -cp /opt/sysncUserApp:syncUser-1.0.jar  com.haiziwang.syncUser.Main $1 >/opt/sysncUserApp/nswork/logs/syncUser.log 2>&1 &
  
 