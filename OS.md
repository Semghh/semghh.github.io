





什么是协程？一句话描述 : 用户态下轻量级的线程。哪里轻量级？

```
线程之间的切换，需要OS的内核态和系统态的相互切换，这需要很大开销。

而协程是在线程内部，由线程自己实现上下文切换调度的工作单位。包括定义一些“执行现场”“执行入口”等。

这意味者，协程之间的调度不会发生系统内核的切换。全部在一个线程内完成。
```







C/C++的内存分配？栈和堆的区别？为什么栈快？

https://blog.csdn.net/baidu_37964071/article/details/81428139

```
栈快。
栈更符合程序局部性原理，栈有专门的寄存器，压栈和出栈效率很高。堆需要由OS动态调度。
```

