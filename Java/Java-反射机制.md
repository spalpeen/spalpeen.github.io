## 反射


#### 获取Class对象

1.每个类被加载之后系统会为该类生成一个对应的Class对象，通过该Class对象就会访问到JVM中的这个类，在java中获取Class对象通常有三种方式：

    1）使用Class类的froName(String clazzName)静态方法，该方法需要传入字符串参数，该字符串参数的值是某个类的全限定类名，也就是带包信息

    2）调用某个类的class属性来获取该类对用的class对象，例如：Nenmo.class将会返回Nenmo类的对象

    3）调用某个类的getClass()方法，该方法是java.lang.Object类中的一个方法，所以所有的java对象都可以调用该方法，该方法将会返回对象所属类对应的Class对象

2.对于第一种方式和第二种方式都是直接根据类来获取该类的Class对象，相比之下，第二种方式有如下优势：

    1）代码更安全，程序在编译阶段就可以检查需要访问的Calss对象是否存在

    2）程序性能更好，因为这种方式无需调用方法，所以性能更好

3.Class类提供了大量的实例方法来获取该Class对象所对应类的详细信息，Class类大概包含如下方法：

Class类所包含的构造器：

1）Connstructor<T> getConnstructor(Class<?>...parameterTypes)：返回此Class对象对应类的，带指定形参列表的public 构造器

2）Connstructor<?>[] getConnstructor()：返回此Class对象所对应类的所有public构造器

3）Connstructor<T> getDeclaredConstructor(Class<?>...parameterTypes)：返回此Class对象对应类的，带指定形参列表的构造器，与构造器访问权限无关

4）Connstructor<?> getDeclaredConstructor()：返回此Class对象对应类的所有构造器，与构造器的访问权限无关

Class对应类所包含的方法

1）Method getMethod(String name,Class<?>...parameterTypes)：返回此Class对应类的，带指定形参列表的public方法

2）Method[] getMethods()：返回此Class对象所表示的类的所有public方法

3）Method getDeclaredMethod(String name,Class<?>...parameterTypes)：返回此Class对象对应类的，带指定形参列表的方法，与访问权限无关

4）Method[] getDeclaredMethod()： 返回此Class对象对应类的所有方法，与访问权限无关

```
public class FansheTest {
    private FansheTest() {
        System.out.println("不带形参的构造器");
    }

    public FansheTest(String name) {
        System.out.println("带有形参的构造器");
    }

    public void dance() {
        System.out.println("不带形参的方法");
    }

    public void dance(String name) {
        System.out.println("带有形参的方法");
    }


    public static void main(String[] args) throws Exception {
        Class<FansheTest> clazz = FansheTest.class;
        Constructor[] constructors = clazz.getDeclaredConstructors();
        System.out.println("FansheTest类的全部构造方法：");
        for (Constructor constructor : constructors) {
            System.out.println(constructor);
        }

        //获取全部public方法
        Method[] methods = clazz.getMethods();
        System.out.println("FansheTest类的全部方法：");
        for (Method method : methods) {
            System.out.println(method);
        }

        //获取指定方法
        System.out.println(clazz.getMethod("dance",String.class));
    }

}

```

#### 使用反射生成并操作对象

通过反射来生成对象有两种方式：

1）使用Class对象的newInstance()方法来创建该Class对象对应类的实例，这种方式要求该Class对象的对应类有默认构造器，而执行newInstance()方法时实际上是利用默认构造器来创建该类的实例


```
public class ObjectCreate {
    private Object createObject(String clazzName)throws InstantiationException ,
            IllegalAccessException,ClassNotFoundException{
        Class<?> clazz = Class.forName(clazzName);
        return clazz.newInstance();
    }
}
```

2）先使用Class对象获取指定Constructor对象，在调用Constructor对象的newInstance()方法来创建该Class对象对应类的实例，通过这种方式可以选择使用指定的构造器来创建实例

```
public class ObjectCreate {
    private Object createObject(String clazzName)throws Exception{
        Class<?> clazz = Class.forName(clazzName);
        Constructor constructor = clazz.getConstructor(String.class);
        return constructor.newInstance();
    }
}
```




