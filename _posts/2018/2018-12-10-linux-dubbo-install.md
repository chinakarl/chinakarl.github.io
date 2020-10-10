---
layout: post
title:  dubbo搭建
categories: Java,Linux
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
   
   查看 sh zkCli.sh -server 192.168.192.133:2181  
   
   ls /

## dubbo-admin 部署到tomcat中
  
   下载dubbo-admin 项目(地址：https://github.com/dangdangdotcom/dubbox.git)
   
   通过maven命令编译生成war，将生成好的war上传到刚才部署的tomcat的webapp下，启动项目
   
   注意： 要将 dubbo/WEB-INF/dubbo.properties 下的配置的address 修改为 zk的ip：port
     
   dubbo.registry.address=zookeeper://192.168.192.133:2181
    
   浏览:
    
   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/dubbo-admin.png)


## dubbo spring-boot下应用

  在项目resources 下新建 dubbo文件 并且添上 dubbo,dubbo-consumer,dubbo-provider 三个 xml
  
  dubbo.xml:
  
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
      
          <!-- 提供方应用信息，用于计算依赖关系 -->
          <dubbo:application name="${dubbo.application.name}" environment="${dubbo.application.environment}"/>
          <!-- 使用multicast广播注册中心暴露服务地址 -->
          <dubbo:registry protocol="zookeeper" address="${dubbo.registry.address}" file="${dubbo.registry.file}"/>
      
          <!-- 用dubbo协议在20880端口暴露服务 -->
          <dubbo:protocol name="dubbo" host="${dubbo.protocol.host:}" port="${dubbo.protocol.port}" threadpool="${dubbo.protocol.threadpool}"  threads="${dubbo.protocol.threads}"/>
      
          <!-- 提供方的缺省值，当ProtocolConfig和ServiceConfig某属性没有配置时，采用此缺省值，可选。-->
          <dubbo:provider connections="${dubbo.provider.connections}" timeout="${dubbo.provider.timeout}" retries="${dubbo.provider.retries}" group="${dubbo.provider.group}" version="${dubbo.provider.version}" />
      
          <!-- 消费方缺省配置，当ReferenceConfig某属性没有配置时，采用此缺省值，可选。-->
          <dubbo:consumer check="${dubbo.consumer.check}" group="${dubbo.consumer.group}" version="${dubbo.provider.version}"/>
      
          <!-- 监控中心配置，用于配置连接监控中心相关信息，可选。-->
          <!--<dubbo:monitor protocol="registry"/>-->
      
      </beans>
  
  dubbo-consumer.xml:
  
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
      
          <!-- 引用远程服务配置, 可以和本地bean一样使用 -->
          <!--<dubbo:reference id="commonParamApi" interface="com.ai.market.system.api.ICommonParamApi" />-->
      
      </beans>
  
dubbo-provider.xml:
  
      <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo
              http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
      
          <!-- 声明需要暴露的服务接口 -->
          <dubbo:service ref="loginService" interface="com.xinke.management.service.LoginService"/>
      
      </beans>
  
  再新增 application-dubbo.yml 文件
  
      dubbo:
        application:
          name: haixiangtest
          environment: develop
        protocol:
          port: 20880
          threadpool: fixed
          threads: 200
        registry:
          address: 192.168.192.133:2181
          file: dubbo-registry-cache/general-management{replicas.id:}.cache
        provider:
          timeout: 30000
          connections: 5
          group: prodGroup
          version: 1.0.0
          retries: 0
        consumer:
          check: false
          group: prodGroup
      
      ---
      
      spring:
        profiles: test
      
      dubbo:
        registry:
          address: 192.168.192.128:2181
      
      ---
      
      spring:
        profiles: prod
      
      dubbo:
        application:
          environment: product
        registry:
          address: 192.168.231.129:2181
      
      ---
      
      spring:
        profiles: local
      
      dubbo:
        registry:
          address: 127.0.0.1:2181 
  
  在application.yml文件中 include
  
      spring:
        profiles:
          include: datasource,dubbo
      

 在springbootapplication中添加
 
 @ImportResource({"classpath:/dubbo/*.xml"})
 
 注解，导入 dubbo下的所有xml