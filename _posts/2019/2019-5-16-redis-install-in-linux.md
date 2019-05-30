---
layout: post
title:  redis在linux下的安装和部署
categories: 学习,linux,redis
description: redis在linux环境下安装部署
keywords: 学习,redis,linux
---

 对redis在linux下的安装和部署

1. 首先下载redis安装包，我用的是4.0.6

   官网地址：https://redis.io/download
   
   本人云盘 链接: https://pan.baidu.com/s/1voczG6E9-hY9b0izEc026Q 提取码: 4wfk 

2. 上传到linux服务器
   
   将redis包传到 /user/redis 目录下
   
   运行命令 tar redis-4.0.6.tar.gz -zxvf 解压redis
   
   解压完成后进入 src目录，运行make install 安装 redis
   
3. 启动服务
  
  3.1 客户端启动 
  
   到 src 目录下 直接运行 ./redis-server 启动
   
  3.2 服务端启动
   
   在user下新建 startupsh文件，建 redis文件目录，在redis下建 bin和etc目录
   
   然后运行
   mv src/redis-conf /usr/startupsh/redis/etc
   
   mv mkreleasdhdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /usr/startupsh/redis/bin
   
   将redis文件移到指定新建目录，再编辑
   
   vim /usr/startupsh/redis/etc/redis-conf
   
   将daemonize属性改为 yes（是否在后台运行），再运行 
   
   redis-server /usr/startupsh/redis/etc/redis-conf
   

   