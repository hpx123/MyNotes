### Zset有序集合

```xm
 zadd A1 200 a 400 b 600 c
 //得到abc
 ZRANGE A1 0 -1
 //得到a200b400c600
 ZRANGE A1 0 -1 WITHSCORES
 //200
 ASCORE A1 a
 //0
 ZRANK A1 a
 
```

 ```java
//往有序集合key中加入带分值元素
ZADD key score member [[score member]...]
//从有序集合key中删除元素
ZREM key member [member...]
//返回有序集合key中元素member的分值
ZSCORE key member
//为有序集合key中元素member的分值加上increment
ZINCRBY key increment member
//返回有序集合key中元素个数
ZCARD key
//正序获取有序集合key从start下标到stop下标
ZRANGE key start stop WITHSCORES
//倒叙
ZREVRANGE   
 ```

#####  微博排行榜

```java
//点击新闻
ZINCRBY hotNews：20201227 1 胡程刘昊然官宣
//展示当日排行前十
ZREVRANGE hotNews：20201227 0 9 WITHSCORES

```



---

### Set应用场景

##### 微信微博点赞，收藏，标签

```java
//点赞
SADD like:{消息ID}{用户ID}
//取消点赞
SREM like:{消息ID}{用户ID}
//检查用户是否点过赞
SISMEMBER like:{消息ID}{用户ID}
//获取点赞的用户列表
SMEMBERS like:{消息ID}
//获取点赞用户数
SCARD like:{消息ID}
```

##### 实现共同关注

交集、并集、差集

```java
//交集
SINTER set1 set2 set3 ->{三个集合公共的数据}
//并集
SUNION set1 set2 set3 ->{三个集合公共的数据去重后的数据}
//差集
SDIFF set1 set2 set3 ->{第一个集合减去后面集合的并集(并集和第一个集合公共的部分)}
eg:
//胡胖仙的关注
hpxSet->{yq,yl,yy,gsy,wcs}
//杨倩的关注
yqSet->{yyl,yl,gsy,tbw}
//胡胖仙和杨倩的共同关注
SINTER hpxSet yqSet->{yq,yyl,gsy}
//我关注的人也关注他
SISMEMBER hpxSet 人
//我可能认识的人
SDIFF ta ni
```



