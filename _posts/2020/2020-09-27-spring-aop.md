---
layout: post
title:  spring aop
categories: spring
description: spring aop详解
keywords: spring，aop
---

   AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
 AOP是OOP（面向对象编程）的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容（Spring核心之一），是函数式编程的一种衍生范型。
 利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。


# 作用

   为了更清晰的逻辑，可以让你的业务逻辑去关注自己本身的业务，而不去想一些其他的事情，这些其他的事情包括：安全，事务，日志等

# 概念  

## 通知

  
##  spring 事务
  
### spring 事务实现方式

1.  基于 TransactionProxyFactoryBean的声明式事务管理
2.  基于 @Transactional 的声明式事务管理
3.  基于Aspectj AOP配置事务
   
   
   
#### TransactionProxyFactoryBean 方式实现
   
    <bean id="articleService" class="com.haixiang.service.impl.ArticleServiceImpl"></bean>
   
     <bean id="serviceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
           <property name="transactionManager" ref="txManager"></property>
           <property name="target" ref="articleService"></property>
           <property name="transactionAttributes">
               <props>
                   <!-- 主要 key 是方法
                       ISOLATION_DEFAULT  事务的隔离级别
                       PROPAGATION_REQUIRED  传播行为
                   -->
                   <prop key="add*">ISOLATION_DEFAULT,PROPAGATION_REQUIRED</prop>
                   <!-- -Exception 表示发生指定异常回滚，+Exception 表示发生指定异常提交 -->
                   <prop key="buyStock">ISOLATION_DEFAULT,PROPAGATION_REQUIRED,-BuyStockException</prop>
               </props>
           </property>
   
       </bean>
   
####   @Transactional 方式实现
   
   配置文件添加一下配置
   
     <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource"/>
       </bean>
   
     <tx:annotation-driven transaction-manager="txManager" />
  
  代码方法上添加此注解
 
  * @Transactional(timeout=60)
  
   在配置的时间内，如果事务内CRUD操作未执行完毕，则报超时错误。
 
   @Transactional(rollbackFor=Exception.class)
   
   事务异常后回滚。什么情况下回滚呢？
   在抛出 RuntimeException 或者 Error 的时候。
   
   注意：如果事务在try{}catch(Exception e){e.printStackTrace();}中跑，并且catch中只是打印e的话，那么事务不会rollback。因为异常被catch掉了，框架不知道发生了异常。
   　　如果想要rollback，可以加上rollbackFor=Exception.class，然后：
   　　①在方法上添加 throws  Exception，将方法中出现的异常抛出给spring事务，
   　　②去掉方法体中的try catch
   　　③catch (Exception e) {  throw e;}继续向上抛，目的是让spring事务捕获这个异常。
   　　　　　　rollbackFor=Exception.class，catch(){
       　　　　　　　throw new RunTimeException();
   　　　　　　}
   　　如果不加rollbackFor=Exception.class，抛出new Exception() 是不会回滚的。Spring源码如下：
   　　　　public boolean rollbackOn(Throwable ex) { 
        　　　　return (ex instanceof RuntimeException || ex instanceof Error);
   　　　　} 
   
   
####  基于Aspectj AOP配置事务
 
    <tx:advice id="txAdvice"  transaction-manager="txManager" >
           <tx:attributes>
               <tx:method name="publish*" propagation="REQUIRED"/>
               <tx:method name="save*" propagation="REQUIRED"/>
               <tx:method name="add*" propagation="REQUIRED"/>
               <tx:method name="update*" propagation="REQUIRED"/>
               <tx:method name="insert*" propagation="REQUIRED"/>
               <tx:method name="create*" propagation="REQUIRED"/>
               <tx:method name="del*" propagation="REQUIRED"/>
               <tx:method name="load*" propagation="REQUIRED"/>
               <tx:method name="init*" propagation="REQUIRED"/>
               <tx:method name="*"  propagation="SUPPORTS"/>
           </tx:attributes>
       </tx:advice>
    
        <!--aop配置-->
        <aop:config>
            <!--
                1. 第一个 * 号表示返回值 *表示所有类型
                2. com.service.. 表示com.service包和他所有的子包
                3. *Impl 表示所有以Impl结尾的类
                4. *(..) *号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数
            -->
            <aop:pointcut id="aspectPointcut" expression="execution(public * com.service.impl.ArticleServiceImpl.saveArticle(..))" />
            <aop:aspect ref="aspectDemo">
                <aop:before method="before" pointcut-ref="aspectPointcut"></aop:before>
                <aop:after-returning method="after" pointcut-ref="aspectPointcut"></aop:after-returning>
            </aop:aspect>
    
            <!--事务配置-->
            <!--<aop:advisor pointcut-ref="transactionPointcut"-->
            <!--advice-ref="txAdvice" />-->
            <!--<aop:pointcut id="aspectPointcut" expression="execution(* com.service..*.*(..))" />-->
        </aop:config>   
       
