## **Volatile**
### 为什么引入可见性？
现代处理器为了提高处理的速度，在处理器和内存之间加了一个多级缓存，处理器不会直接和内存通信，而是将数据读到内部缓存中再进行操作。多级缓存容易导致数据不一致问题

### 如何实现可见性（缓存一致性协议）
处理器通过嗅探总线传播的数据来检查自己的数据是否过期了。当处理器发现自己缓存行对应的地址改变了，就将缓存行设置为无效。当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。

#### volatile是两条实现原则：
##### 1.Lock前缀指令会引起处理器缓存写到内存
当对volatile修饰的变量进行写操作时，jvm会给处理器发送一个一个lock前缀指令，将缓存中的变量刷回系统内存。
##### 2.一个处理器的缓存回写到内存会导致其他处理器的缓存失效

处理器使用嗅探技术保证内部缓存 系统内存和其他处理器的缓存的数据在总线上保持一致。

**综合上面两条实现原则，我们了解到：如果一个变量被volatile所修饰的话，在每次数据变化之后，其值都会被强制刷入主存。而其他处理器的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中。这就保证了一个volatile在并发编程中，其值在多个缓存中是可见的。**

为了保证内存的可见性，除了缓存一致性协议还有一个happends-before关系。[happends-before](https://blog.csdn.net/lc13571525583/article/details/90345760)

**注意的点**
volatile不具有传染性，用volatile修饰的对象的内部属性不具有可见性，反之，被volatile修饰的内部属性也不能保证该对象的可见性。**数组与对象实例中的volatile，针对的是引用，对象获数组的地址具有可见性，但是数组或对象内部的成员改变不具备可见性。**

---
###### volatile只是一个变量的修饰符，并不能保证操作的原子性
**volatile**：能保证一个线程写、一个线程读，每次读都能保证读到最新的值。

**Atomic**：能保证一个线程写一个线程读，他们操作的原子性。

**Sychronized**:保证代码块内部整个的原子性。