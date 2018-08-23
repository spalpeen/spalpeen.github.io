## Redis应用-延时队列

#### 异步消息队列

  ![set](https://github.com/kmjueban/studious-funicular/blob/master/static/redis_list_queue.png)

* Redis的列表数据结构通常用来作为异步消息队列来使用
* 使用rpush/lpush操作队列,使用lpop/rpop来出队列

```
127.0.0.1:6379> rpush kongming-queue kongming01 kongming02 kongming03
(integer) 3
127.0.0.1:6379> llen kongming-queue
(integer) 3
127.0.0.1:6379> lpop kongming-queue
"kongming01"
127.0.0.1:6379> llen kongming-queue
(integer) 2
127.0.0.1:6379> lpop kongming-queue
"kongming02"
127.0.0.1:6379> llen kongming-queue
(integer) 1
127.0.0.1:6379> lpop kongming-queue
"kongming03"
127.0.0.1:6379> llen kongming-queue
(integer) 0

```

##### 队列空了

* 客户端通过pop指令来操作队列获取消息,进行处理,处理完之后继续获取消息,循环往复
* 如果队列空了,客户端就会陷入pop的死循环,不停的pop,没有数据也pop,这是在空轮询
* 空轮询拉高客户端的cpu,redis 的QPS,客户端多起来 Redis的慢查询可能会显著增多
* 通常用sleep来解决,让线程睡一会,客户端的cup和Redis的QPS都会降下来

##### 队列延迟

* 睡眠方法可以解决问题,但是睡眠会导致消息的延迟增大,如果一个消费者就会延迟1s,多个消费者,延迟会下降,
  因为每个消费者的睡眠时间是岔开的
* blpop/brpop b代表blocking 阻塞读
* 阻塞读在队列没有数据的时候会进入休眠状态,一但数据到来就立刻醒过来,消息延迟机会为0

##### 空闲自动断开

* 阻塞都可以完美解决延迟问题 但同样引申出来另外一个问题---空闲连接
* 线程一直阻塞在那里,Redis的客户端连接就成了闲置连接,限制久了服务器一般会主动断开连接,减少闲置资源的占用
  这时候blpop/brpop就会抛出异常
* 所以需要进行异常捕捉及重试


#### 延时队列的实现

* 延时队列可以通过Redis的zset来实现
* 将消息序列化为字符串当作zset的value,这个消息的到期时间为分数,然后多个线程轮询zset
  获取到期的任务进行处理,多线程是为了确保可用性,挂掉一个线程其他线程还可以继续处理,
  但带来的就是并发争抢问题,需要确保任务不能被多次执行



