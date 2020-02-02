## 异常处理

#### 要点

* 异常

* 异常类继承体系

* finally 块

* 自定义异常类


#### 异常

* java把非正常情况分为两种，异常（Exception）和错误（Error），它们都继承Throwable父类

* Error错误一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，动态链接失败，这些错误是无法恢复或不可捕获的，将导致应用程序中断

* 异常分为两种 Runtime异常和Checked异常，Runtime异常无需处理，Checked异常都是可以在编译阶段被处理的异常，所以它强制程序处理所有的Checked异常

* 常见异常情况：

    1.IndexOutOfBoundsException:数组越界异常
    
    2.NumberFormatException:数字格式异常
    
    3.ArithmeticException:算数异常

    4.NullPointerException:空指针异常
    
* 捕获异常时一定要记住先捕获小的异常，再捕获大的异常

* Java7开始提供多异常捕获

    ```
     try{
        ...
     }cache(IndexOutOfBoundsException|NumberFormatException|ArithmeticException e){
        ...
     }
        
     //捕获多种异常时，异常变量有隐式的final修饰，程序不能对异常变量重新赋值
        
     //以下错误
     try{
        ...
     }cache(IndexOutOfBoundsException|NumberFormatException|ArithmeticException e){
            e = new NumberFormatException("异常");
            ...
     }
    ```
    
* 如果需要了解异常相关信息，可以通过异常对象的常用方法

    1.getMessage():返回异常描述字符串
    
    2.printStackTrace():标准错误输出异常的跟踪栈信息
    
    3.printStackTrace(PrintStream s):输出流的方式输出异常的跟踪栈信息
    
    4.getStackTrace():返回异常跟踪栈信息
    
#### 异常类继承体系


  ![异常继承体系](../static/异常继承体系.png)

#### finally 块

有些时候需要使用一些物理资源，比如数据库连接，网络连接，磁盘文件等，这些资源必须显示回收，而java的垃圾回收机制不会回收任何的物理资源，垃圾回收只能回收堆内存中对象所占用的内存
所以为了保证一定能回收try块中打开的物理资源，异常处理机制提供了finally块，finally语句总是会执行

* 尽量避免在finally块里使用return，throw等导致方法终止的语句，否则可能会出现一些奇怪的现象

* java7以后允许在try后面跟一对圆括号，圆括号声明的资源会被自动关闭，这些资源类必须实现AutoCloseable或Closeable接口，实现这两个接口必须实现close方法

#### 自定义异常类  
    
* 自定义的异常类都应该继承Exception基类，如果希望自定义Runtime异常那么就应该继承RuntimeException基类

* 定义异常类通常需要提供两个构造器，一个是无参数构造器，另外一个是带字符串描述异常信息的构造器（getMessage()返回值）

```
    public class NenmoException extends Exception{
        public NenmoException(){}
        
        public NenmoException(String msg){
            super(msg);
        }
    }
```

#### 异常机制的效率要流程控制差 所以不要使用异常处理来代替正常的操作流程