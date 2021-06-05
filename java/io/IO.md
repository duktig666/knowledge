## IO流

#### 25.1流的分类

1. 操作数据单位：字节流、字符流
2. 数据的流向：输入流、输出流
3. 流的角色：节点流、处理流

##### 按操作方式分类结构图：

![image-20200616203751649](https://gitee.com/koala010/typora/raw/master/img/20200726214205.png)

##### 按操作对象分类结构图：

![image-20200616203828060](https://gitee.com/koala010/typora/raw/master/img/20200726214208.png)



#### 25.2 流的体系结构

| 抽象基类     | 节点流（或文件流）                            | 缓冲流（处理流的一种）                                     |
| ------------ | --------------------------------------------- | ---------------------------------------------------------- |
| InputStream  | FileInputStream   (read(byte[] buffer))       | BufferedInputStream (read(byte[] buffer))                  |
| OutputStream | FileOutputStream  (write(byte[] buffer,0,len) | BufferedOutputStream (write(byte[] buffer,0,len) / flush() |
| Reader       | FileReader (read(char[] cbuf))                | BufferedReader (read(char[] cbuf) / readLine())            |
| Writer       | FileWriter (write(char[] cbuf,0,len)          | BufferedWriter (write(char[] cbuf,0,len) / flush()         |

#### 25.3 IO原理

- I/O是Input/Output的缩写， I/O技术是非常实用的技术， 用于处理设备之间的数据传输。 如读/写文件，网络通讯等。
- Java程序中，对于数据的输入/输出操作以“流(stream)” 的方式进行。
- java.io包下提供了各种“流”类和接口，用以获取不同种类的数据，并通过标准的方法输入或输出数据。
- 输入input： 读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中。
- 输出output： 将程序（内存）数据输出到磁盘、光盘等存储设备中。



#### 25.4 节点流和处理流

> 节点流：直接从数据源或目的地读写数据

<img src="https://gitee.com/koala010/typora/raw/master/img/20200726214214.png" alt="image-20200616210233212" style="zoom:67%;" />

> 处理流：不直接连接到数据源或目的地，而是“连接” 在已存在的流（节点流或处理流）之上，通过对数据的处理为程序提供更为强大的读写功能。

<img src="https://gitee.com/koala010/typora/raw/master/img/20200726214217.png" alt="image-20200616210242248" style="zoom:67%;" />

##### 节点流(或文件流)：注意点

- 定义文件路径时，注意：可以用“/”或者“\\”。
- 在写入一个文件时，如果使用构造器FileOutputStream(file)，则目录下有同名文件将被覆盖。
- 如果使用构造器FileOutputStream(file,true)，则目录下的同名文件不会被覆盖，在文件内容末尾追加内容。
- 在读取文件时，必须保证该文件已存在，否则报异常。
- 字节流操作字节，比如： .mp3， .avi， .rmvb， mp4， .jpg， .doc， .ppt
- 字符流操作字符，只能操作普通文本文件。 最常见的文本文件： .txt， .java， .c， .cpp 等语言的源代码。尤其注意.doc,excel,ppt这些不是文本文件。

#### 25.5 处理流

##### 25.5.1.缓冲流

为了提高数据读写的速度， Java API提供了带缓冲功能的流类，在使用这些流类时，会创建一个内部缓冲区数组，**缺省使用8192个字节(8Kb)的缓冲区**。

<img src="https://gitee.com/koala010/typora/raw/master/img/20200726214221.png" alt="image-20200616210512913" style="zoom: 50%;" />

**缓冲流注意事项**

- 当读取数据时，数据按块读入缓冲区，其后的读操作则直接访问缓冲区
- 当使用BufferedInputStream读取字节文件时， BufferedInputStream会一次性从文件中读取8192个(8Kb)， 存在缓冲区中， 直到缓冲区装满了， 才重新从文件中读取下一个8192个字节数组。
- 向流中写入字节时， 不会直接写到文件， 先写到缓冲区中直到缓冲区写满，BufferedOutputStream才会把缓冲区中的数据一次性写到文件里。使用方法flush()可以强制将缓冲区的内容全部写入输出流
- 关闭流的顺序和打开流的顺序相反。只要关闭最外层流即可， 关闭最外层流也会相应关闭内层节点流
- flush()方法的使用：手动将buffer中内容写入文件
- 如果是带缓冲区的流对象的close()方法， 不但会关闭流， 还会在关闭流之前刷新缓冲区， 关闭后不能再写出

<img src="https://gitee.com/koala010/typora/raw/master/img/20200726214224.png" alt="image-20200616210638626" style="zoom: 50%;" />



##### 25.5.2.转换流

- 转换流提供了在字节流和字符流之间的转换
- Java API提供了两个转换流：
  - InputStreamReader：将InputStream转换为Reader
  - OutputStreamWriter：将Writer转换为OutputStream
- 字节流中的数据都是字符时，转成字符流操作更高效。
- 很多时候我们使用转换流来处理文件乱码问题。实现编码和
  解码的功能。



<img src="https://gitee.com/koala010/typora/raw/master/img/20200726214231.png" alt="image-20200616210822816" style="zoom: 33%;" />



##### 25.5.3.标准输入、输出流

- System.in和System.out分别代表了系统标准的输入和输出设备
- 默认输入设备是： 键盘， 输出设备是：显示器
- System.in的类型是InputStream
- System.out的类型是PrintStream，其是OutputStream的子类FilterOutputStream 的子类
- 重定向：通过System类的setIn， setOut方法对默认设备进行改变。
  - public static void setIn(InputStream in)
  - public static void setOut(PrintStream out)

##### 25.5.4.打印流

实现将基本数据类型的数据格式转化为字符串输出

打印流： PrintStream和PrintWriter

- 提供了一系列重载的print()和println()方法，用于多种数据类型的输出
- PrintStream和PrintWriter的输出不会抛出IOException异常
- PrintStream和PrintWriter有自动flush功能
- PrintStream 打印的所有字符都使用平台的默认字符编码转换为字节。在需要写入字符而不是写入字节的情况下，应该使用 PrintWriter 类。
- System.out返回的是PrintStream的实例

##### 25.5.5.数据流

- 为了方便地操作Java语言的基本数据类型和String的数据，可以使用数据流。
- 数据流有两个类： (用于读取和写出基本数据类型、 String类的数据）
  - DataInputStream 和 DataOutputStream
  - 分别“套接”在 InputStream 和 OutputStream 子类的流上

##### 25.5.6.对象流

- ObjectInputStream和OjbectOutputSteam
  - 用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。
- 序列化： 用ObjectOutputStream类保存基本类型数据或对象的机制
- 反序列化： 用ObjectInputStream类读取基本类型数据或对象的机制
- ObjectOutputStream和ObjectInputStream不能序列化static和transient修饰的成员变量

#### 25.6 既然有了字节流,为什么还要有字符流?

问题本质想问：**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**

字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

#### 25.7 BIO,NIO,AIO 有什么区别?

> BIO (Blocking I/O): **同步阻塞 I/O 模式**，数据的读取写⼊必须阻塞在⼀个线程内等待其完成。在活动连接数不是特别⾼（⼩于单机 1000）的情况下，这种模型是⽐较不错的，可以让每⼀个连接专注于⾃⼰的 I/O 并且编程模型简单，也不⽤过多考虑系统的过载、限流等问题。线程池本身就是⼀个天然的漏⽃，可以缓冲⼀些系统处理不了的连接或请求。但是，当⾯对⼗万甚⾄百万级连接的时候，传统的 BIO 模型是⽆能为⼒的。因此，我们需要⼀种更⾼效的 I/O 处理模型来应对更⾼的并发量。



> NIO (Non-blocking/New I/O): NIO 是⼀种**同步⾮阻塞的 I/O 模型**，在 Java 1.4 中引⼊了 NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer 等抽象。NIO 中的 N 可以理解为 Non-blocking，不单纯是 New。它⽀持⾯向缓冲的，基于通道的 I/O 操作⽅法。 **NIO 提供了与传统 BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都⽀持阻塞和⾮阻塞两种模式**。阻塞模式使⽤就像传统中的⽀持⼀样，⽐较简单，但是性能和可靠性都不好；⾮阻塞模式正好与之相反。对于低负载、低并发的应⽤程序，可以使⽤同步阻塞 I/O 来提升开发速率和更好的维护性；对于⾼负载、⾼并发的（⽹络）应⽤，应使⽤ NIO 的⾮阻塞模式来开发。



> AIO (Asynchronous I/O): AIO 也就是 NIO 2。在 Java 7 中引⼊了 NIO 的改进版 NIO 2,它是**异步⾮阻塞的 IO 模型**。异步 IO 是**基于事件和回调机制**实现的，也就是应⽤操作之后会直接返回，不会堵塞在那⾥，当后台处理完成，操作系统会通知相应的线程进⾏后续的操作。AIO 是异步 IO 的缩写，虽然 NIO 在⽹络操作中，提供了⾮阻塞的⽅法，但是 NIO 的 IO ⾏为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程⾃⾏进⾏ IO 操作，IO 操作本身是同步的。查阅⽹上相关资料，我发现就⽬前来说 AIO 的应⽤还不是很⼴泛，Netty 之前也尝试使⽤过 AIO，不过⼜放弃了。