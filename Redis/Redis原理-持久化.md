## Redis持久化


大家都知道redis数据全部存在内存中，但是如果我们的服务器挂掉了，或者断电机器强制关机了，那么内存中的数据就会全部丢失
而redis必须有一种机制来保证数据不会因为故障而丢失，今天我们就来说一下redis的持久化机制

#### RDB持久化

* 内存数据的二进制序列化形式，存储非常紧凑

* 一次全量备份

* rdb文件保存目录及文件名

```
 # The filename where to dump the DB
  dbfilename dump.rdb      
 # The working directory.
 #
 # The DB will be written inside this directory, with the filename specified
 # above using the 'dbfilename' configuration directive.
 #
 # The Append Only File will also be created inside this directory.
 #
 # Note that you must specify a directory here, not a file name.
 dir ./
            
```

* 手动保存的两种方式

```
127.0.0.1:6379> save
OK
127.0.0.1:6379> bgsave
Background saving started
```

* rdb自动保存规则

```
 # Save the DB on disk:
 #
 #   save <seconds> <changes>
 #
 #   Will save the DB if both the given number of seconds and the given
 #   number of write operations against the DB occurred.
 #
 #   In the example below the behaviour will be to save:
 #   after 900 sec (15 min) if at least 1 key changed
 #   after 300 sec (5 min) if at least 10 keys changed
 #   after 60 sec if at least 10000 keys changed
 #
 #   Note: you can disable saving completely by commenting out all "save" lines.
 #
 #   It is also possible to remove all the previously configured save
 #   points by adding a save directive with a single empty string argument
 #   like in the following example:
 #
 #   save ""
  
 save 900 1 	//服务器在900s内对数据库进行了至少1次修改
 save 300 10	//服务器在300s内对数据库进行了至少10次修改
 save 60 10000	//服务器在60s内对数据库进行了至少10000次修改
 ```

##### 快照原理

* redis一边需要处理大量的客户端请求一边还需要进行持久化,而对内存的快照一定会涉及到严重影响服务器的性能的IO操作
  持久化的同时内存的数据结构还在改变,redis在持久化时会fork一个子进程,而持久化的过程完全交给子进程来做,父进程继续处理客户端请求，子进程刚产生时会与父进程共享内存里的数据段，子进程不会修改先有内存中的数据结构，只是对现有数据结构进行遍历，序列化后写进磁盘，而父进程则会在相应客户端请求不断的修改内存中的数据结构，这个时候就需要COW机制来进行数据段页面的分离

* COW：数据段是由很多的操作系统页面来组成,当父进程对其中一个页面进行修改时，会将这个页面复制一份分离出来，然后对这个页面的数据进行修改，而相应的子进程的页面是没有变化的还是进程产生的那一刻的数据,随着父进程修改的越来越多,被分离出来的页面也越来越多，但一定不会超过原页面的两倍,毕竟全部分离出来才是原页面的两倍



#### AOF持久化

* 内存数据修改的指令文本

* 连续增量备份

* AOF在长期的的运行过程中会很大，重启时加载AOF记录的指令日志就会变得很慢 所以需要定期进行AOF重写

* AOF持久化保存的文件名及是否开启

```
 appendonly no                                                                                                                     
  
 # The name of the append only file (default: "appendonly.aof")
  
 appendfilename "appendonly.aof"
```

* AOF持久化保存的规则

```
 # The fsync() call tells the Operating System to actually write data on disk
 # instead of waiting for more data in the output buffer. Some OS will really flush
 # data on disk, some other OS will just try to do it ASAP.
 #
 # Redis supports three different modes:
 #
 # no: don't fsync, just let the OS flush the data when it wants. Faster.
 # always: fsync after every write to the append only log. Slow, Safest.
 # everysec: fsync only one time every second. Compromise.
 #
 # The default is "everysec", as that's usually the right compromise between
 # speed and data safety. It's up to you to understand if you can relax this to
 # "no" that will let the operating system flush the output buffer when
 # it wants, for better performances (but if you can live with the idea of
 # some data loss consider the default persistence mode that's snapshotting),
 # or on the contrary, use "always" that's very slow but a bit safer than
 # everysec.
 #
 # More details please check the following article:
 # http://antirez.com/post/redis-persistence-demystified.html
 #
 # If unsure, use "everysec".
  
 # appendfsync always 	//一写指令就备份一次。这样做虽然安全，但是系统性能会降低。不推荐使用
 appendfsync everysec	//每一秒中备份一次。不管一秒钟变化了多少key，只备份一次，性能得到一定的保护
 # appendfsync no 		//会查看当前服务器状态,如果状态良好,就进行备份(随机)这种备份方式数据是没有保证的
```

##### fsync

* 在突然宕机的情况下AOF日志还没有来得及刷到磁盘中,这个时候linux提供提供fsync函数将指定文件强制从内核刷到磁盘,只要redis进程实时调用fsync函数就可以保证AOF日志不丢失(fsync是磁盘IO操作)

##### AOF原理

* AOF持久化是存储redis服务器的修改指令记录顺序,记录在一个日志里,redis重启时就读取这个日志文件，对所有操作进行回放达到数据恢复的目的

* redis在收到客户端指令后首先进行参数校验进行逻辑处理,在没有问题的情况下将修改指令存储到AOF日志中,也就是说先执行指令在进行日志存盘操作,在长期的操作工程中AOF日志会越来越长,机器宕机会恢复的时间将变的越来越长,所以需要对AOF日志瘦身

##### AOF瘦身

* redis提供BGREWRITEAOF指令对aof瘦身，原理就是开辟子进程对目前内存进行遍历转换为一系列redis操作指令序列化到AOF日志中,序列化完毕后再将操作期间发生的增量AOF日志追加到这个新的AOF日志中,代替老旧的AOF日志,瘦身结束

* 当AOF日志大小增加指定百分比时，Redis能够自动重写日志文件，隐式调用BGREWRITEAOF

```
Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling                                                          
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100 //当前AOF日志是上次日志的二倍
auto-aof-rewrite-min-size 64mb	//rewrite 最小大小
```

#### Redis 4.0 混合持久化

重启redis时会如果使用rdb来恢复内存状态会丢失大量数据,而使用AOF日志重放则性能会慢很多,Redis4.0则是将rdb文件的内容和增量的AOF日志存放在一起,这时AOF日志将不是全量日志而是持久化开始到持久化结束这段时间的,通常这部分文件很小,于是在redis重启时先加载rdb文件再重放AOF日志
