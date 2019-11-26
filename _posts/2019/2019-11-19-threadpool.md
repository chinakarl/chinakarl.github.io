---
layout: post
title:  线程池
categories: java
description: 线程池
keywords: 线程池
---

   对4种线程池的简单学习

# 几种线程的简单使用
  
## newCachedThreadPool线程池
  
       //当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，因此会在池中寻找可用线程来执行，若有可以线程则执行，若没有可用线程则创建一个线程来执行该任务；若池中线程空闲时间超过指定大小，则该线程会被销毁。
        //执行很多短期异步的小程序或者负载较轻的服务器
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i=1;i<=10;i++)
        {
            final int index = i;
            try {
                Thread.sleep(index*1000);
            }catch (InterruptedException e)
            {
                e.printStackTrace();
            }
            executorService.execute(new Runnable() {
                public void run() {
                    System.out.println(index);
                }
            });
        }
## newFixedThreadPool线程池

        //接收参数为所设定线程数量nThread，corePoolSize为nThread，maximumPoolSize为nThread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：new LinkedBlockingQueue<Runnable>() 无解阻塞队列
        //创建可容纳固定数量线程的池子，每隔线程的存活时间是无限的，当池子满了就不在添加线程了；如果池中的所有线程均在繁忙状态，对于新任务会进入阻塞队列中(无界的阻塞队列)
        //执行长期的任务，性能好很多 线程数量劲量根据系统资源进行设置(如Runtime.getRuntime().availableProcessors())
        ExecutorService fixedExecutorService = Executors.newFixedThreadPool(3);
        for (int i=1;i<=10;i++)
        {
            final int index=i;
            fixedExecutorService.execute(new Runnable() {
            public void run() {
                    try {
                        System.out.println(index);
                        Thread.sleep(2000);
                    }catch (InterruptedException e)
                    {

                    }
                }
            });
         }
## newSingleThreadExecutor线程池

        //corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：new LinkedBlockingQueue<Runnable>() 无解阻塞队列
        //创建一个只有一个线程的线程池，如果线程繁忙，等待线程则进入阻塞模式。
        ExecutorService singleExecutorService = Executors.newSingleThreadExecutor();
        singleExecutorService.execute(new Runnable() {
            public void run() {
                System.out.println("single demo");
            }
        });
        
## newScheduledThreadPool线程池
        
        //创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列
        //创建一个固定大小的线程池，线程池内线程存活时间无限制，线程池可以支持定时及周期性任务执行，如果所有线程均处于繁忙状态，对于新任务会进入DelayedWorkQueue队列中，这是一种按照超时时间排序的队列结构
        //适用：周期性执行任务的场景
        ScheduledExecutorService scheduleExecutorService = Executors.newScheduledThreadPool(5);
        scheduleExecutorService.schedule(new Runnable() {
            public void run() {
                System.out.println("test");
            }
        },5,TimeUnit.SECONDS);

        //表示延迟1秒，每3秒执行一次
        scheduleExecutorService.scheduleAtFixedRate(new Runnable() {
            public void run() {
                System.out.println("delay 1 second");
            }
        },1,3,TimeUnit.SECONDS);