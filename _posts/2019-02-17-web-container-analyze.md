---
layout: post
title: 'WEB容器透视'
subtitle: 'WEB容器TOMCAT,Jetty深入了解'
date: 2019-02-17
categories: WEB
author: yates
cover: 'www.baidu.com'
tags: WEB
---

# 前言
JBoss和Weblogic等重量级的应用服务器在微服务架构日益盛行的今天略显笨重，因此tomcat和jetty就是一个很好地选择

在**HTTP1.1**中，请求是按**顺序排队处理**的，前面的HTTP请求处理会**阻塞**后面的HTTP请求，虽然**HTTP pipelining**对连接请求做了改善，但是**复杂度太大**，并没有普及，这个问题在HTTP2.0中得到了解决

服务端会设置连接**超时时间**，如果TCP连接上超过一段时间没有请求数据，服务端会**关闭连接**

敏感的用户信息没有在网络上传输了，但是攻击者拿到**sessionid**也可以**冒充**受害者**发送请求**，这就是为什么我们需要https，**加密**后攻击者就拿不到sessionid了，另外**CSRF**也是一种防止session劫持的方式

tomcat配置server.xml 中**Connector**标签的**maxThreads**是线程池中**线程数**

**spring boot**配置文件中的**server.tomcat.max-connections**也是**tcp连接**的最大值

tomcat维护一个tcp连接池响应所有客户端的请求。虽然**一个tcp请求**可以同时和**多个客户端建立连接**，但是同一个tcp连接在**同一时间只能服务一个客户端请求**，只有这个请求处理完后才会响应其他的请求。类似于每个**连接**都分配一个**阻塞队列**

## tomcat日志

**日志分类**

- **运行日志**，记录**运行过程**中一些信息，尤其是一些**异常错误**日志信息。
- **访问日志**，记录**访问时间，IP地址，访问路径**等信息

**catalina.{}.log**：记录tomcat**启动过程信息**，可以看到启动的JVM参数以及操作系统**配置**等日志信息

**catalina.out**：tomcat**标准输出和标准错误**，如果没有修改tomcat启动脚本，stdout和stderr会重定向到这里。

**localhost.{}.log**：记录web应用在**初始化**过程中遇到的未处理的**异常**

**localhost_access_log.{}.txt**：存放访问tomcat的**请求**日志，包括IP地址以及请求路径，时间，请求协议，状态码

**manager.{}.log/host-manager.{}.log**：存放tomcat**自带manager**项目日志信息

## tomcat支持I/O模型，协议

tomcat支持I/O模型

- NIO：非阻塞I/O，采用java NIO类库实现
- NIO2: 异步I/O，采用JDK7最新NIO2类库实现
- APR: 采用apache可移植运行库实现，是C/C++编写本地库

tomcat支持应用层协议：

- HTTP/1.1:
- AJP:用于和web服务器集成
- HTTP/2：

## **连接器**

tomcat**一个容器**可对接**多个连接器**，组成被称为**Service组件**。一个Tomcat可有多个service，通过**不同端口**访问同一个tomcat上不同应用

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/2.png)

**连接器**对servlet容器**屏蔽了协议**和**I/O模型**的区别，统一向容器传入servletrequest对象，并从servlet接受对象servletresponse将其转换出去

**连接器**的设计符合了我们经常讲的**高内聚，低耦合**原则

高类聚功能：网络通信（endpoint），应用层协议解析（processor），tomcatrequest/response 与servletrequest/response的转换（adapter）

由于**IO模型和协议**之间的组合有很多种，组件之间通过抽象接口交互，tomcat设计了一个**protocolhandler**接口来封装这两种变化点。

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/3.png)

可以清晰看出它们之间继承和层次关系，尽量将稳定的部分放到抽象基类，每一个io模型和协议组合都有相应实现类

**endpoint**
endpoint是通信端点，具体socket接收和发送处理器，对传输层抽象，endpoint用来实现TCP/IP协议

- acceptor：监听socket连接请求。
- socketProcessor：用于处理接收到的socket请求，实现runnable，在run方法里调用协议处理组件process进行处理。

**processor**
processor用来实现HTTP协议，读取字节流解析成tomcatrequest和response对象，并通过adapter将其提交到容器处理，processor是对应应用层协议抽象。定义请求的处理。他的抽象类abstractprocessor对协议共有属性进行封装
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/4.png)

**adapter**
tomcat定义tomcatrequest类来存放请求，但是tomcatrequest不是标准servletrequest，tomcat引入coyoteadapter，传入tomcatrequest将其转换为servletrequest，在调用容器service方法


## 容器
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/5.png)

- engine：表示引擎，用来管理多个虚拟站点，一个service最多只能有一个engine
	- host：表示虚拟主机
		- context：表示web应用程序；
			- wrapper：表示一个servlet

tomcat采用一种**分层架构，组合模式**管理容器，使得servlet容器具有很好的灵活性。

**servlet定位**
tomcat使用mapper组件来处理servlet的定位，mapper组件保留了web应用配置信息，其实就是**容器组件与访问路径的映射关系**，host容器配置域名，context的web路径，wrapper容器servlet映射路径，根据请求路径分解并映射到不同路由最终到servlet

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/6.png)

1. 根据协议和端口号选定service和engine
2. 根据域名选定host
3. 根据URL路径找到context组件
4. context确定后，mapper根据web.xml配置的servlet映射路径找到wrapper，servlet。

并不是servlet处理所有请求内容，在这个调用链上每一个容器都可以使用pipeline-value进行处理

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/7.png)


## **组件的创建、初始化、启动、停止和销毁**

如果一个系统对外提供服务，那么tomcat需要创建，组装并启动这些组件；服务停止时，tomcat需要释放资源，销毁这些组件，这是一个动态过程

- 先创建子组件，在创建父组件，子组件需要被注入到父组件
- 先创建内层组件，在创建外层组件，内层组件被注入到外层组件中

如果按照**从小到大，从内到外顺序创建组件**，然后组装，这样会造成**代码逻辑混乱**和**组件遗漏**，而且也**不利于**后期功能**扩展**

**lifecycle**
组件的共通点，不变点是init，start，stop，destroy，抽象为lifecycle接口，每个组件去实现改接口。**父组件**方法需**创建子组件**并调用子组件**对应方法**，这样我们就可以**只调用最顶层组件**就可以实现**一键式启停**。

整个组件生命周期定义为不同状态，状态的改变看作一个事件，而事件是有监听器的，**监听器**可**方便添加和删除**，**利于扩展**。

LifeCycleBase基类实现了一些公共逻辑，生命状态转变与维护，事件触发，监听器添加，删除。而具体实现由子类去完成，这就是**模板设计模式**


## **tomcat启动**

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/8.png)

1. 启动脚本启动一个jvm，bootstrap初始化类加载器
2. 实例化Catalina，通过解析server.xml，创建相应组件，并调用server的start方法
3. server管理多个service，service负责连接器和容器的管理

- 当我们使用ctrl+c关闭tomcat时，tomcat调用Catalina在JVM注册一个关闭钩子 CatalinaShutdownHook(注册一线程，run方法调用Catalina.this.stop方法)进行调用
- server组件具体实现类是standardServer，继承了lifecycleBase，添加（通过固定数组存放service，动态复制扩展数组长度），启动，停止，管理service。同时server还启动一个socket（await方法中创建，死循环接收特定连接请求指令）来监听停止端口
- service组件具体实现类是 standardService 也继承了lifecycleBase，同时还有server connector，engine和mapper。其中还有一个MapperListener，用于监听容器变化，并把信息更新到mapper，支持tomcat支持热部署
- engine容器继承了containerbase基类，并实现了Engine接口，管理host容器数组，处理请求，把请求转发到host容器，host位于请求中（mapper组件通过请求URL定位相应容器，并把容器对象保存到请求对象中）


# jetty

jetty 由多个connector和多个handler，所有connector被handler共享。以及一个线程池组成
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/9.png)

## **connector**
connector对IO模型和协议进行封装，jetty9只支持NIO

