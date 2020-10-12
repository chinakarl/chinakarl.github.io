---
layout: post
title:  dubbo面试题
categories: dubbo
description: dubbo 常见面试题
keywords: dubbo
---

 归总dubbo常见的面试题

## 一.dubbo的核心服务是什么

  1.远程通讯: 提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
  
  2.集群容错: 提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
  
  3.自动发现: 基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。


## 二.dubbo能做什么？

   1.透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需简单配置，没有任何API侵入。
   
   2.软负载均衡及容错机制，可在内网替代F5等硬件负载均衡器，降低成本，减少单点。
   
   3. 服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者。
   
   采用spring的配置方式进行配置，完全透明化的接入应用，对应用没有任何入侵，只需要spring加载dubbo的配置就可以了。

## 三.服务提供者暴露一个服务的详细过程
  
   首先ServiceConfig类拿到对外提供服务的实际类ref(如：HelloWorldImpl),然后通过ProxyFactory类的getInvoker方法使用ref生成一个AbstractProxyInvoker实例，
   
   到这一步就完成具体服务到Invoker的转化。接下来就是Invoker转换到Exporter的过程。
   
   Dubbo处理服务暴露的关键就在Invoker转换到Exporter的过程(如上图中的红色部分)，下面我们以Dubbo和RMI这两种典型协议的实现来进行说明：
   
   Dubbo的实现
   
   Dubbo协议的Invoker转为Exporter发生在DubboProtocol类的export方法，它主要是打开socket侦听服务，并接收客户端发来的各种请求，通讯细节由Dubbo自己实现。
   
   RMI的实现
   
   RMI协议的Invoker转为Exporter发生在RmiProtocol类的export方法，
   它通过Spring或Dubbo或JDK来实现RMI服务，通讯细节这一块由JDK底层来实现，这就省了不少工作量。
    
    
   ![INNER JOIN](https://chinakarl.github.io/images/posts/dubbo/provider-flow.png)

## 四.服务消费者消费一个服务的详细过程

  首先ReferenceConfig类的init方法调用Protocol的refer方法生成Invoker实例(如上图中的红色部分)，这是服务消费的关键。
  
  接下来把Invoker转换为客户端需要的接口(如：HelloWorld)。
  
  ![INNER JOIN](https://chinakarl.github.io/images/posts/dubbo/consumer-flow.png)

## 五.Dubbo主要的配置项有哪些，作用是什么？

    <dubbo:application name="hello-world-app"  />
   
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
   
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
   
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service ref="loginService" interface="com.xinke.management.service.LoginService"/>
    
     <!-- 引用远程服务配置, 可以和本地bean一样使用 -->
     <dubbo:reference id="commonParamApi" interface="com.ai.market.system.api.ICommonParamApi" />
     
## 六.Dubbo有几种容错机制

   什么是容错机制？容错机制指的是某中系统控制在一定范围的一种允许或包容犯错情况的发生，举个简单的例子，我们在电脑上运行一个程序，有时候会出现无响应的情况，然后系统回弹出一个提示框让我们选择，是立即结束还是继续等待，然后根据我们的选择执行对应的操作，这就是“容错”。
   
   在分布式架构下，网络，硬件，应用都可以发生故障，由于各个服务之间可能存在依赖关系，如果一条链路中的某一个节点出现故障，将会导致雪崩效应。为了减少某一个节点故障的影响范围，所以我们才需要去构建容错服务，来优雅的处理这种中断的响应结果
   
   1.failsafe 失败安全，可以认为是把错误吞掉（记录日志）
   
   2.failover(默认)  重试其他服务器；retries(2)重试的次数，默认为2次
   
   3.failback   失败后自动恢复
   
   4.forking forks. 设置并行数
   
   5.Broadcast 广播，任意一台报错，则执行的方法报错，通过cluster方式，配置制定的容错方案     
   
## 七.dubbo的优先级配置

   1.以timeout为例，显示了配置的查找顺序，其他retries，loadbalance等类似。
   
   （1）方法级优先，接口级次之，全局配置在次之
   
   （2）如果级别一样，则消费方优先，提供方次之
   
   （3）其中，服务提供方配置，通过URL经由注册中心传递给消费方
   
   2.建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置。
   
   
  更多：[地址](https://blog.csdn.net/moakun/article/details/82919804)