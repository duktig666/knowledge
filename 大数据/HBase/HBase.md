# HBase 概述

## 什么是 Hbase？

HBase是一种分布式、可扩展、支持海量数据存储的 **NoSQL数据库**。 

HBase是依赖Hadoop的。为什么HBase能存储海量的数据？**因为HBase是在HDFS的基础之上构建的，HDFS是分布式文件系统**。

HBase在HDFS之上提供了**高并发的随机写和支持实时查询**，这是HDFS不具备的。

基于「列式存储」，**存储数据的“结构”可以地非常灵活**。

## HBase的存储结构

### HBase的逻辑结构

![HBase的逻辑结构](https://cos.duktig.cn/typora/202111061556536.png)

逻辑结构分析：

- Region：相当于表，数据量大的时候会进行切片，相当于数据库的水平分表分库。
- store：每个Store其实就是一个列族的数据（所以我们可以说HBase是基于列族存储的）
- 列族（**Column Family**）：在HBase里边，先有列族，后有列；可以简单理解为：列的属性**类别**。
- 列（**Column Qualifier**，列修饰符）：在HBase中用列修饰符（**Column Qualifier**）来标识每个列。
- 行键（**RowKey**）：定位一行数据的唯一值。

### HBase 物理存储结构

![HBase 物理存储结构](https://cos.duktig.cn/typora/202111061614634.png)

每一行数据会有不同的版本，通过时间戳（TimeStamp）来进行区分。

## HBase的数据模型

### Name Space 

命名空间，类似于关系型数据库的 DatabBase 概念，每个命名空间下有多个表。

HBase有两个自带的命名空间，分别是 hbase 和 default，hbase 中存放的是 HBase 内置的表，default表是用户默认使用的命名空间。 

### Region

类似于关系型数据库的表概念。不同的是，HBase定义表时**只需要声明列族即可**，不需要声明具体的列。这意味着，往HBase写入数据时，**字段可以动态、按需指定**。因此，和关系型数据库相比，HBase能够轻松应对字段变更的场景。 

### Row 

HBase表中的每行数据都由 **一个RowKey**  和 **多个Column（列）** 组成，数据是按照 RowKey的**字典顺序存储**的，并且查询数据时只能根据RowKey 进行检索，所以RowKey的设计十分重要。 

### Column 

HBase中的每个列都由 **Column Family(列族)** 和 **Column Qualifier（列限定符）**进行限定，例如info：name，info：age。建表时，只需指明列族，而列限定符无需预先定义。

### TimeStamp 

用于标识数据的不同版本（version），每条数据写入时，如果不指定时间戳，系统会自动为其加上该字段，其值为写入 HBase的时间。 

### Cell 

由 `{rowkey, column Family：column Qualifier, time Stamp} ` 唯一确定的单元。**cell 中的数据是没有类型的，全部是字节码形式存贮**。 

# HBase的安装

## win10安装HBase

参看：[WIN10下安装HBASE教程](https://www.cnblogs.com/nopassword/p/5882727.html)

HBase建议不要选择太高的版本，当时选择了2.4.8的HBase，但是运行 `hbase shell` 运行失败，信息如下：

```sh
This file has been superceded by packaging our ruby files into a jar
and using jruby's bootstrapping to invoke them. If you need to
source this file fo some reason it is now named 'jar-bootstrap.rb' and is
located in the root of the file hbase-shell.jar and in the source tree at
'hbase-shell/src/main/ruby'.
```

换成了1.4.9的版本就好了。

# HBase的shell操作

## 启动和帮助命令

```sh
# 进入HBase 客户端命令行 
bin/hbase shell 
# 查看帮助命令 
help 
```

## 表操作命令

```sh
# 创建表（至少有一个列族名）
create '表名','列族名'
create '表名','NAME=>列族名,VERSION=>3'
create '命名空间:表名','列族名'
create 'stu','name','age','info'

# 查看表
## 查看当前数据库中有哪些表 
list
## 查看表结构 
describe '表名'

# 删除表
## 1.首先需要先让该表为 disable 状态
disable '表名'
## 2.然后才能删除表
drop '表名'

# 修改表
alter 'student',{NAME=>'info',VERSIONS=>3} 
```

## 命名空间操作命令

```sh
# 列出所有命名空间
list_namespace

# 创建命名空间
create_namespace '命名空间名'

# 在命名空间下创建表
create '命名空间:表名','列族名'

# 删除命名空间（需要先删除命名空间下的表）
drop_namespace '命名空间名'
drop_all
```

## 数据操作命令

```sh
# 新增/更新数据
put '命名空间:表名','行键','列族名:列修饰符（列）','值'
put 'student','1001','info:age','18'

# 扫描查看表数据 
scan '表名' 
scan '表名',{条件1 => '条件值1', 条件2 => 条件值2} 
scan 'student',{STARTROW => '1001'} 

# 查看“指定行”或“指定列族:列”的数据 
get 'student','1001' 
get 'student','1001','info:name' 
get 'student','1001',{COLUMN=>'info:name',VERSIONS=>3} 

# 统计表数据行数 
count 'student' 

# 删除数据 
## 删除某rowkey 的全部数据
deleteall 'student','1001' 
## 删除某rowkey 的某一列数据
delete 'student','1002','info:sex' 

# 清空表数据 （清空表的操作顺序为先 disable，然后再 truncate）
truncate 'student' 
```



## HBase的架构

![HBase的架构](https://cos.duktig.cn/typora/202111061702755.png)



### **Client**

客户端，它提供了访问HBase的接口，并且维护了对应的cache来加速HBase的访问。

### **Zookeeper**

存储HBase的元数据（meta表），无论是读还是写数据，都是去Zookeeper里边拿到meta元数据**告诉给客户端去哪台机器读写数据**。

HBase 通过Zookeeper 来做 Master 的高可用、RegionServer 的监控、元数据的入口以及集群配置的维护等工作。

### **HRegionServer**

它是处理客户端的读写请求，负责与HDFS底层交互，是真正干活的节点。

Region Server为 Region的管理者，其实现类为 HRegionServer，主要作用如下: 

- 对于数据的操作：get, put, delete； 
- 对于 Region的操作：splitRegion、compactRegion。

#### StoreFile

保存实际数据的物理文件，StoreFile 以 HFile 的形式存储在 HDFS 上。每个 Store 会有一个或多个StoreFile（HFile），数据在每个StoreFile 中都是有序的。 

#### MemStore 

写缓存，由于HFile 中的数据要求是有序的，所以数据是先存储在MemStore 中，排好序后，等到达刷写时机才会刷写到HFile，每次刷写都会形成一个新的HFile。 

#### HLog

操作预先写入日志文件。

由于数据要经 MemStore 排序后才能刷写到 HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做 Write-Ahead logfile 的文件中，然后再写入MemStore 中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。 

### HMaster

它在HBase的架构中承担一种什么样的角色呢？**读写请求都没经过Hmaster呀**。

Master是所有Region Server的管理者，其实现类为 HMaster，主要作用如下： 

- 对于表的操作：create, delete, alter 
- 对于RegionServer的操作：分配Regions到每个RegionServer（数据量太大，拆分重新合并）；处理元数据的变更和监控RegionServer的状态；负载均衡和故障转移。 

### HDFS 

HDFS 为HBase 提供最终的底层数据存储服务，同时为HBase 提供高可用的支持。 

# HBase的读写流程

总结大致的流程就是：client请求到Zookeeper，然后Zookeeper返回HRegionServer地址给client，client得到Zookeeper返回的地址去请求HRegionServer，HRegionServer读写数据后返回给client。

![读写流程大致总结](https://cos.duktig.cn/typora/202111071521122.jpg)

## 写流程

![写流程](https://cos.duktig.cn/typora/202111071520018.png)

写流程： 

1. Client 先访问zookeeper，获取hbase:meta 表位于哪个Region Server。 
2. 访问对应的 Region Server，获取 hbase:meta 表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个 Region Server 中的哪个 Region 中。并将该 table 的 region 信息以及meta 表的位置信息缓存在客户端的 meta cache，方便下次访问。 
3. 与目标Region Server 进行通讯； 
4. 将数据顺序写入（追加）到 WAL； 
5. 将数据写入对应的 MemStore，数据会在MemStore 进行排序； 
6. 向客户端发送ack； 
7. 等达到MemStore 的刷写时机后，将数据刷写到HFile。 

## MemStore Flush流程

![Flush流程](https://cos.duktig.cn/typora/202111071529180.png)

MemStore 刷写时机（具体配置参看hbase-default.xml）： 

1、当某个 memstroe 的大小达到了 `hbase.hregion.memstore.flush.size（默认值 128M）`，其 **所在region 的所有 memstore 都会刷写** 。 
当 memstore 的大小达到了 `hbase.hregion.memstore.flush.size（默认值 128M）* hbase.hregion.memstore.block.multiplier（默认值 4）` 时，**会阻止继续往该 memstore 写数据**。 

2、当region server 中memstore 的总大小达到 `java_heapsize * hbase.regionserver.global.memstore.size（默认值 0.4） * hbase.regionserver.global.memstore.size.lower.limit（默认值0.95）`， region 会按照其所有 memstore 的大小顺序（由大到小）依次进行刷写。直到 region server 中所有 memstore 的总大小减小到上述值以下。 
当 region server 中memstore 的总大小达到 `java_heapsize * hbase.regionserver.global.memstore.size（默认值 0.4）`时，会阻止继续往所有的memstore 写数据。 

3、 到达自动刷写的时间，也会触发memstore flush。自动刷新的时间间隔由该属性进行配置`hbase.regionserver.optionalcacheflushinterval（默认1 小时）`。 

4、当 WAL 文件的数量超过 `hbase.regionserver.max.logs`，region 会按照时间顺序依次进行刷写，直到 WAL 文件数量减小到 `hbase.regionserver.max.log` 以下（该属性名已经废弃，现无需手动设置，最大值为 32）。

## 读流程

![读流程](https://cos.duktig.cn/typora/202111071546440.png)

读流程 ：

1. Client 先访问zookeeper，获取hbase:meta 表位于哪个Region Server。
2. 访问对应的 Region Server，获取 hbase:meta 表，根据读请求的 namespace:table/rowkey，查询出目标数据位于哪个 Region Server 中的哪个 Region 中。并将该 table 的 region 信息以及meta 表的位置信息缓存在客户端的 meta cache，方便下次访问。 
3. 与目标Region Server 进行通讯； 
4. **分别在Block Cache（读缓存），MemStore 和 Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。**此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）。 
5. **将从文件中查询到的数据块（Block，HFile 数据存储单元，默认大小为64KB）缓存到Block Cache**。 
6. **将合并后的最终结果返回给客户端**。 

## StoreFile Compaction

由于memstore 每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（Put/Delete）有可能会分布在不同的HFile 中，因此查询时需要遍历所有的HFile。**为了减少HFile 的个数，以及清理掉过期和删除的数据**，会进行 StoreFile Compaction。 

Compaction 分为两种，分别是 Minor Compaction 和 Major Compaction：

- Minor Compaction会将临近的若干个较小的 HFile 合并成一个较大的 HFile，但不会清理过期和删除的数据。
- Major Compaction 会将一个 Store 下的所有的 HFile 合并成一个大 HFile，并且会清理掉过期和删除的数据。 

![StoreFile Compaction](https://cos.duktig.cn/typora/202111071558675.png)

## Region Split 

默认情况下，每个 Table 起初只有一个Region，随着数据的不断写入，Region 会自动进
行拆分。刚拆分时，两个子 Region 都位于当前的 Region Server，但处于负载均衡的考虑，HMaster 有可能会将某个 Region 转移给其他的Region Server。 

Region Split 时机： 

1. 当1 个region 中的某个Store 下所有StoreFile 的总大小超过`hbase.hregion.max.filesize`，该Region 就会进行拆分（0.94 版本之前）。 
2. 当 1 个 region 中的某个 Store 下所有 StoreFile 的总大小超过 `Min(R^2 * 
   "hbase.hregion.memstore.flush.size",hbase.hregion.max.filesize")`，该Region 就会进行拆分，其中R 为当前Region Server 中属于该Table 的个数（0.94 版本之后）。 

![Region Split ](https://cos.duktig.cn/typora/202111071602842.png)

# HBase API

## 相关配置

```java
private Connection connection = null;
private Admin admin = null;

/**
  * 初始化配置
  */
public void init() {
    //使用HBaseConfiguration 的单例方法实例化
    Configuration conf = HBaseConfiguration.create();
    try {
        connection = ConnectionFactory.createConnection(conf);
        admin = connection.getAdmin();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

/**
 * 关闭资源
 */
public void close() {
    if (admin != null) {
        try {
            admin.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    if (connection != null) {
        try {
            connection.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**测试基础方法**：

执行前初始化 `HBaseAPI` 对象，并进行相关配置；执行结束后，关闭必要资源。

```java
public class HBaseAPITest {

    private HBaseAPI hBaseAPI = null;

    @Before
    public void init() {
        hBaseAPI = new HBaseAPI();
        hBaseAPI.init();
    }

    @After
    public void close() {
        hBaseAPI.close();
    }

}
```

## DDL相关操作

```java
/**
 * @return 表是否存在
 */
public boolean isTableExist(String tableName) {
    try {
        return admin.tableExists(TableName.valueOf(tableName));
    } catch (IOException e) {
        e.printStackTrace();
    }
    return false;
}

/**
 * 创建表
 *
 * @param tableName      表明
 * @param columnFamilies 列族列表
 */
public void createTable(String tableName, String... columnFamilies) {
    //判断表是否存在
    if (isTableExist(tableName)) {
        System.out.println("表" + tableName + "已存在");
    } else {
        //创建表属性对象,表名需要转字节
        HTableDescriptor descriptor = new HTableDescriptor(TableName.valueOf(tableName));
        //创建多个列族
        for (String cf : columnFamilies) {
            descriptor.addFamily(new HColumnDescriptor(cf));
        }
        //根据对表的配置，创建表
        try {
            admin.createTable(descriptor);
            System.out.println("表" + tableName + "创建成功！");
        } catch (IOException e) {
            System.out.println("表" + tableName + "创建失败！");
            e.printStackTrace();
        }
    }
}

/**
 * 删除表
 */
public void dropTable(String tableName) {
    if (isTableExist(tableName)) {
        try {
            admin.disableTable(TableName.valueOf(tableName));
            admin.deleteTable(TableName.valueOf(tableName));
        } catch (IOException e) {
            System.out.println("表" + tableName + "删除失败！");
            e.printStackTrace();
        }
        System.out.println("表" + tableName + "删除成功！");
    } else {
        System.out.println("表" + tableName + "不存在！");
    }
}
```

## DML操作

### 向表中插入数据

```java
/**
 * 向表中插入数据
 *
 * @param tableName    表
 * @param rowKey       行键
 * @param columnFamily 列族
 * @param column       列名
 * @param value        值
 */
public void put(String tableName, String rowKey, String columnFamily, String column, String value) {
    try {
        // 创建HTable对象
        Table table = connection.getTable(TableName.valueOf(tableName));
        // 创建put对象
        Put put = new Put(Bytes.toBytes(rowKey));
        //向Put对象中组装数据
        put.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(column), Bytes.toBytes(value));
        table.put(put);
        table.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

如果需要插入多个列，可以重复调用方法 `put.addColumn()`。

### 获取所有行数据

```java
/**
 * 获取所有行数据
 */
public void scan(String tableName) {
    // 创建HTable对象
    try {
        Table table = connection.getTable(TableName.valueOf(tableName));
        ResultScanner scanner = table.getScanner(new Scan());
        for (Result result : scanner) {
            Cell[] cells = result.rawCells();
            for (Cell cell : cells) {
                //得到rowkey
                System.out.println(" 行 键 :" + Bytes.toString(CellUtil.cloneRow(cell)));
                //得到列族
                System.out.println(" 列 族 " + Bytes.toString(CellUtil.cloneFamily(cell)));
                System.out.println(" 列 :" + Bytes.toString(CellUtil.cloneQualifier(cell)));
                System.out.println(" 值 :" + Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

```



### 获取某一行数据

```java
/**
 * 获取某一行数据
 */
public void get(String tableName, String rowKey) {
    // 创建HTable对象
    try {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(Bytes.toBytes(rowKey));
        //get.setMaxVersions();显示所有版本
        //get.setTimeStamp();显示指定时间戳的版本
        Result result = table.get(get);
        for (Cell cell : result.rawCells()) {
            System.out.println(" 行 键 :" +
                    Bytes.toString(result.getRow()));
            System.out.println(" 列 族 " +
                    Bytes.toString(CellUtil.cloneFamily(cell)));
            System.out.println(" 列 :" +
                    Bytes.toString(CellUtil.cloneQualifier(cell)));
            System.out.println(" 值 :" +
                    Bytes.toString(CellUtil.cloneValue(cell)));
            System.out.println("时间戳:" + cell.getTimestamp());
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

/**
 * 获取某一行数据
 */
public void get(String tableName, String rowKey, String family, String qualifier) {
    // 创建HTable对象
    try {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(Bytes.toBytes(rowKey));
        //get.setMaxVersions();显示所有版本
        //get.setTimeStamp();显示指定时间戳的版本
        get.addColumn(Bytes.toBytes(family), Bytes.toBytes(qualifier));
        Result result = table.get(get);
        for (Cell cell : result.rawCells()) {
            System.out.println(" 行 键 :" +
                    Bytes.toString(result.getRow()));
            System.out.println(" 列 族 " +
                    Bytes.toString(CellUtil.cloneFamily(cell)));
            System.out.println(" 列 :" +
                    Bytes.toString(CellUtil.cloneQualifier(cell)));
            System.out.println(" 值 :" +
                    Bytes.toString(CellUtil.cloneValue(cell)));
            System.out.println("时间戳:" + cell.getTimestamp());
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

```



### 删除数据

```java
/**
 * 删除行数据
 */
public void delete(String tableName, String... rows) {
    try {
        Table table = connection.getTable(TableName.valueOf(tableName));
        List<Delete> deleteList = new ArrayList<>();
        for (String row : rows) {
            Delete delete = new Delete(Bytes.toBytes(row));
            deleteList.add(delete);
        }
        table.delete(deleteList);
        table.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## DDL和DML测试

```java
@Test
public void isTableExist() {
    boolean exist = hBaseAPI.isTableExist("test");
    System.out.println(exist);
}

@Test
public void createTable() {
    hBaseAPI.createTable("user", "student");
}

@Test
public void dropTable() {
    hBaseAPI.dropTable("user");
}

@Test
public void addRowData() {
    hBaseAPI.put("user", "1", "student", "name", "小二");
    hBaseAPI.put("user", "2", "student", "name", "张三");
    hBaseAPI.put("user", "3", "student", "name", "李四");
    hBaseAPI.put("user", "4", "student", "name", "王五");
}

@Test
public void scan() {
    hBaseAPI.scan("user");
}

@Test
public void get() {
    hBaseAPI.get("user", "2");
    hBaseAPI.get("user", "3", "student", "name");
}

@Test
public void delete() {
    hBaseAPI.delete("user", "2", "4");
}
```



# MapReduce 与 HBase

## 配置

win10下的HBase在hadoop-env.cmd下配置：

```
set HADOOP_CLASSPATH=D:\javaweb\bigdata\hbase-1.4.9\lib\*
```

linux下的HBase在hadoop-env.sh下配置：

```sh
export HADOOP_CLASSPATH=D:\javaweb\bigdata\hbase-1.4.9\lib\*
```

然后重启hadoop和hbase

如果不配置会报错：

```sh
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/hadoop/hbase/filter/Filter
```

## 官方案例

### 案例1

在Hadoop的bin目录下执行

```sh
yarn jar %HBASE_HOME%\lib\hbase-server-1.4.9.jar rowcounter user
```

### 案例2

创建文件fruit.txt

```
1001 Apple Red 
1002 Pear Yellow 
1003 Pineapple Yellow 
```

将文件上传到hdfs

```sh
hadoop fs -mkdir /hbase
hadoop fs -put  fruit.txt /hbase
```

执行MapReduce 到HBase 的fruit 表中 

```sh
yarn jar %HBASE_HOME%\lib\hbase-server-1.4.9.jar  importtsv -Dimporttsv.columns=HBASE_ROW_KEY,info:name,info:color  fruit hdfs://127.0.0.1:9000/hbase
```

## 自定义HBase-MapReduce

目标：实现将HDFS 中的数据写入到Hbase 表中

1、构建 ReadFruitFromHDFSMapper 于读取 HDFS 中的文件数据 

```java
public class ReadFruitFromHdfsMapper extends Mapper<LongWritable, Text, ImmutableBytesWritable, Put> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //从 HDFS 中读取的数据
        String lineValue = value.toString();
        //读取出来的每行数据使用\t 进行分割，存于String 数组
        String[] values = lineValue.split("\t");

        //根据数据中值的含义取值
        String rowKey = values[0];
        String name = values[1];
        String color = values[2];

        //初始化 rowKey
        ImmutableBytesWritable rowKeyWritable = new ImmutableBytesWritable(Bytes.toBytes(rowKey));
        //初始化 put对象
        Put put = new Put(Bytes.toBytes(rowKey));
        //参数分别:列族、列、值
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes(name));
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("color"), Bytes.toBytes(color));

        context.write(rowKeyWritable, put);
    }
}
```

2、构建 WriteFruitMRFromTxtReducer 类 

```java
public class WriteFruitMRFromTxtReducer extends TableReducer<ImmutableBytesWritable, Put, NullWritable> {

    @Override
    protected void reduce(ImmutableBytesWritable key, Iterable<Put> values, Context context) throws IOException,
            InterruptedException {
        //读出来的每一行数据写入到fruit_hdfs表中
        for (Put put : values) {
            context.write(NullWritable.get(), put);
        }
    }
}
```

3、创建 TxtFruitRunner 装 Job 

```java
public class TxtFruitRunner extends Configured implements Tool {


    @Override
    public int run(String[] strings) throws Exception {
        //得到Configuration
        Configuration conf = this.getConf();

        //创建Job 任务
        Job job = Job.getInstance(conf, this.getClass().getSimpleName());
        job.setJarByClass(TxtFruitRunner.class);
        Path inPath = new Path("hdfs://127.0.0.1:9000/hbase/fruit.txt");
        FileInputFormat.addInputPath(job, inPath);

        //设置Mapper
        job.setMapperClass(ReadFruitFromHdfsMapper.class);
        job.setMapOutputKeyClass(ImmutableBytesWritable.class);
        job.setMapOutputValueClass(Put.class);

        //设置Reducer
        TableMapReduceUtil.initTableReducerJob("fruit", WriteFruitMRFromTxtReducer.class, job);

        //设置Reduce数量，最少 1个
        job.setNumReduceTasks(1);

        boolean isSuccess = job.waitForCompletion(true);
        if (! isSuccess) {
            throw new IOException("Job running with error");
        }

        return isSuccess ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = HBaseConfiguration.create();
        int status = ToolRunner.run(conf, new TxtFruitRunner(), args);
        System.exit(status);
    }

}
```



加载hive-jdbc driver时报错：`java.lang.NoClassDefFoundError:org/apache/hadoop/conf/Configuration`

解决：

引入依赖：

```xml
<dependency>
  <groupId>org.apache.hadoop</groupId>
  <artifactId>hadoop-common</artifactId>
  <version>3.1.3</version>
</dependency>
```

报错：`java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument`

错误原因：hive和hadoop的lib下面的guava.jar版本不一致造成的。

解决：删除hive的lib下面低版本guava.jar，换成和hadoop一致的guava.jar。

# 与Hive集成

## HBase 与 Hive 的对比

### Hive 

(1) 数据仓库 

Hive 的本质其实就相当于将 HDFS 中已经存储的文件在 Mysql 中做了一个双射关系，以方便使用HQL 去管理查询。 

(2) 用于数据分析、清洗 

Hive 适用于离线的数据分析和清洗，延迟较高。


(3) 基于HDFS、MapReduce 

Hive 存储的数据依旧在DataNode 上，编写的HQL 语句终将是转换为MapReduce 代码执行。 

### HBase 

(1) 数据库 

是一种面向列族存储的非关系型数据库。 

(2) 用于存储结构化和非结构化的数据 

适用于单表非关系型数据的存储，不适合做关联查询，类似JOIN 等操作。


(3) 基于HDFS 

数据持久化存储的体现形式是HFile，存放于 DataNode 中，被ResionServer 以 region 的形式进行管理。 

(4) 延迟较低，接入在线业务使用 

面对大量的企业数据，HBase 可以直线单表大量数据的存储，同时提供了高效的数据访问速度。

## HBase 与 Hive 集成使用

Hive提供了与HBase的集成，使得能够在HBase表上使用HQL语句进行查询 插入操作以及进行Join和Union等复杂查询、同时也可以将hive表中的数据映射到Hbase中。

使用场景：

- 将ETL操作的数据存入HBase
- HBase作为Hive的数据源
- 构建低延时的数据仓库



参看：

- [我终于看懂了HBase，太不容易了...](https://zhuanlan.zhihu.com/p/145551967)
- [一文读懂 HBase 使用场景](https://blog.csdn.net/u011598442/article/details/89891926)

