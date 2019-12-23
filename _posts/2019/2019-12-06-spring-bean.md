---
layout: post
title:  spring bean 加载过程
categories: spring
description: spring bean 加载过程
keywords: spring，bean
---

   spring 是一个开源框架，其中心思想是以依赖注入的理念将所有对像以bean的方式注入到spring中，最大限度的实现了解耦，更加方便了
   日常的开发，现已成为我们工作中必会技能之一...

# 细节分析
  
## Bean 加载
  
  Bean加载过程，我们围绕AbstractBeanFactory来讲解
  
  *下面贴出重要代码*
  
    public Object getBean(String name, Object... args) throws BeansException {
          return this.doGetBean(name, (Class)null, args, false);
      }
      
      protected <T> T doGetBean(String name, Class<T> requiredType, final Object[] args, boolean typeCheckOnly) throws BeansException {
              final String beanName = this.transformedBeanName(name);
              Object sharedInstance = this.getSingleton(beanName);
              Object bean;
              if (sharedInstance != null && args == null) {
                  if (this.logger.isDebugEnabled()) {
                      if (this.isSingletonCurrentlyInCreation(beanName)) {
                          this.logger.debug("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
                      } else {
                          this.logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
                      }
                  }
      
                  bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
              } else {
                  if (this.isPrototypeCurrentlyInCreation(beanName)) {
                      throw new BeanCurrentlyInCreationException(beanName);
                  }
      
                  BeanFactory parentBeanFactory = this.getParentBeanFactory();
                  if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
                      String nameToLookup = this.originalBeanName(name);
                      if (args != null) {
                          return parentBeanFactory.getBean(nameToLookup, args);
                      }
      
                      return parentBeanFactory.getBean(nameToLookup, requiredType);
                  }
      
                  if (!typeCheckOnly) {
                      this.markBeanAsCreated(beanName);
                  }
      
                  try {
                      final RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
                      this.checkMergedBeanDefinition(mbd, beanName, args);
                      String[] dependsOn = mbd.getDependsOn();
                      String[] var11;
                      if (dependsOn != null) {
                          var11 = dependsOn;
                          int var12 = dependsOn.length;
      
                          for(int var13 = 0; var13 < var12; ++var13) {
                              String dependsOnBean = var11[var13];
                              if (this.isDependent(beanName, dependsOnBean)) {
                                  throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dependsOnBean + "'");
                              }
      
                              this.registerDependentBean(dependsOnBean, beanName);
                              this.getBean(dependsOnBean);
                          }
                      }
      
                      if (mbd.isSingleton()) {
                          sharedInstance = this.getSingleton(beanName, new ObjectFactory<Object>() {
                              public Object getObject() throws BeansException {
                                  try {
                                      return AbstractBeanFactory.this.createBean(beanName, mbd, args);
                                  } catch (BeansException var2) {
                                      AbstractBeanFactory.this.destroySingleton(beanName);
                                      throw var2;
                                  }
                              }
                          });
                          bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                      } else if (mbd.isPrototype()) {
                          var11 = null;
      
                          Object prototypeInstance;
                          try {
                              this.beforePrototypeCreation(beanName);
                              prototypeInstance = this.createBean(beanName, mbd, args);
                          } finally {
                              this.afterPrototypeCreation(beanName);
                          }
      
                          bean = this.getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
                      } else {
                          String scopeName = mbd.getScope();
                          Scope scope = (Scope)this.scopes.get(scopeName);
                          if (scope == null) {
                              throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                          }
      
                          try {
                              Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                                  public Object getObject() throws BeansException {
                                      AbstractBeanFactory.this.beforePrototypeCreation(beanName);
      
                                      Object var1;
                                      try {
                                          var1 = AbstractBeanFactory.this.createBean(beanName, mbd, args);
                                      } finally {
                                          AbstractBeanFactory.this.afterPrototypeCreation(beanName);
                                      }
      
                                      return var1;
                                  }
                              });
                              bean = this.getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                          } catch (IllegalStateException var21) {
                              throw new BeanCreationException(beanName, "Scope '" + scopeName + "' is not active for the current thread; consider " + "defining a scoped proxy for this bean if you intend to refer to it from a singleton", var21);
                          }
                      }
                  } catch (BeansException var23) {
                      this.cleanupAfterBeanCreationFailure(beanName);
                      throw var23;
                  }
              }
              
              if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
                          try {
                              return this.getTypeConverter().convertIfNecessary(bean, requiredType);
                          } catch (TypeMismatchException var22) {
                              if (this.logger.isDebugEnabled()) {
                                  this.logger.debug("Failed to convert bean '" + name + "' to required type [" + ClassUtils.getQualifiedName(requiredType) + "]", var22);
                              }
              
                              throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
                          }
                      } else {
                          return bean;
                      }
                  }
  代码很多，我们慢慢分析，首先进来的时候我们看到有个方法 this.transformedBeanName(name)
  其中 name是从 getBean中参数传过来的，而getBean中的name可能有这几种情况
  
  * bean name ，可以直接获取到定义 BeanDefinition
  * alias name，别名，需要转化
  * factorybean name, 带 & 前缀，通过它获取 BeanDefinition 的时候需要去除 & 前缀
   
  会调用 SimpleAiasRegistry 的canonicalName方法 ，在调用之前 会调用 BeanFactoryUtils的transformedBeanName方法，其作用主要是
  判断name是否有前缀 & 如果有将其去掉。
  
     this.canonicalName(BeanFactoryUtils.transformedBeanName(name));
     
     public static String transformedBeanName(String name) {
             Assert.notNull(name, "'name' must not be null");
     
             String beanName;
             for(beanName = name; beanName.startsWith("&"); beanName = beanName.substring("&".length())) {
                 ;
             }
             return beanName;
         }
  
