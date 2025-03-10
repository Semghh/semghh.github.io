---
layout: post
title:  "Netty中组件及源码"
date:   2024-12-24 00:00:00
categories: java
tags: netty

---

* content
{:toc}
关于Netty中重要的组件 EventLoop, Pipeline , Handler , FastThreadLocal ,ByteBuf ，Allocator等组件源码分析。







# 1. 组件

## 1.1   DefaultChannelPipeline  组件

Pipeline是Netty异步处理的核心，开发者实现将Handler(处理回调)注册到Pipeline中。 Pipeline注册到EventLoop中，

当io.netty.channel.Channel监听到了IO事件以后，会将Message传播到 Pipeline中。



DefaultChannelPipeline是 ChannelPipeline的默认实现。





整个Pipeline就是责任链模式。

> 责任链设计思路的核心就是：每个handler都处理自己份内的事儿，当前的handler处理消息完毕，event传递到下一个handler。
>
> 使用责任链值得注意的点： Handler的先后顺序有讲究，需要特别注意前后Handler对Message的传递情况。
>
> Netty有入站、出站2种event区分。  “引用计数对象”的retain 和 release。







### `fireChannelRead()` 方法

Pipeline的`fireChannelRead()` 和  context 的`fireChannelRead()` 传递源头是不同的。

> pipeline 从head传播
>
> context 从下一个handler开始传播





每一个io.netty.channel.Channel都有独立的Pipeline。每一个Pipeline的Handler链都有 Head 和 Tail节点。



pipeline 从head传播:

```java
//DefaultChannelPipeline.java
	@Override
    public final ChannelPipeline fireChannelRead(Object msg) {
        //传入了head节点。
        AbstractChannelHandlerContext.invokeChannelRead(head, msg);
        return this;
    }
```





context 从下一个handler开始传播:

```java
//AbstractChannelHandlerContext.java
	@Override
    public ChannelHandlerContext fireChannelRead(final Object msg) {
        //从下一个支持channel_read 事件的handler，然后开始传播
        //findContextInbound是netty内部使用的,用于过滤 handler的
        //每个handler可以设置自己支持的event callback。
        invokeChannelRead(findContextInbound(MASK_CHANNEL_READ), msg);
        return this;
    }
```











### DefaultChannelPipeline写出数据的流程

`channel.writeAndFlush()` 调用了`AbstractChannel.write(Object,boolean,ChannelPromise)`和 `AbstractChannel.flush()`两个方法。

> `flush()` 方法做的操作就是将  ChannelOutboundBuffer内保存的消息标记输出，并调用 `AbstractChannel.flush0()`方法，向远程的socket写出数据。

如果调用pipeline的write方法，消息将通过TailContext层层向上传播，直到HeadContext 统一写出为止。



下面是源码分析：

```java
//AbstractChannelHandlerContext.java
//TailContext是  AbstractChannelHandlerContext  上下文的子类。因此writeAndFlush最后调用了这个write
public ChannelFuture writeAndFlush(Object msg, ChannelPromise promise) {
    write(msg, true, promise);//调用了自己的write()
    return promise;
}


private void write(Object msg, boolean flush, ChannelPromise promise) {
        ObjectUtil.checkNotNull(msg, "msg");
        try {
            if (isNotValidPromise(promise, true)) { //如果promise不合法，就取消写入
                ReferenceCountUtil.release(msg);
                // cancelled
                return;
            }
        } catch (RuntimeException e) {
            ReferenceCountUtil.release(msg);
            throw e;
        }
		//从 context链条中向前找，找到第一出站HandlerContext
        final AbstractChannelHandlerContext next = findContextOutbound(flush ?
                (MASK_WRITE | MASK_FLUSH) : MASK_WRITE);
        final Object m = pipeline.touch(msg, next);
        EventExecutor executor = next.executor();
        if (executor.inEventLoop()) {
            if (flush) {
                next.invokeWriteAndFlush(m, promise);
            } else {
                next.invokeWrite(m, promise);
            }
        } else {
            final WriteTask task = WriteTask.newInstance(next, m, promise, flush);
            if (!safeExecute(executor, task, promise, m, !flush)) {
                // We failed to submit the WriteTask. We need to cancel it so we decrement the pending bytes
                // and put it back in the Recycler for re-use later.
                //
                // See https://github.com/netty/netty/issues/8343.
                task.cancel();
            }
        }
    }

	
	void invokeWriteAndFlush(Object msg, ChannelPromise promise) {
        if (invokeHandler()) {
            invokeWrite0(msg, promise);
            invokeFlush0();
        } else {
            writeAndFlush(msg, promise);
        }
    }

    void invokeWrite(Object msg, ChannelPromise promise) {
        if (invokeHandler()) { //检测ChannelHandler. handlerAdded(ChannelHandlerContext)是否已经被调用。
            invokeWrite0(msg, promise);
        } else {
            write(msg, promise);
        }
    }

private void invokeWrite0(Object msg, ChannelPromise promise) {
        try {
            // DON'T CHANGE
            // Duplex handlers implements both out/in interfaces causing a scalability issue
            // see https://bugs.openjdk.org/browse/JDK-8180450
            final ChannelHandler handler = handler();
            final DefaultChannelPipeline.HeadContext headContext = pipeline.head;
            if (handler == headContext) {
                headContext.write(this, msg, promise);
            } else if (handler instanceof ChannelDuplexHandler) {
                ((ChannelDuplexHandler) handler).write(this, msg, promise);
            } else if (handler instanceof ChannelOutboundHandlerAdapter) {
                ((ChannelOutboundHandlerAdapter) handler).write(this, msg, promise);
            } else {
                ((ChannelOutboundHandler) handler).write(this, msg, promise);
            }
        } catch (Throwable t) {
            notifyOutboundHandlerException(t, promise);
        }
    }
```













