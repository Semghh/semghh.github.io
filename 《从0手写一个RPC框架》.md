《从0手写一个RPC框架》





前置知识：完成了笔记《NIO》

# 1. 准备知识







## 1.1 为什么需要动态代理



![image-20220513085507876](C:\Users\SemgHH\Desktop\笔记\《从0手写一个RPC框架》.assets\image-20220513085507876.png)



为了达成的目标:

```
像调用本地方法一样调用 远程方法。
```





## 1.2 重温动态代理



### 1.2.1 JDK代理

JDK代理只能代理接口



#### InvocationHandler

代理类需要实现 InvocationHandler

```
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {}
```



DebugInvocationHandler

```java
public class DebugInvocationHandler implements InvocationHandler {

    private Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("before : "+method.getName());

        Object res = method.invoke(target, args);
        System.out.println(res);
        System.out.println("after : " + method.getName());

        return res;
    }
}
```

```
其中Object是被代理的对象实例。在创建代理类的时候，需要提供实例对象。



在RPC中，这个实例通过网络传输。由服务提供者自行提供，所以在实现RPC的时候，服务消费者并不需要提供实例对象。只需要提供指定接口，指定方法，指定方法参数，形参即可。
```



ProxyFactory 用于创建一个代理类的工厂

```java
public class ProxyFactory {

    public static Object getProxy(Object target){

        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new DebugInvocationHandler(target));
    }

}
```

```
主要使用Proxy.newInstance()方法生成一个代理类对象。需要传入ClassLoader,被实现的接口，以及InvocationHandler
```



```
newProxyInstance生成一个代理类对象。这个代理类对象执行的所有 [被代理的接口方法] 都会去转而执行
[DebugInvocationHandler]定义的行为。 
```









#### RPC中的动态代理

```
1.1中解释了使用动态代理的目标就是，像调用本地方法一样调用远程过程。
原理: 使用动态代理，屏蔽了向服务端发送请求，获得执行结果的细节。
```

使用时简洁如下:

```java
    public static void main(String[] args) {
        HelloService service  =  getProxyService(HelloService.class);
        String res = service.sayHello("张三");
        System.out.println(res);
    }
```

```
通过getProxyService()方法获得一个代理类对象。通过代理类对象.方法名调用方法。全部的细节都藏在代理类中。
```





![image-20220513101053360](C:\Users\SemgHH\Desktop\笔记\《从0手写一个RPC框架》.assets\image-20220513101053360.png)

```
通过Proxy.newInstance()获得一个属于 T的代理类对象t
通过t.任意被代理的方法。执行 InvocationHandler定义好的方法。
```







### 1.2.2 重温CGLIB代理

Cglib代理拦截的方法可以是非接口方法。通过继承的方式实现。也就是说final 和private 的方法不能够被代理。







代理类执行的方法需要实现MethodInterceptor接口

```java
public class DebugMethodInterceptor implements MethodInterceptor {


    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {

        System.out.println("before Method : " + methodProxy.getSuperName());

        Object o1 = methodProxy.invokeSuper(o, args);

        System.out.println("after Method : " + methodProxy.getSuperName() );

        return o1;
    }
}
```





![image-20220513113532382](C:\Users\SemgHH\Desktop\笔记\《从0手写一个RPC框架》.assets\image-20220513113532382.png)

![image-20220513113512091](C:\Users\SemgHH\Desktop\笔记\《从0手写一个RPC框架》.assets\image-20220513113512091.png)