---
layout: post
title:  "jdk9 内存顺序模型"
date:   2024-12-24 00:00:00
categories: java
tags: 并发

---

* content
{:toc}
本文谈一下对  Java Memeory Mode 的理解。







# 1. Java Memory Modes

术语Java Memeory Mode (在jdk9中被 doug lea 提及) ，这个术语通常用于描述 java 并发模型。

在jdk9中，提供了一套 VarHandle API ， 为高级的Java程序员提供了一种控制并发的方式。

> 提供了一些控制原语。  也可以认为是一些“控制保证”。 程序员可以利用这些”保证“来编写出符合业务逻辑的，安全的，效率的工具。









## 1.1  Varhandle API 的介绍

我们需要使用 `MethodHandles.lookup()` 来获得 VarHandle变量。



//TODO









## 1.2  并发的背景

"并发"总是会出现一种令人意想不到的现象，它总是难以捕捉，例如经典的 `i++` 问题。

导致这种现象的可能原因有：



### 任务并行

- 任务并行。  对于单核处理器来说，两个线程分别执行操作A和B。 那么只有简单的A先于B 或 B先于A 。但是对于多核来说，两个操作可能是无序的（或许是同时进行）



### 内存并行

- 内存并行。除了任务并行(多核并行)以外，内存并行仍然可能导致问题。内存可能同时被多个设备代理(通常是缓存)， 变量不是由"唯一"的物理设备表示（这将会带来强制刷新到内存、失效缓存行等问题）。

  除此以外，处理器一次性能够操作内存的宽度也会引起问题(例如32位cpu读取long类型)，他们不总是原子的操作。



### 指令并行

- 指令并行。  （cpu的流水线作业）cpu包含了多个部件，多个部件是协同工作的。cpu指令总是以叠加的方式处理指令。 因此同一时间可能会处理多个cpu指令。







## 1.3 更多的模式

处理上述 "并行"的概念和技术逐渐成熟， 相同的方法经常出现在不同的编程语言中。



"没有银弹！"   在众多的规则和模型中，没有一个对所有程序中所有代码都有意义。

多核的经验表明，需要更多的模式(mode)来处理常见的并发编码问题。如果没有他们，一些程序员可能会过渡的使用 同步代码，这会让程序变得更慢。

而一些程序员通过使用非标准操作，在特定的JVM和处理上 绕过限制实现了这些行为，虽然可行但会导致移植性较差。





新的 Memeory Order Mode （内存顺序模型），具有积累效果（约束越来越强，性能则递减）。

从最弱到最强：

plain  -> Opaque -> Release/Acquire ->  Volatile



其中plain和 Volatile 与jdk9之前兼容。







### plain

最简单的 `=`操作，对没有volatile修饰的变量操作，就是plain，可能是 plain read  或 plain write。例如：

```java
public class Foo {
    public int value;
}

public void plainRead(Foo f){
    final int v = f.value; 
}

public void plainWrite(Foo f){
    f.value = 5;
}
```

plain 没有任何额外的语义，没有任何额外的保证。





JVM 规范保证，在单线程内，无论指令如何重排，总是保证plain的直观表现：

例如 `  d = (a + b) * (c + b);`

所有的load都可能提前启动 , 指令示意图：

```
 load a, r1 | load b, r2 | load c, r4 | load b, r5
 add r1, r2, r3 | add r4, r5, r6
 mul r3, r6, r7
 store r7, d
```

但最终仍会表现得正确。







### Opaque  Mode

ValueHandle 提供了2个api `getOpaque()`  / `setOpaque()`

比Plain 模式添加了额外的约束， 这些约束提供了 "线程间访问" 中 "变量的"最小感知"。



当使用Opaque（或更强的模式）时，提供保证：

- 一致性： 保证了变量严格的读写顺序。保证了 依赖于先读然后写的操作， 或  写操作后再读的一致性。

  换句话说就是不会破坏  RMW（读-修改-写）的关系。

- 持续：  写操作只保证最终可见。

  换句话说，不保证其他线程立即可见。但最终一定会可见。

