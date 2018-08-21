## Redis基础数据结构

#### string(字符串)

  ![key->value](https://github.com/kmjueban/studious-funicular/blob/master/static/redis_key_value.png)


* 字符串string 是Redis最简单的数据结构,Redis所有的数据结构都是以唯一的key作为名称,通过这个唯一的key获取对应的value值,
  value结构的不同造成是数据结构差异的根本原因
* 字符串最常用的就是缓存用户信息,我们可以将用户的信息JSON序列化为字符串塞进Redis来缓存
* Redis的字符串是动态字符串,内部实现相当于java的ArrayList,采用预分配冗余空间来减少内存的频繁分配
* 扩容方式为加倍现有的空间,超过1M，扩容时只会多扩展1M 最大长度为512M

##### 键值对

   <img src="https://github.com/kmjueban/studious-funicular/blob/master/static/redis_string.png" width="80%" />

##### 批量键值对 批量读写 节省网络耗时开销

    <img src="https://github.com/kmjueban/studious-funicular/blob/master/static/redis_mstring.png" width="80%" />

##### 过期和 set 命令扩展 对key设置过期时间 控制缓存失效时间

   <img src="https://github.com/kmjueban/studious-funicular/blob/master/static/redis_string_setnx.png" width="80%" />

##### 计数  对于value为整数 可以让其进行自增操作

   <img src="https://github.com/kmjueban/studious-funicular/blob/master/static/redis_incr.png" width="80%"/>

---

#### list(列表)

* list


#### hash(字典)

* hash


#### set(集合)

* set


#### zset(有序列表)

* zset
