# Hive概述



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



```sql
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long 
guan_beijing 
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing 
```



```sql
select friends[1],children['xiao song'],address.city from 
test 
where name="songsong";
```



# DDL

## 创建数据库

```sql
CREATE DATABASE [IF NOT EXISTS] database_name 
[COMMENT database_comment] 
[LOCATION hdfs_path] 
[WITH DBPROPERTIES (property_name=property_value, ...)];
```

注意事项：

- 避免要创建的数据库已经存在错误，增加 `if not exists` 判断
- `COMMENT` ：数据库注释
- `LOCATION` ：数据库在 HDFS 上的存储路径（默认存储路径为`/user/hive/warehouse/*.db`）。
- `WITH`：指定数据库参数（不常用）

例子：

```sql
create database if not exists test; 
```

结果：

![Hive创建数据库](https://cos.duktig.cn/typora/202110282206130.png)

## 查询数据库

```sql
# 显示数据库 
show databases; 
# 过滤显示查询的数据库 
show databases like 'db_hive*'; 

# 显示数据库信息 
desc database test; 
# 显示数据库详细信息，extended 
desc database extended test; 
```

## 切换数据库

```sql
use test; 
```

## 修改数据库

用户可以使用`ALTER DATABASE` 命令为某个数据库的`DBPROPERTIES` 设置键-值对属性值，来描述这个数据库的属性信息。 

```sql
alter database test set dbproperties('createtime'='20170830'); 
```

## 删除数据库

```sql
# 删除空数据库 
drop database test;
# 如果删除的数据库不存在，最好采用 if exists 判断数据库是否存在 
drop database if exists test; 
# 如果数据库不为空，可以采用 cascade 命令，强制删除 
drop database test cascade; 
```

## 创建表

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path] 
[TBLPROPERTIES (property_name=property_value, ...)] 
[AS select_statement] 
```

字段解释：

- `EXTERNAL`：可以让用户创建一个外部表。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。
- `LOCATION`：指定表的位置信息
- `COMMENT`：为表和列添加注释
- `PARTITIONED BY` ：创建分区表
- `CLUSTERED BY`： 创建分桶表 
- `ROW FORMAT row_format`：定义行的格式
- `[TBLPROPERTIES (property_name=property_value, ...)]`：额外属性
- `AS`：后跟查询语句，根据查询结果创建表
- `SORTED BY `：不常用，对桶中的一个或多个列另外排序 
- `STORED AS` ：指定存储文件类型 
  - 常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件） 
  - 如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。 

### 管理表

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive 会（或多或少地）控制着数据的生命周期。Hive 默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。 

**当我们删除一个管理表时，Hive 也会删除这个表中数据。管理表不适合和其他工具共享数据。** 

### 外部表

因为表是外部表，所以 Hive 并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。 

### 管理表和外部表的使用场景

每天将收集到的网站日志定期流入HDFS 文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT 进入内部表。

### 管理表与外部表的互相转换 

```sql
# 查询表的类型 MANAGED_TABLE 
desc formatted test; 
# 修改内部表 test 为外部表 
alter table test set tblproperties('EXTERNAL'='TRUE'); 
```

注意：`('EXTERNAL'='TRUE')`和`('EXTERNAL'='FALSE')`为固定写法，区分大小写！ 

## 修改表

```sql
# 重命名表
ALTER TABLE table_name RENAME TO new_table_name;
# 更新列
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name] 
# 增加和替换列 
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)  
```

注：

- `ADD` 是代表新增一字段，字段位置在所有列后面(partition 列前)
- `REPLACE` 则是表示替换表中所有字段。 

## 删除表

```sql
drop table test; 
```



# DML

## 数据导入

### 向表中装载数据（Load） 

```sql
load data [local] inpath '数据的 path' [overwrite] into table test [partition (partcol1=val1,…)]; 
```

字段解释：

- `load data`:表示加载数据 
- `local`:表示从本地加载数据到 hive 表；否则从HDFS 加载数据到 hive 表 
- `inpath`:表示加载数据的路径 
- `overwrite`:表示覆盖表中已有数据，否则表示追加 
- `into table`:表示加载到哪张表 
- `test`:表示具体的表 
- `partition`:表示上传到指定分区 

### 通过查询语句向表中插入数据（Insert） 

```sql
# 创建一张表 
create table student_par(id int, name string) row format delimited fields terminated by '\t'; 
# 基本插入数据 
insert into table  student_par values(1,'wangwu'),(2,'zhaoliu');
# 基本模式插入（根据单张表查询结果） 
insert overwrite table student_par select id, name from student where month='201709'; 
# 多表（多分区）插入模式（根据多张表查询结果） 
from student 
insert overwrite table student partition(month='201707') 
select id, name where month='201709' 
insert overwrite table student partition(month='201706') 
select id, name where month='201709'; 
```

insert into：以追加数据的方式插入到表或分区，原有数据不会删除 

insert overwrite：会覆盖表中已存在的数据 

注意：insert 不支持插入部分字段 

### 查询语句中创建表并加载数据（As Select） 

```sql
create table if not exists student3 as select id, name from student; 
```

根据查询结果创建表（查询的结果会添加到新创建的表中） 

### Import 数据到指定Hive 表中 

注意：先用 export 导出后，再将数据导入。 

```sql
import table student2 from '/user/hive/warehouse/export/student'; 
```

## 数据导出

### Insert导出

```sql
# 将查询的结果导出到本地 
insert overwrite local directory '/opt/module/hive/data/export/student' 
select * from student; 
# 将查询的结果格式化导出到本地 
insert overwrite local directory 
'/opt/module/hive/data/export/student1' 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
select * from student; 
# 将查询的结果导出到 HDFS 上(没有 local) 
insert overwrite directory '/user/atguigu/student2' 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'  
select * from student; 
```

### Hadoop 命令导出到本地 

```sql
dfs -get /user/hive/warehouse/student/student.txt 
/opt/module/data/export/student3.txt; 
```

### Hive Shell 命令导出

```sh
bin/hive -e 'select * from default.student;' > 
 /opt/module/hive/data/export/student4.txt; 