## 单例加载
     
 检查是否已经加载过该 Bean的实例，如在初始化或者其它地方调用过getBean()初始化
     
     Object sharedInstance = this.getSingleton(beanName);
 
 > 缓存加载Bean
     
 到DefaultSingletonBeanRegistry 的 getSingleton 方法，代码如下
 
     //保存 BeanName 和创建 bean 实例之间的关系 bean name --> bean instance
     private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);
     
     //保存 BeanName 和创建 bean 实例的工厂之间的关系 bean name --> ObjectFactory
     private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);
     
     //保存 BeanName 和创建 bean 实例之间的关系 bean name --> bean instance */
     //与 singletonObjects 不同的是当一个单例 bean 被放到里面后，那么在 bean 在创建过程中，就可以通过 getBean 方法获取到，可以用来检测循环引用。
     private final Map<String, Object> earlySingletonObjects = new HashMap(16);
     
     //保存当前所有已注册的 bean
     private final Set<String> registeredSingletons = new LinkedHashSet(256);
     
     
     protected Object getSingleton(String beanName, boolean allowEarlyReference) {
             // 尝试从缓存获取实例
             Object singletonObject = this.singletonObjects.get(beanName);
             if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
                 Map var4 = this.singletonObjects;
                 synchronized(this.singletonObjects) {
                     singletonObject = this.earlySingletonObjects.get(beanName);
                     if (singletonObject == null && allowEarlyReference) {
                         ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                         if (singletonFactory != null) {
                             singletonObject = singletonFactory.getObject();
                             this.earlySingletonObjects.put(beanName, singletonObject);
                             this.singletonFactories.remove(beanName);
                         }
                     }
                 }
             }
     
             return singletonObject != NULL_OBJECT ? singletonObject : null;
         }   
                 
 > 构建实例(非缓存)
 
  如果缓存中没有创建个Bean
  
     public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
               Assert.notNull(beanName, "'beanName' must not be null");
               Map var3 = this.singletonObjects;
               synchronized(this.singletonObjects) {
               Object singletonObject = this.singletonObjects.get(beanName);
               if (singletonObject == null) {
                   if (this.singletonsCurrentlyInDestruction) {
                       throw new BeanCreationNotAllowedException(beanName, "Singleton bean creation not allowed while the singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)");
                   }
                   if (this.logger.isDebugEnabled()) {
                       this.logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
                   }
                   this.beforeSingletonCreation(beanName);
                   boolean newSingleton = false;
                   boolean recordSuppressedExceptions = this.suppressedExceptions == null;
                   if (recordSuppressedExceptions) {
                       this.suppressedExceptions = new LinkedHashSet();
                   }
                   try {
                       singletonObject = singletonFactory.getObject();
                       newSingleton = true;
                   } catch (IllegalStateException var16) {
                       singletonObject = this.singletonObjects.get(beanName);
                       if (singletonObject == null) {
                           throw var16;
                       }
                   } catch (BeanCreationException var17) {
                       BeanCreationException ex = var17;
                       if (recordSuppressedExceptions) {
                           Iterator var8 = this.suppressedExceptions.iterator();
   
                           while(var8.hasNext()) {
                               Exception suppressedException = (Exception)var8.next();
                               ex.addRelatedCause(suppressedException);
                           }
                       }
                       throw ex;
                   } finally {
                       if (recordSuppressedExceptions) {
                           this.suppressedExceptions = null;
                       }
                       this.afterSingletonCreation(beanName);
                   }
                   if (newSingleton) {
                       this.addSingleton(beanName, singletonObject);
                   }
               }
               return singletonObject != NULL_OBJECT ? singletonObject : null;
           }
       }
 
 看调用的地方，是创建了一个ObjectFactory
 
    if (mbd.isSingleton()) {
             sharedInstance = this.getSingleton(beanName, new ObjectFactory<Object>() {
                 public Object getObject() throws BeansException {
                     try {
                         return AbstractBeanFactory.this.createBean(beanName, mbd, args);
                     } catch (BeansException var2) {
                         AbstractBeanFactory.this.destroySingleton(beanName);
                         throw var2;
                     }
                 }
             });
             bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
       } 
 