#### 写出数据是否有线程安全问题？

`write()` 、`writeAndFlush()` 方法，都会将写出的消息(Object对象) 通过pipeline层层处理传递，最终交付给 HeadContext对象。

虽然每个ChannelHandler可能绑定了不同的 executor，但最终写出操作都由`HeadContext`来完成。 HeadContext对象由pipeline创建，它绑定的executor是reactor线程。因此不会出现线程安全问题。











#### `writeAndFlush`比`write`多了什么？

AbstractChannel内拥有一个 Unsafe对象，它来完成数据的交换任务。

数据写出时，有一个缓冲区（ChannelOutboundBuffer），如果消息不flush的话，消息不会向socket发出。

`write()`消息时，会把消息添加到ChannelOutboundBuffer， 当调用flush以后，会把buffer内所有消息都发射出去。



HeadContext 的`write()`方法由Netty的Unsafe接口写出数据。

Unsafe负责io.netty.channel.Channel中底层的IO操作。

Unsafe 有一个 ChannelOutboundBuffer对象，是channel写出数据的缓冲区。

```java
//AbstructUnsafe的 write方法
public final void write(Object msg, ChannelPromise promise) {
            assertEventLoop();
			//拿到了内部的ChannelOutboundBuffer缓冲区
            ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
            if (outboundBuffer == null) {
                try {
                    // release message now to prevent resource-leak
                    ReferenceCountUtil.release(msg);
                } finally {
                    //如果ChannelOutboundBuffer是空，那么证明这个channel被关闭了。
                    //所以设置ChannelPromise为失败。
                    //如果ChannelOutboundBuffer不空，就处理剩余数据。将在flush0中处理。
                    safeSetFailure(promise,
                            newClosedChannelException(initialCloseCause, "write(Object, ChannelPromise)"));
                }
                return;
            }
            int size;
            try {
                msg = filterOutboundMessage(msg);
                //对消息大小进行评估，若无法评估返回<0的int
                size = pipeline.estimatorHandle().size(msg);
                if (size < 0) {
                    size = 0;
                }
            } catch (Throwable t) {
                try {
                    ReferenceCountUtil.release(msg);
                } finally {
                    safeSetFailure(promise, t);
                }
                return;
            }
			//只是将传入的message写入buffer中（而没有flush）。一旦消息完成写入以后，promise将会收到通知
            outboundBuffer.addMessage(msg, size, promise);
        }
```





```java
//ChannelOutboundBuffer.java
public void addMessage(Object msg, int size, ChannelPromise promise) {
    Entry entry = Entry.newInstance(msg, size, total(msg), promise);
    if (tailEntry == null) {
        flushedEntry = null;
    } else {
        Entry tail = tailEntry;
        tail.next = entry;
    }
    tailEntry = entry;
    if (unflushedEntry == null) {
        unflushedEntry = entry;
    }

    // Touch the message to make it easier to debug buffer leaks.

    // this save both checking against the ReferenceCounted interface
    // and makes better use of virtual calls vs interface ones
    if (msg instanceof AbstractReferenceCountedByteBuf) {
        ((AbstractReferenceCountedByteBuf) msg).touch();
    } else {
        ReferenceCountUtil.touch(msg);
    }

    // increment pending bytes after adding message to the unflushed arrays.
    // See https://github.com/netty/netty/issues/1619
    incrementPendingOutboundBytes(entry.pendingSize, false);
}
```



