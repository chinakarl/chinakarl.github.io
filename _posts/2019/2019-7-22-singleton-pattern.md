---
layout: post
title:  单例设计模式
categories: 设计模式
description: 单例设计模式讲解和应用场景
keywords: 单例,设计模式
---

  本章主要讲解单例设计模式写法和应用场景

## 单例设计模式介绍

   单例设计模式，从字面上理解就是单个实例。在我们工作中应该会经常用到单例模式，他是我们接触和使用比较多的设计模式之一了。
   在spring中我们创建bean的时候配置会选择单例还是多例，这里面的单例就是本章说的单例。
   
   什么是单例，举个栗子古代正常一个朝代只有一个皇帝，不会出现一朝多帝的情况，如果我们项目中遇到这种场景皇帝的对象就可以使
   用单例来实现，因为从始到终有且只有一个皇帝出现这种情况我们就可以称为单例。

## 单例设计模式几种写法

   ### 懒汉式，线程不安全   
   
   **代码如下**

    public class Singleton {  
       private static Singleton instance;  
       private Singleton (){}    
       public static Singleton getInstance() {  
        if (instance == null) {  
           instance = new Singleton();  
        }  
        return instance;  
       }  
    } 
   上面代码多线程情况下，A线程进入判断为null 进来然后返回这种情况是正确的。当线程B和线程A同时进来后，都判断为null就会两个返
   回不同的实例。
   
   ### 懒汉式，线程安全
   
   **代码如下**
   
       public class Singleton {  
           private static Singleton instance;  
           private Singleton (){}  
           public static synchronized Singleton getInstance() {  
           if (instance == null) {  
               instance = new Singleton();  
           }  
           return instance;  
           }  
       }  
   上面代码不管是否多线程都是线程安全的，因为我们加了synchronized关键字。虽然线程安全了，但是synchronized关键字在并发大多线程的
   情况下会影响效率，所以此写法用的比较少。
   
   ### 饿汉式  
   
   **代码如下**
   
       public class Singleton {  
           private static Singleton instance = new Singleton();  
           private Singleton (){}  
           public static Singleton getInstance() {  
            return instance;  
           }  
       }  
   上面代码通过静态方法和属性使得线程安全同时也实现了我们所需，而且效率在多线程高并发下也不会受影响。
   
   ### 双检锁/双重校验锁（DCL，即 double-checked locking）
   
   **代码如下**
   
       public class Singleton {  
           private volatile static Singleton singleton;  
           private Singleton (){}  
           public static Singleton getSingleton() {  
           if (singleton == null) {  
               synchronized (Singleton.class) {  
                   if (singleton == null) {  
                       singleton = new Singleton();  
                   }  
               }  
           }  
           return singleton;  
           }  
       } 
   代码中我们看到两次判断singleton==null，还用到了synchronized 关键字代码实现比较复杂，所以不推荐使用。
   
   ### 登记式/静态内部类
   
        public class Singleton {  
           private static class SingletonHolder {  
           private static final Singleton INSTANCE = new Singleton();  
           }  
           private Singleton (){}  
           public static final Singleton getInstance() {  
           return SingletonHolder.INSTANCE;  
           }  
        }
   这种方式比双重锁的简单，是使用静态内部类和静态方法结合的方式。
   
   ### 枚举
   
       public enum Singleton {  
           INSTANCE;  
           public void whateverMethod() {  
           }  
       }       
   这种方式没有被广泛使用，是 Effective Java 作者 Josh Bloch 提出的方式，这种方式自动支持序列化，而且写起来简单。