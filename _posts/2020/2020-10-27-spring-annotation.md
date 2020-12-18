---
layout: post
title:  注解简介
categories: 注解
description: 记录一些不常见的注解
keywords: 注解，annotation
---

  因在工作或学习中会遇到一些不常用的注解，这些注意在解决某些特定事情有奇效，所以特此在这记录下这些注解，方便以后查看和学习。


## EnableConfigurationProperties

 作用：
  
  @EnableConfigurationProperties注解的作用是：使使用 @ConfigurationProperties 注解的类生效。
  
说明
  
  如果一个配置类只配置@ConfigurationProperties注解，而没有使用@Component，那么在IOC容器中是获取不到properties 配置文件转化的bean。
  
  说白了 @EnableConfigurationProperties 相当于把使用 @ConfigurationProperties 的类进行了一次注入
  
## ConditionalOnBean与ConditionalOnClass 等

   [博客地址](https://www.cnblogs.com/qdhxhz/p/11027546.html) 
   
   https://www.cnblogs.com/qdhxhz/p/11027546.html
   
## @Autowired  和 @Resource

*@Autowired*

 @Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照byType注入。

 @Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。如下：

*@Resource*

 @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。

 所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略