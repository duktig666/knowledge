# Netty 基础

## Netty 是什么？

- Netty 是一款基于 NIO（Nonblocking I/O，非阻塞IO）开发的网络通信框架
- 极大地简化并优化了 TCP 和 UDP 套接字服务器等网络编程
- **支持多种协议** 如 FTP，SMTP，HTTP 以及各种二进制和基于文本的传统协议。

很多开源项目比如我们常用的 Dubbo、RocketMQ、Elasticsearch、gRPC 等等都用到了 Netty。

## 为什么要使用 Netty？

- 使用简单：统一的 API，支持多种传输类型，阻塞和非阻塞的
- 功能强大：自带编解码器解决 TCP 粘包/拆包问题，支持多种主流协议。
- 定制能力强：可以通过 ChannelHandler 对通信框架进行灵活地扩展。
- 性能高：通过与其他业界主流的 NIO 框架对比，Netty 的综合性能最优。
- 社区活跃：Netty 是活跃的开源项目，版本迭代周期短，bug 修复速度快。
- 成熟稳定：经历了大型项目的使用和考验，而且很多开源项目都使用到了 Netty， 比如我们经常接触的 Dubbo、RocketMQ 等等

## Netty的使用场景

Netty 主要用来做**网络通信** :

1. **作为 RPC 框架的网络通信工具** ：我们在分布式系统中，不同服务节点之间经常需要相互调用，这个时候就需要 RPC 框架了。不同服务节点之间的通信是如何做的呢？可以使用 Netty 来做。比如我调用另外一个节点的方法的话，至少是要让对方知道我调用的是哪个类中的哪个方法以及相关参数吧！
2. **实现一个自己的 HTTP 服务器** ：通过 Netty 我们可以自己实现一个简单的 HTTP 服务器，这个大家应该不陌生。说到 HTTP 服务器的话，作为 Java 后端开发，我们一般使用 Tomcat 比较多。一个最基本的 HTTP 服务器可要以处理常见的 HTTP Method 的请求，比如 POST 请求、GET 请求等等。
3. **实现一个即时通讯系统** ：使用 Netty 我们可以实现一个可以聊天类似微信的即时通讯系统，这方面的开源项目还蛮多的，可以自行去 Github 找一找。
4. **实现消息推送系统** ：市面上有很多消息推送系统都是基于 Netty 来做的。

## Netty高性能体现在什么地方？

- IO 线程模型：同步非阻塞，用最少的资源做更多的事。
- 内存零拷贝：尽量减少不必要的内存拷贝，实现了更高效率的传输。
- 内存池设计：申请的内存可以重用，主要指直接内存。内部实现是用一颗二叉查找树管理内存分配情况。
- 串形化处理读写：避免使用锁带来的性能开销。
- 高性能序列化协议：支持 protobuf 等高性能序列化协议。

## Netty 核心组件有哪些？分别有什么作用？

### 1. Channel

Channel 接⼝是 Netty 对⽹络操作抽象类，它除了包括基本的 I/O 操作，如 bind() 、 connect() 、 read() 、 write() 等。

⽐较常⽤的 Channel 接⼝实现类是 NioServerSocketChannel （服务端）和 NioSocketChannel （客户端），这两个 Channel 可以和 BIO 编程模型中的 ServerSocket 以及 Socket 两个概念对应上。

### 2. EventLoop 和 EventloopGroup

#### EventLoop 

EventLoop 定义了 Netty 的核⼼抽象，⽤于处理连接的⽣命周期中所发⽣的事件。

 **EventLoop 的主要作⽤实际就是负责监听⽹络事件并调⽤事件处理器进⾏相关 I/O 操作的处理。**

那 Channel 和 EventLoop 直接有啥联系呢？

Channel 为 Netty ⽹络操作(读写等操作)抽象类， EventLoop 负责处理注册到其上的 Channel 处理 I/O 操作，两者配合参与 I/O 操作。

#### EventloopGroup

