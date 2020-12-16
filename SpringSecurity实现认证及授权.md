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

 我们需要通过 **`SecurityContext`** 来获取`Authentication`，`SecurityContext`就是我们的上下文对象。

 这个上下文对象则是交由 **`SecurityContextHolder`** 进行管理 。

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```

 以看到调用链路是这样的：`SecurityContextHolder` `SecurityContext` `Authentication`。 

`SecurityContextHolder`的实现原理JWT的一样，都是用`ThreadLocal`来保证线程传递一样的对象。

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