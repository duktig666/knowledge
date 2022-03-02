##  基础

### UNIX网络编程的5种I/O模型

**阻塞I/O模型**：文件操作的阻塞的。在进程空间中调用recvfrom，其系统调用直到数据包到达且被复制到应用进程的缓冲区中或者发生错误时才返回，在此期间一直会等待，进程在从调用recvfrom开始到它返回的整段时间内都是被阻塞的。

![image-20220212111311628](https://cos.duktig.cn/typora/202202121114216.png)

**非阻塞I/O模型**：recvfrom从应用层到内核的时候，如果该缓冲区没有数据的话，就直接返回一个EWOULDBLOCK错误，一般都对非阻塞I/О模型进行轮询检查这个状态，看内核是不是有数据到来。

![image-20220212111530049](https://cos.duktig.cn/typora/202202121115050.png)

**I/O复用模型**：Linux提供 select/poll，进程通过将一个或多个fd传递给select或poll系统调用，阻塞在select操作上，这样select/poll可以帮我们侦测多个fd是否处于就绪状态。**select/poll 是顺序扫描fd是否就绪，而且支持的fd数量有限，因此它的使用受到了一些制约**。Linux还提供了一个epoll系统调用，**epoll 使用基于事件驱动方式代替顺序扫描，因此性能更高**。当有fd就绪时，立即回调函数rollback。

![image-20220212111545427](https://cos.duktig.cn/typora/202202121115831.png)

**信号驱动1/O模型**：首先开启套接口信号驱动IO功能，并通过系统调用sigaction执行一个信号处理函数（此系统调用立即返回，进程继续工作，它是非阻塞的)。当数据准备就绪时，就为该进程生成一个SIGIO信号，通过信号回调通知应用程序调用recvfrom来读取数据，并通知主循环函数处理数据。

![image-20220212111614410](https://cos.duktig.cn/typora/202202121116263.png)

**异步IO**：告知内核启动某个操作，并让内核在整个操作完成后（包括将数据从内核复制到用户自己的缓冲区）通知我们。这种模型与信号驱动模型的主要区别是：信号驱动IO由内核通知我们何时可以开始一个I/O操作;**异步IO模型由内核通知我们IO操作何时已经完成**。

![image-20220212111627757](https://cos.duktig.cn/typora/202202121117813.png)

### I/O多路复用技术

在IO编程过程中，当需要同时处理多个客户端接入请求时，可以利用多线程或者IO多路复用技术进行处理。

I/О多路复用技术通过把多个I/O的阻塞复用到同一个select 的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。

与传统的多线程/多进程模型比，**I/O 多路复用的最大优势是系统开销小，系统不需要创建新的额外进程或者线程，也不需要维护这些进程和线程的运行，降低了系统的维护工作量，节省了系统资源**。

I/О多路复用的主要应用场景如下：

- 服务器需要同时处理多个处于监听状态或者多个连接状态的套接字;
- 服务器需要同时处理多种网络协议的套接字。

目前支持IO多路复用的系统调用有select、pselect、poll、epoll，在 Linux网络编程过程中，很长一段时间都使用select做轮询和网络事件通知，然而 select的一些固有缺陷导致了它的应用受到了很大的限制，最终Linux不得不在新的内核版本中寻找select的替代方案，最终选择了epoll。

epoll作了很多重大改进，现总结如下：

- **支持一个进程打开的socket描述符(FD，文件描述符)不受限制(仅受限于操作系统的最大文件句柄数)**。
  - select最大的缺陷就是单个进程所打开的FD是有一定限制的，它由FD_SETSIZE设置，默认值是1024。对于那些需要支持上万个TCP连接的大型服务器来说显然太少了。
  - epoll并没有这个限制，它所支持的FD上限是**操作系统的最大文件句柄数**，这个数字远远大于1024。例如，在1GB内存的机器上大约是10 万个句柄左右，具体的值可以通过 `cat/proc/sys/fs/file- max`察看，通常情况下这个值跟系统的内存关系比较大。
- **I/О效率不会随着FD数目的增加而线性下降**
  - 任一时刻只有少部分的 socket 是“活跃”的，但是select/poll每次调用都会线性扫描全部的集合，导致效率呈现线性下降。
  - epoll 不存在这个问题，它只会对“活跃”的socket进行操作——这是因为在内核实现中, epoll是根据每个fd上面的callback函数实现的。那么，只有“活跃”的 socket才会去主动调用callback函数。
- **使用mmap加速内核与用户空间的消息传递。**
  - 无论是select、poll还是epoll都需要内核把FD消息通知给用户空间，如何避免不必要的内存复制就显得非常重要，**epoll是通过内核和用户空间mmap同一块内存来实现的**。
- **epoll 的API更加简单**

## NIO基础

非阻塞I/O (Non-block I/O)，与Socket类和ServerSocket类相对应，NIO也提供了SocketChannel和ServerSocketChannel两种不同的套接字通道实现。这两种新增的通道都支持阻塞和非阻塞两种模式。

### NIO 类库

#### 1．缓冲区Buffer

Buffer是一个对象，它包含一些要写入或者要读出的数据。

**在NIO库中，所有数据都是用缓冲区处理的**。在读取数据时，它是直接读到缓冲区中的;在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

缓冲区实质上是一个数组。通常它是一个字节数组（ByteBuffer)，也可以使用其他种类的数组。但是一个缓冲区不仅仅是一个数组，缓冲区提供了对数据的结构化访问以及维护读写位置( limit）等信息。

#### 2．通道Channel

Channel是一个**全双工的 通道**，网络数据通过Channel 读取和写入。通道与流的不同之处在于通道是双向的，**流只是在一个方向上移动（一个流必须是InputStream或者OutputStream的子类)，而通道可以用于读、写或者二者同时进行**。

#### 3．多路复用器Selector

多路复用器Selector，**提供选择已经就绪的任务的能力**。Selector 会不断地轮询注册在其上的Channel，如果某个Channel上面发生读或者写事件，这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey可以获取就绪Channel的集合，进行后续的I/O操作。

**一个多路复用器Selector可以同时轮询多个Channel**，由于JDK使用了epoll()代替传统的select实现，所以它并没有最大连接句柄1024/2048的限制。这也就意味着只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端。

### NIO 实现

#### NIO服务端序列图

![image-20220212152159295](https://cos.duktig.cn/typora/202202121522298.png)

#### NIO客户端序列图

![image-20220212153455920](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20220212153455920.png)

### 选择 Netty 的理由

Netty是业界最流行的NIO框架之一，它的健壮性、功能、性能、可定制性和可扩展性在同类框架中都是首屈一指的，它已经得到成百上千的商用项目验证，例如Hadoop 的RPC框架Avro就使用了Netty 作为底层通信框架，其他还有业界主流的RPC框架，也使用Netty来构建高性能的异步通信能力。

通过对Netty 的分析，我们将它的优点总结如下。

- API使用简单，开发门槛低;
- 功能强大，预置了多种编解码功能，支持多种主流协议﹔
- 定制能力强，可以通过ChannelHandler对通信框架进行灵活地扩展;性能高，通过与其他业界主流的NIO框架对比，Netty的综合性能最优;
- 成熟、稳定，Netty修复了已经发现的所有JDK NIO BUG，业务开发人员不需要再为NIO的 BUG而烦恼;
- 社区活跃，版本迭代周期短，发现的BUG可以被及时修复，同时，更多的新功能会加入;
- 经历了大规模的商业应用考验，质量得到验证。Netty在互联网、大数据、网络游戏、企业应用、电信软件等众多行业已经得到了成功商用，证明它已经完全能够满足不同行业的商业应用了。正是因为这些优点，Netty逐渐成为了Java NIO编程的首选框架。

### 第一个Netty程序

#### 服务端

TimeServer

```java
public class TimeServer {

    public void bind(int port) throws Exception {

        // 配置 Reactor 线程组
        // 接受 客户端连接
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        // 进行SocketChannel的网络读写
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            // NIO 服务端启动服务类
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChildChannelHandler());

            // 绑定端口 同步阻塞，等待绑定操作完成
            ChannelFuture f = b.bind(port).sync();

            // 等待服务端链路关闭后再结束 主函数
            f.channel().closeFuture().sync();
        } finally {
            // 退出，释放线程池资源
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }


    }

    private class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

        @Override
        protected void initChannel(SocketChannel socketChannel) throws Exception {
            socketChannel.pipeline().addLast(new TimeServerHandler());
        }

    }

    public static void main(String[] args) throws Exception {
        int port = 8081;
        new TimeServer().bind(port);
    }
    
}
```

TimeServerHandler

```java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("TimeServerHandler:" + msg);
        // 转换成 ByteBuf 对象
        ByteBuf buf = (ByteBuf) msg;
        // 创建 可读字节数 的数组
        byte[] req = new byte[buf.readableBytes()];
        // 将缓冲区的字节数组，复制到新的数组当中
        buf.readBytes(req);
        String body = new String(req, StandardCharsets.UTF_8);
        System.out.println("The time server receive order : " + body);
        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ?
                new java.util.Date(System.currentTimeMillis()).toString() : "BAD ORDER";
        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        // 异步发送应答消息给 客户端（write方法只是把待发送的消息放到发送缓冲数组中，真正发送要使用 flush()）
        ctx.write(resp);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        /*
        发送队列中的消息写入到SocketChannel 中发送给对方。

        从性能角度考虑，为了防止频繁地唤醒Selector进行消息发送，Netty的 write方法并不直接将消息写入SocketChannel 中，
        调用write方法只是把待发送的消息放到发送缓冲数组中，再通过调用flush方法，
        将发送缓冲区中的消息全部写到SocketChannel 中。
         */
        ctx.flush();
    }

    /**
     * 连接异常或者退出时 执行
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

}
```

#### 客户端

TimeClient

```java
public class TimeClient {
    public void connect(int port, String host) throws Exception {//配置客户端NIo线程组
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group).channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        /** 初始化时，将Handler设置到ChannelPipeline */
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            //发起异步连接操作，等待连接成功
            ChannelFuture f = b.connect(host, port).sync();
            //等待客户端链路关闭
            f.channel().closeFuture().sync();
        } finally {
            //优雅退出，释放NIo线程组
            group.shutdownGracefully();
        }
    }


    public static void main(String[] args) throws Exception {
        int port = 8081;
        new TimeClient().connect(port, "127.0.0.1");
    }
}
```

TimeClientHandler

```java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    private final ByteBuf firstMessage;

    public TimeClientHandler() {
        byte[] req = "QUERY TIME ORDER".getBytes();
        firstMessage = Unpooled.copiedBuffer(req);
        firstMessage.writableBytes();
    }

    /**
     * 客户端与服务端连接后，会调用此方法
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        ctx.writeAndFlush(firstMessage);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, StandardCharsets.UTF_8);
        System.out.println("Now is : " + body);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //释放资源
        System.out.println("Unexpected exception from downstream : " + cause.getMessage());
        ctx.close();
    }

}
```

#### 执行结果

服务端输出：

```java
TimeServerHandler:PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024)
The time server receive order : QUERY TIME ORDER
```

客户端输出：

```java
Now is : Sat Feb 12 17:29:20 CST 2022
```

## Tcp的拆包和粘包

### 问题概述

> 一个完整的包可能会被TCP拆分成多个包进行发送，也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。

TCP 粘包/拆包 就是你基于 TCP 发送数据的时候，出现了多个字符串“粘”在了⼀起或者⼀个字符串被“拆”开的问题。⽐如你多次发送：“你好,你真帅啊！哥哥！”，但是客户端接收到的可能是下⾯这样的：

![image-20220108155937215](https://cos.duktig.cn/typora/202201081559263.png)

应用程序写入的字节大小大于套接字发送缓冲区的大小，会发生拆包现象，而应用程序写入数据小于套接字缓冲区大小，网卡将应用程序多次写入的数据封装成一个数据包发送到网络上，这将会发生粘包现象。

### 解决办法

TCP以流的方式进行数据传输，上层的应用协议为了对消息进行区分，往往采用如下4种方式。

(1）消息长度固定，累计读取到长度总和为定长LEN 的报文后，就认为读取到了个完整的消息;将计数器置位，重新开始读取下一个数据报;

(2）将回车换行符作为消息结束符，例如FTP协议，这种方式在文本协议中应用比较广泛;

(3）将特殊的分隔符作为消息的结束标志，回车换行符就是一种特殊的结束分隔符;

(4）通过在消息头中定义长度字段来标识消息的总长度。

具体解决办法如下：

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

### 拆包和粘包代码还原

还是借助上文的 `TimeServer` 进行改造，代码如下：

#### 服务端代码：

```java
public class TimeServer {

    public void bind(int port) throws Exception {

        // 配置 Reactor 线程组
        // 接受 客户端连接
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        // 进行SocketChannel的网络读写
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            // NIO 服务端启动服务类
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChildChannelHandler());

            // 绑定端口 同步阻塞，等待绑定操作完成
            ChannelFuture f = b.bind(port).sync();

            // 等待服务端链路关闭后再结束 主函数
            f.channel().closeFuture().sync();
        } finally {
            // 退出，释放线程池资源
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }


    }

    private class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

        @Override
        protected void initChannel(SocketChannel socketChannel) throws Exception {
            socketChannel.pipeline().addLast(new TimeServerHandler());
        }

    }

    /**
     * 应该收到 100条 数据
     * 否则就是出现了 粘包问题
     * <p>
     * 服务端运行结果表明它只接收到了两条消息，第一条包含57条“QUERY TIMEORDER”指令，第二条包含了43条“QUERY TIME
     * ORDER”指令，总数正好是100条。我们期待的是收到100条消息，每条包含一条“QUERY TIME ORDER”指令。这说明发生了TCP粘包。
     */
    public static void main(String[] args) throws Exception {
        int port = 8081;
        new TimeServer().bind(port);
    }

}

```



```java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    /** 记录 接收的数据条数 */
    private int counter;

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 转换成 ByteBuf 对象
        ByteBuf buf = (ByteBuf) msg;
        // 创建 可读字节数 的数组
        byte[] req = new byte[buf.readableBytes()];
        // 将缓冲区的字节数组，复制到新的数组当中
        buf.readBytes(req);
        String body = new String(req, StandardCharsets.UTF_8)
                .substring(0, req.length - System.getProperty("line.separator").length());
        System.out.println("The time server receive order : " + body + "; the counter is :" + ++ counter);
        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ?
                new java.util.Date(System.currentTimeMillis()).toString() : "BAD ORDER";
        currentTime = currentTime + System.getProperty("line.separator");

        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        // 异步发送应答消息给 客户端（write方法只是把待发送的消息放到发送缓冲数组中，真正发送要使用 flush()）
        ctx.write(resp);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        /*
        发送队列中的消息写入到SocketChannel 中发送给对方。

        从性能角度考虑，为了防止频繁地唤醒Selector进行消息发送，Netty的 write方法并不直接将消息写入SocketChannel 中，
        调用write方法只是把待发送的消息放到发送缓冲数组中，再通过调用flush方法，
        将发送缓冲区中的消息全部写到SocketChannel 中。
         */
        ctx.flush();
    }

    /**
     * 连接异常或者退出时 执行
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

}
```



#### 客户端代码：

```java
public class TimeClient {
    public void connect(int port, String host) throws Exception {//配置客户端NIo线程组
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group).channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        /** 初始化时，将Handler设置到ChannelPipeline */
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            //发起异步连接操作，等待连接成功
            ChannelFuture f = b.connect(host, port).sync();
            //等待客户端链路关闭
            f.channel().closeFuture().sync();
        } finally {
            //优雅退出，释放NIo线程组
            group.shutdownGracefully();
        }
    }

    /**
     * 按照设计初衷，客户端应该收到100条当前系统时间的消息，但实际上只收到了一条。
     * 这不难理解，因为服务端只收到了2条请求消息，所以实际服务端只发送了2条应答，由于请求消息不满足查询条件，所以返回了2条“BAD
     * ORDER”应答消息。但是实际上客户端只收到了一条包含2条“BAD ORDER”指令的消息，说明服务端返回的应答消息也发生了粘包。
     */
    public static void main(String[] args) throws Exception {
        int port = 8081;
        new TimeClient().connect(port, "127.0.0.1");
    }
}
```



```java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    private byte[] req;
    /** 记录 发送的数据条数 */
    private int counter;

    public TimeClientHandler() {
        req = ("QUERY TIME ORDER" + System.getProperty("line.separator")).getBytes(StandardCharsets.UTF_8);
    }

    /**
     * 客户端与服务端连接后，会调用此方法
     * <p>
     * 循环发送 100 条数据，观察是否出现粘包或者拆包问题
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        ByteBuf message = null;
        for (int i = 0; i < 100; i++) {
            message = Unpooled.buffer(req.length);
            message.writeBytes(req);
            ctx.writeAndFlush(message);
        }
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, StandardCharsets.UTF_8);
        System.out.println("Now is : " + body + "; the counter is :" + ++ counter);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //释放资源
        System.out.println("Unexpected exception from downstream : " + cause.getMessage());
        ctx.close();
    }

}
```



#### **结果**

**服务端执行结果：**

```java
The time server receive order : QUERY TIME ORDER
(55行)QUERY TIME ORDER
QUERY TIME ORD; the counter is :1
TimeServerHandler:PooledUnsafeDirectByteBuf(ridx: 0, widx: 776, cap: 16384)
The time server receive order : 
(42行)QUERY TIME ORDER
```

服务端运行结果表明它只接收到了两条消息，第一条包含57条“QUERY TIMEORDER”指令，第二条包含了43条“QUERY TIME

ORDER”指令，总数正好是100条。我们期待的是收到100条消息，每条包含一条“QUERY TIME ORDER”指令。这说明发生了TCP粘包。

**客户端执行结果：**

```java
Now is : BAD ORDER
BAD ORDER
; the counter is :1
```

按照设计初衷，客户端应该收到100条当前系统时间的消息，但实际上只收到了一条。这不难理解，因为服务端只收到了2条请求消息，所以实际服务端只发送了2条应答，由于请求消息不满足查询条件，所以返回了2条“BAD ORDER”应答消息。但是实际上客户端只收到了一条包含2条“BAD ORDER”指令的消息，说明服务端返回的应答消息也发生了粘包。

### 拆包和粘包问题的解决

#### 服务端代码修改

`TimeServer` 添加 `LineBasedFrameDecoder` 和 `StringDecoder` 解码器。

![image-20220212214340540](https://cos.duktig.cn/typora/202202122143784.png)

`TimeServerHandler` 中 `msg` 直接强转成 `String` 即可。

![image-20220212214522969](https://cos.duktig.cn/typora/202202122145424.png)

#### 客户端代码修改

客户端与服务端修改类似

![image-20220212214646272](https://cos.duktig.cn/typora/202202122146026.png)



![image-20220212214703415](https://cos.duktig.cn/typora/202202122147893.png)



#### 结果测试

服务端结果：

```java
The time server receive order : QUERY TIME ORDER; the counter is :1
The time server receive order : QUERY TIME ORDER; the counter is :2
The time server receive order : QUERY TIME ORDER; the counter is :3
The time server receive order : QUERY TIME ORDER; the counter is :4
……
The time server receive order : QUERY TIME ORDER; the counter is :99
The time server receive order : QUERY TIME ORDER; the counter is :100
```

客户端结果：

```java
Now is : Sat Feb 12 21:41:38 CST 2022; the counter is :1
Now is : Sat Feb 12 21:41:38 CST 2022; the counter is :2
Now is : Sat Feb 12 21:41:38 CST 2022; the counter is :3
Now is : Sat Feb 12 21:41:38 CST 2022; the counter is :4
……
Now is : Sat Feb 12 21:41:38 CST 2022; the counter is :99
Now is : Sat Feb 12 21:41:38 CST 2022; the counter is :100
```

从结果可知，TCP 的 粘包和拆包问题已经解决。

###  LineBasedFrameDecoder和 StringDecoder的原理分析

LineBasedFrameDecoder的工作原理是它依次遍历 ByteBuf中的可读字节，判断看是否有“\n”或者“\r\n”，如果有，就以此位置为结束位置，从可读索引到结束位置区间的字节就组成了一行。它是以换行符为结束标志的解码器，支持携带结束符或者不携带结束符两种解码方式，同时支持配置单行的最大长度。如果连续读取到最大长度后仍然没有发现换行符，就会抛出异常，同时忽略掉之前读到的异常码流。
StringDecoder的功能非常简单，就是将接收到的对象转换成字符串，然后继续调用后面的Handler。

LineBasedFrameDecoder + StringDecoder组合就是按行切换的文本解码器它被设计用来支持TCP的粘包和拆包。



## 编解码技术

### Java 序列化的缺点

1. 无法跨语言
2. 序列化后的码流太大
3. 序列化性能太低

评判一个编解码框架的优劣时，往往会考虑以下几个因素。

- 是否支持跨语言，支持的语言种类是否丰富;
- 编码后的码流大小;
- 编解码的性能;
- 类库是否小巧，API使用是否方便;
- 使用者需要手工开发的工作量和难度。

### 业界主流的编解码框架

#### Facebook的 Thrift介绍（了解）

Thrift源于Facebook,在.2007年Facebook 将Thrift作为一个开源项目提交给了Apache基金会。对于当时的Facebook来说，创造Thrift 是为了解决Facebook各系统间大数据量的传输通信以及系统之间语言环境不同需要跨平台的特性，因此Thrift可以支持多种程序语言，如C++、C#、Cocoa、Erlang、Haskell、Java、Ocami、Perl、PHP、Python、Ruby和 Smalltalk。

在多种不同的语言之间通信，Thrift可以作为高性能的通信中间件使用，它支持数据(对象）序列化和多种类型的RPC服务。Thrift适用于静态的数据交换，需要先确定好它的数据结构，当数据结构发生变化时，必须重新编辑IDL文件，生成代码和编译，这一点跟其他IDL工具相比可以视为是 Thrift的弱项。Thrift 适用于搭建大型数据交换及存储的通用工具，对于大型系统中的内部数据传输，相对于JSON和 XML在性能和传输大小上都有明显的优势。

Thrift支持三种比较典型的编解码方式。

1. 通用的二进制编解码;
2. 压缩二进制编解码;
3. 优化的可选字段压缩编解码。

![image-20220212224839043](https://cos.duktig.cn/typora/202202122248398.png)

#### Google的Protobuf介绍

Protobuf全称Google Protocol Buffers，它由谷歌开源而来，在谷歌内部久经考验。它将数据结构以.proto 文件进行描述，通过代码生成工具可以生成对应数据结构的POJO对象和 Protobuf相关的方法和属性。

它的特点如下：

(1）在谷歌内部长期使用，产品成熟度高;

(2）跨语言、支持多种语言，包括C++、Java和 Python;

(3）编码后的消息更小，更加有利于存储和传输;

(4）编解码的性能非常高;

(5）支持不同协议版本的前向兼容;

(6）支持定义可选和必选字段。

下面我们看下Protobuf编解码和其他几种序列化框架的性能对比数据：

![image-20220212224433525](https://cos.duktig.cn/typora/202202122244060.png)



![image-20220212224442100](https://cos.duktig.cn/typora/202202122244324.png)



### Google的Protobuf 详解

#### 下载

下载：[https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf)

选择对应的平台：

![image-20220214093943031](https://cos.duktig.cn/typora/202202140940179.png)

#### 编写proto文件

```
syntax = "proto3";// 指定protobuf版本
option java_package = "cn.duktig.netty.protobuf.protobuf";// 指定包名
option java_outer_classname = "MessageProtobuf";// 指定生成的类名

message Msg {
  Head head = 1;// 消息头
  string body = 2;// 消息体
}

message Head {
  string msgId = 1;// 消息id
  int32 msgType = 2;// 消息类型
  int32 msgContentType = 3;// 消息内容类型
  string fromId = 4;// 消息发送者id
  string toId = 5;// 消息接收者id
  int64 timestamp = 6;// 消息时间戳
  int32 statusReport = 7;// 状态报告
  string extend = 8;// 扩展字段，以key/value形式存放的json
}

```

#### 代码生成

```sh
protoc.exe -I=proto的输入目录 --java_out=java类输出目录 proto的输入目录包括包括proto文件
```

注意：proto文件中的 `java_package 包名` 不需要包含在 **输出目录中**（否则目录会有重复）。

```
protoc.exe -I=D:\IDE\code\learn-example\netty\src\main\java\cn\duktig\netty\protobuf\serverexample\protobuf  --java_out=D:\IDE\code\learn-example\netty\src\main\java\ SubscribeReq.proto

protoc.exe -I=D:\IDE\code\learn-example\netty\src\main\java\cn\duktig\netty\protobuf\serverexample\protobuf  --java_out=D:\IDE\code\learn-example\netty\src\main\java\ SubscribeResp.proto
```

<img src="https://cos.duktig.cn/typora/202202141149256.png" alt="image-20220214114934810" style="zoom:67%;" />

#### 测试

```java
public class SubscribeReqProtoTest {

    /**
     * photobuf 编码
     */
    private static byte[] encode(SubscribeReqProto.SubscribeReq req) {
        return req.toByteArray();
    }

    /**
     * photobuf 解码
     */
    private static SubscribeReqProto.SubscribeReq decode(byte[] body) throws InvalidProtocolBufferException {
        return SubscribeReqProto.SubscribeReq.parseFrom(body);
    }

    private static SubscribeReqProto.SubscribeReq createSubscribeReq() {
        SubscribeReqProto.SubscribeReq.Builder builder = SubscribeReqProto.SubscribeReq.newBuilder();
        builder.setSubReqID(1);
        builder.setUserName("Lilinfeng");
        builder.setProductName("Netty Book ");
        builder.setAddress("NanJing YuHuaTai");
        return builder.build();
    }

    public static void main(String[] args) throws InvalidProtocolBufferException {
        SubscribeReqProto.SubscribeReq req = createSubscribeReq();
        System.out.println("Before encode : " + req.toString());

        SubscribeReqProto.SubscribeReq req2 = decode(encode(req));
        System.out.println("After decode : " + req.toString());

        System.out.println("Assert equal : --> " + req2.equals(req));
    }

}
```

结果：

```java
Before encode : subReqID: 1
userName: "Lilinfeng"
productName: "Netty Book "
address: "NanJing YuHuaTai"

After decode : subReqID: 1
userName: "Lilinfeng"
productName: "Netty Book "
address: "NanJing YuHuaTai"

Assert equal : --> true

Process finished with exit code 0
```



#### Netty整合Protobuf编解码器 实例

##### 服务端

```java
public class SubReqServer {

    public void bind(int port) throws Exception {

        // 配置 Reactor 线程组
        // 接受 客户端连接
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        // 进行SocketChannel的网络读写
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            // NIO 服务端启动服务类
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 100)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(
                            // 5. channel 代表和客户端进行数据读写的通道 Initializer 初始化，负责添加别的 handler
                            new ChannelInitializer<NioSocketChannel>() {
                                @Override
                                protected void initChannel(NioSocketChannel ch) throws Exception {
                                    // 6. 添加具体 handler
                                    // 用于半包处理
                                    ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());
                                    // Protobuf 解码器
                                    ch.pipeline().addLast(new ProtobufDecoder(SubscribeReqProto.SubscribeReq.getDefaultInstance())); // 将 ByteBuf 转换为字符串
                                    ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
                                    // Protobuf 编码器，不需要手动编码
                                    ch.pipeline().addLast(new ProtobufEncoder());
                                    ch.pipeline().addLast(new SubReqServerHandler());
                                }
                            })
            ;

            // 绑定端口 同步阻塞，等待绑定操作完成
            ChannelFuture f = b.bind(port).sync();

            // 等待服务端链路关闭后再结束 主函数
            f.channel().closeFuture().sync();
        } finally {
            // 退出，释放线程池资源
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 8081;
        new TimeServer().bind(port);
    }
}
```



```java
public class SubReqServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        SubscribeReqProto.SubscribeReq req = (SubscribeReqProto.SubscribeReq) msg;
        if ("Lilinfeng".equalsIgnoreCase(req.getUserName())) {
            System.out.println("Service accept client subscribe req : [ " + req.toString() + "]");
            ctx.writeAndFlush(this.resp(req.getSubReqID()));
        }
    }

    private SubscribeRespProto.SubscribeResp resp(int subReqId) {
        SubscribeRespProto.SubscribeResp.Builder builder = SubscribeRespProto.SubscribeResp.newBuilder();
        builder.setSubReqId(subReqId);
        builder.setRespCode(0);
        builder.setDesc("handle success!");
        return builder.build();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //释放资源
        System.out.println("Unexpected exception from downstream : " + cause.getMessage());
        ctx.close();
    }

}
```



##### 客户端

```java
public class SubReqClient {

    public void connect(int port, String host) throws Exception {//配置客户端NIo线程组
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group).channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        /** 初始化时，将Handler设置到ChannelPipeline */
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            // 用于半包处理
                            ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());
                            // Protobuf 解码器
                            ch.pipeline().addLast(new ProtobufDecoder(SubscribeRespProto.SubscribeResp.getDefaultInstance())); // 将 ByteBuf 转换为字符串
                            ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
                            // Protobuf 编码器，不需要手动编码
                            ch.pipeline().addLast(new ProtobufEncoder());
                            ch.pipeline().addLast(new SubReqClientHandler());
                        }
                    });
            //发起异步连接操作，等待连接成功
            ChannelFuture f = b.connect(host, port).sync();
            //等待客户端链路关闭
            f.channel().closeFuture().sync();
        } finally {
            //优雅退出，释放NIo线程组
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 8081;
        new TimeClient().connect(port, "127.0.0.1");
    }

}
```



```java
public class SubReqClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        for (int i = 0; i < 10; i++) {
            ctx.write(this.subReq(i));
        }
        ctx.flush();
    }

    private SubscribeReqProto.SubscribeReq subReq(int i) {
        SubscribeReqProto.SubscribeReq.Builder builder = SubscribeReqProto.SubscribeReq.newBuilder();
        builder.setSubReqID(1);
        builder.setUserName("Lilinfeng");
        builder.setProductName("Netty Book ");
        builder.setAddress("NanJing YuHuaTai");
        return builder.build();
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("Receive server response : [" + msg + "]");
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        /*
        发送队列中的消息写入到SocketChannel 中发送给对方。

        从性能角度考虑，为了防止频繁地唤醒Selector进行消息发送，Netty的 write方法并不直接将消息写入SocketChannel 中，
        调用write方法只是把待发送的消息放到发送缓冲数组中，再通过调用flush方法，
        将发送缓冲区中的消息全部写到SocketChannel 中。
         */
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //释放资源
        System.out.println("Unexpected exception from downstream : " + cause.getMessage());
        ctx.close();
    }

}
```



##### 结果

服务端输出：

```java
The time server receive order : QUERY TIME ORDER; the counter is :1
The time server receive order : QUERY TIME ORDER; the counter is :2
The time server receive order : QUERY TIME ORDER; the counter is :3
The time server receive order : QUERY TIME ORDER; the counter is :4
^
The time server receive order : QUERY TIME ORDER; the counter is :99
The time server receive order : QUERY TIME ORDER; the counter is :100
```

客户端输出：

```java
Now is : Mon Feb 14 11:45:13 CST 2022; the counter is :1
Now is : Mon Feb 14 11:45:13 CST 2022; the counter is :2
Now is : Mon Feb 14 11:45:13 CST 2022; the counter is :3
Now is : Mon Feb 14 11:45:13 CST 2022; the counter is :4
^
Now is : Mon Feb 14 11:45:13 CST 2022; the counter is :99
Now is : Mon Feb 14 11:45:13 CST 2022; the counter is :100
```



## Netty的协议栈

### 协议栈功能

Netly 协议栈承载了业务内部各模块之间的消息交互和服务调用，它的主要功能如下：

(1）基于Netty 的NIO通信框架，提供高性能的异步通信能力;

(2）提供消息的编解码框架，可以实现POJO的序列化和反序列化;

(3）提供基于IP地址的白名单接入认证机制;

(4）链路的有效性校验机制;

(5）链路的断连重连机制。



### 通信模型

![image-20220216111912571](https://cos.duktig.cn/typora/202202161119789.png)

具体步骤如下：

( 1) Netty协议栈客户端发送握手请求消息，携带节点ID等有效身份认证信息;

( 2) Netty协议栈服务端对握手请求消息进行合法性校验，包括节点ID有效性校验、节点重复登录校验和IP地址合法性校验，校验通过后，返回登录成功的握手应答消息;

(3）链路建立成功之后，客户端发送业务消息;

(4）链路成功之后，服务端发送心跳消息;

(5）链路建立成功之后，客户端发送心跳消息;

(6) 链路建立成功之后，服务端发送业务消息;

(7）服务端退出时，服务端关闭连接，客户端感知对方关闭连接后，被动关闭客户端连接。

双方之间的心跳采用Ping-Pong 机制，当链路处于空闲状态时，客户端主动发送Ping消息给服务端,服务端接收到Ping消息后发送应答消息Pong给客户端，如果客户端连续发送N条 Ping消息都没有接收到服务端返回的Pong消息,说明链路已经挂死或者对方处于异常状态，客户端主动关闭连接，间隔周期T后发起重连操作，直到重连成功。



### 可靠性设计

Netty协议栈可能会运行在非常恶劣的网络环境中，**网络超时、闪断、对方进程僵死或者处理缓慢等情况都有可能发生**。为了保证在这些极端异常场景下Netty协议栈仍能够正常工作或者自动恢复，需要对它的可靠性进行统一规划和设计。

#### 心跳机制

在凌晨等业务低谷期时段，如果发生网络闪断、连接被Hang 住等网络问题时，由于没有业务消息，应用进程很难发现。到了白天业务高峰期时，会发生大量的网络通信失败，严重的会导致一段时间进程内无法处理业务消息。为了解决这个问题，**在网络空闲时采用心跳机制来检测链路的互通性，一旦发现网络故障，立即关闭链路主动重连**。

具体的设计思路如下：

(1）当网络处于空闲状态持续时间达到T(连续周期T没有读写消息）时，客户端主动发送Ping心跳消息给服务端。

(2）如果在下一个周期T到来时客户端没有收到对方发送的Pong 心跳应答消息或者读取到服务端发送的其他业务消息，则心跳失败计数器加1。

(3）每当客户端接收到服务的业务消息或者Pong应答消息时，将心跳失败计数器清零;连续N次没有接收到服务端的Pong消息或者业务消息，则关闭链路，间隔INTERVAL时间后发起重连操作。

(4）服务端网络空闲状态持续时间达到T后，服务端将心跳失败计数器加1;只要接收到客户端发送的Ping 消息或暑其他业务消息，计数器清零。

(5）服务端连续N次没有接收到客户端的Ping消息或者其他业务消息，则关闭链路，释放资源，等待客户端重连。

通过 **Ping-Pong双向心跳机制**，可以保证无论通信哪一方出现网络故障，都能被及时地检测出来。**为了防止由于对方短时间内繁忙没有及时返回应答造成的误判，只有连续N次心跳检测都失败才认定链路已经损害，需要关闭链路并重建链路**。

当读或者写心跳消息发生I/O异常的时候，说明链路已经中断,此时需要立即关闭链路，**如果是客户端，需要重新发起连接**。**如果是服务端，需要清空缓存的半包信息，等待客户端重连**。

#### 重连机制

**如果链路中断，等待INTERVAL时间后，由客户端发起重连操作，如果重连失败，间隔周期INTERVAL后再次发起重连,直到重连成功。**

为了保证服务端能够有充足的时间释放句柄资源，在首次断连时客户端需要等待INTERVAL时间之后再发起重连，而不是失败后就立即重连。

为了保证句柄资源能够及时释放，无论什么场景下的重连失败，客户端都必须保证自身的资源被及时释放，包括但不限于SocketChannel、Socket等。重连失败后，需要打印异常堆栈信息，方便后续的问题定位。

#### 重复登录保护

**当客户端握手成功之后，在链路处于正常状态下，不允许客户端重复登录，以防止客户端在异常状态下反复重连导致句柄资源被耗尽**。

服务端接收到客户端的握手请求消息之后，首先对IP地址进行合法性检验，**如果校验成功，在缓存的地址表中查看客户端是否已经登录，如果已经登录，则拒绝重复登录,返回错误码-1，同时关闭TCP链路，并在服务端的日志中打印握手失败的原因**。

客户端接收到握手失败的应答消息之后，关闭客户端的TCP连接，等待INTERVAL时间之后，再次发起TCP连接，直到认证成功。

为了防止由服务端和客户端对链路状态理解不一致导致的客户端无法握手成功的问题，**当服务端连续N次心跳超时之后需要主动关闭链路，清空该客户端的地址缓存信息，以保证后续该客户端可以重连成功，防止被重复登录保护机制拒绝掉**。

#### 消息缓存重发

无论客户端还是服务端，当发生链路中断之后，在链路恢复之前，缓存在消息队列中待发送的消息不能丢失，等链路恢复之后，重新发送这些消息，保证链路中断期间消息不丢失。

**考虑到内存溢出的风险，建议消息缓存队列设置上限，当达到上限之后，应该拒绝继续向该队列添加新的消息**。

### 安全性设计

为了保证整个集群环境的安全，内部长连接采用基于IP地址的安全认证机制，服务端对握手请求消息的IP地址进行合法性校验:如果在白名单之内，则校验通过;否则，拒绝对方连接。

如果将Netty协议栈放到公网中使用，需要采用更加严格的安全认证机制，例如 **基于密钥和AES 加密的用户名+密码认证机制**，也可以采用 **SSL/TSL安全传输**。

## Netty服务端、客户端分析

### Netty服务端创建时序图

![Netty服务端创建时序图](https://cos.duktig.cn/typora/202202161141026.png)

### Netty客户端创建时序图

![Netty客户端创建时序图](https://cos.duktig.cn/typora/202202161507707.png)

步骤1:用户线程创建Bootstrap实例，通过API设置创建客户端相关的参数，异步发起客户端连接。

步骤2:创建处理客户端连接、IO读写的Reactor线程组NioEventLoopGroup。可以通过构造函数指定IO线程的个数，默认为CPU内核数的2倍;

步骤3:通过Bootstrap的ChannelFactory和用户指定的Channel类型创建用于客户端连接的NioSocketChannel，它的功能类似于JDK NIO类库提供的SocketChannel;

步骤4:创建默认的Channel Handler Pipeline，用于调度和执行网络事件;

步骤5:异步发起TCP连接,判断连接是否成功。如果成功,则直接将NioSocketChannel注册到多路复用器上，监听读操作位，用于数据报读取和消息发送;如果没有立即连接成功，则注册连接监听位到多路复用器，等待连接结果;

步骤6:注册对应的网络监听状态位到多路复用器;

步骤7:由多路复用器在I/O现场中轮询各Channel，处理连接结果;

步骤8:如果连接成功，设置Future结果，发送连接成功事件，触发ChannelPipeline执行;

步骤9:由ChannelPipeline调度执行系统和用户的ChannelHandler，执行业务逻辑。

## Netty架构

### 架构解析

Netty采用了典型的 **三层网络架构** 进行设计和开发，逻辑架构如图：

![Netty架构图](https://cos.duktig.cn/typora/202202162040823.png)

#### Reactor通信调度层

它由一系列辅助类完成,包括Reactor线程NioEventLoop及其父类,NioSocketChannel/NioServerSocketChannel及其父类，ByteBuffer以及由其衍生出来的各种 Buffer，Unsafe以及其衍生出的各种内部类等。

该层的主要职责就是监听网络的读写和连接操作，负责将网络层的数据读取到内存缓冲区中，然后触发各种网络事件，例如连接创建、连接激活、读事件、写事件等，将这些事件触发到PipeLine 中，由 PipeLine管理的职责链来进行后续的处理。

#### 职责链ChannelPipeline

它负责事件在职责链中的有序传播，同时负责动态地编排职责链。

职责链可以选择监听和处理自己关心的事件，它可以拦截处理和向后/向前传播事件。

不同应用的Handler节点的功能也不同，通常情况下，往往会开发编解码Hanlder用于消息的编解码，它可以将外部的协议消息转换成内部的POJO对象,这样上层业务则只需要关心处理业务逻辑即可不需要感知底层的协议差异和线程模型差异，实现了架构层面的分层隔离。

#### 业务逻辑编排层( Service ChannelHandler)

业务逻辑编排层通常有两类:一类是纯粹的业务逻辑编排，还有一类是其他的应用层协议插件，用于特定协议相关的会话和链路管理。例如CMPP协议，用于管理和中国移动短信系统的对接。

情况下，对于业务开发者，只_需要关心职责链的拦截和业务Handler 的编排。因为应用层协议栈往往是开发一次，到处运行，所以实际上对于业务开发者来说，只需要关心服务层的业务逻辑开发即可。

### Netty的架构设计是如何实现高性能的

(1）采用异步非阻塞的IO类库，基于 Reactor 模式实现，**解决了传统同步阻塞I/o模式下一个服务端无法平滑地处理线性增长的客户端的问题**。

(2)**TCP接收和发送缓冲区使用直接内存代替堆内存，避免了内存复制，提升了IO读取和写入的性能**。

(3）**支持通过内存池的方式循环利用ByteBuf，避免了频繁创建和销毁ByteBuf带来的性能损耗**。

(4）**可配置的IO线程数、TCP参数等**，为不同的用户场景提供定制化的调优参数，满足不同的性能场景。

(5）**采用环形数组缓冲区实现无锁化并发编程，代替传统的线程安全容器或者锁**

(6）合理地使用线程安全容器、原子类等，提升系统的并发处理能力。

(7）关键资源的处理使用单线程串行化的方式，避免多线程并发访问带来的锁竞争和额外的CPU 资源消耗问题。

(8）通过引用计数器及时地申请释放不再被引用的对象，细粒度的内存管理降低了GC的频率，减少了频繁GC带来的时延增大和 CPU损耗。

### 可靠性设计

#### 链路有效性检测

为了保证长连接的链路有效性，往往需要通过心跳机制周期性地进行链路检测。

当有业务消息时，无须心跳检测，可以由业务消息进行链路可用性检测。所以**心跳消息往往是在链路空闲时发送的**。

为了支持心跳，Netty提供了如下两种链路空闲检测机制：

1. **读空闲超时机制**:当连续周期T没有消息可读时，触发超时 Handler，用户可以基于读空闲超时发送心跳消息，进行链路检测;如果连续N个周期仍然没有读取到心跳消息，可以主动关闭链路。
2. **写空闲超时机制**:当连续周期T没有消息要发送时，触发超时Handler，用户可以基于写空闲超时发送心跳消息，进行链路检测;如果连续N个周期仍然没有接收到对方的心跳消息,可以主动关闭链路。

为了满足不同用户场景的心跳定制，Netty提供了空闲状态检测事件通知机制，用户可以订阅空闲超时事件、写空闲超时事件、读或者写超时事件，在接收到对应的空闲事件之后，灵活地进行定制。

#### 内存保护机制

Netty提供多种机制对内存进行保护，包括以下几个方面：

- 通过对象引用计数器对Netty 的 ByteBuf等内置对象进行细粒度的内存申请和释放，对非法的对象引用进行检测和保护。
- 通过内存池来重用ByteBuf，节省内存。
- 可设置的内存容量上限，包括 ByteBuf、线程池线程数等。

#### 优雅退出

指的是当系统退出时，JVM通过注册的Shutdown Hook 拦截到退出信号量，然后执行退出操作，释放相关模块的资源占用，将缓冲区的消息处理完成或者清空，将待刷新的数据持久化到磁盘或者数据库中，等到资源回收和缓冲区消息处理完成之后，再退出。

优雅停机往往需要设置个最大超时时间T，如果达到T后系统仍然没有退出，则通过 `Kill - 9 pid` 强杀当前的进程。