![image-20220108162043255](https://cos.duktig.cn/typora/202201081620462.png)

EventLoopGroup 包含多个 EventLoop （每⼀个 EventLoop 通常内部包含⼀个线程），上⾯我们已经说了 EventLoop 的主要作⽤实际就是负责监听⽹络事件并调⽤事件处理器进⾏相关 I/O 操作的处理。

并且 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理，即 Thread 和 EventLoop 属于 1 : 1 的关系，从⽽保证线程安全。

上图是⼀个服务端对 EventLoopGroup 使⽤的⼤致模块图，其中 **Boss EventloopGroup ⽤于接收连接， Worker EventloopGroup ⽤于具体的处理（消息的读写以及其他逻辑处理）**。

### 3. ChannelFuture

Netty 是异步⾮阻塞的，所有的 I/O 操作都为异步的。

因此，我们不能⽴刻得到操作是否执⾏成功，但是，你可以通过 ChannelFuture 接⼝的 addListener() ⽅法注册⼀个 ChannelFutureListener ，当操作执⾏成功或者失败时，监听就会⾃动触发返回结果。

并且，你还可以通过 ChannelFuture 的 channel() ⽅法获取关联的 Channel

```java
public interface ChannelFuture extends Future<Void> {
    Channel channel();

    ChannelFuture addListener(GenericFutureListener<? extends Future<? super 
                              Void>> var1);
    ......

        ChannelFuture sync() throws InterruptedException;
}
```

通过 ChannelFuture 接⼝的 sync() ⽅法让异步的操作变成同步的。

### 4. ChannelHandler 和 ChannelPipeline 

ChannelHandler 是消息的具体处理器。他负责处理读写操作、客户端连接等事情。

ChannelPipeline 为 ChannelHandler 的链，提供了⼀个容器并定义了⽤于沿着链传播⼊站和出站事件流的 API 。当 Channel 被创建时，它会被⾃动地分配到它专属的 ChannelPipeline 。

我们可以在 ChannelPipeline 上通过 addLast() ⽅法添加⼀个或者多个 ChannelHandler ，因为⼀个数据或者事件可能会被多个 Handler 处理。当⼀个 ChannelHandler 处理完之后就将数据交给下⼀个 ChannelHandler 。

### 5. Bootstrap 和 ServerBootstrap

Bootstrap 是客户端的启动引导类/辅助类，具体使⽤⽅法如下：

```java
EventLoopGroup group = new NioEventLoopGroup();
try {
    //创建客户端启动引导/辅助类：Bootstrap
    Bootstrap b = new Bootstrap();
    //指定线程模型
    b.group(group).
        ......
        // 尝试建⽴连接
        ChannelFuture f = b.connect(host, port).sync();
    f.channel().closeFuture().sync();
} finally {
    // 优雅关闭相关线程组资源
    group.shutdownGracefully();
}
```

ServerBootstrap 客户端的启动引导类/辅助类，具体使⽤⽅法如下：

```java
// 1.bossGroup ⽤于接收连接，workerGroup ⽤于具体的处理
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
    //2.创建服务端启动引导/辅助类：ServerBootstrap
    ServerBootstrap b = new ServerBootstrap();
    //3.给引导类配置两⼤线程组,确定了线程模型
    b.group(bossGroup, workerGroup).
        ......
        // 6.绑定端⼝
        ChannelFuture f = b.bind(port).sync();
    // 等待连接关闭
    f.channel().closeFuture().sync();
} finally {
    //7.优雅关闭相关线程组资源
    bossGroup.shutdownGracefully();
    workerGroup.shutdownGracefully();
}
```

从上⾯的示例中，我们可以看出：

1. Bootstrap 通常使⽤ `connet()` ⽅法连接到远程的主机和端⼝，作为⼀个 Netty TCP 协议通信中的客户端。另外， Bootstrap 也可以通过 bind() ⽅法绑定本地的⼀个端⼝，作为 UDP 协议通信中的⼀端。
2. ServerBootstrap 通常使⽤ `bind()` ⽅法绑定本地的端⼝上，然后等待客户端的连接。
3. Bootstrap 只需要配置⼀个线程组— `EventLoopGroup` ,⽽ ServerBootstrap 需要配置两个线程组— EventLoopGroup ，⼀个⽤于接收连接，⼀个⽤于具体的处理。

## 什么是 Netty 的零拷贝？

> 维基百科是这样介绍零拷贝的：
>
> 零复制（英语：Zero-copy；也译零拷贝）技术是指 **计算机执行操作时，CPU 不需要先将数据从某处内存复制到另一个特定区域**。这种技术通常用于通过网络传输文件时节省 CPU 周期和内存带宽。

在 OS 层面上的 Zero-copy 通常指避免在 用户态(User-space) 与 内核态(Kernel-space) 之间来回拷贝数据。而在 Netty 层面 ，零拷贝主要体现在对于数据操作的优化。

Netty 中的零拷贝体现在以下几个方面：

（1）Netty提供CompositeByteBuf组合缓冲区类, 可以将多个ByteBuf合并为一个逻辑上的ByteBuf, 避免了各个ByteBuf之间的拷贝。

（2）Netty提供了ByteBuf的浅层复制操作（slice、duplicate），可以将ByteBuf分解为多个共享同一个存储区域的ByteBuf, 避免内存的拷贝。

（3）在使用Netty进行文件传输时，可以调用FileRegion包装的transferTo方法，直接将文件缓冲区的数据发送到目标Channel，避免普通的循环读取文件数据和写入通道所导致的内存拷贝问题。

（4）在将一个byte数组转换为一个ByteBuf对象的场景，Netty提供了一系列的包装类，避免了转换过程中的内存拷贝。

（5）如果Channel接收和发送ByteBuf都使用direct直接内存进行Socket读写，不需要进行缓冲区的二次拷贝。但是，如果使用JVM的堆内存进行Socket读写，JVM会将堆内存Buffer拷贝一份到直接内存中，然后才写入Socket中，相比于使用直接内存，这种情况在发送过程中会多出一次缓冲区的内存拷贝。所以，在发送ByteBuffer到Socket时，尽量使用直接内存而不是JVM堆内存。

## Netty 的线程模型——Reactor 模式

### 简介

Reactor 模式基于事件驱动，采⽤多路复⽤将事件分发给相应的 Handler 处理，**⾮常适合处理海量 IO 的场景**。

包括三种角色：Reactor、Acceptor 和 Handler。Reactor用来监听事件，包括：连接建立、读就绪、写就绪等。然后针对监听到的不同事件，将它们分发给对应的线程去处理。其中acceptor处理客户端建立的连接，handler对读写事件进行业务处理。

Reactor线程模型消息处理的流程：

- Reactor线程通过多路复用器监控IO事件。
- 如果是连接建立的事件，则由acceptor线程来接受连接，并创建handler来处理之后该连接上的读写事件。
- 如果是读写事件，则Reactor会调用该连接上的handler进行业务处理。

在 Netty 主要靠 `NioEventLoopGroup` 线程池来实现具体的线程模型的 。

我们实现服务端的时候，⼀般会初始化两个线程组：

1. `bossGroup` :接收连接。
2. `workerGroup` ：负责具体的处理，交由对应的 Handler 处理。

下⾯我们来详细看⼀下 Netty 中的线程模型吧！

### Reactor线程模型的三种模式

#### 单Reactor单线程模式

⼀个线程需要执⾏处理所有的 accept 、 read 、 decode 、 process 、 encode 、 send 事件。对于⾼负载、⾼并发，并且对性能要求⽐较⾼的场景不适⽤。

对应到 Netty 代码是下⾯这样的

```java
//1.eventGroup既⽤于处理客户端连接，⼜负责具体的处理。
EventLoopGroup eventGroup = new NioEventLoopGroup(1);
//2.创建服务端启动引导/辅助类：ServerBootstrap
ServerBootstrap b = new ServerBootstrap();
boobtstrap.group(eventGroup, eventGroup)
    //......
```

使⽤ NioEventLoopGroup 类的⽆参构造函数设置线程数量的默认值就是 CPU 核⼼数 *2 。

#### 单Reactor多线程模式

⼀个 Acceptor 线程只负责监听客户端的连接，⼀个 NIO 线程池负责具体处理： accept 、 read 、 decode 、 process 、 encode 、 send 事件。满⾜绝⼤部分应⽤场景，并发连接量不⼤的时候没啥问题，但是遇到并发连接⼤的时候就可能会出现问题，成为性能瓶颈。

```java
// 1.bossGroup ⽤于接收连接，workerGroup ⽤于具体的处理
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
    //2.创建服务端启动引导/辅助类：ServerBootstrap
    ServerBootstrap b = new ServerBootstrap();
    //3.给引导类配置两⼤线程组,确定了线程模型
    b.group(bossGroup, workerGroup)
        //......
```

#### 主从Reactor多线程模式

在单Reactor多线程模式的基础上，使用两个Reactor线程分别对建立连接事件和读写事件进行监听，每个Reactor线程拥有一个多路复用器。当主Reactor线程监听到连接建立事件后，创建SocketChannel，然后将SocketChannel注册到子Reactor线程的多路复用器中，使子Reactor线程监听连接的读写事件。

```java
// 1.bossGroup ⽤于接收连接，workerGroup ⽤于具体的处理
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
    //2.创建服务端启动引导/辅助类：ServerBootstrap
    ServerBootstrap b = new ServerBootstrap();
    //3.给引导类配置两⼤线程组,确定了线程模型
    b.group(bossGroup, workerGroup)
        //......
```

## Netty 为什么使用长连接？

TCP 在进⾏读写之前，server 与 client 之间必须提前建⽴⼀个连接，建⽴连接的过程，需要我们常说的三次握⼿，释放/关闭连接的话需要四次挥⼿。这个过程是⽐较消耗⽹络资源并且有时间延迟的。

短连接说的就是 server 端 与 client 端建⽴连接之后，读写完成之后就关闭掉连接，如果下⼀次再要互相发送消息，就要重新连接。短连接的有点很明显，就是管理和实现都⽐较简单，缺点也很明显，每⼀次的读写都要建⽴连接必然会带来⼤量⽹络资源的消耗，并且连接的建⽴也需要耗费时间。

⻓连接说的就是 client 向 server 双⽅建⽴连接之后，即使 client 与 server 完成⼀次读写，它们之间的连接并不会主动关闭，后续的读写操作会继续使⽤这个连接。⻓连接的可以省去较多的 TCP 建⽴和关闭的操作，降低对⽹络资源的依赖，节约时间。对于频繁请求资源的客户来说，⾮常适⽤⻓连接。

## 为什么需要⼼跳机制？Netty 中⼼跳机制了解么？

在 TCP 保持⻓连接的过程中，可能会出现断⽹等⽹络异常出现，异常发⽣的时候， client 与 server 之间如果没有交互的话，它们是⽆法发现对⽅已经掉线的。为了解决这个问题, 我们就需要引⼊ **⼼跳机制** 。

**⼼跳机制的⼯作原理是**: 在 client 与 server 之间在⼀定时间内没有数据交互时, 即处于 idle 状态时, 客户端或服务器就会发送⼀个特殊的数据包给对⽅, 当接收⽅收到这个数据报⽂后, 也⽴即发送⼀个特殊的数据报⽂, 回应发送⽅, 此即⼀个 PING-PONG 交互。所以, 当某⼀端收到⼼跳消息后, 就知道了对⽅仍然在线, 这就确保 TCP 连接的有效性.

TCP 实际上⾃带的就有⻓连接选项，本身是也有⼼跳包机制，也就是 TCP 的选项： SO_KEEPALIVE 。 但是，TCP 协议层⾯的⻓连接灵活性不够。所以，⼀般情况下我们都是在应⽤层协议上实现⾃定义⼼跳机制的，也就是在 Netty 层⾯通过编码实现。通过 Netty 实现⼼跳机制的话，核⼼类是 IdleStateHandler 。

## 什么是 TCP的粘包和拆包？Netty如何解决的？

TCP 粘包/拆包 就是你基于 TCP 发送数据的时候，出现了多个字符串“粘”在了⼀起或者⼀个字符串被“拆”开的问题。⽐如你多次发送：“你好,你真帅啊！哥哥！”，但是客户端接收到的可能是下⾯这样的：

![image-20220108155937215](https://cos.duktig.cn/typora/202201081559263.png)

应用程序写入的字节大小大于套接字发送缓冲区的大小，会发生拆包现象，而应用程序写入数据小于套接字缓冲区大小，网卡将应用程序多次写入的数据封装成一个数据包发送到网络上，这将会发生粘包现象。

那有什么解决办法呢？

1、使用 Netty 自带的解码器

- `LineBasedFrameDecoder `: 发送端发送数据包的时候，每个数据包之间以换⾏符作为分隔， LineBasedFrameDecoder 的⼯作原理是它依次遍历 ByteBuf 中的可读字节，判断是否有换⾏符，然后进⾏相应的截取。
- `DelimiterBasedFrameDecoder` : 可以⾃定义分隔符解码器， LineBasedFrameDecoder 实际上是⼀种特殊的 DelimiterBasedFrameDecoder 解码器。
- `FixedLengthFrameDecoder` : 固定⻓度解码器，它能够按照指定的⻓度对消息进⾏相应的拆包。
- `LengthFieldBasedFrameDecoder`：数据长度解码器，将发送的消息分为header和body，header存储消息的长度（字节数），body是发送的消息的内容。同时发送方和接收方要协商好这个header的字节数，因为int能表示长度，long也能表示长度。接收方首先从字节流中读取前n（header的字节数）个字节（header），然后根据长度读取等量的字节，不够就从下一个数据流中查找。

2、⾃定义序列化编解码器

在 Java 中⾃带的有实现 Serializable 接⼝来实现序列化，但由于它性能、安全性等原因⼀般情
况下是不会被使⽤到的。

通常情况下，我们使⽤ Protostuff、Hessian2、json 序列⽅式⽐较多，另外还有⼀些序列化性能
⾮常好的序列化⽅式也是很好的选择：

专⻔针对 Java 语⾔的：Kryo，FST 等等
跨语⾔的：Protostuff（基于 protobuf 发展⽽来），ProtoBuf，Thrift，Avro，MsgPack 等等



# Netty 高级

## Netty 百万级连接调优

### 操作系统参数调优

1. 修改 **操作系统连接的最大句柄数** 为100W（默认1024）。

   1. `cat /proc/sys/fs/file-max` 查看最大句柄数是否满足需求

   2. `vim /etc/sysctl.conf` 插入配置

      ```sh
      fs.file-max =1000000
      ```

   3. 执行 `sysctl -p` 使命令生效

2. 修改 **单进程打开的最大句柄数**为100W

   1. `ulimit -a` 命令查看当前设置的值是否满足要求

      ![image-20220108170836545](https://cos.duktig.cn/typora/202201081708869.png)

      当并发接入的TCP连接数超过上限时，就会提示“too many open files”，所有新的客户端接入将失败。

   2. 通过 `vi /etc/security/limits.conf` 命令添加如下配置参数:

      ```java
      * soft nofile 1000000
      * hard nofile 1000000
      ```

   3. 修改之后保存，注销当前用户，重新登录，通过`ulimit -a`命令查看修改是否生效。

### TCP/IP相关参数

1. net.ipv4.tcp_rmem:为每个TCP连接分配的读缓冲区内存大小。第一个值是socket接收缓冲区分配的最小字节数。第二个值是默认值，缓冲区在系统负载不高的情况下可以增长到该值。第三个值是接收缓冲区分配的最大字节数。

2. net.ipv4.tcp_wmem:为每个TCP连接分配的写缓冲区内存大小。第一个值是socket发送缓冲区分配的最小字节数。第二个值是默认值，缓冲区在系统负载不高的情况下可以增长到该值。第三个值是发送缓冲区分配的最大字节数。

3. net.ipv4.tcp_mem:内核分配给TCP连接的内存，单位是page ( 1个page通常为4096字节，可以通过#getconf PAGESIZE命令查看)，包括最小、默认和最大三个配置项。  （分配给tcp连接的内存 一个TCP连接大约占7.5KB ）

4. net.ipv4.tcp_keepalive_time:最近一次数据包发送与第一次keep alive探测消息发送的时间间隔，用于确认TCP连接是否有效。

5. tcp_keepalive_intvl:在未获得探测消息响应时，发送探测消息的时间间隔。
6. tcp_keepalive_probes:判断TCP连接失效连续发送的探测消息个数，达到之后判定连接失效。
7. net.ipv4.tcp_tw_reuse:是否允许将TIME_WAIT Socket重新用于新的TCP连接，默认为0，表示关闭。
8. net.ipv4.tcp_tw_recycle:是否开启TCP连接中TIME_WAIT Socket 的快速回收功能，默认为0，表示关闭。
9. net.ipv4.tcp_fin_timeout:套接字自身关闭时保持在FIN_WAIT_2状态的时间，默认为60。
10. **net.ipv4.ip_local_port_range = 1024 65535: 客户端修改端口范围的限制，如果按上述端口范围进行设置，则理论上单独一个进程最多可以同时建立60000多个TCP客户端连接**。
11. **net.ipv4.tcp_mem = 786432 2097152 3145728： 分配给tcp连接的内存，单位是page（1个Page通常是4KB，可以通过getconf PAGESIZE命令查看），三个值分别是最小、默认、和最大。比如以上配置中的最大是3145728，那分配给tcp的最大内存=31457284 / 1024 / 1024 = 12GB。一个TCP连接大约占7.5KB，粗略可以算出百万连接≈7.5*1000000/4=1875000 3145728足以满足测试所需**。

通过 `vi/etc/sysctl.conf` 命令对上述网络参数进行优化，具体修改如下（大约可以接入50万个连接，可以根据业务需要调整参数)：

```sh
net.ipv4.tcp_wmem = 4096 87380 4194304
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_mem = 64608 1048576 2097152
net.ipv4.tcp_keepalive_time = 1800
net.ipv4.tcp_keepalive_intvl = 20
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
```

修改完成后，通过执行 `sysctl -p` 命令使配置立即生效。

### Netty 性能调优

#### 设置合理的线程数

**boss线程池优化**

对于Netty服务端，通常只需要启动一个监听端口用于端侧设备接入，但是如果集群实例较少，甚至是单机部署，那么在短时间内大量设备接入时，需要对服务端的监听方式和线程模型做优化，即服务端监听多个端口，利用主从Reactor线程模型。由于同时监听了多个端口，每个ServerSocketChannel都对应一个独立的Acceptor线程，这样就能并行处理，加速端侧设备的接人速度，减少端侧设备的连接超时失败率，提高单节点服务端的处理性能。

**work线程池优化（IO工作线程池）**
对于I/O工作线程池的优化，可以先采用系统默认值（cpu内核数*2）进行性能测试，在性能测试过程中采集I/O线程的CPU占用大小，看是否存在瓶颈。

#### 心跳优化

- 及时检测失效的连接，将其剔除，防止无效的连接句柄积压，导致OOM等问题
- 设置合理的心跳周期，防止心跳定时任务积压，造成频繁的老年代GC(新生代和老年代都有导致STW的GC,不过耗时差异较大)，导致应用暂停
- 使用Nety提供的链路空闲检测机制，不要自己创建定时任务线程池，加重系统的负担，以及增加潜在的并发安全问题。

#### 接收和发送缓冲区调优

在一些场景下，端侧设备会周期性地上报数据和发送心跳，单个链路的消息收发量并不大，针对此类场景，可以通过调小TCP的接收和发送缓冲区来降低单个TCP连接的资源占用率。

#### 合理使用内存池

随着JVM虚拟机和JT即时编译技术的发展,对象的分配和回收是一个非常轻量级的工作。但是对于缓冲区 Buffer,情况却稍有不同,特别是堆外直接内存的分配和回收,是一个耗时的操作。

每次消息读写都需要创建和释放ByteBuf对象，如果有100万个连接，每秒上报一次数据或者心跳，就会有100万次/秒的ByteBuf对象申请和释放，即便服务端的内存可以满足要求，GC的压力也会非常大。

如果使用内存池，则当A链路接收到新的数据报时，从NioEventLoop 的内存池中申请空闲的ByteBuf，解码后调用release将 ByteBuf释放到内存池中，供后续的B链路使用。

Netty默认的IO读写操作采用的都是内存池的堆外直接内存模式,如果用户需要额外使用 ByteBuf,建议也采用内存池方式;如果不涉及网络IO操作(只是纯粹的内存操作),可以使用堆内存池,这样内存的创建效率会更高一些。

#### 防止I/O线程被意外阻塞

通常情况下，大家都知道不能在Netty 的IO线程上做执行时间不可控的操作，例如访问数据库、调用第三方服务等。但是有些隐形的阻塞操作却容易被忽略，例如打印日志。

在生产环境中，通常需要实时打印接口日志，其他日志处于ERROR级别，当服务发生I/O异常时，会记录异常日志。如果当前磁盘的 WIO比较高，写日志文件操作可能会被同步阻塞（阻塞时间无法预测)。这就会导致Netty的 NioEventLoop线程被阻塞，Socket链路无法被及时关闭，其他的链路也无法进行读写操作。

#### I/O线程与业务线程分离

如果服务端业务逻辑简单，则可以通过调大NioEventLoop工作线程池的方式，直接在I/O线程执行业务ChannelHandler，这样便减少了一次线程上下文切换，性能反而更高。

如果有复杂的业务逻辑，则建议I/O线程与业务线程分离，对于I/O线程，不存在锁竞争，可以创建一个大的NioEventLoopGroup线程组，所有channel共享同一个线程池，对于后端的业务线程，则建议创建多个小的业务线程池，线程池与I/O线程绑定，这样既减少了锁竞争，又提升了后端的处理性能。

### JVM层面相关性能优化

JVM层面的调优主要涉及GC参数优化,GC参数设置不当会导致频繁GC,甚至OOM异常，对服务端的稳定运行产生重大影响。

**1、确定GC优化目标**

(1）吞吐量:是评价GC能力的重要指标，在不考虑GC引起的停顿时间或内存消耗时，吞吐量是GC能支撑应用程序达到的最高性能指标。

(2）延迟:GC能力的最重要指标之一，是由于GC引起的停顿时间，优化目标是缩短延迟时间或完全消除停顿（ STW)，避免应用程序在运行过程中发生抖动。

(3）内存占用:GC正常时占用的内存量。

JVM GC调优的三个基本原则如下：

( 1) Minor GC回收原则：每次新生代GC回收尽可能多的内存，减少应用程序发生Full GC的频率。

(2)GC内存最大化原则：垃圾收集器能够使用的内存越大，垃圾收集效率越高，应用程序运行也越流畅。但是过大的内存一次Full GC耗时可能较长,如果能够有效避免 FullGC，就需要做精细化调优。

(3)3选2原则：吞吐量、延迟和内存占用不能兼得，无法同时做到吞吐量和暂停时间都最优，需要根据业务场景做选择。（对于大多数IoT应用，吞吐量优先，其次是延迟。当然对于时延敏感型的业务，需要调整次序。）

**2、确定服务端内存占用**

在优化GC之前，需要确定应用程序的内存占用大小，以便为应用程序设置合适的内存，提升GC效率。内存占用与活跃数据有关，活跃数据指的是应用程序稳定运行时长时间存活的Java对象。

活跃数据的计算方式：通过GC日志采集GC数据，获取应用程序稳定时老年代占用的Java堆大小，以及永久代（元数据区）占用的Java 堆大小，两者之和就是活跃数据的内存占用大小。

**3、Java堆大小设置原则**

![image-20220108200405259](https://cos.duktig.cn/typora/202201082004545.png)

**4、垃圾收集器的选择**

JDK1.8以上版本，建议以选择G1，如果较低版本，可以使用“ParNew+CMS”。

CMS吞吐量调优

         1. 增加新生代空间，降低新生代GC频率，减少固定时间内新生代GC的次数。
         2. 增加老年代空间，降低CMS的频率并减少内存碎片，最终减小并发模式失效引起FullGC发生的概率。
         3. 调整新生代Eden和Survivor 空间的大小比例，减少由新生代晋升到老年代的对象数目，降低CMS GC频率

G1调优

1. 不要使用-Xmn选型或者-XX:NewRatio等其他相关选型显式设置年轻代的大小，这样会覆盖暂停时间指标。
2. 暂停时间不要设置得太小，否则为了达到暂停时间目标会增加垃圾回收的开销，影响吞吐量指标。
3. 防止触发Full GC:在某些情况下，例如并发模式失败，GI会触发Full GC,这时GI会退化使用Serial 收集器来完成垃圾清理工作，它仅使用单线程来完成GC, GC暂停时间可能会达到秒级。

**5、一些GC调优的误区**

(1）在生产环境中不配置GC日志打印参数，担心影响业务性能。

(2) GC日志格式选择-XX:+PrintGCTimeStamps，导致GC日志很难跟其他业务日志对应起来。

(3）GC日志文件路径设置为静态路径，例如 gc.log，没有配置绕接、切换策略,导致重启之后日志被覆盖。

(4) GC日志文件没有配置单个文件大小、绕接和备份机制，导致单个GC文件过大。

(5)只有Full GC才会导致应用暂停,分析STW问题时直接使用Full GC关键字搜索，其他的不看。

(6）给出最优的GC参数或者明确GC优化方向之后，通过一次调整就能解决问题。

(7）业务内存使用不当导致的性能问题，希望通过GC参数优化解决问题，业务不用改代码。

(8）只有GC才会导致应用暂停。

## 参看

- 《Netty进阶之路 跟着案例学Netty》
- [阿里大牛总结的Netty最全常见面试题，面试再也不怕被问Netty了](https://zhuanlan.zhihu.com/p/148726453)
- [互联网大厂Java面试题——Netty 面试题解析](https://cloud.tencent.com/developer/article/1400748)
- [Netty面试题和解答(一)](https://www.cnblogs.com/xiaoyangjia/p/11526197.html)

