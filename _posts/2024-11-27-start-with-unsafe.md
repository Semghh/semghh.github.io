---
layout: post
title:  "从多个角度考量 Unsafe"
date:   2024-11-27 00:00:00
categories: java
tags: 并发

---

* content
{:toc}
一开始对Unsafe的认知仅仅局限于CAS操作，随着认知增加 ，实际上：

- juc包用它获得offset。

- nio用Unsafe来判断平台大小端，分配内存。
- 我们甚至可以用它在java平台分配超大内存进行计算。





# 1. Unsafe

Unsafe是 sun提供的工具`sun.misc.Unsafe` 。通过它我们可以使用更底层的手段来操作Java内存模型，例如：

- 保证原子性的交换操作： `compareAndSwapObject()` ，通常用它来原子的修改状态位，制造一个临界区。

  常见的`while(true)` 重试使用它。

- 通过加屏障，具有可见的set操作 ： `putObjectVolatile`  ，通常用它来保证set操作的可见性，和禁止重排。

- 只保证 禁重排，但不立即保证可见性的set操作  ： `putOrderedObject` 性能更好，在特殊场景下能够优化 putVolatile 。例如 Future中设置状态位（单向的，且状态位含义独立）。

- 分配内存 `allocateMemory()` , 回收内存` reallocateMemory()` :  nio用它来判断 平台的大小端。

- 获得数组 baseOffset `arrayBaseOffset()`  

  获得 数组元素的scale `arrayIndexScale()` 

  访问数组元素公式： 索引为n的元素在数组中的偏移量 indexOffset =  base + n* scale

  在j.u.c.a.AtomicIntegerArray 及类似原子数组中使用。





下面是相关代码的举例分析





# 2.  代码举例





## 2.1 如何拿到Unsafe

我们无法直接调用 `Unsafe.getUnsafe()` 来获得Unsafe ，因此需要使用反射的方式拿到它。



我们从Netty中copy一个Permit工具类：

```java
public class Permit {

    public static Unsafe getUnsafe(){
        return (Unsafe) Permit.ReflectiveStaticField(Unsafe.class,"theUnsafe");
    }

    public static Object ReflectiveStaticField(Class<?> cls,String fieldName){
        try {
            Field f =  cls.getDeclaredField(fieldName);
            f.setAccessible(true);
            return f.get(null);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            return null;
        }
    }
    public static Field GetField(Class<?> clz,String fieldName) throws NoSuchFieldException{

        Class<?> c = clz;
        Field f = null;
        while (c != null){
            try {
                f = c.getDeclaredField(fieldName);
                break;
            } catch (NoSuchFieldException e) {
                c = c.getSuperclass();
            }
        }
        if (f==null){
            throw new NoSuchFieldException(c.getName() + "::" + fieldName);
        }
        return f;
    }
}
```

















## 2.2 判断平台大小端



仿照Bits， 我们使用分配内存，再放入一个特殊的long类型，求出他的低地址，然后判断大小端：

```java
public static ByteOrder nativeByteOrder=null;

static {
    Unsafe unsafe = getUnsafe();
    long l = unsafe.allocateMemory(8); //分配一个8byte
    try {			//8个byte， 等于 16个16进制位。
        unsafe.putLong(l,0x0102030405060708);  
        byte value = unsafe.getByte(l);
        switch (value){
            case 0x01: { //如果低地址是01那么是大端
                nativeByteOrder = ByteOrder.BIG_ENDIAN;
                break;
            }
            case 0x08:{ //如果低地址是08那么是小端
                nativeByteOrder = ByteOrder.LITTLE_ENDIAN;
            }
        }
    }finally {
        unsafe.freeMemory(l);
    }
}
```





## 2.3 使用cas模拟临界区

对于一个volatile变量，如果你只想检查它的状态做判断，那么无需使用原子性。



但是如果想模拟一个临界区，那么必须使用原子的修改操作，来修改状态位。

它通常是用一个 while死循环来模拟，这样的公式代码在juc中随处可见：