## spring 事务特性

### spring 事务传播属性

  所谓spring事务的传播属性，就是定义在存在多个事务同时存在的时候，spring应该如何处理这些事务的行为
  
*1. PROPAGATION_REQUIRED
 
   使用该级别的特点是，如果上下文中已经存在事务，那么就加入到事务中执行，如果当前上下文中不存在事务，则新建事务执行。所以这个级别通常能满足处理大多数的业务场景。
  
*2. PROPAGATION_SUPPORTS
 
   从字面意思就知道，supports，支持，该传播级别的特点是，如果上下文存在事务，则支持事务加入事务，如果没有事务，则使用非事务的方式执行。
 所以说，并非所有的包在transactionTemplate.execute中的代码都会有事务支持。这个通常是用来处理那些并非原子性的非核心业务逻辑操作。应用场景较少。
  
*3. PROPAGATION_MANDATORY
 
   该级别的事务要求上下文中必须要存在事务，否则就会抛出异常！配置该方式的传播级别是有效的控制上下文调用代码遗漏添加事务控制的保证手段。
 比如一段代码不能单独被调用执行，但是一旦被调用，就必须有事务包含的情况，就可以使用这个传播级别。
  
*4. PROPAGATION_REQUIRES_NEW 

  从字面即可知道，new，每次都要一个新事务，该传播级别的特点是，每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。
  
  这是一个很有用的传播级别，举一个应用场景：现在有一个发送100个红包的操作，在发送之前，要做一些系统的初始化、验证、数据记录操作，然后发送100封红包，然后再记录发送日志，发送日志要求100%的准确，如果日志不准确，那么整个父事务逻辑需要回滚。
  怎么处理整个业务需求呢？就是通过这个PROPAGATION_REQUIRES_NEW 级别的事务传播控制就可以完成。发送红包的子事务不会直接影响到父事务的提交和回滚。
  
*5. PROPAGATION_NOT_SUPPORTED

  这个也可以从字面得知，not supported ，不支持，当前级别的特点就是上下文中存在事务，则挂起事务，执行当前逻辑，结束后恢复上下文的事务。
  
  这个级别有什么好处？可以帮助你将事务极可能的缩小。我们知道一个事务越大，它存在的风险也就越多。所以在处理事务的过程中，要保证尽可能的缩小范围。
  比如一段代码，是每次逻辑操作都必须调用的，比如循环1000次的某个非核心业务逻辑操作。这样的代码如果包在事务中，势必造成事务太大，导致出现一些难以考虑周全的异常情况。
  所以这个事务这个级别的传播级别就派上用场了。用当前级别的事务模板抱起来就可以了。
  
*6. PROPAGATION_NEVER
 
  该事务更严格，上面一个事务传播级别只是不支持而已，有事务就挂起，而PROPAGATION_NEVER传播级别要求上下文中不能存在事务，一旦有事务，就抛出runtime异常，强制停止执行！这个级别上辈子跟事务有仇。
  
*7. PROPAGATION_NESTED

  字面也可知道，nested，嵌套级别事务。该传播级别特征是，如果上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务
  
### spring 事务隔离级别

在TransactionDefinition接口中定义了五个不同的事务隔离级别

*1. ISOLATION_DEFAULT
 
  这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.另外四个与JDBC的隔离级别相对应
 
*2. ISOLATION_READ_UNCOMMITTED 

  这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读

*3. ISOLATION_READ_COMMITTED 

  保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。

*4. ISOLATION_REPEATABLE_READ 

  这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。

*5. ISOLATION_SERIALIZABLE 

  这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。


