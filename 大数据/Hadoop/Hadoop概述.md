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

#### 什么是HDFS？

Hadoop Distributed File System，简称 **HDFS**，是一个**分布式文件系统**。 用于存储文件，通过目录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。 

> **分布式文件系统产生的背景**
>
> 随着数据量越来越大，在一个操作系统存不下所有的数据，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，迫切**需要一种系统来管理多台机器上的文件**，这就是分布式文件管理系统。

Hadoop Distributed File System，简称 **HDFS**，是一个**分布式文件系统**。 

- NameNode（nn）：**存储文件的元数据**，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。**用来标识数据都存储在什么位置**。
- DataNode(dn)：在本地文件系统**存储文件块数据**，以及**块数据的校验和**。
- Secondary NameNode(2nn)：每隔一段时间**对NameNode元数据备份**。

#### HDFS的使用场景

**适合一次写入，多次读出的场景**。一个文件经过创建、写入和关闭之后就不需要改变。 

#### HDFS的优缺点

##### 优点

- **高容错性**
  - 数据自动保存多个副本。它通过增加副本的形式，提高容错性。
  - 某一个副本丢失以后，它可以自动恢复。
- **适合处理大数据**
  - 数据规模：能够处理数据规模达到GB、TB、甚至**PB级别的数据**；
  - 文件规模：能够处理百万规模以上的文件数量，数量相当之大。
- 可**构建在廉价机器上**，通过多副本机制，提高可靠性。

##### 缺点

- **不适合低延时数据访问**，比如毫秒级的存储数据，是做不到的。
- **无法高效的对大量小文件进行存储**
  - 存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。这样是不可取的，因为NameNode的内存总是有限的；
  - 小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。
- **不支持并发写入、文件随机修改**
  - 一个文件只能有一个写，不允许多个线程同时写
  - 仅支持数据append（追加），不支持文件的随机修改

### YARN

#### 什么是Yarn？

**Yarn**是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的**操作系统平台**，而 **MapReduce** 等运算程序则相当于**运行于操作系统之上的应用程序**。

Yet Another Resource Negotiator 简称 YARN ，另一种资源协调者，是 Hadoop 的**资源管理器**。

1）ResourceManager（RM）：管理整个集群资源（内存、CPU等）

3）ApplicationMaster（AM）：管理单个任务运行

2）NodeManager（NM）：管理单个节点服务器资源

4）Container：容器，相当一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。

![YARN架构](https://cos.duktig.cn/typora/202110061050089.png)

说明1：客户端可以有多个

说明2：集群上可以运行多个ApplicationMaster

说明3：每个NodeManager上可以有多个Container

#### Yarn主要解决的问题

- 如何管理集群资源？
- 如何给任务合理分配资源？

### MapReduce

#### 什么是MapReduce？

MapReduce 是一个**分布式运算程序**的编程框架，是用户开发“基于 Hadoop 的数据分析应用”的核心框架。 

MapReduce 核心功能是 **将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上** 。 

MapReduce 将计算过程分为两个阶段：Map 和 Reduce 
1）Map 阶段并行处理输入数据 
2）Reduce 阶段对 Map 结果进行汇总 

![MapReduce架构](https://cos.duktig.cn/typora/202110061058676.png)

### MapReduce 的使用场景

核心思想：**如何把大问题分解成独立的小问题，再并行解决**。

典型场景：

**计算`URL`的访问频率**：搜索引擎的使用中，会遇到大量的`URL`的访问，所以，可以使用 `MapReduce` 来进行统计，得出（`URL`,次数）结果，在后续的分析中可以使用。

**Top K 问题**：在各种的文档分析，或者是不同的场景中，经常会遇到关于 `Top K` 的问题，例如输出这篇文章的出现前`5`个最多的词汇。这个时候也可以使用 `MapReduce`来进行统计。

### MapReduce优缺点

#### 优点

1、**易于编程**： 用户只关心业务逻辑。 实现框架的接口。
2、**良好扩展性**：可以动态增加服务器，解决计算资源不够问题。
3、**高容错性**：任何一台机器挂掉，可以将任务转移到其他节点。
4、**适合海量数据计算**：（TB/PB） 几千台服务器共同计算。

#### 缺点

1、**不擅长实时计算。 Mysql**（在毫秒或者秒级内返回结果）
2、**不擅长流式计算。 Spark Streaming | flink 。**流式计算的输入数据是动态的，而 MapReduce 的输入数据集是静态的，不能动态变化。
3、**不擅长DAG有向无环图计算。spark 。** 多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce 并不是不能做，而是使用后，每个 MapReduce 作业的输出结果都会写入到磁盘，会造成大量的磁盘 IO，导致性能非常的低下。 

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