**1.** **定义**

**JRE(Java Runtime Enviroment)****是Java的运行环境**。面向Java程序的使用者，而不是开发者。如果你仅下载并安装了JRE，那么你的系统只能运行Java程序。JRE是运行Java程序所必须环境的集合，包含JVM标准实现及 Java核心类库。它包括Java虚拟机、Java平台核心类和支持文件。它不包含开发工具(编译器、调试器等)。

**JDK(Java Development Kit)**又称J2SDK(Java2 Software Development Kit)，是Java开发工具包，它提供了Java的开发环境(提供了编译器javac等工具，用于将java文件编译为class文件)和运行环境(提 供了JVM和Runtime辅助包，用于解析class文件使其得到运行)。如果你下载并安装了JDK，那么你不仅可以开发Java程序，也同时拥有了运 行Java程序的平台。JDK是整个Java的核心，包括了Java运行环境(JRE)，一堆Java工具tools.jar和Java标准类库 (rt.jar)。

**2.** **区别**

JRE主要包含：java类库的class文件(都在lib目录下打包成了jar)和虚拟机(jvm.dll)；JDK主要包含：java类库的 class文件(都在lib目录下打包成了jar)并自带一个JRE。那么为什么JDK要自带一个JRE呢？而且jdk/jre/bin下的client 和server两个文件夹下都包含jvm.dll(说明JDK自带的JRE有两个虚拟机)。

---

#### 编译和运行的区别

1.编译就是将java代码交给编译器进行语法检查，如果没有错误就生成.class文件

2.运行就是将字节码文件(.class)交给java虚拟机执行，如果没有逻辑错误，就成功出现结果。

编译期和运行期内存的分配

1.编译期仅仅知道内存分配的位置和大小，不做具体的分配操作

2.运行期确定真正的分配，确定分配的大小以及位置

常见的错误

1.编译错误是常见的语法错误：缺少分号，大小写

2.运行错误是逻辑错误：空指针异常，越界访问，除数为0等

 启动类加载器加载的是jre和jre/lib目录下的核心库，具体路径要看你的jre安装在哪里。

---

### 杀死进程

```
ps aux | grep chrome
```

kill -9 xxx

首先可以用**lsof**查看占用端口的进程号

```shell
	lsof -i:端口号
```

然后**kill**掉占用进程，就可以再次启动server了

```shell
	kill -9 进程号
```

当然上述还是有些麻烦，因此可以用以下一条命令替代：

```shell
	sudo kill -9 $(lsof -i:端口号 -t)
```

云服务器永久启动

screen  -S “随便啥名字"

然后再打开springboot

同一个网络下ip是可以ping的

#### CMD

2.查看指定端口的连接信息
使用netstat -ano | findstr “8080”，在“|”前面的命令执行结果作为后一个命令执行的输入。 

3.查看进程列表
使用tasklist | findstr “java”,查找进程名包含“java”的所有进程，详细使用方式使用tasklist/?查看。

4.通过查看占用端口号的进程，可以直接杀掉进程，

命令行使用taskkill /PID 进程号 -F -T

---

### @Repository和@Controller、@Service、@Component的作用差不多，都是把对象交给spring管理。@Repository用在持久层的接口上，这个注解是将接口的一个实现类交给spring管理。

### @Repository的作用：

**这是因为该注解的作用不只是将类识别为Bean，同时它还能将所标注的类中抛出的数据访问异常封装为 Spring 的数据访问异常类型。 Spring本身提供了一个丰富的并且是与具体的数据访问技术无关的数据访问异常结构，用于封装不同的持久层框架抛出的异常，使得异常独立于底层的框架。**

---

 

#### 开启Mybatis下划线命名转驼峰命名

 mybatis.configuration.map-underscore-to-camel-case=true 

```yml
mybatis:  
	configuration:    
		map-underscore-to-camel-case: true
```

---

#### 上传到阿里云服务器

 scp sta-0.0.1-SNAPSHOT.jar root@116.62.179.174://opt/tomcat/apache-tomcat-8.5.51/webapps
