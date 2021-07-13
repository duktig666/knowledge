# 面试技巧

## 面试瓶颈在哪里？

1. 小项目行不通，要见识高难度项目
2. 要真正理解核心的底层原理，并且知道用在哪
3. “理论经验”到“实践经验”的转变

## 系统架构

![系统架构](https://gitee.com/koala010/typora/raw/master/img/20210710095909.png)

![系统架构2](https://gitee.com/koala010/typora/raw/master/img/20210710134805.png)

## 重点问题

![image-20210710134911482](https://gitee.com/koala010/typora/raw/master/img/20210710134911.png)

## 一面

展现综合素质，加深面试官印象

一般问题：

- 如何做自我介绍
- 聊聊项目
- 提问环节
- 你有什么要问的吗

### 自我介绍

- 3-4min
- 结构性：经历简介、项目经历、技术总结
- 凸显能力：技术经验能力、学习思考能力

技术经验能力：技术栈、使用在什么场景中

#### 介绍前的思考

- 你的一面面试官和你未来工作的关系?
- 他想要从我的自我介绍中了解什么?（经历、项目、总结）

#### 介绍内容

经历简介（快速）

个人信息，教育背景，职业生涯，工作年限。

项目经历

电商项目:微服务，领域模型驱动设计，结果产出,qps，高并发场景，担当什么?

技能总结

你的技术栈，开发语言，主要框架，中间件(存储,微服务相关，大数据，部署相关)

业余时间学习or最近在学什么（一定要补充？）

#### 提问

- 介绍下你做的项目
- 你项目的业务场景，系统架构，工作职责分别是什么样的
- 你的项目中有哪些亮点，你是如何处理和解决这些难点问题的

### 一面上半场

聊聊项目：

- 项目场景介绍
  - 项目做了什么？
- 系统架构方案
  - 系统架构方案
  - 业务架构方案
- 你负责了哪块？

#### 系统架构

系统架构，最好可以手画

![商品服务架构图](https://gitee.com/koala010/typora/raw/master/img/20210710143750.png)



#### 业务架构

![领域模型](https://gitee.com/koala010/typora/raw/master/img/20210710144247.png)

![支付流程（成功）](https://gitee.com/koala010/typora/raw/master/img/20210710144701.png)

![支付流程（失败）](https://gitee.com/koala010/typora/raw/master/img/20210710152011.png)

#### 负责模块

##### 普通研发

- 快速理解需求，产出代码
- 如何充分单元测试,快速上线

##### 系统负责人

- 对系统的边界职责是否清晰
- 系统稳定性考虑:连接池,监控，限流做了吗

##### 架构师

- 整个链路在电商场景中的位置
- 未来的扩展性
- 如何发现瓶颈，快速解决

### 一面下半场

#### 项目过程中你遇到了什么样的难解决问题?你是如何解决的?

常见问题

偏门问题

正常问题

踩坑问题

面试官主要是想看下你做的内容的深度实操经验以及你解决问题的思考和手段

#### 交易一致性问题

##### 重复支付问题

![重复支付流程](https://gitee.com/koala010/typora/raw/master/img/20210710151714.png)

并发情况下，会出现问题：

![伪防重幂等操作](https://gitee.com/koala010/typora/raw/master/img/20210710151755.png)



![防重幂等操作](https://gitee.com/koala010/typora/raw/master/img/20210710151538.png)

##### 超时退问题（分布式系统同步问题）

- 如何回滚内容。
- 回滚失败如何解决
- 重复回滚如何预防

考察点:分布式事务,流水号应用，重试方式

###### 分布式原理

**CAP**

一致性(Consistency):客户端知道一系列的操作都会同时发生(生效)

可用性(Availability):每个操作都必须以可预期的响应结束

分区容错性(Partition tolerance):即使出现单个组件无法可用,操作依然可以完成

**Base**

Basically Available(基本可用)

Soft state(软状态)

Eventually consistent(最终一致性)

###### 分布式事务

二阶段提交（强一致性）

异步确保型

事务型消息

 TCC型


**二阶段提交**

二阶段提交

读写相等

Raft协议：要求超过一半以上的节点同步成功即可算写成
功，**外加二阶段提交**

![image-20210710153650601](https://gitee.com/koala010/typora/raw/master/img/20210710153650.png)

![image-20210710153729497](https://gitee.com/koala010/typora/raw/master/img/20210710153729.png)

 

![Raft协议](https://gitee.com/koala010/typora/raw/master/img/20210710155238.png)



 **TCC型**

- try
- confirm
- cancel



**异步确保型**

采用异步消息的方式确保事务可以最终—致

交易表

fail支付失败待回滚

voucher_back优惠券退成功

stock_back优惠券退成功

流水表

加入back状态

![image-20210710163421980](https://gitee.com/koala010/typora/raw/master/img/20210710163422.png)



**事务型消息**

![image-20210710163844322](https://gitee.com/koala010/typora/raw/master/img/20210710163844.png)

#### 面试官提问

有没有遇到过java程序崩溃问题?如何排查?

`ps -ef | grep java`

- jps:虚拟机进程状态工具
  - 示例: `jps -v | grep pid`
-  jinfo: jvm参数信息工具
  - 示例: `jinfo -flags pid`
- jstat:查看虚拟机各种运算状态
  - 示例: `jstat -gcutil pid`

> SO:新生代中Survivor space 0区已使用空间的百分比
>
> S1:新生代中Survivor space 1区已使用空间的百分比
>
> E:新生代已使用空间的百分比
>
> 〇:老年代已使用空间的百分比
>
> M:元数据区已使用空间的百分比
>
> CCS:压缩类空间利用率百分比
>
> YGC:从应用程序启动到当前，发生Yang GC的次数
>
> YGCT:从应用程序启动到当前，Yang GC所用的时间【单位秒】
>
> FGC:从应用程序启动到当前，发生Full GC的次数
>
> FGCT:从应用程序启动到当前，Full GC所用的时间
>
> GCT:从应用程序启动到当前,用于垃圾回收的总时间【单位秒】            

- jstack:线程快照工具

  示例:`jstack -l pid`

- jmap: HeapDump工具

  示例:

  `jmap -heap pid`查看堆信息

  `jmap -dump:format=b,file=heapDump.hprof pid` 导出堆文件并用jhat查看

  `jhat -port 8899 heapDump.hprof`

  浏览器访问:http://ip:8899

![image-20210710170721511](https://gitee.com/koala010/typora/raw/master/img/20210710170721.png)

jprofiler——jmap文件可视化

#### 你有什么问题吗？

- 背景:一面的面试官和你职级相同
- 信息了解:最好的了解实际工作内容和细节的机会
- 坦诚表现:对岗位的表现浓厚兴趣

#### 雷点提示

- 一定要准备的很清晰，不能在项目结构上有盲点，否则容易被深问。
- 不是自己负责的部分要么就弄清楚，要么就直接说不清楚

## 二面

java语言基础

- 数据结构
- 多线程高并发
- IO

数据库基础

- 事务ACID特性
- 索引调优

缓存基础

- 数据结构
- 高性能及原子性原因
- 淘汰策略

消息队列基础

- 消息队列集群的结构，常见的名词
- 消息确认机制

网络及操作系统基础

- select和epoll
- http协议进阶
- 进程，线程，协程知识

### 自我介绍

二面一般是你未来的leader

凸显能力:技术基础能力扎实，喜欢探究原理

> 平时喜欢探究源码，有意关注一些底层实现，常用技术的版本更新

### Java语言基础

![Java基础数据结构](https://gitee.com/koala010/typora/raw/master/img/20210710172548.png)

#### ArrayList和LinkedList

#### Hashmap的内存结构, ConcurrentHashMap的加锁力度

阈值小于8链表和红黑树查询效率相差无几，但是红黑树却要进行染色和旋转，性能反而得不偿失

java1.8前，segment对三个槽位加锁。Java1.8转为Node减小加锁粒度

#### LinkedHashMap的加工

LinkedHashMap保证链表的节点插入顺序，一个链表记录插入的顺序。

#### TreeMap的有序性

红黑树，compareTo的旋转和染色

#### HashSet及LinkedHashSet的内部结构

LinkedHashSet继承自HashSet，源码更少、更简单，唯一的区别是LinkedHashSet内部使用的是LinkHashMap。这样做的意义或者好处就是LinkedHashSet中的元素顺序是可以保证的，也就是说遍历序和插入序是一致的。

#### 多线程

- 线程池
- synchronized和ReentrantLock
- 三种锁：偏向锁、轻量级锁、重量级锁
- CopyOnWrite：写串行，读并行
  - 读写并发，写副本并替换，读快照
  - 两次写，加锁，串行写
- volatile

#### IO

- BIO（同步阻塞）
- NIO（同步非阻塞） 广泛应用
- AIO（异步非阻塞） 编程要求较高，类似于Node.js异步事件模型

![image-20210712092721572](https://gitee.com/koala010/typora/raw/master/img/20210712092728.png)

### 数据库

#### 当前读和快照度

MVCC？

可以通过事务的隔离级别进行理解

#### 行锁、表锁、间歇锁

无索引，当前读为表锁

间隙锁：普通索引，无数据锁定区间  左开右闭 ( ]，只对insert操作   select c=6 锁定     5<x<=10

 

mysql当前读对区间进行锁，不包括左右 ()

![image-20210712093918541](https://gitee.com/koala010/typora/raw/master/img/20210712093918.png)

c=10 不仅仅锁定10，     5<x<15    锁间隙，

不能保证别的事务再次insert 10

唯一索引不会产生间隙锁

#### 事务

#### mysql索引

- 经常被查询的区分度高的列做索引

- 最左原则

- 回盘排序

  排序列遵从索引，可建立联合索引

- 覆盖索引

  可以解决非聚簇索引，查找数据的性能问题

- 小表驱动大表   

![image-20210712101455455](https://gitee.com/koala010/typora/raw/master/img/20210712101455.png)

![image-20210712101540220](https://gitee.com/koala010/typora/raw/master/img/20210712101540.png)



调优 explain

### Redis

#### zset

跳表+压缩表

以空间换时间

#### 数据持久化

#### 缓存淘汰策略

#### 单线程及原子性

setnx

### 消息队列

#### 消息消费确认机制

### 网络及操作系统

#### select和epoll的区别

- 数量上限
- 轮询or回调

![image-20210712111341685](https://gitee.com/koala010/typora/raw/master/img/20210712111342.png)



![image-20210712111609218](https://gitee.com/koala010/typora/raw/master/img/20210712111609.png)



#### https加密

- 非对称运算
- 对称运算

#### http2.0 

- 二进制传输
- 多路复用
- 服务端推送

![image-20210712140444960](https://gitee.com/koala010/typora/raw/master/img/20210712140445.png)

### 性能优化

制约程序性能的根源

#### 常用的性能评估指标

- 并发:同一时间多少请求访问
- TPS: transaction per second
- QPS: query per second
- 耗时:端到端耗时，服务端耗时，应用程序耗时
- 95线:95%的请求落在什么范围内
- 99线: 99%的请求落在什么范围内

####  性能问题

- 网络
- 应用本身
- 数据库
- 缓存
- 消息
- 操作系统
- 内存
- lo
- CPU

#### 如何优化?

先进行单机优化

##### JVM内存

![image-20210712142117244](https://gitee.com/koala010/typora/raw/master/img/20210712142117.png)

![image-20210712142154621](https://gitee.com/koala010/typora/raw/master/img/20210712142154.png)

元数据区替代永久代，存放类信蠢，常量，静态变量等，字符串在1.7开始放到了堆中

##### GC算法

##### 分带回收算法

年轻代:1 Eden 2 Survivor复制算法

老年代:标记整理

![image-20210712142618120](https://gitee.com/koala010/typora/raw/master/img/20210712142647.png)

CMS和G1

![image-20210712143417275](https://gitee.com/koala010/typora/raw/master/img/20210712143512.png)

![image-20210712143933728](https://gitee.com/koala010/typora/raw/master/img/20210712143933.png)

![image-20210712144032587](https://gitee.com/koala010/typora/raw/master/img/20210712144104.png)

#### 内存大小的取舍

1. 扩大内存可以更少的触发gc

2. 内存太大触发gc时候的停顿时间会长

因此要根据你实际的业务场景设置成一个“合适”的值，并配合压测和线上环境的实际情况做不断的调优

吞吐量=花费在非GC停顿上的工作时间/总时间至少需要优化到95%

-Xms 启动JVM时堆内存的大小

-Xmx堆内存最大限制

两者需要设置的一样防止扩缩容

-XX:NewSize 年轻代大小

-XX:MaxNewSize最大年轻代大小

两者需要设置的一样防止扩缩容，

-XX:SurvivorRatio Eden Survivor占比，默认为8

Eden需要比Survivor尽可能的大，防止多次触发young gc导致年龄快速增长到可以进入老年代的case

-XX:MetaspaceSize元空间初始空间大小

-XX:MaxMetaspaceSize=512m元空间最大空间，默认是没有限制的

元空间不建议设置

#### GC优化

- 将进入老年代的对象减少到最低
- young gc: 40ms内
- major gc: stop the world时间总和100ms内
- full gc:尽可能少，且时间在1s内

除了cms和g1外，其余的major gc = full gc

**CMS Full GC条件**

Promotion failure:由于内存碎片导致的晋升空间不足

Concurrent mode failed:还未完成cms又触发了下一次major gc

**CMS调优**

-XX:ParallelGCThreads=N设置年轻代的并行收集线程数，避免docker踩坑（早期读物理机的核数，读docker会出问题）

-XX:ParallelCMSThreads=N设置cms的并行收集线程数，避免docker踩坑（指定第一个，即可参考，第二个一般不设置）

-XX:+UseCMSCompactAtFullCotection FullGC情况下的lnitial remark or Final remark都整理内存碎片

-XX:+CMSFullGCsBeforeCompaction=4两次FullGC情况下的Initial remark or Final remark 4次后才整理内存碎片（两个配合，2次full gc触发内存碎片整理）

-XX:+UseCMSInitiatingOccupancyOnly 让阈值驱动cms触发时机

-XX:CMSInitiatingOccupancyFraction=70 70%占满才触发cms（两个配合使用）

-XX:+CMSParallelRemarkEnabled并行remark

-XX:+CMSScavengeBeforeRemark remark前先做一次minor gc

**G1参数调优**

-XX:+UseG1GC开启G1参数

-XX:MaxGCPauseMillis=n GC最大停顿时间，软性参数

-XX:G1HeapRegionSize=n每个region的大小

-2HMBR-日NXFVIRmP4

#### 调优Best pratise

多分析线上case，并设置不同的内存大小观察gc日志，寻找最佳策略
通过改善参数避免common类型问题

#### 日志优化

- 同步日志/异步日志
- 日志归档时间 （归档加锁）
- 日志大小拆分

#### 池化策略

idle数量， cpu核数x2（IO） 计算（cpu+1） 连接池连接数量



#### 数据库读写性能

- 查询优化
- 批量写
- 索引优化
- innodb相关优化

查询优化

- 主键查询千万条记录1-10ms
- 唯一索引千万条记录10-100ms
- 非唯一索引千万条记录100-1000ms
- 无索引百万条记录1000ms+

批量写

- for each {insert into table values (1) }
- Execute once insert into table values (1),(2),(3),(4)....;
- Sql编译N次和1次的时间与空间复杂度
- 网络消耗的时间复杂度
- 磁盘寻址的复杂度

单机配置优化

- max_connection=1000增加最大链接数,默认为100
- innodb_file_per_table=1可以存储每个innodb表和他的索引在自己的文件中（不同表的索引、数据、日志放在一起，会增加磁盘寻址时间）
- innodb_buffer_pool_size=1G缓存池大小，设置为当前数据库服务内存的60%-80%
- inodb_log_file_size=256M一般取256M可以兼顾性能和recovery的速度，写满后只能切换日志靠buffer存储
- innodb_log_buffer_size=16M该参数确保有足够大的日志缓冲区来保存脏数据在被写入到日志文件之前可以继续mysql事务操作
- innodb_flush_log_at_trx_commit = 2

1时，在每个事务提交时，日志缓冲被写到日志文件，对日志文件做到磁盘操作的刷新。Truly ACID。速度慢。

2时，在每个事务提交时，日志缓冲被写到系统缓冲，但不对日志文件做到磁盘操作的刷新。然后根据
innodb_fush_log_at_timeout(默认为1s)时间flush disk 只有操作系统崩溃或掉电才会删除最后一秒的事务，不然不会丢失事务。

0时，效率更高，但安全性差。每秒才write日志任何mysqld进程的崩溃会删除崩溃前最后一秒的事务

- innodb_data_file_path=ibdata1:1G;ibdata2:1G;ibdata3:1G:autoexte
  nd指定表数据和索引存储的空间，可以是一个或者多个文件。

读写分离

- 一主多从
- 读库延迟问题处理
  - 前端等待一定时间（应用层让步 loading）
  - ms级别也接受不了，转读master
- 主从切换处理
  - 半同步，至少有一个slave回应，master再进行事务commit
  - 

![image-20210712175911889](https://gitee.com/koala010/typora/raw/master/img/20210712175912.png)

![image-20210712180046487](https://gitee.com/koala010/typora/raw/master/img/20210712180046.png)

分库分表

- 垂直拆分
  - 相关性强的放一张表，join
- 水平拆分
  -  异步消息队列
  - 不实时数据，借助搜索引擎ES
- 多主多从



#### 缓存问题

穿透，击穿，雪崩

穿透:数据不存在

击穿:同一数据击穿到数据库

雪崩:不同的数据击穿到数据库

### 网络瓶颈

网络瓶颈的根源

- 公网:带宽，出口调用量
- 内网:带宽，出口调用量

中间件也会有内网网络瓶颈

解决：

- 分散（分布式）
- 压缩（数据压缩）

### 解决问题

通过看源代码来解决百度不到的问题

你有解决过哪些有挑战的问题，如何解决的？

- 查问题
- 搜“百度“
- 自行源代码解决

 

### 源代码

读源码的前提

- 先会用框架

- 官方文档要吃透

如何看源代码？

- 先看框架架构
- 再看启动流程
- 最后在看执行方法

### SpringBoot

#### 启动流程

- 创建SpringApplication
- 初始化构造器Initializer和listener调用run方法
- 开启定时器StopWatch
- 根据SpringApplicationRunListeners以及参数来启动环境
- 准备环境配置，注意ConfigFileApplicationListener是用来加载properorties文日
- 打印banner
- 创建容器上下文
- prepare准备容器上下文，执行之前的Initializer初始化，并loadbean
- refresh容器上下文，里面onRresh可以启动web server
- afterRefresh执行初始化完成后要做的操作
- listener启动完成
- 关闭计时器
- 打印启动时间

### Dubbo

#### 原理

![image-20210713083440268](https://gitee.com/koala010/typora/raw/master/img/20210713083447.png)

#### 线程模型

- IO线程池
- 业务工作线程池
- Boss线程



all:—律进工作线程池. 

direct:一律lO线程池

message:除了请求和响应走工作线程池，其他都是IO线程池. 

execution:除了请求，其他都走lO线程池

connection:连接和断连进队列，其他都走工作线程池



fixed:固定大小线程池

cached:缓存线程池,一分钟后释放

limited:可伸缩线程池，只会增长不会收缩

eager: min到max，超过后进队列，队列满后拒绝

![image-20210713084632050](https://gitee.com/koala010/typora/raw/master/img/20210713084632.png)

#### Dubbo和Spring cloud区别

RPC及服务治理

Dubbo使用RPC二进制协议，Spring Cloud使用Http Restful

#### 配置

Application应用纬度

Service服务纬度

Method方法纬度

Provider提供者纬度

Consumer消费者纬度

#### 常见的调用属性有哪些

Timeout超时消费者>提供者。

Retries 失败重试

LoadBalance负载均衡策略，轮询，随机，最少活跃调用，一致性hash

#### 服务发布有依赖怎么办

默认check="true"，可以通过check="false”关闭检查

#### 序列化方式

hessian，fastjson，jdk，protobuf，protostuf

#### 服务降级

 ![image-20210713093238037](https://gitee.com/koala010/typora/raw/master/img/20210713093238.png)



#### 服务暴露

聊一下服务暴露,发现以及调用的过程吧

- 每个ServiceBean作为一个ApplicationListener会在contextRefresh时做服务export，将自身的服务提供内容发送到注册中心上，并且在本地也暴露服务
- 每个ReferenceBean作为一个ApplicationContextAware会在contextRefresh时做bean的afterPropertiesSet，将从注册中心拉取远端服务，并且在本地生成服务存根
- 调用时通过代理的方式转移到ReferenceBean上，做对应的服务调用

#### 高效的限流,灵活的熔断

你是如何解决微服务的异常问题的？

- 限流
- 熔断

##### 限流的维度

- 接口限流
- 总限流

##### 限流的单位

- 限并发
- 限QPS/TPS

##### 限流的分类

- 单机限流
- 集群限流

##### 单机限流

![image-20210713101556185](https://gitee.com/koala010/typora/raw/master/img/20210713101556.png)

![image-20210713103047895](https://gitee.com/koala010/typora/raw/master/img/20210713103048.png)



##### 集群限流

负载均衡

更好的集群限流（redis）

![image-20210713105331105](https://gitee.com/koala010/typora/raw/master/img/20210713105331.png)

##### 熔断的本质

- 失败率触发
- 失败总次数触发

#### 360度微服务监控

##### 监控常用维度

- 接口
- DB
- Redis
- ……
- 硬件( CPU,IO Wait,Memory,网卡)

##### 监控指标

- 接口TPS/QPS,AVG耗时，95线,99线, 99.99线
- CPU load average
- Memory 占用
- Io Wait
- 网卡占用

##### 监控工具

- 大众点评开源CAT
- Zabbix

##### 只有这些就够了吗

客户端监控

## 三面

### 自我介绍的额外价值挖掘

#### 三面的人是谁?

- 到达了三面已经基本通过了基础关卡
- 部门负责人?
- 三面的人往往注重你的潜力以及未来的可能

#### 如何自我介绍

- 技术栈，项目介绍一笔带过
- 着重说你的积累，例如:开源项目贡献，公众号github等，主要凸显你的与众不同，并且让面试官有兴趣挖掘你的额外价值

### 保证系统安全的行为

#### 背景

- 大厂都有安全部
- 上线前的折磨和上线后的痛苦

#### SQL注入的原理是什么，如何预防

例如:

SELECT *FROM user WHERE username= ";DROP DATABASE (DB Name)

预防:

1. 做输入校验

2. 使用预编译sql语句执行

3. mybatis中`#`和`$`的区别

`#`预编译，可以防止sql注入

#### XSS跨站点脚本攻击

反射型

存储型

![image-20210713150454759](https://gitee.com/koala010/typora/raw/master/img/20210713150524.png)



预防:

1. 做输入校验，替换

2. 设置cookie为http-only访问方式
   1. 没有办法通过script获取cookie，只能通过原始http请求

#### CSRF跨站点请求攻击

触发跨站请求

![image-20210713151205297](https://gitee.com/koala010/typora/raw/master/img/20210713151205.png)

浏览器同源，请求攻击

预防:

1. cookie hash  校验位

2. web token

### 容器化部署

#### Docker的核心技术是什么

- **NameSpace命名空间**，隔离进程，用户、网络、IPC以及UTS等的基础。
- CGroups 控制组限制硬件资源
- UnionFS做了镜像管理

#### 容器docker 与虚拟机的区别

- 进程与系统的区别



### 云原生

#### 云原生是什么

云原生=微服务+ DevOps+持续交付＋容器化

#### 云原生的优势

- 自动化
- 模糊开发，测试，运维的边界
- 成本，弹性

### K8S

![image-20210713153041479](https://gitee.com/koala010/typora/raw/master/img/20210713153041.png)

### 算法

![image-20210713154430848](https://gitee.com/koala010/typora/raw/master/img/20210713154431.png)

![image-20210713154606433](https://gitee.com/koala010/typora/raw/master/img/20210713154606.png)

## 四面

程序员的职业素养:靠谱

### 素养

- 解决问题能力
- 团队协作能力
- 自我驱动能力

### 解决问题能力

#### 何为好的解决问题能力

- 快速定位
- 深入分析
- 取舍解决

##### 快速定位

- 依据报警快速粗筛:接口耗时，数据库耗时，网络耗时,JVM
  OOM?
- 快速止血:熔断降级,限流，重启?
- 验证尝试:是否恢复

集联问题最难解决:系统边界划分，容错设计

##### 深入分析

- 具体原因是什么:数据量过大，慢sql，发布bug，被刷了?
- 触发条件:线上运行的好好的突然出问题一定是有触发条件
- 总结原因:分析定位总结原因

##### 取舍解决

用什么方案解决什么问题，还会不会引入别的问题，后续如何改进

#### 举例

问题:线上系统突然崩溃,用户间歇性无法登录交易

**快速定位**

排查无法交易的接口监控报错或者token异常

错误日志和监控显示redis client无法获取链接

redis cluster server单机故障，CPU打满，关注到有大量key写入排查写入接口发现登录请求巨大

重启redis cluster server，客户端重连

故障间歇性恢复

**深入分析**

被恶意登录刷token，刷交易量

异常流量处理,网络层封杀对应客户端IP故障完全恢复

**取舍解决**

系统需要具备可以防刷的能力

token下发能力限制，对IP封杀(可能会引入误伤)，验证码机制
(影响正常用户体验)

### 团队协作能力

#### 何为好的团队协作能力？

- 协作模式
- 冲突解决
- 主导项目

#### 协作模式

- 润滑剂模式:正面向上，互帮互助。
- 权责模式:明确边界，大公无私型模式
- 混合模式:明确边界，正面向上

#### 冲突解决

遇到冲突积极解决,明确职责范畴，互相体谅，共同改进进步

#### 主导项目

- 明确目标
- 职责分工
- 过程管理
- 结果导向
- 复盘改进

### 自我驱动能力

#### 何为好的自我驱动能力

- 主动学习
- 积极承担
- 自我迭代

#### 主动学习

最近主动学了什么，有什么收获，分享一下?

#### 积极承担

模糊边界地带积极承担责任，拿到结果

#### 自我迭代

复盘了吗

自己工作结束后，会不会进行复盘

## HR面（值得重复看几次）

### 注意点

- 技术方面一笔带过
- 讲经历一定要讲业绩，荣誉和认可
- 简单讲下规划

### 讲讲你的项目吧

面对不太了解技术的hr，如何描述你的项目

注意点

**HR不在乎项目本身,在乎的是**

- 你给你的团队提供了什么价值
- 如何应对问题，处理问题。
- 你最大的收获是什么

### 你的优缺点是什么

认知对齐?规划理解

注意点

- 优点放大
- 缺点改善

### 目前你有其他offer吗

为什么这么问？

- 认知确认
- 风险控制

注意点

- 大厂实事求是，小厂忽略简介
- 需要连带讲明选择你们的未来的职业规划
- 诚恳谦逊

### 你有什么问题吗

借助这个问题加分

- 聊公司前景
- 聊业务发展
- 聊职业规划