- acceptor：用于接收请求，创建_acceptor数组用于存放acceptor。acceptor通过阻塞方式接受连接。连接成功后调用accepted函数，accepted函数将socketchannel设置
为阻塞模式，然后交于selector处理。
- selectormanager：selectormanager管理selector，selector的register用于处理channel注册到selector上，得到一个selectionKey。创建endpoint和connection，并跟selectionKey绑在一起，ManagedSelector并没有调用直接EndPoint的方法去处理数据，而是通过调用EndPoint的方法返回一个Runnable，然后把这个Runnable扔给线程池执行，所以你能猜到，这个Runnable才会去真正读数据和处理请求。
- connection：上述endpoint的runnable会调用connection回调方法来处理请求。connection类比tomcat的processor，解析协议。
	- 请求处理：endpoint调用connection回调方法，使用endpoint接口读数据，读完后让HTTP解析器解析字节流，解析后数据存放到request对象里
	- 响应处理：connection调用handler进行业务处理，handler通过response操作响应流，向流写入数据，httpconnection再通过endpoint写数据到channel。

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/10.png)

1. Acceptor监听连接请求，当有连接请求到达时就接受连接，一个连接对应一个Channel，Acceptor将Channel交给ManagedSelector来处理。

2. ManagedSelector把Channel注册到Selector上，并创建一个EndPoint和Connection跟这个Channel绑定，接着就不断地检测I/O事件。

3. I/O事件到了就调用EndPoint的方法拿到一个Runnable，并扔给线程池执行。

4. 线程池中调度某个线程执行Runnable。

5. Runnable执行时，调用回调函数，这个回调函数是Connection注册到EndPoint中的。

6. 回调函数内部实现，其实就是调用EndPoint的接口方法来读数据。

7. Connection解析读到的数据，生成请求对象并交给Handler组件去处理。


### handler
connector会将servlet请求交给handler处理

**handler类型**

- 协调handler，赋值将请求路由到一组handler中
- 过滤器handler，自己处理请求，然后转发下一个handler
- 内容handler，真正调用Servlet来处理请求，生成响应的内容

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/11.png)

AbstractHandlerContainer：为了实现链式调用，内部有其他handler引用
HandlerWrapper：只包含一个其他handler引用
	- 子类：
		- Server：Handler模块的入口，必然要将请求传递给其他Handler来处理
		- ScopedHandler：实现了“具有上下文信息”的责任链调用
HandlerCollection：包含多个其他handler引用，Jetty可能需要同时支持多个Web应用，如果每个Web应用有一个Handler入口，那么多个Web应用的Handler就成了一个数组

ContextHandler：创建并初始化Servlet规范里的ServletContext对象，同时ContextHandler还包含了一组能够让你的Web应用运行起来的Handler
ServletHandler：实现了Servlet规范中的Servlet、Filter和Listener的功能
SessionHandler：用来管理Session

除此还有一些其他通用组件handler，安全，解压缩，，jetty提供任意添加，裁剪这些模块，从而实现高度可定制化

tomcat和jetty整体结构都是基于组件，通过xml或代码方式灵活选择组件搭建web容器。这种组件化设计，有两点很重要，面向**接口编程和通过责任链模式**处理请求。因为高度灵活定制化，web容器启动前
通过**反射机制动态创建组件**。通过生命周期方式来管理组件，父组件负责子组件创建，启停，销毁；并通过监听组件状态转变**事件作为触发条件**


## **tomcat 优化**

- **清理不必要的web应用**,tomcat启动时，会启动webapps文件夹下的所有工程，所以清理不需要启动的工程很有必要
- **清理XML配置文件**，XML解析代价不算小，tomcat启动时会解析所有XML配置文件。所以保持配置文件简洁很重要。
- **清理JAR包**：JVM类加载器在加载类是，需要查找每一个依赖JAR包，WEB应用中lib目录下不应该出现servlet API或者Tomcat自身的JAR
- **清理其他文件**：及时清理日志，删除logs文件夹下不需要的日志文件。还有work文件夹下catalina文件夹（tomcat吧JSP转换为Class文件工作目录，有时修改了代码，重启tomcat没有生效。这时可以尝试删除文件夹，删除后，下次启动后自动生成）
- **禁止tomcat TLD扫描**：为了支持JSP，在应用启动时，tomcat会扫描JAR包里面的TLD文件，加载里面定义的标签库，所以tomcat会在启动日志中说明，扫描web应用下的JAR包，如果没有发现TLD文件，会提示建议配置tomcat不要扫描这些JAR包
    - **禁用TLD扫描**（不使用JSP作为Web页面模板）：打开Tomcat的conf/context.xml文件，在该文件下Context标签下，加上JarScanner和JarScanFilter子标签：
    ```java
    
    <Context>
    	<JarScanner>
    		<JarScanFilter defaultTldScan="false"/>
    	</JarScanner>
    </Context>
    ```
    - **指定扫描包含TLD文件的jar包**（使用JSP作为页面模板情况下）打开conf/catalina.properties文件，在配置项jarsToSkip中加上JAR包()
    ```java
    tomcat.util.scan.StandardScanFilter.jarsToSkip=xxx.jar
    ```
- **关闭WebSocket支持**：tomcat会扫描WebSocket注解的API实现，比如@ServerEndpoint注解的类。注解扫描一般是比较慢的。打开conf/context.xml，给context标签添加containerSciFilter的属性
```java
<Context containerSciFilter="org.apache.tomcat.websocket.server.WsSci">
</Context>
```
- **关闭JSP支持**：打开conf/context.xml，给context标签添加containerSciFilter的属性
```java
<Context containerSciFilter="org.apache.jasper.servlet.JasperInitializer">
</Context>
```
- **禁止Servlet注解扫描**：打开web.xml文件设置web-app属性
```java
<web-app metadata-complete="true">  
</web-app>
```
- **配置Web-Fragment扫描**：Servlet3.0引入了web模块部署描述符片段”的web-fragment.xml，这是一个部署描述文件，可以完成web.xml的配置功能。而这个web-fragment.xml文件必须存放在JAR文件的META-INF目录下，而JAR包通常放在WEB-INF/lib目录下，因此Tomcat需要对JAR文件进行扫描才能支持这个功能。打开web.xml文件设置<absolute-ordering>元素直接指定了哪些JAR包需要扫描web fragment
```java
<web-app>
    <absolute-ordering/> // 不需要扫描
</web-app>
```
- **随机数熵源优化**：tomcat7以上版本通过java的SecureRandom类生成随机数。而JVM默认使用阻塞式熵源(安全性较高一些)，可能会导致tomcat启动偏慢。通过设置JVM参数
```java
-Djava.security.egd=file:/dev/./urandom
```
或者设置$JAVA_HOME/jre/lib/java.security文件
```java
securerandom.source=file:/dev/./urandom
```
- **并行启动多个web应用**
tomcat启动默认一个一个启动web应用，通过修改server.xml文件中Host元素的startStopThreads属性设置用多少线程进行来完成并行启动。
```java
<Engine startStopThreads="0">
    <Host startStopThreads="0">
    </Host>
</Engine
```
engine元素里配置表示tomcat配置多少线程启动Host

## **tomcat非阻塞I/O实现**

tomcat的NioEndPoint组件实现了I/O多路复用模型，

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/12.png)

