# IO流

## 1.流的分类

1. 操作数据单位：字节流、字符流
2. 数据的流向：输入流、输出流
3. 流的角色：节点流、处理流

Java Io 流共涉及 40 多个类，这些类看上去很杂乱，但实际上很有规则，⽽且彼此之间存在⾮常紧密的联系， Java I0 流的 40 多个类都是从如下 4 个抽象类基类中派⽣出来的。

InputStream/Reader: 所有的输⼊流的基类，前者是字节输⼊流，后者是字符输⼊流。
OutputStream/Writer: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

按操作方式分类结构图：

![image-20200616203751649](https://gitee.com/koala010/typora/raw/master/img/20200726214205.png)

按操作对象分类结构图：

![image-20200616203828060](https://gitee.com/koala010/typora/raw/master/img/20200726214208.png)



| 抽象基类     | 节点流（或文件流）                            | 缓冲流（处理流的一种）                                     |
| ------------ | --------------------------------------------- | ---------------------------------------------------------- |
| InputStream  | FileInputStream   (read(byte[] buffer))       | BufferedInputStream (read(byte[] buffer))                  |
| OutputStream | FileOutputStream  (write(byte[] buffer,0,len) | BufferedOutputStream (write(byte[] buffer,0,len) / flush() |
| Reader       | FileReader (read(char[] cbuf))                | BufferedReader (read(char[] cbuf) / readLine())            |
| Writer       | FileWriter (write(char[] cbuf,0,len)          | BufferedWriter (write(char[] cbuf,0,len) / flush()         |



## 2.既然有了字节流,为什么还要有字符流?

问题本质想问：**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**

字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

## 3.BIO,NIO,AIO 有什么区别?

> BIO (Blocking I/O): **同步阻塞 I/O 模式**，数据的读取写⼊必须阻塞在⼀个线程内等待其完成。在活动连接数不是特别⾼（⼩于单机 1000）的情况下，这种模型是⽐较不错的，可以让每⼀个连接专注于⾃⼰的 I/O 并且编程模型简单，也不⽤过多考虑系统的过载、限流等问题。线程池本身就是⼀个天然的漏⽃，可以缓冲⼀些系统处理不了的连接或请求。但是，当⾯对⼗万甚⾄百万级连接的时候，传统的 BIO 模型是⽆能为⼒的。因此，我们需要⼀种更⾼效的 I/O 处理模型来应对更⾼的并发量。



> NIO (Non-blocking/New I/O): NIO 是⼀种**同步⾮阻塞的 I/O 模型**，在 Java 1.4 中引⼊了 NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer 等抽象。NIO 中的 N 可以理解为 Non-blocking，不单纯是 New。它⽀持⾯向缓冲的，基于通道的 I/O 操作⽅法。 **NIO 提供了与传统 BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都⽀持阻塞和⾮阻塞两种模式**。阻塞模式使⽤就像传统中的⽀持⼀样，⽐较简单，但是性能和可靠性都不好；⾮阻塞模式正好与之相反。对于低负载、低并发的应⽤程序，可以使⽤同步阻塞 I/O 来提升开发速率和更好的维护性；对于⾼负载、⾼并发的（⽹络）应⽤，应使⽤ NIO 的⾮阻塞模式来开发。



> AIO (Asynchronous I/O): AIO 也就是 NIO 2。在 Java 7 中引⼊了 NIO 的改进版 NIO 2,它是**异步⾮阻塞的 IO 模型**。异步 IO 是**基于事件和回调机制**实现的，也就是应⽤操作之后会直接返回，不会堵塞在那⾥，当后台处理完成，操作系统会通知相应的线程进⾏后续的操作。AIO 是异步 IO 的缩写，虽然 NIO 在⽹络操作中，提供了⾮阻塞的⽅法，但是 NIO 的 IO ⾏为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程⾃⾏进⾏ IO 操作，IO 操作本身是同步的。查阅⽹上相关资料，我发现就⽬前来说 AIO 的应⽤还不是很⼴泛，Netty 之前也尝试使⽤过 AIO，不过⼜放弃了。