---
layout: post
title:  ssm 配置
categories: spring
description: ssm框架简单的搭建和spring的事务aop等简单实现
keywords: spring
---

 ssm框架简单的搭建和spring的中通过配置和注解来实现事务和aop等


## 配置

   1. web.xml 配 置
   
     <context-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:mybatis/spring.xml</param-value>
       </context-param>
       配置监听器，加载各个配置用
       <listener>
           <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
       </listener>
       配置分发器，springmvc请求用到
       <servlet>
           <servlet-name>dispatcher</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:spring/springmvc.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       配置mapping，url要用/不能用/*
       <servlet-mapping>
           <servlet-name>dispatcher</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
       
   2. springmvc.xml 配置
   
     扫描所有controller
        <context:component-scan base-package="com.haixiang.*.controller"/>
        启动注解方式
        <mvc:annotation-driven  />
        
   3. spring.xml 配置 

     <!-- 引入属性文件，读取jdbc，为mybatis做铺垫 -->
        <bean id="propertyConfigurer"
            class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
            <property name="systemPropertiesModeName" value="SYSTEM_PROPERTIES_MODE_OVERRIDE" />
            <property name="ignoreResourceNotFound" value="true" />
            <property name="locations">
                <list>
                    <value>classpath:mybatis/jdbc.properties</value>
                </list>
            </property>
        </bean>
        导入mybatis文件
        <import resource="mybatis.xml"/>
        
   4. mybatis.xml 配置
   
     <context:component-scan base-package="com.haixiang"></context:component-scan>
         <!-- 配置数据库相关参数properties的属性：${url} -->
         <context:property-placeholder location="classpath:mybatis/jdbc.properties"/>
     
         <!-- 数据库连接池，这里使用的是阿里的连接池 -->
         <bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
             <property name="url" value="${jdbc.url}" />
             <property name="username" value="${jdbc.username}" />
             <property name="password" value="${jdbc.password}" />
             <property name="initialSize" value="${jdbc.initialSize}" />
             <property name="minIdle" value="${jdbc.minIdle}" />
             <property name="maxActive" value="${jdbc.maxActive}" />
             <property name="maxWait" value="${jdbc.maxWait}" />
             <property name="timeBetweenEvictionRunsMillis" value="${jdbc.timeBetweenEvictionRunsMillis}" />
             <property name="minEvictableIdleTimeMillis" value="${jdbc.minEvictableIdleTimeMillis}" />
             <property name="validationQuery" value="${jdbc.validationQuery}" />
             <property name="testWhileIdle" value="${jdbc.testWhileIdle}" />
             <property name="testOnBorrow" value="${jdbc.testOnBorrow}" />
             <property name="testOnReturn" value="${jdbc.testOnReturn}" />
             <property name="removeAbandoned" value="${jdbc.removeAbandoned}" />
             <property name="removeAbandonedTimeout" value="${jdbc.removeAbandonedTimeout}" />
             <!--	    <property name="logAbandoned" value="${jdbc.logAbandoned}" /> -->
             <property name="filters" value="${jdbc.filters}" />
             <!-- 关闭abanded连接时输出错误日志 -->
             <property name="logAbandoned" value="true" />
         </bean>
          <!-- 配置SqlSessionFactory -->
         <bean name="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
             <property name="dataSource" ref="dataSource"/>
             <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
             <!-- 配置mapper地址,就是mapper.xml所在的路径 -->
             <property name="mapperLocations" value="classpath:mapper/*.xml"/>
         </bean>
         <!-- 配置mybatis扫描的dao，就是写的接口和上面配置的mapper 1-1对应的 -->
         <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
             <property name="basePackage" value="com.haixiang.common.dao" />
             <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
         </bean>
     
         <!--事务管理器-->
         <!--<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">-->
             <!--<property name="dataSource" ref="dataSource"/>-->
         <!--</bean>-->
         <!--事务拦截方式的配置,记得一定要加入下面的aop配置，不然拦截的事务不会起作用的-->
         <!--<tx:advice id="txAdvice"  transaction-manager="txManager" >-->
             <!--<tx:attributes>-->
                 <!--<tx:method name="publish*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="save*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="add*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="update*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="insert*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="create*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="del*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="load*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="init*" propagation="REQUIRED"/>-->
                 <!--<tx:method name="*"  propagation="SUPPORTS"/>-->
             <!--</tx:attributes>-->
         <!--</tx:advice>-->
         <!--<bean id="aopServiceImpl" class="com.service.impl.ArticleServiceImpl"></bean>-->
         <!--<bean id="aspectDemo" class="com.service.aop.TransactionImpl"></bean>-->
         <aop:aspectj-autoproxy/>
         <!--aop配置-->
         <!--<aop:config>-->
             <!--&lt;!&ndash;-->
                 <!--1. 第一个 * 号表示返回值 *表示所有类型-->
                 <!--2. com.service.. 表示com.service包和他所有的子包-->
                 <!--3. *Impl 表示所有以Impl结尾的类-->
                 <!--4. *(..) *号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数-->
             <!--&ndash;&gt;-->
             <!--<aop:pointcut id="aspectPointcut" expression="execution(public * com.service.impl.ArticleServiceImpl.saveArticle(..))" />-->
             <!--<aop:aspect ref="aspectDemo">-->
                 <!--<aop:before method="before" pointcut-ref="aspectPointcut"></aop:before>-->
                 <!--<aop:after-returning method="after" pointcut-ref="aspectPointcut"></aop:after-returning>-->
             <!--</aop:aspect>-->
     
             <!--&lt;!&ndash;事务配置&ndash;&gt;-->
             <!--&lt;!&ndash;<aop:advisor pointcut-ref="transactionPointcut"&ndash;&gt;-->
             <!--&lt;!&ndash;advice-ref="txAdvice" />&ndash;&gt;-->
             <!--&lt;!&ndash;<aop:pointcut id="aspectPointcut" expression="execution(* com.service..*.*(..))" />&ndash;&gt;-->
         <!--</aop:config>-->
         <!-- 开启事务注解，标注@Transactional的类和方法将具有事务性 -->
         <!--<tx:annotation-driven transaction-manager="txManager" />-->
        
   示例地址 ：https://github.com/chinakarl/ssm.git
   

   