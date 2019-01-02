---
layout: post
title:  maven在eclipse中设置和使用
categories: Java
description: 简单的学习下maven在eclips中的使用和部署第一个web项目
keywords: maven,java,eclipse
---

maven在eclipse中设置和使用

##  下载maven
 
 本人用的是1.8的JDK,并且下载的是 jdk-8u162-linux-x64.tar.gz 的压缩包
 下载地址 http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
 所以下载了3.5.3的maven压缩包，地址 http://maven.apache.org/download.cgi
 
 ![INNER JOIN](https://chinakarl.github.io/images/posts/java/maven-download.png)

 下载的图中的两个(Binary是用的工具，而Source是maven的源码)

 

## eclipse中配置maven
   
   将下载下来的包解压并且打开conf文件下的settings.xml文件，在文件中添加节点

   <localRepository>J:/JAVA/maven/Repository</localRepository> 这是配置本地maven仓库


   打开eclipse -> window -> preferences 选择maven中的User Setting

   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/maven-eclipse-usersetting.png)

   将global setting和user setting都设置为 J:\JAVA\maven\apache-maven-3.5.3\conf\settings.xml(当前maven解压包的setting.xml)

   将local repository 配置为自己本地的mave仓库，本人的为(J:/JAVA/maven/Repository)


## eclipse中配置新建maven web项目

   new->other->maven(文件夹) ->maven project ->next -> maven-archetype-webapp(选择) -> 

   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/maven-create-project.png)

   新建完成之后会看见一个web项目在里面有一个pom.xml文件，里面的groupid就是新建的id，artifactId就是新建的项目名称






  