## 合并RootBeanDefinition

### 合并RootBeanDefinition过程

 使用的 BeanDefinition 是 RootBeanDefinition 类型而非 GenericBeanDefinition。这是为什么？

 答案很明显，GenericBeanDefinition 在有继承关系的情况下，定义的信息不足：

* 如果不存在继承关系，GenericBeanDefinition 存储的信息是完整的，可以直接转化为 RootBeanDefinition。

* 如果存在继承关系，GenericBeanDefinition 存储的是 增量信息 而不是 全量信息。
 
*为了能够正确初始化对象，需要完整的信息才行*  需要递归 合并父类的定义：

见 AbstractBeanFactory.doGetBean

      protected <T> T doGetBean ... {
          ...
          
          // 合并父类定义
          final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
              
          ...
              
          // 使用合并后的定义进行实例化
          bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
              
          ...
      }
     
   在判断 parentName 存在的情况下，说明存在父类定义，启动合并。如果父类还有父类怎么办？递归调用，继续合并。
     
   见AbstractBeanFactory.getMergedBeanDefinition 方法
     
         protected RootBeanDefinition getMergedBeanDefinition(String beanName, BeanDefinition bd, BeanDefinition containingBd) throws BeanDefinitionStoreException {
                Map var4 = this.mergedBeanDefinitions;
                synchronized(this.mergedBeanDefinitions) {
                    RootBeanDefinition mbd = null;
                    if (containingBd == null) {
                        mbd = (RootBeanDefinition)this.mergedBeanDefinitions.get(beanName);
                    }
                    if (mbd == null) {
                        if (bd.getParentName() == null) {
                            if (bd instanceof RootBeanDefinition) {
                                mbd = ((RootBeanDefinition)bd).cloneBeanDefinition();
                            } else {
                                mbd = new RootBeanDefinition(bd);
                            }
                        } else {
                            BeanDefinition pbd;
                            try {
                                String parentBeanName = this.transformedBeanName(bd.getParentName());
                                if (!beanName.equals(parentBeanName)) {
                                    //递归调用
                                    pbd = this.getMergedBeanDefinition(parentBeanName);
                                } else {
                                    if (!(this.getParentBeanFactory() instanceof ConfigurableBeanFactory)) {
                                        throw new NoSuchBeanDefinitionException(bd.getParentName(), "Parent name '" + bd.getParentName() + "' is equal to bean name '" + beanName + "': cannot be resolved without an AbstractBeanFactory parent");
                                    }
        
                                    pbd = ((ConfigurableBeanFactory)this.getParentBeanFactory()).getMergedBeanDefinition(parentBeanName);
                                }
                            } catch (NoSuchBeanDefinitionException var9) {
                                throw new BeanDefinitionStoreException(bd.getResourceDescription(), beanName, "Could not resolve parent bean definition '" + bd.getParentName() + "'", var9);
                            }
        
                            mbd = new RootBeanDefinition(pbd);
                            mbd.overrideFrom(bd);
                        }
        
                        if (!StringUtils.hasLength(mbd.getScope())) {
                            mbd.setScope("singleton");
                        }
        
                        if (containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()) {
                            mbd.setScope(containingBd.getScope());
                        }
        
                        if (containingBd == null && this.isCacheBeanMetadata()) {
                            this.mergedBeanDefinitions.put(beanName, mbd);
                        }
                    }
                    return mbd;
                }
            }
            
   每次合并完父类定义后，都会调用 RootBeanDefinition.overrideFrom 对父类的定义进行覆盖，获取到当前类能够正确实例化的 全量信息。
   
