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
    1.1. 继承了 AbstractList<E>
       
    1.2. 实现了 List<E>, RandomAccess, Cloneable, java.io.Serializable
       
       1. List不用解释
       2. RandomAccess 说明可以随机获取
       3. Cloneable 说明可以克隆(Clone)
       4. Serializable说明可以被序列化
     和Vector不同，ArrayList的操作不是线程安全的  

  2. 源码基本解析
   
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
  3 看源码分析ArrayList线程不安全
  
    源码中有这么一段代码，是新增元素的
    public boolean add(E e) {
        // 调用方法
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }
    private void ensureCapacityInternal(int minCapacity) {
            ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    private void ensureExplicitCapacity(int minCapacity) {
            modCount++;
            if (minCapacity - elementData.length > 0)
                grow(minCapacity);
      }
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
            // 如果当前的数值数据和默认缺损对象一样，将默认容量DEFAULT_CAPACITY扩大到当前size+1
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
           }
          return minCapacity;
    }
    private void grow(int minCapacity) {
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1); //oldCapacity >> 1 移位1相当于除以2
            if (newCapacity - minCapacity < 0)
                newCapacity = minCapacity;
            if (newCapacity - MAX_ARRAY_SIZE > 0)
                newCapacity = hugeCapacity(minCapacity);
            elementData = Arrays.copyOf(elementData, newCapacity);//将新的数组拷贝到老的中
     }
    逻辑如下
    
    1.覆盖
     当前数组大小10
     线程A进入size为10，调用ensureCapacityInternal(size + 1)，发现elementData为10可以添加
     线程B进入size也为10，调用ensureCapacityInternal(size + 1)，发现elementData为10可以添加
     线程A进入grow方法进行扩容10下标的
     线程B进入grow方法进行扩容10下标的,这时候B就会覆盖A的数据
     
    2.null
     线程A进入size为10，发现elementData为10可以添加
     线程B进入size也为1，发现elementData为10可以添加
     但是这时候A已经调用elementData[size++] = e，使得size=11，这时候B再来调用会报出ArrayIndexOutOfBoundsException数组越界的异常
     
     看如下代码
      public static  void main(String[] args) throws InterruptedException
        {
            final List<Integer> integers = new ArrayList<Integer>();
            new Thread(new Runnable() {
                public void run() {
                    for (int i=0;i<10;i++)
                    {
                        try {
                            integers.add(i);
                            Thread.sleep(1);
                        }catch (Exception e)
                        {
                            e.printStackTrace();
                        }
                    }
                }
            }).start();
            new Thread(new Runnable() {
                public void run() {
    
                    for (int i=10;i<20;i++)
                    {
                        try {
                            integers.add(i);
                            Thread.sleep(1);
                        }catch (Exception e)
                        {
                            e.printStackTrace();
                        }
                    }
                }
            }).start();
            Thread.sleep(1000);
            // 打印所有结果
            for (int i = 0; i < integers.size(); i++) {
                System.out.println("第" + (i + 1) + "个元素为：" + integers.get(i));
            }
        } 
   结果 1
   
     第2个元素为：1
     第3个元素为：10
     第4个元素为：11
     第5个元素为：2
     第6个元素为：3
     第7个元素为：12
     第8个元素为：4
     第9个元素为：14
     第10个元素为：5
     第11个元素为：6
     第12个元素为：15
     第13个元素为：7
     第14个元素为：16
     第15个元素为：8
     第16个元素为：17
     第17个元素为：18
     第18个元素为：9
     第19个元素为：19
     
    第8个元素将原有的13覆盖了  
   结果2
   
     第1个元素为：0
     第2个元素为：10
     第3个元素为：11
     第4个元素为：1
     第5个元素为：12
     第6个元素为：2
     第7个元素为：13
     第8个元素为：3
     第9个元素为：14
     第10个元素为：4
     第11个元素为：null
     第12个元素为：15
     第13个元素为：16
     第14个元素为：6
     第15个元素为：17
     第16个元素为：7
     第17个元素为：18
     第18个元素为：9
     第19个元素为：19
     
    出现了null，说明出现了数组越界
   
  3. 遍历方式
  
    ArrayList有3中遍历方式
     1.第一种，通过迭代器遍历。即通过Iterator去遍历。
     2.第二种RandomAccess遍历,索引值遍历(因为是有序有，每个元素都有一个对应的索引)
     3.第三种传统遍历(for循环遍历)
     
  看如下示例与运行结果   
![INNER JOIN](https://chinakarl.github.io/images/posts/java/arraylist-demo.png)
   iterator：6 ms 
   randomAccess：4 ms
   for：21 ms
   可知 根据索引值循环最快，for最慢(因为for是通过iterator再次封装的)

   