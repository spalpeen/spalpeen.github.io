## java多线程

#### 要点：

1.线程与进程的区别及多线程的优势

2.创建线程的两种方式

3.三种创建线程方式对比

4.线程生命周期，死亡的几种情况

5.控制线程的常用方法

6.线程同步

7.线程池

8.线程相关类


#### 线程与进程的区别及多线程的优势

进程是系统分配资源调度的最小单位，当一个程序进入内存运行时，即变成一个进程

1.进程有以下三个特性：

* 进程是系统中独立存在的实体，可以拥有自己独立的资源，每个进程都有自己私有的地址空间，没有允许的情况下一个用户进程不可以直接访问其他进程的地址空间

* 进程与程序的区别在于程序只是一个静态的指令集合，而进程是一个正在系统中活动的指令集合，进程拥有自己的声明周期及各种状态

* 多个进程可以在单个处理器上并发执行，多个进程间不相互影响


线程也被称作轻量级进程，线程是进程的执行单元，就像进程在操作系统中的地位，线程在程序中是独立的，并发的执行流，当进程被初始化后，主线程就被创建了

2.线程有以下四个特性：

* 线程是进程的重要组成部分，一个进程可以拥有多个线程，一个线程必须有一个父进程，线程可以拥有自己的堆栈，自己的程序计数器和自己的局部变量，但不能拥有系统资源，只能与父进程中的其他线程共享该进程所拥有的全部资源

* 线程可以完成一定的任务，可以与其他线程共享父进程中的共享变量及部分环境，相互之间协同来完成进程所需要完成的任务

* 线程是独立运行的，它并不知道进程中是否还有其他线程的存在，线程总是抢占式的，也就说当前线程在任何时候都可能被挂起，以便另外一个线程运行

* 一个线程可以创建和撤销另一个线程，同一进程中的多个线程可以并发执行

线程在程序中是独立的，并发的执行流，与分隔的进程相比，进程中线程之间的隔离程度要小，它们共享内存，文件句柄和其他每个进程应有的状态

3.多线程有以下三个优点：

* 进程之前不能共享内存，但是线程之前共享内容很容易

* 系统创建进程需要为该进程重新分配系统资源，但创建线程代价要小得多，因此使用多线程来实现多任务并发比多进程效率高

* java内置了多线程功能支持，而不是作为底层操作系统的调度方式


总结：操作系统可以执行多个任务，每个任务就是进程，进程可以同时执行多个任务，每个任务就是线程


#### 创建线程的两种方式

1.继承Thread类来创建线程类

* 通过继承Thread类来创建多线程步骤：

1）定义Thread类的子类，并重写run方法，该run方法的方法体就代表了线程需要完成的逻辑，run方法称为线程执行体

2）创建Thread子类的实例，既创建线程对象

3）调用线程对象的start()方法启动该线程

```
public class ThreadTest extends Thread {

    private int i;

    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(getName() + " " + i);
        }
    }

    public static void main(String[] args) {//主线程
        System.out.println(Thread.currentThread().getName());
        new ThreadTest().start();//一个线程
        new ThreadTest().start();//另一个线程
    }
}
```


使用继承Thread类方法来创建线程类时，多个线程之前无法共享线程类的实例变量

2.实现Runnable接口来创建线程类

* 实现Runnable接口来创建并启动线程类的步骤：

1）定义Runnable接口的实现类，并重写该接口的run方法，该run方法的方法体就代表了线程需要完成的逻辑

2）创建Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象


```
//创建runnable实现类的实例
RunnableTest runnableTest = new RunnableTest();
//以此实例作为Thread的target来创建Thread对象
new Thread(runnableTest);
//new Thread(runnableTest,"这是起名字的线程");//为线程起名字
```


3）调用线程对象的start()方法启动该线程