### 处理循环依赖

   > 什么是循环依赖
   
   比如三个类，A,B,C A依赖B，B依赖C，C又依赖A，这样子就叫循环依赖。
   
   循环依赖根据注入的时机分成两种类型：   
   
  * 构造器循环依赖。依赖的对象是通过构造器传入的，发生在实例化 Bean 的时候。
  
  * 设值循环依赖。依赖的对象是通过 setter 方法传入的，对象已经实例化，发生属性填充和依赖注入的时候。
  
  *如果是构造器循环依赖，本质上是无法解决的*
  
  比如我们准调用 A 的构造器，发现依赖 B，于是去调用 B 的构造器进行实例化，发现又依赖 C，于是调用 C 的构造器去初始化，结果依赖 A，整个形成一个死结，导致 A 无法创建。
  
  *如果是设值循环依赖，Spring 框架只支持单例下的设值循环依赖*
  
  Spring 通过对还在创建过程中的单例，缓存并提前暴露该单例，使得其他实例可以引用该依赖
  
## 创建实例

   获取到完整的 RootBeanDefintion 后，就可以拿这份定义信息来实例具体的 Bean
   
   在AbstractBeanFactory 中调用了 createBean
   
    AbstractBeanFactory.this.createBean(beanName, mbd, args);
    
   其继承类 AbstractAutowireCapableBeanFactory 实现了具体方法，调用了 doCreateBean，doCreateBean 中调用 createBeanInstance 方法
   
   返回 Bean 的包装类 BeanWrapper，一共有三种策略：
   
   * 使用工厂方法创建，instantiateUsingFactoryMethod 。
   
   * 使用有参构造函数创建，autowireConstructor。
   
   * 使用无参构造函数创建，instantiateBean。
   
   看了三个方法的代码，我们发现这三个实例化方式，最后都会走 getInstantiationStrategy().instantiate
   
   虽然拿到了构造函数，并没有立即实例化。因为用户使用了 replace 和 lookup 的配置方法，用到了动态代理加入对应的逻辑。如果没有的话，直接使用反射来创建实例。
   
   创建实例后，就可以开始注入属性和初始化等操作。
   
   但这里的 Bean 还不是最终的 Bean。返回给调用方使用时，如果是 FactoryBean 的话需要使用 getObject 方法来创建实例
   
   