- bitwise 原子性 :  如果使用 Opaque (或更强的模式)访问，读取所有的数据类型，包括(long ,double) 在bitwise尺度上都是原子性的（保证一次完整读取所有bit），不会混合读取多次并发写入的bit（不会出现前一半儿bit是A操作写入，后一半儿bit是B写入，导致数据组合在一起是无意义的）。







### release/acqiure 模式

通过 `VarHandle.setRelease()`  `VarHandle.getAcquire()` api来实现 release/acquire 模式。并且比Opaque提供了积累(拥有Opaque的全部约束保证)的约束：

- 在线程T内， 如果一个访问操作A  源码上在 Release(或更强的模型)写操作W 之前。那么在本地线程内，A操作在W之前。

  

- 在线程T内，如果 acquire(或更强的模式)读操作R 在源码上在访问操作A之前。 那么acquire R操作先发生，A访问后发生。



RA(release/acquire)模式是因果一致性系统的主要思想。 因果性在大多数的通信形式上是必不可少的。例如，如果我做好了晚饭，我告诉你已经做好了晚饭，你听到了我的声音，那么你可以确定晚餐一定存在。 在听到ready 时，你一定能访问到正确的dinner 

例如：

```
 volatile int ready; // 初始化的值是0 ,使用 VarHandle REDAY 引用该变量
 int dinner;         // mode does not matter here  
 
 
 Thread 1                   |  Thread 2 
 dinner = 17;               |  if (READY.getAcquire(this) == 1)  //如果看到了 ready =1
 READY.setRelease(this, 1); |    int d = dinner; // 那么一定保证看到dinner=17
```

如果看到了Release操作做出的改变，那么在Release之前的编码的操作一定执行完毕。



在生产者消费者设计、消息传递设计中，需要 RA模式的保证。





#### RA栅栏

你可以使用api风格更明确的 RA模式：

先调用 `VarHandle.releaseFence()`  ，然后使用 `VarHandle.setOpaque()`来代替`setRelease()` 。如果你的变量是  bitwise原子性的，那么甚至可以用 plain 来代替。

同样的，可以使用 `VarHandle.acquireFence()` 插入acquire屏障，然后逐步用opaque、plain 代替acquire



例如：

```java
//Thread 1:

dinner = 17;
VarHandle.releaseFence();//R屏障
ready = 1;

Thread 2:
VarHandle.acquireFence();
if(ready==1)
    int d = dinner;  //此处一定能看到dinner=17
```



但是` releaseFence()`  可能比 `setRelease()` 提供更多的额外约束 ： 

releaseFence是一个栅栏，将前面的所有访问和 后面的所有写入分开， ` releaseFence()` 保证Fence之前的所有访问都完成，再执行后续的写入操作。













#### 提供一个简单的验证程序

```java
public class MainForFences {

    public volatile int ready;

    public int dinner;

    public static VarHandle varHandle;
    static {

        try {
            varHandle = MethodHandles.lookup()
                    .findVarHandle(MainForFences.class,"ready",int.class);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) throws ClassNotFoundException, InterruptedException {


        int a=0;
        int b=0;

        for (int i = 0; i < 10000000; i++) {
            if (i%10000==0){
                System.out.println(a);
                System.out.println(b);
            }
            int callable = callable();
            if (callable==0) a++;
            if (callable==19) b++;

        }
        System.out.println(a);
        System.out.println(b);
    }

    private static int callable() throws InterruptedException {
        MainForFences mainForFences = new MainForFences();

        AtomicInteger res = new AtomicInteger();


        Thread t1 = new Thread(() -> {
            mainForFences.dinner = 19;
            VarHandle.releaseFence();
            mainForFences.ready=1;
        });


        Thread t2 = new Thread(() -> {
            while (true) {
                VarHandle.acquireFence();

                if (mainForFences.ready == 1) {
                    res.set(mainForFences.dinner);
                    break;
                }
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        return res.get();
    }
}
```











# 参考

https://www.lenshood.dev/2021/01/27/java-varhandle/#opaque

https://gee.cs.oswego.edu/dl/html/j9mm.html#summarysec
