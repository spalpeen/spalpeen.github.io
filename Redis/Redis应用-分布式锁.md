## Redis应用-分布式锁


#### 分布式锁
* 分布式锁本质上要实现的目标是在Redis 里占据一个坑位, 当别的进程要进来占用时,发现坑位已经被占用,不得不进行等待或重试

1.占坑位一般使用setnx(set if not exists) 原子性,只允许一个客户端占用,先来先占,用完del释放

```
127.0.0.1:6379> setnx lock:name kongming
(integer) 1
...do something...
127.0.0.1:6379> del lock:name
(integer) 1
```

2.如果逻辑执行过程中出现异常,del没有被执行,这样锁就永远不会被释放,造成死循环,于是我们给锁加上过期时间


```
127.0.0.1:6379> setnx lock:name kongming
(integer) 1
127.0.0.1:6379> expire lock:name 10
(integer) 1
...do something...
127.0.0.1:6379> del lock:name
(integer) 0
```

3.如果在执行setnx后服务器断电expire没有被执行 也会造成死锁,这类问题的根本原因就是setnx与expire结合不是原子性的指令
expire是依赖于setnx结果,如果setnx没有抢到锁,expire不会执行 所以事物也不能解决

  ![lock](../static/lock.gif)

```
# set key value ex time nx 就是setnx与expire组合在一起的原子性指令(version >=2.8)

127.0.0.1:6379> set lock:name kongming ex 10 nx
OK
....do something...
127.0.0.1:6379> del lock:name
(integer) 1
```

#### 超时问题

* Redis分布式锁不能解决超时问题
* 如果在加锁和释放锁的过程中逻辑还在执行,超出了锁的超时限制,就会出现问题
* (我抢占了头牌鼓励师,服务时间是3s,时间到但由于服务还差1s结束,下一位已经抢占了这位鼓励师,1s后我发布结束,这时第三位又抢占了这位鼓励师)
* 为value参数设置一个随机数,释放锁先匹配随机数,再删除key,确保当前线程的锁不会其它线程释放,只有自己或过期释放,但匹配value和删除key又不是原子操作

#### 锁冲突处理

处理请求时加锁没加成功通常分为以下三种解决方案
1. 抛出异常,通知稍后再试
2. sleep一会再试
3. 将请求转至延时队列,过一会再试




