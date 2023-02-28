# 1.使用JProfiler







## 1.1 开启dump文件自动导出

JProfiler可以分析 `.dump`的堆信息文件。



Java支持手动导出 dump文件，以及当OOM发生时自动导出dump文件。当线上生产出现OOM的时刻是未知的,所以自动导出`dump`文件是必须的。



### 1.1.1手动导出

```
jmap -dump:format=b,file=heap.hprof <pid>
```



![image-20221015113351756](JProfiler.assets/image-20221015113351756.png)



### 1.1.2 当OOM时自动导出

需要在启动app时添加VM参数

```
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heapdump.hprof	
```



此外，OnOutOfMemoryError参数允许用户指定当出现oom时，指定某个脚本来完成一些动作。



```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -XX:OnOutOfMemoryError="sh ~/test.sh"
```