当真正调用了 `AbstractUnsafe.flush()` 标记消息的flush状态，并`flush0()` 以后，才会调用`doWrite()`写到remote socket

```java
public final void flush() {
    assertEventLoop();

    ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
    if (outboundBuffer == null) {
        return;
    }

    outboundBuffer.addFlush();
    flush0();
}

@SuppressWarnings("deprecation")
protected void flush0() {
    if (inFlush0) {
        // Avoid re-entrance
        return;
    }

    final ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
    if (outboundBuffer == null || outboundBuffer.isEmpty()) {
        return;
    }

    inFlush0 = true;

    // Mark all pending write requests as failure if the channel is inactive.
    if (!isActive()) {
        try {
            // Check if we need to generate the exception at all.
            if (!outboundBuffer.isEmpty()) {
                if (isOpen()) {
                    outboundBuffer.failFlushed(new NotYetConnectedException(), true);
                } else {
                    // Do not trigger channelWritabilityChanged because the channel is closed already.
                    outboundBuffer.failFlushed(newClosedChannelException(initialCloseCause, "flush0()"), false);
                }
            }
        } finally {
            inFlush0 = false;
        }
        return;
    }

    try {
        doWrite(outboundBuffer);
    } catch (Throwable t) {
        handleWriteError(t);
    } finally {
        inFlush0 = false;
    }
}
```



























## 1.2 TailContext 组件

DefaultChannelPipeline的尾节点类型 TailContext。



在Handler消费一个ReferenceCounted对象时，必须要先调用`retain()`防止它被意外回收。

这么做的原因在于，DefaultChannelPipeline的尾节点，默认会释放1次ReferenceCounted对象的引用。



```java
//TailContext
	protected void onUnhandledInboundMessage(Object msg) {
        try {
            logger.debug(
                    "Discarded inbound message {} that reached at the tail of the pipeline. " +
                            "Please check your pipeline configuration.", msg);
        } finally {
            ReferenceCountUtil.release(msg);
        }
    }
```











## 1.3 DefaultChannelHandlerContext 组件

默认的HandlerContext对象。

由于Pipeline的存在，在这条链上的Handler都有上下游的关系。 而有些Handler是 Shareable的，向上下游传播状态信息需要则需要借助 Context。



万幸的是Netty提供了这样的功能（当然，对于非Shareable的Handler ， Netty更推荐成员内部类的方式，它更直观简洁）。



使用` attr()` 在上下文之间传递状态。

```java
//DefaultChannelHandlerContext.java
	public <T> Attribute<T> attr(AttributeKey<T> key) {
        //实际上是 io.netty.channel.Channel.attr() 在工作
        return channel().attr(key);
    }
```



context的attr绑定在了 Channel中 ,  io.netty.channel.Channel 继承了DefaultAttributeMap。

DefaultAttributeMap 是属性存储的实现。













## 1.4  ChannelOutboundBuffer









### 为什么要设计这个Buffer

1. 首先他是一个buffer，拥有buffer的作用和好处。

2. 通过高水位线，控制channel是否不可写。

   当超过高水位线后。 unwritable 标志位将设置非0值。 只有unwritable==0时才表示channel可写。

