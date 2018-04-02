---
layout: post
title:  spring的所有jar包的用途
categories: Java
description: 简单阐述下spring的所有jar包的用途
keywords: 阿里云,jar,aliyun
---

spring的所有jar包的用途
  
1.spring.jar

　　是包含有完整发布模块的单个jar 包。

2. org.springframework.aop

　　包含在应用中使用Spring的AOP特性时所需的类。

3. org.springframework.asm

　　Spring独立的asm程序, Spring2.5.6的时候需要asmJar 包, 3.0开始提供他自己独立的asmJar。

4. org.springframework.aspects

　　提供对AspectJ的支持，以便可以方便的将面向方面的功能集成进IDE中， 比如Eclipse AJDT。

5. org.springframework.beans

　　所有应用都要用到的，它包含访问配置文件、创建和管理bean 以及进行Inversion of Control / Dependency Injection（IoC/DI）操作相关的所有类。

6. org.springframework.context.support

　　包含支持缓存Cache（ehcache）、JCA、JMX、 邮件服务（Java Mail、COS Mail）、任务计划Scheduling（Timer、Quartz）方面的类。

7. org.springframework.context

　　为Spring核心提供了大量扩展。可以找到使用Spring ApplicationContext 特性时所需的全部类，JDNI所需的全部类，UI方面的用来与模板（Templating）引擎如 Velocity、FreeMarker、 JasperReports集成的类，以及校验Validation方面的相关类。

8. org.springframework.core

　　包含Spring框架基本的核心工具类，Spring其它组件要都要使用到这个包里的类， 是其它组件的基本核心。

9. org.springframework.expression

　　Spring表达式语言。

10. org.springframework.instrument.tomcat

　　Spring3.0对Tomcat的连接池的集成。

11. org.springframework.instrument

　　Spring3.0对服务器的代理接口。

12. org.springframework.jdbc

　　包含对Spring对JDBC数据访问进行封装的所有类。

13. org.springframework.jms

　　提供了对JMS 1.0.2/1.1的支持类。http://baike.baidu.com/link?url=HfioUoiPUtvgtXEECsRhPJ2Ek2oWQwZgWObWUMF36PQHJ2CFiJ2nb-hxGcfj4kFumjr6-0J9FAnHOkVrbkubYq

14. org.springframework.orm

　　包含Spring对DAO特性集进行了扩展，使其支持 iBATIS、JDO、OJB、TopLink， 因为Hibernate已经独立成包了，现在不包含在这个包里了。这个jar文件里大部分的类都要依赖spring-dao.jar 里的类，用这个包时你需要同时包含spring-dao.jar包。

15. org.springframework.oxm

　　Spring 对Object/XMl的映射支持,可以让Java与XML之间来回切换。

16. org.springframework.test

　　对Junit等测试框架的简单封装。

17. org.springframework.transaction

　　为JDBC、Hibernate、JDO、JPA等提供的一致的声明式和编程式事务管理。

18. org.springframework.web.portlet

　　SpringMVC的增强。

19. org.springframework.web.servlet

　　对J2EE6.0 的Servlet3.0的支持。

20. org.springframework.web.struts

　　Struts框架支持，可以更方便更容易的集成Struts框架。

21. org.springframework.web

　　包含Web应用开发时，用到Spring框架时所需的核心类，包括自动载入 WebApplicationContext特性的类、Struts与JSF集成类、文件上传的支持类、Filter类和大量工具辅助类

22.Spring webmvc：

　　包含SpringMVC框架相关的所有类。包含国际化、标签、Theme、视图展现的FreeMarker、JasperReports、Tiles、Velocity、XSLT相关类。当然，如果你的应用使用了独立的MVC框架，则无需这个JAR文件里的任何类。
 23.Spring webmvc portlet：Spring MVC的增强

24.spring-tx.3.2.2.jar

spring提供对事务的支持，事务的相关处理以及实现类就在这个Jar包中

  

  
