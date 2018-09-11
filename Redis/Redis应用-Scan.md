## Scan
日常系统维护中我们经常会在Redis的很多key里面查找特定的key 进行修改或删除操作
而在成千上万的key中找出我们要的数据进行显然是很暴力的
接下来就给大家介绍Scan 的用法

```
127.0.0.1:6379> set name1 kongming01
OK
127.0.0.1:6379> set name2 kongming02
OK
127.0.0.1:6379> set name3 kongming03
OK
127.0.0.1:6379> set name4 kongming04
OK
127.0.0.1:6379> set name5 kongming05
OK
127.0.0.1:6379> set name6 kongming06
OK
127.0.0.1:6379> set name7 kongming07
OK
127.0.0.1:6379> set name8 kongming08
OK
127.0.0.1:6379> set name9 kongming09
OK
127.0.0.1:6379> keys *
1) "name4"
2) "name2"
3) "name3"
4) "name6"
5) "name1"
6) "name8"
7) "name7"
8) "name5"
9) "name9"
127.0.0.1:6379> keys name*
1) "name4"
2) "name2"
3) "name3"
4) "name6"
5) "name1"
6) "name8"
7) "name7"
8) "name5"
9) "name9"

```
查询所有的key有两个致命的缺点
* 没有offset 和limit 参数 一次性吐出所有符合条件的key 如果有很多的key那么你满屏都在跑数据
* keys 的时间复杂度是O(n) 如果服务中的key很多就会导致服务器卡顿 所有的读写redis指令都会被延后甚至报错

scan指令（版本2.8以上使用）

* scan 具有以下优点：
1. 复杂度虽然也是O(n) 但是是通过游标分布进行 不会阻塞线程
2. 提供limit参数 控制返回的条数
3. 提供模糊匹配功能
4. 不需要为游标保存状态

* scan 具有以下缺点：
1. 返回的结果可能会有重复
2. 遍历过程中有修改 改动后的数据能不能遍历到不确定
3. 返回数据为空并不代表遍历结束 要看游标的值是否为0

Scan语法

```
127.0.0.1:6379> scan 0 match key99* count 1000
1) "13976"
2)  1) "key9911"
    2) "key9974"
    3) "key9994"
    4) "key9910"
    5) "key9907"
    6) "key9989"
    7) "key9971"
    8) "key99"
    9) "key9966"
   10) "key992"
   11) "key9903"
   12) "key9905"
127.0.0.1:6379> scan 13976 match key99* count 1000
1) "1996"
2)  1) "key9982"
    2) "key9997"
    3) "key9963"
    4) "key996"
    5) "key9912"
    6) "key9999"
    7) "key9921"
    8) "key994"
    9) "key9956"
   10) "key9919"
127.0.0.1:6379> scan 1996 match key99* count 1000
1) "12594"
2) 1) "key9939"
   2) "key9941"
   3) "key9967"
   4) "key9938"
   5) "key9906"
   6) "key999"
   7) "key9909"
   8) "key9933"
   9) "key9992"
......
127.0.0.1:6379> scan 11687 match key99* count 1000
1) "0"
2)  1) "key9969"
    2) "key998"
    3) "key9986"
    4) "key9968"
    5) "key9965"
    6) "key9990"
    7) "key9915"
    8) "key9928"
    9) "key9908"
   10) "key9929"
   11) "key9944"...
127.0.0.1:6379> scan 0 match key99* count 10
1) "3072"
2) (empty list or set)   //游标不为0 便利没有结束
```
从结果可以看出虽然我们的limit是1000 但是返回的结果只有10 个左右
这是因为limit不是限定返回的条数 而是限定单词遍历字典槽位数量
只要游标值不是0 就意味着遍历没有结束
