# Java 网络编程





# 1. TCP 编程



![image-20220430092003530](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220430092003530.png)





学习目标:

连接一个服务器

创建一个服务器











## 1.1. 连接到服务器



### 1.1.1 使用Socket类连接





```java
public class SocketTest {

    public static void main(String[] args) {
        try (Socket socket = new Socket("time-A.timefreq.bldrdoc.gov",13)){
            InputStream inputStream = socket.getInputStream();
            byte[] buffer = new byte[1024];

            int len;
            while ((len = inputStream.read(buffer)) != -1) {
                System.out.println(new String(buffer,0,len));
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```



输出结果

![image-20220430092018274](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220430092018274.png)



```
使用Socket类建立一个 套接字连接。
```



![image-20220429103136802](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429103136802.png)





#### 1.1.1.1 socket的构造方法







在1.1中使用了Socket的这个构造方法

```
public Socket(String host,int port) ;//指定 主机host, 和端口号
```



![image-20220430092027151](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220430092027151.png)

```
事实上new了一个 InetSocketAddress (Internet网络套接字地址) 
```







除此以外的构造方法:



![image-20220430092033898](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220430092033898.png)

可以传入一个代理Proxy

也可以传入一个InetAddress  因特网地址。



InetAddress 的方法不是public，而是提供了 getByName() 获得InetAddress实例对象

![image-20220430092042122](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220430092042122.png)





```java
public static void main(String[] args) {

        InetAddress byName;
        try {
            byName= InetAddress.getByName("time-A.timefreq.bldrdoc.gov");
        } catch (UnknownHostException e) {
            throw new RuntimeException(e);
        }
        
        try (Socket socket = new Socket(byName,13)){ //传入InetAddress 构建Socket
            InputStream inputStream = socket.getInputStream();
            byte[] buffer = new byte[1024];

            int len;
            while ((len = inputStream.read(buffer)) != -1) {
                System.out.println(new String(buffer,0,len));
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```



![image-20220430092049252](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220430092049252.png)





### 1.1.2 Socket 发送数据





![image-20220429103933092](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429103933092.png)



```
Socket可以通过 getOutputStream()方法获取一个输出流
```









### 1.1.3 套接字超时



![image-20220429104244461](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429104244461.png)



单位毫秒



```java

    public static void main(String[] args) {


        try (Socket socket = new Socket("www.baidu.com",443)){

            socket.setSoTimeout(10000);

            byte[] buffer = new byte[1024];
            InputStream inputStream = socket.getInputStream();

            int len;
            while ((len = inputStream.read(buffer)) != -1){

                System.out.println(new String(buffer,0,len));
            }


        } catch (IOException e) {
            throw new RuntimeException(e);
        }

    }
```





![image-20220429104850079](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429104850079.png)







![image-20220429111729276](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429111729276.png)





![image-20220429112558563](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429112558563.png)







### 1.1.4 因特网地址

直译就是InetAddress 类 ，事实上在前文我们提到过InetAddress 类

![image-20220429112707503](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429112707503.png)



```
使用InetAddress 把主机名和Ip地址相互转换
```





#### 1.1.4.1 InetAddress的方法







```
getByName()  //获得域名的一个ip地址
```







```
getAllByName()  //获得域名的全部ip地址
```



```java
        ...省略
		try {
            InetAddress[] allByName = InetAddress.getAllByName("www.baidu.com");
            for (InetAddress inetAddress : allByName) {
                System.out.println("host Address "+ inetAddress.getHostAddress());
                System.out.println("host name "+inetAddress.getHostName());
                
            }
        } catch (UnknownHostException e) {
            throw new RuntimeException(e);
        }
		...省略
```

![image-20220429114429612](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429114429612.png)

```
www.baidu.com 就有多个
```



<img src="C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429114537712.png" alt="image-20220429114537712" style="zoom:50%;" />



```
getHostAddress() //获得主机的ip地址

getHostName()// 获得主机名(域名)
```





```
getLocalHost()  //获得本机的ip地址。不是127.0.0.1
```



