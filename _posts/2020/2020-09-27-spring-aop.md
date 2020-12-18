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

   将项目中公有的抽取出来并且设立为独立的抽象

# 概念  

## AOP 术语
  
*切面(Aspect)*
  
    切面是一个横切关注点的模块化，一个切面能够包含同一个类型的不同增强方法，比如说事务处理和日志处理可以理解为两个切面。切面由切入点和通知组成，它既包含了横切逻辑的定义，也包括了切入点的定义。
   
  Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的连接点中。
  
        @Component
        @Aspect
        public class LogAspect {
        }

  正常理解，使用@Aspect 注解的就是切面
  
*目标对象(Target)* 
  
   需要被加强的业务对象或者说是被一个或多个切面所通知的对象。
   
*连接点（Joinpoint)*
  
   程序执行的某个特定位置，如某个方法调用前，调用后，方法抛出异常后，这些代码中的特定点称为连接点。
   
   简单来说，连接点就是被拦截到的程序执行点，因为Spring只支持方法类型的连接点，所以在Spring中连接点就是被拦截到的方法。
  
   @Before("pointcut()")
   @Before("pointcut()") public void log(JoinPoint joinPoint) { //这个JoinPoint
   }
   
*切点（PointCut）*

   切入点是对连接点进行拦截的条件定义。切入点表达式如何和连接点匹配是AOP的核心，Spring缺省使用AspectJ切入点语法。 
    一般认为，所有的方法都可以认为是连接点，但是我们并不希望在所有的方法上都添加通知，而切入点的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配连接点，给满足规则的连接点添加通知。
    
    @Pointcut("execution(* com.remcarpediem.test.aop.service..*(..))") 
    ublic void pointcut() { }
    
*通知（Advice）*    

   通知是指拦截到连接点之后要执行的代码,包括了“around”、“before”和“after”等不同类型的通知。
   Spring AOP框架以拦截器来实现通知模型,并维护一个以连接点为中心的拦截器链。
    
    // @Before说明这是一个前置通知，log函数中是要前置执行的代码，JoinPoint是连接点，
    @Before（"pointcut"）
    public void log(JoinPoint joinPoint){}
    
   前置通知(before):在执行业务代码前做些操作，比如获取连接对象
   后置通知(after):在执行业务代码后做些操作，无论是否发生异常，它都会执行，比如关闭连接对象
   异常通知（afterThrowing）:在执行业务代码后出现异常，需要做的操作
   返回通知(afterReturning),在执行业务代码后无异常，会执行的操作
   环绕通知(around)，一般在线程安全情况下使用
   
*织入（Weaving）*

  织入是将切面和业务逻辑对象连接起来, 并创建通知代理的过程。织入可以在编译时，类加载时和运行时完成。
   在编译时进行织入就是静态代理，而在运行时进行织入则是动态代理。
   
   
## Spirng Aop 底层实现方式

  spring aop 底层是用代理实现的，分为编译时和运行时。如果编译时 就是 静态代理，如果是运行时就是动态代理。
  
*静态代理*
  
   所谓静态代理就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强。ApsectJ是静态代理的实现之一，也是最为流行的。
   静态代理由于在编译时就生成了代理类，效率相比动态代理要高一些。AspectJ可以单独使用，也可以和Spring结合使用。   

*动态代理*

  与静态代理不同，动态代理就是说AOP框架不会去修改编译时生成的字节码，而是在运行时在内存中生成一个AOP代理对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。
 
   Spring AOP中的动态代理主要有两种方式：JDK动态代理和CGLIB动态代理。
   
   JDK代理通过反射来处理被代理的类，并且要求被代理类必须实现一个接口。核心类是 InvocationHandler接口 和 Proxy类。
   
   而当目标类没有实现接口时，Spring AOP框架会使用CGLIB来动态代理目标类。
   CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类。
   CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。
   核心类是 MethodInterceptor 接口和Enhancer 类
   
   看图
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/springaop/springaop.png)
   
 
