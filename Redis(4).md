### 为什么要用redis？它解决了什么问题？

是一个高性能的key-value内存数据库， 整个数据库统统加载在内存当中进行操作。他支持常用的5种数据结构：String，Hash哈希表，List列表，Set集合，Zset有序集合等数据类型。

##### 解决的问题？

1.性能(速度)

通常数据库的操作，一般都需要十几秒，而redis的读操作仅需要不到一毫秒。通常只要把数据库的数据缓存进redis，就能得到几十倍甚至上百倍的性能提升。

2.并发

在大并发的情况下，所有的请求直接访问数据库，数据库会出现连接异常，甚至卡死在数据库中。为了解决卡死的问题，一般的做法是采用redis的一个缓冲操作，让请求先访问到redis，而不是直接访问数据库。

3.持久化

 由于所有数据保持在内存中，所以对数据的更新将异步地保存到磁盘上，Redis提供了一些策略来保存数据，比如根据时间或更新次数。数据超过内存，使用swap，保证数据； 



#### Q:什么是持久化

 将数据从掉电易失的内存放到永久存储的设备上 

#### RDB模式（默认开启）

 RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。 

##### RDB快照有两种触发方式

1.通过配置参数

```java
//900秒有1条更新就生成一个快照
save 900 1
save 300 10
save 60 10000
```

2、通过手动执行bgsave /save，显示出发生成快照

save：阻塞主进程

bgsave：fork一个子进程



```python
1、保存真实的数据，将服务器包含的所有数据库数据以二进制文件的形式保存到硬盘里面
2、开启RDB方式redis会在指定的时间段内将内存中的数据快照到磁盘中，redis启动时再恢复到内存中。
3、Redis会单独创建（fork）一个线程，fork子进程来完成写操作，让主进程继续处理命令。使用单独子进程来进行持久化，保证了redis的高性能
2、当重启恢复数据的时候，数据量比较大时，Redis直接解析RDB二进制文件，生成对应的数据存储在内存中
如果需要进行大规模的数据恢复，并且对于数据恢复不是很敏感，RDB的方式比AOF方式更加高效，RDB的缺点就在于最后一次持久化后的数据有可能会丢失。
```

 ![img](https://img2018.cnblogs.com/blog/1481291/201809/1481291-20180925141429889-1694430603.png) 

RDB 的缺点

```python
1、创建RDB文件需要将服务器所有的数据库的数据都保存起来，这是一个非常消耗资源和时间的操作，所以服务器需要隔一段时间才创建一个新的RDB文件，(默认是当一条数据写入时15分钟持久化一次，当10条数据发生变化5分钟,持久化一次，当10000条数据发生变化1分钟进行持久化。)也就是说，创建RDB文件不能执行的过于频繁，否则会严重影响服务器的性能
2、可能丢失数据
```

#### AOF（AppendOnlyFile）

 ![img](https://img2018.cnblogs.com/blog/1481291/201809/1481291-20180925141527592-2105439510.png) 

 对于AOF的存储方式redis并没有默认开启。通过配置开启如下：
![这里写图片描述](https://img-blog.csdn.net/20170125105752233?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmZlbmdkZWp1YW5saWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

 AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾。 

- **AOF持久化原理及优点**

  ```
  **原理**
     1、每当有修改数据库的命令被执行时,以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，
     2、只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis
  重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
  
  **优点**
    用户可以根据自己的需要对AOF持久化进行调整，Redis在遭遇意外停机时不丢失任何数据，或者只丢失一秒钟的数据，这比RDB持久化丢失的数据要少的多
  ```

+ **AOF重写**

**思考：AOF文件中是否会产生很多的冗余命令？**

```python
AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,
当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩。通过这个功能，服务器可以产生一个新的AOF文件
原理
 AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。
  -- 新的AOF文件会使用尽可能少的命令来记录数据库数据，因此新的AOF文件的提及通常会小很多
  -- AOF重写期间，服务器不会被阻塞，可以正常处理客户端发送的命令请求
```

##### AOF存储方式优点：

1、 每秒同步。
2、 每修改同步。

##### AOF存储方式缺点：

1、 AOF文件远大于EDB。
2、 运行效率慢。

---

### Redis如何处理过期数据

对于已过期的数据，Redis使用两种策略**惰性删除和定期删除**

##### 惰性删除

不主动删，访问到key过期时再删除，如果执行删除的话返回null，否则就正常返回信息给客户端

#####  定期删除

Redis会周期性的随机测试一批设置了过期时间的key并进行处理。测试到的已过期的key将被删除

具体算法如下：

1、Redis配置项hz定义了serverCron任务的执行周期，默认为10，代表了每秒执行10次；

2、每次过期key清理的时间不超过cpu时间的25%，比如hz默认为10，则一次清理时间最大为25ms；

3、清理时依次便利所有的db；

4、从db中随机取20个key，判断是否过期，如果过期则逐出；

5、若有4个以上key过期，则重复步骤4.否则便利下一个db‘

6、在清理过程中，若达到了25%cpu时间，退出清理过程；

---

#### 当内存不够用时Redis又是如何处理的？

##### Redis 内存淘汰策略



#### LUR

随着时间越长，越新的时间加入的数据越后淘汰，越早时间加入的数据先淘汰。

当一个key加进来时，首先要遍历之前一整个链表，如果没有则直接加进去，如果有则删除之前的节点，并将当前的节点加进去。

缺点：

​    每次加数据都要遍历之前的链表，性能非常低

**优化**

用双向链表，并将所有的节点放到hashtale里。这样查找可以保证O(1)的时间复杂度