- LimitLatch：连接控制器，控制最大连接数，NIO模式下默认10000，
```java
public class LimitLatch {
    private class Sync extends AbstractQueuedSynchronizer {
     
        @Override
        protected int tryAcquireShared() {
            long newCount = count.incrementAndGet();
            if (newCount > limit) {
                count.decrementAndGet();
                return -1;
            } else {
                return 1;
            }
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
            count.decrementAndGet();
            return true;
        }
    }

    private final Sync sync;
    private final AtomicLong count;
    private volatile long limit;
    
    //线程调用这个方法来获得接收新连接的许可，线程可能被阻塞
    public void countUpOrAwait() throws InterruptedException {
      sync.acquireSharedInterruptibly(1);
    }

    //调用这个方法来释放一个连接许可，那么前面阻塞的线程可能被唤醒
    public long countDown() {
      sync.releaseShared(0);
      long result = getCount();
      return result;
   }
}
```
- Acceptor：在一个单独线程里，在一个死循环里调用accept方法接受新连接，一旦有新连接，accept方法返回一个channel对象，接着把channel对象交给poller去处理。ServerSocketChannel通过accept()接受新的连接，accept()方法返回获得SocketChannel对象，然后将SocketChannel对象封装在一个PollerEvent对象中，并将PollerEvent对象压入Poller的Queue里，这是个典型的生产者-消费者模式，Acceptor与Poller线程之间通过Queue通信。
```java
serverSock = ServerSocketChannel.open();
serverSock.socket().bind(addr,getAcceptCount());// 操作系统的等待队列长度,操作系统能继续接收的最大连接数就是这个队列长度，可以通过acceptCount参数配置，默认是100
serverSock.configureBlocking(true);// 以阻塞的方式接收连接
```
- Poller：在一个单独线程里。维护一个channel数组，在死循环不断检测channnel数据状态，一旦有Channel可读，就生成一个SocketProcessor任务对象扔给Executor去处理
```java
private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();// SynchronizedQueue的方法都使用synchronize关键字加锁，保证队列只被一个acceptor读写。多个Poller线程运行，每个都有自己queue，可被多个acceptor调用注册pollerEvent。poller不断通过内部selector想内核查询channel状态，一旦刻度就生成任务类socketprocessor交于executor处理。同时循环遍历检查管理socketchannel是否超时，超时则关闭
```
- socketProcessor：用来定义任务，主要调用http11processor组件处理请求。Http11Processor读取Channel的数据来生成ServletRequest对象。tomcat使用包装类socketwrapper屏蔽支持channel的非阻塞IO和异步IO模型的差异
- Executor就是线程池，负责运行SocketProcessor任务类，调用SocketProcessor的run方法会调用Http11Processor来读取和解析请求数据。我们知道，Http11Processor是应用层协议的封装，它会调用容器里servlet获得响应，再把响应通过Channel写出

## tomcat异步I/O实现
**NIO.2**
异步最大的特点：应用程序**不需要自己取触发**数据从内核空间到用户空间的**拷贝**。内核主动将数据拷贝到用户空间通知应用程序or等待应用程序通过**selector查询**，当数据**就绪**后，应用程序发起**read**调用，这时内核再把数据从内核拷贝到用户空间（这是应用程序是阻塞的），readAPI告诉内核：1数据准备好了拷贝到哪个buffer。2调用哪个函数去处理这些数据；内核在街道read指令后，等待数据到达后，产生硬件中断，内核在中断程序把数据从网卡拷贝到内核空间，然后做TCP/IP协议的数据解包和重组，再把数据拷贝到应用程序指定buffer并调用指定回调函数

java NIO.2实现
```java
public class Nio2Server {

   void listen(){
      //1.创建一个线程池,用来执行内核回调请求
      ExecutorService es = Executors.newCachedThreadPool();

      //2.创建异步通道群组，绑定线程池
      AsynchronousChannelGroup tg = AsynchronousChannelGroup.withCachedThreadPool(es, 1);
      
      //3.创建服务端异步通道，绑定到异步通道群组
      AsynchronousServerSocketChannel assc = AsynchronousServerSocketChannel.open(tg);

      //4.绑定监听端口
      assc.bind(new InetSocketAddress(8080));

      //5. 监听连接，传入回调类处理连接请求
      assc.accept(this, new AcceptHandler()); 
   }
}

//AcceptHandler类实现了CompletionHandler接口的completed方法。它还有两个模板参数，第一个是异步通道，第二个就是Nio2Server本身
public class AcceptHandler implements CompletionHandler<AsynchronousSocketChannel, Nio2Server> {

   //具体处理连接请求的就是completed方法，它有两个参数：第一个是异步通道，第二个就是上面传入的NioServer对象
   @Override
   public void completed(AsynchronousSocketChannel asc, Nio2Server attachment) {      
      //调用accept方法继续接收其他客户端的请求
      attachment.assc.accept(attachment, this);
      
      //1. 先分配好Buffer，告诉内核，数据拷贝到哪里去
      ByteBuffer buf = ByteBuffer.allocate(1024);
      
      //2. 调用read函数读取数据，除了把buf作为参数传入，还传入读回调类
      channel.read(buf, buf, new ReadHandler(asc)); 

}

```

**NioEndpoint2**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/13.png)

NioEndpoint2总体工作流程大致和NioEndpoint是相似的。Nio2Acceptor扩展了Acceptor，用异步IO方式接受连接，跑在一个单独线程，NioEndpoint2没有Poller组件，也就是没有Selector。而Selector的工作交给内核来做了

**Nio2Acceptor**
Nio2Acceptor监听连接过程不是不断调accept方法，而是通过回调函数完成

监听连接过程
```java
serverSock.accept(null, this);
```

Nio2Acceptor实现了CompletionHandler接口
```java
protected class Nio2Acceptor extends Acceptor<AsynchronousSocketChannel>
    implements CompletionHandler<AsynchronousSocketChannel, Void> {
    
@Override
public void completed(AsynchronousSocketChannel socket,
        Void attachment) {
        
    if (isRunning() && !isPaused()) {
        if (getMaxConnections() == -1) {
            //如果没有连接限制，继续接收新的连接
            serverSock.accept(null, this);
        } else {
            //如果有连接限制，就在线程池里跑Run方法，Run方法会检查连接数
            getExecutor().execute(this);
        }
        //处理请求
        if (!setSocketOptions(socket)) {
            closeSocket(socket);
        }
    } 
}
```

- 如果没有连接限制，继续在本线程中调用accept方法接收新的连接。
- 如果有连接限制，就在线程池里跑run方法去接收新的连接。那为什么要跑run方法呢，因为在run方法里会检查连接数，当连接达到最大数时，线程可能会被LimitLatch阻塞。为什么要放在线程池里跑呢？这是因为如果放在当前线程里执行，completed方法可能被阻塞，会导致这个回调方法一直不返回。

completed方法会调用setSocketOptions方法，在这个方法里，会创建Nio2SocketWrapper和SocketProcessor，并交给线程池处理

**Nio2SocketWrapper**
主要作用是封装Channel，并提供接口给Http11Processor读写数据。Http11processor是不能阻塞等待数据的，processor调用wrapper的read方法时需要注册回调类。read立即返回，如果立即返回后processor没有读到数据那么这个请求不就失败了吗

http11processor通过2次调用完成读取操作

- 第一次read调用：连接刚刚建立好后，Acceptor创建SocketProcessor任务类交给线程池去处理，Http11Processor在处理请求的过程中，会调用Nio2SocketWrapper的read方法发出第一次读请求，同时注册了回调类readCompletionHandler，因为数据没读到，Http11Processor把当前的Nio2SocketWrapper标记为数据不完整。接着SocketProcessor线程被回收，Http11Processor并没有阻塞等待数据。这里请注意，Http11Processor维护了一个Nio2SocketWrapper列表，也就是维护了连接的状态。
- 第二次read调用：当数据到达后，内核已经把数据拷贝到Http11Processor指定的Buffer里，同时回调类readCompletionHandler被调用，在这个回调处理方法里会重新创建一个新的SocketProcessor任务来继续处理这个连接，而这个新的SocketProcessor任务类持有原来那个Nio2SocketWrapper，这一次Http11Processor可以通过Nio2SocketWrapper读取数据了，因为数据已经到了应用层的Buffer。

**配置nio2**
```java
server.xml中：
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
maxThreads="150" SSLEnabled="true">

</Connector>
```

## AprEndpoint
APR是Apache可移植运行时库，目的是向上层应用程序提供一个跨平台操作系统接口库。AprEndpoint通过JNI调用APR本地库实现非阻塞I/O

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/14.png)

和NioEndpoint相比，acceptor和poller的实现是不同的

**Acceptor**
java使用JNI调用C语言的API实现acceptor。

**Poller**
acceptor接受到新的socket连接后，把socket交给poller查询I/O事件（通过JNI调用APR中的poll方法，APR又调用操作系统的epoll API实现）。AprEndpoint中，可以配置一个deferAccept参数，对应TCP协议中的TCP_DEFER_ACCEPT，当TCP客户端有新的连接请求到达时，TCP服务端不建立连接，而是等到请求数据发过来是再建立连接。（不用selector反复查询是否就绪）

**JVM堆/本地内存**
每个进程都有自己虚拟空间，JVM内存是进程空间一部分（此外还有代码段，数据段，内存映射区，内核空间等），从JVM角度看，JVM内存之外部分叫做本地内存

java NIO提供两种buffer来接收数据，HeapByteBuffer和DirectByteBuffer

