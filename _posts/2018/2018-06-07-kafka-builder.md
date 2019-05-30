---
layout: post
title: kafka linux环境下的搭建
categories: kafka
description: kafka linux环境下简单搭建
keywords: kafka
---
本篇是结束kafka在linux下的搭建


## 下载kafka

官方地址 http://apache.fayea.com/kafka/

下载后上传到环境指定目录

解压(tar -zxvf kafka_2.12-2.1.1)

## 配置kafka

在配置kafka之前需要部署好zookeeper

进入到config目录

cd /usr/kafka/config

### 配置server.properties文件

vim server.properties

broker.id=0

port=9092 #端口号

host.name=192.168.192.128 #服务器IP地址，修改为自己的服务器IP

log.dirs=/usr/kafka/kafka/log #日志存放路径，上面创建的目录

zookeeper.connect=192.168.192.128:2181 #zookeeper地址和端口，单机配置部署，localhost:2181

编辑后wq!退出

### 配置zookeeper

vim zookeeper.properties #编辑修改相应的参数

dataDir=/usr/kafka/zookeeper #zookeeper数据目录

dataLogDir=/usr/kafka/zookeeper/log #zookeeper日志目录

clientPort=2181

maxClientCnxns=100

tickTime=2000

initLimit=10

syncLimit=5

wq!保存并退出

## 编写启动/停止脚本

### 启动脚本
vim kafkastart.sh #编辑，添加以下代码

//#!/bin/sh
/usr/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties & #启动zookeeper

sleep 3   #等3秒后执行

/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties & #启动kafka

:wq! #保存退出

### 停止脚本
vim kafkastopp.sh #编辑，添加以下代码

//#!/bin/sh
/usr/local/kafka/bin/zookeeper-server-stop.sh /usr/local/kafka/config/zookeeper.properties & #关闭zookeeper

sleep 3 #等3秒后执行

/usr/local/kafka/bin/kafka-server-stop.sh /usr/local/kafka/config/server.properties & #关闭kafka

:wq! #保存退出

## 启动kafka并且单机测试

./kafkastart.sh 启动kafka

查看kafka是否启动成功 输入 jps命令，如果有kafka进程则成功

创建topic

./kafka-topics.sh --create --zookeeper 192.168.192.128:2181 --replication-factor 1 --partitions 1 --topic test

启动生产者

./kafka-console-producer.sh --broker-list 192.168.192.128:9092 --topic test

启动消费者

./kafka-console-consumer.sh --bootstrap-server 192.168.192.128:9092 --topic test