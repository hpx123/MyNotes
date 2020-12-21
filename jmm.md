## jmm实现和底层原理

#### 线程间的通信

线程的通信是指线程之间以何种机制来交换信息。在编程中，线程之间的通信机制有两种，**共享内存和消息传递。**

+ 共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信，典型的共享内存通信方式就是通过共享对象进行通信。

+ 在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信，在java中典型的消息传递方式就是wait()和notify()。 

#### 线程间的同步

同步是指程序用于控制不同线程之间操作发生相对顺序的机制。

+ 在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。

+ 在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。 

### Java内存模型——JMM

---

- Java的并发采用的是共享内存模型

##### 缓存系统的引入

由于计算机的存储设备与处理器的运算速度有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的**高速缓存（Cache）**来作为内存与处理器之间的缓冲：**将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，**这样处理器就无须等待缓慢的内存读写了。

#####  缓存一致性问题

由于多个处理器拥有不同的高级缓存，但却同时共享同一主内存。 当多个处理器的运算任务都涉及同一块主内存区域时，将可能导致各自的缓存数据不一致，举例说明变量在多个CPU之间的共享。如果真的发生这种情况，那同步回到主内存时以谁的缓存数据为准呢？ 为了解决一致性的问题，需要各个处理器访问缓存时都遵循一些协议，在读写时要根据协议来进行操作，这类协议有MSI、MESI（Illinois Protocol）、MOSI、Synapse、Firefly及Dragon Protocol等。

 ![img](https://upload-images.jianshu.io/upload_images/4222138-49df5535c55287c4.png?imageMogr2/auto-orient/strip|imageView2/2/w/562/format/webp) 

##### 如何保证可见性

[volatile保证可见性](https://blog.csdn.net/u012556994/article/details/81262237)

[Java锁保证可见性](http://ifeve.com/java%E9%94%81%E6%98%AF%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E6%95%B0%E6%8D%AE%E5%8F%AF%E8%A7%81%E6%80%A7%E7%9A%84/)

##### 重排序问题

#### Happens-Before

用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系 。

两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second） 。

1）如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。(对程序员来说)

2）两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序是允许的(对编译器和处理器 来说)

理解happens-before是理解JMM的关键。JMM这么做的原因是：程序员对于这两个操作是否真的被重排序并不关心，程序员关心的是程序执行时的语义不能被改变（即执行结果不能被改变）。因此，**happens-before关系本质上和as-if-serial语义是一回事。as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变。**

 ![img](https://upload-images.jianshu.io/upload_images/4222138-8aeb6d12568cac67.png?imageMogr2/auto-orient/strip|imageView2/2/w/677/format/webp) 

[详情](https://www.jianshu.com/p/1f19835e05c0)