```
public class RunnableTest implements Runnable {

    private int i;

    @Override
    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        //创建runnable实现类的实例
        RunnableTest runnableTest = new RunnableTest();
        //以此实例作为Thread的target来创建Thread对象
        new Thread(runnableTest, "线程1").start();//一个线程
        new Thread(runnableTest, "线程2").start();//另一个线程
    }
}
```


使用Runnable接口来创建多线程时，多个线程之前可以共享线程类的实例变量，Thread类的作用就是把run方法包装成线程执行体


3.使用Callable和Future创建线程

* java5开始java提供了Callable接口，Callable接口提供了一个call()方法可以作为线程执行体，call()方法比run()方法功能更加强大

* call()方法可以声明抛出异常

* call()方法还有一个返回值，call()方法并不是直接调用，它是作为线程执行体被调用，java5提供了Future接口来代表call()方法的返回值，并为Future接口提供了一个实现类FutureTask，该实现还实现了Runnable接口，可以作为Thread类的target

    1）boolean cancel(boolean mayInterruptIfRunning):试图取消改Future里关联的Callable任务

    2）V get():返回Callable任务里call()方法的返回值，调用该方法将导致程序阻塞，必须等到子线程结束后才会得到返回值

    3）V ger(long timeout,TimeUnit unit):返回Callable任务里call()方法的返回值，该方法最多阻塞timeout的unit的指定时间，时间内没有返回将会抛出Timeout异常

    4）boolean isCancelled():如果在Callable任务完成前被取消，返回true

    5）boolean isDone():如果在Callable任务已完成，返回true

* Callable接口有泛型限制，Callable接口里的泛型形参类型与call方法返回值类型相同，而却Callable接口是函数式接口，因此可使用Lambda表达式创建Callable对象

* 创建并启动Callable接口线程的步骤如下：

1）创建Callable接口实现类，并实现call()方法，该call()方法将作为线程执行体，且该call()方法有返回值，再创建Callable实现类的实例

2）使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象call()方法的返回值

3）使用FutureTask对象作为Thread对象的target创建并启动新线程

4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值


```
public class CallableTest {

    public static void main(String[] args) {
        //创建Callbale对象
        //CallableTest callableTest = new CallableTest();
        //使用Lambda表达式创建Callable<Integer>对象，所以无需实现Callable
        //使用FutureTask类来包装Callable对象
        FutureTask<Integer> task = new FutureTask<Integer>((Callable<Integer>) () -> {
            int i = 0;
            for (; i < 100; i++) {
                System.out.println(Thread.currentThread().getName() + " " + i);
            }
            return i;
        });

        System.out.println(Thread.currentThread().getName() + " ");
        new Thread(task, "这个线程有返回值").start();

        try {
            System.out.println("子线程返回值" + task.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

```


#### 三种创建线程方式对比

继承Thread类和实现Runnable和Callable类都可以创建多线程，Runnable接口与Callable接口基本方式相同，只是Callable定义的方法有返回值能抛出异常，所以归为一类

1.实现Runnable接口与Callable接口方式创建多线程优缺点：

* 线程实现类还可以集成其他类

* 多个线程共享一个target对象，非常适合多个线程来处理同一资源的情况从而将CPU，代码和数据分开，形成清晰模型

* 想要访问线程必须使用Thread.currentThread()方法

2.继承Thread类方式创建多线程优缺点：

* 已经集成Thread类不能继承其他类

* 简单，访问线程直接使用this就可以获取


#### 线程生命周期，死亡的几种情况

1.线程有5种状态：新建（New），就绪（Runnable），运行（Running），阻塞（Blocked），死亡（Dead）

2.当线程启动后，它不可能一直霸占CPU，CPU需要在多条线程之前切换，于是线程的状态也会在运行，阻塞之间切换

