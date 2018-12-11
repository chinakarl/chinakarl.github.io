---
layout: post
title:  dubbo搭建
categories: Java，Linux
description: linux下dubbo的搭建
keywords: linux,dubbo
---

记录下linux下手动搭建dubbo并且调用的过程

## 环境

   1. vmware,centos7
   2. jdk 1.8 , tomcat 9.0.6 ,
   3. zookeeper 3.4.12
   4. dubbo-admin 项目


## 部署

   安装完jdk后，将tomcat和zookeepr上传到服务器，配置zookeeepr并且启动

## dubbo-admin 部署到tomcat中
  
   下载dubbo-admin 项目(地址：https://github.com/dangdangdotcom/dubbox.git)
   
   通过maven命令编译生成war，将生成好的war上传到刚才部署的tomcat的webapp下，启动项目
    
   浏览:
    
   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/dubbo-admin.png)

  
