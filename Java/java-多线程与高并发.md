## 多线程与高并发

1.什么是线程

2.线程常用方法


3.启动线程的5种方式

    1)new Thread().start()
    
    2)new Thread(实现Runnable).start()

    3)new Thread(()->{
        ...
    }).start()
    
    4)new Thread(new FutureTask<String> (new 实现Runnable接口的类)).start()
    
    5)ExecutorService server  = Executors.newCacheThreadPool();

4.线程同步的基本概念

    synchronized锁的是对象不是代码

5.synchronized关键字的字节码原语

    1)

6.volatile关键字的字节码原语

7.synchronized与volatile的硬件级实现

8.无锁，偏向锁，轻量级锁，重量级锁的升级过程

9.内存屏障的基本概念

10.JVM规范如何要求内存屏障

11.硬件层级内存屏障如何帮助java实现高并发

12.线程间通讯的8种解法
