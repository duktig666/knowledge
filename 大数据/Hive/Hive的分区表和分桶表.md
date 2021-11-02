> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 更多文章参看github知识库：[https://github.com/duktig666/knowledge](https://github.com/duktig666/knowledge)

# 背景

学习完Hadoop，有没有感到编写一个MapReduce程序非常复杂，想要进行一次分析和统计需要很大的开发成本。那么不如就来了解了解Hadoop生态圈的另一名成员——Hive。让我们一起来了解，如何使用类SQL语言进行快速查询和分析数据吧。

前边文章我们了解了Hive的概述、DDL语句和**DML语句（重点）**，这篇文章主要了解Hive的分桶表和分区表。

Hive系列文章如下：

- [大数据基础之Hive（一）—— Hive概述](https://blog.csdn.net/qq_42937522/article/details/121096763?spm=1001.2014.3001.5501)
- [大数据基础之Hive（二）—— DDL语句和DML语句](https://blog.csdn.net/qq_42937522/article/details/121096833?spm=1001.2014.3001.5501)
- [大数据基础之Hive（三）—— 分区表和分桶表](https://blog.csdn.net/qq_42937522/article/details/121096891?spm=1001.2014.3001.5501)
- [大数据基础之Hive（四）—— 常用函数和压缩存储](https://blog.csdn.net/qq_42937522/article/details/121096983?spm=1001.2014.3001.5501)
- [大数据基础之Hive（五）——Hive实战（统计电影排名的各种问题）](https://blog.csdn.net/qq_42937522/article/details/121097029)Hive系列文章如下：

# 分区表

分区表实际上就是对应一个 HDFS 文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。**Hive 中的分区就是分目录**，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过 WHERE 子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。 

## 创建分区表

```sql
create table dept_partition( 
deptno int, dname string, loc string
) 
partitioned by (day string) 
row format delimited fields terminated by ' ';  
```

### 数据准备——引入分区表（需要根据日期对日志进行管理, 通过部门信息模拟）

dept_20211027.log

```tex
10 ACCOUNTING 1700
20 RESEARCH 1800
```

dept_20211028.log

```tex
30 SALES 1900
```

dept_20211029.log

```tcl
40 OPERATIONS 1700
50 DK 1500
60 GEN 1200
```

### 加载数据到分区表

```sql
load data local inpath 
'./dept_20211027.log' into table dept_partition 
partition(day='20211027');

load data local inpath 
'./dept_20211028.log' into table dept_partition 
partition(day='20211028');

load data local inpath 
'./dept_20211029.log' into table dept_partition 
partition(day='20211029');
```

注意：分区表加载数据时，必须指定分区 

![分区表加载数据](https://cos.duktig.cn/typora/202110292207675.png)

## 分区表基本操作

### 查询分区表中数据

```sql
# 单分区查询 
select * from dept_partition where day='20211027'; 
# 多分区联合查询 
select * from dept_partition where day='20211027' 
      union 
      select * from dept_partition where day='20211028' 
      union 
      select * from dept_partition where day='20211029'; 
## 两个操作结果相同，这个更快              
select * from dept_partition where day='20211027' or day='20211028' or day='20211029';  
```

### 增加分区

```sql
alter table dept_partition add partition(day='20211030');
alter table dept_partition add partition(day='20211031') 
partition(day='20211101'); 
```

### 删除分区

```sql
alter table dept_partition drop partition(day='20211030');
alter table dept_partition drop partition(day='20211031') 
partition(day='20211101'); 
```

### 查看分区表有多少分区

```sql
show partitions dept_partition; 
```

### 查看分区表结构

```sql
desc formatted dept_partition; 
```

## 二级分区

思考: 如果一天的日志数据量也很大，如何再将数据拆分? 

### 创建二级分区表

```sql
create table dept_partition2( 
deptno int, dname string, loc string
) 
partitioned by (day string, hour string) 
row format delimited fields terminated by ' '; 
```

### 正常的加载数据

加载数据到二级分区表中 

```sh
load data local inpath 
'./dept_20211027.log' into table dept_partition2 
partition(day='20211027',hour='12');
```

查询二级分区数据

```sql
select * from dept_partition2 where day='20211027' and hour='12'; 
```

### 把数据直接上传到分区目录上，让分区表和数据产生关联的三种方式

#### 方式一：上传数据后修复

上传数据 

```sh
hadoop fs -mkdir /user/hive/warehouse/test.db/dept_partition2/day=20211028/hour=13;
hadoop fs -put ./dept_20211028.log /user/hive/warehouse/test.db/dept_partition2/day=20211028/hour=13; 
```

查询数据（查询不到刚上传的数据） 

```sql
select * from dept_partition2 where day='20211028' and hour='13'; 
```

执行修复命令 

```sh
msck repair table dept_partition2; 
```

再次查询数据，即可查到。

#### 方式二：上传数据后添加分区

上传数据 

```sh
hadoop fs -mkdir -p /user/hive/warehouse/test.db/dept_partition2/day=20211028/hour=14;
hadoop fs -put ./dept_20211028.log /user/hive/warehouse/test.db/dept_partition2/day=20211028/hour=14; 
```

执行添加分区 

```sql
alter table dept_partition2 add partition(day='20211028',hour='14'); 
```

查询数据

```sql
select * from dept_partition2 where day='20211028' and hour='14'; 
```

#### 方式三：创建文件夹后load 数据到分区

创建目录

```sh
hadoop fs -mkdir -p /user/hive/warehouse/test.db/dept_partition2/day=20211029/hour=15;
```

上传数据 

```sql
load data local inpath 
'./dept_20211028.log ' into table 
 dept_partition2 partition(day='20211028',hour='15'); 
```

查询数据

```sql
select * from dept_partition2 where day='20211028' and hour='15'; 
```

#### 经过实际测试发现如下问题

` hadoop fs -mkdir -p  `目录中含有 ` = ` 并不能如期创建文件夹。

但是在有分区表的基础上，直接执行添加分区命令会自行创建目录

```sql
alter table dept_partition2 add partition(day='20211029',hour='18'); 
```

然后上传数据

```sql
load data local inpath './dept_20211029.log'  into table dept_partition2 partition(day="20211029",hour="18"); 
```

查询数据

```sql
select * from dept_partition2 where day='20211029' and hour='18'; 
```

## 动态分区调整

关系型数据库中，对分区表Insert 数据时候，数据库自动会根据分区字段的值，将数据插入到相应的分区中，Hive 中也提供了类似的机制，即动态分区(Dynamic Partition)，只不过，使用Hive 的动态分区，需要进行相应的配置。 

### 参数配置

1. 开启动态分区功能（默认true，开启） 

   ```sh
   set hive.exec.dynamic.partition=true;
   ```

2. 设置为非严格模式（动态分区的模式，默认 strict，表示必须指定至少一个分区为
   静态分区，nonstrict 模式表示允许所有的分区字段都可以使用动态分区。）

   ```sh
   set hive.exec.dynamic.partition.mode=nonstrict;
   ```

3. 在所有执行 MR 的节点上，最大一共可以创建多少个动态分区。默认1000 

   ```sh
   set hive.exec.max.dynamic.partitions=1000;
   ```

4. **在每个执行 MR 的节点上，最大可以创建多少个动态分区**。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即 day 字段有365 个值，那么该参数就需要设置成大于 365，如果使用默认值100，则会报错。 

   ```sh
   set hive.exec.max.dynamic.partitions.pernode=100 
   ```

5. 整个MR Job 中，最大可以创建多少个HDFS 文件。默认100000 

   ```sh
   set hive.exec.max.created.files=100000;
   ```

6. 当有空分区生成时，是否抛出异常。一般不需要设置。默认 false 

   ```sh
   set hive.error.on.empty.partition=false;
   ```

### 实例操作

需求：将 dept 表中的数据按照地区（loc 字段），插入到目标表 dept_partition 的相应分区中。 

1. 创建目标分区表

   注意分区字段与表字段不能相同。

   ```sql
   create table dept_partition_dy(id int, name string) 
   partitioned by (loc int) row format delimited fields terminated by ' '; 
   ```

2. 设置动态分区

   ```sh
   set hive.exec.dynamic.partition.mode = nonstrict;
   ```

3. 插入数据

   hive3.0可以在插入时不指定分区字段。

   ```sql
   insert into table dept_partition_dy partition(loc) select 
   deptno, dname, loc from dept; 
   ```

4. 查看目标分区表的分区情况

   ```sh
   show partitions dept_partition_dy; 
   ```

dept内容：

```tex
10 ACCOUNTING 1700
20 RESEARCH 1800
30 SALES 1900
40 OPERATIONS 1700
```

结果：按照1700、1800、1900进行三个分区

![动态分区结果](https://cos.duktig.cn/typora/202110301102169.png)

思考：目标分区表是如何匹配到分区字段的？ 

# 分桶表 

分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区。对于一张表或者分区，Hive 可以进一步组织成桶，也就是更为细粒度的数据范围划分。 

分桶是将数据集分解成更容易管理的若干部分的另一个技术。

**分区针对的是数据的存储路径；分桶针对的是数据文件**。 

## 创建分桶表

数据准备：新建student.txt

```tex
1001 ss1 
1002 ss2 
1003 ss3 
1004 ss4 
1005 ss5 
1006 ss6 
1007 ss7 
1008 ss8 
1009 ss9 
1010 ss10 
1011 ss11 
1012 ss12 
1013 ss13 
1014 ss14 
1015 ss15 
1016 ss16 
```

创建分桶表 

```sql
create table stu_buck(id int, name string) 
clustered by(id)  
into 4 buckets 
row format delimited fields terminated by ' '; 
```

查看表结构

```sql
desc formatted stu_buck;
```

将本地文件上传到hdfs

```sh
hadoop fs -put ./student.txt /user/hive/warehouse
```

导入数据到分桶表中，load 的方式 

```sql
load data inpath '/user/hive/warehouse/student.txt' overwrite into table stu_buck; 
```

导入数据时会报错如下：

```sh
Diagnostic Messages for this Task:
Error: java.lang.RuntimeException: org.apache.hadoop.hive.ql.metadata.HiveException: Hive Runtime Error while processing row (tag=0) {"key":{},"value":{"
_col0":1011,"_col1":"ss11"}}
```

解决：

stackoverflow 给的解决方案：[Hive Runtime Error while processing row in Hive](https://stackoverflow.com/questions/28674753/hive-runtime-error-while-processing-row-in-hive)，但是经过实际测试没有成功。

查看创建的分桶表中是否分成4 个桶

![分桶表结果](https://cos.duktig.cn/typora/202110301448179.png)

查询分桶的数据 

```sql
select * from stu_buck; 
```

分桶规则： 

根据结果可知：Hive 的分桶采用对分桶字段的值进行哈希，然后除以桶的个数求余的方 
式决定该条记录存放在哪个桶当中 

## 分桶表操作注意事项

1. reduce 的个数设置为-1,让 Job 自行决定需要用多少个 reduce 或者将 reduce 的个数设置为大于等于分桶表的桶数 

2. 从hdfs 中load 数据到分桶表中，避免本地文件找不到问题 

3. insert 方式将数据导入分桶表 

   ```sql
   insert into table stu_buck select * from student_insert;
   ```

## 抽样查询

对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive 可以通过对表进行抽样来满足这个需求。 

语法: 

```sql
TABLESAMPLE(BUCKET x OUT OF y)  
```

查询表stu_buck 中的数据。 

```sql
select * from stu_buck tablesample(bucket 1 out of 4 on id); 
```

注意：x 的值必须小于等于 y 的值，否则报错：

```sh
FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck 
```