3. 触发`ChannelWritabilityChanged`（通道可写状态改变事件）。

   Netty设计了一个 channel可写数据大小的水位线。

   在ChannelOutboundBuffer中的消息实际还没有被写入socket中。

   当ChannelOutboundBuffer中等待写的数据大小超过水位线以后，将更新 unwritable 的状态。当由0变化为>0时，将会触发`ChannelWritabilityChanged`事件。

   ```java
   //ChannelOutboundBuffer
       
   //添加消息。但没有flush。主要是在更新 消息Entry的状态。
   	public void addMessage(Object msg, int size, ChannelPromise promise) {
           Entry entry = Entry.newInstance(msg, size, total(msg), promise);
           if (tailEntry == null) {
               flushedEntry = null;
           } else {
               Entry tail = tailEntry;
               tail.next = entry;
           }
           tailEntry = entry;
           if (unflushedEntry == null) {
               unflushedEntry = entry;
           }
   
           if (msg instanceof AbstractReferenceCountedByteBuf) {
               ((AbstractReferenceCountedByteBuf) msg).touch();
           } else {
               ReferenceCountUtil.touch(msg);
           }
   
           // increment pending bytes after adding message to the unflushed arrays.
           // See https://github.com/netty/netty/issues/1619
           incrementPendingOutboundBytes(entry.pendingSize, false);
       }
   
   	//增加 还没输出数据的占用的字节数量。
   	private void incrementPendingOutboundBytes(long size, boolean invokeLater) {
           if (size == 0) {
               return;
           }
   
           long newWriteBufferSize = TOTAL_PENDING_SIZE_UPDATER.addAndGet(this, size);
           //当占用字节数量超过了高水位。
           if (newWriteBufferSize > channel.config().getWriteBufferHighWaterMark()) {
               //更改 unwritable状态位的值
               setUnwritable(invokeLater);
           }
       }
   
       private void setUnwritable(boolean invokeLater) {
           for (;;) {
               final int oldValue = unwritable;
               final int newValue = oldValue | 1; //按位或。 无论原本值是几，最终结果值一定大于等于1
               if (UNWRITABLE_UPDATER.compareAndSet(this, oldValue, newValue)) {
                   if (oldValue == 0) {
                       //当由0变为其他值时，触发通道可写性事件。是否稍后执行传播。false表示立即传播。
                       fireChannelWritabilityChanged(invokeLater);
                   }
                   break;
               }
           }
       }
   
   	private void fireChannelWritabilityChanged(boolean invokeLater) {
           final ChannelPipeline pipeline = channel.pipeline();
           if (invokeLater) {
               //稍后执行，添加到任务队列里。
               Runnable task = fireChannelWritabilityChangedTask;
               if (task == null) {
                   fireChannelWritabilityChangedTask = task = new Runnable() {
                       @Override
                       public void run() {
                           pipeline.fireChannelWritabilityChanged();
                       }
                   };
               }
               channel.eventLoop().execute(task);
           } else {
               //立即执行（会在addMessage时立即执行）。
               pipeline.fireChannelWritabilityChanged();
           }
       }
   ```

   













### 写数据流程

通过`AbstrachChannel`的`writeAndFlush()` 方法，我们知道期望写出的Messge，先通过pipeline的tail层层传递到 headContext，

然后通过 ChannelOutboundBuffer，触发高水位线，改变 `isWritable()`的返回值。

随后，调用了留给子类实现的抽象方法`Abstract.doWrite()`  来写出数据。



下面，我们看一下`NioSocketChannel.doWrite()`的实现。

```java
protected void doWrite(ChannelOutboundBuffer in) throws Exception {
    SocketChannel ch = javaChannel();
    //获得自旋锁的次数
    int writeSpinCount = config().getWriteSpinCount();
    do {
        if (in.isEmpty()) {
			//清除 OP_WRITE感兴趣事件
            clearOpWrite();
            return;
        }

        // Ensure the pending writes are made of ByteBufs only.
        int maxBytesPerGatheringWrite = ((NioSocketChannelConfig) config).getMaxBytesPerGatheringWrite();
        ByteBuffer[] nioBuffers = in.nioBuffers(1024, maxBytesPerGatheringWrite);
        int nioBufferCnt = in.nioBufferCount();

        // Always use nioBuffers() to workaround data-corruption.
        // See https://github.com/netty/netty/issues/2761
        switch (nioBufferCnt) {
            case 0:
                // We have something else beside ByteBuffers to write so fallback to normal writes.
                writeSpinCount -= doWrite0(in);
                break;
            case 1: {
                // Only one ByteBuf so use non-gathering write
                // Zero length buffers are not added to nioBuffers by ChannelOutboundBuffer, so there is no need
                // to check if the total size of all the buffers is non-zero.
                ByteBuffer buffer = nioBuffers[0];
                int attemptedBytes = buffer.remaining();
                final int localWrittenBytes = ch.write(buffer);
                if (localWrittenBytes <= 0) {
                    incompleteWrite(true);
                    return;
                }
                adjustMaxBytesPerGatheringWrite(attemptedBytes, localWrittenBytes, maxBytesPerGatheringWrite);
                in.removeBytes(localWrittenBytes);
                --writeSpinCount;
                break;
            }
            default: {
                // Zero length buffers are not added to nioBuffers by ChannelOutboundBuffer, so there is no need
                // to check if the total size of all the buffers is non-zero.
                // We limit the max amount to int above so cast is safe
                long attemptedBytes = in.nioBufferSize();
                final long localWrittenBytes = ch.write(nioBuffers, 0, nioBufferCnt);
                if (localWrittenBytes <= 0) {
                    incompleteWrite(true);
                    return;
                }
                // Casting to int is safe because we limit the total amount of data in the nioBuffers to int above.
                adjustMaxBytesPerGatheringWrite((int) attemptedBytes, (int) localWrittenBytes,
                        maxBytesPerGatheringWrite);
                in.removeBytes(localWrittenBytes);
                --writeSpinCount;
                break;
            }
        }
    } while (writeSpinCount > 0);

    incompleteWrite(writeSpinCount < 0);
}
```























