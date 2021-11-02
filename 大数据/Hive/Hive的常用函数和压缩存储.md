> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 更多文章参看github知识库：[https://github.com/duktig666/knowledge](https://github.com/duktig666/knowledge)

# 背景

学习完Hadoop，有没有感到编写一个MapReduce程序非常复杂，想要进行一次分析和统计需要很大的开发成本。那么不如就来了解了解Hadoop生态圈的另一名成员——Hive。让我们一起来了解，如何使用类SQL语言进行快速查询和分析数据吧。

前边文章我们了解了Hive的概述、DDL语句和**DML语句（重点）**、分桶表和分区表，这篇文章主要了解Hive的常用函数和压缩存储。

Hive系列文章如下：

- [大数据基础之Hive（一）—— Hive概述](https://blog.csdn.net/qq_42937522/article/details/121096763?spm=1001.2014.3001.5501)
- [大数据基础之Hive（二）—— DDL语句和DML语句](https://blog.csdn.net/qq_42937522/article/details/121096833?spm=1001.2014.3001.5501)
- [大数据基础之Hive（三）—— 分区表和分桶表](https://blog.csdn.net/qq_42937522/article/details/121096891?spm=1001.2014.3001.5501)
- [大数据基础之Hive（四）—— 常用函数和压缩存储](https://blog.csdn.net/qq_42937522/article/details/121096983?spm=1001.2014.3001.5501)
- [大数据基础之Hive（五）——Hive实战（统计电影排名的各种问题）](https://blog.csdn.net/qq_42937522/article/details/121097029)

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

### 窗口函数（开窗函数）*

#### 函数说明：

- `OVER()`：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化。 
  - `CURRENT ROW`：当前行 
  - `n PRECEDING`：往前n 行数据 
  - `n FOLLOWING`：往后n 行数据 
  - `UNBOUNDED`：起点，
    -  `UNBOUNDED PRECEDING` 表示从前面的起点， 
    -  `UNBOUNDED FOLLOWING` 表示到后面的终点 
- `LAG(col,n,default_val)`：往前第n 行数据 
- `LEAD(col,n, default_val)`：往后第n 行数据 
- `NTILE(n)`：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从 1 开始，对于每一行，NTILE 返回此行所属的组的编号。注意：n 必须为int 类型。 

### Rank 

#### 函数说明

- RANK() 排序相同时会重复，总数不会变 
- DENSE_RANK() 排序相同时会重复，总数会减少 
- ROW_NUMBER() 会根据顺序计算 

### 其他常用函数

```sh
常用日期函数
unix_timestamp:返回当前或指定时间的时间戳	
select unix_timestamp();
select unix_timestamp("2020-10-28",'yyyy-MM-dd');

from_unixtime：将时间戳转为日期格式
select from_unixtime(1603843200);

current_date：当前日期
select current_date;

current_timestamp：当前的日期加时间
select current_timestamp;

to_date：抽取日期部分
select to_date('2020-10-28 12:12:12');

year：获取年
select year('2020-10-28 12:12:12');

month：获取月
select month('2020-10-28 12:12:12');

day：获取日
select day('2020-10-28 12:12:12');

hour：获取时
select hour('2020-10-28 12:12:12');

minute：获取分
select minute('2020-10-28 12:12:12');

second：获取秒
select second('2020-10-28 12:12:12');

weekofyear：当前时间是一年中的第几周
select weekofyear('2020-10-28 12:12:12');

dayofmonth：当前时间是一个月中的第几天
select dayofmonth('2020-10-28 12:12:12');

months_between： 两个日期间的月份
select months_between('2020-04-01','2020-10-28');

add_months：日期加减月
select add_months('2020-10-28',-3);

datediff：两个日期相差的天数
select datediff('2020-11-04','2020-10-28');

date_add：日期加天数
select date_add('2020-10-28',4);

date_sub：日期减天数
select date_sub('2020-10-28',-4);

last_day：日期的当月的最后一天
select last_day('2020-02-30');

date_format(): 格式化日期
select date_format('2020-10-28 12:12:12','yyyy/MM/dd HH:mm:ss');

常用取整函数
round： 四舍五入
select round(3.14);
select round(3.54);

ceil：  向上取整
select ceil(3.14);
select ceil(3.54);

floor： 向下取整
select floor(3.14);
select floor(3.54);

常用字符串操作函数
upper： 转大写
select upper('low');

lower： 转小写
select lower('low');

length： 长度
select length("atguigu");

trim：  前后去空格
select trim(" atguigu ");

lpad： 向左补齐，到指定长度
select lpad('atguigu',9,'g');

rpad：  向右补齐，到指定长度
select rpad('atguigu',9,'g');

regexp_replace：使用正则表达式匹配目标字符串，匹配成功后替换！
SELECT regexp_replace('2020/10/25', '/', '-');

集合操作
size： 集合中元素的个数
select size(friends) from test3;

map_keys： 返回map中的key
select map_keys(children) from test3;

map_values: 返回map中的value
select map_values(children) from test3;

array_contains: 判断array中是否包含某个元素
select array_contains(friends,'bingbing') from test3;

sort_array： 将array中的元素排序
select sort_array(friends) from test3;

grouping_set:多维分析

```

### 自定义函数

Hive 自带了一些函数，比如：max/min 等，但是数量有限，自己可以通过自定义 UDF 来方便的扩展。 

当Hive 提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。

根据用户自定义函数类别分为以下三种： 

- UDF（User-Defined-Function） 一进一出 
- UDAF（User-Defined Aggregation Function）  聚集函数，多进一出 类似于：count/max/min 
- UDTF（User-Defined Table-Generating Functions） 一进多出   如lateral view explode() 

编程步骤： 

1. 继承Hive 提供的类 

   ```java
    org.apache.hadoop.hive.ql.udf.generic.GenericUDF 
    org.apache.hadoop.hive.ql.udf.generic.GenericUDTF
   ```

2. 实现类中的抽象方法 

3. 添加jar，或者将jar包放在hive的lib文件夹下重启

   ```sh
   add jar  xxx
   ```

4. 在hive 的命令行窗口创建函数 

   ```sh
   create [temporary] function [dbname.]function_name AS class_name; 
   ```

5. 在hive 的命令行窗口删除函数 

   ```sh
   drop [temporary] function [if exists] [dbname.]function_name; 
   ```



# 压缩和存储

## Hadoop压缩配置

### MapReduce支持的压缩编码

| 压缩格式 | Hadoop自带？     | 算法    | 文件扩展名 | 是否可切片 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ---------------- | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用     | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用     | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用     | bzip2   | .bz2       | **是**     | 和文本处理一样，不需要修改             |
| LZO      | **否，需要安装** | LZO     | .lzo       | **是**     | **需要建索引，还需要指定输入格式**     |
| Snappy   | 是，直接使用     | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

要在Hadoop中启用压缩，可以配置如下参数

| 参数                                                         | 默认值                                         | 阶段        | 建议                                                  |
| ------------------------------------------------------------ | ---------------------------------------------- | ----------- | ----------------------------------------------------- |
| io.compression.codecs    （在core-site.xml中配置）           | 无，这个需要在命令行输入hadoop checknative查看 | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器          |
| mapreduce.map.output.compress（在mapred-site.xml中配置）     | false                                          | mapper输出  | 这个参数设为true启用压缩                              |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec     | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据         |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） | false                                          | reducer输出 | 这个参数设为true启用压缩                              |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec     | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2### 压缩的使用 |

### 开启 Map 输出阶段压缩（MR 引擎）

开启 map 输出阶段压缩可以减少 job 中 map 和 Reduce task 间数据传输量。具体配置如下： 

1. 开启hive 中间传输数据压缩功能 

   ```sh
   set hive.exec.compress.intermediate=true; 
   ```

2. 开启mapreduce 中map 输出压缩功能

   ```sh
   set mapreduce.map.output.compress=true; 
   ```

3. 设置mapreduce 中map 输出数据的压缩方式 

   ```sh
   set mapreduce.map.output.compress.codec= org.apache.hadoop.io.compress.SnappyCodec; 
   ```

### 开启 Reduce 输出阶段压缩 

当 Hive 将 输 出 写 入 到 表 中 时 ， 输 出 内 容 同 样 可 以 进 行 压 缩 。 属 性hive.exec.compress.output 控制着这个功能。用户可能需要保持默认设置文件中的默认值false，这样默认的输出就是非压缩的纯文本文件了。用户可以通过在查询语句或执行脚本中设置这个值为true，来开启输出结果压缩功能。 

1. 开启hive 最终输出数据压缩功能 

   ```sh
   set hive.exec.compress.output=true; 
   ```

2. 开启mapreduce 最终输出数据压缩 

   ```sh
   set mapreduce.output.fileoutputformat.compress=true; 
   ```

3. 设置mapreduce 最终数据输出压缩方式 

   ```sh
   set mapreduce.output.fileoutputformat.compress.codec = org.apache.hadoop.io.compress.SnappyCodec; 
   ```

4. 设置mapreduce 最终数据输出压缩为块压缩 

   ```sh
   set mapreduce.output.fileoutputformat.compress.type=BLOCK; 
   ```

5. 测试一下输出结果是否是压缩文件 

   ```sh
   insert overwrite local directory './distribute-result' select * from emp distribute by deptno sort by empno desc; 
   ```

## Hive的文件存储格式

Hive 支持的存储数据的格式主要有：TEXTFILE 、SEQUENCEFILE、ORC、PARQUET。 

###  列式存储和行式存储

![列式存储和行式存储](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211101175247546.png)

如图所示左边为逻辑表，右边第一个为行式存储，第二个为列式存储。 

**行存储的特点** 

查询满足条件的一整行数据的时候，列存储则需要去每个聚集的字段找到对应的每个列的值，行存储只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。 

列存储的特点 

因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。

TEXTFILE 和 SEQUENCEFILE 的存储格式都是基于行存储的； 

ORC 和PARQUET 是基于列式存储的。 

### TextFile 格式 

默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合 Gzip、Bzip2 使用，但使用Gzip 这种方式，hive 不会对数据进行切分，从而无法对数据进行并行操作。 

### Orc 格式 

Orc (Optimized Row Columnar)是Hive 0.11 版里引入的新的存储格式。 

### Parquet 格式 

Parquet 文件是以二进制方式存储的，所以是不可以直接读取的，文件中包括该文件的数据和元数据，因此**Parquet 格式文件是自解析的**。 

在实际的项目开发当中，hive 表的数据存储格式一般选择：orc 或 parquet。压缩方式一般选择snappy，lzo。 

