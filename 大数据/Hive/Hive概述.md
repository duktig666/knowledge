> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 更多文章参看github知识库：[https://github.com/duktig666/knowledge](https://github.com/duktig666/knowledge)

# 背景

学习完Hadoop，有没有感到编写一个MapReduce程序非常复杂，想要进行一次分析和统计需要很大的开发成本。那么不如就来了解了解Hadoop生态圈的另一名成员——Hive。让我们一起来了解，如何使用类SQL语言进行快速查询和分析数据吧。

Hive系列文章如下：

- [大数据基础之Hive（一）—— Hive概述](https://blog.csdn.net/qq_42937522/article/details/121096763?spm=1001.2014.3001.5501)
- [大数据基础之Hive（二）—— DDL语句和DML语句](https://blog.csdn.net/qq_42937522/article/details/121096833?spm=1001.2014.3001.5501)
- [大数据基础之Hive（三）—— 分区表和分桶表](https://blog.csdn.net/qq_42937522/article/details/121096891?spm=1001.2014.3001.5501)
- [大数据基础之Hive（四）—— 常用函数和压缩存储](https://blog.csdn.net/qq_42937522/article/details/121096983?spm=1001.2014.3001.5501)
- [大数据基础之Hive（五）——Hive实战（统计电影排名的各种问题）](https://blog.csdn.net/qq_42937522/article/details/121097029)

# Hive概述

## 什么是 Hive ？

### Hive简介

Hive：由Facebook 开源用于解决 **海量结构化日志的数据统计** 的工具。 

Hive 是基于 Hadoop 的一个 **数据仓库工具**，可以 **将结构化的数据文件映射为一张表，并提供类 SQL 查询功能**。 

### Hive本质

**将 HQL 转化成MapReduce 程序** 。主要如下：

- Hive 处理的数据存储在 HDFS 
- Hive 分析数据底层的实现是 MapReduce 
- 执行程序运行在 Yarn 上 

![将 HQL 转化成MapReduce 程序](https://cos.duktig.cn/typora/202111021035124.png)

## Hive 的优缺点

### 优点

- 操作接口采用 **类 SQL 语法**，提供快速开发的能力（简单、容易上手）。 
- **避免了去写 MapReduce**，减少开发人员的学习成本。 
- Hive可以处理大数据量。
- Hive 支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。 

### 缺点

- Hive 的 HQL 表达能力有限 
  - 迭代式算法无法表达 
  - 数据挖掘方面不擅长，由于 MapReduce 数据处理流程的限制，效率更高的算法却无法实现。 
- Hive 的效率比较低 
  - Hive 自动生成的 MapReduce 作业，通常情况下不够智能化 
  - Hive 调优比较困难，粒度较粗 
- Hive的执行延迟比较高。

## Hive 的使用场景

因为Hive 的执行延迟比较高，所以

1. Hive 常用于数据分析，对实时性要求不高的场合。
2. Hive 优势在于处理大数据，对于处理小数据没有优势。 

## Hive的架构

![Hive架构](https://cos.duktig.cn/typora/202111021049951.png)

- **用户接口Client**： CLI（command-line interface）、JDBC/ODBC(jdbc 访问 hive)、WEBUI（浏览器访问hive）
- **元数据Metastore**：元数据包括：表名、表所属的数据库（默认是 default）、表的拥有者、列/分区字段、
  表的类型（是否是外部表）、表的数据所在目录等； （默认存储在自带的 derby 数据库中，推荐使用 MySQL 存储Metastore ）
- Hadoop：使用HDFS 进行存储，使用 MapReduce 进行计算，运行在Yarn上。
- 驱动器Driver 
  - 解析器（SQL Parser）：将SQL 字符串转换成抽象语法树 AST，这一步一般都用第三方工具库完成，比如 antlr；对AST 进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。 
  - 编译器（Physical Plan）：将AST 编译生成逻辑执行计划。 
  - 优化器（Query Optimizer）：对逻辑执行计划进行优化。 
  - 执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。 

**Hive运行机制**：

![Hive运行机制](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211102105231171.png)

Hive 通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的 Driver，结合元数据(MetaStore)，将这些指令翻译成 MapReduce，提交到Hadoop 中执行，最后，将执行返回的结果输出到用户交互接口。 

## Hive 和数据库比较

由于 Hive 采用了类似 SQL 的查询语言 HQL(Hive Query Language)，因此很容易将 Hive 理解为数据库。从结构上来看，Hive 和数据库除了拥有类似的查询语言，再无类似之处。数据库可以用在 Online 的应用中，但是Hive 是为数据仓库而设计的。

### 查询语言

由于 SQL 被广泛的应用在数据仓库中，因此，专门针对 Hive 的特性设计了类 SQL 的查询语言HQL。熟悉 SQL 开发的开发者可以很方便的使用Hive 进行开发。 

### 数据更新 

由于Hive 是针对数据仓库应用设计的，而**数据仓库的内容是读多写少的**。因此，**Hive 中不建议对数据的改写，所有的数据都是在加载的时候确定好的**。

而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO … VALUES 添加数据，使用 UPDATE … SET 修改数据。 

### 执行延迟 

Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。

另外一个导致 Hive 执行延迟高的因素是 MapReduce 框架。由于MapReduce 本身具有较高的延迟，因此在利用MapReduce 执行Hive 查询时，也会有较高的延迟。

相对的，数据库的执行延迟较低。

当数据规模大到超过数据库的处理能力的时候，Hive 的并行计算显然能体现出优势。 

### 数据规模 

由于Hive 建立在集群上并可以利用 MapReduce 进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。 

# win10安装Hive3.x

win10安装Hive

参看：[win10安装Hive3.0.0](https://blog.csdn.net/qq262593421/article/details/104961689)

注意：在修改 **hive-site.xml** 时有问题。

官网给出的文件如下：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at
       http://www.apache.org/licenses/LICENSE-2.0
   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->

<configuration>

</configuration>
```

注意将 `property` 属性添加到 `configuration` 标签内。

jdbc连接属性参考：

```mysql
jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true&amp;characterEncoding=latin1&amp;useSSL=false&amp;serverTimezone=Asia/Shanghai
```

hive的启动

```sh
# 启动 Hive 元数据
hive --service metastore
# 启动 Hive server2 服务
hive --service hiveserver2
# 启动 hive 命令行
hive
```



# 问题

## Hive进行drop时终端卡死？

[hive删除表时直接卡死](https://www.cnblogs.com/birdmmxx/p/11845988.html)

## hive show locks异常问题

报错：

```sh
ERROR exec.DDLTask: Failed
org.apache.hadoop.hive.ql.metadata.HiveException: show Locks LockManager not specified
```

解决：

$HIVE_HOME/conf/hive-site.xml中加入:

```xml
<property>
    <name>hive.support.concurrency</name>
    <value>true</value>
</property>


<property>
    <name>hive.txn.manager</name>
    <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
```

然后重启metastore和hiveserver2。

卡死hive的metastore报错：

```xml
<!--元数据存储授权--> 
<property> 
    <name>hive.metastore.event.db.notification.api.auth</name> 
    <value>false</value> 
</property>
```

## Hive中文乱码问题

在mysql的hive元数据库中执行如下代码：

```sql
-- 修改表字段注释字符集
ALTER TABLE COLUMNS_V2 MODIFY COLUMN `COMMENT` varchar(256) CHARACTER SET utf8;
-- 修改表字段名字符集
ALTER TABLE COLUMNS_V2 MODIFY COLUMN `COLUMN_NAME` varchar(767) CHARACTER SET utf8;

-- 修改表属性Key和Value字符集
ALTER TABLE TABLE_PARAMS MODIFY COLUMN `PARAM_VALUE` varchar(4000) CHARACTER SET utf8;
ALTER TABLE TABLE_PARAMS MODIFY COLUMN `PARAM_KEY` varchar(256) CHARACTER SET utf8;

-- 修改分区属性Key和Value字符集
ALTER TABLE PARTITION_PARAMS MODIFY COLUMN `PARAM_KEY` varchar(256) CHARACTER SET utf8;
ALTER TABLE PARTITION_PARAMS MODIFY COLUMN `PARAM_VALUE` varchar(4000) CHARACTER SET utf8;
-- 修改分区字段Key和Value字符集
ALTER TABLE PARTITION_KEYS MODIFY COLUMN `PKEY_COMMENT` varchar(4000) CHARACTER SET utf8;
ALTER TABLE PARTITION_KEY_VALS MODIFY COLUMN `PART_KEY_VAL` varchar(256) CHARACTER SET utf8;
-- 修改分区的分区名字符集
ALTER TABLE `PARTITIONS` MODIFY COLUMN `PART_NAME` varchar(767) CHARACTER SET utf8;

-- 修改索引属性Key和Value字符集
ALTER TABLE INDEX_PARAMS MODIFY COLUMN `PARAM_KEY` varchar(256) CHARACTER SET utf8;
ALTER TABLE INDEX_PARAMS MODIFY COLUMN `PARAM_VALUE` varchar(4000) CHARACTER SET utf8;
```

修改hive-site.xml中JDBC的连接编码为utf8

# Hive实战

建表：

```sql
create table test( 
name string, 
friends array<string>, 
children map<string, int>, 
address struct<street:string, city:string> 
) 
row format delimited fields terminated by ',' 
collection items terminated by '_' 
map keys terminated by ':' 
lines terminated by '\n';
```

表的数据test.txt：

```sql
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long 
guan_beijing 
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing 
```

上传表的数据：

```sh
load data local inpath './test.txt' overwrite into table test; 
```

查询数据：

```sql
select friends[1],children['xiao song'],address.city from 
test 
where name="songsong";
```

具体的DDL和DML语句将在下篇文章分析。