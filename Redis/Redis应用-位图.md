## 位图

* 平时开发中经常会用到签到功能,假设签了是1 未签是0,使用key/value 存储每个人的key就是365个
用户量大起来存储数量将会很惊人

* 为了解决这个问题Redis提供了位图数据结构,这样每天的签到记录只占据一个位,364天就是365个位,
稍长一点的字符串就可以完全容纳,大大节省了存储空间

* 位图不是特殊的数据结构，它其实就是一个普通的字符串，也就是byte数组，我们可以使用普通的get/set
直接获取和设置整个位图的内容，也可以使用getbit/setbit等将byte数组看成位数组来处理

#### 基本使用

* Redis的位数组是自动扩展
* 如果设置某个偏移位置超出了现有的内容范围就会自动将位数组进行零扩充

我们使用位操作符将字符串设置为hello(并不是使用set)

1. 得到hello的ASCII 码(php得到为7位 实际最高位还有一个0)
```
<?php
echo decbin(ord('h'))."\n";
1101000   #01101000
echo decbin(ord('e'))."\n";
1100101
echo decbin(ord('l'))."\n";
1101100
echo decbin(ord('l'))."\n";
1101100
echo decbin(ord('o'))."\n";
1101111

```

```
                   h        e       l           l       o
hello 连起来为:01101000->01100101->01101100->01101100->01101111
              01234567  89....
```

接下来我们设置一个字符串h，h字符串只有1/2/4位需要设置

1. 零存整取
```
127.0.0.1:6379> setbit s 1 1
(integer) 0
127.0.0.1:6379> setbit s 2 1
(integer) 0
127.0.0.1:6379> setbit s 4 1
(integer) 0
127.0.0.1:6379> setbit s 9 1
(integer) 0
127.0.0.1:6379> setbit s 10 1
(integer) 0
127.0.0.1:6379> setbit s 13 1
(integer) 0
127.0.0.1:6379> setbit s 15 1
(integer) 0
127.0.0.1:6379> get s
"he"

```

2. 零存零取
```
127.0.0.1:6379> setbit w 1 1
(integer) 0
127.0.0.1:6379> setbit w 2 1
(integer) 0
127.0.0.1:6379> setbit w 4 1
(integer) 0
127.0.0.1:6379> getbit w 1
(integer) 1
127.0.0.1:6379> getbit w 2
(integer) 1
127.0.0.1:6379> getbit w 3
(integer) 0
127.0.0.1:6379> getbit w 4
(integer) 1
```

3. 整存零取
````
127.0.0.1:6379> set w h
OK
127.0.0.1:6379> getbit w 1
(integer) 1
127.0.0.1:6379> getbit w 2
(integer) 1
127.0.0.1:6379> getbit w 4
(integer) 1
127.0.0.1:6379> getbit w 5
(integer) 0
````

4. 如果对应位字节不可打印字符，显示该字符的16进制形式
```
127.0.0.1:6379> setbit x 0 1
(integer) 0
127.0.0.1:6379> setbit x 1 1
(integer) 0
127.0.0.1:6379> get x
"\xc0"
```

##### 统计和查找

* Redis提供位图统计指令bitcount和位图查找指令bitpos
* bitcount用来统计指定范围内1的个数
* bitpos用来查找指定范围内出现的第一个0或1

```
127.0.0.1:6379> set w hello
OK
127.0.0.1:6379> bitcount w  
(integer) 21
127.0.0.1:6379> bitcount w 0 0  # 第一个字符串中1的个数
(integer) 3
127.0.0.1:6379> bitcount w 0 1  # 前两个字符串中1的个数
(integer) 7
127.0.0.1:6379> bitpos w 0      #第一个0位
(integer) 0
127.0.0.1:6379> bitpos w 1      #第一个1位
(integer) 1
127.0.0.1:6379> bitpos w 1 1 1  #第二个字符串起第一个1位
(integer) 9
127.0.0.1:6379> bitpos w 1 2 2  #第三个字符串起第一个1位
(integer) 17

```

