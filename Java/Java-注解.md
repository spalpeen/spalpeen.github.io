## Annotation 注解

要点：

1. Annotation意义

2. 5个基本Annotation

3. JDK元Annotation

4. 自定义Annotation

##### Annotation意义

* Annotation是代码里的特殊标记，这些标记可以在编译，类加载，运行时读取，并执行相应的处理

* 通过使用注解可以在不改变原有逻辑的基础上在源文件中嵌入一些补充信息，如代码分析工具，监控

* Annotation是一个接口，程序可以通过反射来获取指定元素的Annotation对象，然后通过Annotation对象来取得注解里的元数据

##### 基本Annotation

存在于java.lang包下

* @Override ：用来指定方法覆载，强制子类必须覆盖父类方法，只能修饰方法不能修饰其他

* @Deprecated ：用来表示某个程序元素（类，方法等）已过时，其他程序使用发出警告

```
class Nenmo
{
    @Deprecated
    public void qianqian()
    {
        System.out.println("qianqian dance");
    }
}

public class Boss()
{
    public static void main(String[] args)
    {
        new Nenmo().qianqian();
    }
}
//编译时将发出警告
```

* @SuppressWarnings ：抑制编译器警告，作用于该程序元素的所有子元素

* @SafeVarargs ：抑制堆污染警告

* @FunctionalInterface ：用来指定某个接口必须是函数式接口，只能修饰接口，不能修饰其他
（函数式接口：接口中只有一个抽象方法，可以包含多个默认方法及多个static方法）

##### JDK元Annotation

* JDK除了在java.lang下提供了5个基本Annotation之外，还在java.lang.annotation包下提供了6个Meta Annotation,其中5个元Annotation都用于修饰其他的Annotation定义

* @Retention :只能用于修饰Annotation定义，用于指定被修饰的Annotation可以保留多长时间，@Retention包含一个RetentionPolicy类型的value成员变量，所以使用@Retention时必须为该value成员变量指定值

    value成员变量的值只能是如下三个：
    
    1. RetentionPolicy.CLASS :编译器将把Annotation记录在.class中，当运行java程序，JVM不可获取Annotation信息，这是默认值
    
    2. RetentionPolicy.RUNTIME :编译器将把Annotation记录在.class中，当运行java程序，JVM可获取Annotation信息，程序可通过反射获取该Annotation信息
    
    3. RetentionPolicy.SOURCE : Annotation只保留在源代码中，编译器直接丢弃这种Annotation

* @Target : 只能用于修饰Annotation定义，用于指定被修饰的Annotation能用于修饰哪些程序单元，@Target元Annotation也包含一个value成员变量，值如下

    value成员变量值：
    
    1. ElementType.ANNOTATION_TYPE:指定该策略的Annotation只能修饰Annotation
    
    2. ElementType.CONSTRUCTOR:指定该策略的Annotation只能修饰构造器
    
    3. ElementType.FIELD:指定该策略的Annotation只能修饰成员变量

    4. ElementType.LOCAL_VARIABLE:指定该策略的Annotation只能修饰局部变量

    5. ElementType.METHOD:指定该策略的Annotation只能修饰方法定义
    
    6. ElementType.PACKAGE:指定该策略的Annotation只能修饰包定义
    
    7. ElementType.PARAMETER:指定该策略的Annotation可以修饰参数
    
    8. ElementType.TYPE:指定该策略的Annotation可以修饰类，接口，枚举
```
@Target(ElementType.FIELD)
public interface Nenmo{}
```    

* @Documented ：用于指定被该元Annotation修饰的Annotation类将被javadoc工具提取成文档

* @Inherited ：用于指定被他修饰的Annotation具有继承性

##### 自定义Annotation

* 定义新的Annotation类型使用关键字@interface


```
//定义
public @interface Nenmo
{
    //成员变量以方法形式来定义
    String name();
    //String name() default "qianqian"; 定义初始值
    int age();
    //int age() default "20";
}

//使用时为成员变量指定值(指定值后默认值无效)
public class Boss
{
    @Nenmo(name="qianqian",age="20")
    public void dance()
    {
    }
}

//默认情况下Annotation可用于修饰任何程序元素，包括类，接口，方法等
```

* 没有定义成员变量的Annotation类型称为标记，如

```
//定义
public @interface Nenmo
{
}
```

* 包含成员变量的Annotation类型称为元数据Annotation

* 提取Annotation信息

    1.使用Annotation修饰类，方法，成员变量等成员后不会自动生效，必须提供相应的工具来提取并处理Annotation信息
    
    2.java5后版本在java.lang.reflect包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素，主要有以下实现类
    
        Class:类定义
        Constructot:构造器定义
        Field:类的成员变量定义
        Method:类的方法定义
        Package:类的包定义
    
    3.java5开始 java.lang.reflect包所提供的反射API增加了读取运行时Annotation的能力，只有当定义Annotation时使用了@Retention(RetentionPolicy.RUNTIME)修饰，该Annotation才会在运行时可见，JVM才会在装在.class文件时读取保存在class文件中的Annotation

// todo list

1.提取Annotation信息

2.演示实例