---
layout: post
title:  rabbitmq搭建
categories: mq，Linux
description: linux下rabbitmq的搭建
keywords: linux,rabbitmq
---

记录下linux系统下linux的搭建

## 环境

   1. vmware,centos7
   3. Erlang 20.2
   4. rabbitmq 3.6.15


## 安装Erlang

   Erlang就不多介绍了

   1. 在安装Erlang前先安装依赖文件 yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto

   2. 安装Erlang 官网 http://www.erlang.org/downloads

      用wget下载 wget -c http://erlang.org/download/otp_src_20.2.tar.gz

      下载后解压

   3. 编译安装( 我这里指定编译安装后放在/usr/local/erlang目录里面，这个你们可以改成其他的 )：

　　　　[root@localhost otp_src_20.2]# ./configure --prefix=/usr/mq/rabbitmq/erlang

　　　　[root@localhost otp_src_20.2]# make && make install

   4. 测试安装是否成功：

　　　　[root@localhost erlang]# cd /usr/local/erlang/bin/ 

　　　　[root@localhost bin]# ./erl

  ![INNER JOIN](https://chinakarl.github.io/images/posts/mq/erlang-success.png)
     

       出现以上就表示安装成功

   5. 配置环境变量

　　　　[root@localhost ~]# vim /etc/profile

　　　　在末尾加入这么一行即可：export PATH=$PATH:/usr/local/erlang/bin　

　　　　更新配置文件：[root@localhost ~]# source /etc/profile

　　　　更新之后在任意地方输入erl能进入命令行， 那么就说明配置成功了。

## rabbitmq安装
  
   1. 下载rabbitmq 官网(地址：http://www.rabbitmq.com/releases/rabbitmq-server/)

      用wget下载 wget -c http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-generic-unix-3.6.15.tar.xz
   
      解压：

　　　　　　[root@localhost local]# xz -d rabbitmq-server-generic-unix-3.6.15.tar.xz 

　　　　　　[root@localhost local]# tar -xvf rabbitmq-server-generic-unix-3.6.15.tar
    
   2. 配置rabbitmq的环境变量

　　　　[root@local local]# vim /etc/profile

　　　　在末尾加入以下配置：export PATH=$PATH:/usr/mq/rabbitmq/rabbitmq_server-3.6.15/sbin

　　　　更新配置文件：[root@local local]# source /etc/profile

  3. 开启rabbitmq远程访问

　　　　添加用户:rabbitmqctl add_user zhaihx zhaihx123　　//zhaihx是用户名， zhaihx123是用户密码

　　　　添加权限:rabbitmqctl set_permissions -p "/" zhaihx ".*" ".*" ".*"

　　　　修改用户角色:rabbitmqctl set_user_tags zhaihx administrator

　　　　然后就可以远程访问了，然后可直接配置用户权限等信息

  4. 启动rabbitmq
 
       [root@local local]# cd /usr/mq/rabbitmq/rabbitmq_server-3.6.15/sbin

       [root@localhost sbin]# ./rabbitmq-server 

       启动成功后就可以用ip地址和端口号访问了

       ![INNER JOIN](https://chinakarl.github.io/images/posts/mq/rabbitmq-plugins.png)

