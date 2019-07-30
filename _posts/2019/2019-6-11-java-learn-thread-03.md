---
layout: post
title:  线程-thread类的API(方法)
categories: 线程
description: 主要对线程API的学习比较
keywords: 线程
---

 主要对线程API的学习比较，测试其功能作用和部分相似代码作用区分


## API讲解

  对API的讲解里面是个人的看法和代码实验
  
### isAlive方法
  
   isAlive()方法是判断当前线程是否处于活动状态的。什么是活动状态，就是线程已启动且尚未停止。线程处于正在运行或者准备运行的状态。
   示例代码：
   ```java
    public static void main(String [] agrs)
     {
        AliveThread thread = new AliveThread();
        System.out.println("begin="+thread.isAlive());
        thread.start();
        try {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
        }
        System.out.println("end="+thread.isAlive());
     }
    
    public class AliveThread extends Thread
    {
        @Override
        public void run()
        {
            System.out.println("run="+this.isAlive());
        }
    }
```
    结果为 ：
    begin=false
    run=true
    end=false
    开始和结束都是false说明线程已经运行结束。如果把 Thread.sleep(1000);代码删除掉 end有可能是true值，
    这个是因为线程启动是异步的。
    
 
### sleep，getId方法
    
   指当前时间内(毫秒)让线程休眠或者暂停
  ```java
      public static void main(String [] agrs)
         {
             SleepThread thread = new SleepThread();
             System.out.println("begin "+Thread.currentThread().getId()+"   "+System.currentTimeMillis());
             thread.start();
             System.out.println("end "+Thread.currentThread().getId()+"   "+System.currentTimeMillis());
         }
     }
     public class SleepThread extends Thread
     {
         @Override
         public void run()
         {
             try {
                 System.out.println("run begin"+Thread.currentThread().getId()+System.currentTimeMillis());
                 Thread.sleep(2000);
                 System.out.println("run end"+Thread.currentThread().getId()+System.currentTimeMillis());
             }catch (InterruptedException e)
             {
                 throw new RuntimeException(e);
             }
         }
     }
  ```
     运行结果：
     begin 1   1563963132994
     end 1   1563963132995
     run begin12   1563963132995
     run end12   1563963134995
     从中我们可以看到因为异步先打印了begin,end 然后打印了run方法中的run begin 两秒后才打印run end ，说明sleep起作用了，在打印的结果中id不同，因为一个是主线程id一个是当前线程id。
     

### interrupt，interrupted,IsInterrupted方法
   
   对interrupt，interrupted,IsInterrupted等方法的基本讲解
   
#### interrupt方法
   调用interrupt只是在当前线程打个标记，并不是真正的停止线程。
   ```java
       public static void main(String [] agrs)
        {
               Thread thread = new InterruptThread();
               thread.start();
               try {
                   Thread.sleep(200);
               }catch (InterruptedException e)
               {
               }
               thread.interrupt();
           }
       }
       class InterruptThread extends Thread
       {
           @Override
           public void run()
           {
               for (int i=0;i<100000;i++)
               {
                   System.out.println("currentNumber="+i);
               }
           }
       }
   ```
      运行结果：输出了所有的所有的结果，但是如果将interrupt改成stop会运行到一半剩余的没有打印出来，
      这说明stop是真正kill掉了线程，就好像for循环中的break。
       
       

#### interrupted,IsInterrupted方法
   
     interrupted 测试当前线程是否已经中断，清除状态标志
     IsInterrupted 测试线程是否已经中断，不清除状态标志
     
   ```java  
     public static void main(String [] agrs)
         {
             InterruptThread thread = new InterruptThread();
             thread.start();
             try {
                 Thread.sleep(200);
             }catch (InterruptedException e)
             {
               e.printStackTrace();
             }
             thread.interrupt();
             System.out.println("是否停止？1"+thread.interrupted());
             System.out.println("是否停止？2"+thread.interrupted());
         }
     }
     class InterruptThread extends Thread
     {
         @Override
         public void run()
         {
             for (int i=0;i<100000;i++)
             {
                 //System.out.println("currentNumber="+i);
             }
         }
     }
   ```
     结果是：
     是否停止？1 false
     是否停止？2 false
     想想，不应该啊，我调用了thread.interrupt()a,应该已经停止了，为什么会两个都是false呢，再想想哪里出了问题，思考中...
     是不是thread.interrupted()描述的当前线程是主线程才会出现这个结果，想到这里我们赶紧测试，在thread.interrupt()后面加上
     Thread.currentThread().interrupt();再次运行
     结果出来了:
     是否停止？1 true
     是否停止？2 false
     一个true一个false，这又是怎么回事，我们再看下解释。 测试当前线程是否已中断，线程中断状态由该方法清除，这样子就是说第一次是true，然后将状态清除了，第二次调用才是false，原来如此。
     
     我们将上面代码改成两个IsInterrupted()执行结果如下：
     是否停止？1true
     是否停止？2true
     说明IsInterrupted不会清除标记
     

### suspend,resume方法
   
   suspend是将线程暂停，resume是将线程回复运行 ,这两个方法现在已经被标记为废弃，为什么会被标记为废弃呢，我们来看下代码。
   
