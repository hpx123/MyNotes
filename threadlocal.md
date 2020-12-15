## ThreadLocal
###### 针对的不是程序的全局变量，是线程的全局变量

ThreadLocal本身不存任何值
ThreadLocalMap
底层是一个Entry[]数组，和hashmap类似，通过哈希运算将V赋值。如果产生哈希冲突，则采用开放寻址法(如果某个数据经过散列函数散列之后，存储位置已经被占用了，我们就从当前位置开始，依次往后查找，看是否有空闲位置，直到找到为止)。

>[源码分析](https://blog.csdn.net/u014532775/article/details/100904191?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control)