3.新建和就绪

    1）当使用new关键字来创建一个线程后，该线程就处于新建状态，此时java虚拟机为其分配内存，并初始化其成员变量的值

    2）当线程对象调用start()方法后，该线程处于就绪状态，java虚拟机将为其创建方法调用栈和程序计数器，处于这个状态的线程并没有开始运行，只是表示可以运行，至于何时运行取决于JVM里线程调度器的调度

    3）启动线程使用的是start()方法而不是run()方法，如果直接调用run()方法将把run()方法当作普通方法来执行

4.运行和阻塞

    1）如果处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于运行状态

    2）如果机器只有一个CPU，那么任何时刻只有一个线程在运行状态

    3）线程在运行过程中需要被中断，目的是使其他线程获得执行的机会

    4）对于抢占式策略而言，系统会给每一个可执行线程一个时间段来执行任务，当时间段用完后，就会被剥夺所占据的资源，在选择下一个线程时系统会考虑线程的优先级

    5)线程从阻塞状态只能进入就绪状态，无法直接进入运行状态

    6）当正在执行的线程被阻塞后其他线程可以获得执行机会，被阻塞的线程将会在合适的时候重新进入就绪状态

5.死亡

    1）run()方法，call()方法执行结束

    2）线程抛出一个未捕获异常或Error

    3)直接调用stop()方法来结束--该方法容易导致死锁，不推荐使用

    4）当主线程结束时，其他线程不受影响，并不会随之结束，一旦子线程启动起来就拥有和主线程相同的地位，不会受主线程影响

    5）不要对一个死亡的线程调用start()方法，死亡就是死亡不可最为线程执行

    6）调用isAlive()方法，就绪，运行，阻塞状态会返回true，新建和死亡状态将返回false


#### 控制线程的常用方法

1.join线程

    1）线程调用join()方法让一个线程等待另一个线程完成，当在某个线程中调用join()方法，调用线程将被阻塞，直到被join()方法加入的join()线程执行完为止

    2）join()方法通常由使用线程的程序调用，以将大问题划分成许多小问题，每个小问题分配一个线程，当所有小问题都得到处理后，再调用主线程操作


```
public class JoinThread extends Thread {

    public JoinThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        //启动子线程
        new JoinThread("线程").start();
        for (int i = 0; i < 100; i++) {
            if(i ==20){
                JoinThread joinThread = new JoinThread("被Join线程");
                joinThread.start();
                try {
                    joinThread.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```


    3）join()方法有三种重载形式

        1）join()：被join线程执行完成

        2）join(long millis)：等待被join线程最长时间 单位毫秒

        3）join(long millis,int nanos)：等待被join线程最长时间 单位毫秒+毫微秒


2.后台线程

    1）后台线程任务是为其他线程提供服务，又称守护线程，精灵线程，JVM的垃圾回收线程就是典型的后台线程

    2）后台线程有一个特征：如果所有的前台线程都死亡，后台线程会自动死亡

    3）调用Thread的setDaemon(true)方法可将指定线程设置成后台线程，当整个虚拟机中只剩下后台线程，程序就没有执行的必要了所以虚拟机也就退出了

    4）将某一线程设置为后台线程必须在这个线程启动之前，也就是调用start()方法之前

```
public class DaemonThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        DaemonThread daemonThread = new DaemonThread();
        daemonThread.setDaemon(true);
        daemonThread.start();
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```

3.sleep线程睡眠

    1）如果要某一正在执行的线程暂停一段时间，并进入阻塞状态，则可以通过Thread类的sleep()方法来实现，sleep()方法有两种重载形式

        static void sleep(long millis)：让当前正在执行的线程暂停millis秒，并进入阻塞状态，该方法受到系统计时器和线程调度器的精度与准确度影响

        static void sleep(long millis,int nanos)：多个毫微秒

    2）当当前线程调用sleep()方法进入阻塞状态后，其睡眠时间内不会获得执行机会，即使系统中没有可执行线程

```
public class SleepThread {

    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 100; i++) {
            Thread.sleep(1000);
        }
    }
}
//暂停1s
```