#### suspend和resume缺点--独占
   ```java
      public static void main(String [] agrs){
             try {
                 final SynchronizedObject synchronizedObject = new SynchronizedObject();
                 Thread thread1 = new Thread() {
                     @Override
                     public void run()
                     {
                         synchronizedObject.printString();
                     }
                 };
                 thread1.start();
                 thread1.setName("test");
                 Thread.sleep(1000);
                 Thread thread2 = new Thread() {
                     @Override
                     public void run()
                     {
                         System.out.println("我感觉我进不来了，被锁在外面了");
                         synchronizedObject.printString();
                     }
                 };
                 thread2.start();
             }catch (InterruptedException e)
             {
                 e.printStackTrace();
             }
         }
     }
     public class SynchronizedObject
     {
         synchronized public void printString() {
             if(Thread.currentThread().getName().equals("test"))
             {
                 System.out.println("只有执行到test才进来");
                 Thread.currentThread().suspend();
             }
         }
     }
   ```  
     执行结果：
     只有执行到test才进来
     我感觉我进不来了，被锁在外面了
     看执行结果知道，当调用suspend后锁一直被占用不能释放，所以线程2进不来了。还有一种写法是和println结合的，
     因为println内部用的是同步锁，当调用suspend后，锁没被释放，println一直打印不出来。
     

#### suspend和resume缺点--不同步
  ```java 
    public static  void main(String[] args)
       {
           try {
               final MyObject myObject = new MyObject();
               Thread thread1 = new Thread() {
                   @Override
                   public void run()
                   {
                       myObject.setValue("zhaihaixiang","123456");
                   }
               };
               thread1.start();
               thread1.setName("test");
               Thread.sleep(1000);
               Thread thread2 = new Thread() {
                   @Override
                   public void run()
                   {
                       myObject.printInfo();
                   }
               };
               thread2.start();
           }catch (InterruptedException e)
           {
               e.printStackTrace();
           }
       }
       class MyObject
       {
           private String name ="1";
           private String password ="11";
           public void setValue(String name ,String password) {
               this.name = name;
               if(Thread.currentThread().getName().equals("test"))
               {
                   System.out.println("停止线程a");
                   Thread.currentThread().suspend();
               }
               this.password = password;
           }
           public void printInfo()
           {
               System.out.println("name="+name+"\npassword="+password);
           }
       }
   ```    
       停止线程a
       name=zhaihaixiang
       password=11
       看结果password不是最新的值，看第一个独占的问题就能看出为什么password值没被改变了。
       从这两个坑中我们能猜到为啥这两个方法被废弃的原因。
   

### yield方法
   
  yield方法是让cpu空闲一段时间让其他线程去执行任务，过段时间再次执行
  
  ```java
    public static  void main(String[] args)
       {
           YieldThread yieldThread = new YieldThread();
           yieldThread.start();
       }
    }
    public class YieldThread extends Thread
    {
       @Override
       public void run() {
           Long beginTime = System.currentTimeMillis();
           int count =0;
           for (int i=0;i<50000000;i++)
           {
               count += i;
               //Thread.yield();
           }
           Long endTime = System.currentTimeMillis();
           System.out.println("总耗时"+(endTime-beginTime)+"毫秒");
       }
    }
  ```  
    运行结果:
    总耗时17毫秒
    将Thread.yield();方法放开再次执行 
    总耗时2980毫秒
    时间变长说明yield方法起作用了，他不规律的将cpu给了别的任务跑。
    

### 线程优先级方法(setPriority)
  
  设置线程优先级，可以帮助“线程规划器”确定在下一次执行选择哪个线程。线程的优先级分为1-10,10个等级，如果小于1或者大于10则抛出异常。
  
#### 线程优先级的继承性
  
  ```java
      public static void main(String [] agrs)
       {
              System.out.println("main priority="+Thread.currentThread().getPriority());
              //Thread.currentThread().setPriority(6);
              System.out.println("main priority="+Thread.currentThread().getPriority());
              PriorityThread1 thread = new PriorityThread1();
              thread.start();
       }
      public class PriorityThread1 extends Thread {
          @Override
          public void run() {
              System.out.println("Thread1 count priority =" +this.getPriority());
              PriorityThread2 thread2 = new PriorityThread2();
              thread2.start();
          }
      }
      public class PriorityThread2 extends Thread {
          @Override
          public void run() {
              System.out.println("Thread2 count priority =" +this.getPriority());
          }
      }
  ```    
      看如上代码，运行结果：
      main priority=5
      main priority=5
      Thread1 count priority =5
      Thread2 count priority =5
      是jdk中默认是值5，如果我们把 Thread.currentThread().setPriority(6)前的双斜杠去掉，运行结果如下：
      main priority=5
      main priority=6
      Thread1 count priority =6
      Thread2 count priority =6
      从结果中我们能看出，当主线程的优先级被设置成6，其子线程都被改成了6，这个就说明优先级的继承性。
      

#### 线程优先级具有规则性
  
     高优先级的线程大多数被优先执行完，不是说高优先级的线程就一定先执行完。设置优先级别且相差较大的情况下，不是先调用哪个线程，
     哪个线程就被优先执行。
     

#### 线程优先级具有随机性
  
     上面说优先级较高的优先执行，这个说法不是完全的，因为线程还有随机性，因为优先级较高的线程不一定每次都是先执行完。