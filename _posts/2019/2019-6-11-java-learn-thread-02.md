---
layout: post
title:  线程学习02
categories: 线程
description: 实现多线程的方式和start与run的区别
keywords: 线程
---

  讲解实现多线程的方式和案例，并且通过代码作为案例讲解start与run的区别

### 实现方式
   实现多线程的方式有3中
   
   01.继承Thread
    
    public static void main(String [] args)
    {
            Thread thread1 = new ThreadTest();
            Thread thread2 = new ThreadTest();
            Thread thread3 = new ThreadTest();
            thread1.start();
            thread2.start();
            thread3.start();
    }
     class ThreadTest extends Thread
     {
         private int ticket=10;
         public void run()
         {
             for (int i=0;i<10;i++)
             {
             if(this.ticket>0){
                 System.out.println(this.getName()+" 卖票：ticket"+this.ticket--);
             }
             }
         }
     }
   02.实现Runnable
   
       public static void main(String [] args)
        {
            RunnableTest Runnable1 = new RunnableTest();
            Thread thread6 = new Thread(Runnable1);
            Thread thread4 = new Thread(Runnable1);
            Thread thread5 = new Thread(Runnable1);
            thread6.start();
            thread4.start();
            thread5.start();
       }
       class RunnableTest implements Runnable
       {
           private int ticket=10;
           public void run()
           {
               for (int i=0;i<10;i++)
               {
                   if(this.ticket>0){
                       System.out.println(Thread.currentThread().getName()+" 卖票：ticket"+this.ticket--);
                   }
               }
           }
       }
   
   03.线程池
   
    ExecutorService executorService = Executors.newCachedThreadPool();
       for (int i = 0; i < 10; i++) {
           executorService.execute(new Runnable() {
               public void run() {
                   System.out.println("threadName:"+Thread.currentThread().getName());
               }
           });
           try {
               Thread.sleep(1);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
    
  上面例子中我们看到了实现多线程三种方式的简单例子，我们看1,2执行后结果一样，那为什么jdk中却还是区分这两种方式呢，带着这个疑问我们看看下面说的是什么
  
   Runnable 是接口 Thread是实现类，这就说明Runnable扩展性更好，而Thread本身就实现了Runnable。
   此外，Runnable还可以用于“资源的共享”。即，多个线程都是基于某一个Runnable对象建立的，它们会共享Runnable对象上的资源。通常，建议通过“Runnable”实现多线程！

### start与run区别

 先不废话，撸下代码
 
     public static void main(String [] args)
     {
          Thread mythread=new MyThread("mythread");
              System.out.println(Thread.currentThread().getName()+" call mythread.run()");
              mythread.run();
     
              System.out.println(Thread.currentThread().getName()+" call mythread.start()");
              mythread.start();
     }
     class MyThread extends Thread
     {
         public MyThread(String name)
         {
             super(name);
         }
         public void run()
         {
             System.out.println(Thread.currentThread().getName()+" is running");
         }
    }
    运行结果
    main call mythread.run()
    main is running
    main call mythread.start()
    mythread is running
   
  从运行结果中是否发现，前3个都是mian线程最后一个才是我们自己定义的线程，这是为什么呢？
  因为在start的时候会创建一个新的线程去运行，而run的时候不需要，看源码
  
    public synchronized void start() {
     
      if (threadStatus != 0)
          throw new IllegalThreadStateException();

      group.add(this);

      boolean started = false;
      try {
          start0();
          started = true;
      } finally {
          try {
              if (!started) {
                  group.threadStartFailed(this);
              }
          } catch (Throwable ignore) {
              /* do nothing. If start0 threw a Throwable then
                it will be passed up the call stack */
          }
      }
    }
  
    public void run() {
            if (target != null) {
                target.run();
            }
        }
        
  从上面源码中我们可以分析到
  1.start方法中调用了一个start0方法，该方法调用了一个新的线程去启动。
  run方法中是直接执行target的run方法，而target是Runnable对象，所以是直接执行本线程的。
  
  2.start方法钱加上了 synchronized，说明被同步锁了，那么我们就可以分析出，如果我们第一次调用start，但是start0启动线程后却没执行run方法呢，
  也就是还没结束呢后面再调用一个start，这时候start已经被锁住，第二个start会无效报异常。而run方法没有。
  这样子我们就能推断出，run可以被多次调用，而start只能调用一次。      