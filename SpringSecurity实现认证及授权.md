### SpringSecurity

---

 Spring Security对Web系统的支持**就是基于这一个个过滤器组成的过滤器链**： 

 ![img](https://pic1.zhimg.com/80/v2-09d7a2f2b057ade11766e85bd66bab98_1440w.jpg)  用户请求都会经过`Servlet`的过滤器链 

 在`Servlet`过滤器链中，Spring Security向其添加了一个`FilterChainProxy`过滤器，这个代理过滤器会创建一套Spring Security自定义的过滤器链，然后执行一系列过滤器。 

 ![img](https://pic2.zhimg.com/80/v2-9219af17ade8575f17593bf3c9a73edd_1440w.jpg) 

 我们只需要重点关注两个过滤器即可：`UsernamePasswordAuthenticationFilter`负责登录认证，`FilterSecurityInterceptor`负责权限授权。 

 **Spring Security的核心逻辑全在这一套过滤器中，过滤器里会调用各种组件完成功能 **



#### 登陆认证

---

 我们系统中会有许多用户，确认当前是哪个用户正在使用我们系统就是登录认证的最终目的。 

  我们就提取出了一个核心概念：**当前登录用户/当前认证用户**。 

 这一概念在Spring Security中的体现就是 **`Authentication`**，它存储了认证信息，代表当前登录用户。 

 我们需要通过 **`SecurityContext`** 来获取`Authentication`，`SecurityContext`就是我们的上下文对象。（ 这种在一个线程中横跨若干方法调用，需要传递的对象，我们通常称之为上下文（Context）。上下文对象是非常有必要的，否则你每个方法都得额外增加一个参数接收对象，实在太麻烦了。 ）

###### 上下文对象: 即存储认证授权的相关信息，实际上就是存储"**当前用户**"账号信息和相关权限。这个接口只有两个方法，Authentication对象的getter、setter。 

 这个上下文对象则是交由 **`SecurityContextHolder`** 进行管理 。

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```

 以看到调用链路是这样的：`SecurityContextHolder` `SecurityContext` `Authentication`。 

`SecurityContextHolder`的实现原理JWT的一样，都是用`ThreadLocal`来保证线程传递一样的对象。

######  SecurityContextHolder工具类就是把SecurityContext存储在当前线程中。 以利用它获取当前用户的SecurityContext进行请求检查，和访问控制等。

 这种在一个线程中横跨若干方法调用，需要传递的对象，我们通常称之为上下文（Context）。上下文对象是非常有必要的，否则你每个方法都得额外增加一个参数接收对象，实在太麻烦了。 

在Web环境下，SecurityContextHolder是利用ThreadLocal来存储SecurityContext的。

现在我们已经知道了Spring Security中三个核心组件：

```
Authentication：存储了认证信息，代表当前登录用户
SeucirtyContext：上下文对象，用来获取Authentication
SecurityContextHolder：上下文管理对象，用来在程序任何地方获取SecurityContext
```

 ![img](https://pic2.zhimg.com/80/v2-7ddb334e9d9e77e79fd831e158273681_1440w.jpg) 

`Authentication`中那三个玩意就是认证信息：

`Principal`：用户信息，没有认证时一般是用户名，认证后一般是用户对象

`Credentials`：用户凭证，一般是密码

`Authorities`：用户权限

####  流程

```java
//判断用户名是否正确
// 账号密码正确了才将认证信息放到上下文中（用户权限需要再从数据库中获取，后面再说，这里省略）
Authentication authentication = new UsernamePasswordAuthenticationToken(用户名, 用户密码, 用户的权限集合);
SecurityContextHolder.getContext().setAuthentication(authentication);
```

 ![img](https://img2018.cnblogs.com/blog/1313132/201901/1313132-20190120155653707-1681055739.png) 

 ![img](https://img-blog.csdnimg.cn/20181202095539982.png) 

#### Spring Security的授权发生在`FilterSecurityInterceptor`过滤器中：

1. 首先调用的是 **`SecurityMetadataSource`**，来获取当前请求的鉴权规则
2. 然后通过`Authentication`获取当前登录用户所有权限数据： **`GrantedAuthority`**，这个我们前面提过，认证对象里存放这权限数据
3. 再调用 **`AccessDecisionManager`** 来校验当前用户是否拥有该权限
4. 如果有就放行接口，没有则抛出异常，该异常会被 **`AccessDeniedHandler`** 处理

##### 鉴权规则源SecurityMetadataSource

该接口我们只需要关注一个方法：

```java
public interface SecurityMetadataSource {
    /**
     * 获取当前请求的鉴权规则
     * @param object 该参数就是Spring Security封装的FilterInvocation对象，包含了很多request信息
     * @return 鉴权规则对象
     */
    Collection<ConfigAttribute> getAttributes(Object object);

}
```

##### ConfigAttribute就是我们所说的鉴权规则，该接口只有一个方法：

```java
public interface ConfigAttribute {
    /**
     * 这个字符串就是规则，它可以是角色名、权限名、表达式等等。
     * 你完全可以按照自己想法来定义，后面AccessDecisionManager会用这个字符串
     */
    String getAttribute();
}
```