```java
//我们必须给state规定几个特殊值，表示对象处于不同的状态。 这一步非常重要。
private volatile int state=0;

private final Unsafe unsafe;

//我们还必须拿到 state在 class的offset;  通过 Unsafe.objectFieldOffset()方法拿到
private final long STATE_OFFSET;


//按照序号解读代码，更容易懂。

public boolean foo(){
    int i ;
    for(;;){
        //1.读一次state的值到操作数栈(本地),这个值可能不是最新的，但没关系。若不为最新CAS操作一定会失败，然后走for循环再次尝试
      	i=state;
        A:
        //2.这里我们通过原子操作，保证了临界区内有且只有1个线程操作,让this对象的状态由0过渡到-1
        //2.代码块中的 do something就是过渡过程中，执行的逻辑。
        if(unsafe.compareAndSetInt(this,STATE_OFFSET,0,-1)){
            //3. safe to do something start
            
            //4. 这里是处理具体业务逻辑的地方。这里算作临界区内部，这里不用担心任何并发，可以随意执行”非原子操作“
            
            //5.safe to do something ending
            
            state=1; //6.处理成功以后我们可以安全的，不用任何修饰的修改state的值
            		//6.这里我修改为1,没有什么特别的含义。仅仅表示一种独特的状态。你可以（根据业务需要）将state修改为任意值。
            
            return true; //7. 处理成功以后，我们就要退出死循环了。这里可以是break（然后处理其他逻辑）。
            			//7. 也可以直接return（根据需要）
        }
    }
}
```



值得注意的是： 上面[模板](https://marketing.csdn.net/p/3127db09a98e0723b83b2914d9256174?pId=2782?utm_source=glcblog&spm=1001.2101.3001.7020)代码"可能"存在死循环状态 。

如果state变为1以后，如果【后续】没有任何其他线程将state其变为0，那么将会死循环。

> 这个 “可能”想表达的含义是，这么写代码 , 没有语法错误，并且能正常执行，但它可能不符合业务需求。











## 2.4 顺序put优化volatile

在FutureTask中， state 过渡到终止状态以后，不会再发生改变。并且每一种状态都是独一无二的。



因此，可以使用 `putOrderedInte`  来优化 `putIntVolatile()` : 

```java
//FutureTask.java

public boolean cancel(boolean mayInterruptIfRunning) {
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                //有且只能有一个线程能够竞争到   INTERRUPTING 或 CANCELLED
                //因此晚一点儿让其他线程看到  从INTERRUPTING  -->  INTERRUPTED 是没问题的。
                //因为其他人没有竞争到INTERRUPTING 也就不会进行其他修改状态的操作
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();
    }
    return true;
}
```













# 3. java中的大内存

在java中申请数组，数组最大长度必须小于等于 `Integer.MAX_VALUE`



在考虑到内存充裕的情况下，常规手段java能申请到多大的内存空间处理数据？

- 对于  long[] 来说， 最多能够申请到 `long[Integer.MAX_VALUE]`  约等于16G的数据  8B * (2^31-1) /1024 /1024/1024  = 15.99GB
- 对于  int[] 来说， 最多能够申请到 `iny[Integer.MAX_VALUE]`  约等于8G的数据  4B * (2^31-1) /1024 /1024/1024  = 7.99GB
- 对于 byte[] 来说，最多能够申请到  `byte[]` 与等于1.9GB      1B * (2^31-1) /1024 /1024/1024  = 1.99GB



也就是说，我们可以通过  “使用`long[]`申请到超大的内存，然后把它看作byte来处理的方式" 来变相扩大byte数组的分配空间。



在java中想要这样做，需要借助Unsafe 访问数组 : 

















# 4. 一些其他的思考





## 4.1 什么场景下使用原子数组

AtomicIntegerArray保证了get操作的可见性，意味着对数组元素变化是敏感的。

如果某些业务， 严格依赖数组元素当前状态下的值，可能会用到。

```java
//AtomicIntegerArray.java
public final int get(int i) {
    return getRaw(checkedByteOffset(i));
}

private int getRaw(long offset) {
    return unsafe.getIntVolatile(array, offset);
}
```









# 5. 参考

https://segmentfault.com/a/1190000000441670
