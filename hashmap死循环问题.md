#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HashMap在多线程并发执行下，在使用get（）方法时会发生CPU占用率高达100%的情况。
# 1. 原因分析
#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在内部，HashMap使用一个Entry数组保存key，value数据，当一对key，value被加入时，会通过一个hash算法得到数组下标index，根据key的hash值，对数组的大小取模hash&（length-1），并把结果插入数组该位置，如果该位置上已经有元素了，则说明存在hash冲突，这样会在index位置生成链表。
#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果存在hash冲突，最坏情况就是所有元素定位到同一个位置，形成一个长长的链表，这样get一个值的时候，需要遍历所有节点，性能变成了O（n），所以元素的hash算法和HashMap的初始化大小很重要。 
#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当插入一个新节点时，如果不存在相同的key，则会判断当前内部元素是否已经达到阀值（默认是数组大小的0.75），如果已经达到阀值，会对数组进行扩容，也会对链表中的元素进行rehash。
## 1.1 put方法实现：
#### （1）判断key是否存在，如果存在，则替换value，并返回旧的值，不存在则插入新的元素；
#### （2）检查容量是否达到阀值threshold，如果元素个数已经达到阀值，则扩容，并把原来的元素移动过去；
## 1.2 扩容
#### （1）新建一个更大的数组，并通过transfer方法，移动元素；
#### （2）遍历原来table中每个位置的链表，并对每个元素进行重新hash，在新的newTable找到位置并插入；

# 2. 案例分析
#### 假设HashMap初始化大小为4，插入三个节点，这3个节点都hash到同一个位置，如果按照默认的负载因子的话，插入第三个节点就会扩容，为了验证效果，假设负载因子是1；
![image](C96580CACBD7498F9AD04CE098D0F313)

#### 插入第四个节点时，发生rehash，假设现在有两个线程同时进行，线程1和线程2，两个线程都会新建数组；
![image](760BFF7D7C3049C1BF364C2A83B5C411)

#### 假设线程2执行到Entry<K，V>next=e.next;之后，cpu时间片用完了，这时变量指向节点a，变量next指向节点b。
#### 线程1继续执行，此时假设a，b，c节点rehash之后又是在同一个位置7，开始移动节点：
#### 第一步移动节点a；
![image](B3FD12C545B0424CA3F70924D92BC620)
#### 第二步移动节点b；
![image](B6F7193D263B4D33B9384B1B34183C26)
#### 注意：这里是头插法，顺序会相反
![image](73BE67E7722A483988B46BC5F49C4562)

#### 这个时候线程1的时间片用完，内部的table还没有设置成新的newTable，线程2开始执行，这时内部的引用关系如下：
![image](9EF55A6FC8DF4806A8074E1F2E164EA6)
#### 此时在线程2中，中间节点指向a，next指向b，开始执行剩余的逻辑，执行完后的引用关系如下图：
![image](1ED0DFCB29FD44799587BC5EB073012C)
#### 执行后，变量e指向节点b，因为e不是null，则继续执行循环体，执行后的引用关系：
![image](38397194144A426A8FE020DF8E274D31)

#### 变量e又重新指向节点a，只能继续执行循环体：
#### （1）执行完Entry<K，V>next = e.next,目前节点a没有next，所以变量next指向null；
#### （2）e.next = newTable[i],其中newTable[i]指向节点b,那就是把a的next指向了节点b，这样a和b就互相引用了，形成了一个环；
#### （3）newTable[i] = e 把节点a放到了数组i位置；
#### （4）e = next; 把变量e赋值为null，因为第一步中变量next就是指向null；
#### 所以最终的引用关系是这样的：
![image](FB344D6697B64E03BDAF1850E6657F0E)
#### 节点a和b互相引用，形成了一个环，当在数组该位置get寻找对应的key时，就发生了死循环；
#### 另外，如果线程2把newTable设置成到内部的table，节点c的数据就丢失了，还会造成数据丢失问题。