## 注入属性

  实例创建完后开始进行属性的注入，如果涉及到外部依赖的实例，会自动检索并关联到该当前实例。

  Ioc 思想体现出来了。正是有了这一步操作，Spring 降低了各个类之间的耦合 
  
     protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
          PropertyValues pvs = mbd.getPropertyValues();
          if (bw == null) {
              if (!((PropertyValues)pvs).isEmpty()) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
              }
          } else {
              boolean continueWithPropertyPopulation = true;
              if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
                  terator var6 = this.getBeanPostProcessors().iterator();
                  
                  // InstantiationAwareBeanPostProcessor 前处理
                  while(var6.hasNext()) {
                      BeanPostProcessor bp = (BeanPostProcessor)var6.next();
                      if (bp instanceof InstantiationAwareBeanPostProcessor) {
                          InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                          if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                              continueWithPropertyPopulation = false;
                              break;
                          }
                      }
                  }
              }
  
              if (continueWithPropertyPopulation) {
                  if (mbd.getResolvedAutowireMode() == 1 || mbd.getResolvedAutowireMode() == 2) {
                      MutablePropertyValues newPvs = new MutablePropertyValues((PropertyValues)pvs);
                      
                       // 根据名称注入
                      if (mbd.getResolvedAutowireMode() == 1) {
                          this.autowireByName(beanName, mbd, bw, newPvs);
                      }
  
                      // 根据类型注入
                      if (mbd.getResolvedAutowireMode() == 2) {
                          this.autowireByType(beanName, mbd, bw, newPvs);
                      }
  
                      pvs = newPvs;
                  }
  
                  boolean hasInstAwareBpps = this.hasInstantiationAwareBeanPostProcessors();
                  boolean needsDepCheck = mbd.getDependencyCheck() != 0;
                  if (hasInstAwareBpps || needsDepCheck) {
                      PropertyDescriptor[] filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                      if (hasInstAwareBpps) {
                          Iterator var9 = this.getBeanPostProcessors().iterator();
  
                          // InstantiationAwareBeanPostProcessor 后处理
                          while(var9.hasNext()) {
                              BeanPostProcessor bp = (BeanPostProcessor)var9.next();
                              if (bp instanceof InstantiationAwareBeanPostProcessor) {
                                  InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                                  pvs = ibp.postProcessPropertyValues((PropertyValues)pvs, filteredPds, bw.getWrappedInstance(), beanName);
                                  if (pvs == null) {
                                      return;
                                  }
                              }
                          }
                      }
  
                      if (needsDepCheck) {
                          this.checkDependencies(beanName, mbd, filteredPds, (PropertyValues)pvs);
                      }
                  }
  
                  this.applyPropertyValues(beanName, mbd, bw, (PropertyValues)pvs);
              }
          }
      }  
      
* 应用 InstantiationAwareBeanPostProcessor 处理器，在属性注入前后进行处理。假设我们使用了 @Autowire 注解，这里会调用到 AutowiredAnnotationBeanPostProcessor 来对依赖的实例进行检索和注入的，它是 InstantiationAwareBeanPostProcessor 的子类。

* 根据名称或者类型进行自动注入，存储结果到 PropertyValues 中。

* 应用 PropertyValues，填充到 BeanWrapper。这里在检索依赖实例的引用的时候，会递归调用  BeanFactory.getBean 来获得。

## 初始化

### 触发 Aware

如果我们的 Bean 需要容器的一些资源该怎么办？比如需要获取到 BeanFactory、ApplicationContext 等等。

Spring 提供了 Aware 系列接口来解决这个问题。比如有这样的 Aware：

* BeanFactoryAware，用来获取 BeanFactory。
* ApplicationContextAware，用来获取 ApplicationContext。
* ResourceLoaderAware，用来获取 ResourceLoaderAware。
* ServletContextAware，用来获取 ServletContext。
* Spring 在初始化阶段，如果判断 Bean 实现了这几个接口之一，就会往 Bean 中注入它关心的资源


    private void invokeAwareMethods(String beanName, Object bean) {
        if (bean instanceof Aware) {
            if (bean instanceof BeanNameAware) {
                ((BeanNameAware)bean).setBeanName(beanName);
            }

            if (bean instanceof BeanClassLoaderAware) {
                ((BeanClassLoaderAware)bean).setBeanClassLoader(this.getBeanClassLoader());
            }

            if (bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware)bean).setBeanFactory(this);
            }
        }
    }
    
