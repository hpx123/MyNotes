## ThreadLocal
 ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响， 

##### 针对的不是程序的全局变量，是线程的全局变量

####  ThreadLocal类提供的几个方法： 

```java
public T get() { }//用来获取当前线程保存的变量的副本
public void set(T value) { }//设置添加
public void remove() { }//移除
protected T initialValue() { }//一般是用来在使用时进行重写的，它是一个延迟加载方法
```

**ThreadLocal本身不存任何值**

##### ThreadLocalMap

###### Entry 继承了WeakReference

###### key ThreadLocal

###### value(存副本) Object

底层是一个Entry[]数组，和hashmap类似，通过哈希运算将V赋值。如果产生哈希冲突，则采用**开放寻址法**(如果某个数据经过散列函数散列之后，存储位置已经被占用了，我们就从当前位置开始，依次往后查找，看是否有空闲位置，直到找到为止)。

#### 注

* **在进行get之前，必须先set，否则会报空指针异常 **



##### 上面说到了Entry继承了WeakReference，所以这里以一个弱引用指向ThreadLocal对象

```java
public void func1() {
        ThreadLocal tl = new ThreadLocal<Integer>(); //line1
         tl.set(100);   //line2
         tl.get();       //line3
}
```

 line1新建了一个ThreadLocal对象，t1 是强引用指向这个对象；line2调用set（）后，新建一个Entry，通过源码可知entry对象里的 k是弱引用指向这个对象。如图： 

 ![img](https://img2018.cnblogs.com/i-beta/1743446/201912/1743446-20191227164355954-1650595248.png) 

 当func1方法执行完毕后，栈帧销毁，强引用 tl 也就没有了，但此时线程的ThreadLocalMap里某个entry的 k 引用还指向这个对象。若这个k 引用是强引用，就会导致k指向的ThreadLocal对象及v指向的对象不能被gc回收，造成内存泄漏，但是弱引用就不会有这个问题。使用弱引用，就可以使ThreadLocal对象在方法执行完毕后顺利被回收，而且在entry的k引用为null后，再调用get,set或remove方法时，就会尝试删除key为null的entry，可以释放value对象所占用的内存。 

>[源码分析]( )



