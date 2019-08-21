---
layout: post
title:  redis持久化RDB,AOF
categories: redis
description: redis持久化方式RDB与AOF的设计思想与过程解析
keywords: redis
---

 大家都知道持久化，应该也知道为什么需要持久化，在什么场景下需要持久化。而redis作为内存缓存，如何避免断点，宕机等之后的数据丢失问题呢，这就是
 这章讲的内容，redis持久化之RDB,AOF。

## RDB--数据快照（Snapshot）
   
   RDB 可以通过手动执行命令 SAVE/BGSAVE 把Snapshot保存到RDB文件中，也可以通过配置多长时间自动执行一次。
   Redis也可以通过读取RDB文件，将数据加载读取到Redis中
  
### RDB文件创建

   连接Redis，设置值后执行SAVE命令
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/rdb/save-command.jpg)
   
   然后可以查看下redis.conf的持久化工作目录。进入目录可以看到保存了一个dump.rdb文件。该文件是一个二进制文件，无法直接正常打开。
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/rdb/rdb-file.jpg)
   
   以上是手动执行的过程。但在生产我们很少会手动登上服务去执行操作，所以更多的时候是依赖Redis的配置，定时保存RDB文件
   
   打开redis.conf配置文件，找到SNAPSHOTTING的配置，Save Point的设置。
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/rdb/save-point.jpg)
   
   图中的配置意思是，当至少有一个key变更时，900秒后会执行一次SAVE。其他配置同理，有10次变更，300秒后保存一次.....
   
   在Redis中，这个自动保存RDB的功能是默认开启的，默认的值就是图中的值。
   
   至于SAVE/BGSAVE的区别，就是前置是阻塞执行，此时服务不会接受请求，后者是Fork一个子进程出来，由该进程去执行保存RDB文件的操作，不影响用户请求。
   
   PS. Redis是单进程的，所以BGSAVE只能Fork一个子进程，而不是创建一个线程处理。
   
### RDB文件加载

   当启动Redis服务的时候会看见这么一行 
   
   DB loaded from disk: 0.000 seconds
   
   并且Redis中，依然有之前设置的值。说明Redis在启动的时候，会加载数据初始化。
   
   不过，这里加载的初始化数据不一定是RDB的。如果Redis开启了AOF，会优先从AOF初始化数据，否则才会加载RDB的数据。
   
### RDB优缺点
   
   #### 优点：
   
   1.RDB是某一时间点的快照，是一个紧凑的单文件，更多用于数据备份。可以按每小时或每日来备份，方便从不同的版本恢复数据。
   2.单文件容易传输到远程服务做故障恢复。
   3.RDB可以Fork子进程进行持久化，使Redis可以更好地处理用户请求
   4.在大量数据的情况下，RDB相比较于AOF会更快的加载。
   
   #### 缺点：
   
   如果Redis不及时保存RDB文件，会造成数据的丢失。例如系统突然断电，但未来得及保存数据。即使你设置更多的Save point，也无法保证100%的数据不丢失。
   RDB经常需要fork子进程去执行，但如果再大量数据的情况下，这个fork操作会非常耗CPU资源的。对比AOF虽然也是fork，但是它的数据保存处理是可以控制的，不需要全量保存。
 
## AOF——日志追加（Append-Only）   

   Redis的另外一种持久化方案就是AOF，Append Only File。AOF相当于一个操作的日志记录，每次对于数据的变更都会记录追加到AOF日志。
   当服务启动的时候就会读这些操作日志，重新执行一次操作，从而恢复原始数据。
   
### AOF-启动   

   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/aof/aof-on.jpg)
   
   AOF默认是关闭的。打开redis.conf配置文件，找到appendonly no改成appendonly yes
   
   光配置appendonly是没用的，记得要在客户端执行这两个命令
   redis-cli config set appendonly yes
   redis-cli config set save ""
   只有执行了这两个命令，安装目录下才会有aof文件
   