使用HeapByteBuffer接受网络数据时，需要吧数据从内核拷贝到一个临时本地内存，再从临时本地内存拷贝到JVM堆（GC会造成堆地址失效），而**本地内存到JVM堆拷贝过程中JVM可以保证不做GC**（Hotspot VM层面，这时是没有safepoint的）。而DirectByteBuffer对象本身在JVM堆上，但是字节数组直接从本地内存分配少一次拷贝。从稳定性考虑，NioEndpoint和Nio2Endpoint不直接使用本地内存，因为本地内存不好管理，若内存泄露难以定位。

**sendfile**
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/15.png)

传统的静态文件处理流程经历了6次内存拷贝，且read和write等系统调用导致进程从用户态到内核态切换，会耗费大量CPU和内存资源。

而AprEndpoint通过操作系统层面sendfile解决问题。

1. 将文件内容读取到内核缓冲区
2. 将记录数据位置和长度的描述符添加到Socket缓冲区；而数据直接从内核缓冲区传递给网卡。

服务访问高峰期，重启或发布服务，没有启动完成时，不要把流量送过来，避免出现阻塞线程过多情况。

## 线程池

java线程池核心类ThreadPoolExecutor，在每次**提交任务**时，如果线程数还**没达到核心线程数corePoolSize**，就**创建**新的线程,**否则**将其放道工作队列**workQueue**；而corePool中线程通过poll方法获取queue中任务，若**workQueue**有界，会创建临时线程直到数量**maximumPoolSize**，转而执行拒绝策略**handler**。若corePool中线程通过poll方法获取不到queue中任务，则该线程会被销毁。

 - FixedThreadPool：固定长度线程数组，而queue是无界队列
 - CachedThreadPool：无限创建临时时队列（Integer.Max_VALUE）
 
tomcat定制线程池和原生java线程池唯一区别是在达到**maximumPoolSize**时，**继续尝试把任务添加到任务队列中**（其实现是通过捕获原生线程池抛出RejectedExecutionException异常，继续尝试放入任务队列）,如下：

```java
  public void execute(Runnable command, long timeout, TimeUnit unit) {
  // Tomcat需要维护已提交任务数这个变量，它的目的就是在任务队列的长度无限制的情况下，让线程池有机会创建新的线程
      submittedCount.incrementAndGet();
      try {
          //调用Java原生线程池的execute去执行任务
          super.execute(command);
      } catch (RejectedExecutionException rx) {
         //如果总线程数达到maximumPoolSize，Java原生线程池执行拒绝策略
          if (super.getQueue() instanceof TaskQueue) {
              final TaskQueue queue = (TaskQueue)super.getQueue();
              try {
                  //继续尝试把任务放到任务队列中去
                  if (!queue.force(command, timeout, unit)) {
                      submittedCount.decrementAndGet();
                      //如果缓冲队列也满了，插入失败，执行拒绝策略。
                      throw new RejectedExecutionException("...");
                  }
              } 
          }
      }
}
```

上述代码中，第一行解决任务队列长度无限制情况
```java
public class TaskQueue extends LinkedBlockingQueue<Runnable> {

   @Override
  //线程池调用任务队列的方法时，当前线程数肯定已经大于核心线程数了
  public boolean offer(Runnable o) {

      //如果线程数已经到了最大值，不能创建新线程了，只能把任务添加到任务队列。
      if (parent.getPoolSize() == parent.getMaximumPoolSize()) 
          return super.offer(o);
          
      //执行到这里，表明当前线程数大于核心线程数，并且小于最大线程数。
      //表明是可以创建新线程的，那到底要不要创建呢？分两种情况：
      
      //1. 如果已提交的任务数小于当前线程数，表示还有空闲线程，无需创建新线程
      if (parent.getSubmittedCount()<=(parent.getPoolSize())) 
          return super.offer(o);
          
      //2. 如果已提交的任务数大于当前线程数，线程不够用了，返回false去创建新线程
      if (parent.getPoolSize()<parent.getMaximumPoolSize()) 
          return false;
          
      //默认情况下总是把任务添加到任务队列
      return super.offer(o);
  }
  
}
```

可以看出Tomcat维护已提交任务数，在任务队列的长度无限制的情况下，让线程池有机会创建新的线程

## websocket

对于实现实时性高的需求，有Ajax和comet技术，前者本质还是轮询，后者是在HTTP长连接基础上做了一些hack，实时性不高，而且频繁请求会给服务器带来压力，造成网络流量和带宽的浪费。Html5推出了WebSocket技术，使得浏览器和服务器之间任何一方都可以主动发消息给对方。

websocket和socket不一样，socket是对TCP/IP协议抽象出来的API，对应一个IP地址和端口号。而webSocket是一个**应用层协议**，为了兼容HTTP，和HTTP进行**一次握手**，握手之后数据通过TCP的**socket继续传输**，与HTTP无关了。数据以**frame形式传输**，将一条消息分为几个frame，先后传输。这样的好处：

- 大数据传输可以分片传输，不用考虑数据大小问题
- 同HTTP的chunk一样，边生成数据边传输，提高传输效率

websocket应用程序由一些列websocket endpoint组成，Endpoint是一个java对象，代码websocket连接的一端，好像处理HTTP请求的servlet一样。当浏览器连接到一个Endpoint时，tomcat会给这个链接创建一个唯一session，session的websocket在连接关闭时销毁，Session是socket的封装，endpoint通过它与浏览器通信。

**Endpoint加载和WebSocket请求处理**

WebSocket加载通过SCI机制完成，SCI是Servlet 3.0规范中定义的**用来接收Web应用启动事件的接口**。通过监听servlet容器启动事件，做一些扫描和加载websocket需要的endpoint类的初始化工作。

**SCI使用**
实现ServletContainerInitializer接口，并增加HandlesTypes注解。

**加载**
tomcat启动时通过SCI扫描指定类，作为SCI的onStartup方法参数，调用SCI的onStartup方法。构造一个websocketcontainer实例（专门处理websocket请求的endpoint容器），并且建立URL到ServerEndpoint的映射关系。

**请求处理**
当第一个websocket请求到达时，tomcat将HTTP协议升级成WebSocket协议，并将该Socket连接processor替换成UpgradeProcess。socket不会立即关闭，接下来请求，tomcat通过upgradeProcessor直接调用相应serverPoint处理

## jetty线程策略

Jetty的connector支持NIO模型，Jetty在Java原生Selector基础上封装自己的selector，ManagedSelector。将**I/O时间侦测和处理用同一个线程处理。充分利用CPU缓存并减少线程上下文切换开销**。

将**I/O检测**和**业务处理**两种工作分开是传统selector的做法，但是这样存在缺点。当selector检测读就绪事件时，数据已经拷贝到内核中缓存，CPU缓存中也有这些数据了，如果应用程序此时去读取这些数据是另一个线程去读，很有可能使用另一个CPU核，这样之前那个CPU缓存中数据就用不上了。并且线程切换也需要开销。

**ManagedSelector**
ManagedSelector本身就是一个selector，负责I/O**时间检测和分发**。

```java
public class ManagedSelector extends ContainerLifeCycle implements Dumpable
{
    //原子变量，表明当前的ManagedSelector是否已经启动
    private final AtomicBoolean _started = new AtomicBoolean(false);
    
    //表明是否阻塞在select调用上
    private boolean _selecting = false;
    
    //管理器的引用，SelectorManager管理若干ManagedSelector的生命周期
    private final SelectorManager _selectorManager;
    
    //ManagedSelector不止一个，为它们每人分配一个id
    private final int _id;
    
    //关键的执行策略，生产者和消费者是否在同一个线程处理由它决定
    private final ExecutionStrategy _strategy;
    
    //Java原生的Selector
    private Selector _selector;
    
    //"Selector更新任务"队列
    private Deque<SelectorUpdate> _updates = new ArrayDeque<>();
    private Deque<SelectorUpdate> _updateable = new ArrayDeque<>();
    
    ...
}
```
**SelectorUpdate**
对selector的操作无非是将**channel注册到selector或告诉selector感兴趣的I/O事件**。都是对**selector状态的更新**。jetty把这些操作抽象为SelectorUpdate

```java
/**
 * A selector update to be done when the selector has been woken.
 */
public interface SelectorUpdate
{
    void update(Selector selector);
}
```
意味你需向managedSelector提交一个任务类，任务类实现SelectorUpdate接口，update方法定义对managedselector做的操作。managedSelector在一个死循环里拉取每个任务执行

