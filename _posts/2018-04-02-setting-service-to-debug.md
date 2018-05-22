---
layout: post
title:  设置tomcat服务调试项目
categories: Java
description: 简单配置tomcat服务，在eclipse中调试项目
keywords: tomcat,调试,debug
---

设置tomcat服务调试项目


## 添加server

 新建项目后，右击 new -> other 搜索 server ，如果是用sts打开，不要选择sts的server要选择当前你部署的服务器的server

 我部署的是tomcat9.0,所以选择了9.0，选择之后将当前要调试的项目add到服务器上面

 注意事项

 1. 如果安装成功了，并且启动debug了，浏览localhost:8080时候一直是404，

 双击server(之前要将server下的web删除，再将server clean下，不然不好更改，选择框是灰色的)，然后将deploy path 更改为webapp(默认是wtpwebapp)


 ![INNER JOIN](https://chinakarl.github.io/images/posts/java/edit-server-deploy-path.png)

 2. 当点击Add And Remove,添加不了

 需要在pom文件中配置以下节点

       <plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
     </plugins>




  
