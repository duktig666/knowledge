# Hadoop概述

## Hadoop是什么？

1. Hadoop是一个由Apache基金会所开发的**分布式系统基础架构**。
2. 主要解决，海量数据的**存储**和海量数据的**分析计算**问题。
3. 广义上来说，Hadoop通常是指一个更广泛的概念——**Hadoop生态圈**。

## Hadoop 发展历史（了解） 

1. Hadoop创始人**Doug Cutting**，为了实现与Google类似的全文搜索功能，他在**Lucene框架**基础上进行优化升级，查询引擎和索引引擎。
2. 2001年年底Lucene成为Apache基金会的一个子项目。
3. 对于海量数据的场景，Lucene框架面对与Google同样的困难，**存储海量数据困难，检索海量速度慢**。
4. 学习和模仿Google解决这些问题的办法 ：微型版Nutch。
5. 可以说Google是Hadoop的思想之源（Google在大数据方面的三篇论文）
   1. GFS --->HDFS
   2. Map-Reduce --->MR
   3. BigTable --->HBase
6. 2003-2004年，Google公开了部分GFS和MapReduce思想的细节，以此为基础Doug Cutting等人用了2年业余时间实现了DFS和MapReduce机制，使Nutch性能飙升。
7. 2005 年Hadoop 作为Lucene的子项目Nutch的一部分正式引入Apache基金会。
8. 2006 年3 月份，Map-Reduce和Nutch Distributed File System （NDFS）分别被纳入到Hadoop 项目中，Hadoop就此正式诞生，标志着大数据时代来临。
9. 名字来源于Doug Cutting儿子的玩具大象

**总结**：

1. 创始人：**Doug Cutting**
2. 名字由来：Doug Cutting儿子的玩具大象
3. 基础：**Lucene框架**
4. 背景：Lucene框架和Google面对的困难：**存储海量数据困难，检索海量速度慢**
5. 思想之源：谷歌三篇论文
   1. GFS --->HDFS
   2. Map-Reduce --->MR
   3. BigTable --->HBase

## Hadoop 三大发行版本（了解） 

Hadoop 三大发行版本：Apache、Cloudera、Hortonworks。 

- Apache 版本最原始（最基础）的版本，对于入门学习最好。2006 
- Cloudera 内部集成了很多大数据框架，对应产品 CDH。2008 
- Hortonworks 文档较好，对应产品 HDP。2011 
- Hortonworks 现在已经被 Cloudera 公司收购，推出新的品牌 CDP。 2018
- 2021宣布所有版本收费

## Hadoop 优势（4 高） 

高可靠性：Hadoop底层维护多个数据副本，所以即使Hadoop某个计算元
素或存储出现故障，也不会导致数据的丢失。

