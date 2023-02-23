

# 1.响应式编程一些概念







## 1.1 响应式编程 要解决什么问题



响应式编程是关注于数据交换和传播的编程范式



```
传统命令式编程强调 "下一步的运行,基于上一步的执行"

异步响应式编程强调  “下一步的运行,基于上一步的结果”



//传统 命令式编程下一步的运行必须阻塞等待上一步的执行完毕。
//响应式编程更关注于处理数据。关注从上游接收什么样的数据，经过自己加工，传递给下游特定的数据。
```





响应式编程 和 传统命令式编程 在请求较少时,处理业务的表现相差不大。尤其是硬件资源没有吃光的情况下。







### 1.1.1 问题描述

假设有1位客人要去餐厅点餐, 服务员为顾客提供服务，需要经过3个步骤 

````
 1.”询问点菜“ 
 2.”准备做饭“ 
 3.”给客人上菜“
````

注意，这三个步骤是有先后依赖关系的。必须先执行 点菜，才能做饭，和上菜。







传统命令式编程,  服务员(线程)必须询问好了菜单以后，必须等待厨师(线程)做完饭（或者干脆自己成为厨师做饭，

即使自己不转职成为厨师,也不能为其他顾客服务。意味着执行上下文的绑定    ），然后为其上菜。

```
由于有  cpu分时复用技术，此时只需要阻塞住 "服务员"线程，看起来还是不那么浪费。

但事实上，线程上下文切换依然会有不小的开销。 (此处可以使用协程等技术，避免在OS级别进行上下文切换)
```



用图表示就是这样:

![image-20221021212841636](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221021212841636.png)

同样的，对于厨师来说，如果没有菜单会怎么样？阻塞等待。好在线程并不是在空转，他们在阻塞等待。那么如果短时间内，来了很多位顾客会怎么办？

生成多个 “服务员”线程，和“厨师”线程为这位顾客服务。如果这位顾客可能“点单”非常慢，那么“服务员”线程就不得不阻塞等待为其服务。



```
由于阻塞等待可以让出cpu时间片,这种模式还算可以维持。那么如果顾客的数量非常多,会怎么样？
```

创建的多个 “服务员”线程会逐渐占用越来越多的系统资源，直到耗尽为止，那么此时 “顾客”的吞吐量就达到上限了。

```
由于fork出相当多的线程资源,OS不得不调度如此多的线程.
```











参考

https://blog.csdn.net/netyeaxi/article/details/112621600







### 1.1.2 响应式编程

对于响应式来说，这种方式如何服务  “顾客”呢？



"服务员" 只做点菜的活， “厨师”只会做饭，“杂工”只负责上菜

```
服务员不会固定一位顾客点菜,为第一个顾客点单完毕以后,不会阻塞线程，而是为下一个服务。 “服务员”不关心厨师是否做完菜，只关心菜名(输入)

厨师也不会固定一位顾客做饭， 厨师不关心“是哪一位顾客，点的菜”，只关心 “输入与输出”是否正确(菜名与做出的菜)

杂工，不关心是 “哪一位服务员为哪一个顾客点的单，也不关心是哪一个厨师做出的哪一道菜”，只关心这道菜送给谁
```





当短时间内，顾客数量增多，响应式会怎么做？

```
同样会fork出多个 “服务员” “厨师” “杂工”

由于cpu时间片的分配是OS决定的,即使是完全相同的一道菜,可能实际完成的时间也不尽相同。

响应式编程就像流水线一样, 服务员处理完当前的点单，就会处理下一个点单。不再有服务员线程阻塞切换等待厨师做饭的开销。

会极大的提高  高负载，高IO情况下的 吞吐量。

所有的响应式都只关心 上下游的输入输出是否正确。只要正确，就可以处理。
```



对应的以图形的方式如下：

 ![image-20221021221703895](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221021221703895.png)







## 1.2 响应式编程是什么

响应式编程是一种编程范式，基于它的API、库和语言特性会采用特定的设计模式，这种设计模式的目的是完成异步响应式程序的执行。







## 1.3 背压 backpressure

参考

https://www.jianshu.com/p/2c4799fa91a4

https://zhuanlan.zhihu.com/p/384059111



### 1.3.1 什么是 back pressure

背压特指 ： Producer 产出快，而 Consumer消费不完的场景， 此时产生背压。







「背压现象」指的是一种生产者速率大于消费者速率的现象。

「背压机制」指的是一种预防背压现象出现的机制。







背压是另外一个关于数据流速控制复杂名词。它是为了提高系统的稳定性和完整性才被使用的。

背压试图通过限制在指定时间内允许的操作数来解决这个问题（也就是限制执行速率）





### 1.3.2 back pressure场景





服务间通讯 - 客户端对服务 A qps 为 100, 服务 A 需要请求内部服务 B, B 的 QPS 为 75, 如果不做额外的处理，服务 A 每秒钟就会积压 25 个请求。



文件IO - 读快写慢 假设读速度 150MB/s 写速度 100MB/s, 那么每一秒就必须缓冲 50MB, 也就是 20 多秒就接近 1GB, 可想而知，如果不注意策略的话，内存很快就撑爆。



### 1.3.3 解决策略



1. 限制数据产出速率
2. 增加数据消费速率
3. 缓冲数据 buffer
4. 丢弃数据 drop



如果不支持 backpressure 的情况首先带来的是内存的激增，其次是

1. 拖垮其他进程
2. 垃圾收集频率变高
3. 内存耗尽





















## 1.4 发布者 publisher



发布者就是数据生产者。这个组件为系统产生想要的数据。

```
在实践中，这可能是一个数据库的查询结果、twitter feed、股票市场报价feed，等等。
```





`publisher`有许多名字。 不同的 Reactive lib库有不同的名字：

```
Observable (RxJava) / Mono (Project Reactor)： 这种[不产生任何数据]的发布者有许多名字。在Reactive Streams的规范中，它就叫做发布者。


Single (RxJava) / Mono (Project Reactor)： 这种[最多产生一条数据]的发布者有许多名字。在Reactive Streams的规范中，它就叫做发布者。


Observable (RxJava) / Flux (Project Reactor)： 这种[产生多条数据]的发布者有许多名字。在Reactive Streams的规范中，它仍就叫做发布者。
```