```

### Export 导出到HDFS 上 

```sql
export table default.student to '/user/hive/warehouse/export/student'; 
```

### 清除表中数据（Truncate）

注意：Truncate 只能删除管理表，不能删除外部表中数据 

```sql
truncate table student; 
```

# 基本查询

## 数据准备（辅助查询）

**原始数据准备**

dept.txt:

```tex
10 ACCOUNTING 1700 
20 RESEARCH 1800 
30 SALES 1900 
40 OPERATIONS 1700
```

emp.txt:

```tex
7369 SMITH CLERK 7902 1980-12-17 800.00 20 
7499 ALLEN SALESMAN 7698 1981-2-20 1600.00 300.00 30 
7521 WARD SALESMAN 7698 1981-2-22 1250.00 500.00 30 
7566 JONES MANAGER 7839 1981-4-2 2975.00 20 
7654 MARTIN SALESMAN 7698 1981-9-28 1250.00 1400.00 30 
7698 BLAKE MANAGER 7839 1981-5-1 2850.00 30 
7782 CLARK MANAGER 7839 1981-6-9 2450.00 10 
7788 SCOTT ANALYST 7566 1987-4-19 3000.00 20 
7839 KING PRESIDENT 1981-11-17 5000.00 10 
7844 TURNER SALESMAN 7698 1981-9-8 1500.00 0.00 30 
7876 ADAMS CLERK 7788 1987-5-23 1100.00 20 
7900 JAMES CLERK 7698 1981-12-3 950.00 30 
7902 FORD ANALYST 7566 1981-12-3 3000.00 20 
7934 MILLER CLERK 7782 1982-1-23 1300.00 10 
```

**使用test数据库**

```sql
use test;
```

**创建部门表**

```sql
create table if not exists dept( 
deptno int, 
dname string, 
loc int 
) 
row format delimited fields terminated by ' '; 
```

**创建员工表**

```sql
create table if not exists emp( 
empno int, 
ename string, 
job string, 
mgr int, 
hiredate string,  
sal double,  
comm double, 
deptno int
) 
row format delimited fields terminated by ' '; 
```

**导入数据**

```sql
load data local inpath './dept.txt' overwrite into table dept; 
load data local inpath './emp.txt' overwrite into table emp; 
```

![数据导入成功](https://cos.duktig.cn/typora/202110291750028.png)

## 全表和特定列查询

```sql
# 全表查询 
select * from emp; 
select empno,ename,job,mgr,hiredate,sal,comm,deptno from emp ; 
# 选择特定列查询 
select empno, ename from emp; 
```

注意事项：

- SQL 语言大小写不敏感。 
- SQL 可以写在一行或者多行 
- 关键字不能被缩写也不能分行 
- 各子句一般要分行写。 
- 使用缩进提高语句的可读性。 

## 别名

重命名一个列/表，便于计算 ，紧跟列名，也可以在列名和别名之间加入关键字‘AS’。

```sql
select ename AS name, deptno dn from emp; 
```

## 一些特别需要注意的查询关键字

Hive中的关键字大多数与mysql相同，就不一一罗列了，这里举出一些特殊的关键字

#### RLIKE

RLIKE 子句是 Hive 中这个功能的一个扩展，其可以通过 Java 的正则表达式这个更强大的语言来指定匹配条件。 

```sql
# 查找名字中带有A 的员工信息 
select * from emp where ename  RLIKE '[A]'; 
```

#### Group By 分组语句 

GROUP BY 语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然后对每个组执行聚合操作。 

```sql
# 计算emp 表每个部门的平均工资 
select t.deptno, avg(t.sal) avg_sal from emp t group by t.deptno; 
# 计算emp 每个部门中每个岗位的最高薪水 
select t.deptno, t.job, max(t.sal) max_sal from emp t group by t.deptno, t.job; 
```

#### Having 语句 

having 与where 不同点 

1. where 后面不能写分组函数，而 having 后面可以使用分组函数。 
2. having 只用于group by 分组统计语句。

```sql
# 求每个部门的平均工资 
select deptno, avg(sal) from emp group by deptno; 
# 求每个部门的平均薪水大于 2000 的部门 
select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000; 
```



## 连接查询

与mysql相同，特别介绍 **满外连接** 
**满外连接**：将会返回所有表中符合 WHERE 语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL 值替代。 

```sql
select e.empno, e.ename, d.deptno from emp e full join dept d on e.deptno = d.deptno; 
```

## 排序

### 全局排序

Order By（默认ASC升序）：全局排序，只有一个Reducer 

```sql
select * from emp order by sal; 
select * from emp order by sal desc; 
```

### 按照别名排序 

```sql
# 按照员工薪水的 2 倍排序 
select ename, sal*2 twosal from emp order by twosal; 
```

### 多个列排序

```sql
# 按照部门和工资升序排序 
select ename, deptno, sal from emp order by deptno, sal; 
```

### 每个 Reduce 内部排序（Sort By） 

Sort By：对于大规模的数据集 order by 的效率非常低。在很多情况下，并不需要全局排序，此时可以使用 sort by。 
Sort by 为每个 reducer 产生一个排序文件。每个 Reducer 内部进行排序，对全局结果集来说不是排序。 

实例操作如下：

1. 设置 reduce 个数 

   ```sh
   set mapreduce.job.reduces=3; 
   ```

2. 查看设置 reduce 个数 

   ```sh
   set mapreduce.job.reduces; 
   ```

3. 根据部门编号降序查看员工信息 

   ```sql
   select * from emp sort by deptno desc; 
   ```

4. 将查询结果导入到文件中（按照部门编号降序排序） 

   ```sql
   insert overwrite local directory './sortby-result' select * from emp sort by deptno desc; 
   ```

## 分区（Distribute By） 

Distribute By： 在有些情况下，我们需要控制某个特定行应该到哪个 reducer，通常是为了进行后续的聚集操作。distribute by 子句可以做这件事。distribute by 类似 MR 中partition（自定义分区），进行分区，结合sort by 使用。 

对于 distribute by 进行测试，一定要分配多 reduce 进行处理，否则无法看到 distributeby 的效果。 

**实例操作如下**：

先按照部门编号分区，再按照员工编号降序排序。 

```sql
set mapreduce.job.reduces=3; 
insert overwrite local directory './distribute-result' 
select * from emp distribute by deptno sort by empno desc; 
```

注意事项：

- distribute by 的分区规则是根据分区字段的 hash 码与 reduce 的个数进行模除后，余数相同的分到一个区。 
-  Hive 要求DISTRIBUTE BY 语句要写在 SORT BY 语句之前。 

## Cluster By 

当 distribute by 和 sorts by 字段相同时，可以使用 cluster by 方式。 

cluster by 除了具有 distribute by 的功能外还兼具 sort by 的功能。但是排序只能是升序排序，不能指定排序规则为 ASC 或者DESC。 

```sql
# 以下两种写法等价 
select * from emp cluster by deptno; 
select * from emp distribute by deptno sort by deptno;
```

注意：按照部门编号分区，不一定就是固定死的数值，可以是20 号和30 号部门分到一
个分区里面去。



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



# 函数

## 系统内置函数 

```sh
# 查看系统自带的函数 
show functions; 
# 显示自带的函数的用法 
desc function upper; 
# 详细显示自带的函数的用法 
desc function extended upper; 
```

## 常用内置函数

### 空字段赋值 NVL 

NVL：给值为 NULL 的数据赋值，它的格式是 NVL( value，default_value)。

它的功能是**如果value 为 NULL，则NVL 函数返回default_value 的值，否则返回 value 的值**，如果两个参数都为NULL ，则返回NULL。 

### CASE WHEN THEN ELSE END 

```sql
create table emp_sex( 
name string,  
dept_id string,  
sex string)  
row format delimited fields terminated by " "; 
```

```sql
select 
  dept_id, 
  sum(case sex when '男' then 1 else 0 end) male_count, 
  sum(case sex when '女' then 1 else 0 end) female_count 