![高可靠性](https://cos.duktig.cn/typora/202110061035051.png)

高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点。可实现不停机扩展节点。

![高扩展性](https://cos.duktig.cn/typora/202110061035664.png)

高效性：在MapReduce的思想下，Hadoop是并行工作的，以加快任务处
理速度。

![高效性](https://cos.duktig.cn/typora/202110061035670.png)

高容错性：能够自动将失败的任务重新分配。

![高容错性](https://cos.duktig.cn/typora/202110061035280.png)

## Hadoop 组成（重点）

![Hadoop1.x、2.x、3.x区别](https://cos.duktig.cn/typora/202110061036778.png)

### HDFS

Hadoop Distributed File System，简称 **HDFS**，是一个**分布式文件系统**。 

- NameNode（nn）：**存储文件的元数据**，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。**用来标识数据都存储在什么位置**。
- DataNode(dn)：在本地文件系统**存储文件块数据**，以及**块数据的校验和**。
- Secondary NameNode(2nn)：每隔一段时间**对NameNode元数据备份**。

### YARN

Yet Another Resource Negotiator 简称 YARN ，另一种资源协调者，是 Hadoop 的**资源管理器**。

1）ResourceManager（RM）：管理整个集群资源（内存、CPU等）

3）ApplicationMaster（AM）：管理单个任务运行

2）NodeManager（NM）：管理单个节点服务器资源

4）Container：容器，相当一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。

![YARN架构](https://cos.duktig.cn/typora/202110061050089.png)

说明1：客户端可以有多个

说明2：集群上可以运行多个ApplicationMaster

说明3：每个NodeManager上可以有多个Container

### MapReduce

MapReduce 将计算过程分为两个阶段：Map 和 Reduce 
1）Map 阶段并行处理输入数据 
2）Reduce 阶段对 Map 结果进行汇总 

![MapReduce架构](https://cos.duktig.cn/typora/202110061058676.png)

### HDFS、YARN、MapReduce 三者关系 

![HDFS、YARN、MapReduce 三者关系 ](https://cos.duktig.cn/typora/202110061100391.png)

# Hadoop的环境配置

## win10 安装 Hadoop3.x

win10下安装Hadoop3.x参看：

- [Win10安装使用Hadoop3.0.0](https://blog.csdn.net/songhaifengshuaige/article/details/79575308)
- [win10下安装Hadoop快速搞定——亲测有效](https://www.cnblogs.com/caiyishuai/p/12392070.html)

启动：

管理员命令下运行

```
hdfs namenode -format
sbin/start-all.cmd
```

测试：

- **hadoop的web界面**：[http://localhost:9870/](http://localhost:9870/)

- **yarn的web界面**：[http://localhost:8088/cluster](http://localhost:8088/cluster)

问题：

1、win10安装hadoop启动所有进程后，发现resourcemanager报错：

```java
FATAL resourcemanager.Resourcelanager : Error starting ResourceManager
java.lang.NoClassDefF oundBrror: org/apache/hadoop/yarn/server/timelineservice/col1ector/TimelineCol1ectoranager
```

解决：

将`hadoop安装目录下\share\hadoop\yarn\timelineservice\hadoop-yarn-server-timelineservice-3.1.1.jar`

移动到`hadoop安装目录下\share\hadoop\yarn\hadoop-yarn-server-timelineservice-3.1.1.jar`

2、win10安装hadoop启动所有进程后，发现nodemanager报错：

```java
2022021-10-06 16:39:15,916 ERROR nodemanager.NodelManager: Error starting Nodellanager
b36org.apache. hadoop,yarn. exceptions. YarnfuntimeException: Failed to setup local dir /tmp/hadoup-rsw/m-1ocal-dir，which wa202s marked as good.
```

解决：以管理员身份运行`sbin/start-all.cmd`即可。

在hadoop根目录测试：

```
hadoop jar /D:\javaweb\bigdata\hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar /D:\javaweb\bigdata\hadoop-3.1.3\intput\word.txt /D:\javaweb\bigdata\hadoop-3.1.3\output


hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount input/word.txt  output 
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount input output 
```



## 配置历史服务器

### win10环境下

配置 mapred-site.xml 

```xml
<!-- 历史服务器端地址 --> 
<property> 
    <name>mapreduce.jobhistory.address</name> 
    <value>localhost:10020</value> 
</property> 
 
<!-- 历史服务器 web端地址 --> 
<property> 
    <name>mapreduce.jobhistory.webapp.address</name> 
    <value>localhost:19888</value> 
</property> 
```

在bin目录下，cmd运行命令：

```shell
mapred historyserver
```

查看 JobHistory 
[http://hadoop102:19888/jobhistory](http://hadoop102:19888/jobhistory ) 

## 配置日志的聚集

配置`yarn-site.xml`

```xml
<!-- 开启日志聚集功能 --> 
<property> 
    <name>yarn.log-aggregation-enable</name> 
    <value>true</value> 
</property> 
<!-- 设置日志聚集服务器地址 --> 
<property>   
    <name>yarn.log.server.url</name>   
    <value>http://hadoop102:19888/jobhistory/logs</value> 
</property> 
<!-- 设置日志保留时间为 7天 --> 
<property> 
    <name>yarn.log-aggregation.retain-seconds</name> 
    <value>604800</value> 
</property> 
```

重启程序。

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
  -  某一个副本丢失以后，它可以自动恢复。
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

# MapReduce

##  MapReduce 概述 

### 定义

MapReduce 是一个**分布式运算程序**的编程框架，是用户开发“基于 Hadoop 的数据分析应用”的核心框架。 
MapReduce 核心功能是**将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上**。 

### MapReduce优缺点

#### 优点

1、易于编程。 用户只关心，业务逻辑。 实现框架的接口。
2、良好扩展性：可以动态增加服务器，解决计算资源不够问题
3、高容错性。任何一台机器挂掉，可以将任务转移到其他节点。
4、适合海量数据计算（TB/PB） 几千台服务器共同计算。

#### 缺点

1、**不擅长实时计算。 Mysql**（在毫秒或者秒级内返回结果）
2、**不擅长流式计算。 Spark Streaming | flink 。**流式计算的输入数据是动态的，而 MapReduce 的输入数据集是静态的，不能动态变化。
3、**不擅长DAG有向无环图计算。spark 。** 多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce 并不是不能做，而是使用后，每个 MapReduce 作业的输出结果都会写入到磁盘，会造成大量的磁盘 IO，导致性能非常的低下。 

### MapReduce 核心思想

![MapReduce 核心思想 ](https://cos.duktig.cn/typora/202110071622121.png)

### MapReduce 进程

一个完整的 MapReduce 程序在分布式运行时有三类实例进程： 

（1）MrAppMaster：负责整个程序的过程调度及状态协调。 

（2）MapTask：负责 Map 阶段的整个数据处理流程。 

（3）ReduceTask：负责 Reduce 阶段的整个数据处理流程。 

### 常用数据序列化类型

| Java 类型 | Hadoop Writable 类型 |
| --------- | -------------------- |
| Boolean   | BooleanWritable      |
| Byte      | ByteWritable         |
| Int       | IntWritable          |
| Float     | FloatWritable        |
| Long      | LongWritable         |
| Double    | DoubleWritable       |
| String    | Text                 |
| Map       | MapWritable          |
| Array     | ArrayWritable        |
| Null      | NullWritable         |
|           |                      |

### MapReduce 编程规范

1．Mapper阶段

（1）用户自定义的Mapper要继承自己的父类

（2）Mapper的输入数据是KV对的形式（KV的类型可自定义）

（3）Mapper中的业务逻辑写在map()方法中

（4）Mapper的输出数据是KV对的形式（KV的类型可自定义）

（5）**map()方法（MapTask进程）对每一个<K,V>调用一次**

2．Reducer阶段

（1）用户自定义的Reducer要继承自己的父类

（2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV

（3）Reducer的业务逻辑写在reduce()方法中

（4）**ReduceTask进程对每一组相同k的<k,v>组调用一次reduce()方法**

3．Driver阶段

相当于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是
封装了MapReduce程序相关运行参数的job对象

### MapReduce实战——WordCount 统计单词次数

按照 MapReduce 编程规范，分别编写 Mapper，Reducer，Driver。 

![统计一堆文件中单词出现的个数（WordCount案例）](https://cos.duktig.cn/typora/202110071646744.png)

#### 编写 Mapper 类 

```java
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    Text outK = new Text();
    IntWritable outV = new IntWritable(1);

    /**
     * 一行数据执行一次
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 1 获取一行: duktig duktig
        String line = value.toString();
        // 2 切割
        // duktig
        // duktig
        String[] words = line.split(" ");
        // 3 输出
        for (String word : words) {
            // outK 和 outV for循环执行多次，一行数据执行一次 map方法，避免频繁创建对象，故写成成员变量
            outK.set(word);
            context.write(outK, outV);
        }
    }

}
```

#### 编写 Reducer 类 

```java
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    IntWritable outV = new IntWritable();

    /**
     * 一个key执行一次
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException,
            InterruptedException {
        // 1 累加求和 duktig,(1,1)
        int sum = 0;
        for (IntWritable count : values) {
            sum += count.get();
        }

        // 2 输出
        outV.set(sum);
        context.write(key, outV);
    }

}
```

#### 编写 Driver 驱动类

```java
public class WordCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // 1 获取配置信息以及获取 job 对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 2 关联本 Driver 程序的 jar
        job.setJarByClass(WordCountDriver.class);

        // 3 关联 Mapper 和 Reducer 的 jar
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        // 4 设置 Mapper 输出的 kv 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 5 设置最终输出 kv 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 6 设置输入和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 7 提交 job
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);
    }

}
```

#### 测试

在项目模块的resources下编写输入文件——WordCountInput.txt

```txt
duktig duktig
FPX
RNG EDG
LNG
FPX
```

在  **Driver 驱动类** 的第6步，没有直接读取文件路径，而是读main方法的参数，开发环境下可以利用idea写参数。*（如果只是为了测试，路径写死即可）*

第一个参数：输入文件路径

第二个参数：输出文件目录

![测试参数](https://cos.duktig.cn/typora/202110071800851.png)

参数内容：

```java
hadoop-example/src/main/resources/WordCountInput.txt hadoop-example/src/main/resources/WordCountOutput
```

注意：junit相对路径以src开始，main方法相对路径以module开始。

结果：

![测试结果](https://cos.duktig.cn/typora/202110071800562.png)

part-r-00000统计结果：

```txt
EDG	1
FPX	2
LNG	1
RNG	1
duktig	2
```

生产环境下，如果在linux，可将项目模块打成jar包，然后执行命令运行：

```sh
hadoop jar  xxx.jar 
 cn.duktig.mapreduce.wordcount.WordCountDriver 输入文件路径 输出文件目录
```

## MapReduce序列化

###  序列化概述

#### 什么是序列化 ？

序列化就是**把内存中的对象，转换成字节序列**（或其他数据传输协议）以便于存储到磁盘（持久化）和网络传输。  
反序列化就是将收到字节序列（或其他数据传输协议）或者是**磁盘的持久化数据，转换成内存中的对象**。 

#### 为什么要序列化 ？

一般来说，“活的”对象只生存在内存里，关机断电就没有了。而且“活的”对象只能由本地的进程使用，不能被发送到网络上的另外一台计算机。 然而**序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机**。 

#### 为什么不用 Java 的序列化 ？

Java 的序列化是一个重量级序列化框架（`Serializable`），一个对象被序列化后，会附带很多额外的信息（各种校验信息，Header，继承体系等），不便于在网络中高效传输。所以，Hadoop 自己开发了一套序列化机制（`Writable`）。 

#### Hadoop 序列化特点： 

（1）紧凑 ：高效使用存储空间。 

（2）快速：读写数据的额外开销小。 

（3）互操作：支持多语言的交互 

### 自定义 bean 对象实现序列化接口（Writable） 

在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在 Hadoop 框架内部传递一个 bean 对象，那么该对象就需要实现序列化接口。 

具体实现 bean 对象序列化步骤如下 7 步：

1. 必须实现 `Writable` 接口 

2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造 

   ```java
   public FlowBean() { 
       super(); 
   } 
   ```

3. 重写序列化方法 

   ```java
   @Override 
   public void write(DataOutput out) throws IOException { 
       out.writeLong(upFlow); 
       out.writeLong(downFlow); 
       out.writeLong(sumFlow); 
   } 
   ```

4. 重写反序列化方法 

   ```java
   @Override 
   public void readFields(DataInput in) throws IOException { 
       upFlow = in.readLong(); 
       downFlow = in.readLong(); 
       sumFlow = in.readLong(); 
   } 
   ```

5. **注意反序列化的顺序和序列化的顺序完全一致** 

6. 要想把结果显示在文件中，需要重写 `toString()`，可用"`\t`"分开，方便后续用。 

7. 如果需要将自定义的 bean 放在 key 中传输，则还需要实现 `Comparable` 接口，因为MapReduce 框中的 Shuffle 过程要求对 key 必须能排序。

   ```java
   @Override 
   public int compareTo(FlowBean o) { 
       // 倒序排列，从大到小 
       return this.sumFlow > o.getSumFlow() ? -1 : 1; 
   } 
   ```



### 序列化案例（统计手机号的流量）

**需求：**

统计每一个手机号耗费的总上行流量、总下行流量、总流量 

**输入数据：**

phone_data .txt

```
id   手机号码       网络ip                        上行流量 下行流量 网络状态码 
1	13736230513	192.196.100.1	www.atguigu.com	2481	24681	200
2	13846544121	192.196.100.2			264	0	200
3 	13956435636	192.196.100.3			132	1512	200
4 	13966251146	192.168.100.1			240	0	404
5 	18271575951	192.168.100.2	www.atguigu.com	1527	2106	200
……
```

**输出数据：**

期望输出数据格式 

```
13560436666 1116 954 2070 
手机号码 上行流量 下行流量 总流量 
```

**需求分析：**

![需求分析](https://cos.duktig.cn/typora/202110081447362.png)

**编写 MapReduce 程序** 

**（1）编写流量统计的 Bean 对象** 

```java
public class FlowBean implements Writable {

    /** 上行流量 */
    private long upFlow;
    /** 下行流量 */
    private long downFlow;
    /** 总流量 */
    private long sumFlow;

    public FlowBean() {
    }

    /**
     * 序列化方法 (序列化和反序列化顺序要一致)
     */
    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeLong(upFlow);
        dataOutput.writeLong(downFlow);
        dataOutput.writeLong(sumFlow);
    }

    /**
     * 反序列化方法 (序列化和反序列化顺序要一致)
     */
    @Override
    public void readFields(DataInput dataInput) throws IOException {
        this.upFlow = dataInput.readLong();
        this.downFlow = dataInput.readLong();
        this.sumFlow = dataInput.readLong();
    }

    /**
     * 重写 ToString
     */
    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }
}
```

**（2）编写 Mapper 类** 

```java
public class FlowMapper extends Mapper<LongWritable, Text, Text, FlowBean> {

    private Text outK = new Text();
    private FlowBean outV = new FlowBean();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1 获取一行数据,转成字符串
        String line = value.toString();

        //2 切割数据
        String[] split = line.split("\t");

        //3 抓取我们需要的数据:手机号,上行流量,下行流量
        String phone = split[1];
        String up = split[split.length - 3];
        String down = split[split.length - 2];

        //4 封装 outK outV
        outK.set(phone);
        outV.setUpFlow(Long.parseLong(up));
        outV.setDownFlow(Long.parseLong(down));
        outV.setSumFlow(Long.parseLong(up) + Long.parseLong(down));

        //5 写出 outK outV
        context.write(outK, outV);
    }

}
```

**（3）编写 Reducer 类** 

```java
public class FlowReducer extends Reducer<Text, FlowBean, Text, FlowBean> {

    private FlowBean outV = new FlowBean();

    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException,
            InterruptedException {
        long totalUp = 0;
        long totalDown = 0;

        //1 遍历 values,将其中的上行流量,下行流量分别累加
        for (FlowBean flowBean : values) {
            totalUp += flowBean.getUpFlow();
            totalDown += flowBean.getDownFlow();
        }

        //2 封装 outKV
        outV.setUpFlow(totalUp);
        outV.setDownFlow(totalDown);
        outV.setSumFlow(totalUp + totalDown);

        //3 写出 outK outV
        context.write(key, outV);
    }

}
```

**（4）编写 Driver 驱动类** 

```java
public class FlowDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        //1 获取 job 对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        //2 关联本 Driver 类
        job.setJarByClass(FlowDriver.class);

        //3 关联 Mapper 和 Reducer
        job.setMapperClass(FlowMapper.class);
        job.setReducerClass(FlowReducer.class);

        //4 设置 Map 端输出 KV 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);

        //5 设置程序最终输出的 KV 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //6 设置程序的输入输出路径
        FileInputFormat.setInputPaths(job, new Path("hadoop-example/src/main/resources/phone_data.txt"));
        FileOutputFormat.setOutputPath(job, new Path("hadoop-example/src/main/resources/phone_output"));

        //7 提交 Job
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);

    }

}
```

**结果**：

```java
13470253144	180	180	360
13509468723	7335	110349	117684
13560439638	918	4938	5856
13568436656	3597	25635	29232
13590439668	1116	954	2070
13630577991	6960	690	7650
13682846555	1938	2910	4848
13729199489	240	0	240
13736230513	2481	24681	27162
13768778790	120	120	240
13846544121	264	0	264
13956435636	132	1512	1644
13966251146	240	0	240
13975057813	11058	48243	59301
13992314666	3008	3720	6728
15043685818	3659	3538	7197
15910133277	3156	2936	6092
15959002129	1938	180	2118
18271575951	1527	2106	3633
18390173782	9531	2412	11943
84188413	4116	1432	5548
```

## MapReduce 框架原理

![MapReduce 框架原理](https://cos.duktig.cn/typora/202110081525220.png)

### InputFormat 数据输入

#### 切片与 MapTask 并行度决定机制

##### 问题引出

MapTask 的并行度决定 Map 阶段的任务处理并发度，进而影响到整个 Job 的处理速度。 
**思考：1G 的数据，启动 8 个 MapTask，可以提高集群的并发处理能力。那么 1K 的数据，也启动 8 个 MapTask，会提高集群性能吗？MapTask 并行任务是否越多越好呢？哪些因素影响了 MapTask 并行度？**

##### MapTask 并行度决定机制

数据块：Block 是 HDFS 物理上把数据分成一块一块。**数据块是 HDFS 存储数据单位**。
数据切片：数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。**数据切片是 MapReduce 程序计算输入数据的单位**，一个切片会对应启动一个 MapTask。

![MapTask 并行度决定机制 ](https://cos.duktig.cn/typora/202110081544504.png)

#### InputFormat实现类

思考：**在运行 MapReduce 程序时，输入的文件格式包括：基于行的日志文件、二进制格式文件、数据库表等**。那么，**针对不同的数据类型，MapReduce 是如何读取这些数据的呢？** 

##### FileInputFormat 实现类 

`FileInputFormat` 常见的接口实现类包括：`TextInputFormat`、`KeyValueTextInputFormat`、`NLineInputFormat`、`CombineTextInputFormat` 和自定义 `InputFormat` 等。 

##### TextInputFormat 

`TextInputFormat` 是默认的 `FileInputFormat` 实现类。按行读取每条记录。键是存储该行在整个文件中的起始字节偏移量， `LongWritable` 类型。值是这行的内容，不包括任何行终止符（换行符和回车符），`Text` 类型。 

以下是一个示例，比如，一个分片包含了如下 4 条文本记录。 

```txt
Rich learning form 
Intelligent learning engine 
Learning more convenient 
From the real demand for more close to the enterprise 
```

每条记录表示为以下键/值对： 

```txt
(0,Rich learning form) 
(20,Intelligent learning engine) 
(49,Learning more convenient) 
(74,From the real demand for more close to the enterprise) 
```

#### CombineTextInputFormat 切片机制 

框架默认的 `TextInputFormat` 切片机制是对任务按文件规划切片，**不管文件多小，都会是一个单独的切片**，都会交给一个 MapTask，这样**如果有大量小文件，就会产生大量的MapTask，处理效率极其低下**。 

**1）应用场景：** 

`CombineTextInputFormat` 用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个 MapTask 处理。 

**2）虚拟存储切片最大值设置** 

```java
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m 
```

注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

**3）切片机制** 

生成切片过程包括：虚拟存储过程和切片过程二部分。 

![CombineTextInputFormat切片机制](https://cos.duktig.cn/typora/202110081609776.png)

（1）虚拟存储过程： 
将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块：**当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时将文件均分成 2 个虚拟存储块（防止出现太小切片）**。 

例如 setMaxInputSplitSize 值为 4M，输入文件大小为 8.02M，则先逻辑上分成一个4M。剩余的大小为 4.02M，如果按照 4M 逻辑划分，就会出现 0.02M 的小的虚拟存储文件，所以将剩余的 4.02M 文件切分成（2.01M 和 2.01M）两个文件。 

（2）切片过程： 

（a）判断虚拟存储的文件大小是否大于 setMaxInputSplitSize 值，大于等于则单独形成一个切片。 

（b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

（c）**测试举例：有 4 个小文件大小分别为 1.7M、5.1M、3.4M 以及 6.8M 这四个小文件，则虚拟存储之后形成 6 个文件块，大小分别为：** 

**1.7M，（2.55M、2.55M），3.4M 以及（3.4M、3.4M）** 

**最终会形成 3 个切片，大小分别为：** 

**（1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M** 

**操作：**

驱动类中添加代码如下： 

```java
// 如果不设置 InputFormat，它默认用的是 TextInputFormat.class 
job.setInputFormatClass(CombineTextInputFormat.class); 

//虚拟存储切片最大值设置 4m 
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304); 
```

### MapReduce详细工作流程

![MapReduce工作流程1](https://cos.duktig.cn/typora/202110081644694.png)

![MapReduce工作流程2](https://cos.duktig.cn/typora/202110081645208.png)

### Shuffle 机制

#### Shuffle 机制

`Map` 方法之后，`Reduce`方法之前的数据处理过程称之为 Shuffle。 

![Shuffle 机制](https://cos.duktig.cn/typora/202110081649265.png)

#### Partition 分区

##### 问题引出

要求将统计结果**按照条件输出到不同文件中**（分区）。比如：将统计结果按照手机
归属地不同省份输出到不同文件中（分区）

##### 默认Partitioner分区

```java
public class HashPartitioner<K, V> extends Partitioner<K, V>{

    public int getPartition(K key, V value, int numReduceTasks) {
        return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
    }

}
```

默认分区是根据key的`hashCode`对`ReduceTasks`个数取模得到的。**用户没法控制哪个key存储到哪个分区**。

##### 自定义Partitioner步骤

（1）自定义类继承`Partitioner`，重写`getPartition()`方法

```java
public class CustomPartitioner extends Partitioner<Text, FlowBean> {
    @Override
    public int getPartition(Text key, FlowBean value, int numPartitions) {
        // 控制分区代码逻辑
        … …
            return partition;
    }
}
```

（2）在Job驱动中，设置自定义Partitioner

```java
job.setPartitionerClass(CustomPartitioner.class);
```

（3）自定义Partition后，要根据自定义Partitioner的逻辑设置相应数量的ReduceTask

```java
job.setNumReduceTasks(5);
```

##### 分区总结

**（1）如果ReduceTask的数量> getPartition的结果数，则会多产生几个空的输出文件part-r-000xx；**
**（2）如果1<ReduceTask的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception；**
**（3）如果ReduceTask的数量=1，则不管MapTask端输出多少个分区文件，最终结果都交给这一个ReduceTask，最终也就只会产生一个结果文件part-r-00000；**

##### 案例分析

例如：假设自定义分区数为5，则

（1）job.setNumReduceTasks(1);

（2）job.setNumReduceTasks(2);

（3）job.setNumReduceTasks(6);

会正常运行，只不过会产生一个输出文件会报错大于5，程序会正常运行，会产生空文件

（4）分区号必须从零开始，逐一累加。

#### Partition 分区实例

##### **需求** 

将统计结果按照手机归属地不同省份输出到不同文件中（分区） 

输入数据：phone_data .txt 

期望输出数据 ：

 手机号 136、137、138、139 开头都分别放到一个独立的 4 个文件中，其他开头的放到一个文件中。 

##### **需求分析**

![需求分析](https://cos.duktig.cn/typora/202110081718203.png)

##### 代码实现

在之前的 [writable案例（不同手机号的流量统计）](#序列化案例（统计手机号的流量）) 的基础上增加分区操作

**增加一个分区类** 

```java
public class ProvincePartitioner extends Partitioner<Text, FlowBean> {

    /**
     * 分区逻辑
     */
    @Override
    public int getPartition(Text text, FlowBean flowBean, int i) {
        //获取手机号前三位 prePhone
        String phone = text.toString();
        String prePhone = phone.substring(0, 3);

        //定义一个分区号变量 partition,根据 prePhone 设置分区号
        int partition;

        switch (prePhone) {
            case "136":
                partition = 0;
                break;
            case "137":
                partition = 1;
                break;
            case "138":
                partition = 2;
                break;
            case "139":
                partition = 3;
                break;
            default:
                partition = 4;
                break;
        }

        //最后返回分区号 partition
        return partition;
    }

}
```

**在驱动函数中增加自定义数据分区设置和 ReduceTask 设置** 

```java
public class FlowPartitionerDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        //1 获取 job 对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        //2 关联本 Driver 类
        job.setJarByClass(FlowPartitionerDriver.class);

        //3 关联 Mapper 和 Reducer
        job.setMapperClass(FlowMapper.class);
        job.setReducerClass(FlowReducer.class);

        //4 设置 Map 端输出 KV 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);

        //5 设置程序最终输出的 KV 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //8 ** 指定自定义分区器
        job.setPartitionerClass(ProvincePartitioner.class);

        //9 ** 同时指定相应数量的 ReduceTask
        job.setNumReduceTasks(5);

        //6 设置程序的输入输出路径
        FileInputFormat.setInputPaths(job, new Path("hadoop-example/src/main/resources/phone_data.txt"));
        FileOutputFormat.setOutputPath(job, new Path("hadoop-example/src/main/resources/phone_partition_output"));

        //7 提交 Job
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);

    }

}
```

*本操作对应步骤8和步骤9。*

##### **运行结果**

分了5个文件，如下图

![分区结果](https://cos.duktig.cn/typora/202110081730114.png)

具体文件内容如下：

```txt
# part-r-00000
13630577991	6960	690	7650
13682846555	1938	2910	4848

# part-r-00001
13729199489	240	0	240
13736230513	2481	24681	27162
13768778790	120	120	240

# part-r-00002
13846544121	264	0	264

# part-r-00003
13956435636	132	1512	1644
13966251146	240	0	240
13975057813	11058	48243	59301
13992314666	3008	3720	6728

# part-r-00004
13470253144	180	180	360
13509468723	7335	110349	117684
13560439638	918	4938	5856
13568436656	3597	25635	29232
13590439668	1116	954	2070
15043685818	3659	3538	7197
15910133277	3156	2936	6092
15959002129	1938	180	2118
18271575951	1527	2106	3633
18390173782	9531	2412	11943
84188413	4116	1432	5548
```

从结果看，符合需求。

##### 注意事项

```
//9 ** 同时指定相应数量的 ReduceTask
job.setNumReduceTasks(5);
```

- 设置`ReduceTask`数量小于分区数时，会报错`Illegal partition ……设置`
- 设置`ReduceTask`数量 = 1 时，结果只会出现一个文件；
- 设置`ReduceTask`数量大于分区数时，会出现空文件。

#### WritableComparable 排序

##### 排序概述

排序是MapReduce框架中最重要的操作之一。

MapTask 和ReduceTask 均会对数据**按照key** 进行排序。该操作属于Hadoop的默认行为。**任何应用程序中的数据均会被排序，而不管逻辑上是否需要**。

默认排序是按照**字典顺序排序**，且实现该排序的方法是**快速排序**。

对于MapTask，它会将处理的结果暂时放到环形缓冲区中，**当环形缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序**，并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会**对磁盘上所有文件进行归并排序**。

对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大
小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到
一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者
数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完
毕后，**ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序**。

##### 排序分类

（1）部分排序

MapReduce根据输入记录的键对数据集排序。保证**输出的每个文件内部有序**。

（2）全排序

**最终输出结果只有一个文件，且文件内部有序**。实现方式是只设置一个ReduceTask。但该方法在处理大型文件时效率极低，因为一台机器处理所有文件，完全丧失了MapReduce所提供的并行架构。

（3）辅助排序：（GroupingComparator分组）

在Reduce端对key进行分组。应用于：在接收的key为bean对象时，想让一个或几个字段相同（全部字段比较不相同）的key进入到同一个reduce方法时，可以采用**分组排序**。

（4）二次排序

在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。

##### 自定义排序 WritableComparable 原理分析 

bean 对象做为 key 传输，需要实现 `WritableComparable` 接口重写 `compareTo` 方法，就可以实现排序。 

```java
@Override 
public int compareTo(FlowBean bean) { 

    int result; 

    // 按照总流量大小，倒序排列 
    if (this.sumFlow > bean.getSumFlow()) { 
        result = -1; 
    }else if (this.sumFlow < bean.getSumFlow()) { 
        result = 1; 
    }else { 
        result = 0; 
    } 

    return result; 
} 
```

#### WritableComparable 排序实例（全排序）

##### 需求

在之前的 [writable案例（不同手机号的流量统计）](#序列化案例（统计手机号的流量）) 的基础上进行全排序（一个文件，倒序排列）。

期望输出数据：

```txt
13509468723 7335 110349 117684 
13736230513 2481 24681 27162 
13956435636 132 1512 1644 
13846544121 264 0 264 
……
```

##### 需求分析 

![全排序需求分析](https://cos.duktig.cn/typora/202110081754324.png)

##### 代码实现

**（1）FlowBean 对象在增加了比较功能** 

```java
public class FlowBean implements WritableComparable<FlowBean> {

    /** 上行流量 */
    private long upFlow;
    /** 下行流量 */
    private long downFlow;
    /** 总流量 */
    private long sumFlow;

    public FlowBean() {
    }

    /**
     * 序列化方法 (序列化和反序列化顺序要一致)
     */
    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeLong(upFlow);
        dataOutput.writeLong(downFlow);
        dataOutput.writeLong(sumFlow);
    }

    /**
     * 反序列化方法 (序列化和反序列化顺序要一致)
     */
    @Override
    public void readFields(DataInput dataInput) throws IOException {
        this.upFlow = dataInput.readLong();
        this.downFlow = dataInput.readLong();
        this.sumFlow = dataInput.readLong();
    }

    /**
     * 重写 ToString
     */
    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }

    @Override
    public int compareTo(FlowBean o) {
        //按照总流量比较,倒序排列
        if (this.sumFlow > o.sumFlow) {
            return - 1;
        } else if (this.sumFlow < o.sumFlow) {
            return 1;
        } else {
            // 按照上行流量的正序排
            if (this.upFlow > o.upFlow) {
                return - 1;
            } else if (this.upFlow < o.upFlow) {
                return 1;
            } else {
                return 0;
            }
        }
    }


}
```

**（2）编写 Mapper 类** 

Mapper类的**输出类型调换了顺序**，保证可以在Reduce阶段对`FlowBean`进行排序处理。

```java
public class FlowMapper extends Mapper<LongWritable, Text, FlowBean, Text> {

    private FlowBean outK = new FlowBean();
    private Text outV = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1 获取一行数据,转成字符串
        String line = value.toString();

        //2 切割数据
        String[] split = line.split("\t");

        //3 封装 outK outV
        outK.setUpFlow(Long.parseLong(split[1]));
        outK.setDownFlow(Long.parseLong(split[2]));
        outK.setSumFlow(Long.parseLong(split[1]) + Long.parseLong(split[2]));
        outV.set(split[0]);

        //4 写出 outK outV
        context.write(outK, outV);
    }

}
```

**（3）编写 Reducer 类** 

Reducer类的**输入类型调换了顺序**，保证可以在Reduce阶段可以接收Map阶段的结果。输出时将K和V进行调换，保持原有的输出逻辑。

```java
public class FlowReducer extends Reducer<FlowBean, Text, Text, FlowBean> {

    @Override
    protected void reduce(FlowBean key, Iterable<Text> values, Context context) throws IOException,
            InterruptedException {
        //遍历 values 集合,循环写出,避免总流量相同的情况
        for (Text value : values) {
            //调换 KV 位置,反向写出
            context.write(value, key);
        }
    }

}
```

**（4）编写 Driver 类** 

Driver 类输入文件为 序列化案例的结果文件。

```java
public class FlowDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        //1 获取 job 对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        //2 关联本 Driver 类
        job.setJarByClass(FlowDriver.class);

        //3 关联 Mapper 和 Reducer
        job.setMapperClass(FlowMapper.class);
        job.setReducerClass(FlowReducer.class);

        //4 设置 Map 端输出 KV 类型 (更换原有类型)
        job.setMapOutputKeyClass(FlowBean.class);
        job.setMapOutputValueClass(Text.class);

        //5 设置程序最终输出的 KV 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //6 设置程序的输入输出路径
        // writablecomparable_all 目录为之前序列化案例的文件
        FileInputFormat.setInputPaths(job, new Path("hadoop-example/src/main/resources/writablecomparable_all_input"));
        FileOutputFormat.setOutputPath(job, new Path("hadoop-example/src/main/resources/writablecomparable_all_output"
        ));

        //7 提交 Job
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);

    }

}
```

结果：

```txt
13509468723	7335	110349	117684
13975057813	11058	48243	59301
13568436656	3597	25635	29232
13736230513	2481	24681	27162
18390173782	9531	2412	11943
13630577991	6960	690	7650
15043685818	3659	3538	7197
13992314666	3008	3720	6728
15910133277	3156	2936	6092
13560439638	918	4938	5856
84188413	4116	1432	5548
13682846555	1938	2910	4848
18271575951	1527	2106	3633
15959002129	1938	180	2118
13590439668	1116	954	2070
13956435636	132	1512	1644
13470253144	180	180	360
13846544121	264	0	264
13729199489	240	0	240
13768778790	120	120	240
13966251146	240	0	240
```

可以看出整体上按照 **总流量降序排序** ，但是最后几个总流量相同时，还是按照key的自然升序排序；如果需要在总流量相同时，按照上行流量排序该怎么办？

##### 二次排序

**二次排序**，只需要更改 FlowBean  的对比方法即可：

```java
@Override
public int compareTo(FlowBean o) {
    //按照总流量比较,倒序排列
    if (this.sumFlow > o.sumFlow) {
        return - 1;
    } else if (this.sumFlow < o.sumFlow) {
        return 1;
    } else {
        // 按照上行流量的正序排
        if (this.upFlow > o.upFlow) {
            return - 1;
        } else if (this.upFlow < o.upFlow) {
            return 1;
        } else {
            return 0;
        }
    }
}
```

结果：
![二次排序结果](https://cos.duktig.cn/typora/202110082015695.png)

可以看出，当总流量为240时，3行相同的数据，按照上行流量倒序排序。

#### WritableComparable 排序实例（区内排序）

##### 需求 
要求每个省份手机号输出的文件中按照总流量内部排序。 

##### 需求分析

 基于前一个需求，增加自定义分区类，分区按照省份手机号设置。

![区内排序需求分析](https://cos.duktig.cn/typora/202110082021047.png) 

##### 代码实现

**（1）增加自定义分区类** 

在Map阶段之后进行分区，所以K/V泛型与Map阶段输出的类型一致。

```java
public class PartProvincePartitioner extends Partitioner<FlowBean, Text> {

    /**
     * 分区逻辑
     */
    @Override
    public int getPartition(FlowBean flowBean, Text text, int i) {
        //获取手机号前三位 prePhone
        String phone = text.toString();
        String prePhone = phone.substring(0, 3);

        //定义一个分区号变量 partition,根据 prePhone 设置分区号
        int partition;

        switch (prePhone) {
            case "136":
                partition = 0;
                break;
            case "137":
                partition = 1;
                break;
            case "138":
                partition = 2;
                break;
            case "139":
                partition = 3;
                break;
            default:
                partition = 4;
                break;
        }

        //最后返回分区号 partition
        return partition;
    }

}

```

**（2）在驱动类中添加分区类** 

```java
//8 ** 指定自定义分区器
job.setPartitionerClass(PartProvincePartitioner.class);

//9 ** 同时指定相应数量的 ReduceTask
job.setNumReduceTasks(5);
```

结果：

```
# part-r-00000
13630577991	6960	690	7650
13682846555	1938	2910	4848

# part-r-00001
13736230513	2481	24681	27162
13729199489	240	0	240
13768778790	120	120	240

# part-r-00002
13846544121	264	0	264

# part-r-00003
13975057813	11058	48243	59301
13992314666	3008	3720	6728
13956435636	132	1512	1644
13966251146	240	0	240

# part-r-00004
13509468723	7335	110349	117684
13568436656	3597	25635	29232
18390173782	9531	2412	11943
15043685818	3659	3538	7197
15910133277	3156	2936	6092
13560439638	918	4938	5856
84188413	4116	1432	5548
18271575951	1527	2106	3633
15959002129	1938	180	2118
13590439668	1116	954	2070
13470253144	180	180	360
```

生成了5个文件，每个文件按照总流量倒序，总流量相同时按照上行流量倒序排序。

#### Combiner 合并 

（1）Combiner是MR程序中Mapper和Reducer之外的一种组件。

（2）Combiner组件的父类就是Reducer。

（3）Combiner和Reducer的区别在于运行的位置

- **Combiner是在每一个MapTask所在的节点运行;**
- **Reducer是接收全局所有Mapper的输出结果；**

（4）Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量。

（5）**Combiner能够应用的前提是不能影响最终的业务逻辑**，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。

（6）**自定义 Combiner 实现步骤** .

- 自定义一个 Combiner 继承 Reducer，重写 Reduce 方法 

```java
public class WordCountCombiner extends Reducer<Text,IntWritable, Text, IntWritable> { 

    private IntWritable outV = new IntWritable(); 

    @Override 
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException,InterruptedException { 

        int sum = 0; 
        for (IntWritable value : values) { 
            sum += value.get(); 
        } 

        outV.set(sum); 

        context.write(key,outV); 
    } 
} 
```

- 在 Job 驱动类中设置： 

```java
job.setCombinerClass(WordCountCombiner.class); 
```

####  Combiner 合并实例

##### 需求 

统计过程中对每一个 MapTask 的输出进行局部汇总，以减小网络传输量即采用
Combiner 功能。 

期望：Combine 输入数据多，输出时经过合并，输出数据降低。 

##### 需求分析 

![Combiner 合并需求分析](https://cos.duktig.cn/typora/202110082047902.png)

##### 代码实现——方案1

**（1）增加一个 WordCountCombiner 类继承 Reducer** 

```java
public class WordCountCombiner extends Reducer<Text, IntWritable, Text, IntWritable> {

    private IntWritable outV = new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context
            context) throws IOException, InterruptedException {

        int sum = 0;
        for (IntWritable value : values) {
            sum += value.get();
        }

        //封装 outKV
        outV.set(sum);
        //写出 outKV
        context.write(key, outV);
    }

}
```

（2）在 WordcountDriver 驱动类中指定 Combiner 

```java
// 指定需要使用 combiner，以及用哪个类作为 combiner 的逻辑 
job.setCombinerClass(WordCountCombiner.class); 
```

##### 代码实现——方案2

（1）将 WordcountReducer 作为 Combiner 在 WordcountDriver 驱动类中指定 

```java
// 指定需要使用 Combiner，以及用哪个类作为 Combiner 的逻辑 
job.setCombinerClass(WordCountReducer.class); 
```

结果：

![Combiner 合并结果](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211008210526769.png)



### OutputFormat 数据输出

#### OutputFormat 接口实现类

`OutputFormat`是MapReduce输出的基类，所有实现MapReduce输出都实现了`OutputFormat`接口。下面我们介绍几种常见的`OutputFormat`实现类。

![OutputFormat实现类](https://cos.duktig.cn/typora/202110082111974.png)

默认输出格式 `TextOutputFormat`。

也可使用`自定义OutputFormat`。

应用场景：

例如：输出数据到MySQL/HBase/Elasticsearch等存储框架中。

自定义OutputFormat步骤

- 自定义一个类继承FileOutputFormat。
- 改写RecordWriter，具体改写输出数据的方法write()。

#### 自定义 OutputFormat 实例

##### 需求

过滤输入的 log 日志，包含 duktig 的网站输出到 resources/OutputFormat_output/duktig .log，不包含duktig 的网站输出到 resources/OutputFormat_output/other.log。 

##### 代码实现

**（1）编写 LogMapper 类** 

```java
public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //不做任何处理,直接写出一行 log 数据
        context.write(value, NullWritable.get());
    }
}
```

**（2）编写 LogReducer 类** 

```java
public class LogReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException,
            InterruptedException {
        // 防止有相同的数据,迭代写出
        for (NullWritable value : values) {
            context.write(key, NullWritable.get());
        }
    }

}
```

**（3）自定义一个 LogOutputFormat 类** 

```java
public class LogOutputFormat extends FileOutputFormat<Text, NullWritable> {

    @Override
    public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext taskAttemptContext) throws IOException
            , InterruptedException {
        //创建一个自定义的 RecordWriter 返回
        return new LogRecordWriter(taskAttemptContext);
    }

}
```

**（4）编写 LogRecordWriter 类** 

```java
public class LogRecordWriter extends RecordWriter<Text, NullWritable> {
    private FSDataOutputStream duktigOut;
    private FSDataOutputStream otherOut;

    public LogRecordWriter(TaskAttemptContext job) {
        try {
            //获取文件系统对象
            FileSystem fs = FileSystem.get(job.getConfiguration());
            //用文件系统对象创建两个输出流对应不同的目录
            duktigOut = fs.create(new Path("hadoop-example/src/main/resources/OutputFormat_output/duktig.log"));
            otherOut = fs.create(new Path("hadoop-example/src/main/resources/OutputFormat_output/other.log"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void write(Text key, NullWritable value) throws IOException, InterruptedException {
        String log = key.toString();
        //根据一行的 log 数据是否包含 duktig,判断两条输出流输出的内容
        if (log.contains("duktig")) {
            duktigOut.writeBytes(log + "\n");
        } else {
            otherOut.writeBytes(log + "\n");
        }
    }

    @Override
    public void close(TaskAttemptContext context) throws IOException, InterruptedException {
        //关流
        IOUtils.closeStream(duktigOut);
        IOUtils.closeStream(otherOut);
    }
}
```

**（5）编写 LogDriver 类** 

```java
public class LogDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(LogDriver.class);
        job.setMapperClass(LogMapper.class);
        job.setReducerClass(LogReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        //设置自定义的 outputformat
        job.setOutputFormatClass(LogOutputFormat.class);

        FileInputFormat.setInputPaths(job, new Path("hadoop-example/src/main/resources/OutputFormat_input"));
        //虽 然 我 们 自 定 义 了 outputformat， 但 是 因 为 我 们 的 outputformat 继 承 自fileoutputformat
        //而 fileoutputformat 要输出一个_SUCCESS 文件，所以在这还得指定一个输出目录
        FileOutputFormat.setOutputPath(job, new Path("hadoop-example/src/main/resources/OutputFormat_output"));

        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }

}
```

##### 结果

初始文件 OutputFormat_input.txt：

```
https://github.com/duktig666
https://duktig.cn
http://duktig.cn
http://www.duktig.cn
http://api.duktig.cn
https://www.baidu.com
https://www.zhihu.com
https://www.google.com
```

结果：

duktig.log

```
http://api.duktig.cn
http://duktig.cn
http://www.duktig.cn
https://duktig.cn
https://github.com/duktig666
```

other.log

```
https://www.baidu.com
https://www.google.com
https://www.zhihu.com
```

需求实现，而且结果也是根据key进行自然升序排序。



## MapReduce深度解析

### MapTask 工作机制

![MapTask 工作机制](https://cos.duktig.cn/typora/202110082153299.png)

（1）Read 阶段：MapTask 通过 InputFormat 获得的 RecordReader，从输入 InputSplit 中解析出一个个 key/value。 

 （2）Map 阶段：该节点主要是将解析出的 key/value 交给用户编写 map()函数处理，并产生一系列新的 key/value。 

 （3）Collect 收集阶段：在用户编写 map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的 key/value 分区（调用Partitioner），并写入一个环形内存缓冲区中。 

 （4）Spill 阶段：即“溢写”，当环形缓冲区满后，MapReduce 会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。 

 溢写阶段详情： 

 步骤 1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition 进行排序，然后按照 key 进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照 key 有序。 

 步骤 2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件 output/spillN.out（N 表示当前溢写次数）中。如果用户设置了 Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。 

 步骤 3：将分区数据的元信息写到内存索引数据结构 SpillRecord 中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过 1MB，则将内存索引写到文件 output/spillN.out.index 中。 

 （5）Merge 阶段：当所有数据处理完成后，MapTask 对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。 当所有数据处理完后，MapTask 会将所有临时文件合并成一个大文件，并保存到文件output/file.out 中，同时生成相应的索引文件 output/file.out.index。  在进行文件合并过程中，MapTask 以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并 mapreduce.task.io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。 让每个 MapTask 最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。 

### ReduceTask 工作机制

![ReduceTask 工作机制](https://cos.duktig.cn/typora/202110082203874.png)

（1）Copy 阶段：ReduceTask 从各个 MapTask 上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。 

 （2）Sort 阶段：在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照 MapReduce 语义，用户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了局部排序，因此，ReduceTask 只需对所有数据进行一次归并排序即可。 

 （3）Reduce 阶段：reduce()函数将计算结果写到 HDFS 上。 

### ReduceTask 并行度决定机制 

回顾：**MapTask 并行度由切片个数决定，切片个数由输入文件和切片规则决定**。 

思考：**ReduceTask 并行度由谁决定**？ 

**1）设置 ReduceTask 并行度（个数）** 

ReduceTask 的并行度同样影响整个 Job 的执行并发度和执行效率，但与 MapTask 的并发数由切片数决定不同，ReduceTask 数量的决定是可以直接手动设置： 

```java
// 默认值是 1，手动设置为 4 
job.setNumReduceTasks(4); 
```

**2）实验：测试 ReduceTask 多少合适** 

（1）实验环境：1 个 Master 节点，16 个 Slave 节点：CPU:8G，内存: 2G 

（2）实验结论： 

![改变ReduceTask](https://cos.duktig.cn/typora/202110090901523.png)

（1）ReduceTask=0，表示没有Reduce阶段，输出文件个数和Map个数一致。

（2）ReduceTask默认值就是1，所以输出文件个数为一个。

（3）如果数据分布不均匀，就有可能在Reduce阶段产生数据倾斜

（4）ReduceTask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个ReduceTask。

（5）具体多少个ReduceTask，需要根据集群性能而定。

（6）如果分区数不是1，但是ReduceTask为1，是否执行分区过程。答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1肯定不执行。

## Join 应用

### Reduce Join 

**Map 端的主要工作**：为来自不同表或文件的 key/value 对，**打标签以区别不同来源的记录**。然后**用连接字段作为 key**，其余部分和新加的标志作为 value，最后进行输出。 

 **Reduce 端的主要工作**：在 Reduce 端**以连接字段作为 key** 的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录（在 Map 阶段已经打标志）分开，最后进行合并就 ok 了。 

### Reduce Join 实例

#### 需求

![Join实例需求](https://cos.duktig.cn/typora/202110090919281.png)

#### 需求分析

通过将关联条件作为 Map 输出的 key，将两表满足 Join 条件的数据并携带数据所来源的文件信息，发往同一个 ReduceTask，在 Reduce 中进行数据的串联。

![Join实例需求分析](https://cos.duktig.cn/typora/202110090921969.png) 

#### 代码实现

（1）创建商品和订单合并后的 TableBean 类 

两个文件的共同字段的bean

```java
public class TableBean implements Writable {

    /** 订单 id */
    private String id;
    /** 产品 id */
    private String pid;
    /** 产品数量 */
    private int amount;
    /** 产品名称 */
    private String name;
    /** 判断是 order 表还是 pd表的标志字段 */
    private String flag;

    public TableBean() {
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(id);
        out.writeUTF(pid);
        out.writeInt(amount);
        out.writeUTF(name);
        out.writeUTF(flag);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.id = in.readUTF();
        this.pid = in.readUTF();
        this.amount = in.readInt();
        this.name = in.readUTF();
        this.flag = in.readUTF();
    }

    @Override
    public String toString() {
        return id + "\t" + name + "\t" + amount;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getPid() {
        return pid;
    }

    public void setPid(String pid) {
        this.pid = pid;
    }

    public int getAmount() {
        return amount;
    }

    public void setAmount(int amount) {
        this.amount = amount;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getFlag() {
        return flag;
    }

    public void setFlag(String flag) {
        this.flag = flag;
    }

}
```

（2）编写 TableMapper 类 

```java
/**
 * description:join合并实战——map阶段处理 order 和 product 数据
 * <p>
 * 以连接字段 pid 作为key
 *
 * <p>
 * order文件（map阶段）：
 * 订单id  pid  数量
 * 1001   01   1
 * 1002   02   2
 * ……
 * <p>
 * product文件（map阶段）：
 * pid  产品名称
 * 01   小米
 * 02   华为
 * ……
 * <p>
 * 合并后的文件（reduce阶段）：
 * 订单id  产品名称  数量
 * 1001   小米   1
 * 1002   华为   2
 * ……
 *
 * @author RenShiWei
 * Date: 2021/10/9 9:30
 * blog: https://duktig.cn/
 * github知识库: https://github.com/duktig666/knowledge
 **/
public class TableMapper extends Mapper<LongWritable, Text, Text, TableBean> {

    private String filename;
    private Text outK = new Text();
    private TableBean outV = new TableBean();

    /**
     * 初始化
     * 如果代码写在 map方法 中，没获取一行都会执行。所以写在这里
     */
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        //获取对应文件名称
        // 得到切片信息
        InputSplit split = context.getInputSplit();
        FileSplit fileSplit = (FileSplit) split;
        filename = fileSplit.getPath().getName();
    }

    /**
     * 处理order和product文件中的数据
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //获取一行
        String line = value.toString();

        //判断是哪个文件,然后针对文件进行不同的操作
        //订单表的处理
        if (filename.contains("order")) {
            String[] split = line.split("\t");
            //封装 outK
            outK.set(split[1]);
            //封装 outV
            outV.setId(split[0]);
            outV.setPid(split[1]);
            outV.setAmount(Integer.parseInt(split[2]));
            outV.setName("");
            outV.setFlag("order");
        } else {                             //商品表的处理
            String[] split = line.split("\t");
            //封装 outK
            outK.set(split[0]);
            //封装 outV
            outV.setId("");
            outV.setPid(split[0]);
            outV.setAmount(0);
            outV.setName(split[1]);
            outV.setFlag("pd");
        }
        //写出 KV
        context.write(outK, outV);
    }

}
```

（3）编写 TableReducer 类 

```java
/**
 * description:join实例——reduce阶段，合并 order 和 product 数据
 * <p>
 * 以连接字段 pid 作为key
 *
 * <p>
 * order文件（map阶段）：
 * 订单id  pid  数量
 * 1001   01   1
 * 1002   02   2
 * ……
 * <p>
 * product文件（map阶段）：
 * pid  产品名称
 * 01   小米
 * 02   华为
 * ……
 * <p>
 * 合并后的文件（reduce阶段）：
 * 订单id  产品名称  数量
 * 1001   小米   1
 * 1002   华为   2
 * ……
 * <p>
 * 到达reduce的数据（两个文件的数据都会 以pid为key 到达reduce阶段）：
 * 1001   01   1
 * 1002   02   2
 * 01   小米
 * 02   华为
 * ……
 *
 * @author RenShiWei
 * Date: 2021/10/9 10:31
 * blog: https://duktig.cn/
 * github知识库: https://github.com/duktig666/knowledge
 **/
public class TableReducer extends Reducer<Text, TableBean, TableBean, NullWritable> {

    @Override
    protected void reduce(Text key, Iterable<TableBean> values, Context context) throws IOException,
            InterruptedException {
        ArrayList<TableBean> orderBeans = new ArrayList<>();
        TableBean pdBean = new TableBean();

        // 相同的key，循环遍历value
        for (TableBean value : values) {
            //判断数据来自哪个表
            if ("order".equals(value.getFlag())) {   //订单表
                //创建一个临时 TableBean 对象接收 value
                TableBean tmpOrderBean = new TableBean();

                /*
                    这里不能使用 orderBeans.add(value);
                    因为 Iterable<TableBean> values 不是Java中的迭代器，是Hadoop提供的，添加的是地址而且会覆盖
                    造成的结果是：最终只会添加一个对象
                    也可以考虑使用 高效一点的 BeanUtils.copy
                 */
                tmpOrderBean.setAmount(value.getAmount());
                tmpOrderBean.setFlag(value.getFlag());
                tmpOrderBean.setId(value.getId());

                //将临时 TableBean 对象添加到集合 orderBeans
                orderBeans.add(tmpOrderBean);
            } else {
                //商品表
                pdBean.setName(value.getName());
                pdBean.setFlag(value.getFlag());
            }
        }

        //遍历集合 orderBeans,替换掉每个 orderBean 的 pid 为 name,然后写出
        for (TableBean orderBean : orderBeans) {
            orderBean.setName(pdBean.getName());
            //写出修改后的 orderBean 对象
            context.write(orderBean, NullWritable.get());
        }
    }
}
```

（4）编写 TableDriver 类 

```java
public class TableDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Job job = Job.getInstance(new Configuration());

        job.setJarByClass(TableDriver.class);
        job.setMapperClass(TableMapper.class);
        job.setReducerClass(TableReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(TableBean.class);

        job.setOutputKeyClass(TableBean.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.setInputPaths(job, new Path("hadoop-example/src/main/resources/join_reduce_input"));
        FileOutputFormat.setOutputPath(job, new Path("hadoop-example/src/main/resources/join_reduce_output"));

        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }

}
```

#### 测试

输入文件

```
# order.txt
1001	01	1
1002	02	2
1003	03	3
1004	01	4
1005	02	5
1006	03	6

# pd.txt
01	小米
02	华为
03	格力
```

结果：

```
1004	小米	4
1001	小米	1
1005	华为	5
1002	华为	2
1006	格力	6
1003	格力	3
```

#### 总结

缺点：这种方式中，合并的操作是在 Reduce 阶段完成，Reduce 端的处理压力太大，Map节点的运算负载则很低，资源利用率不高，且在 Reduce 阶段极易产生数据倾斜。 

解决方案：**Map 端实现数据合并**。 

### Map Join 

#### 使用场景 

Map Join 适用于一张表十分小、一张表很大的场景。 

#### 优点 

思考：**在 Reduce 端处理过多的表，非常容易产生数据倾斜**。怎么办？ 

在 Map 端缓存多张表，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数据的压力，尽可能的减少数据倾斜。 

具体办法：采用 DistributedCache 

 （1）在 Mapper 的 setup 阶段，将文件读取到缓存集合中。 

 （2）在 Driver 驱动类中加载缓存。 

```java
//缓存普通文件到 Task 运行节点（这里用本地文件模拟分布式缓存）
job.addCacheFile(new URI("file:///e:/cache/pd.txt")); 
//如果是集群运行,需要设置 HDFS 路径 
job.addCacheFile(new URI("hdfs://hadoop102:8020/cache/pd.txt"));
```

### Map Join 实例

需求同Reduce Join 实例一致

#### 需求分析

![MapJoin实例-需求分析](https://cos.duktig.cn/typora/202110091124540.png)

#### 代码实现

（1）先在 MapJoinDriver 驱动类中添加缓存文件

```java
public class MapJoinMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    private Map<String, String> pdMap = new HashMap<>();
    private Text outK = new Text();

    /**
     * 初始化——任务开始前将 pd 数据缓存进 pdMap
     * 如果代码写在 map方法 中，没获取一行都会执行。所以写在这里
     */
    @Override
    protected void setup(Context context) throws IOException,
            InterruptedException {

        //通过缓存文件得到小表数据 pd.txt
        URI[] cacheFiles = context.getCacheFiles();
        Path path = new Path(cacheFiles[0]);

        //获取文件系统对象,并打开流
        FileSystem fs = FileSystem.get(context.getConfiguration());
        FSDataInputStream fis = fs.open(path);

        //通过包装流转换为 reader,方便按行读取
        BufferedReader reader = new BufferedReader(new InputStreamReader(fis, StandardCharsets.UTF_8));
        //逐行读取，按行处理
        String line;
        while (StringUtils.isNotEmpty(line = reader.readLine())) {
            //切割一行： 01 小米
            String[] split = line.split("\t");
            pdMap.put(split[0], split[1]);
        }

        //关流
        IOUtils.closeStream(reader);
    }

    @Override
    protected void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {

        //读取大表数据： 1001 01 1
        String[] fields = value.toString().split("\t");

        //通过大表每行数据的 pid,去 pdMap 里面取出 pname
        String name = pdMap.get(fields[1]);

        //将大表每行数据的 pid 替换为 name
        outK.set(fields[0] + "\t" + name + "\t" + fields[2]);

        //写出
        context.write(outK, NullWritable.get());
    }

}
```

（2）在 MapJoinMapper 类中的 setup 方法中读取缓存文件 

```java
public class MapJoinDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException,
            URISyntaxException {
        // 1 获取 job 信息
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        // 2 设置加载 jar 包路径
        job.setJarByClass(MapJoinDriver.class);
        // 3 关联 mapper
        job.setMapperClass(MapJoinMapper.class);
        // 4 设置 Map 输出 KV 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);
        // 5 设置最终输出 KV 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        // ** 加载缓存数据
        job.addCacheFile(new URI("hadoop-example/src/main/resources/join_map/cache/pd.txt"));
        // ** Map 端 Join 的逻辑不需要 Reduce 阶段，设置 reduceTask 数量为 0
        job.setNumReduceTasks(0);

        // 6 设置输入输出路径
        FileInputFormat.setInputPaths(job, new Path("hadoop-example/src/main/resources/join_map/input"));
        FileOutputFormat.setOutputPath(job, new Path("hadoop-example/src/main/resources/join_map/output"));
        // 7 提交
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }

}
```

#### 测试

结果：

```
1001	小米	1
1002	华为	2
1003	格力	3
1004	小米	4
1005	华为	5
1006	格力	6
```

没有reduce阶段，小文件从缓存读取与大文件直接在Map阶段处理，提高效率。

## 数据清洗（ETL）

“ETL，是英文 Extract-Transform-Load 的缩写，用来描述将数据从来源端经过抽取（Extract）、转换（Transform）、加载（Load）至目的端的过程。ETL 一词较常用在数据仓库，但其对象并不限于数据仓库 。

在运行核心业务 MapReduce 程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。**清理的过程往往只需要运行 Mapper 程序，不需要运行 Reduce 程序**。 

## MapReduce 总结 

1、**输入数据接口：InputFormat**

- 默认的实现类是`TextInputformat`一次读一行文本， key：该行的起始偏移量，value :一行内容
- 处理小文件`CombineTextInputFormat` 把多个小文件合并成一个切片处理，提高处理效率。 

**2、逻辑处理接口：Mapper**

- `setup()`初始化；  
- `map()`用户的业务逻辑； 
- `clearup() `关闭资源；

**3、分区：Partitioner**

- 默认分区`HashPartitioner` ，默认按照 key的hash值 和 numReduces返回分区号：`key.hashCode()&Integer.MAXVALUE % numReduces `
- 如果业务上有特别的需求，可以自定义分区

**4、排序：WritableComparable**

当我们用**自定义的对象作为 key** 来输出时，就必须要实现 `WritableComparable `接口，重写其中的 `compareTo()`方法。

- 部分排序 ：每个输出的文件内部有序。
- 全排序：  对所有数据进行排序，通常只有一个 Reduce。
- 自定义排序 ：实现 writableCompare接口， 重写compareTo方法

**5、合并 ：Combiner** 

Combiner 合并可以提高程序执行效率，减少 IO 传输。

- 前提：不影响最终的业务逻辑（求和 没问题   求平均值）
- 提前聚合map  => 解决数据倾斜的一个方法

**6、逻辑处理接口：Reducer**

- `setup()`初始化；
- `reduce()`用户的业务逻辑；
-  `clearup()` 关闭资源；

**7、输出数据接口：OutputFormat**

- 默认实现类是 TextOutputFormat，功能逻辑是：将每一个 KV 对，向目标文本文件输出一行。 
- 自定义

##  MapReduce数据压缩

### 压缩概述

**压缩的优缺点**

- 压缩的优点：以减少磁盘 IO、减少磁盘存储空间。 
- 压缩的缺点：增加 CPU 开销。 

**压缩原则**

- 运算密集型的 Job，少用压缩 
- IO 密集型的 Job，多用压缩 

### MapReduce支持的压缩编码

#### 压缩算法对比介绍

| 压缩格式 | Hadoop自带？     | 算法    | 文件扩展名 | 是否可切片 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ---------------- | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用     | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用     | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用     | bzip2   | .bz2       | **是**     | 和文本处理一样，不需要修改             |
| LZO      | **否，需要安装** | LZO     | .lzo       | **是**     | **需要建索引，还需要指定输入格式**     |
| Snappy   | 是，直接使用     | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

#### 压缩性能的比较

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

>
> Snappy is a compression/decompression library. It **does not aim for maximum compression**, or compatibility with any other compression library; instead, it **aims for very high speeds** and reasonable compression. For instance, compared to the fastest mode of zlib, Snappy is an order of magnitude faster for most inputs, but the resulting compressed files are anywhere from 20% to 100% bigger.On a single core of a Core i7 processor in 64-bit mode, Snappy **compresses** at about **250 MB/sec** or more and **decompresses** at about **500 MB/sec** or more. 
>
> [http://google.github.io/snappy/](http://google.github.io/snappy/ ) 
>
> snappy的压缩和解压缩的效率极高

### 压缩方式选择

压缩方式选择时重点考虑：**压缩/解压缩速度、压缩率（压缩后存储大小）、压缩后是否可以支持切片**。 

**Gzip 压缩**

优点：压缩率比较高； 

缺点：不支持 Split；压缩/解压速度一般； 

**Bzip2 压缩**

优点：压缩率高；支持 Split； 

缺点：压缩/解压速度慢。 

**Lzo 压缩**

优点：压缩/解压速度比较快；支持 Split； 

缺点：压缩率一般；想支持切片需要额外创建索引。 

**Snappy 压缩**

优点：压缩和解压缩速度快；


缺点：不支持 Split；压缩率一般； 

#### 压缩位置选择

压缩可以在 MapReduce 以下三个阶段进行：

![MapReduce压缩位置选择](https://cos.duktig.cn/typora/202110091500760.png)



### 压缩参数配置

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

2）要在Hadoop中启用压缩，可以配置如下参数

| 参数                                                         | 默认值                                         | 阶段        | 建议                                          |
| ------------------------------------------------------------ | ---------------------------------------------- | ----------- | --------------------------------------------- |
| io.compression.codecs    （在core-site.xml中配置）           | 无，这个需要在命令行输入hadoop checknative查看 | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器  |
| mapreduce.map.output.compress（在mapred-site.xml中配置）     | false                                          | mapper输出  | 这个参数设为true启用压缩                      |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec     | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） | false                                          | reducer输出 | 这个参数设为true启用压缩                      |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec     | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2       |

### 压缩的使用

#### Map 输出端采用压缩

即使你的 MapReduce 的输入输出文件都是未压缩的文件，你仍然可以对 Map 任务的中间结果输出做压缩，因为它要写在硬盘并且通过网络传输到 Reduce 节点，对其压缩可以提高很多性能，这些工作只要设置两个属性即可。

在Driver 中配置，代码如下：

```java
Configuration conf = new Configuration(); 
 
// 开启 map 端输出压缩 
conf.setBoolean("mapreduce.map.output.compress", true); 
 
// 设置 map 端输出压缩方式 
conf.setClass("mapreduce.map.output.compress.codec", 
BZip2Codec.class,CompressionCodec.class); 
```

Mapper和Reducer不变

因为只在Map输出端压缩，在Reduce输入端会自动解压缩，Reduce输出端未压缩，所以最终结果并不存在压缩文件。

#### Reduce 输出端采用压缩

在Driver 中配置，代码如下：

```java
// 设置 reduce 端输出压缩开启 
FileOutputFormat.setCompressOutput(job, true); 
 
 // 设置压缩的方式 
FileOutputFormat.setOutputCompressorClass(job,BZip2Codec.class); 
```

Mapper 和 Reducer 保持不变。

最终Reduce使用什么压缩方式，会产生对应的压缩格式文件。

## MapReduce 常见错误及解决方案 

1、Mapper 中第一个输入的参数必须是 `LongWritable` 或者 `NullWritable`，不可以是 `IntWritable`。报的错误是类型转换异常。 

2、`java.lang.Exception: java.io.IOException: Illegal partition for 13926435656 (4)`

说明 Partition和 ReduceTask 个数没对上，调整 ReduceTask 个数。 

3、如果分区数不是 1，但是 reducetask 为 1，是否执行分区过程。答案是：不执行分区过程。因为在 MapTask 的源码中，执行分区的前提是先判断 ReduceNum 个数是否大于 1。不大于1 肯定不执行。

4、报类型转换异常。 通常都是在驱动函数中设置 Map 输出和最终输出时编写错误。 Map 输出的 key 如果没有排序，也会报类型转换异常。 

5、自定义 `Outputformat` 时，注意在 `RecordWirter` 中的 `close` 方法必须关闭流资源。否则输出的文件内容中数据为空。 

```java
@Override 
public void close(TaskAttemptContext context) throws IOException, 
InterruptedException { 
    if (atguigufos != null) { 
        atguigufos.close(); 
    } 
    if (otherfos != null) { 
        otherfos.close(); 
    } 
} 
```



# Yarn 资源调度器

## Yarn 概述

思考：

1. 如何管理集群资源？
2. 如何给任务合理分配资源？

**Yarn**是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的**操作系统平台**，而 **MapReduce** 等运算程序则相当于**运行于操作系统之上的应用程序**。

## Yarn 基础架构

YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等组件构成。

![Yarn 基础架构](https://cos.duktig.cn/typora/202110091704417.png)



## Yarn工作机制

![Yarn 工作机制](https://cos.duktig.cn/typora/202110091707853.png)

详细流程：

（1）MapReduce 程序提交到客户端所在的节点。

（2）YarnRunner向ResourceManager申请一个Application。

（3）RM将该应用程序的资源路径返回给YarnRunner。

（4）该程序将运行所需资源提交到HDFS上。

（5）程序资源提交完毕后，申请运行mrAppMaster。

（6）RM将用户的请求初始化成一个Task。

（7）其中一个NodeManager领取到Task任务。

（8）该NodeManager创建容器Container，并产生MRAppmaster。

（9）Container从HDFS上拷贝资源到本地。

（10）MRAppmaster向RM 申请运行MapTask资源。

（11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

（12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

（13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

（14）ReduceTask向MapTask获取相应分区的数据。

（15）程序运行完毕后，MR会向RM申请注销自己。



## 作业提交全过程

### HDFS、YARN、MapReduce三者关系

![HDFS、YARN、MapReduce三者关系](https://cos.duktig.cn/typora/202110091711699.png)



### 作业提交过程之YARN

参看上文：[Yarn工作机制](#Yarn工作机制)



### 作业提交过程之HDFS & MapReduce

![作业提交过程之HDFS & MapReduce](https://cos.duktig.cn/typora/202110091713573.png)



### 作业提交全过程详解

（1）作业提交

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

（2）作业初始化

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

（3）任务分配

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

（4）任务运行

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

（5）进度和状态更新

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

（6）作业完成

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

## Yarn调度器和调度算法

目前，Hadoop作业调度器主要有三种：FIFO、容量（Capacity Scheduler）和公平（Fair Scheduler）。

- Apache Hadoop3.1.3默认的资源调度器是Capacity Scheduler。
- CDH框架默认调度器是Fair Scheduler。

具体设置详见：yarn-default.xml文件

```xml
<property>

    <description>The class to use as the resource scheduler.</description>

    <name>yarn.resourcemanager.scheduler.class</name>

    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>

</property>
```

### 先进先出调度器（FIFO）

FIFO调度器（First In First Out）：单队列，根据提交作业的先后顺序，先来先服务。

![FIFO调度器](https://cos.duktig.cn/typora/202110091716045.png)

优点：简单易懂；

缺点：不支持多队列，生产环境很少使用；

### 容量调度器（Capacity Scheduler）

Capacity Scheduler是 Yahoo 开发的多用户调度器。

![容量调度器（Capacity Scheduler）](https://cos.duktig.cn/typora/202110091719091.png)

特点：

1、**多队列**：每个队列可配置一定的资源量，**每个队列采用FIFO调度策略**。

2、容量保证：管理员可为每个队列设置资源最低保证和资源使用上限

3、灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列。

4、多租户：

- 支持多用户共享集群和多应用程序同时运行。
- 为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。

**容量调度器资源分配算法**：

![容量调度器资源分配算法](https://cos.duktig.cn/typora/202110091722827.png)

### 公平调度器（Fair Scheduler）

#### 特点

Fair Schedulere是Facebook开发的多用户调度器。

![image-20211009172355581](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211009172355581.png)

1）与容量调度器相同点

（1）多队列：支持多队列多作业

（2）容量保证：管理员可为每个队列设置资源最低保证和资源使用上线

（3）灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列。

（4）多租户：支持多用户共享集群和多应用程序同时运行；为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。

2）与容量调度器不同点

（1）核心调度策略不同

容量调度器：优先选择资源利用率低的队列

公平调度器：优先选择对资源的缺额比例大的

（2）每个队列可以单独设置资源分配方式

容量调度器：FIFO、DRF

公平调度器：FIFO、FAIR、DRF

#### 公平调度器——缺额

![公平调度器——缺额](https://cos.duktig.cn/typora/202110091726367.png)

公平调度器设计目标是：在时间尺度上，所有作业获得公平的资源。

- 某一时刻一个作业应获资源和实际获取资源的差距叫“缺额”
- 调度器会优先为缺额大的作业分配资源。

#### 公平调度器队列资源分配方式

![公平调度器队列资源分配方式](C:\Users\rsw\Pictures\公平调度器队列资源分配方式.png)

**公平调度器资源分配算法**

**分配额度**

```
root
|---queueA 20%
|---queueB 50%
|---queueC 30%
	|---ss   50%
	|---cls 50%
```

**（1）队列资源分配**

需求：集群总资源100，有三个队列，对资源的需求分别是：

queueA -> 20，queueB ->50，queueC -> 30

```
第一次算：100 / 3 = 33.33
queueA：分33.33 → 多13.33
queueB：分33.33 → 少16.67
queueC：分33.33 → 多3.33
第二次算：（13.33 + 3.33）/ 1 = 16.66
queueA：分20
queueB：分33.33 + 16.66 = 50
queueC：分30
```

![队列资源分配](https://cos.duktig.cn/typora/202110091730650.png)

**（2）作业资源分配**

![作业资源分配](https://cos.duktig.cn/typora/202110091731149.png)

**3）DRF策略**

DRF（Dominant Resource Fairness），我们之前说的资源，都是单一标准，例如只考虑内存（也是Yarn默认的情况）。但是很多时候我们资源有很多种，例如内存，CPU，网络带宽等，这样我们很难衡量两个应用应该分配的资源比例。

那么在YARN中，我们用DRF来决定如何调度：

假设集群一共有100 CPU和10T 内存，而应用A需要（2 CPU, 300GB），应用B需要（6 CPU，100GB）。则两个应用分别需要A（2%CPU, 3%内存）和B（6%CPU, 1%内存）的资源，这就意味着A是内存主导的, B是CPU主导的，针对这种情况，我们可以选择DRF策略对不同应用进行不同资源（CPU和内存）的一个不同比例的限制。

### Yarn的调度器总结

1. 调度器：FIFO/容量/公平
2. apache 默认调度器  容量； CDH默认调度器 公平
3. 公平/容量调度器，默认一个default队列 ，需要创建多队列
4. 多队列选择（创建）
   1. 中小企业：hive  spark flink  mr  
   2. 中大企业：业务模块：登录/注册/购物车/营销
5. 队列好处：解耦  降低风险  11.11  6.18  降级使用=
6. 每个调度器特点：
   1. 相同点：支持多队列，可以借资源，支持多用户
   2. 不同点：
      1. 容量调度器：优先满足先进来的任务执行
      2. 公平调度器，在队列里面的任务公平享有队列资源
7. 生产环境怎么选调度器：
   1. 中小企业，对并发度要求不高，选择容量
   2. 中大企业，对并发度要求比较高，选择公平。

## Yarn 常用命令 

```bash
# 查看任务 yarn application
## 列出所有 Application
yarn application -list 
## 根据 Application 状态过滤：yarn application -list -appStates （所有状态：ALL、NEW、NEW_SAVING、SUBMITTED、ACCEPTED、RUNNING、FINISHED、FAILED、KILLED） 
yarn application -list -appStates <appStates>
## Kill 掉 Application
yarn application -kill <application_id>

# 查看日志 yarn logs
## 查询 Application 日志
yarn logs -applicationId <ApplicationId> 
## 查询 Container 日志
yarn logs -applicationId <ApplicationId> -containerId <ContainerId> 

# 查看尝试运行的任务 yarn applicationattempt
## 列出所有 Application 尝试的列表
yarn applicationattempt -list <ApplicationId> 
## 打印 ApplicationAttemp 状态
yarn applicationattempt -status <ApplicationAttemptId> 

# 查看容器 yarn container
## 列出所有 Container
yarn container -list <ApplicationAttemptId> 
## 打印 Container 状态
yarn container -status <ContainerId> 
 
# 查看节点状态  yarn node
## 列出所有节点
yarn node -list -all 

# 更新配置 yarn rmadmin 
## 加载队列配置
yarn rmadmin -refreshQueues 

# 查看队列 yarn queue 
## 打印队列信息
yarn queue -status <QueueName>
```