![image-20220429114820345](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429114820345.png)



```java
        try {
            InetAddress localHost = InetAddress.getLocalHost();

            System.out.println(localHost.getHostName());
            System.out.println(localHost.getHostAddress());

        } catch (UnknownHostException e) {
            throw new RuntimeException(e);
        }
```



运行结果:

![image-20220429115138227](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429115138227.png)







### 1.1.5 Socket半关闭





![image-20220429144202575](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429144202575.png)



![image-20220429144225061](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429144225061.png)











## 1.2. 实现服务器



### 1.2.1 使用ServerSocket 类做服务器





实现一个简单的EchoServer，原样返回Client输入的信息。当遇到exit ，退出服务。

#### 1.2.1.1 EchoServer

```java
    public static void main(String[] args) {


        try (ServerSocket echoServer = new ServerSocket(8888);) {
            while (true) {
                Socket accept = echoServer.accept();

                clientConnectionConsoleLog(accept);

                new Thread(new serverWorkHandler(accept)).start();

            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```



```
使用ServerSocket开启一个服务器。  ServerSocket.accept()是一个阻塞方法。直到有Client和Server建立了连接，阻塞方法accept就会返回。

然后调用clientConnectionConsoleLog() 方法。打印一下连接日志。

但后启动一个Thread为Client服务。其中//serverWorkHandler 是Server服务器的主要工作内容
```



```java
private static void clientConnectionConsoleLog(Socket socket) {

    InetAddress clientInetAddr = socket.getInetAddress();

    String clientHostName = clientInetAddr.getHostName();

    String clientHostAddress = clientInetAddr.getHostAddress();

    System.out.println("get Connection from [" + clientHostName + " " + clientHostAddress
            + ":" + socket.getPort() + "]");

}
```

```
向控制台打印Socket连接。
```





```java
static class serverWorkHandler implements Runnable {

    private Socket socket;

    public serverWorkHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {

        try(BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(),StandardCharsets.UTF_8));
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream(),StandardCharsets.UTF_8))){

            while (true){

                String s = bufferedReader.readLine();
                if (s.equals("exit")){
                    break;
                }else {
                    bufferedWriter.write(s);
                    bufferedWriter.newLine();
                    bufferedWriter.flush();
                }
            }


        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        System.out.println("close Connection from " + socket.getInetAddress());

        try {
            socket.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```
实现了Runnable
使用BufferedWriter 作为输入
使用BufferedReader 作为输出

