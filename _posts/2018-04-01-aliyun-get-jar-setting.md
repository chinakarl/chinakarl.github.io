---
layout: post
title:  获取阿里云jar包配置
categories: Java
description: 获取阿里云jar包的简单配置
keywords: 阿里云,jar,aliyun
---

获取阿里云jar包的简单配置

## maven中配置

  在maven中的settings.xml

  <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>


## pom文件中配置dependency节点

   进入 http://mvnrepository.com/ 站点，搜索相关的jar名称，点击进入，将其拷贝到pom的<dependencys>节点下

   因为我用到了springmvc所以我引用了这几个

    <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>4.3.14.RELEASE</version>
	</dependency>

	<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>4.3.14.RELEASE</version>
	</dependency>

	<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>4.3.14.RELEASE</version>
	</dependency>

	<dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>

  

  
