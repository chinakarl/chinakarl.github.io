---
layout: post
title:  springmvc加载过程
categories: springmvc
description: springmvc源码解析
keywords: springmvc,源码
---

研究下springmvc加载过程


## 基础

   在springmvc使用中，我相信大家对其中的web.xml中配置都很熟悉，特别是ContextLoaderListener
   
   和DispatcherServlet

   我们先来看下 ContextLoaderListener 类的层次结构

   ![INNER JOIN](https://chinakarl.github.io/images/posts/springmvc/ContextLoaderListener.png)
  
   从中我们发现 其继承了 ContextLoader 和实现了 ServletContextListener
   
   现在我们就先从 ContextLoader入手
   既然 ContextLoaderListener实现了ServletContextListener，我们知道ServletContextListener我们知道是web容器创建的时候开始执行contextInitialized() 方法，该方法中调用了 initWebApplicationContext(初始化WebApplicationContext对象)方法，
   
   initWebApplicationContext 解析:
   
           //01 首先判断servletContext中是否存在WebApplicationContext实例，如果存在说明ServletContextListener在web.xml中多次声明，并抛出异常。
            if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
                       throw new IllegalStateException("Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!");
                   } else {
                       Log logger = LogFactory.getLog(ContextLoader.class);
                       servletContext.log("Initializing Spring root WebApplicationContext");
                       if (logger.isInfoEnabled()) {
                           logger.info("Root WebApplicationContext: initialization started");
                       }
                       long startTime = System.currentTimeMillis();
                       try {
                           if (this.context == null) {
                               //02 调用 createWebApplicationContext() 方法创建WebApplicationContext实例
                               this.context = this.createWebApplicationContext(servletContext);
                           }
                           if (this.context instanceof ConfigurableWebApplicationContext) {
                               ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)this.context;
                               if (!cwac.isActive()) {
                                   if (cwac.getParent() == null) {
                                       ApplicationContext parent = this.loadParentContext(servletContext);
                                       cwac.setParent(parent);
                                   }
                                   //03 调用configureAndRefreshWebApplicationContext() 方法通过WebApplicationContext进行解析web.xml中配置的applicationContext.xml
                                   this.configureAndRefreshWebApplicationContext(cwac, servletContext);
                               }
                           }
                           //04 把WebApplicationContext实例添加到ServletContext中
                           servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
  
  

   createWebApplicationContext 方法:
              过createWebApplicationContext(servletContext)创建root上下文（即IOC容器），
              之后Spring会以WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE属性为Key，将该root上下文存储到ServletContext中
  
                protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
                        //01 获取WebApplicationContext class实例
                        Class<?> contextClass = this.determineContextClass(sc);
                        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
                            throw new ApplicationContextException("Custom context class [" + contextClass.getName() + "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
                        } else {
                            //02 返回WebApplicationContext对象
                            return (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
                        }
                    }
                    

   determineContextClass 方法
               方法用于查找root上下文的Class类型,如果web.xml中配置了实现ConfigurableWebApplicationContext的contextClass类型就用那个参数，
               否则使用默认的XmlWebApplicationContext
               
                 protected Class<?> determineContextClass(ServletContext servletContext) {
                     String contextClassName = servletContext.getInitParameter("contextClass");
                     //01 判断 servletContext中是否存在contextClass
                     if (contextClassName != null) {
                         try {
                             return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
                         } catch (ClassNotFoundException var4) {
                             throw new ApplicationContextException("Failed to load custom context class [" + contextClassName + "]", var4);
                         }
                     } else {
                         //02 从defaultStrategies中通过WebApplicationContext全类名（包名和类名）获取要实现WebApplicationContext接口的类
                         contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
                         try {
                             return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
                         } catch (ClassNotFoundException var5) {
                             throw new ApplicationContextException("Failed to load default context class [" + contextClassName + "]", var5);
                         }
                     }
                 }
                 
                 
   determineContextClass 方法
   
                 protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
                         String configLocationParam;
                         if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
                             configLocationParam = sc.getInitParameter("contextId");
                             if (configLocationParam != null) {
                                 wac.setId(configLocationParam);
                             } else {
                                 wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(sc.getContextPath()));
                             }
                         }
                 
                         wac.setServletContext(sc);
                         //01 获取web.xml中配置的applicationContext.xml路径 
                         configLocationParam = sc.getInitParameter("contextConfigLocation");
                         if (configLocationParam != null) {
                             wac.setConfigLocation(configLocationParam);
                         }
                 
                         ConfigurableEnvironment env = wac.getEnvironment();
                         if (env instanceof ConfigurableWebEnvironment) {
                             ((ConfigurableWebEnvironment)env).initPropertySources(sc, (ServletConfig)null);
                         }
                 
                         this.customizeContext(sc, wac);
                         //02 调用refresh()进行加载xml，解析bean
                         wac.refresh();
                     }