BufferedWriter 输出以后,要调用newLine()方法,再flush输出一下
```



运行截图：

![image-20220429143826604](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429143826604.png)













#### 1.2.1.2 EchoClient



```java
public static void main(String[] args) {


        Scanner in = new Scanner(System.in);
        try (Socket socket = new Socket("localhost", 8888);
             BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
             BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {
            while (true){
                String s = in.nextLine();
                System.out.println("send to Server : "+s);
                bufferedWriter.write(s);
                bufferedWriter.newLine();
                bufferedWriter.flush();


                String read = bufferedReader.readLine();
                System.out.println("get from Server : "+ read);

                if (s.equals("exit")){
                    break;
                }
            }


        } catch (UnknownHostException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```



```
使用Socket建立连接。
使用Scanner接收来自控制台的 字符串。

使用BufferedWriter 作为输入，向Server输出
使用BufferedReader 作为输出,接收来自Server的输入
```



运行截图

![image-20220429143835842](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429143835842.png)













# 2.UDP 编程









# 3.发送Email



发送Email的协议：SMTP 

Simple Mail Transfer Protocol  简单邮件传输协议。默认使用25号端口，或者加密端口465/587

收发邮件都将遵守SMTP协议。



SMTP 协议是建立在TCP协议之上的，我们无需关心SMTP协议底层是如何实现。对于Java语言来说，使用JavaMail  的API即可完成发送邮件。





## 3.1 Email是这样发的



![image-20220429151013629](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429151013629.png)

像Outlook , QQ邮箱写信的客户端，就是Mail User Agent ，用户邮件代理。提供一个友好的界面，提供编辑修改功能，包装用户的Email信息，使之符合SMTP协议。并将Mail发送到 MTA



MTA  Mail Transfer Agent ，邮件传递代理，帮助邮件转发到MDA。


MDA Mail Delivery Agent ，邮件投递代理。真正把用户邮件存储的服务器。用户的邮件会存储在MDA的磁盘中，并负责把邮件正确的分发给对应的收件人。





## 3.2 邮件提供商

常见的邮件提供商

![image-20220429154225154](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429154225154.png)



事实上，我们也无需知道邮件到底经过多少个 MTA，最终到达哪个MDA。我们只需要将写好的邮件按照SMTP协议的格式 发送至【邮件提供商】的【指定端口】即可。

```
由邮件提供商保证邮件的分发,投递，最终交付给正确的收件人。
```



## 3.3 JavaMail











# 4. Http编程

















## 4.1 Http请求

HTTP请求的格式是固定的，它由HTTP Header和HTTP Body两部分构成。第一行总是`请求方法 路径 HTTP版本`



后续的每一行都是固定的`Header: Value`格式，我们称为HTTP Header，服务器依靠某些特定的Header来识别客户端请求。

例如：

![image-20220429182554827](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429182554827.png)



#### 4.1.1 Get和Post的区别



如果是Get则没有请求体。如果是Post请求则包含请求体。

例如:

![image-20220429182700668](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429182700668.png)





Get请求参数只能在URL中，使用URLEncode编码。URL长度限制，Get请求参数不能过多。

Post参数长度没有限制，Post请求参数必须放在 请求体中。

Post请求参数不一定是URLEncode，只需要在Content-Type中表明即可。



如一个JSON  Post请求:

![image-20220429183013082](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429183013082.png)





## 4.2 HTTP 响应



Http响应 由2部分组成: Head 和 Body



响应的第一行总是  `HTTP版本 响应代码 响应说明`

例如，`HTTP/1.1 200 OK`表示版本是`HTTP/1.1`，响应代码是`200`，响应说明是`OK`。客户端只依赖响应代码判断HTTP响应是否成功。




![image-20220429183948048](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429183948048.png)



一个HTTP响应

![image-20220429184213215](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429184213215.png)



```
在浏览器收到服务器发来的Http响应以后，总是会继续发出一系列HTTP请求。

Http的无状态性,决定了服务器只能被动的接收 客户端的请求。
无法实时向用户推送消息。此时可以使用WebSocket协议
```







## 4.3 Http 1.0 /1.1/2.0



http1.1

```
对于HTTP/1.0协议，每次发送一个HTTP请求，客户端都需要先创建一个新的TCP连接.然后，收到服务器响应后，关闭这个TCP连接。

TCP建立的过程比较耗时，所以HTTP1.1 允许在一个TCP连接中反复发送 请求-响应。大大提高效率
```



1.1示意图:

<img src="C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429184935561.png" alt="image-20220429184935561" style="zoom:67%;" />











http 2.0

```
因为HTTP协议是一个请求-响应协议，客户端在发送了一个HTTP请求后，必须等待服务器响应后，才能发送下一个请求，这样一来，如果某个响应太慢，它就会堵住后面的请求。

所以，为了进一步提速，HTTP/2.0允许客户端在没有收到响应的时候，发送多个HTTP请求，服务器返回响应的时候，不一定按顺序返回，只要双方能识别出哪个响应对应哪个请求，就可以做到并行发送和接收：
```



![image-20220429185105959](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429185105959.png)











## 4.4 JDK 中的HTTP编程



jdk11以前的HTTP



```java
public static void main(String[] args) {


        try {
            URL url = new URL("https","www.baidu.com",443,"/"); //创建URL 请求路径/
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            //开启一个连接
            connection.setRequestMethod("GET");//设置请求方法GET
            connection.setUseCaches(false); //不使用缓存
            connection.setConnectTimeout(5000); //设置连接超时时间
            connection.setReadTimeout(5000);//设置读操作超时时间
            connection.setRequestProperty("Accept", "*/*");
            //设置请求头
            connection.setRequestProperty("User-Agent", "Mozilla/5.0 (compatible; MSIE 11; Windows NT 5.1)");
            connection.connect(); //创建连接

            if (connection.getResponseCode()!=200) {
                throw new RuntimeException("bad Response"); //如果响应码不是200则抛出异常
            }
            
            Map<String, List<String>> headerFields = connection.getHeaderFields();
            //获得全部的响应头
            headerFields.forEach((x,y)-> System.out.println(x + " : " +y));
            //输出响应头


            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));

            while (reader.ready()) {
                System.out.println(reader.readLine());//写Response Body
            }


        } catch (MalformedURLException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }


    }
```



![image-20220429221114345](C:\Users\SemgHH\Desktop\笔记\1.网络.assets\image-20220429221114345.png)







# 5. RMI 远程调用



Java的RMI远程调用是指，一个JVM中的代码可以通过网络实现远程调用另一个JVM的某个方法。RMI是Remote Method Invocation的缩写。



提供服务的一方我们称之为服务器，而实现远程调用的一方我们称之为客户端。





## 5.1 一个简单例子



要实现RMI，服务器和客户端必须共享同一个接口。



Java的RMI规定此接口必须派生自`java.rmi.Remote`，并在每个方法声明抛出`RemoteException`。



现在模拟实现一个加法的远程过程。





定义一个接口 AddWork 继承 Remote

```java
public interface AddWork extends Remote {

    int add(int a, int b) throws RemoteException;

}
```





在服务器端，继承并实现 AddWork接口

```java
public class AddWorkService implements AddWork{

    @Override
    public int add(int a, int b) throws RemoteException {
        return a + b;
    }
}
```



在服务器端，注册部署RMI

```java
    public static void main(String[] args) {

        AddWork addWorkService = new AddWorkService();
        //服务器端需要创建实现类的  实例对象

        try {
            AddWork remote = (AddWork) UnicastRemoteObject.exportObject(addWorkService, 0);

            Registry registry = LocateRegistry.createRegistry(8888);
            //创建一个注册，指定接口

            registry.rebind("add",remote);
            //重新绑定 , 绑定  远程过程服务名 和 对应的实例对象


        } catch (RemoteException e) {
            throw new RuntimeException(e);
        }

    }
```







客户端必须共用 addWork接口。也就是必须拷贝一份 addWork.java



客户端的调用:

```java
    public static void main(String[] args) throws RemoteException, NotBoundException {


        Registry registry = LocateRegistry.getRegistry("localhost", 8888);
        //连接到指定主机的指定端口
        
        AddWork add = (AddWork) registry.lookup("add");
        //调用lookup()   通过 服务名调用接口。反序列化出来一个 Addwork对象

        int result = add.add(1, 2);
        //通过对象.方法   

        System.out.println(result);


    }
```







## 5.2 RMI总结



RMI   (Remote Method Invocation ) 直译过来是 远程方法调用。 但是通过前面的例子可以知道，JDK的RMI事实上是一个 “远程对象创建”。

```
addWorkSerive.java  对于客户端一开始来说并不知道是怎么实现的。但是通过RMI 最终在客户端的本地反序列化了一个实例对象。
然后在客户端的本地调用这个对象的方法,达到了远程方法的调用过程。

也就是说,无论是服务端发来的什么样的实现类代码。最终都会通过网络传输到客户端,然后反序列化成对象，最终调用。
```







### 5.2.1 优点



客户端无需知道,存储 远程方法的具体实现。只需要动态的向服务器发送正确的请求，即可完成方法调用。是分布式的基础。







### 5.2.2 缺点





```
Java的反序列化，因为生成了对象。同时代码也是在本地运行的，会有严重的安全问题。(除了设计到调用相关的数据，还传输了二进制字节码文件，这样会传输恶意代码)

使用RMI时必须是内网相互信任的机器。
```



RMI决定了调用的只能 双方都是Java程序，而且需要共用接口。

```
如果跨语言,就需要使用更通用的协议。如 gRPC
```







