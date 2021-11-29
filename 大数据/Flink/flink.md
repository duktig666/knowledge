# Flink概述

## Flink简介

Apache Flink 是一个框架和分布式处理引擎，用于对 **无界和有界数据流** 进行**状态**计算。

## 为什么要使用Flink？

- 流数据更真实地反映了我们的生活方式
- 传统的数据架构是基于有限数据集的
- 目标
  - 低延迟（Spark Streaming 的延迟是秒级，Flink 延迟是毫秒级）
  - 高吞吐（阿里每秒钟使用 Flink 处理 4.6PB，双十一大屏）
  - 结果的准确性和良好的容错性（exactly-once）

## 流数据使用场景

电商和市场营销：数据报表、广告投放、业务流程需要

物联网（IOT）：传感器实时数据采集和显示、实时报警，交通运输业（自动
驾驶）

电信业：基站流量调配

银行和金融业：实时结算和通知推送，实时检测异常行为（信用卡盗卡）

## Flink的运行架构

### 传统数据处理架构

事务处理（OLTP）：

![image-20211122200218443](https://cos.duktig.cn/typora/202111222003529.png)

分析处理（OLAP）：

将数据从业务数据库复制到数仓，再进行分析和查询

![image-20211122200257708](https://cos.duktig.cn/typora/202111222003696.png)

### 有状态的流式处理

![image-20211122200406028](https://cos.duktig.cn/typora/202111222004362.png)

lambda 架构：用两套系统，同时保证低延迟和结果准确

![image-20211122200436367](https://cos.duktig.cn/typora/202111222004159.png)

## 流处理的演变

![image-20211122200512855](https://cos.duktig.cn/typora/202111222005626.png)

## Flink的主要特点

**事件驱动**

![image-20211122201146939](https://cos.duktig.cn/typora/202111222011416.png)



**基于流的世界观**

在 Flink 的世界观中，一切都是由流组成的，离线数据是有界的流；实时数据是一个没有界限的流：这就是所谓的有界流和无界流

![image-20211122201457567](https://cos.duktig.cn/typora/202111222015734.png)

**Flink 的分层 API**

越顶层越抽象，表达含义越简明，使用越方便

越底层越具体，表达能力越丰富，使用越灵活

![image-20211122201704054](https://cos.duktig.cn/typora/202111222017452.png)

其他：

- 支持事件时间（event-time）和处理时间（processing-time）语义
- 精确一次（exactly-once）的状态一致性保证
- 低延迟，每秒处理数百万个事件，毫秒级延迟（实际上就是没有延迟）
- 与众多常用存储系统的连接（ES，HBase，MySQL，Redis⋯）
- 高可用（zookeeper），动态扩展，实现 7*24 小时全天候运行

## Flink和Spark Streaming的区别

- 流（stream）和微批
- 数据模型
  - Spark 采用 RDD 模型，Spark Streaming 的 DStream 实际上也就是一组组小批数据 RDD 的集合
  - Flink 基本数据模型是数据流，以及事件（Event）序列（Integer、String、Long、POJO Class）
- 运行时架构
  - Spark 是批计算，将 DAG 划分为不同的 Stage，一个 Stage完成后才可以计算下一个 Stage
  - Flink 是标准的流执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理

# Flink安装

## win10安装flink

建议使用1.9.3的版本，1.10和1.11有问题。

安装参看：[win10下安装Flink](https://blog.csdn.net/yamaxifeng_132/article/details/102717306)

# Flink运行架构

## Flink 运行时的组件

Flink 运行时由两种类型的进程组成：一个 JobManager 和一个或者多个 TaskManager。

典型的 Master-Slave 架构

![image-20211122211018059](https://cos.duktig.cn/typora/202111222110310.png)



### 作业管理器 (JobManager)

控制一个应用程序执行的主进程，也就是说，每个应用程序都会被一个不同的 JobManager 所控制执行。

JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图 (JobGraph)、逻辑数据流图和打包了所有的类、库和其它资源的 JAR 包。

JobManager 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫做“执行图”(ExecutionGraph)，包含了所有可以并发执行的任务。

JobManager 会向资源管理器 (Flink 的资源管理器) 请求执行任务必要的资源，也就是任务管理器 (TaskManager) 上的任务插槽（slot）。一旦它获取到了足够的资源，就会将执行图 (DAG) 分发到真正运行它们的 TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检查点 (checkpoints) 的协调。

### 资源管理器 (ResourceManager)

主要负责管理任务管理器(TaskManager)的插槽(slot) ,TaskManager插槽是Flink中定义的处理资源单元。

Flink为不同的环境和资源管理工具提供了不同资源管理器，比如YARN、Mesos、Kubernetes(管理docker容器组成的集群)，以及Standalone(独立集群)部署。

当JobManager 申请插槽资源时，Flink的资源管理器会将有空闲插槽的TaskManager 分配给JobManager。如果Flink的资源管理器没有足够的插槽来满足JobManager 的请求，它还可以向Yarn的资源管理器发起会话，以提供启动TaskManager进程的容器。

### 分发器 (Dispatcher)

可以跨作业运行，它为应用提交提供了RESTful接口(GET/PUT/DELETE/POST)。

当一个应用被提交执行时，分发器就会启动并将应用移交给一个JobManager。

Dispatcher 也会启动一个 Web Ul(localhost:8081)，用来方便地展示和监控作业行的信息。

Dispatcher 在架构中可能并不是必需的，这取决于应用提交运行的方式。

### JobMaster

JobMaster 负责管理单个 JobGraph 的执行。Flink 集群中可以同时运行多个作业，每个作业都有自己的JobMaster。

## 任务提交流程

![image-20211122212648752](https://cos.duktig.cn/typora/202111222126258.png)

![image-20211122212954588](https://cos.duktig.cn/typora/202111222130840.png)



![image-20211122213417523](https://cos.duktig.cn/typora/202111222134372.png)

## TaskManager 和 Slots

![image-20211122213642814](https://cos.duktig.cn/typora/202111222136599.png)

Flink 中每一个 TaskManager 都是一个 JVM 进程，每一个任务插槽都会启动一个线程，它可能会在独立的线程上执行一个或多个 subtask，每一个子任务占用一个任务插槽（Task Slot）

为了控制一个 TaskManager 能接收多少个 task，TaskManager 通过 task slot 来进行控制（一个 TaskManager至少有一个 slot）

默认情况下，Flink 允许子任务共享 slot。这样的结果是，一个 slot 可以保存作业的整个管道。

Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力。

## 程序与数据流 (DataFlow)

![image-20211122215310212](https://cos.duktig.cn/typora/202111222153198.png)

所有的 Flink 程序都是由三部分组成的：Source、Transformation 和 Sink。

Source 负责读取数据源，Transformation 利用各种算子进行处理加工，Sink 负责输出。

在运行时，Flink 上运行的程序会被映射成“逻辑数据流”（dataflows），它包含了这三部分

每一个 dataflow 以一个或多个 sources 开始以一个或多个sinks 结束。dataflow 类似于任意的有向无环图（DAG）

在大部分情况下，程序中的转换运算（transformations）跟dataflow 中的算子（operator）是一一对应的关系

![image-20211122215338526](https://cos.duktig.cn/typora/202111222153496.png)

## 图数据结构的转化

![image-20211122215511531](https://cos.duktig.cn/typora/202111222155154.png)

- StreamGraph：是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构。
- JobGraph：StreamGraph 在编译的阶段经过优化后生成了JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件（窄依赖，没有 shuffle）的算子 chain 在一起作为一个节点。
- ExecutionGraph：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph 是 JobGraph 的并行化版本，是调度层最核心的数据结构。
- 物理执行图：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个 TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。 

![slide2](https://cos.duktig.cn/typora/202111222203969.png)

## 并行度

![image-20211122220208047](https://cos.duktig.cn/typora/202111222202799.png)

![image-20211122220433482](https://cos.duktig.cn/typora/202111222204741.png)

-  一个特定算子的子任务（subtask）的个数被称之为其并行度（parallelism）。一般情况下，一个 stream 的并行度，可以认为就是其所有算子中最大的并行度。
- 算子之间传输数据的形式可以是 one-to-one (forwarding) 的模式也可以是 redistributing 的模式，具体是哪一种形式，取决于算子的种类
  - One-to-one：stream 维护着分区以及元素的顺序（比如source 和 map 之间）。这意味着 map 算子的子任务看到的元素的个数以及顺序跟 source 算子的子任务生产的元素的个数、顺序相同。map、filter、flatMap 等算子都是 one-to-one的对应关系。
  - Redistributing：stream 的分区会发生改变。每一个算子的子任务依据所选择的 transformation 发送数据到不同的目标任务。例如，keyBy 基于 hashCode 重分区、而 broadcast 和rebalance 会随机重新分区，这些算子都会引起 redistribute
    过程，而 redistribute 过程就类似于 Spark 中的 shuffle 过程。

## 任务链

![slide2](https://cos.duktig.cn/typora/202111222206707.png)

Flink 采用了一种称为任务链的优化技术，可以在特定条件下减少本地通信的开销。为了满足任务链的要求，必须将两个或多个算子设为相同的并行度，并通过本地转发（localforward）的方式进行连接

相同并行度的 one-to-one 操作，Flink 这样相连的算子链接在一起形成一个 task，原来的算子成为里面的 subtask

并行度相同、并且是 one-to-one 操作，两个条件缺一不可

# Flink的API

## 常用api操作

1. 声明流执行的环境

   ```
   StreamExecutionEnvironment env  = StreamExecutionEnvironment.getExecutionEnvironment();
   ```

2. 设置并行度（数值一般与cpu核心数一致）

   ```java
   env.setParallelism(1);
   ```

3. 设置数据源（有多种方式可供选择）

   ```java
   //1. 从离线数据中读取
   env.fromElements("Mary", "Bob", "Alice", "Liz");
   //2. 从文件中读取
   env.readTextFile("UserBehavior.csv")
   //3. 从 socket 中读取
   env.socketTextStream(host, port);
   ```

**从 Kafka 读取数据**

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
```

```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
properties.setProperty("group.id", "consumer-group");
properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("auto.offset.reset", "latest");

env.addSource(new FlinkKafkaConsumer<String>(
    "userbehavior",
    new SimpleStringSchema(),
    properties
));
```

**自定义数据源读取**

```java
public class CustomSourceExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
		
        // 设置自定义的数据源
        DataStreamSource<Event> stream = env.addSource(new ClickSource());

        stream.print();

        env.execute();
    }

    /**
     * SourceFunction并行度只能为1
     * 自定义并行化版本的数据源，需要使用ParallelSourceFunction
     */
    public static class ClickSource implements SourceFunction<Event> {
        private boolean running = true;
        private String[] userArr = {"Mary", "Bob", "Alice", "Liz"};
        private String[] urlArr = {"./home", "./cart", "./fav", "./prod?id=1", "./prod?id=2"};
        private Random random = new Random();

        @Override
        public void run(SourceContext<Event> ctx) throws Exception {
            while (running) {
                // collect方法，向下游发送数据
                ctx.collect(
                        new Event(
                                userArr[random.nextInt(userArr.length)],
                                urlArr[random.nextInt(urlArr.length)],
                                Calendar.getInstance().getTimeInMillis()
                        )
                );
                Thread.sleep(1000L);
            }
        }

        @Override
        public void cancel() {
            running = false;
        }
    }

    public static class Event {
        public String user;
        public String url;
        public Long timestamp;

        public Event() {
        }

        public Event(String user, String url, Long timestamp) {
            this.user = user;
            this.url = url;
            this.timestamp = timestamp;
        }

        @Override
        public String toString() {
            return "Event{" +
                    "user='" + user + '\'' +
                    ", url='" + url + '\'' +
                    ", timestamp=" + new Timestamp(timestamp) +
                    '}';
        }
    }

}
```

**Flink程序结构如下：**

1. 创建Flink 程序执行环境。

2. 从数据源读取一条或者多条流数据
3. 使用流转换算子实现业务逻辑
4. 将计算结果输出到一个或者多个外部设备（可选）
5. 执行程序

## 基本转换算子

基本转换算子的定义：作用在数据流中的每一条单独的数据上的算子。

基本转换算子会针对流中的每一个单独的事件做处理，也就是说每一个输入数据会产生一个输出数据。单值转换，数据的分割，数据的过滤，都是基本转换操作的典型例子。我们将解释这些算子的语义并提供示例代码。

Map：抽取字段

```java
readings.map(r -> r.itemId);

readings.map(new MapFunction<UserBehavior, String>() {
@Override
public String map(UserBehavior r) throws Exception {
return r.itemId;
}
});

readings.map(new IdExtractor());

public static class IdExtractor implements MapFunction<UserBehavior, String> {
@Override
public String map(UserBehavior r) throws Exception {
return r.itemId;
}
}
```

Fliter：过滤字段

```java
readings.filter(r -> r.behaviorType.equals("pv"));

readings.filter(new FilterFunction<UserBehavior>() {
    @Override
    public Boolean filter(UserBehavior r) throws Exception {
        return r.behaviorType.equals("pv");
    }
});

readings.filter(new PvExtractor());

public static class IdExtractor implements FilterFunction<UserBehavior> {
    @Override
    public Boolean filter(UserBehavior r) throws Exception {
        return r.behaviorType.equals("pv");
    }
}
```

FlatMap：复杂操作

```java
DataStreamSource<String> stream = env.fromElements("white", "black", "gray");

stream.flatMap(
    (FlatMapFunction<String, String>)(s, out) -> {
        if (s.equals("white")) {
            out.collect(s);
        } else if (s.equals("black")) {
            out.collect(s);
            out.collect(s);
        }
    })
    .returns(Types.STRING)
    .print();

stream.flatMap(new MyFlatMap()).print();

public static class MyFlatMap implements FlatMapFunction<String, String> {
    @Override
    public void flatMap(String value, Collector<String> out) throws Exception {
        if (value.equals("white")) {
            out.collect(value);
        } else if (value.equals("black")) {
            out.collect(value);
            out.collect(value);
        }
    }
}
```

## 键控流转换算子

很多流处理程序的一个基本要求就是要能对数据进行分组，分组后的数据共享某一个相同的属性。DataStream API 提供了一个叫做KeyedStream 的抽象，此抽象会从逻辑上对 DataStream 进行分区，分区后的数据拥有同样的 Key 值，分区后的流互不相关。

KeyedStream 可以使用map，ﬂatMap 和ﬁlter 算子来处理。使用keyBy 算子来将DataStream 转换成KeyedStream，基于key 的转换操作：滚动聚合和reduce算子。

### KEYBY

keyBy 通过指定 key 来将 DataStream 转换成 KeyedStream。基于不同的 key，流中的事件将被分配到不同的分区中去。所有具有相同 key 的事件将会在接下来的操作符的同一个子任务槽中进行处理。拥有不同 key 的事件可以在同一个任务中处理。但是算子只能访问当前事件的key 所对应的状态。

```java
KeyedStream<UserBehavior, String> keyed = stream.keyBy(r -> r.itemId);

stream.keyBy(new KeySelector<UserBehavior, String>() {
        @Override
        public String getKey(UserBehavior value) throws Exception {
            return value.itemId;
        }
    });
```

### 滚动聚合

滚动聚合算子由KeyedStream 调用，并生成一个聚合以后的DataStream，例如：sum，minimum，maximum。一个滚动聚合算子会为每一个观察到的key 保存一个聚合的值。针对每一个输入事件，算子将会更新保存的聚合结果，并发送一个带有更新后的值的事件到下游算子。滚动聚合不需要用户自定义函数，但需要接受一个参数，这个参数指定了
在哪一个字段上面做聚合操作。DataStream API 提供了以下滚动聚合方法。

- sum()：在输入流上对指定的字段做滚动相加操作。
- min()：在输入流上对指定的字段求最小值。
- max()：在输入流上对指定的字段求最大值。
- minBy()：在输入流上针对指定字段求最小值，并返回包含当前观察到的最小值的事件。
- maxBy()：在输入流上针对指定字段求最大值，并返回包含当前观察到的最大值的事件。

滚动聚合算子无法组合起来使用，每次计算只能使用一个单独的滚动聚合算子。

下面的例子根据第一个字段来对类型为 Tuple3<Int, Int, Int> 的流做分流操作，然后
针对第二个字段做滚动求和操作。

```java
DataStreamSource<Tuple3<Integer, Integer, Integer>> inputStream = env.fromElements(
    Tuple3.of(1, 2, 2),
    Tuple3.of(2, 3, 1),
    Tuple3.of(2, 2, 4),
    Tuple3.of(1, 5, 3)
);

DataStream<Tuple3<Integer, Integer, Integer>> resultStream = inputStream
    .keyBy(0) // key on first field of the tuple
    .sum(1); // sum the second field of the tuple in place
```

### REDUCE

reduce 算子是滚动聚合的泛化实现。它将一个 ReduceFunction 应用到了一个 KeyedStream 上面去。reduce 算子将会把每一个输入事件和当前已经 reduce 出来的值做聚合计算。reduce 操作不会改变流的事件类型。输出流数据类型和输入流数据类型是一样的。

reduce 函数可以通过实现接口 ReduceFunction 来创建一个类。ReduceFunction 接口定义了 reduce() 方法，此方法接收两个输入事件，输出一个相同类型的事件。

```java
inputStream
    .keyBy(r -> r.f0)
    .reduce(new ReduceFunction<Tuple3<Integer, Integer, Integer>>() {
        @Override
        public Tuple3<Integer, Integer, Integer> reduce(Tuple3<Integer, Integer,Integer> value1, Tuple3<Integer, Integer, Integer> value2) throws Exception{
            if (value1.f1 > value2.f1) {
                return value1;
            } else {
                return value2;
            }
        }
    }).print();
```



























