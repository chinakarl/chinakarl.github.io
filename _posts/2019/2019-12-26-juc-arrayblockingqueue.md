---
layout: post
title:  ArrayBlockingQueue 详解
categories: jdk
description: ArrayBlockingQueue 详解
keywords: jdk，juc
---

   本章是对ArrayBlockingQueue的理解与分析
  
## ArrayBlockingQueue 介绍
  
  ArrayBlockingQueue 根据名字我们可以看出，它是个基于数组的阻塞队列。 并且是有界的。
  有界是只对应的数组是有界的。阻塞是指当多个线程访问资源时，该资源被其中一个线程访问并使用时，其它线程需要阻塞等待。
  ArrayBlockingQueue 是根据FIFO（先进先出）原则对元素进行排序的
  
## 常用的方法

    // 将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则抛出 IllegalStateException。
    boolean add(E e)
    // 自动移除此队列中的所有元素。
    void clear()
    // 将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则返回 false。
    boolean offer(E e)
    // 将指定的元素插入此队列的尾部，如果该队列已满，则在到达指定的等待时间之前等待可用的空间。
    boolean offer(E e, long timeout, TimeUnit unit)
    // 获取但不移除此队列的头；如果此队列为空，则返回 null。
    E peek()
    // 获取并移除此队列的头，如果此队列为空，则返回 null。
    E poll()
    // 获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
    E poll(long timeout, TimeUnit unit)
    // 将指定的元素插入此队列的尾部，如果该队列已满，则等待可用的空间。
    void put(E e)
    // 从此队列中移除指定元素的单个实例（如果存在）。
    boolean remove(Object o)
    // 返回此队列中元素的数量。
    int size()
    // 获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
    E take()
    // 返回在无阻塞的理想情况下（不存在内存或资源约束）此队列能接受的其他元素数量。
    int remainingCapacity()
    
## ArrayBlockingQueue 源码介绍(基于jdk1.8.0_162)

### 构建方法

    public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }
    
    public ArrayBlockingQueue(int capacity, boolean fair) {
            if (capacity <= 0)
                throw new IllegalArgumentException();
            this.items = new Object[capacity];
            lock = new ReentrantLock(fair);
            notEmpty = lock.newCondition();
            notFull =  lock.newCondition();
        }
        
* items是保存“阻塞队列”数据的数组。它的定义如下：
  
  final Object[] items
  
* fair是“可重入的独占锁(ReentrantLock)”的类型。fair为true，表示是公平锁；fair为false，表示是非公平锁  
   