---
layout: post
title:  新建第一个springmvc项目
categories: Java
description: 简单的新建一个springmvc项目，在新建过程中学习简单的配置
keywords: spring,springmvc,新建项目
---

maven新建第一个springmvc项目

## 下载spring tool suite(sts)
 
 下载地址是 http://download.springsource.com/release/TOOLS/update/3.9.3.RELEASE/e4.7/springsource-tool-suite-3.9.3.RELEASE-e4.7.3-updatesite.zip
 
 下载后解压，然后打开eclipse -> help -> install new software -> add
  
 操作如下

 ![INNER JOIN](https://chinakarl.github.io/images/posts/java/install-sts.png)



## 配置相关配置项目

 ### 1.web.xml配置

    <!--
	     注意事项
		   1.初始化参数 init-param，param-value 更改成 classpath:spring-mvc.xml

		   2.servlet-mapping的url-pattern 改为 / ,表明servlet允许任何访问方式访问
	-->
    <servlet>
 		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>   
	</servlet-mapping>

  ### 2.spring-mvc.xml配置

    new -> other 搜索spring 选择 spring dean configruation file 新建spring xml

	新建完成在namespace中导入context的命名空间

	![INNER JOIN](https://chinakarl.github.io/images/posts/java/useing-context-namespace.png)

    导入完成后，新增两个配置文件

	<!-- 当前请求的controller的package名称(其实这里是扫描指定package，如果填写的是*,只扫描全部,还有如果写的是package全名后面不能以/*结尾) -->
	<context:component-scan base-package="com.springmvc.handler"></context:component-scan> 

	<!-- 视图页的配置，当前配置是当浏览/都自动浏览.jsp (比如浏览 web/user 其实指向的是user.jsp) -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	    <property name="prefix" value="/"></property>     <!-- 目录前缀 -->
	    <property name="suffix" value=".jsp"></property>  <!-- 目录后缀 -->
	</bean>

## 新建controller

   新建class，如下图

   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/create-class.png)
   
   编写简单代码，代码如下
     
    import org.springframework.stereotype.Controller;
    @Controller
	public class springcontroller {
		@RequestMapping("/login")
		public String Login()
		{
			System.out.print("login....");
			return "success";
		}
	}

	至此就新建完成了,效果如下

	![INNER JOIN](https://chinakarl.github.io/images/posts/java/first-springmvc-web.png)



  
