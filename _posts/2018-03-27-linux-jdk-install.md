---
layout: post
title:  Linux中安装JDK
categories: Java,Linux
description: 在linux系统中安装部署JDk
keywords: linux,JDK
---

linux中jdk安装部署

##  下载JDK
 
 本人用的是1.8的JDK,并且下载的是 jdk-8u162-linux-x64.tar.gz 的压缩包
 下载地址 http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

 ![INNER JOIN](https://chinakarl.github.io/images/posts/java/jdk-download.png)

## 将下载的包拷贝到linux系统

 安装rz的包
 yum install -y  lrzsz
 安装完成后输入 rz，之后会跳出窗口，选择想要上传的文件，等待上传完毕

 如果是下载的话用 sz命令

 安装好后在usr目录下新建目录为javaweb的文件
 mkdir javaweb

 然后将压缩包拷贝到当前目录并且解压

 tar -zxvf jdk-8u162-linux-x64.tar.gz

## 配置环境变量
   vim /etc/profile 打开文件

   i 进行编辑，在文件末尾添加入下配置

	export JAVA_HOME=/usr/javaweb/jdk1.8.0_131 /*这是解压的jdk文件目录*/
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
	export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
	export PATH=$PATH:${JAVA_PATH}


  