**Selectable**
I/O事件到达时，managedSelector通过实现接口Selectable任务类进行回调处理。
```java
public interface Selectable
{
    //当某一个Channel的I/O事件就绪后，ManagedSelector会调用的回调函数
    Runnable onSelected();

    //当所有事件处理完了之后ManagedSelector会调的回调函数，我们先忽略。
    void updateKey();
}
```
managedSelector检查某个channel的I/O事件就绪时，就调用该channel绑定附件类的onSelected方法拿到一个runnable，然后扔到线程池取执行。

endpoint的onselced实现
```java
public Runnable onSelected()
{
    int readyOps = _key.readyOps();

// 读事件，写事件
    boolean fillable = (readyOps & SelectionKey.OP_READ) != 0;
    boolean flushable = (readyOps & SelectionKey.OP_WRITE) != 0;

    // return task to complete the job
    Runnable task= fillable 
            ? (flushable 
                    ? _runCompleteWriteFillable 
                    : _runFillable)
            : (flushable 
                    ? _runCompleteWrite 
                    : null);

    return task;
}
```

**ExecutionStrategy**
ManagedSelector将I/O事件的生产和消费看作是生产者消费者模式。jetty定义ExecutionStrategy接口
```java
public interface ExecutionStrategy
{
    //只在HTTP2中用到，简单起见，我们先忽略这个方法。
    public void dispatch();

    //实现具体执行策略，任务生产出来后可能由当前线程执行，也可能由新线程来执行
    public void produce();
    
    //任务的生产委托给Producer内部接口，
    public interface Producer
    {
        //生产一个Runnable(任务)
        Runnable produce();
    }
}
```

ExecutionStrategy具体实现类

- ProduceConsume：任务生产者自己依次生产和执行任务，就是用一个线程来侦测和处理一个ManagedSelector上所有的I/O事件，后面的I/O事件要等待前面的I/O事件处理完。图中绿色表示生产，蓝色表示消费。
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/16.png)
- ProduceExecuteConsume：任务生产者开启新线程来运行任务I/O事件侦测和处理用不同的线程来处理，不能利用CPU缓存，线程切换成本高。棕色表示线程切换
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/17.png)
-  ExecuteProduceConsume：任务生产者自己运行任务，该策略可能会新建一个新线程以继续生产和执行任务。这种策略被称为“吃掉你杀的猎物”（狩猎伦理，认为一个人不应该杀死他不吃掉的东西）。它的优点是能利用CPU缓存，但是如果处理I/O事件的业务代码执行时间过长，会导致**线程大量阻塞和饥饿**。
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/18.png)
- EatWhatYouKill:对ExecuteProduceConsume改良，线程池线程充足时等同ExecuteProduceConsume；不足时，使用ProduceExecuteConsume策略（上面说过ExecuteProduceConsume导致线程大量阻塞，最坏连I/O事件侦测都没线程可用，导致拒绝连接请求）。

**jetty的ExecutionStrategy实现**
SelectorProducer是ManagedSelector内部类

```java
private class SelectorProducer implements ExecutionStrategy.Producer
{
    private Set<SelectionKey> _keys = Collections.emptySet();
    private Iterator<SelectionKey> _cursor = Collections.emptyIterator();

    @Override
    public Runnable produce()
    {
        while (true)
        {
            //如果Channel集合中有I/O事件就绪，调用前面提到的Selectable接口获取Runnable,直接返回给ExecutionStrategy去处理
            Runnable task = processSelected();
            if (task != null)
                return task;
            
           //如果没有I/O事件就绪，就干点杂活，看看有没有客户提交了更新Selector的任务，就是上面提到的SelectorUpdate任务类。
            processUpdates();
            updateKeys();

           //继续执行select方法，侦测I/O就绪事件
            if (!select())
                return null;
        }
    }
 }
```

## **对象池**
一个比较大，比较复杂的java对象，创建，初始化，GC需要耗费CPU和内存资源。Jetty和Tomcat都是用了对象池技术

**Tomcat--SynchronizedStack**
```java
public class SynchronizedStack<T> {

    //内部维护一个对象数组,用数组实现栈的功能,（不使用链表来维护，减少节点维护内存开销，只支持扩容，不支持缩容，也就不会被GC，最低代价实现无界容器）
    private Object[] stack;

    //这个方法用来归还对象，用synchronized进行线程同步
    public synchronized boolean push(T obj) {
        index++;
        if (index == size) {
            if (limit == -1 || size < limit) {
                expand();//对象不够用了，扩展对象数组
            } else {
                index--;
                return false;
            }
        }
        stack[index] = obj;
        return true;
    }
    
    //这个方法用来获取对象
    public synchronized T pop() {
        if (index == -1) {
            return null;
        }
        T result = (T) stack[index];
        stack[index--] = null;
        return result;
    }
    
    //扩展对象数组长度，以2倍大小扩展
    private void expand() {
      int newSize = size * 2;
      if (limit != -1 && newSize > limit) {
          newSize = limit;
      }
      //扩展策略是创建一个数组长度为原来两倍的新数组
      Object[] newStack = new Object[newSize];
      //将老数组对象引用复制到新数组
      System.arraycopy(stack, 0, newStack, 0, size);
      //将stack指向新数组，老数组可以被GC掉了
      stack = newStack;
      size = newSize;
   }
}
```

**Jetty---ByteBufferPool**
本质是一个ByteBuffer对象池，jetty进行网络数据读写时直接获取预先分配的buffer
```java
public interface ByteBufferPool
{
// 指定buffer从本地内存还是JVM堆分配
    public ByteBuffer acquire(int size, boolean direct);

    public void release(ByteBuffer buffer);
}
```
ArrayByteBufferPool
```java
public class ArrayByteBufferPool implements ByteBufferPool
{
    private final int _min;//最小size的Buffer长度
    private final int _maxQueue;//Queue最大长度
    
    //用不同的Bucket(桶)来持有不同size的ByteBuffer对象,同一个桶中的ByteBuffer size是一样的
    private final ByteBufferPool.Bucket[] _direct;
    private final ByteBufferPool.Bucket[] _indirect;
    
    //ByteBuffer的size增量
    private final int _inc;
    
    public ArrayByteBufferPool(int minSize, int increment, int maxSize, int maxQueue)
    {
        //检查参数值并设置默认值
        if (minSize<=0)//ByteBuffer的最小长度
            minSize=0;
        if (increment<=0)
            increment=1024;//默认以1024递增
        if (maxSize<=0)
            maxSize=64*1024;//ByteBuffer的最大长度默认是64K
        
        //ByteBuffer的最小长度必须小于增量
        if (minSize>=increment) 
            throw new IllegalArgumentException("minSize >= increment");
            
        //最大长度必须是增量的整数倍
        if ((maxSize%increment)!=0 || increment>=maxSize)
            throw new IllegalArgumentException("increment must be a divisor of maxSize");
         
        _min=minSize;
        _inc=increment;
        
        //创建maxSize/increment个桶,包含直接内存的与heap的
        _direct=new ByteBufferPool.Bucket[maxSize/increment];
        _indirect=new ByteBufferPool.Bucket[maxSize/increment];
        _maxQueue=maxQueue;
        int size=0;
        for (int i=0;i<_direct.length;i++)
        {
          size+=_inc;
          _direct[i]=new ByteBufferPool.Bucket(this,size,_maxQueue);
          _indirect[i]=new ByteBufferPool.Bucket(this,size,_maxQueue);
        }
    }
}

//分配Buffer
public ByteBuffer acquire(int size, boolean direct)
{
    //找到对应的桶，没有的话创建一个桶
    ByteBufferPool.Bucket bucket = bucketFor(size,direct);
    if (bucket==null)
        return newByteBuffer(size,direct);
    //这里其实调用了Deque的poll方法
    return bucket.acquire(direct);
        
}

//释放Buffer
public void release(ByteBuffer buffer)
{
    if (buffer!=null)
    {
      //找到对应的桶
      ByteBufferPool.Bucket bucket = bucketFor(buffer.capacity(),buffer.isDirect());
      
      //这里调用了Deque的offerFirst方法
  if (bucket!=null)
      bucket.release(buffer);
    }
}
```
buffer的获取和释放通过对相应的桶中deque进行操作

