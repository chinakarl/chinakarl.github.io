---
layout: post
title:  maven项目在eclipse中运行
categories: Java
description: 实现maven建的web项目在eclipse中运行
keywords: maven,java,eclipse
---

maven项目在eclipse中运行

##  下载tomcat

    tomcat 官网 https://tomcat.apache.org/ 我下载的是9.0.6版本

	下载后解压，解压有个bin文件夹，startup.bat双击执行就可以(.sh结尾的是linux下的)

##  tomcat配置

    进入/conf/tomcat_users.xml配置文件，添加

	<role rolename="admin-gui"/>
    <role rolename="admin-script"/>
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="manager-jmx"/>
    <role rolename="manager-status"/>

    <user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-script,admin-gui"/>  

	以下是配置说明

	说明出自  https://blog.csdn.net/xiaohe6688/article/details/72627045

    如上所示，我们只需要在tomcat-users节点中配置相应的role(角色/权限)和user(用户)即可。一个user节点表示单个用户，属性username和password分别表示登录的用户名和密码，属性roles表示该用户所具备的权限。

	user节点的roles属性值与role节点的rolename属性值相对应，表示当前用户具备该role节点所表示的角色权限。当然，一个用户可以具备多种权限，因此属性roles的值可以是多个rolename，多个rolename之间以英文逗号隔开即可。

	稍加思考，我们就应该猜测到，rolename的属性值并不是随意的内容，否则Tomcat怎么能够知道我们随便定义的rolename表示什么样的权限呢。实际上，Tomcat已经为我们定义了4种不同的角色——也就是4个rolename，我们只需要使用Tomcat为我们定义的这几种角色就足够满足我们的工作需要了。

	以下是Tomcat Manager 4种角色的大致介绍(下面URL中的*为通配符)：

	manager-gui
	允许访问html接口(即URL路径为/manager/html/*)
	manager-script
	允许访问纯文本接口(即URL路径为/manager/text/*)
	manager-jmx
	允许访问JMX代理接口(即URL路径为/manager/jmxproxy/*)
	manager-status
	允许访问Tomcat只读状态页面(即URL路径为/manager/status/*)
	从Tomcat Manager内部配置文件中可以得知，manager-gui、manager-script、manager-jmx均具备manager-status的权限，也就是说，manager-gui、manager-script、manager-jmx三种角色权限无需再额外添加manager-status权限，即可直接访问路径/manager/status/*

##  maven中配置
    
	<server>  
        <id>tomcat</id>  
        <username>admin</username>  这是tomcat中配置的用户名和密码
        <password>admin</password>  
    </server>

##  eclipse中配置tomcat

   在新建的web项目的pom.xml文件的build节点中添加

   1.重新部署
       <plugins>  
        <plugin>  
            <groupId>org.codehaus.mojo</groupId>  
            <artifactId>tomcat-maven-plugin</artifactId>  
            <configuration>  
                <warFile>target/demo-web.war</warFile>  这是运行后war文件地址
                <server>tomcat</server>                 这是服务器名称要和maven中配置的server的id一致
                <url>http://localhost:8080/manager/text</url>  
                <path>/demo-web</path>                  这是项目名称
            </configuration>  
        </plugin>  
    </plugins>  

	在global中填入 package tomcat:redeploy (这是将war包重新部署到tomcat的webapps下)

  2.直接maven部署

    在global中填入 clean install
    

  
