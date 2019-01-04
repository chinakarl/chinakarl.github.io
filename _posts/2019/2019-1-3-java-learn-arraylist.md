---
layout: post
title:  arraylist学习
categories: spring
description: 对arraylist的基本学习使用和源码解析
keywords: java
---

 对arraylist的基本学习使用和源码解析，并且记录下自己想到或者看见却不理解的，进行重点记录


## arraylist基本实现方式和结构图

   1. 先看下结构图

    ![INNER JOIN](https://chinakarl.github.io/images/posts/java/arraylist-struct.png)

    从上图中我们可以看到
    1. 继承了 AbstractList<E>
       
    2. 实现了 List<E>, RandomAccess, Cloneable, java.io.Serializable
       
       1. List不用解释
       2. RandomAccess 说明可以随机获取
       3. Cloneable 说明可以克隆(Clone)
       4. 说明可以被序列化

   3. 源码基本解析
     
    3.1 默认常量
    private static final long serialVersionUID = 8683452581122892189L;
    // 默认容量
    private static final int DEFAULT_CAPACITY = 10;
    // 当ArrayList的构造方法中显示指出ArrayList的数组长度为0时，类内部将EMPTY_ELEMENTDATA 这个空对象数组赋给elemetData数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 当ArrayList的构造方法中没有显示指出ArrayList的数组长度时，类内部使用默认缺省时对象数组为DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 保存ArrayList中数据的数组,声明为transient说明是不可以被序列化的
    transient Object[] elementData;
    // 实际ArrayList中存放的元素的个数，默认时为0个元素
    private int size;
    // 最大数组容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
   

   