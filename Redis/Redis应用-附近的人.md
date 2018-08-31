## Redis 应用GeoHash位置计算

1. 寻常数据库计算附近的人
我们都知道地球经度范围 (-180, 180]，纬度范围 (-90, 90],计算两个地点的距离使用经纬度算法得到一个大约的距离
而经纬度信息存储在数据库中 那我我们的写法如下:
```
距离半径为r, 经度 x, 纬度 y
select id from table where x0-r < x < x0+r and y0-r < y < y0+r
```
这样的做法计算量太大,高并发的情况下性能也会很低,需要进行全表扫描,所以我们引入Redis 的GeoHash进行位置计算

2. GeoHash进行位置计算附近的人

* 在Redis里面经纬度使用52位的整数进行编码,放在zset里面,zset的value是元素的key,score 是 GeoHash 的 52 位整数值
* 查询时使用score值进行排序

#### 使用
```
127.0.0.1:6379> geoadd friend 118.14205 40.000001 kongming01
(integer) 1
127.0.0.1:6379> geoadd friend 117.14302 39.000110 kongming02
(integer) 1
127.0.0.1:6379> geoadd friend 117.24801 39.001100 kongming03
(integer) 1
127.0.0.1:6379> geoadd friend 115.14821 39.001601 kongming04 115.238321 42.421342 kongming05
(integer) 2
127.0.0.1:6379> geodist friend kongming01 kongming03 km
"135.0227"
127.0.0.1:6379> geodist friend kongming01 kongming03 m
"135022.7261"
127.0.0.1:6379> geopos friend kongming01
1) 1) "118.14205080270767212"
   2) "39.99999991084916218"
127.0.0.1:6379> geopos friend kongming03
1) 1) "117.24801152944564819"
   2) "39.00109925332875349"

```

* Redis 没有提供Geo删除指令,因为存储结构是zset我们想要删除只需要zrem删除就可以了
* geoadd的数据与geopos的数据有稍微的偏差,但完全是可以接受的偏差
* geoHash还可以获取元素经纬度编码的字符串
```
127.0.0.1:6379> geohash friend kongming01
1) "wxh583u2vk0"
```