对象池的存在会造成**线程同步**的问题，但是却避免了**创建，销毁对象**的开销。从对象池本身设计上来看，需尽量做到**无锁化**。如果内存够大，考虑使用ThreadLocal（注意内存泄漏问题）；必须对**对象池大小做限制（自动扩容，缩容）**. 
CPU在执行系统调用时，会从用户态切换到内核态，在用户态下，使用的是用户空间的内存；内核态下，使用的是内核空间。只有**内核可访问各种硬件资源（磁盘，网卡）**，即系统调用。

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/19.png)

共享库和mmap映射区是Linux将文件内容映射到这个内存区域，用户通过读写这段内存，从而对文件进行读写，无需通过read/write系统调用，省去了内核和用户空间的数据拷贝。（不支持socket读写，只支持磁盘文件）

taskstruct结构体本身分配在内核空间，保存进程相关信息（进程号，打开文件，创建socket以及CPU运行上下文）它的vm_struct成员变量保存内存区域起始和终止地址。

线程有自己的taskstruct结构体和运行栈区，其他资源跟父进程共用（虚拟地址空间，打开文件，socket）。

**read系统调用**
linux内核将线程当作一个进程进行CPU调度，内核维护一个可允许进程队列，所有处于task_running状态进程放入运行队列，本质使用双向链表将taskstruct链接，排队选择使用CPU时间片。
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/20.png)

- **调度：**从可运行列表选择一个进程，再从CPU列表选择一个可用CPU，将进程上下文恢复到CPU寄存器，执行进程下一条指令。
- **阻塞：**将进程taskstruct移除可运行队列，添加到等待队列，将进程状态更改，一次调度让出CPU
- **唤醒：**线程加入等待队列时向内核注册一个回调函数，当socket上数据到达后，产生硬件终端，内核调用回调函数唤醒进程。将taskstruct从等待队列移动运行队列，修改状态，等待机会获取CPU时间片。（期间内核将数据从内核空间拷贝到用户空间）

## **热部署和热加载**

- 热部署：由后台线程定式检查Web应用变化，重新加载整个Web应用。会清空session，比热加载更干净，彻底。
- 热加载：Web容器启动一个后台线程，定期检查类文件变化，如果有变化，就重新加载类，不会清空session。

**Tomcat后台线程**
```java
bgFuture = exec.scheduleWithFixedDelay(
              new ContainerBackgroundProcessor(),//要执行的Runnable,需要周期性执行的任务类，也是containerBase内部类，containerbase是所有容器组件基类，
              backgroundProcessorDelay, //第一次执行延迟多久
              backgroundProcessorDelay, //之后每次执行间隔多久
              TimeUnit.SECONDS);        //时间单位
```
**ContainerBackgroundProcessor实现**
```java
protected class ContainerBackgroundProcessor implements Runnable {

    @Override
    public void run() {
        //请注意这里传入的参数是"宿主类"的实例
        processChildren(ContainerBase.this);
    }

    protected void processChildren(Container container) {
        try {
            //1. 调用当前容器的backgroundProcess方法。
            container.backgroundProcess();
            
            //2. 遍历所有的子容器，递归调用processChildren，
            //这样当前容器的子孙都会被处理            
            Container[] children = container.findChildren();
            for (int i = 0; i < children.length; i++) {
            //这里请你注意，容器基类有个变量叫做backgroundProcessorDelay，如果大于0，表明子容器有自己的后台线程，无需父容器来调用它的processChildren方法。
                if (children[i].getBackgroundProcessorDelay() <= 0) {
                    processChildren(children[i]);
                }
            }
        } catch (Throwable t) { ... }
```

processChildren方法调用当前容器backprocess方法以及递归调用子孙backgroundprocess。这样就只需一个后台线程就能处理该容器下所有子容器，和tong

**backgroundprocess方法实现**
```java
public void backgroundProcess() {

    //1.执行容器中Cluster组件的周期性任务
    Cluster cluster = getClusterInternal();
    if (cluster != null) {
        cluster.backgroundProcess();
    }
    
    //2.执行容器中Realm组件的周期性任务
    Realm realm = getRealmInternal();
    if (realm != null) {
        realm.backgroundProcess();
   }
   
   //3.执行容器中Valve组件的周期性任务
    Valve current = pipeline.getFirst();
    while (current != null) {
       current.backgroundProcess();
       current = current.getNext();
    }
    
    //4. 监听 触发容器的"周期事件"，Host容器的监听器HostConfig就靠它来调用
    fireLifecycleEvent(Lifecycle.PERIODIC_EVENT, null);
}
```

### **Tomcat热加载**
tomcat热加载在context容器中实现
```java
public void backgroundProcess() {

    //WebappLoader周期性的检查WEB-INF/classes和WEB-INF/lib目录下的类文件
    Loader loader = getLoader();
    if (loader != null) {
        loader.backgroundProcess();        
    }
    
    //Session管理器周期性的检查是否有过期的Session
    Manager manager = getManager();
    if (manager != null) {
        manager.backgroundProcess();
    }
    
    //周期性的检查静态资源是否有变化
    WebResourceRoot resources = getResources();
    if (resources != null) {
        resources.backgroundProcess();
    }
    
    //调用父类ContainerBase的backgroundProcess方法
    super.backgroundProcess();
}

```
context容器的WebappLoader调用context容器的reload方法。reload方法主要做了以下的事

- 停止和销毁context容器及其所有子容器（wrapper），servlet实例
- 停止和销毁context容器相关Listener和filter
- 停止和销毁context容器pipeline和value
- 停止和销毁context容器的类加载器，以及其加载的类文件资源
- 启动context容器，重新创建前面被销毁的资源

context关联session没有被销毁

**启用tomcat热加载**
```java
Context.xml文件下

<Context reloadable="true"/>
```

### **Tomcat热部署**
热部署会重新部署web应用，原来context对象销毁（包括session），于是HOST就成为了部署发起者

Host监听周期事件
```java
public void lifecycleEvent(LifecycleEvent event) {
    // 执行check方法。
    if (event.getType().equals(Lifecycle.PERIODIC_EVENT)) {
        check();
    } 
}

protected void check() {
    if (host.getAutoDeploy()) {
        // 检查这个Host下所有已经部署的Web应用
        DeployedApplication[] apps =
            deployed.values().toArray(new DeployedApplication[0]);
            
        for (int i = 0; i < apps.length; i++) {
            //检查Web应用目录是否有变化
            checkResources(apps[i], false);
        }

        //执行部署
        deployApps();
    }
}
```

- web应用目录是否被删除，把相应context容器销毁
- 是否有新的web应用目录加入，部署相应web应用

HostConfig做的事比较宏观，检查web应用目录级别变化

**tomcat类加载**

tomcat类加载打破了jvm类加载的双亲委派原则

**findClass方法实现流程**

- 先在web应用本地目录下查找要加载的类
- 如果没找到，交给父加载器查找，即AppClassLoader
- 如果父加载器也没找到，抛出ClassNotFound


**loadClass方法实现流程**

- 先本地cache查找该类是否加载，也就是tomcat的类加载器是否已经加载
- 如果tomcat类加载器没有加载，再看看系统类加载器是否加载过
- 若无，让ExtClassLoader加载，**目的防止Web应用自己的类覆盖JRE的核心类**
- 若无，就在本地Web应用目录下查找加载
- 若无，说明不是web应用定义的类，由系统类加载器去加载。
- 若无，throw exception

## web应用隔离

**WebAppClassLoader**
我们部署两个web应用，两个web应用都有相同的servlet，tomcat自定义一个类加载器WebAppClassLoader，并给每个Web应用创建一个类加载器实例。context负责创建，维护实例，**不同加载器加载的类认为是不同的类**。
**SharedClassLoader**
两个web应用之间共享类库，通过增加一个SharedClassLoader来加载，作为WebAppClassLoader父加载器，如果webAppClassLoader没有加载到某个类，就委托父加载器去加载这个类，SharedClassLoader会在指定目录下加载共享类，在返回给webAppClassLoader。
**CatalinaClassloader**
CatalinaClassloader专门负责加载Tomcat自身的类，和SharedClassLoader是同一个级
**CommonClassLoader**
CommonClassLoader作为CatalinaClassloader和SharedClassLoader的父加载器，

