> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 本篇文章源码参看：[https://github.com/duktig666/big-data](https://github.com/duktig666/big-data)

# HDFS

## 概述

### HDFS 产生背景 

随着数据量越来越大，在一个操作系统存不下所有的数据，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，迫切**需要一种系统来管理多台机器上的文件**，这就是分布式文件管理系统。**HDFS 只是分布式文件管理系统中的一种**。 

### HDFS 定义

HDFS（Hadoop Distributed File System），它是一个文件系统，用于存储文件，通过目录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。 
**HDFS 的使用场景**：**适合一次写入，多次读出的场景**。一个文件经过创建、写入和关闭之后就不需要改变。 

### HDFS的优缺点

#### 优点

- **高容错性**
  - 数据自动保存多个副本。它通过增加副本的形式，提高容错性。
  - 某一个副本丢失以后，它可以自动恢复。
- **适合处理大数据**
  - 数据规模：能够处理数据规模达到GB、TB、甚至**PB级别的数据**；
  - 文件规模：能够处理百万规模以上的文件数量，数量相当之大。
- 可**构建在廉价机器上**，通过多副本机制，提高可靠性。

#### 缺点

- **不适合低延时数据访问**，比如毫秒级的存储数据，是做不到的。
- **无法高效的对大量小文件进行存储**
  - 存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。这样是不可取的，因为NameNode的内存总是有限的；
  - 小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。
- **不支持并发写入、文件随机修改**
  - 一个文件只能有一个写，不允许多个线程同时写
  - 仅支持数据append（追加），不支持文件的随机修改

### HDFS组成架构

![HDFS组成架构](https://cos.duktig.cn/typora/202110062048937.png)

**NameNode（nn）：就是Master，它是一个主管、管理者。**

- 管理HDFS的名称空间；
- 配置副本策略；
- 管理数据块（Block）映射信息；
- 处理客户端读写请求。

**DataNode：就是Slave。NameNode下达命令，DataNode执行实际的操作。**

- 存储实际的数据块；
- 执行数据块的读/写操作。

**Client：就是客户端**

- 文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传；
- 与NameNode交互，获取文件的位置信息；
- 与DataNode交互，读取或者写入数据；
- Client提供一些命令来管理HDFS，比如NameNode格式化；
- Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作；

**Secondary NameNode：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。**

- 辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode ；
- 在紧急情况下，可辅助恢复NameNode。

### HDFS 文件块大小（面试重点）

HDFS 中的文件在物理上是分块存储（Block ），块的大小可以通过配置参数
( dfs.blocksize）来规定，**默认大小在Hadoop2.x/3.x版本中是128M，1.x版本中是64M**。

- 如果寻址时间约为10ms，即查找到目标block的时间为10ms
- **寻址时间为传输时间的1%时，则为最佳状态。（专家）**因此，传输时间=10ms/0.01=1000ms=1s
- 而目前磁盘的传输速率普遍为100MB/s
- block大小=1s*100MB/s=100MB

#### 思考：为什么块的大小不能设置太小，也不能设置太大？

（1）HDFS的块设置**太小**，**会增加寻址时间**，程序一直在找块的开始位置；

（2）如果块设置的**太大**，从**磁盘传输数据的时间**会明显**大于定位这个块开**
**始位置所需的时间**。导致程序在处理这块数据时，会非常慢。

总结：**HDFS块的大小设置主要取决于磁盘传输速率**。

*机械硬盘一般设置128M，固态硬盘一般设置256M。*



## HDFS的Shell相关操作（开发重点）

*`hadoop fs` 具体命令 OR `hdfs dfs` 具体命令 两个是完全相同的*

### 命令大全

`hadoop fs `

```sh
[-appendToFile <localsrc> ... <dst>] 
        [-cat [-ignoreCrc] <src> ...] 
        [-chgrp [-R] GROUP PATH...] 
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...] 
        [-chown [-R] [OWNER][:[GROUP]] PATH...] 
        [-copyFromLocal [-f] [-p] <localsrc> ... <dst>] 
        [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>] 
        [-count [-q] <path> ...] 
        [-cp [-f] [-p] <src> ... <dst>] 
        [-df [-h] [<path> ...]] 
        [-du [-s] [-h] <path> ...] 
        [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>] 
        [-getmerge [-nl] <src> <localdst>] 
        [-help [cmd ...]] 
        [-ls [-d] [-h] [-R] [<path> ...]] 
        [-mkdir [-p] <path> ...] 
        [-moveFromLocal <localsrc> ... <dst>] 
        [-moveToLocal <src> <localdst>] 
        [-mv <src> ... <dst>] 
        [-put [-f] [-p] <localsrc> ... <dst>] 
        [-rm [-f] [-r|-R] [-skipTrash] <src> ...] 
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...] 
<acl_spec> <path>]] 
        [-setrep [-R] [-w] <rep> <path> ...] 
        [-stat [format] <path> ...] 
        [-tail [-f] <file>] 
        [-test -[defsz] <path>] 
        [-text [-ignoreCrc] <src> ...] 
```

创建一个文件夹，用于后续测试

```sh
hadoop fs -mkdir /sanguo 
```

### 上传

1、`-moveFromLocal`：从本地剪切粘贴到 HDFS 

在根目录下创建input/shuoguo.txt，文件内容为`shuguo`

```sh
hadoop fs  -moveFromLocal  ./input/shuguo.txt  /sanguo 
```

结果：本地文件删除，hadoop的web页面中查询出此文件的数据

2、`-copyFromLocal`：从本地文件系统中拷贝文件到 HDFS 路径去 

在根目录下创建input/weiguo.txt，文件内容为`weiguo`

```sh
hadoop fs  -copyFromLocal  ./input/weiguo.txt  /sanguo 
```

结果：本地文件文件上传成功而且没有删除

3、`-put`：等同于 `copyFromLocal`，生产环境更习惯用 `put `

在根目录下创建input/wuguo.txt，文件内容为`wuguo`

```sh
hadoop fs  -put  ./input/wuguo.txt  /sanguo 
```

4、`-appendToFile`：追加一个文件到已经存在的文件末尾 

```sh
hadoop fs -appendToFile ./input/liubei.txt /sanguo/shuguo.txt 
```

可以在web页面进行查看：

![hadoop上传命令测试](https://cos.duktig.cn/typora/202110062119679.png)

### 下载

在hadoop根目录下创建output文件夹，用以测试

1、`-copyToLocal`：从 HDFS 拷贝到本地 

```sh
hadoop fs -copyToLocal /sanguo/shuguo.txt ./output/shuguo.txt
```

2、`-ge`t：等同于 `copyToLocal`，生产环境更习惯用 get 

```sh
hadoop fs -get /sanguo/weiguo.txt ./output/weiguo.txt 
```

*如果拷贝到本地的路径中存在文件夹，需要提前创建好。*

### HDFS 直接操作 

```sh
# -ls: 显示目录信息
hadoop fs -ls /sanguo 
# -cat：显示文件内容 
hadoop fs -cat /sanguo/shuguo.txt 
# -chgrp、-chmod、-chown：Linux 文件系统中的用法一样，修改文件所属权限 
hadoop-3.1.3]$ hadoop fs  -chmod 666  /sanguo/shuguo.txt 
# -mkdir：创建路径(文件夹)
# -cp：从 HDFS 的一个路径拷贝到 HDFS 的另一个路径 
# -mv：在 HDFS 目录中移动文件 
# -rm：删除文件或文件夹 
# -rm -r：递归删除目录及目录里面内容 
# -tail：显示一个文件的末尾 1kb 的数据 
hadoop fs -tail /sanguo/shuguo.txt 
# -du 统计文件夹的大小信息
## 查看总共的大小信息
hadoop fs -du -s -h /sanguo
## 查看每个文件的大小信息
hadoop fs -du  -h /sanguo 
# -setrep：设置 HDFS 中文件的副本数量 
hadoop fs -setrep 10 /sanguo/shuguo.txt 
```

这里设置的副本数只是记录在 NameNode 的元数据中，是否真的会有这么多副本，还得看 DataNode 的数量。因为目前只有 3 台设备，最多也就 3 个副本，只有节点数的增加到 10台时，副本数才能达到 10。 



## HDFS的客户端API

开发前提条件：如果是win10本地开发，需要安装win10版本的Hadoop，参看上文：[win10 安装 Hadoop3.x](#win10 安装 Hadoop3.x)

引入依赖：

```xml
<dependency> 
    <groupId>org.apache.hadoop</groupId> 
    <artifactId>hadoop-client</artifactId> 
    <version>3.1.3</version> 
</dependency> 
```

### 获取文件系统和关闭资源

每次执行代码前需要获取文件系统，执行完后关闭资源。可以使用`@Before`和`@After`来方便测试。

```java
public class HdfsClient {

    private FileSystem fs;

    @Before
    public void init() {
        // 1 获取文件系统
        Configuration configuration = new Configuration();
        try {
            fs = FileSystem.get(new URI("hdfs://localhost:9000"), configuration, "rsw");
        } catch (IOException | InterruptedException | URISyntaxException e) {
            e.printStackTrace();
        }
    }

    @After
    public void close() {
        // 3 关闭资源
        try {
            fs.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    

    


}
```

### 创建目录

```java
/**
 * 测试 创建目录
 */
@Test
public void testMkdir() {
    // 2 创建目录
    try {
        fs.mkdirs(new Path("/duktig/"));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 文件上传

```java
/**
 * 测试 上传文件
 */
@Test
public void testCopyFromLocalFile() {
    try {
        // 参数1：是否删除文件 参数2：是否允许覆盖 参数3：本地文件路径 参数4：上传目标路径
        fs.copyFromLocalFile(false, true, new Path("src/main/resources/sunwukong.txt"), new Path("/duktig"));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 参数优先级

**HDFS的默认副本数量为3**：

```xml
<property>
  <name>dfs.replication</name>
  <value>3</value>
  <description>Default block replication. 
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
</property>
```

**配置前的文件副本数量**：

![配置前的文件副本数量](https://cos.duktig.cn/typora/202110071028338.png)

将 hdfs-site.xml 拷贝到项目的 resources 资源目录下 （**配置副本数量为1**）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <!--  HDFS的副本数量  -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>

```

代码配置副本数量（**配置副本数量为2**）：

```java
// 代码配置文件的默认副本数量
configuration.set("dfs.replication", "2");
```

配置后的文件副本数量：

![配置后的文件副本数量](https://cos.duktig.cn/typora/202110071028379.png)

由此得出优先级结论：

参数优先级排序：**（1）客户端代码中设置的值 >（2）ClassPath 下的用户自定义配置文件 >（3）然后是服务器的自定义配置（xxx-site.xml）>（4）服务器的默认配置（xxx-default.xml）**

### 文件下载

```java
/**
 * 测试文件下载
 */
@Test
public void testCopyToLocalFile() {
    try {
        /*
            执行下载操作
            boolean delSrc 指是否将原文件删除
            Path src 指要下载的文件路径
            Path dst 指将文件下载到的路径
            boolean useRawLocalFileSystem 是否开启文件校验
         */
        fs.copyToLocalFile(false, new Path("/duktig/sunwukong.txt"),
                new Path("src/main/resources/sunwukong2.txt"), true);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

本地有同名文件，不会报错，方式会覆盖原有的本地文件的数据。

### 文件/目录 重命名和移动

```java
/**
 * 文件/目录 重命名和移动
 * 参数1：源文件路径  参数2：目标文件路径
 */
@Test
public void testRename() {
    try {
        fs.rename(new Path("/duktig/sunwukong.txt"), new Path("/duktig/sunwukong-copy.txt"));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 删除 文件/目录

```java
/**
 * 测试 删除文件和目录
 * <p>
 * 参数1：删除的路径 参数2：是否递归删除（即删除目录）
 */
@Test
public void testDelete() {
    try {
        fs.delete(new Path("/duktig"), false);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

目录不为空，第二个参数为`false`时，会报错：

```java
org.apache.hadoop.fs.PathIsNotEmptyDirectoryException: ``/duktig is non empty': Directory is not empty
```

### HDFS 文件详情查看 

```java
/**
 * 获取文件详情
 */
@Test
public void testListFiles() {
    // 获取文件详情
    RemoteIterator<LocatedFileStatus> listFiles = null;
    try {
        // 参数1：文件路径 参数2：是否递归获取
        listFiles = fs.listFiles(new Path("/duktig"), true);
        while (listFiles.hasNext()) {
            LocatedFileStatus fileStatus = listFiles.next();
            System.out.println("========" + fileStatus.getPath() + "=========");
            System.out.println("权限：" + fileStatus.getPermission());
            System.out.println("所有者：" + fileStatus.getOwner());
            System.out.println("组：" + fileStatus.getGroup());
            System.out.println("文件长度：" + fileStatus.getLen());
            System.out.println("修改时间：" + fileStatus.getModificationTime());
            System.out.println("分片数量：" + fileStatus.getReplication());
            System.out.println("块大小：" + fileStatus.getBlockSize());
            System.out.println("文件名：" + fileStatus.getPath().getName());

            // 获取块信息
            BlockLocation[] blockLocations = fileStatus.getBlockLocations();
            System.out.println("文件的块信息：" + Arrays.toString(blockLocations));
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

结果：

```java
========hdfs://localhost:9000/duktig/sunwukong-copy.txt=========
权限：rw-r--r--
所有者：rsw
组：supergroup
文件长度：10
修改时间：1633573140393
分片数量：1
块大小：134217728
文件名：sunwukong-copy.txt
文件的块信息：[0,10,DELL-RSW.mshome.net]
```

### HDFS 文件和文件夹判断 

```java
/**
 * 文件和文件夹判断
 */
@Test
public void testListStatus() {
    // 判断是文件还是文件夹
    try {
        FileStatus[] listStatus = fs.listStatus(new Path("/duktig"));
        for (FileStatus fileStatus : listStatus) {
            // 如果是文件
            if (fileStatus.isFile()) {
                System.out.println("f:" + fileStatus.getPath().getName());
            } else {
                System.out.println("d:" + fileStatus.getPath().getName());
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## HDFS的读写流程（面试重点）

### HDFS的写流程

#### 文件写入整体流程

![HDFS的写流程](https://cos.duktig.cn/typora/202110071200957.png)

流程详解：

1. 客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。 
2. NameNode 返回是否可以上传（检查权限和目录是否存在）。
3. 客户端请求第一个 Block 上传到哪几个 DataNode 服务器上
4. NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3
5. 客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3 逐级应答客户端。 
7. 客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存），以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给dn3；dn1 **每传一个 packet会放入一个应答队列等待应答**。 
8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。（重复执行 3-7 步）。 

####  网络拓扑-节点距离计算

 在 HDFS 写数据的过程中，NameNode 会选择距离待上传数据最近距离的 DataNode 接收数据。那么这个最近距离怎么计算呢？ 

**节点距离：两个节点到达最近的共同祖先的距离总和**。 

![网络拓扑-节点距离计算](https://cos.duktig.cn/typora/202110071402203.png)



![网络拓扑-节点距离计算](https://cos.duktig.cn/typora/202110071401536.png)

#### 副本存储节点选择

> [http://hadoop.apache.org/docs/r3.1.3/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Data_Replication ](http://hadoop.apache.org/docs/r3.1.3/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Data_Replication)
>
> 源码说明：server端（hadoop安装包的源码）查找 `BlockPlacementPolicyDefault`，在该类中查找 `chooseTargetInOrder` 方法

![副本存储节点选择](https://cos.duktig.cn/typora/202110071406949.png)

### HDFS的读流程

![HDFS的读流程](https://cos.duktig.cn/typora/202110071415563.png)

流程详解：

1. 客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。
2. 挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。 
   1. 就近选择
   2. 就近服务器达到负载后，随机选择
3. DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。 
4. 客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

## NameNode 和 SecondaryNameNode

### 思考：NameNode 中的元数据是存储在哪里的？

如果存储在 NameNode 节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。**因此产生在磁盘中备份元数据的FsImage**。 

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新 FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦 NameNode 节点断电，就会产生数据丢失。

**因此，引入 Edits 文件（只进行追加操作，效率很高）。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到 Edits 中**。这样，一旦 NameNode 节点断电，可以通过 FsImage 和 Edits 的合并，合成元数据。 

但是，如果长时间添加数据到 Edits 中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行 FsImage 和 Edits 的合并，如果这个操作由NameNode 节点完成，又会效率过低。**因此，引入一个新的节点 SecondaryNamenode，专门用于 FsImage 和 Edits 的合并**。 

### 工作机制

![NameNode和SecondaryNameNode 的工作机制](https://cos.duktig.cn/typora/202110071442451.png)

#### 第一阶段：NameNode 启动 

1. 第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。 
2. 客户端对元数据进行增删改的请求。 
3. NameNode 记录操作日志，更新滚动日志。 
4. NameNode 在内存中对元数据进行增删改。 

#### 第二阶段：Secondary NameNode 工作 

1. Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接返回 NameNode是否检查结果。
2. Secondary NameNode 请求执行 CheckPoint。
3. NameNode 滚动正在写的 Edits 日志。 
4. 将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。 
5. Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。 
6. 生成新的镜像文件 fsimage.chkpoint。
7. 拷贝 fsimage.chkpoint 到 NameNode。 
8. NameNode 将 fsimage.chkpoint 重新命名成 fsimage。 

### Fsimage 和 Edits 解析

#### 概念

NameNode被格式化之后，将在data/tmp/dfs/name/current目录中产生如下文件

（1）Fsimage文件：HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目录和文件inode的序列化信息。

```
fsimage_0000000000000000000
fsimage_0000000000000000000.md5
seen_txid
VERSION
```

（2）Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先会被记录到Edits文件中。
（3）seen_txid文件保存的是一个数字，就是最后一个edits_的数字
（4）每次NameNode启动的时候都会将Fsimage文件读入内存，加载Edits里面的更新操作，保证内存中的元数据信息是最新的、同步的，可以看成NameNode启动的时候就将Fsimage和Edits文件进行了合并。

#### oiv 查看 Fsimage 文件

```
hdfs oiv -p 文件类型 -i 镜像文件 -o 转换后文件输出路径 
```

**思考**：**根据文件可以看出，Fsimage 中没有记录块所对应 DataNode，为什么？** 
在集群启动后，要求 DataNode 上报数据块信息，并间隔一段时间后再次上报。 

#### oev 查看 Edits 文件 

```
hdfs oev -p 文件类型 -i 编辑日志 -o 转换后文件输出路径 
```

**思考：NameNode 如何确定下次开机启动的时候合并哪些 Edits？** 

### CheckPoint 时间设置

通常情况下，SecondaryNameNode 每隔一小时执行一次。 

```xml
# hdfs-default.xml
<property> 
  <name>dfs.namenode.checkpoint.period</name> 
  <value>3600s</value> 
</property> 
```

一分钟检查一次操作次数，当操作次数达到 1 百万时，SecondaryNameNode 执行一次。 

```xml
<property> 
    <name>dfs.namenode.checkpoint.txns</name> 
    <value>1000000</value> 
    <description>操作动作次数</description> 
</property> 

<property> 
    <name>dfs.namenode.checkpoint.check.period</name> 
    <value>60s</value> 
    <description> 1 分钟检查一次操作次数</description> 
</property> 
```

*一般会搭建高可用NameNode集群，不使用SecondaryNameNode。*

## Datanode

### Datanode工作机制

![Datanode工作机制](https://cos.duktig.cn/typora/202110071506634.png)



1. 一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

2. DataNode 启动后向 NameNode 注册，通过后，周期性（6 小时）的向 NameNode 上报所有的块信息。 

DN 向 NN 汇报当前解读信息的时间间隔，默认 6 小时； 

```xml
<property> 
    <name>dfs.blockreport.intervalMsec</name> 
    <value>21600000</value> 
    <description>Determines block reporting interval in 
        milliseconds.</description> 
</property> 
```

DN 扫描自己节点块信息列表的时间，默认 6 小时 

```xml
<property> 
    <name>dfs.datanode.directoryscan.interval</name> 
    <value>21600s</value> 
    <description>Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk. Support multiple time unit suffix(case insensitive), as described in dfs.heartbeat.interval. 
    </description> 
</property> 
```

3. 心跳是每 3 秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块数据到另一台机器，或删除某个数据块。如果超过 10 分钟+30s（10次）没有收到某个 DataNode 的心跳，则认为该节点不可用。 
4. 集群运行中可以安全加入和退出一些机器。 

### 数据完整性

> 思考：如果电脑磁盘里面存储的数据是控制高铁信号灯的红灯信号（1）和绿灯信号（0），但是存储该数据的磁盘坏了，一直显示是绿灯，是否很危险？同理 DataNode 节点上的数据损坏了，却没有发现，是否也很危险，那么如何解决呢？ 

如下是 DataNode 节点保证数据完整性的方法。 
（1）当 DataNode 读取 Block 的时候，它会计算 CheckSum。 

（2）如果计算后的 CheckSum，与 Block 创建时值不一样，说明 Block 已经坏。

（3）Client 读取其他 DataNode 上的 Block。 

（4）常见的校验算法 **crc（32）**，md5（128），sha1（160） 

（5）DataNode 在其文件创建后周期验证 CheckSum。 

![数据完整性](https://cos.duktig.cn/typora/202110071525356.png)

### 掉线时限参数设置

![DataNode掉线时限参数设置](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211007152704165.png)

需要注意的是 hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒。 

```xml
<property> 
    <name>dfs.namenode.heartbeat.recheck-interval</name> 
    <value>300000</value> 
</property> 

<property> 
    <name>dfs.heartbeat.interval</name> 
    <value>3</value> 
</property> 
```
