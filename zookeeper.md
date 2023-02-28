# 1.入门







## 1.1 Zookeeper 命令





### 1.1.1 服务命令

首先需要准备好相应配置。

```
通常在zookeeper的conf文件夹下,会有一个官方给出的简单配置文件示例(zoo_sample.cfg)
```

![image-20220809082645358](zookeeper.assets/image-20220809082645358.png)



运行bin/zkServer.sh



zkServer.sh支持多个服务命令:

```sh
zkServer.sh start    #启动
zkServer.sh status   #查看zk状态
zkServer.sh stop     #停止
zkServer.sh	restart  #重启zk
```







### 1.1.2  zk客户端命令

首先使用zkCli.sh 启动zk的客户端。

```
zk使用 zkCli客户端和 zk集群交互。 zkCli支持一些独有的命令来操纵zk集群。
```



客户端命令:

```zookeeper
ls /             #查看路径   /为根路径

create /zk "test"           #创建一个新的节点，以及语气关联的 "test"字符串
get /zk                     #获得 /zk节点的内容
set /zk                     #设置 /zk节点的内容
delete /zk                  #删除该节点
```