4.yield线程让步

    1）yield()方法与sleep()方法有些相似，都是可以让当前正在执行的线程暂停，不同是yield()方法会暂停一下，不会阻塞，而是将该线程转入就绪状态，所以可能出现暂停后线程调度器又将其调用出来执行

    2）yield()方法只会给优先级相同或者更高优先级的线程执行机会

```
public class YieldThread extends Thread {
    public void run() {

        for (int i = 0; i < 100; i++) {
            if (i == 20) {
                yield();
            }
        }
    }

    public static void main(String[] args) {
        YieldThread yieldThread1 = new YieldThread();
        yieldThread1.start();
        YieldThread yieldThread2 = new YieldThread();
        yieldThread2.start();
    }
}
```

5.改变线程优先级

    1）每个线程的默认优先级都与创建他的父线程优先级相同

    2）Thread提供setPriority(int new Priority)，getPriority()方法来设置和返回指定线程的优先级，其中setPriority()参数可以是个整数范围在0~10之间，也可使使用静态常量

    MAX_PRIORITY //1O

    MIN_PRIORITY //1

    NORM_PROORITY //5


```
public class PriorityThread extends Thread {
    public PriorityThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("priority" + getPriority() + " i=" + i);
        }
    }

    public static void main(String[] args) {

        Thread.currentThread().setPriority(8);
        System.out.println(Thread.currentThread().getPriority());

        for (int i = 0; i < 100; i++) {
            if (i == 10) {
                PriorityThread priorityThreadLow = new PriorityThread("low");
                priorityThreadLow.setPriority(Thread.NORM_PRIORITY);
                priorityThreadLow.start();
            }

            if (i == 20) {
                PriorityThread priorityThreadH = new PriorityThread("hight");
                priorityThreadH.setPriority(Thread.MAX_PRIORITY);
                priorityThreadH.start();
            }
        }

    }
}
```


#### 线程同步

1.java多线程安全支持同步方法synchronized关键字来修饰某个方法，该方法称为同步方法

```
public class Account {

    private String accountNo;
    private double balance;

    public synchronized void draw(double drawAmount) {
        if(balance >= drawAmount){
            System.out.println("Success" + accountNo);
            try {
                Thread.sleep(1);
                balance -= balance-drawAmount;
            }catch (Exception e){
            }
        }
        System.out.println("Error" + accountNo);
    }
}
```

2.对于synchronized修饰的实例方法（非static方法）而言，无需显示指定同步监视器，同步方法的同步监视器是this，也就是调用该方法的对象

3.通过使用同步方法可以非常方便地实现线程安全的类，线程安全的类具有如下特称：

    1）该类的对象可以被多个线程安全的访问

    2）每个线程调用该对象的任意方法之后都将得到正确结果

    3）每个线程调用该对象的任意方法之后，该对象依然保持合理状态

4.synchronized关键字可以修饰方法，可以修饰代码块，但是不能修饰构造器，成员变量等

5.释放同步监视器的锁定

    1）当前线程的同步方法，同步代码块执行结束，当前线程即释放同步监视器

    2）当前线程在同步代码块，同步方法中使用了break,return

    3)当前线程在同步代码块，同步方法中出现了Error或Exception

    4)当前线程执行同步代码块，同步方法时，程序执行了同步监视器的wait()方法

6.线程不会释放同步监视器

    1）当前线程执行同步代码块，同步方法时，程序调用Thread.sleep()，Thread.yield()

    2）程序执行同步代码块时，其他线程调用了该线程的suspend()方法将该线程挂起

7.同步锁（Lock）

    1）java 5开始提供了一种功能更强大的线程同步机制，通过显示定义同步锁对象来实现同步