## 1.5 FastThreadLocal

FastThreadLocal是   ThreadLocal的一个变种。  使用FastThreadLocalThread(继承自Thread，内部有InternalThreadLocalMap)来访问FastThreadLocal时可以提供更好的访问性能 。





FastThreadLocal为什么快？

FastThreadLocal 通过索引常量，从数组中读取数据。 而不是使用 hashcode ,hash table 来寻找变量 ，每一个ThreadLocal对象都存储了一个int类型的索引。表示他在InternalThreadLocalMap中的位置。



虽然比使用hashtable只有轻微的性能优势，  但在频繁访问ThreadLocal的场景下，非常有用。





为了使用这样的优势（FastThreadLocal），你必须使用 FastThreadLocalThread或它的子类。



默认情况下，由io.netty.util.concurrent.DefaultThreadFactory创建的线程都是 FastThreadLocalThread 的原因也是如此（获得更高的ThreadLocal性能）。



reference https://zhuanlan.zhihu.com/p/662172520



必须是Thread的子类FastThreadLocalThread才可以使用 FastThreadLocal。

每一个FastThreadLocal对象都有一个 InternalThreadLocalMap的内部成员，以及 一个 Int类型表示index , 在InternalThreadLocalMap指定索引的位置存放ThreadLocal存储的数据。

> 详细流程参看 `ThreadLocal.get()`方法





FastThreadLocal 允许存储null值 ，null代表了一种合法的存储值。











### InternalThreadLocalMap

FastThreadLocal内部存储映射，是FastThreadLocal的设计核心。



所有的存储对象，都存放在了 `Object[]` 数组中。

```java
private Object[] indexedVariables;
```















### `get()` 

首先拿到FastThreadLocalThread中的存储结构 InternalThreadLocalMap ，然后在 index位置上，获取数据。

若该位置为UNSET表示这个ThreadLocal还没有设置值或者值已经被清空。(注意，FastThreadLocal中可以存储一个null ，null可以拥有实际的业务意义。)

没有设置值，或者值被清空时， 存储的是UNSET对象。 我们可以`ThreadLocal.set(null);`来表示业务上这个值设置过，并且为null。

```java
public final V get() {
    InternalThreadLocalMap threadLocalMap = InternalThreadLocalMap.get();
    Object v = threadLocalMap.indexedVariable(index);
    if (v != InternalThreadLocalMap.UNSET) {
        return (V) v;
    }
    //初始化一个默认值。
    return initialize(threadLocalMap);
}
```















### `removeAll()`

移除绑定到当前线程的全部ThreadLocal。

ThreadLocalMap中有一个slot存储了所有的ThreadLocal。 Slot中存放了一个`Set<ThreadLocal>` ,在 `ThradLocal.set()`时都会添加到Set中。

```java
public static void removeAll() {
        InternalThreadLocalMap threadLocalMap = InternalThreadLocalMap.getIfSet();
        if (threadLocalMap == null) {
            return;
        }

        try {
            //从slot中拿到所有的ThreadLocal，依次移除。
            Object v = threadLocalMap.indexedVariable(VARIABLES_TO_REMOVE_INDEX);
            if (v != null && v != InternalThreadLocalMap.UNSET) {
                @SuppressWarnings("unchecked")
                Set<FastThreadLocal<?>> variablesToRemove = (Set<FastThreadLocal<?>>) v;
                FastThreadLocal<?>[] variablesToRemoveArray =
                        variablesToRemove.toArray(new FastThreadLocal[0]);
                for (FastThreadLocal<?> tlv: variablesToRemoveArray) {
                    tlv.remove(threadLocalMap);
                }
            }
        } finally {
            InternalThreadLocalMap.remove();
        }
    }
```

























# 参考

