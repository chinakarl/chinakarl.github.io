---
layout: post
title:  elasticsearch部署
categories: elasticsearch
description: elasticsearch在linunx上搭建
keywords: es,elasticsearch
---

   本章主要是说明elasticsearch在linux下是如何搭建的

# ElasticSearch 介绍

   ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。
   
   Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。
   
   ElasticSearch用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便
   
# 部署

## 准备
   
   需要安装jdk1.8 及以上版本，下载elasticsearch安装包
   
   ElasticSearch下载地址
   
   链接: https://pan.baidu.com/s/1QzqudWpReKghMEpc4fy5XQ 提取码: muyc
   
## 安装
   
### JDK 安装
   
   JDK 安装可以自己百度也可以看下面链接
   
   https://chinakarl.github.io/2018/03/27/linux-jdk-install/
    
### ElasticSearch 安装
  
   将 elasticsearch-5.5.2.tar.gz  上传到服务器，执行解压命令
         
   > tar -zxvf elasticsearch-5.5.2 
     
 *问题一*
  
  > cd /usr/elasticsearch/elasticsearch-5.5.2/bin
    sh elasticsearch
  
  进入目录执行报错，错误如下
  
    [o.e.b.ElasticsearchUncaughtExceptionHandler] [ELK-node1] uncaught exception in thread [main]
    org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:127) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:114) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:67) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:122) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.cli.Command.main(Command.java:88) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:91) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:84) ~[elasticsearch-5.5.2.jar:5.5.2]
    Caused by: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:106) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:194) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:351) ~[elasticsearch-5.5.2.jar:5.5.2]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:123) ~[elasticsearch-5.5.2.jar:5.5.2]
        ... 6 more
  
  由于Elasticsearch可以接收用户输入的脚本并且执行,为了系统安全考虑,不允许root账号启动,所以建议给Elasticsearch单独创建一个用户来运行Elasticsearch
  
  >useradd elastic(用户名) -g elastic (所属组名)
   chown -R elastic:elastic /usr/elasticsearch/elasticsearch-5.5.2 (要更改的文件路径)
   su elastic (切换用户)
   
  执行后再启动，如果成功的话不会报错，并且会有starting和started关键词
  
  然后执行
  
  > curl http://localhost:9200
  
  ![INNER JOIN](https://chinakarl.github.io/images/posts/elasticsearch/curl.jpg)
  
  看到如上图说明启动成功
  
  
  *问题二*
  
   当配置远程连接的时候
  
  >vim elasticsearch.yml
  
  更改ip地址和端口，如下图
  
  ![INNER JOIN](https://chinakarl.github.io/images/posts/elasticsearch/remote-config.jpg)
  
  更改后重启，报如下错误
  
    max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
  
  切换到root用户
  
  >vim /etc/security/limits.conf
  
  执行上面命令，并将下面配置到开头
   
    * soft nofile 65536
    * hard nofile 131072
    * soft nproc 2048
    * hard nproc 4096
  
  然后再次重启，出现如下错误
  
  max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

  >vim /etc/sysctl.conf 

   执行上面命令，在 文件中添加 vm.max_map_count = 655360
   
  >sysctl -p
  
  执行上面命令，使文件生效
  
  >vim /usr/elasticsearch/elasticsearch-5.5.2/config/jvm.options
  
      -Xms2g  --》修改为512m
      -Xmx2g  --》修改为512m
      
  不然会报内存不够的错误
  
  配置完再次启动，成功的话我们可以浏览
  
  ![INNER JOIN](https://chinakarl.github.io/images/posts/elasticsearch/browse.jpg)
  
  如果出现如图所示说明安装成功