from emp_sex 
group by dept_id; 
```

### 行转列 （多个列拼接成一个列）

#### 函数说明

`CONCAT(string A/col, string B/col…)`：返回输入字符串连接后的结果，支持任意个输入字符串; 
`CONCAT_WS(separator, str1, str2,...)`：它是一个特殊形式的 CONCAT()。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间; 

`COLLECT_SET(col)`：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生Array 类型字段。 

#### 案例实操

数据准备，COLLECT.txt:

字段：name constellation blood_type

```tex
孙悟空 白羊座 A
大海 射手座 A
宋宋 白羊座 B
猪八戒 白羊座 A
凤姐 射手座 A
苍老师 白羊座 B
```

需求，把星座和血型一样的人归类到一起。结果如下： 

```tex
射手座,A 大海|凤姐 
白羊座,A 孙悟空|猪八戒 
白羊座,B 宋宋|苍老师 
```

创建 hive 表

```sql
create table person_info( 
name string comment '名字',  
constellation string comment '星座',  
blood_type string comment '血型')  
row format delimited fields terminated by " "; 
```

导入数据 

```sh
load data local inpath "./person_info.txt" into table person_info;
```

按需求查询数据 

```sql
SELECT 
t1.c_b, 
CONCAT_WS("|",collect_set(t1.name)) 
FROM ( 
SELECT 
NAME, 
CONCAT_WS(',',constellation,blood_type) c_b 
FROM person_info 
)t1 
GROUP BY t1.c_b;
```



### 列转行 （一个行转成多个行）

#### 函数说明

`EXPLODE(col)`：将hive 一列中复杂的 Array 或者 Map 结构拆分成多行

`LATERAL VIEW `：用于和 split, explode 等UDTF 一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。 

用法：

```sql
LATERAL VIEW udtf(expression) tableAlias AS columnAlias 
```

#### 实例操作

创建本地 movie.txt

```tex
《疑犯追踪》	悬疑,动作,科幻,剧情 
《Lie to me》	悬疑,警匪,动作,心理,剧情 
《战狼 2》		战争,动作,灾难 
```

需求：

将电影分类中的数组数据展开。结果如下： 

```tex
《疑犯追踪》      悬疑 
《疑犯追踪》      动作 
《疑犯追踪》      科幻 
《疑犯追踪》      剧情 
《Lie to me》   悬疑 
《Lie to me》   警匪 
《Lie to me》   动作 
《Lie to me》   心理 
《Lie to me》   剧情 
《战狼 2》 战争 
《战狼 2》 动作 
《战狼 2》 灾难 
```

创建 hive 表并导入数据 

```sql
create table movie_info( 
    movie string, 
    category string) 
row format delimited fields terminated by "\t"; 

load data local inpath "./movie.txt" into table movie_info; 
```

按需求查询数据 

```sql
SELECT 
movie, 
category_name 
FROM 
movie_info 
lateral VIEW 
explode(split(category,",")) movie_info_tmp  AS category_name; 
```

### 窗口函数（开窗函数） 

#### 函数说明：

- `OVER()`：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化。 
  - `CURRENT ROW`：当前行 
  - `n PRECEDING`：往前n 行数据 
  - `n FOLLOWING`：往后n 行数据 
  - `UNBOUNDED`：起点，
    -  `UNBOUNDED PRECEDING` 表示从前面的起点， 
    - `UNBOUNDED FOLLOWING` 表示到后面的终点 
- `LAG(col,n,default_val)`：往前第n 行数据 
- `LEAD(col,n, default_val)`：往后第n 行数据 
- `NTILE(n)`：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从 1 开始，对于每一行，NTILE 返回此行所属的组的编号。注意：n 必须为int 类型。 