CommonClassLoader加载的类可被CatalinaClassloader和SharedClassLoader使用，而CatalinaClassloader和SharedClassLoader加载的类互相隔离，WebAppClassLoader可使用SharedClassLoader加载的类，但各个WebAppClassLoader实例之间相互隔离

**默认加载路径**
CommonClassLoader对应<Tomcat>/common/*
CatalinaClassLoader对应 <Tomcat >/server/*
SharedClassLoader对应 <Tomcat >/shared/*
WebAppClassloader对应 <Tomcat >/webapps/<app>/WEB-INF/*目录

JVM有条隐含的默认规则，如果一个类由类加载器A加载，那这个类依赖类也由相同类加载器加载。spring作为第三方JAR包，本身由SharedClassLoader来加载，spring又要加载业务类，但是业务类在web应用目录下，为了解决这个问题，tomcat使用**线程上下文加载器**。他保存在线程私有数据里，当启动web应用时，线程里设置线程上下文加载器，这样spring在启动时就将线程上下文加载器取出来，用来加载bean。同时JDBC也使用上下文类加载器加载不同数据库驱动

获取上下文加载器
```java
cl = Thread.currentThread().getContextClassLoader();
```

- 每个web应用自己java类文件和依赖JAR包，分别放在WEB-INF/classes和WEB-INF/lib目录下
- 多个应用共享Java类文件和JAR包，分别放在Web容器指定共享目录下
- 常见classNotFound错误，应检查你的类加载器是否正确

StandardContext启动方法中，将当前线程上下文加载器设置为WebAppClassLoader

```java
originalClassLoader = Thread.currentThread().getContextClassLoader();
Thread.currentThread().setContextClassLoader(webApplicationClassLoader);

// 启动方法结束后，恢复线程上下文加载器
Thread.currentThread().setContextClassLoader(originalClassLoader);
```

## servlet管理

wrapper容器用于管理servlet
```java
protected volatile Servlet instance = null;

// 实例化servlet，
public synchronized Servlet loadServlet() throws ServletException {
    Servlet servlet;
  
    //1. 创建一个Servlet实例
    servlet = (Servlet) instanceManager.newInstance(servletClass);    
    
    //2.调用了Servlet的init方法，这是Servlet规范要求的
    initServlet(servlet);
    
    return servlet;
}
```

默认为延迟加载servlet，通过loadOnStartup=true设置非延迟加载；但是wrapper是会创建的。每个容器组价都有自己pipeline，每个pipeline有一个Value链，每个容器组件有一个BasicValue。Wrapper也有Pipline和BasicValve（StandardWrapperValue）

**standardWrapperValue的invoke过程**
- 第一步，创建Servlet实例
- 第二步，为当前请求创建Filter链
- 第三步，调用这个Filter链

Filter和servlet都可在web.xml文件里进行配置，filter作用域是整个web应用，context容器对他进行管理，使用Map集合保存filter，每个请求生成一个filter链，请求处理完了，filter链也被回收了。

Listener用于监听容器内部发生事件，主要两类事件
- 生命状态的变化，Context容器启动和停止，Session创建和销毁
- 属性的变化，比如Context容器某个属性值变了，Session某个属性值变了以及新的请求来了

```java
// 监听属性值变化的监听器,属性值可变所以用copyonwritearray

private List<Object> applicationEventListenersList = new CopyOnWriteArrayList<>();

//监听生命事件的监听器，不能动态改变，没有现存安全问题
private Object applicationLifecycleListenersObjects[] = new Object[0];
```

## 异步servlet

新的请求会导致tomcat和jetty从线程池拿出一个线程处理请求，如果web应用需要较长时间处理请求（数据库查询/下游服务调用返回），tomcat线程一直不回收，占用系统资源，极端会导致线程饥饿。异步servlet会启动一个单独线程执行耗时请求，tomcat线程立即返回

**使用方法**
```java
@WebServlet(urlPatterns = {"/async"}, asyncSupported = true)
public class AsyncServlet extends HttpServlet {

    // Web应用线程池，用来处理异步Servlet
    ExecutorService executor = Executors.newSingleThreadExecutor();

    public void service(HttpServletRequest req, HttpServletResponse resp) {
        // 1. 调用startAsync获取异步上下文，上下文保存了请求和响应对象
        final AsyncContext ctx = req.startAsync();

       // 用线程池来执行耗时操作
        executor.execute(new Runnable() {

            @Override
            public void run() {

                //在这里做耗时的操作
                try {
                    ctx.getResponse().getWriter().println("Handling Async Servlet");
                } catch (IOException e) {}

                //3. 异步Servlet处理完了调用异步上下文的complete方法告诉tomcat，请求处理完成
                ctx.complete();
            }

        });
    }
}
```
异步servlet默认30秒，如果30秒请求没处理完，Tomcat触发超时规则，向浏览器返回超时错误，此时web应用若在调用ctx.complete方法，会得到illegalstateException异常

**startAsync**
startAsync创建异步上下文代替tomcat线程释放时保存request和response对象；并且告诉tomcat当处理方法返回时，不要把响应发到浏览器。tomcat使用CoyoteAdaptor进行flush响应数据和销毁request和response。CoyoteAdapter判断请求如果是异步请求，把当前socket协议处理者processor缓存起来，将SocketWrapper对象和相应的Processor存到一个同步Map数据结构

**complete**当请求处理完成时，调用request对象的action方法，action方法实现调用processor的processsocketevent方法，processsocketevent方法调用socketwrapper的processsocket方法，processSocket方法创建任务类，通过tomcat线程池处理

![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/21.png)

如果你发现tomcat线程不够了，大量线程阻塞在等待web应用处理上，而web应用又没有优化空间，确实需要长时间处理，可以尝试使用异步servlet

## springboot对容器的使用

为了支持多种容器，springboot对容器进行了抽象，定义了WebServer接口
```java
public interface WebServer {
    void start() throws WebServerException;
    void stop() throws WebServerException;
    int getPort();
}
```

定义ServletWebServerFactory创建Web容器，返回webServer
```java
public interface ServletWebServerFactory {
    WebServer getWebServer(ServletContextInitializer... initializers);
}
```
ServletContextInitializer表示ServletContext初始化器
```java
public interface ServletContextInitializer {
    void onStartup(ServletContext servletContext) throws ServletException;
}
```
如果你想注册你自己的Servlet，可以实现一个ServletContextInitializer，在web容器启动时，springboot会把所有实现了ServletContextInitializer接口的类收集起来，统一调用onStartup方法

springboot默认给我们注册了DispatcherSetvlet。

**内嵌式Web容器的创建和启动**
springboot通过抽象实现类AbstractApplicationContext实现refresh方法，refresh方法新建或刷新一个ApplicationContext，AbstractApplicationContext的子类可重写onRefresh方法，实现内嵌式web容器。springboot的refresh方法通过web容器工厂来创建web容器（调用tomcat或jetty的API创建各种组件）

**servlet注册方式**

- springboot启动类使用@ServletComponentScan注解来扫描注解了@WebServlet、@WebFilter、@WebListener的类。并注册到servlet容器中
- 使用ServletRegistrationBean、FilterRegistrationBean和ServletListenerRegistrationBean三个类分别注册servlet，filter，listener。
```java
@Bean
public ServletRegistrationBean servletRegistrationBean() {
return new ServletRegistrationBean(new HelloServlet(),"/hello");
}
```
- 动态注册,创建一个实现了ServletContextInitializer接口的类，并注册为一个bean，springboot负责调用onStartUp方法

**容器定制**

- 通过web容器工厂定制容器通用参数
```java
@Component
public class MyGeneralCustomizer implements
  WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
  
    public void customize(ConfigurableServletWebServerFactory factory) {
        factory.setPort(8081);
        factory.setContextPath("/hello");
     }
}
```
- 通过特定web容器工厂TomcatServletWebServerFactory进一步定制。
```java
class TraceValve extends ValveBase {
    @Override
    public void invoke(Request request, Response response) throws IOException, ServletException {

        request.getCoyoteRequest().getMimeHeaders().
        addValue("traceid").setString("1234xxxxabcd");

        Valve next = getNext();
        if (null == next) {
            return;
        }

        next.invoke(request, response);
    }
}
```

tomcat和jetty处理请求都通过责任链模式，前者Pipeline-value，后者通过HandlerWrapper实现。jetty的责任链实现了**回溯的链式调用**（先调用Handler初始化方法，再调用各Handler请求处理方法）。jetty通过ScopedHandler实现回溯的链式调用。ScopedHandler通过递归的方式来设置_outScope和_nextScope两个变量。层层递归调用中需要用到一些变量，如__outerScope，保存了Handler链中的头节点，ScopedHandler通过线程私有数据ThreadLocal来保存变量，兼顾变量保存和线程安全问题
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/22.png)

## tomcat日志框架
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/23.png)

slf4j时常用的门面日志，对外提供一套通用的日志记录API，而具体的日志输出服务有log4j，logback等

Servlet规范中定义了HttpServletRequest和HttpSession接口，Tomcat实现了这些接口，但具体实现细节并没有暴露给开发者，因此定义了两个包装类，RequestFacade和StandardSessionFacade。

Tomcat是通过Manager来管理Session的，默认实现是StandardManager。StandardContext持有StandardManager的实例，并存放了HttpSessionListener集合，Session在创建和销毁时，会通知监听器。

## tomcat的session同步

session同步方式

- 一种把所有Session数据放到一台服务器/数据库，集群中所有节点访问session服务器。
- 集群中节点间进行session数据同步拷贝
    - 将一个节点Session拷贝到集群中其他所有节点
    - 将一个节点的Session数据拷贝到另一个备份节点


**单播**：信息的接收和传递在两个节点之间进行，点对点通信。
**组播**：一台主机向指定一组主机发送数据报包。

每一个tomcat节点在启动时和运行时都会周期性发送组播心跳包，同一个集群内节点都在相同组播地址和端口监听这些信息；默认3s内不发送组播报文节点就会被认为已经奔溃了，从集群中删去。

**tomcat集群通信配置**
```java
// server.xml添加
<Cluster className-"org.apache.catalina.ha.tcp.SimpleTcpCluster/"
```

tomcat在集群规模少的时候使用deletamanager管理session，规模大时使用backmanager管理。

## JMX
JMX是一个为应用程序，设备，系统植入监控管理功能的框架。JMX通过管理MBean监控业务资源，MBean在JMX MBean服务器注册。
我们可以使用**JMX**来监控tomcat的指标（吞吐量，响应时间，错误数，线程池，CPU，JVM内存）

在tomcat的bin目录下新建一个setenv.sh文件
```java
export JAVA_OPTS="${JAVA_OPTS} -Dcom.sun.management.jmxremote"
export JAVA_OPTS="${JAVA_OPTS} -Dcom.sun.management.jmxremote.port=9001"
export JAVA_OPTS="${JAVA_OPTS} -Djava.rmi.server.hostname=x.x.x.x"
export JAVA_OPTS="${JAVA_OPTS} -Dcom.sun.management.jmxremote.ssl=false"
export JAVA_OPTS="${JAVA_OPTS} -Dcom.sun.management.jmxremote.authenticate=false"
```

使用Jconsole ip:port 访问tomcat运行情况

**常用tomcat指标查看命令**
```java
// 找到tomcat进程ID
ps -ef | gerp tomcat

// 查看tomcat进程状态信息
cat/proc/<pid>/status

// 查看CPU和内存使用情况
top -p <pid>

// 查看tomcat网络连接
netstat -na | grep 8080

// 查看网络流量
ifstat
```

**tomcat I/O模型选择**
一般情况默认使用NIO，除非WEB应用用到了TLS加密传输，而且对性能要求极高，可以考虑使用APR。APR通过OpenSSL处理TLS握手和加/解密。效率更高。
如果tomcat跑在windows平台，HTTP请求量很大，可以考虑NIO.2，因为Windows平台从操作系统层面真正实现了异步I/O，能够体现异步I/O效果。而linux只是在应用层面通过epoll模拟异步I/O模型

**线程数量设置**

- 利特尔法则：系统中请求数 = 请求的到达速率 * 每个请求处理时间

这样得到的每一个线程都有任务执行，但是对于I/O密集型应用，还是会在用户空间和内核空间之间的拷贝上发生阻塞问题，所以这类应用需增加线程数量

- 线程池大小 - （线程I/O阻塞时间 + 线程CPU时间）/线程CPU时间

总结：

- 请求处理时间越长，所需线程数越多，前提CPU核数足够
- I/O等待时间越长，所需线程数越多，前提CPU时间和I/O时间比率要足够准确
- 请求速率越快，所需线程数越多，前提CPU核数足够

具体的线程个数，要通过公式和实际压测调整看效果，达到最优。先设置一个较小线程数，进行压测，达到系统极限时（错误数增加，响应时间大幅增加），逐步加大线程数，当增加到某个值，效果再也没提升，甚至TPS下降，那可认为时最佳线程数

## tomcat拒绝连接原因

**java.net.SocketTimeoutException**

- 连接超时：指socket.connect方法超时，常常是因为网络不稳定
- 读取超时：指socket.read方法超时，可能是下游服务响应过长

**java.net.BindException: Address already in use: JVM_Bind**
端口被占用

**java.net.ConnectException: Connection refused: connect**
当客户端调用new socket(ip,port)或socke.connect函数时，可能抛出该异常。表示找不到该IP或port

**java.net.SocketException: Socket is closed**
通信一方主动关闭了Socket连接，而又进行读写时会报错

**java.net.SocketException: Connection reset/Connect reset by peer: Socket write error**
通信一方关闭socket，此时另一方正在进行**写**数据，报Connect reset by peer错误；此时另一方正在进行**读**数据，报Connection reset错误

**java.net.SocketException: Broken pipe**
通信一方收到Connect reset by peer后，依旧写数据会报错误

**java.net.SocketException: Too many open files**
进程打开文件句柄数超过限制。通常出现在并发用户数大的情况下，也可能是大量未关闭未使用socket连接

**maxConnections和acceptCount**
在TCP建立连接的三次握手过程中，服务端恢复syn+ack时，将这个连接保存到**半连接队列**。客户端返回ACK完成握手，服务端将链接移入accept队列，等待tomcat取走连接。

我们咋创建serversocket时会设置backlog值也就是tomcat中**acceptcount**，tomcat有somaxconn值，默认为128，tomcat会在两者间选择最小值作为accept队列大小。高并发情况下，tomcat来不及处理新连接时，会堆积在accept队列中，当满时，会报connection reset异常

maxConnections值tomcat任意时刻接收和处理最大连接数。满时acceptor线程不再从accept队列中取走连接。默认值根据IO模型有关（NIO 10000 APR 8192）

综述：acceptCount设置过大，请求等待时间较长，设置过小，高并发情况下会触发reset错误

## 定位CPU使用率高问题

- 使用TOP查看具体占用高CPU进程
- top -H -P <PID> 查看具体进程占用情况
- 查看其中线程情况
- 判断是单个线程占用过高还是，线程切换问题（vmstat查看操作系统层面线程上下文切换活动，）


## 操作系统层面调优
通过调整linux操作系统的默认值限制
/etc/security/limis.conf文件中：
```java
// 调整TCP缓冲区大小到16MB
sysctl -w net.core.rmem_max = 16777216
sysctl -w net.core.wmem_max = 16777216
sysctl -w net.ipv4.tcp_rmem =“4096 87380 16777216”
sysctl -w net.ipv4.tcp_wmem =“4096 16384 16777216”

// 高并发情况小调整TCP连接队列大小
sysctl -w net.core.somaxconn = 4096

// 控制java程序传入数据包队列大小
sysctl -w net.core.netdev_max_backlog = 16384
sysctl -w net.ipv4.tcp_max_syn_backlog = 8192
sysctl -w net.ipv4.tcp_syncookies = 1

// 增加端口使用范围
sysctl -w net.ipv4.ip_local_port_range =“1024 65535”
sysctl -w net.ipv4.tcp_tw_recycle = 1

// 为特定用户增加文件句柄数
用户名 hard nofile 40000
用户名 soft nofile 40000

// 使用合适的内核可用的拥塞控制算法列表 cubic
sysctl -w net.ipv4.tcp_congestion_control = cubic
```

jetty的调优从两点出发，acceptors个数应设为大于1，小于cpu核数；threadPoll队列通过和tomcat线程池一样，通过调试观察最佳线程数