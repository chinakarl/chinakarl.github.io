---
layout: post
title:  线程-并发访问-01
categories: 线程
description: 主要正对并发访问时同步关键字synchronized的讲解
keywords: 线程,synchronized
---

 主要正对并发访问时同步关键字synchronized的讲解


## synchronized同步方法

  在我们开发中经常会提到"线程安全"和"非线程安全"，线程安全就是一个变量或者其它在多线程情况下，不会因为其它线程改变值，非线程安全
  反之。那么非线程安全我们怎么去解决呢，今天我们就讲解下解决方法之一 synchronized 方法。
  
### 方法体里面的变量是线程安全的
  
   "非线程安全"的问题存在于实例变量中，而方法中的变量是不会存在"非线程安全"问题，也就是"线程安全的"，为什么线程安全呢，是因为方法
   中的变量访问修饰符的问题，其是私有的，其它线程是无法改变其值的。
 

### 实例变量非线程安全
    
   指当前时间内(毫秒)让线程休眠或者暂停
  ```java
       public static void main(String [] args)
          {
              Test test = new Test();
              ThreadA threadA = new ThreadA(test);
              threadA.start();
              ThreadB threadB = new ThreadB(test);
              threadB.start();
          }
      public class  ThreadA extends  Thread
      {
          private Test test;
          public ThreadA(Test test)
          {
              super();
              this.test =test;
          }
          @Override
          public void run()
          {
              super.run();
              test.add("a");
          }
      }
     public class  ThreadB extends  Thread
      {
          private Test test;
          public ThreadB(Test test)
          {
              super();
              this.test =test;
          }
          @Override
          public void run()
          {
              super.run();
              test.add("b");
          }
      }
      public class Test
      {
          private int num = 0;
          public void add(String name)
          {
              try {
                  if(name.equals("a"))
                  {
                      num = 10;
                      System.out.println("a print over!");
                      Thread.sleep(2000);
                  }
                  else
                  {
                      num = 20;
                      System.out.println("b print over!");
                  }
                  System.out.println(name+" num = "+num);
              }catch (InterruptedException e)
              {
                  e.printStackTrace();
              }
          }
      }
  ```
     运行结果：
     a print over!
     b print over!
     b num = 20
     a num = 20
     从中我们可以看到线程a num的值被b的值覆盖了。
     我们将synchronized加在add方法上，运行结果：
     a print over!
     a num = 10
     b print over!
     b num = 20
     看上面的运行结果，我们看到前后顺序不一致，那是因为加了synchronized关键字的问题。
     没加前线程A,B 被调用同时进来，所以会出现同时打赢over消息，但是因为判断了线程a线程睡眠2秒所以会先打印B的num然后因为num被B改了，所以A的num也是20.
     加了synchronized后线程A,B同时执行，但是当线程A进来后需要执行完所有才会释放锁让线程B执行，所以就出现了顺序执行的结果。

### 多个对象多个锁
   我们将上面的main函数改成如下
   ```java
    public static void main(String [] args)
         {
             Test testA = new Test();
             Test testB = new Test();
             ThreadA threadA = new ThreadA(testA);
             threadA.start();
             ThreadB threadB = new ThreadB(testB);
             threadB.start();
         }
   ```
     运行结果：
     a print over!
     b print over!
     b num = 20
     a num = 10
     因为加了锁所以数据没有串了，但是运行顺序是会变的，为什么呢，这个是因为synchronized是对象锁，不是针对方法的锁，第一次我们创建了
     一个对象，所以就一个锁，上面代码我们创建了两个对象，JVM会创建两个锁。在一个对象中，有多个方法，多个线程可以异步调用加锁的方法和
     无锁的方法，比如 A线程调用对象ObjectA中带锁方法 a的同时B线程可以直接调用对象ObjectA中无锁方法b
   
#### 脏读
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
     

