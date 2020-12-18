---
layout: post
title:  hashmap学习
categories: jdk
description: 对hashmap的学习
keywords: java
---

 对HashMap基础学习和部分原理剖析


# 前言

 HashMa是Java中最常用的集合类框架，也是Java语言中非常典型的数据结构，同时也是我们需要掌握的数据结构。

# HashMap 数据结构
  
  HashMap 底层数据结构数组和链表。
  
## 数组

* 存储区间是连续，且占用内存严重，空间复杂也很大，时间复杂为O（1）。
* 优点：是随机读取效率很高，原因数组是连续（随机访问性强，查找速度快）。
* 缺点：插入和删除数据效率低，因插入数据，这个位置后面的数据在内存中要往后移的，且大小固定不易动态扩展。

## 链表

* 区间离散，占用内存宽松，空间复杂度小，时间复杂度O(N)。
* 优点：插入删除速度快，内存利用率高，没有大小固定，扩展灵活。
* 缺点：不能随机查找，每次都是从第一个开始遍历（查询效率低）。

## Hash 表

 在上面看出数组和链表都有自己的缺点，对于这样的缺点，是否可以解决呢？答案是可以的，这就是Hash 表，他使用
数组和链表(jdk 1.8 开始使用红黑树)相互结合的思想解决了数组插入删除慢，链表查询慢的缺点。使得查询，插入和删除得以优化。

结构图如下：

![INNER JOIN](https://chinakarl.github.io/images/posts/java/hashmap/hash-struct.png)

# HashMap 实现和原理
 
## HashMap put,get 的实现

下面有一段代码
  
      HashMap<String,String> hashMap = new HashMap<>();
      hashMap.put("1","翟海翔");
      hashMap.put("2","翟小心");
      hashMap.put("3","王重阳");
      hashMap.put("4","小明");
      hashMap.put("5","小红");

### put 的原理
      
* put 步骤
1. 我们在put key=1,value=翟海翔 时候，会先根据 key 得到hashcode，得到的hashcode 用于确定将key存放到数组的什么位置。
2. 将得到的hashcode和value 存入到Node 节点中
3. 通过哈希表函数/哈希算法，将hash值转换成数组的下标，下标位置上如果没有任何元素，就把Node添加到这个位置上。如果说下标对应的位置上有链表。此时，就会拿着k和链表上每个节点的k进行equal。如果所有的equals方法返回都是false，那么这个新的节点将被添加到链表的末尾。如其中有一个equals返回了true，那么这个节点的value将会被覆盖

* put 过程

当put key=1，value=翟海翔，这时候查出数组长度为16， hashcode = 0， 0%16 = 0，存入数组下标0位子。

入下图

![INNER JOIN](https://chinakarl.github.io/images/posts/java/hashmap/hashmap-put1.png)

之后put，key=2 value=翟小心，key=3 value=王重阳 的，如下图：

![INNER JOIN](https://chinakarl.github.io/images/posts/java/hashmap/hashmap-put2.png)

这时候又put了，key=4 value=小明，key=5 value=小红，这时候发现 hashcode%16 都为0，就会在数组中下标为0的进行对比，
开始有值为翟海翔，这时候就用小明的key equals 翟海翔 的 key，发现 返回false，数组下添加链表元素为 小明的。
小红进来 equals 翟海翔和小明的key发现都返回 false ，则在链表末尾添加新元素。如果为true，则覆盖当前元素。 

链表用的好好的，为什么1.8之后转换成了红黑树呢？
在上面也说了，链表查询比较慢，这就会出现链表数据大的时候查询效率降低，所以就添加了红黑树。当链表格式超过8的时候，会转变成红黑树。


* 注意

1. 在得到hashcode的时候我们会发现用的是Object的 public native int hashCode() 方法。（native 是调用本地方法栈，也就是c或者c++等）。
2. 哈希码特点
  
   对于同一个对象如果没有被修改（使用equals比较返回true）那么无论何时它的hashcode值都是相同的
   
   对于两个对象如果他们的equals返回false,那么他们的hashcode值也有可能相等

### get 的原理

* get 过程
1. 先根据key 获取 hashcode，根据hashcode 获取 数组下标
2. 先根据数组的下标和数组中元素对比，如果数组中没有元素则返回null。如果不同且数组中有单向链表则继续用key和链表中的key进行equals。
   如果返回true则返回当前元素，false则继续对比，直到比完。如果没有返回true的值并且对比完，则返回null。

## hashmap 源码解读

 在看hashmap源码的时候，两个非常重要的元素一定要记得
 
1. 负载因子（DEFAULT_LOAD_FACTOR，默认值0.75）
2. 初始容量 DEFAULT_INITIAL_CAPACITY = 1<<4

   初始容量大小是创建时给数组分配的容量大小，默认值为16，用数组容量大小乘以加载因子得到一个值，一旦数组中存储的元素个数超过该值就会调用rehash方法将数组容量增加到原来的两倍，专业术语叫做扩容
  
注： 在做扩容的时候会生成一个新的数组，原来的所有数据需要重新计算哈希码值重新分配到新的数组，所以扩容的操作非常消耗性能.
   
### hashmap 的 resize
 
 当hashmap中的元素越来越多的时候，碰撞的几率也就越来越高（因为数组的长度是固定的），所以为了提高查询的效率，就要对hashmap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，所以这是一个通用的操作，很多人对它的性能表示过怀疑，不过想想我们的“均摊”原理，就释然了，而在hashmap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。
 
 那么hashmap什么时候进行扩容呢？
 
 当hashmap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，也就是说，默认情况下，数组大小为16，那么当hashmap中元素个数超过16*0.75=12的时候，就把数组的大小扩展为2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知hashmap中元素的个数，那么预设元素的个数能够有效的提高hashmap的性能。比如说，我们有1000个元素new HashMap(1000), 但是理论上来讲new HashMap(1024)更合适，不过上面annegu已经说过，即使是1000，hashmap也自动会将其设置为1024。 但是new HashMap(1024)还不是更合适的，因为0.75*1000 < 1000, 也就是说为了让0.75 * size > 1000, 我们必须这样new HashMap(2048)才最合适，既考虑了&的问题，也避免了resize的问题。

   