```
public class Account {
    //可重入锁
    private final ReentrantLock reentrantLock = new ReentrantLock();
    private String accountNo;
    private double balance;

    public void draw(double drawAmount) {
        reentrantLock.lock();
        try {
            if (balance >= drawAmount) {
                System.out.println("Success" + accountNo);
                try {
                    Thread.sleep(1);
                    balance -= balance - drawAmount;
                } catch (Exception e) {
                }
            }
            System.out.println("Error" + accountNo);
        }finally {
            reentrantLock.unlock();
        }
    }
}
```

    2）Lock提供比synchronized方法和synchronized代码块更广泛的锁定操作

    3）Lock是控制多线程共享资源进行访问的工具，通常锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前需要先获得Lock对象


#### 线程池

1.系统启动一个新线程涉及到操作系统交互所以成本是比较高的，所以引入线程池的概念，线程池在启动时创建大量空闲的线程，程序将一个Runnable对象或Callable对象传给线程池，线程池就会启动一个线程来执行他们的run()方法或者call()方法，当run()方法或者call()方法直接结束后，该线程并不会死亡，而是重新进入线程池中变成空闲状态

2.线程池可以有效的控制系统中并发线程的数量，当系统中包含大量并发线程时，会导致系统性能剧烈下降，甚至导致JVM崩溃，而线程池的最大线程数可以控制系统中并发线程数不会超过此数

3.java 5以前需要手动实现自己的线程池，从java5开始支持java内建线程池，Executors工厂类来产生线程池，该工厂类包含以下几个静态工厂方法来创建线程池：

    1）newCachedThreadPool(int nThreads)：创建一个具有缓存功能的线程池，这些线程将被缓存在线程池中

    2）newFixedThreadPool(int nThreads)：创建一个可重复的，具有固定线程数的线程池

    3）newSingleThreadExecutor()：创建一个单线程的线程池

    4）newScheduledThreadPool(int corePoolSize)：创建具有指定线程数的线程池，可以指定延迟后执行线程任务

    5）newSingleThreadScheduledExecutor()：创建一个单线程的线程池，可以指定延迟后执行线程任务

    6）ExecutorService newWorkStealingPool(int parallelism)：创建持有足够的线程的线程池来支持给定的并行级别，该方法还会使用多个队列来减少竞争

    7）ExecutorService newWorkStealingPool()：与上一个方法相比值与CPU相同


4.使用线程池来创建线程任务的步骤：

1）调用Executors类的静态工厂方法创建一个ExecutorService对象

2）创建Runnable或者Callable实现类的实例作为线程执行任务

3）调用ExecutorService对象的submit()方法来提交Runnable或者Callable实例

4）关闭线程池调用ExecutorService对象的showdown()方法


```
public class ThreadPoolTest {

    private static ExecutorService threadPool = Executors.newFixedThreadPool(100);

    public static void main(String[] args) {

        Future<Integer> googleFuture = threadPool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return getSuggestByGoogle();
            }
        });

        //facebook suggest
        Future<Integer> fbFuture = threadPool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return getSuggestByFaceBook();
            }
        });

        threadPool.shutdown();
    }

    private static Integer getSuggestByGoogle(){
        return 1;
    }

    private static Integer getSuggestByFaceBook(){
        return 2;
    }
}

```

#### 线程相关类

1.java为线程安全提供了一些工具类如ThreadLocal类，ThreadLocal类代表一个线程局部变量，通过把数据放在ThreadLocal中就可以让每个线程创建一个该变量的副本，每个线程都可以独立的改变自己的副本，而不会和其他线程副本冲突，从而避免并发访问的线程安全问题

2.ThreadLocal类提供了三个public方法：

    1）T get()：返回此线程局部变量中当前线程副本的值

    2）void remove()：删除此线程局部变量中当前线程的值

    3）void set(T value)：设置此线程局部变量中当前线程副本的值


```
public class ThreadLocalTest extends Thread {

    private Account account;

    public ThreadLocalTest(Account account, String name) {
        super(name);
        this.account = account;
    }


    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            if (i == 5) {
                account.setThreadLocal(getName());
            }
            System.out.println(account.getThreadLocal() + "i=" + i);
        }
    }
}

```
