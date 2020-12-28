#### 启动

```
redis-server
```

直接打开redis-cli就能用

### 以后台进程方式启动redis

> > 为什么？
> >
> > 因为 ./redis-server 以这种方式启动redis需要一直打开窗口，不能进行其他操作，不太方便。
> >
> > 按ctrl+c可以关闭窗口

```
(守护进程)将daemonize no 修改为 daemonize yes
```

### Redis通讯协议RESP( Redis Serialization Protocol (Redis序列化协议) )

---

间隔符号，在Linux下是\r\n，在Windows下是\n 

##### 客户端发送命令的格式(类型)：5种类型

+ 简单字符串  

  ​      格式：\+ 字符串 \r\n

  ​      字符串不能包含 CR或者 LF(不允许换行)

  ​	 eg: "+humei\r\n" 

+ 错误 

   	格式： \- 错误前缀 错误信息 \r\n 

  ​    错误信息不能包含 CR或者 LF(不允许换行) 

     eg: "-Error unknow command 'foobar'\r\n" 

+ 整形

  ​    格式：: 数字 \r\n

     eg: ":1000\r\n"

+ 大字符串 Buik String（长度限制512M）

     格式：$ 字符串的长度 \r\n 字符串 \r\n

  ​        字符串不能包含 CR或者 LF(不允许换行);

     eg: "$**6**\r\n**foobar**\r\n"   其中字符串为 foobar，而6就是foobar的字符长度

  ​      "$0\r\n\r\n"    空字符串

  ​      "$-1\r\n"      null

+ 数组类型

  ​    格式：* 数组元素个数 \r\n 其他所有类型 (结尾不需要\r\n)

  ​      （只有元素个数后面的\r\n是属于该数组的，结尾的\r\n一般是元素的)

     eg: "*0\r\n"    空数组

  ​      "*2\r\n$2\r\nfoo\r\n$3\r\nbar\r\n"    数组包含2个元素，分别是字符串foo和bar

  　"*3\r\n:1\r\n:2\r\n:3\r\n"    数组包含3个整数：1、2、3

  ​      "*5\r\n:1\r\n:2\r\n:3\r\n:4\r\n$6\r\nfoobar\r\n"  包含混合类型的数组

  ​      "*-1\r\n"     Null数组

  ​      "*2\r\n*3\r\n:1\r\n:2\r\n:3\r\n*2\r\n+Foo\r\n-Bar\r\n"  数组嵌套，外层数组包含2个数组，整理后如下：

  ​         "*2\r\n

  　　　　　　*3\r\n:1\r\n:2\r\n:3\r\n

  　　　　　　*2\r\n+Foo\r\n-Bar\r\n"

