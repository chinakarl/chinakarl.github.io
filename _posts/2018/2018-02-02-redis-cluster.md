---
layout: post
title: Redis Windows 下集群部署(Redis Cluster)
categories: 
description: 简单部署在Windows下的Redis集群
keywords: Redis,集群,Windows
---

这篇文章应该是2017年最后一篇了，之后会写下年度总结。

在前我就不对Redis进行简单介绍了，详细大家都耳熟能详了。

Redis官方不支持Windows，但是Microsoft Open Tech group在 GitHub上开发了一个Win64的版本,下载地址为： https://github.com/MSOpenTech/redis/releases  个人下载地址:https://pan.baidu.com/s/1racD9gw (切记Redis只有在3.0以后的版本才支持集群，下载时候要注意)

## 1.下载Redis

创建一个文件夹(自己定义，当前用C:\RedisDemo)，将下载的文件放到新建的目录下

启动Redis服务器。(启用的时候可以在文件夹下新增一个start.bat的文件，然后将cmd的命令写在文件中，这样子就可以实现双击启动，不用每次都用命令启动了)，启动之后Redis是默认监听6379的端口。

## 2.安装Ruby并配置环境

Windows安装RubyInstaller，下载地址： http://railsinstaller.org/en 个人下载地址:https://pan.baidu.com/s/1kWhrpNx (里要注意下，下载的Ruby必须要是2.2.2版本以上的，不然installer Redis的时候是安装不了的，因为Ruby只有在2.2.2版本后才支持Redis)

安装完成后执行命令 ruby -v查看当前ruby版本，gem install redis 安装redis

如果在安装过程中 SSL Connect error ，是因为ruby 没有包含 SSL 证书，所以 https 的链接被服务器拒绝。

解决方法很简单，首先在这里下载证书 http://curl.haxx.se/ca/cacert.pem, 然后再环境变量里设置 SSL_CERT_FILE 这个环境变量，并指向 cacert.pem 文件。

**如图:**

![INNER JOIN](https://chinakarl.github.io/images/posts/windows/environmentvariable.png)

## 3.搭建Redis集群

### 3.1 建文件夹

  在RedisDemo文件夹下，新建 7001，7002，7003，7004，7005，7006 六个文件夹，然后将Redis中的Redis-server.exe和Redis.windows.conf拷贝到六个文件夹下。

### 3.2 更改配置

  将六个文件下的Redis.windows.conf配置文件下的节点开启并配置成对应值
  port 7001（对应文件夹的端口号）//端口号
  cluster-enabled yes            //是否开启集群
  cluster-config-file nodes.conf //此配置文件不能人工编辑，它是集群节点自动维护的文件，主要用于记录集群中有哪些节点、他们的状态以及一些持久化参数等，方便在重启时恢复这些状态。通常是在收到请求之后这个文件就会被更新。
  cluster-node-timeout 5000      //集群中的主从节点最长的失联时间
  appendonly yes                 //开启AOF模式

### 3.3 启动Redis
  
  用cmd命令进入相对于的六个文件夹下，输入命令:redis-server.exe redis.windows.conf 分别启动六个服务

### 3.4 创建集群

  启用集群需要redis-trib.rb文件，文件主要是在，启动各个node时候(没有分配solt)，并且将各个node连接起来，再给各个节点分配solt(也就是再分片--reshard)的过程起到作用

  **Redis集群过程图**

  ![INNER JOIN](https://chinakarl.github.io/images/posts/redis/redisflow.jpg)

  最后进入redis-trib.rb文件所在目录执行

  ruby redis-trib.rb create --replicas 1  127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006