## 1.5 订阅者 Subscriber



订阅者“订阅”发布者，当新数据产生、错误发生、或发布者完成数据生产后会收到通知。产生的数据会在订阅者中进行处理。















## 1.6 响应式编程适用于哪些场景？



响应式编程能够提供更大的 “吞吐量上限”，适合于IO密集型应用。

gateway比较适用于响应式。



```
响应式 并不能使接口的响应时间缩短，它仅仅能够提升吞吐量和伸缩性。
```











## 1.7 对于学习Java的Reactive的须知点

以下仅讨论 Java 语言层面上的  `Reactive`学习。 

java to `Reactive`  的学习曲线非常陡峭，需要大量的前置知识，包括但不限于  

​	了解 jdk8`Functional`相关类(例如Function,BiFunction , Consumer , Predicate等) 

​	需要了解跨语言的概念例如 `回调`  `异步`  `函数式`

​	操作系统的基础知识 `阻塞` `异步` 























# 2. Spring WebFlux



响应式编程就是基于reactor的思想，当你做一个带有一定延迟的才能够返回的io操作时，不会阻塞，而是立刻返回一个流，并且订阅这个流，当这个流上产生了返回数据，可以立刻得到通知并调用[回调](https://so.csdn.net/so/search?q=回调&spm=1001.2101.3001.7020)函数处理数据。



首先WebFlux并不能代替 SpringMVC,  只能提供更高的吞吐量上线，和伸缩性。



![image-20221022111530348](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221022111530348.png)





WebFlux适用于网关等高吞吐高IO的组件。













​	



## 2.1  WebFlux与MVC的异同点





![image-20221022110231717](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221022110231717.png)





都可以使用 Spring MVC 注解，如 `@Controller`, 方便我们在两个 Web 框架中自由转换；

均可以使用 Tomcat, Jetty, Undertow Servlet 容器（Servlet 3.1+）



Spring WebFlux：

​	基于 [Functional Endpoints](# 2.2.3 Spring WebFlux 编程模式) ， 基于事件循环Event loop  ，基于并发的模型， 基于 Netty的网络请求处理。



Spring MVC :

​	命令式编程，易于 coding 和 debug  ， JDBC , JPA 基于阻塞的依赖






**注意点：**

- Spring MVC 因为是使用的同步阻塞式，更方便开发人员编写功能代码，Debug 测试等，一般来说，如果 Spring MVC 能够满足的场景，就尽量不要用 WebFlux;
- WebFlux 默认情况下使用 Netty 作为服务器;
- WebFlux 不支持 MySql;





### 2.1.1 使用Flux编程的须知点





![image-20230104144136922](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104144136922.png)

- 如果你已经有一个Spring MVC 应用，并且它仍能胜任当前的需求，那么无需做出改变。 命令式编程是  编写代码，理解代码，debug代码的最简单方式。 你可以最大限度的使用lib库，因为到目前为止大多数lib库都是阻塞式的。









![image-20230104144723140](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104144723140.png)



- 在微服务架构中，你可以混合使用Spring MVC或Spring WebFlux控制器或Spring WebFlux Functional Endpoints。两个框架都支持基于注解的编程模型，这将更容易的重复利用知识点，同时还可以为正确的工作选择正确的工具。





![image-20230104145229587](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104145229587.png)



如果你拥有一个大的团队，请记住学习无阻塞，函数式，声明式编程的学习过程是陡峭的。一个实用的方式是使用 响应式`WebClient`

除此以外，要从小事开始，衡量益处和成本。对于大多数的app来说，切换到 flux是不必要的。















## 2.2 快速入门

将快速搭建一个 `Spring webFlux` 工程，并创建一个响应Http请求的 `Controller`接口。



引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
```







### 2.2.1 编写controller

Spring WebFlux有两种风格：基于`functional endpoint`和 基于`注解 Annotation`的。



基于注释的与Spring MVC非常相近：

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping("/say")
    public String sayHello(){
        return "hello,world";
    }

    @GetMapping("/obj")
    public Mono<User> getUser(){
        User user = new User();
        user.setName("hello world");
        user.setGender("male");
        return Mono.just(user);
    }
}
```



基于Functional Endpoint的方式声明 :



首先需要一个处理请求的 `handler` 

```java
@Component
public class MyUserHandler {
    
    public Mono<ServerResponse> getUser(ServerRequest serverRequest){
        Optional<Object> name = serverRequest.attribute("name");
        if (name.isPresent()) {
            Object o = name.get();
            User user = new User();
            user.setName(o.toString());
            user.setGender("male");
            return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                    .body(BodyInserters.fromValue(user));
        }
        return ServerResponse.badRequest().body(BodyInserters.fromValue("丢失参数 name"));
    }
}
```



然后将 `handler` 注册（指明 uri, 指明回调的method）

```java
@Configuration
public class MyUserRouter {

    @Resource
    private MyUserHandler myUserHandler;

    @Bean
    public RouterFunction<ServerResponse> userRouter(){

        return RouterFunctions.route(RequestPredicates.GET("/user")
                .and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), myUserHandler::getUser);
    }
}
```























### 2.2.2 关于 Spring WebFlux



这一部分参考 https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html







Spring WebFlux 选用的响应式lib库是 [Reactor](https://github.com/reactor/reactor) 。



------

<img src="Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104135028778.png" alt="image-20230104135028778"  />

--------

Reactor库提供了 [Mono](# 3.9 `Mono<T>`) 和 Flux 两个API来服务于 0到1有序的数据，或者是0到N有序的数据。

Mono 和 Flux 类通过丰富的操作器集合来支持  [ReactiveX操作词汇](https://reactivex.io/documentation/operators.html) 。 



同时，Reactor 是一个响应式流 的lib库，所以全部的操作都支持 无阻塞的背压。











![image-20230104140103367](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104140103367.png)



WebFlux 以`Reactor` 作为核心依赖，但是仍然可以通过 Reactive Stream 来使用其他的 响应式lib库完成交互。

相关其他的响应式lib库参考 https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html









### 2.2.3 Spring WebFlux 编程模式

Spring WebFlux 有两种编程模式：

​	1. 基于注解的 Controller ([Annotated Controllers](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-controller))   类似于Spring MVC

 2. 函数式端点  [Functional Endpoints](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-fn)  基于 Lambda表达式，轻量级的，函数式编程。 

    可以将 Functional Endpoints 编程看作一个小型的lib或者工具用于【路由】和【处理】请求

    Functional Endpoints 从头到尾处理 request ，而注解Controller模式是通过注解声明，作为callback被调用。



















































## 2.3 ReactiveX

参考地址

https://reactivex.io/documentation/operators.html











## 2.4  Reactive Core

用于介绍 响应式的核心内容。



处理 request 有2个级别的支持：

`HttpHandler` : 基础的HTTP请求处理形式，具有非阻塞I/O和reactive背压，以及用于Reactor Netty、Undertow、Tomcat、Jetty和任何Servlet容器的适配器。

`WebHandler` : 稍微高级一点，用于请求处理的通用web API，在其之上构建具体的编程模型，如带注释的控制器和功能端点。





![image-20230104164953769](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104164953769.png)

对于客户端，有一个基础的  `ClientHttpConnector` (客户端http连接器)  用于无阻塞IO发送http请求，以及 响应式流背压。



### 2.4.1 HttpHandler





















# 3. 一些类与源码



参考

https://cloud.tencent.com/developer/article/1746119







## 3.1  `Flux<T>`

核心类之一。 

![image-20230105124037431](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230105124037431.png)

表示一个响应式流的 [Publisher](# 3.3 `Publisher<T>`)，可以触发【0到N】多个元素读写操作的Publisher，触发元素以后，可以成功也可以携带Error返回





Flux提供了非常多的有用的方法，来处理这些序列，并且提供了`completion`和`error`的信号通知。

相应的会去调用Subscriber的 `onNext`, `onComplete`, 和 `onError` 方法。

















### 3.1.1 接口方法



#### 3.1.1.1 静态构造方法

一系列静态方法用于构造 `Flux<T>` 流

##### 3.1.1.1.1 from族

from族表示从某些容器（Array,Iterable）中转化为一个Flux



![image-20230113000541381](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000541381.png)

```java
//从一个Publisher中获得 Flux
public static <T> Flux<T> from(Publisher<? extends T> source);
```



![image-20230113000554154](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000554154.png)

```java
//从数组中获得 Flux
public static <T> Flux<T> fromArray(T[] array);
```





![image-20230113000630514](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000630514.png)

`Iterable.iterator()`的方法将被调用至少1次，且至多两次。	 

```java
//从一个Iterable中包装一个Flux
public static <T> Flux<T> fromIterable(Iterable<? extends T> it);
```







```java
//从一个Stream中获得 Flux
public static <T> Flux<T> fromStream(Stream<? extends T> s);

//从一个提供Stream的 Supplier中获得Flux
public static <T> Flux<T> fromStream(Supplier<Stream<? extends T>> streamSupplier);
```















##### 3.1.1.1.2  concat



`concat()`方法:

```java
public static <T> Flux<T> concat(Iterable<? extends Publisher<? extends T>> sources);
```

这个方法用于将  元素为`Publisher`的`Iterable`连接在一起。 其中`Publisher` 提供了真正的元素`T`。

![image-20230106093648733](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106093648733.png)





![image-20230106093935438](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106093935438.png)

```
Concatenation 表现： 按顺序从第一个源中订阅元素，等到第一个源完成以后就会继续订阅第二个源，依次类推直到所有源都订阅完成。

出现任何的Error都会立刻中断序列。
```





##### 3.1.1.1.3  just



![image-20230112235300457](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230112235300457.png)

创建一个能够触发本方法传入元素的`Flux` ，然后它就 `onComplete()`

```java
public static <T> Flux<T> just(T... data);
```



![image-20230112235417840](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230112235417840.png)

创建一个仅触发一个元素的 `Flux`,然后它就 `onComplete()`

```java
public static <T> Flux<T> just(T data);
```









##### 3.1.1.1.4 Interval

定时器族。 都返回 `Flux<Long>` 



![image-20230113000206254](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000206254.png)

创建一个流，并定时向流上发布一个数据，数据从0开始递增。 传入一个时间间隔，按照时间间隔发布数据

```java
public static Flux<Long> interval(Duration period) {
```





比上面多加了一个初始延迟 `delay` 

```java
public static Flux<Long> interval(Duration delay, Duration period);
```

























##### 3.1.1.1.5 其他的构造方法



![image-20230113000310093](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000310093.png)

创建一个永远不会触发任何  【数据】 【异常】和【完成信号】 的`Flux`

```java
public static <T> Flux<T> never() ;
```





![image-20230113000327199](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000327199.png)

返回一个不触发任何数据的`Flux`

```java
public static <T> Flux<T> empty();
```















![image-20230113000354404](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113000354404.png)

返回一个订阅以后就即触发`Error`的`Flux`

```java
public static <T> Flux<T> error(Throwable error);
```

































#### 3.1.1.2 实例方法

实例方法定义了类的行为。





##### 3.1.1.2.1 hook回调

对于`Flux` 来说，首先开始`subscribe()`  触发 `doOnSubscribe()` 回调，当有异常抛出以后，会`cancel()`  触发`doOnCancel()` 回调。

`Flux`  通过`next()` 来获取下一个元素。此时会触发 `doOnNext()` 回调









有如下回调方法 ： 



##### `doOnSubscribe()` 

`doOnSubscribe()` ： 当Flux被订阅的时候，触发 `doOnSubscribe`回调。

![image-20230106122011870](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106122011870.png)

回调方法传入一个 `Consumer` ，提供一个表示本次订阅的 `Subscription`消费。









##### `doOnNext()`

`doOnNext()` :  Flux通过 next() 获得下一个元素，当调用`next()`方法以后，触发`doOnNext()`回调。

![image-20230106123059063](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106123059063.png)



```
当 Flux 触发了一个元素时，通过doOnNext方法触发一个行为表现。

Consumer首先被执行，之后 onNext信号才会被传播到下游。
```









##### `doOnTerminate(Runnable)`

![image-20230106160343532](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106160343532.png)

```
当一个Flux终止时，先调用本hook,然后才会调用 onComplete或者onError
```









##### `doOnError()` 

​	`doOnError`  当 Flux以一个 Error结束时，触发次回调。方法有多个重载:

![image-20230106135609739](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106135609739.png)





##### `doOnError(Consumer<? super Throwable)`

![image-20230106135624873](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106135624873.png)

```
当Flux以一个Error完成时，给Flux 添加一个表现行为触发。
```



##### `doOnError(Class,Consumer)`

![image-20230106135847816](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106135847816.png)

```
当Flux返回的Error的类型为 传入Class的实例，则触发本hook
```





##### `doOnError(Predicate,Consumer)`

![image-20230106155450847](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106155450847.png)



```
传入一个断言，当断言为true，则触发本hook
```











`doOnComplete(Runnable)`

![image-20230106155646158](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106155646158.png)

```
当Flux成功结束以后，触发本hook，执行传入的 Runnable.

Runnable先被执行，然后 onComplete信号向下游传递。
```

















`concatWithValues(T...)`  实例方法，

用于将所给元素连接到当前Flux的末尾。

<img src="Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230106104849735.png" alt="image-20230106104849735" style="zoom:80%;" />

```java
public final Flux<T> concatWithValues(T... values);
```









##### 3.1.1.2.2 limit限制速率

一共有3个 `limitXXX`方法来限制消费速率，来解决背压问题。





![image-20230116102458759](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230116102458759.png)





















## 3.2  ServerRequest



![image-20221022150104115](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221022150104115.png)



```
这个类在 web.reactor 包下
```









![image-20221022135559783](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221022135559783.png)



```
代表了服务器端的HTTP请求(也就是服务器收到的，来自客户的HTTP请求)

可以被HandlerFunction 处理。


可以使用 ServerRequest.Headers 或者 <T> T body(BodyExtractor<T, ? super ServerHttpRequest> extractor)
访问请求头或者请求体。 
```





### 3.2.1 接口方法



![image-20221022150557173](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20221022150557173.png)





```java
//返回http请求的类型,返回一个枚举类  HttpMethod
//GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE;
HttpMethod method();

//将http请求类型的名称转化为String
String methodName();

URI uri();

//返回请求path
String path();

//Get the request path as a PathContainer.
//带有路径参数的数据结构 PathContainer
default PathContainer pathContainer() {return PathContainer.parsePath(path());}


//获取请求头
Headers headers();

//获得cookie
//1个key对应多value的工具类MultiValueMap
MultiValueMap<String, HttpCookie> cookies();


//对方的远程地址 
Optional<InetSocketAddress> remoteAddress();

//本地地址
Optional<InetSocketAddress> localAddress();

//Get the readers used to convert the body of this request.
//获得当前requestbody的 reader
List<HttpMessageReader<?>> messageReaders();


//使用给定的 Body提取器，提取本次request body
<T> T body(BodyExtractor<T, ? super ServerHttpRequest> extractor);

/**
 * 将 reuqestBody提取为Mono
 */
<T> Mono<T> bodyToMono(Class<? extends T> elementClass);



/**
 * 将body提取为一个Flux
 *
 */
<T> Flux<T> bodyToFlux(Class<? extends T> elementClass);


// 返回Optional包裹的 request 属性
default Optional<Object> attribute(String name) {return Optional.ofNullable(attributes().get(name));}


//路径变量
Map<String, String> pathVariables();


Mono<WebSession> session();

//Get the authenticated user for the request, if any.
//获取当前 request的认证用户，如果存在的话
Mono<? extends Principal> principal();











/**
 * Extract the body with the given {@code BodyExtractor} and hints.
 * @param extractor the {@code BodyExtractor} that reads from the request
 * @param hints the map of hints like {@link Jackson2CodecSupport#JSON_VIEW_HINT}
 * to use to customize body extraction
 * @param <T> the type of the body returned
 * @return the extracted body
 */
<T> T body(BodyExtractor<T, ? super ServerHttpRequest> extractor, Map<String, Object> hints);
```







## 3.3 `Publisher<T>`

发布者接口。

![image-20230103165217029](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103165217029.png)



`Publisher` 代表了一个 可以提供无限数据量的有序元素的提供者。并将这些元素发布到订阅它们的 `Subscriber`

一个发布者可以动态的服务于多个 订阅者。



`T` 代表元素的类型。





### 3.3.1 接口方法

`Publisher<T>` 接口是一个函数式接口。只有一个方法 `subscribe(Subscriber)`





## 3.4 `Subscriber<T>`

订阅者接口。

订阅者接口等待来自 `Publisher` 推送的数据。订阅者接口存在数个`回调函数`，满足对应触发条件以后，调用对应的`回调函数`





### 3.4.1 接口方法



#### 3.4.1.1 `onSubscribe(Subscription)`

![image-20230103170902090](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103170902090.png)



当调用了`Publisher.subscribe`方法以后， 调用此回调方法`onSubscribe(Subscription)`。

 其中 [Subscription](# 3.5 Subscription)（订阅） 代表了一个`Publisher`(发布者)和`Subscriber`(订阅者) 一对一的生命周期。 Subscription用于请求数据和解绑订阅。







一个具体的 `subscribe`实现：  (PullPublisher)

```java
    public void subscribe(Flow.Subscriber<? super T> subscriber) {
        Subscription sub;
        if (throwable != null) {
            assert iterable == null : "non-null iterable: " + iterable;
            sub = new Subscription(subscriber, null, throwable);
        } else {
            assert throwable == null : "non-null exception: " + throwable;
            sub = new Subscription(subscriber, iterable.iterator(), null);
        }
        subscriber.onSubscribe(sub);

        if (throwable != null) {
            sub.pullScheduler.runOrSchedule();
        }
    }
```



















#### 3.4.1.2 `onNext(T)`



![image-20230103171846534](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103171846534.png)



```
发布者调用 Subscription.request(long) 方法时，作为数据通知的一个回调函数。 
```













#### 3.4.1.3 `onError(Throwable)`



![image-20230103195029817](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103195029817.png)

这个方法表示： 失败的 "终止状态"。 调用时传入一个 Throwable对象，表示发布一个异常。

调用本方法以后，`Subscription.request()` 方法将失效，不再触发任何event









#### 3.4.1.4 `onComplete()`



![image-20230103201303025](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103201303025.png)

这个方法表示： 成功的"终止状态"

调用本方法以后，`Subscription.request()` 方法将失效，不再触发任何event













### 3.4.2 深入理解`Subscriber`

`Subscriber` 和 `Publisher`的组合是 观察者模式的使用。 



`Subscriber` 有`CoreSubscriber` `BaseSubscriber`等的子接口，无论是哪种，核心都是希望开发者来实现 `onXXX()`回调方法。



在`Flux.subscribe()`方法中，可以直接传入一个 `Subscriber<T>` 或者 `CoreSubscriber<T>`![image-20230115101637347](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230115101637347.png)







































## 3.5 Subscription



![image-20230103171633329](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103171633329.png)



```
Subscription 代表了一个Publisher`(发布者)和Subscriber(订阅者) 之间 一对一的生命周期。


一个Subscriber只能有1个Subscription
```





Subscription可以用于请求数据，也可以用于解绑订阅（并允许释放资源）。





### 3.5.1 接口方法



#### 3.5.1.1 `request(long)`

![image-20230103172335665](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103172335665.png)



当期望的数据到来时，通过本方法请求。传入long 表示请求的数据个数。



![image-20230103181423865](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103181423865.png)



`Publisher` 如果当流读到结尾以后，允许发送少于需求数据的数量。但是后续必须触发 `Subscriber.onError()` 或者

`Subscriber.onComplete()` 两个方法中的一个。











## 3.6 `CorePublisher<T>`

![image-20230103201513892](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103201513892.png)

核心通知的 发布者。区别就是 提供了一个`CoreSubcriber`的订阅方法。主要的核心在  `CoreSubscriber` 类中







### 3.6.1 接口方法

```java
void subscribe(CoreSubscriber<? super T> subscriber);
```



![image-20230103201541578](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103201541578.png)



`CorePublisher`提供了形参为  `CoreSubscriber` 类型的订阅方法。























## 3.7 `CoreSubscriber<T>`



![image-20230103202217074](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103202217074.png)



一种可以提供Context(上下文)的订阅者。  



```
它放宽了§1.3和§3.9的规则。如果在接收到的订阅上执行了一个无效的<= 0请求，该请求将不会产生onError，并将被忽略。
```







### 3.7.1 接口方法



#### 3.7.1.1 `currentContext()`

从包含下游操作的【依赖组件】中返回当前的上下文。  无论是【订阅状态】还是【终止】状态的订阅者都可以返回 `Context`

![image-20230103203852034](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103203852034.png)







#### 3.7.1.2  `onSubscribe(Subscription)`

重写了订阅方法。放宽了 `request()`方法。

不会开始任何数据流，直到调用了 `Subscription.request(long) ` 方法。

![image-20230103204103683](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103204103683.png)

































## 3.8  Context

表示一个上下文。在`package reactor.util.context;` 包下

![image-20230103202344929](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103202344929.png)



一个 【通过上下文协议】 【存储在组件之间】的  【key/value】键值对, 称为 `Context`。

上下文是传输正交信息(如跟踪或安全令牌)的理想选择。





### 3.8.1 接口方法



![image-20230103203127605](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103203127605.png)



```java
empty() ; //返回一个空的 Context

of(); //构造方法

stream(); //返回 包含上下文内容的流 Stream<Entry<Object,Object>>
```







## 3.9 `Mono<T>`

核心类，提供一个具备 【基础读写操作】的响应式流Publisher。 这个流至多通过 `onNext`返回1个元素，然后就转为终结状态,触发`onComplete`信号。

（成功Mono或者没有 返回值， 或者只触发一次 `onError` 表示失败Mono）

![image-20230103205200594](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103205200594.png)





### 3.9.1 接口方法





#### 3.9.1.1 `just(T)`

![image-20230103205830216](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103205830216.png)

以指定的数据（在创建的时候就捕获），创建一个 Mono对象。



示例：

```java
    @GetMapping("/obj")
    public Mono<User> getUser(Integer a){
        User user = new User();
        user.setName("hello world");
        user.setGender("male");
        return Mono.just(user);
    }
```



#### 3.9.1.2 `create(Consumer<MonoSink<T>> callback)`

![image-20230103212724123](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103212724123.png)

创建一个基于回调api的延迟的触发器，这个触发器最多只能返回1个值，1个完成结果，或者1个异常信号。

需要传入一个  [MonoSink ](# 3.10 MonoSink)





![image-20230103214915196](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103214915196.png)

有两种API模式：1.添加/移除监听器 2. 回调处理器





![image-20230103230607448](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230103230607448.png)

使用以下api 初始化一个监听器，调用 sink

```java

  Mono.<String>create(sink -> {
      HttpListener listener = event -> {
          if (event.getResponseCode() >= 400) {
              sink.error(new RuntimeException("Failed"));
          } else {
              String body = event.getBody();
              if (body.isEmpty()) {
                  sink.success();
              } else {
                  sink.success(body.toLowerCase());
              }
          }
      };
 
      client.addListener(listener);
 
      sink.onDispose(() -> client.removeListener(listener));
  });
```







## 3.10 MonoSink

![image-20230104000704740](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104000704740.png)

用于包装一个真实的下游订阅者的API。  可以包装一个单一值，一个异常。或者包装null





### 3.10.1 接口方法

![image-20230104000856855](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104000856855.png)



#### 3.10.1.1 `success()`

![image-20230104000953027](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104000953027.png)



调用此方法，表示 flux 流结束了，不返回任何值结束了。

多次调用此方法 或者  在其他终止方法以后 都是无效的。





#### 3.10.1.2 `success(@Nullable T value)`

表示以一个值结束了流。

![image-20230104001220841](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104001220841.png)

多次调用此方法 或者  在其他终止方法以后 都是无效的。（多次传入的值会被直接忽略）。

调用此方法传入的值是null 等价于调用 `success()` 方法



#### 3.10.1.3 `error(Throwable)`

![image-20230104001406808](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104001406808.png)

表示以一个异常终结流。

多次调用此方法 或者  在其他终止方法以后 是一个 不支持的操作。 将会通过 ` Hooks.onErrorDropped(Consumer)`丢弃异常。



` Hooks.onErrorDropped(Consumer)`默认情况下将抛出一个包装好的异常（ `reactor.core.Exceptions.bubble(Throwable)`）

这样做的目的是避免  【完全】和【无声地】吞下异常。





#### 3.10.1.4 `MonoSink<T> onRequest(LongConsumer consumer)`

![image-20230104001848526](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104001848526.png)



#### 3.10.1.5 `MonoSink<T> onCancel(Disposable d)`

![image-20230104001856607](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104001856607.png)





#### 3.10.1.6 `MonoSink<T> onDispose(Disposable d)`



![image-20230104001932931](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104001932931.png)









## 3.11 Disposable

可处理的，可配置的。



![image-20230104195025094](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104195025094.png)



```
表示一个任务或者一个资源是可以取消的/处置的。

调用 dispose方法应当是幂等性的。
```



### 3.11.1 接口方法



#### 3.11.1.1  `dispose():void`

![image-20230104195742794](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104195742794.png)



取消任务或者资源。











#### 3.11.1.2 `isDisposed()`



![image-20230104195913345](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104195913345.png)



当资源或任务被取消了则可选择返回true。



当资源正在处理时不保整返回true，但是如果资源已经处理完成，则必须保证返回true。











### 3.11.2 内部接口



#### 3.11.2.1 swap



![image-20230104200254044](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104200254044.png)



`Disposable` 对象的容器允许原子性的修改/替换其内部的 `Disposable`对象。同时，`Swap`对象本身就是一个 `Disposable`的容器。



`Swap` 继承了 `Disposable` 和 `Supplier<Disposable>` 表示具有和提供 `Disposable`的能力。







##### 3.11.2.1.1 接口方法



![image-20230104202723425](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104202723425.png)



```java
boolean update(@Nullable Disposable next);
```

原子性的更新容器内的 `Disposable`对象，并处理前一个（如果有的话）。如果容器已经被处理的话，退回到处理下一个。







![image-20230104202956871](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104202956871.png)

原子性的设置容器内下一个 `disposable`对象，但是不处理前一个。如果容器已经被处理的话，退回到处理下一个。





#### 3.11.2.2 composite

`Disposable`的另一个内部接口`Composite` 复合物。 这个接口本身是Disposable，但也是Disposable的容器。



![image-20230104203136981](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104203136981.png)



通过调用1次 `dispose()` 来处理所有积累的 `Disposable` 

调用`add(Dispoable)` 将所有权托管给容器。同时您可以通过`remove(Disposable)`从容器中移除 `Disposable`









##### 3.11.2.2.1 接口方法



![image-20230104212215725](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104212215725.png)



`composite` 重写了 `dispose()` 方法。 原子性的标记容器为 已处理。清除并处理容器内全部的`Disposable` 。

调用`dispose`以后的容器不能被重用， `add/addAll`方法将立刻返回 `false`。





![image-20230104214058725](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104214058725.png)



向容器中添加`Disposable` 但是不会处理。





















## 3.12  Result

`io.r2dbc.spi` 包下的接口 `Result` 表示一次 SQL 查询后的结果。

![image-20230107082702025](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107082702025.png)

```
Result 只能被消费一次，且 只能由 getRowsUpdated() 或者是 map(BiFunction) 两个方法之一消费。
```





![image-20230107083217097](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107083217097.png)



`Result` 对象持有一个消费状态， 这个状态可能被  【当前数据行的游标】调用返回。



`Result` 允许 `statement`语句的【只读】和 【单向】 消费。

因此开发者可以调用 [getRowUpdated()](# 3.12.1.1 `getRowUpdated()`)  返回本次SQL修改的行数(`Publisher<Integer>`) 

或者调用[map](# 3.12.1.12 `map(BiFunciton)`)方法，消费本次【SQL查询出数据】，且必须从第一行消费到最后一行。





### 3.12.1 接口方法

#### 3.12.1.1 `getRowUpdated()`

![image-20230107084454742](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107084454742.png)



```
返回一次SQL查询修改数据库数据的行数。 如果这次SQL没有修改任何行，则返回空。
```









#### 3.12.1.12 `map(BiFunciton)`

![image-20230107084555158](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107084555158.png)



调用一个映射方法。  入参是一个`BiFunction` ,  `BiFunction`的入参是本次SQL查询的行数据(包括Row 和 Row元数据)，出参是期望映射后的数据。







## 3.13 Row

`io.r2dbc.spi` 仍然是标准 `spi`



`Row` 代表数据库SQL查询后返回的 一条行数据。

![image-20230107095031437](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107095031437.png)

可以通过 column name或者 column index 查询某一条列数据。 和`JDBC`一样， Column index 总是从0开始。





### 3.13.1 接口方法



#### 3.13.1.1 `get(int,Class)`

![image-20230107120520225](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107120520225.png)

```
返回Row对应index的列数据。 可以传入Object类，匹配任意类型
```









### 3.13.2 实现类



#### 3.13.2.1 MySqlRow

当引入 `r2dbc-mysql `  依赖以后。在` dev.miku.r2dbc.mysql`包下。 

```xml
<dependency>
    <groupId>dev.miku</groupId>
    <artifactId>r2dbc-mysql</artifactId>
    <version>0.8.2.RELEASE</version>
</dependency>
```







##### 3.13.2.1.1 get(int,Class)源码分析





`MySqlRow.get(int,Class)`: 

![image-20230107130850601](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107130850601.png)

调用 成员变量`Codecs codecs` 的 `decode`方法。

![image-20230107130931544](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107130931544.png)

`Codecs` 编解码器类，用于完成所有类型的编解码任务。 并封装了几种编解码行为

![image-20230107130944569](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107130944569.png)



`get(int,Class)` 调用的就是第一个，带有Class类型的解码。

![image-20230107131044264](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107131044264.png)



`Codecs` 的实现类： `DefaultCodecs`

![image-20230107131127455](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107131127455.png)

查看对应`decode`方法的实现：

![image-20230107131530626](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107131530626.png)





[FieldValue](# )   类， 在`r2dbc-mysql` 依赖下。 表示字段的值。



![image-20230107131945046](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107131945046.png)

两个方法： `isNull()`  ，`nullField()` 返回单例的 `NullFieldValue`

接口有3个实现类，

 `NullFieldValue` 

`NormalFieldValue` （通用字段值）

`LargeFieldValue` 超大字段值

![image-20230107131906622](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107131906622.png)



`LargeFieldValue`超大字段值表示： 字节数组长度超过 `Integer.MAX_VALUE` 的数据。

这种数据在MySQL中将以  `LOB ` 类型存在，例如 `BLOB` `CLOB` 。   `LongText` 类型的长度可以用无符号int32 表示。

![image-20230107132232256](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107132232256.png)



`NormalFieldValue`  通用字段值表示：  字节数组长度小于等于 `Interger.MAX_VALUE` 的字段值。![image-20230107132757923](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230107132757923.png)



`decodeNormal` 解析普通类型。

![image-20230108233551704](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230108233551704.png)

从成员变量 `Codec<?>[] codecs;` 遍历所有的解析器，询问是否能够解析。如果能则使用其解析，并返回结果。

























## 3.14 RowMetadata

















## 3.15 Scheduler

调度器，计划器。是一个接口。

![image-20230113001144910](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113001144910.png)

这个类用于给 `operators`绑定一个异步的调度器。具体将使用线程池实现接口。









### 3.15.1 接口方法



![image-20230113001401661](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113001401661.png)





#### 3.15.1.1 `schedule()`

![image-20230113001435717](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113001435717.png)

让【调度器】 立即调度执行（没有任何延迟） 一个任务。

这个方法是多线程调用安全的,但是不保整 调度器执行task的顺序。

```java
Disposable schedule(Runnable);
```









经过给定延迟以后，才让调度器执行task

```java
Disposable schedule(Runnable task, long delay, TimeUnit unit);
```



带有一个延迟的，周期调用task

```java
Disposable schedulePeriodically(Runnable task, long initialDelay, long period, TimeUnit unit);
```





#### 3.15.1.2 `start()`	



![image-20230113002036023](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113002036023.png)



告诉【调度器】或者【其内部的[Worker](# 3.15.2 内部接口)】，可以开始执行task了 。 `start()`仍然是一个线程安全的调用方法。

但应该避免并发调用 `start()` 和`dispose()` ，因为这样可能使调度程序不确定处于活动或非活动状态。

#### 3.15.1.3 `dispose()`

![image-20230113002339575](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113002339575.png)

告诉调度器，释放全部的资源，然后拒绝执行任何task





#### 3.15.1.4 `createWorker()`

![image-20230113002732029](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113002732029.png)

创建一个属于当前调度器的 worker。

一旦worker不再使用，就应该调用`dispose`释放资源。 worker通常应该先进先出执行任务，但具体依赖是实现类。



















### 3.15.2 内部接口

![image-20230113002459540](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113002459540.png)

![image-20230113002509201](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113002509201.png)



`Worker`表示异步执行task的工作者。



其内部只有 `schedule`方法。没有`dispose`和`start()` 是调度器中真正干活的单位。

![image-20230113002611807](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113002611807.png)









## 3.16 `BaseSubscriber<T>`

实现了`CoreSubscriber`   `Subscription` `Dispoable`  接口。

![image-20230115191550934](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230115191550934.png)

虽然是抽象类，但没有抽象方法。 留有 `hookXXX`方法供开发者快速构造属于自己的`Subscrber<T>`类。

![image-20230115191653641](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230115191653641.png)



同时，写好了重要的 `request(long)` 方法。

![image-20230115191740305](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230115191740305.png)





### 3.16 解决back pressure

通过继承 `BaseSubscriber<T>` 重写 `hookOnSubscriber`  `hookOnNext`  实现按需消费。



```java
public class MyBaseSubscriber<T> extends BaseSubscriber<T> {

    
    @Override
    protected void hookOnSubscribe(Subscription subscription) {
        //刚订阅时 请求1
        request(1);
    }

    @Override
    protected void hookOnNext(T value) {
        System.out.println(value);
        //消费完毕以后，请求
        request(1);
    }
}
```























# 4. R2DBC



`R2DBC`基于`Reactive Streams`反应流规范。它是一个开放的规范，为驱动程序供应商和使用方提供接口（`r2dbc-spi`）。



`R2DBC`和【关系型数据库】交互 ，但和`JDBC`的阻塞特性不同，它是完全Reactive 的非阻塞`API`。



 和`JDBC`  类似，`R2DBC`项目是支持使用反应式编程`API`访问关系型数据库的桥梁，定义【统一接口规范】，不同数据库厂家通过实现该规范提供驱动程序包。

















## 4.1  实现`R2DBC` 的`SPI` 



`R2DBC`定义了所有[数据存储](https://cloud.tencent.com/product/cdcs?from=10680)驱动程序必须实现的`SPI`，目前实现`R2DBC SPI`的驱动程序包括：

- `r2dbc-h2`：为`H2`实现的驱动程序；
- `r2dbc mariadb`：为`Mariadb`实现的驱动程序；
- `r2dbc mssql`：为`Microsoft SQL Server`实现的本机驱动程序；
- `r2dbc mysql`：为`Mysql`实现的驱动程序；
- `r2dbc postgres`：为`PostgreSQL`实现的驱动程序；

同时，`r2dbc`还提供反应式连接池 [r2dbc-pool](https://github.com/r2dbc/r2dbc-pool)。





## 4.2 r2dbc-mysql





### 4.2.1 快速入门

#### 4.2.1.1 引入依赖

```xml
<!-- r2dbc mysql-->
<dependency>
    <groupId>dev.miku</groupId>
    <artifactId>r2dbc-mysql</artifactId>
    <version>0.8.2.RELEASE</version>
</dependency>
```





#### 4.2.1.2 创建连接

首先通过`ConnectionFactoryOptions`创建 `io.r2dbc.spi.ConnectionFactory`  连接工厂对象，

```java
    @Bean(name = "connectionFactory")
    public ConnectionFactory r2dbc() {
        ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
                .option(ConnectionFactoryOptions.DRIVER, "mysql")
                .option(ConnectionFactoryOptions.HOST, "localhost")
                .option(ConnectionFactoryOptions.PORT, 3306)
                .option(ConnectionFactoryOptions.USER, "root")//username
                .option(ConnectionFactoryOptions.PASSWORD, "zxc,./123")/
                .option(ConnectionFactoryOptions.SSL, true)
                .option(ConnectionFactoryOptions.DATABASE, "content2")//数据库名
                .option(ConnectionFactoryOptions.CONNECT_TIMEOUT, Duration.ofSeconds(10))//超时时间
                .build();

        return ConnectionFactories.get(options);//创建ConnectionFactory
    }
```



通过连接工厂`create()`一个 `Publisher` 

```java
Publisher<? extends Connection> publisher = connectionFactory.create();
//返回的是 Connection的 Publisher
```





```java
    @Test
    public void get() throws InterruptedException {

        Publisher<? extends Connection> publisher = connectionFactory.create();

        Disposable subscribe = Mono.from(publisher)
                .flatMapMany(connection -> connection.createStatement("""
                        SELECT *
                        FROM `element`
                        """).execute()
                ).flatMap((Function<Result, Publisher<Element>>) result ->
                        result.map((row, rowMetadata) -> ElementWrapper.wrapper(row)))
                .switchIfEmpty(Mono.empty())
                .onErrorResume(throwable -> {
                    throwable.printStackTrace();
                    return Mono.empty();
                })
                .subscribe(System.out::println);

    }
```





















### 4.2.2  ConnectionFactoryOptions

`r2dbc`连接工厂选项。



![image-20230104230528200](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230104230528200.png)





使用`ConnectionFactoryOptions` 创建一个 ConnectionFactory

```java
    @Bean(name = "connectionFactory")
    public ConnectionFactory r2dbc() {
        ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
                .option(ConnectionFactoryOptions.DRIVER, "mysql")
                .option(ConnectionFactoryOptions.HOST, "localhost")
                .option(ConnectionFactoryOptions.PORT, 3306)
                .option(ConnectionFactoryOptions.USER, "root")//username
                .option(ConnectionFactoryOptions.PASSWORD, "zxc,./123")/
                .option(ConnectionFactoryOptions.SSL, true)
                .option(ConnectionFactoryOptions.DATABASE, "content2")//数据库名
                .option(ConnectionFactoryOptions.CONNECT_TIMEOUT, Duration.ofSeconds(10))//超时时间
                .build();

        return ConnectionFactories.get(options);//创建ConnectionFactory
    }
```









## 4.3 r2dbc-pool

通过`r2dbc-pool` 获得数据库连接。





### 4.3.1 快速入门



引入依赖：

```xml
<dependencies>
  <!-- r2dbc mysql -->
  <dependency>
    <groupId>dev.miku</groupId>
    <artifactId>r2dbc-mysql</artifactId>
    <version>0.8.2.RELEASE</version>
  </dependency>
  <!-- r2dbc-pool -->
  <dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-pool</artifactId>
    <version>0.8.2.RELEASE</version>
  </dependency>
</dependencies>
```





创建`ConnectionPool` ，  通过 `ConnectionFactory` 创建一个  `ConnectionPoolConfiguration` 。 然后 new一个 `ConnectionPool`

```java
	//创建connectionFactory
	@Bean(name = "connectionFactory")
    public ConnectionFactory r2dbc() {
        ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
                .option(ConnectionFactoryOptions.DRIVER, "mysql")
                .option(ConnectionFactoryOptions.HOST, "localhost")
                .option(ConnectionFactoryOptions.PORT, 3306)
                .option(ConnectionFactoryOptions.USER, "root")
                .option(ConnectionFactoryOptions.PASSWORD, "zxc,./123")
                .option(ConnectionFactoryOptions.SSL, true)
                .option(ConnectionFactoryOptions.DATABASE, "content2")
                .option(ConnectionFactoryOptions.CONNECT_TIMEOUT, Duration.ofSeconds(10))
                .build();

        return ConnectionFactories.get(options);
    }

	//注入 ConnectionPool
    @Bean
    public ConnectionPool connectionPool(ConnectionFactory connectionFactory){

        ConnectionPoolConfiguration configuration = ConnectionPoolConfiguration.builder(connectionFactory)
                .maxSize(1000)
                .initialSize(10)
                .maxIdleTime(Duration.ofSeconds(100))
                .build();

        return new ConnectionPool(configuration);
    }
```











## 4.4  spring-data-r2dbc

`spring-data` 工程里，启用了对`r2dbc`的支持。

引入依赖

```xml
    <dependency>
       <groupId>dev.miku</groupId>
       <artifactId>r2dbc-mysql</artifactId>
       <version>0.8.2.RELEASE</version>
    </dependency>

	<!-- 同时引入 r2dbc-pool -->
	<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-r2dbc</artifactId>
       <version>2.3.0.RELEASE</version>
    </dependency>
```



添加`applition.yaml`

```yaml
### r2dbc
spring:
  r2dbc:
    url: r2dbc:mysql://localhost:3306/r2dbc_stu?useUnicode=true&characterEncoding=UTF-8
    username: root
    password: 
    pool:
      enabled: true
      max-size: 10
      initial-size: 10
      validation-query: select 1
```











### 4.4.1  配置类

`spring-data-r2dbc`的配置类 ： `AbstractR2dbcConfiguration` 

继承并重写这些方法可以达到配置的作用。

![image-20230113204315100](Java%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B.assets/image-20230113204315100.png)



```java
@Configuration
public class SpringDataR2dbConfig extends AbstractR2dbcConfiguration {

    

    @Override
    public ConnectionFactory connectionFactory() {
        ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
                .option(ConnectionFactoryOptions.DRIVER, "mysql")
                .option(ConnectionFactoryOptions.HOST, "localhost")
                .option(ConnectionFactoryOptions.PORT, 3306)
                .option(ConnectionFactoryOptions.USER, "root")
                .option(ConnectionFactoryOptions.PASSWORD, "zxc,./123")
                .option(ConnectionFactoryOptions.SSL, true)
                .option(ConnectionFactoryOptions.DATABASE, "content2")
                .option(ConnectionFactoryOptions.CONNECT_TIMEOUT, Duration.ofSeconds(10))
                .build();

        return ConnectionFactories.get(options);
    }


    @Bean
    ReactiveTransactionManager transactionManager(ConnectionFactory connectionFactory) {
        return new R2dbcTransactionManager(connectionFactory);
    }



}
```















### 4.4.2 重要类







#### 4.4.2.1 `DatabaseClient`

`DatabaseClient`是`Spring Data R2DBC`提供的具有功能性的反应式非阻塞`API`，用于与数据库交互。

























