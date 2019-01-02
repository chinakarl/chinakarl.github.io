---
layout: post
title:  筛选器与过滤器的区别
categories: spring
description: 对筛选器和过滤器区别的一些研究和记录
keywords: 筛选器，过滤器
---

 筛选器过滤器的一些区别的总结


## 概念(区别)

   ①拦截器是基于Java的反射机制的，而过滤器是基于函数回调。
   ②拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
   ③拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
   ④拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
   ⑤在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。
   ⑥拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。
   
## 实践

   新建TestFilter1 和TestFilter2 都继承OncePerRequestFilter
          代码如下：
          
            protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
                        throws ServletException, IOException {
                    //在DispatcherServlet之前执行
                    System.out.println("------------------TestFilter1 doFilterInternal executed-------------");
                    filterChain.doFilter(request, response);
                    //在视图页面返回给客户端之前执行，但是执行顺序在Interceptor之后
                    System.out.println("---------TestFilter1 doFilter after--------------");
                }
   
   新建TestIntercptor 实现 HandlerInterceptor
          代码如下
          
              /**
               * 在DispatcherServlet之前执行
               * */
              public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
                  System.out.println("--------TestInterceptor preHandle executed--------");
                  return true;
              }
              
              /**
               * 在controller执行之后的DispatcherServlet之后执行
               * */
              public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3)
                      throws Exception {
                  System.out.println("--------TestInterceptor postHandle executed--------");
              }
          
              /**
               * 在页面渲染完成返回给客户端之前执行
               * */
              public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
                      throws Exception {
                  System.out.println("--------TestInterceptor afterCompletion executed--------");
              }
              
   新建TestController 
         代码如下:
         
             @RequestMapping(value = "/test")
             public ModelAndView testHellow()
             {
                 return new ModelAndView("test.jsp");
             }
             
   在返回的 test.jsp中写上java代码输出
   
         <%
             System.out.println("test.jsp is loading");
         %>
   
   web.xml 中添加过滤器配置
   
       <!--自定义过滤器-->
       <filter>
           <filter-name>testFilter2</filter-name>
           <filter-class>com.springmvc.controller.filter.TestFilter2</filter-class>
       </filter>
       <filter-mapping>
           <filter-name>testFilter2</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
       <filter>
           <filter-name>testFilter1</filter-name>
           <filter-class>com.springmvc.controller.filter.TestFilter1</filter-class>
       </filter>
       <filter-mapping>
           <filter-name>testFilter1</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
         
   在springmvc 配置中添加拦截器配置
   
        <mvc:interceptors>
             <!-- 对所有请求都拦截，公共拦截器可以有多个 -->
             <bean name="baseInterceptor" class="com.springmvc.controller.interceptor.TestInterceptor" />
             <mvc:interceptor>
                  <!-- 对 指定接口进行拦截 -->
                  <mvc:mapping path="/test"/>
                  <!-- 特定请求的拦截器只能有一个 -->
                  <bean class="com.springmvc.controller.interceptor.TestInterceptor" />
             </mvc:interceptor>
        </mvc:interceptors>
        
   执行上面代码运行结果如图
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/filter-intercptor-test.png)
   
   从上图中我们可以看出，过滤器和拦截器执行是有一定顺序的，花了个大概的执行流程图如下
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/java/excute-flow.png)
   
## 总结

   从灵活性上说拦截器功能更强大些，Filter能做的事情，都能做，而且可以在请求前，请求后执行，
   比较灵活。Filter主要是针对URL地址做一个编码的事情、过滤掉没用的参数、安全校验（比较泛的，比如登录不登录之类），太细的话，
   还是建议用interceptor。

   