---
layout: post
title:  mybatis中mapper配置节点的学习
categories: mybatis
description: 对mybatis中的mapper的学习
keywords: mybatis,mapper
---

对mybatis中mapper的配置学习

## 什么事mybatis

 MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。
 
 MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
 
 MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。


## 从 XML 中构建 SqlSessionFactory

  每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得

  XML 配置文件（configuration XML）中包含了对 MyBatis 系统的核心设置，包含获取数据库连接实例的数据源（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）

  正常xml中配置的
	<configuration>
	  <environments default="development">
		<environment id="development">
		  <transactionManager type="JDBC"/>
		  <dataSource type="POOLED">
			<property name="driver" value="${driver}"/>
			<property name="url" value="${url}"/>
			<property name="username" value="${username}"/>
			<property name="password" value="${password}"/>
		  </dataSource>
		</environment>
	  </environments>
	  <mappers>
		<mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>

  springmvc中配置

  <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="${db.driverClassName}"/>
        <property name="url" value="${db.url}"/>
        <property name="username" value="${db.username}"/>
        <property name="password" value="${db.password}"/>
        <property name="maxTotal" value="${db.maxTotal}"/>
        <property name="maxIdle" value="${db.minIdle}"/>
        <property name="maxWaitMillis" value="${db.maxWaitMillis}"/>
        <property name="defaultAutoCommit" value="${db.defaultAutoCommit}"/>
        <property name="validationQuery" value="${db.validationQuery}"/>
        <property name="testOnBorrow" value="${db.testOnBorrow}"/>
        <property name="poolPreparedStatements" value="${db.poolPreparedStatements}"/>
    </bean>

## 不使用 XML 构建 SqlSessionFactory



  
