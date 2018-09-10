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