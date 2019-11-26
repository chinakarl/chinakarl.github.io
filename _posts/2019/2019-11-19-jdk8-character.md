---
layout: post
title:  java8 新特性
categories: java
description: java8 新特性
keywords: java8 新特性
---

   java8十大新特性的详细讲解

## 接口的默认方法

Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可

    interface Formula {
        double calculate(int a);
        default double sqrt(int a) {
          return Math.sqrt(a);
        }
     }
     
     Formula formula = new Formula() {
         @Override
         public double calculate(int a) {
             return sqrt(a * 100);
         }
     };
     
     formula.calculate(100);     // 100.0
     formula.sqrt(16);           // 4.0
     
  从上面代码中我们可以看到，默认方可以被直接调用。
       
## Lambda 表达式

   我们先看下，老的版本中是如何排序的：

     List<String> names = Arrays.asList("zhaihaixiang", "chenle", "wangjin", "ren");
     
     Collections.sort(names, new Comparator<String>() {
                 @Override
                 public int compare(String o1, String o2) {
                     return o1.compareTo(o2);
                 }
             });
             
   从中我们可以看到，只需要给静态方法 Collections.sort 传入一个List对象以及一个比较器来按指定顺序排列。
   通常做法都是创建一个匿名的比较器对象然后将其传递给sort方法
   
   我们来看看java8中如何排序
   
     Collections.sort(names,(String a,String b)->a.compareTo(b));
   
   看是不是很简单，省去了很多代码。其实还可以更简单，写成
   
     Collections.sort(names,(a,b)->a.compareTo(b)); 
   
   stream 流的方式
   
      List<Integer> numbers = Arrays.asList(10,12,32,5);
       // 取大于5的数字
      numbers.stream().filter(n->n>5).collect(Collectors.toList())
       // 排序后取前2条数据
       numbers.stream().sorted().limit(2).collect(Collectors.toList())
       // 求和
      numbers.stream().mapToInt(Integer::byteValue).sum();
       // 转成其它类型
       Set<Integer> setNumber = numbers.stream().collect(Collectors.toSet());
       // 根据奇偶数分组
       Map<Boolean,List<Integer>> mapNumbers = numbers.stream().collect(Collectors.groupingBy(a->a%2==0)); 
       
## 函数式接口         

      我们可以将lambda表达式当作任意只包含一个抽象方法的接口类型，确保你的接口一定达到这个要求，你只需要给你的接口添加 @FunctionalInterface 注解，
      编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错的    
      
## 方法与构造函数引用

先定义个Person类

    public class Person {
        String firstName;
        String lastName;
    
        Person() {}
    
        Person(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
    
        public static String doSomething()
        {
           return "---doSomething---";
        }
    }
可以调用静态方法

    Supplier<String> doSomething = (Person::doSomething);
    
再定义个PersonFactory接口

    public interface PersonFactory<P extends Person> {
      P create(String firstName, String lastName);
    }
    
 main中调用
 
     PersonFactory<Person> personFactory = Person::new;
     Person person = personFactory.create("anxiang","anxu");
          
 我们只需要使用 Person::new 来获取Person类构造函数的引用，Java编译器会自动根据PersonFactory.create方法的签名来选择合适的构造函数。  
 
 
## 访问局部变量

        final int num = 2;
        Converter<Integer, String> stringConverter = (from) -> String.valueOf(from + num);
        stringConverter.convert(3); 
        
  上面结果为5。我们看下，num为final，那么可以把final去掉么？去掉的话，还会成功么？ 试下就知道，final去掉也是没问题的。但是如果在之后更改了num的值，
  在编译期间就会报错。因为访问的局部变量必须是不能更改，最终的值。
 
## Date API
  
  Java 8 在包java.time下包含了一组全新的时间日期API。新的日期API和开源的Joda-Time库差不多，但又不完全一样，
  下面的例子展示了这组新API里最重要的一些部分      
  
### Clock 时钟

Clock类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代 System.currentTimeMillis() 来获取当前的微秒数。
某一个特定的时间点也可以使用Instant类来表示，Instant类也可以用来创建老的java.util.Date对象

      Clock clock = Clock.systemDefaultZone();
      long millis = clock.millis();
      
      Instant instant = clock.instant();
      Date legacyDate = Date.from(instant);
      
### Timezones 时区

 在新API中时区使用ZoneId来表示。时区可以很方便的使用静态方法of来获取到。 时区定义了到UTS时间的时间差，
 在Instant时间点对象到本地日期对象之间转换的时候是极其重要的
 
    System.out.println(ZoneId.getAvailableZoneIds());
    
    ZoneId zone1 = ZoneId.of("Europe/Berlin");
    ZoneId zone2 = ZoneId.of("Brazil/East");
    System.out.println(zone1.getRules());
    System.out.println(zone2.getRules());
    
### LocalTime 本地时间    

LocalTime 定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。
之后比较时间并以小时和分钟为单位计算两个时间的时间差

    LocalTime now1 = LocalTime.now(zone1);
    LocalTime now2 = LocalTime.now(zone2);
    
    System.out.println(now1.isBefore(now2));  // false
    
    long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
    long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);
    
    System.out.println(hoursBetween);       // -3
    System.out.println(minutesBetween);  
    
### LocalDateTime 本地日期时间

   LocalDateTime 同时表示了时间和日期，相当于前两节内容合并到一个对象上了。LocalDateTime和LocalTime还有LocalDate一样，都是不可变的。
   LocalDateTime提供了一些能访问具体字段的方法
   
       LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);
       
       DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
       System.out.println(dayOfWeek);      // WEDNESDAY
       
       Month month = sylvester.getMonth();
       System.out.println(month);          // DECEMBER
       
       long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
       System.out.println(minuteOfDay);
       
   只要附加上时区信息，就可以将其转换为一个时间点Instant对象，Instant时间点对象可以很容易的转换为老式的java.util.Date
   
       Instant instant = sylvester
               .atZone(ZoneId.systemDefault())
               .toInstant();
       
       Date legacyDate = Date.from(instant);
       System.out.println(legacyDate);     
       
   格式化LocalDateTime和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式：
   
       DateTimeFormatter formatter =
           DateTimeFormatter
               .ofPattern("MMM dd, yyyy - HH:mm");
       
       LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
       String string = formatter.format(parsed);
       System.out.println(string);
       
## Annotation 注解

  在Java 8中支持多重注解了，先看个例子来理解一下是什么意思。
  首先定义一个包装类Hints注解用来放置一组具体的Hint注解：
  
      public @interface Hints {
      
          Hint[] value();
      }
      
      @Repeatable(Hints.class)
      @interface Hint {
          String value();
      }
      
  Java 8允许我们把同一个类型的注解使用多次，只需要给该注解标注一下@Repeatable即可。
  
  使用包装类当容器来存多个注解（老方法）
  
    @Hints({@Hint("hint1"), @Hint("hint2")})      
    
  使用多重注解（新方法）
  
     @Hint("hint1")
     @Hint("hint2")    