### 触发 BeanPostProcessor

   在 Bean 的初始化前或者初始化后，我们如果需要进行一些增强操作怎么办？
   
   这些增强操作比如打日志、做校验、属性修改、耗时检测等等.Spring 框架提供了 BeanPostProcessor 来达成这个目标。
   比如我们使用注解 @Autowire 来声明依赖，就是使用 AutowiredAnnotationBeanPostProcessor 来实现依赖的查询和注入的

    public interface BeanPostProcessor {
        Object postProcessBeforeInitialization(Object var1, String var2) throws BeansException;
    
        Object postProcessAfterInitialization(Object var1, String var2) throws BeansException;
    }    
    
   在AbstractBeanFactory中我们会看到 beanPostProcessors ，这时spring中只要实现 BeanPostProcessor 接口的bean都会被注册到beanPostProcessors中，
   只要实现BeanPostProcessor加载的时候会被spring自动识别这些bean，自动注册。   
   
   然后在 Bean 实例化前后，Spring 会去调用我们已经注册的 beanPostProcessors 把处理器都执行一遍。
   
      public abstract class AbstractAutowireCapableBeanFactory ... {
              
          @Override
          public Object applyBeanPostProcessorsBeforeInitialization ... {
      
              Object result = existingBean;
              for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
                  result = beanProcessor.postProcessBeforeInitialization(result, beanName);
                  if (result == null) {
                      return result;
                  }
              }
              return result;
          }
      
          @Override
          public Object applyBeanPostProcessorsAfterInitialization ... {
      
              Object result = existingBean;
              for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
                  result = beanProcessor.postProcessAfterInitialization(result, beanName);
                  if (result == null) {
                      return result;
                  }
              }
              return result;
          }   ...
      }
   
   这里使用了责任链模式，Bean 会在处理器链中进行传递和处理。当我们调用 BeanFactory.getBean 的后，执行到 Bean 的初始化方法 AbstractAutowireCapableBeanFactory.initializeBean 会启动这些处理器。
   
      protected Object initializeBean ... {   
          ...
          wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
          ...
          // 触发自定义 init 方法
          invokeInitMethods(beanName, wrappedBean, mbd);
          ...
          wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
          ...
      }
      
### 触发自定义 init
   自定义初始化有两种方式可以选择：
   
   * 实现 InitializingBean。提供了一个很好的机会，在属性设置完成后再加入自己的初始化逻辑。
   * 定义 init 方法。自定义的初始化逻辑。
   
## 类型转换

Bean 已经加载完毕，属性也填充好了，初始化也完成了。

在返回给调用者之前，还留有一个机会对 Bean 实例进行类型的转换。见 AbstractBeanFactory.doGetBean ：   

     protected <T> T doGetBean ... {
         ...
         if (requiredType != null && bean != null && !requiredType.isInstance(bean)) {
             ...
             return getTypeConverter().convertIfNecessary(bean, requiredType);
             ...
         }
         return (T) bean;
     }
     
## 总结

抛开一些细节处理和扩展功能，一个 Bean 的创建过程无非是：

获取完整定义 -> 实例化 -> 依赖注入 -> 初始化 -> 类型转换。

作为一个完善的框架，Spring 需要考虑到各种可能性，还需要考虑到接入的扩展性。

所以有了复杂的循环依赖的解决，复杂的有参数构造器的匹配过程，有了 BeanPostProcessor 来对实例化或初始化的 Bean 进行扩展修改。

先有个整体设计的思维，再逐步击破针对这些特殊场景的设计，整个 Bean 加载流程迎刃而解。     