### 脏读
   
   脏读，就是在多线程下，一个线程读取的数据被另外一个线程更改。
   ```java
    public static void main(String [] args)
       {
           try {
               DirtyReadTest dirtyReadTest = new DirtyReadTest();
               DirtyReadThreadA dirtyReadThreadA = new DirtyReadThreadA(dirtyReadTest);
               dirtyReadThreadA.setName("a");
               dirtyReadThreadA.start();
               Thread.sleep(1000);
               dirtyReadTest.getValue();
           }catch (InterruptedException e)
           {
               e.printStackTrace();
           }
       }
   public class  DirtyReadThreadA extends  Thread
   {
       private DirtyReadTest test;
       public DirtyReadThreadA(DirtyReadTest test)
       {
           super();
           this.test =test;
       }
       @Override
       public void run()
       {
           super.run();
           test.setValue("Azhai","A123");
       }
   }
 public class DirtyReadTest
   {
       private String name = "zhai";
       private String password = "123";
       synchronized public void setValue(String name ,String password)
       {
           try {
               this.name =name;
               Thread.sleep(2000);
               this.password = password;
               System.out.println("内部 name ="+this.name+" password ="+this.password);
           }
           catch (InterruptedException e)
           {
               e.printStackTrace();
           }
   
       }
       public void getValue()
       {
           System.out.println("name ="+this.name+" password ="+this.password);
       }
   }
   ```
    运行结果：
    name =Azhai password =123
    内部 name =Azhai password =A123
    在get的时候获取到的密码是原来定义的，这个就是getValue方法没有同步导致的，解决办法只要在getValue方法钱加上synchronized就可以了。
    运行结果：
    内部 name =Azhai password =A123
    name =Azhai password =A123
    看正确了吧
    
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
   

### synchronized锁重入
   
  synchronized具有锁重入的功能，也就是一个对象得到锁后，在没有释放前，另外一个对象还可以得到该对象的锁，如果没有锁重入的功能的话，
  会造成死锁。这个功能在父子类继承关系中也有用。
    

### 出现异常锁自动释放
  
  当出现异常的时候，锁会自动释放。
  ```java
    public static void main(String [] args)
      {
          try {
              Service service = new Service();
              ExceptionThreadA exceptionThreadA=new ExceptionThreadA(service);
              exceptionThreadA.setName("a");
              exceptionThreadA.start();
              Thread.sleep(500);
              ExceptionThreadA exceptionThreadB=new ExceptionThreadA(service);
              exceptionThreadB.setName("b");
              exceptionThreadB.start();
          }catch (InterruptedException e)
          {
              e.printStackTrace();
          }
      }
  public class ExceptionThreadA extends  Thread
  {
      Service service =null;
     public ExceptionThreadA(Service service)
     {
         super();
         this.service = service;
     }
    @Override
    public  void run()
    {
        service.testMethod();
    }
  }
  public class ExceptionThreadB extends  Thread
  {
      Service service =null;
      public ExceptionThreadB(Service service)
      {
          super();
          this.service = service;
      }
      @Override
      public  void run()
      {
          service.testMethod();
      }
  }
  public class Service
  {
      synchronized   public void testMethod()
      {
          if(Thread.currentThread().getName().equals("a"))
          {
              System.out.println("threadName ="+Thread.currentThread().getName()+" run begin time "+System.currentTimeMillis());
              int i =1;
              while (i==1)
              {
                   System.out.println("threadName ="+Thread.currentThread().getName()+" run expetion time "+System.currentTimeMillis());
                   Integer.parseInt("a");
              }
          }
          else
          {
              System.out.println("threadName ="+Thread.currentThread().getName()+" threadB run time "+System.currentTimeMillis());
          }
      }
  }
  ```
    运行结果：
    threadName =a run begin time 1564472560852
    threadName =a run expetion time 1564472560852
    Exception in thread "a" java.lang.NumberFormatException: For input string: "a"
    	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    	at java.lang.Integer.parseInt(Integer.java:580)
    	at java.lang.Integer.parseInt(Integer.java:615)
    	at jdksoundcode.thread.concurrence.Service.testMethod(ExceptionTest.java:67)
    	at jdksoundcode.thread.concurrence.ExceptionThreadA.run(ExceptionTest.java:39)
    threadName =b threadB run time 1564472561352
    当线程a执行转换出现异常并抛出后，线程b跟着也执行了，这样子就说明a执行抛出异常后释放了锁，b得到锁后再次执行。

### 同步不具有继承性
   就是说同步语句，在其子类中不起到作用。比如线程ChildrenA继承A，A中方法a添加了synchronized并被其子类ChildrenA继承重写，但是重写
   的a方法不是同步的。这个其实在上面提到的锁是针对对象的，子类相对于父类来说就是一个单独的对象，所以是无法被继承的，如果想子类方
   法也同步，那么就需要在子类重写的方法前加上synchronized
