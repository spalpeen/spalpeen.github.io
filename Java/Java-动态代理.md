## 动态代理

动态代理是一种代理机制，代理可以看作是对调用目标的一个包装，这样我们对目标代码的调用不是直接发生，而是通过代理完成，通过代理可以让调用者与实现者解耦，比如进行RPC调用，框架内部的寻址，序列化，反序列化等。


#### JDK动态代理的一个例子

```

public class MyDynamicProxy {
    public satic void main (String[] args) {
        HelloImpl hello = new HelloImpl();
        MyInvocationHandler handler = new MyInvocationHandler(hello);
        // 构造代码实例
        Hello proxyHello = (Hello) Proxy.newProxyInsance(HelloImpl.class.getClassLoader(), HelloImpl.class.getInterfaces(), handler); // 调用代理方法
        proxyHello.sayHello();
    } 
}


interface Hello {
    void sayHello();
}

class HelloImpl implements Hello {
    @Override
    public void sayHello() {
        Sysem.out.println("Hello World"); 
    }
}

class MyInvocationHandler implements InvocationHandler { //实现InvocationHandler
    private Object target;
    public MyInvocationHandler(Object target) {
        this.target = target; 
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Sysem.out.println("Invoking sayHello");
        Object result = method.invoke(target, args);
        return result;
    }
}
```

从API设计和实现角度，这种实现仍然有局限性，因为它是以接口为中心的，相当于添加了一种对于被调用者没有太大意义的限制。我们实例化的是Proxy对象，而不是真正的被调用者，这在实践中还是会带来各种不便

如果被调用者没有实现接口，而我们还是希望利用动态代理，可以考虑cglib，cglib动态代理采取的是创建目标类的子类方式。因为是子类化，可以达到近似使用被调用本身的效果


#### JDK Proxy的优势：


<li>最小化依赖关系，减少依赖意味着简化开发和维护，JDK本身支持，可能比cglib更可靠

<li>平滑升级JDK版本，而字节码类库通常需要进行更新以保证在新版本上使用

<li>代码实现简单



#### cglib的优势：

<li>有的时候调用目标可能不便实现额外接口，cglib动态代理没有这种限制

<li>只操作我们关心的类，而不必为其他类怎加工作量

<li>高性能
