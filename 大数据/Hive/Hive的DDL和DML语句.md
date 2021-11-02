> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 更多文章参看github知识库：[https://github.com/duktig666/knowledge](https://github.com/duktig666/knowledge)

# 背景

学习完Hadoop，有没有感到编写一个MapReduce程序非常复杂，想要进行一次分析和统计需要很大的开发成本。那么不如就来了解了解Hadoop生态圈的另一名成员——Hive。让我们一起来了解，如何使用类SQL语言进行快速查询和分析数据吧。

上一篇文章我们了解了Hive的概述，这篇文章我们来了解Hive的DDL语句和**DML语句（重点）**。

Hive系列文章如下：

- [大数据基础之Hive（一）—— Hive概述](https://blog.csdn.net/qq_42937522/article/details/121096763?spm=1001.2014.3001.5501)
- [大数据基础之Hive（二）—— DDL语句和DML语句](https://blog.csdn.net/qq_42937522/article/details/121096833?spm=1001.2014.3001.5501)
- [大数据基础之Hive（三）—— 分区表和分桶表](https://blog.csdn.net/qq_42937522/article/details/121096891?spm=1001.2014.3001.5501)
- [大数据基础之Hive（四）—— 常用函数和压缩存储](https://blog.csdn.net/qq_42937522/article/details/121096983?spm=1001.2014.3001.5501)
- [大数据基础之Hive（五）——Hive实战（统计电影排名的各种问题）](https://blog.csdn.net/qq_42937522/article/details/121097029)

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
- Hive 要求DISTRIBUTE BY 语句要写在 SORT BY 语句之前。 

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