### AOF-fsync 同步规则   

   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/aof/aof-on.jpg)
   
   fsync()是一个系统调用函数，告诉操作系统把数据写到硬盘上，而不是缓存更多数据才写到硬盘。这样的调用可以及时保存数据到硬盘上。
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/aof/aof-fsync.jpg)
   
   Redis提供了三种fsync的调用方式
   
   1.appendfsync always，每次操作记录都同步到硬盘上，最低效，最安全。
   2.appendfsync everysec，每秒执行一次把操作记录同步到硬盘上。
   3.appendfsync no，不执行fysnc调用，让操作系统自动操作把缓存数据写到硬盘上，不可靠，但最快。
   默认开启 appendfsync everysec
   
### AOF-文件格式解析

   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/aof/aof-rule.jpg)

   文件解析说明：
  
   *，表示命令的参数个数，例如set a 1是三个参数，所以是*3
   $，表示参数的字节数，例如set这个参数是三字节，所以是$3，key值a是一个字节，所以是$1
   无符号，表示是参数的数据，例如set,a,1就是具体的数据
   
### AOF-日志重写   

   AOF虽然比RDB更可靠，但缺点也是比较明显的，就是每次写操作都要把操作日志写到文件上，这样会导致文件非常冗余。

   假若你要自增一个计数器100次，如果不重写，AOF文件就就会有这100次的自增记录，如INCR a。如果执行了日志重写，
   那么文件只会保留set a 100而不是100条INCR a。这样拥有相同的结果，但可以大大减少AOF的文件大小，并且可以让AOF载入的时候提升载入的效率。

   看回redis.conf配置，有两项控制rewrite的选项。
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/data-persistence/aof/aof-rewrite.jpg)
   
   auto-aof-rewrite-percentage 100，当文件增长100%（一倍）时候，自动重写。
   auto-aof-rewrite-min-size 64mb，日志重写最小文件大小，如果小于该大小，不会自动重写。
   
### AOF优缺点
   
   #### 优点：
   
   1.AOF可以设置 完全不同步、每秒同步、每次操作同，默认是每秒同步。因为AOF是操作指令的追加，所以可以频繁的大量的同步。
   2.AOF文件是一个值追加日志的文件，即使服务宕机为写入完整的命令，也可以通过redis-check-aof工具修复这些问题。
   3.如果AOF文件过大，Redis会在后台自动地重写AOF文件。重写后会使AOF文件压缩到最小所需的指令集。
   4.AOF文件是有序保存数据库的所有写入操作，易读，易分析。即使如果不小心误操作数据库，也很容易找出错误指令，恢复到某个数据节点。例如不小心FLUSHALL，可以非常容易恢复到执行命令之前。
   
   #### 缺点
   
   1.相同数据量下，AOF的文件通常体积会比RDB大。因为AOF是存指令的，而RDB是所有指令的结果快照。但AOF在日志重写后会压缩一些空间。
   2.在大量写入和载入的时候，AOF的效率会比RDB低。因为大量写入，AOF会执行更多的保存命令，载入的时候也需要大量的重执行命令来得到最后的结果。RDB对此更有优势。
   
## 如何选择

   实际当中，我们到底怎么去选择用哪种持久化方式呢？
   
   一般来说，不考虑硬盘大小，最安全的做法是RDB与AOF同时使用，即使AOF损坏无法修复，还可以用RDB来恢复数据。
   
   如果Redis的数据在你的服务中并不是必要的数据，例如只是当简单的缓存，没有缓存也不会造成缓存雪崩。说明数据的安全可靠性并不是首要考虑范围内，那么单独只使用RDB就可以了。
   
   不推荐单独使用AOF，因为AOF对于数据的恢复载入来说，比RDB慢。
   
   并且Redis官方也说明了，AOF有一个罕见的bug。出了问题无法很好的解决。所以使用AOF的时候，最好还是有RDB作为数据备份