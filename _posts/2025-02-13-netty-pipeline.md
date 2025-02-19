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





# 1.    DefaultChannelPipeline  组件

Pipeline是Netty异步处理的核心，开发者实现将Handler(处理回调)注册到Pipeline中。 Pipeline注册到EventLoop中，

当io.netty.channel.Channel监听到了IO事件以后，会将Message传播到 Pipeline中。



DefaultChannelPipeline是 ChannelPipeline的默认实现。





整个Pipeline就是责任链模式。

> 责任链设计思路的核心就是：每个handler都处理自己份内的事儿，当前的handler处理消息完毕，event传递到下一个handler。
>
> 使用责任链值得注意的点： Handler的先后顺序有讲究，需要特别注意前后Handler对Message的传递情况。
>
> Netty有入站、出站2种event区分。  “引用计数对象”的retain 和 release。







## 1.1  `fireChannelRead()` 方法

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











# 2. DefaultChannelHandlerContext 组件

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



context的attr绑定在了 Channel中。





















# 参考

