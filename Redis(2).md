###  如何实现千万级用户，统计连续打卡机制

```java
查看string的基本api
help @String
查看redis底层种类
object encoding key
不底层
type key

set K1 V1
setbit K1 “offset” “0或1”
getbit K1 “offset”
    
eg：setbit K1 4 1
data：   0 0 0 0 1 0 0 0
offset： 0 1 2 3 4 5 6 7
```

##### 如何实现？

可以把offset看成用户id，如果用户登录了就把offset对应的data设为1。

```xm
setbit login:2020:1224 1 1
setbit login:2020:1224 2 1
setbit login:2020:1224 3 1

bitcount login:2020:1224 0 -1
3
```

##### 如何确定该用户是否是连续登录？

两天的data值进行与运算

