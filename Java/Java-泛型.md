## 泛型

Java集合有个缺点，当把一个对象丢进集合里后，集合就会忘记这个对象的数据类型，当再次取出该对象时，该对象的编译类型就变成了Object类型（运行时类型不变）

集合对元素类型没有任何限制

把对象丢进集合后，集合丢失了对象的状态信息，集合只知道它盛装的是Object，取出集合元素后通常还需要强制类型转换，既增加了程序的复杂度又容易引发ClassCaseException

Java5以后引入了参数化类型概念，也就是泛型，所谓泛型就是允许在定义类，接口，方法时使用类型形参，这个类型形参将在声明变量，创建对象，调用方法时动态的指定


#### Java7以后采用菱形语法

```
List<String> strList = new ArrayList<>();
```


#### 定义泛型接口，类

```
//List接口代码片段
public interface List<E>
{
    //在该接口里，E可作为类型使用
    
    //使用E作为参数类型
    void add(E x);
    
    Iterator<E> iterator;
}
```

```
//Iterator接口代码片段
public interface Iterator<E>
{
    //E 作为类型使用
    E next();
    
    boolean hasNext();
}

```

```
//Map接口代码片段
public interface Map<K , V>
{
    //K,V作为类型使用    
    Set<K> keySet();
    V put(K key,V value);
}

```
包含泛型声明的类型可以在定义变量，创建对象时传入一个类型参数，从而可以动态的生成无数个逻辑上的子类，但这种子类物理上不存在

可以为任何类，接口增加泛型声明，并不是只有集合类才可以使用泛型声明，虽然集合类是泛型的重要使用场所

```
public class Girl<T>
{
    private T nenmo;
    
    public Nenmo(T nenmo)
    {
        this.nenmo = nenmo;
    }
    
    public show(T nenmo)
    {
        System.out.println(nenmo."dance");
    }
    
    public static void main(String[] args)
    {
        Girl<String> nenmo = new Gril<>("qianqian"); 
    }
}
```

程序定义了一个带泛型声明的类，当创建带泛型声明的自定义类，为该类定义构造器时，构造器还是原来的类名，不要增加泛型声明

#### 从泛型类派生子类

* 当创建了带泛型声明的接口，父类之后，可以为该接口创建实现类，或从该父类派生子类，需要指出的是，当使用这些接口，父类时不能再包含类型形参

```
//错误演示
public class Nenmo extends Gril<T>{}
```

* 定义类，接口，方法时可以声明类型形参，使用类，接口，方法时应该为类型形参传入实际的类型

```
public class Girl<T>
{
    private T nenmo;
    
    public Nenmo(T nenmo)
    {
        this.nenmo = nenmo;
    }
    
    public show(T nenmo)
    {
        System.out.println(nenmo."dance");
    }
}

public class Nenmo extends<String>{}
```

* 定义方法时可以声明数据形参，调用方法时必须为这些数据形参传入实际的数据

```
Girl<String> nenmo = new Gril<>();
nenmo.show("qianqian");
```

#### 泛型类

* `ArrayList<String>`对象只能添加String对象作为集合元素，但实际上系统并没有为`ArrayList<String>`生成新的class，而且也不会把`ArrayList<String>`当成新类来处理


```
List<String> l1 = new ArrayList<>();

List<Integer> l2 = new ArrayList<>()
System.out.println(l1.getClass() == l2.getClass());
//true
```
不管泛型的实际类型参数是什么，它们在运行时总有同样的类（class）,不管为泛型的类型形参传入哪一种类型的实参，它们依然被当成同一个类处理，在内存中也只占用一块内存空间

因此在静态方法，静态初始化块，或者静态变量的声明和初始化块中不允许使用类型形参

```
public class Girl<T>
{
    static T nenmo;//错误，不能在静态变量声明中使用类型形参
    
    public static void dance(T nenmo){}//错误，不能在静态方法声明中使用类型形参
}
```

由于系统中并不会真正生成泛型类，所以instanceof运算符后不能使用泛型类

```
Collecction<String> cs = new ArrayList<>();
if(cs instanceof ArrayList<String>){}
```

#### 类型通配符

* 当使用一个泛型时（包括声明变量和创建变量两种情况），都应该为这个泛型类传入一个类型实参，如果没有传入类型实际参数，编辑器会提出泛型警告

```
public void dance(List nenmos)
{
    for(int i; i < nenmos.size; i ++)
    {
        System.out.println(nenmo.get(i));
    }
}
//此处使用List接口时没有传入实际类型参数将会引起警告
```
修改：

```
public void dance(List<Object> nenmos)
{
    for(int i; i < nenmos.size; i++)
    {
        System.out.println(nenmo.get(i));
    }
}

//调用该方法传入的形参类型不是我们所期望的 将会引起List<String> 这将会产生编译错误，这表明List<String> 对象不能当成List<Object>对象使用，也就是说List<String>并不是List<Object>的子类
```
#### 类型通配符

* 为了表示各种泛型List的父类，可以使用类型通配符，符号 ：？，将一个问号作为类型实参传给List集合，写作：List<?> 意思是元素类型未知的List,这个问号被称为通配符，它的元素类型可以匹配任何类型，

```
public void dance(List<?> nenmos)
{
    for(int i; i < nenmos.size; i++)
    {
        System.out.println(nenmo.get(i));
    }
}
```

* 类型通配符上限

当使用List<?> 可表示这个list集合可以是任何泛型List的父类，但如果只希望它代表某一类的泛型List的父类

```
public void dance(List<? extends Girl> nenmos)
{
    for(int i; i < nenmos.size; i++)
    {
        System.out.println(nenmo.get(i));
    }
}
```

* 类型形参上限

Java泛型不仅允许在使用通配符形参上时设定上限，而且可以在定义类型形参时设定上限，用于表示传给该类型形参的实际类型要么是该上限类型，要么是该上限子类型

```
public class Leg<T extends Number>{
    public static void main(String[] args){
        Leg<Integer> li = new Leg<>();
        Leg<Double> ld = new Leg<>();
        Leg<String> ls = new Leg<>();//错误
    }
}

//Leg类的类型形参上限是Number类，T只能是Number或者Number的子类
```

* 为类型形参设定多个上限

```
public class Leg<T extends Number & java.io.Serializable>{
    public static void main(String[] args){
        Leg<Integer> li = new Leg<>();
        Leg<Double> ld = new Leg<>();
        Leg<String> ls = new Leg<>();//错误
    }
}
//如果需要为类型形参指定多个上限时，所有接口上限必须位于类上限之后
```

* 通配符下限: <? super Type> 表示它必须是Type本身或Type父类


```
public static <T> T dance(Collection<? super T> girl , Collection<T> nenmo)
{
    for(T g :nenmo){
        //do something
    }
}
```
