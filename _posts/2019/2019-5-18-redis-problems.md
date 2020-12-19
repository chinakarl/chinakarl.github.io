---
layout: post
title:  redis 问题
categories: redis
description: redis使用和面试中遇到的问题
keywords: redis
---

  我们在平时工作中，面试中都会用到redis。那么随之而来的就是许多问题，比如缓存穿透，雪崩 等...这篇就是记录
  这类问题的

## Redis的并发竞争问题如何解决
   
### 方案一
   
   利用redis自带的incr命令 [地址](http://doc.redisfans.com/string/incr.html)
   
### 方案二
   
   使用乐观锁的方式进行解决（成本较低，非阻塞，性能较高），用redis中的watch就可以解决了。
   
### 方案三
   
   利用redis的setnx实现内置的锁

### 方案四

   针对客户端来说，使用java的synchronized或lock对同一个key进行限制。
  
## Redis 缓存失效策略

   磁盘空间到临界值等时候，我们需要清理缓存，这时候就要选择清理哪些缓存了， 缓存失效策略有FIFO 、LRU、LFU三种，我们来看下三种的区别
   
### FIFO

   First In First Out，先进先出。判断被存储的时间，离目前最远的数据优先被淘汰。
   
### LRU

   Least Recently Used，最近最少使用。判断最近被使用的时间，目前最远的数据优先被淘汰。
   
### LFU

   Least Frequently Used，最不经常使用。在一段时间内，数据被使用次数最少的，优先被淘汰。
    
## 缓存穿透，击穿和雪崩的原因+解决办法

###  缓存穿透

  缓存对应的key的数据和数据源都不存在，请求此key时都会透过缓存请求到数据源，从而可能压垮数据源

#### 解决方案一

   布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力
   
#### 解决方案二

  如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟
  
### 缓存击穿

  key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题  
  
  **使用互斥锁(mutex key)**
  
  业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，
  而是使用setnx(是set if not exists的缩写，也叫 分布式锁)去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法

     public String get(key) {
           String value = redis.get(key);
           if (value == null) { //代表缓存值过期
               //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
           if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
                    value = db.get(key);
                           redis.set(key, value, expire_secs);
                           redis.del(key_mutex);
                   } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
                           sleep(50);
                           get(key);  //重试
                   }
               } else {
                   return value;      
               }
      }
      
### 缓存雪崩

   缓存雪崩和缓存击穿道理差不多，只是击穿是一个key，雪崩是大量的。缓存雪崩是在某个时刻所有的缓存失效。比如设置了1000个缓存，
   都是3分钟后失效，然后失效的时候，突然进来10000个请求，这时候就穿过了缓存，直接访问db，这回让db压力增大，甚至崩溃。
   
#### 方案一

   在请求db的时候使用队列或者在请求db的操作上加上锁，这样子可以缓解db的压力。但是当大批量用户请求时候，虽然db压力减少了，但是
   用户会等待很长时间或者超时，这样给用户体验非常的太好。所以真正生产中，很少用这种方式的。
   
#### 方案二

   在我们添加缓存数据的时候，给每个缓存的失效时间加上个随机数，这样子可以避免一个问题，大多数缓存不会在同一时间段内失效。这样子
   就算穿到了db也不会产生大批量穿，db压力就会减少。   
   
   
## Redis哨兵、复制、集群的设计原理，以及区别

  [地址](https://www.cnblogs.com/telwanggs/p/10856157.html)
  
### 业务逻辑时间超出分布式锁过期时间

Redisson，watch dog

[地址](https://blog.csdn.net/wutengfei_java/article/details/100699538)

## Redis 如何实现延迟队列

[地址](https://blog.csdn.net/why15732625998/article/details/104890079/)
   
## Redis数据淘汰策略

### 主动清理

   当前已用内存超过maxmemory限定时，触发主动清理策略。
   清理时会根据用户配置的maxmemory-policy来做适当的清理。
   主动清理策略主要有一下六种:

* volatile-lru :    从已设置过期时间的数据集(server.db[i].expires)中挑选最近最少使用 的数据淘汰。
* volatile-ttl :    从已设置过期时间的数据集(server.db[i].expires)中挑选将要过期的数 据淘汰。
* volatile-random : 从已设置过期时间的数据集(server.db[i].expires)中任意选择数据 淘汰。
* allkeys-lru :     从数据集(server.db[i].dict)中挑选最近最少使用的数据淘汰。
* allkeys-random :  从数据集(server.db[i].dict)中任意选择数据淘汰。
* no-enviction :    禁止驱逐数据。
