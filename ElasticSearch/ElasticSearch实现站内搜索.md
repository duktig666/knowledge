## ES实现站内搜索

### ES实现站内搜索

#### 流程图

![image-20210203092957723](https://gitee.com/koala010/typora/raw/master/img/ElasticSearch站内搜索流程图.png)



#### 站内搜索实现分析

以小米手机搜索为例，来进行站内搜索的分析：

①在前端搜索框输入搜索词，利用ES的ik分词器进行分词，到ES进行快速检索；

②ES中存储关键信息，一般为一条记录的唯一标识信息和可进行分词检索的信息（例如小米手机可以根据手机名和价格进行检索，那么将id、手机名还有价格的手机信息存储到ES中）

③检索到的信息返回给前端

④前端点击手机信息，例如小米11，那么用ES检索出的手机ID去数据库查询小米11的详细信息。

⑤可以利用搜索词的搜索频度存储到redis，用来实现热搜排行榜等功能（具体的业务引入具体的技术）

⑥利用远程词典可以控制ik分词器不能分词的网络热词

**⑦关键点在于数据库与ES的数据同步方案。**



### SpringBoot整合SpringData ElasticSearch

ElasticSearch的更新迭代比较快，目前已经迭代到了7.x版本。使用SpringBoot整合SpringData ElasticSearch对版本的要求比较高。这里我们使用SpringBoot 2.3.x（比如`2.3.8.RELEASE`）,对应可以使用ElasticSearch7.x。



#### 1. 引入依赖：

```java
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.8.RELEASE</version>
    </parent>
        
    <dependencies>
        <!-- elasticsearch -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <!-- 其余依赖视情况引入 -->
    </dependencies>    
```

#### 2.配置application.yml

```yml
spring:
# elasticsearch
  elasticsearch:
    rest:
      uris: 127.0.0.1:9200
```

#### 3.编写实体类

可以将数据库的实体类抽离出来，依据上述站内搜索的分析，将一条记录的唯一标识和可分词检索的字段存储到ES中。ES实体的属性对应es存储的字段。

```java
@Document(indexName = "phone")
@Data
@Accessors(chain = true)
public class MenuES {

    @Id
    private Long id;

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String name;

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String title;

}
```

#### 4.编写Repository类

SpringData Elasticsearch的Repository和SpringData JPA使用上极其类似。

```java
public interface MenuRepository extends ElasticsearchRepository<MenuES, Long> {
}
```

泛型第一个类型对应ES实体类，第二个对应主键的类型。

#### 5.测试

```java
@SpringBootTest(classes = AppRun.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class TestSpringDataElasticSearch {

    @Autowired
    private MenuRepository menuRepository;

    /**
     * 添加索引和更新索引 id 存在更新 不存在添加
     */
    @Test
    public void testSaveOrUpdate() {
        MenuES menu = new MenuES();
        menu.setId(1L)
                .setName("系统管理")
                .setName("系统管理");
        menuRepository.save(menu);
    }

    /**
     * 查询所有
     */
    @Test
    public void testFindAll() {
        Iterable<MenuES> menuIterable = menuRepository.findAll();
        for (MenuES menu : menuIterable) {
            System.out.println(menu);
        }
    }


    /**
     * 查询一个
     */
    @Test
    public void testFindOne() {
        Optional<MenuES> byId = menuRepository.findById(1L);
        System.out.println(byId.get());
    }

}
```

这里只是举出了一个简单的例子，具体SpringBoot整合ElasticSearch实现站内搜索功能要看具体的业务逻辑。



### MySQL与ES数据同步方案

数据同步效果主要分为两种**实时同步**和**最终同步**。

实时同步对同步频率要求较高。需要保证mysql数据发生增删改变更后，需要立马将变更后的数据同步给ES；保证ES检索到的数据近乎实时都是准确的。

最终同步对同步频率要求不高，但需要保证最终数据库与ES的数据是一致的。至于“最终”这个时间段时多少，根据不同的业务场景，确定不同的时间。



#### 方案一：业务逻辑实现

##### 实现方式

使用SpringData ElasticSearch可以快速进行ES相关的操作，在进行mysql增删改操作的同时，对ES所对应的索引库进行增删改操作，以保持数据的一致性。

从实现角度考虑，这样的方式属于数据实时同步。但是可能在数据同步的时候会出现一些问题：

1. 只有当mysql执行增删改成功之后，才可以进行ES操作的执行，在代码实现的角度考虑，会使业务变得更加复杂。这个问题在这个实现方式下，基本不可避免。
2. 如果mysql执行成功，但是ES执行失败，怎么处理？如果不处理会引起数据不一致的问题。

##### 问题及解决

首先，这样的问题没有必要从事务的角度考虑，即mysql执行成功，ES执行失败，mysql执行回退操作，没有必要让mysql重新执行。

所以，可以考虑的点还是从ES数据补偿的角度出发，可以将ES同步失败的数据缓存到redis或者是消息中间件（例如RabbitMQ），等待服务器不忙碌的时候（例如凌晨2点），将redis/消息中间件中的数据同步到ES中，采用订阅消费模式，直至未同步的数据同步到ES，确保数据一致性。

这样的解决方案，在一定的程度上降低了数据同步的实时性，但是也确保了数据的最终一致性。

##### 分析

###### 优点：

1. 逻辑简单，代码实现容易。
2. 保证了数据的最终一致性，在一定程度上保证了数据同步的实时性（如果ES没有问题，即可保证数据的实时同步，但这是不现实的）。

###### 缺点：

1. ES的数据同步与业务的耦合性太大，不方便维护和扩展。
2. 数据库数据变更一次，就要同步一次ES数据，数据同步过于频繁。
3. 从数据同步的实时性出发，但是并不能保证数据同步的实时性（虽然保证了数据同步的最终一致性）。



#### 方案二：消息中间件/redis+定时任务实现

为了解决数据同步频繁的问题，并主要实现数据最终同步的目的，故出现了这个方案。

##### 实现方式

实现方式与上文的业务逻辑方式类似，只是在mysql数据发生增删改变更的时候，不直接将变更的数据同步到ES，而是先缓存到redis或者消息中间件，等到流量低（例如凌晨2点）的时间利用定时任务将数据同步给ES，采用订阅/消费模式直到数据同步成功为止。

##### 分析

###### 优点

1. 实现了数据最终同步
2. 降低了服务器压力，不再频繁地进行数据同步

###### 缺点

1. 即使借助消息中间件和redis，数据同步也与业务耦合比较大
2. 数据同步内嵌在代码中



#### 方案三：借助插件实现数据同步

MySQL与ElasticSearch数据同步的插件有很多，在这里列出来常用的一些，例如 ：

- [elasticsearch-jdbc](https://github.com/jprante/elasticsearch-jdbc)
- [elasticsearch-river-mysql](https://github.com/scharron/elasticsearch-river-mysql)
- [go-mysql-elasticsearch](https://github.com/siddontang/go-mysql-elasticsearch)
- [logstash-input-jdbc](https://github.com/logstash-plugins/logstash-input-jdbc)

##### 插件优缺点对比 

| 插件名称                  | 优点                                                         | 缺点                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| elasticsearch-jdbc        | 1.相对通用<br/>2.GitHub活跃度高，版本更新快                  | 不能实现同步删除操作mysql删除了，ES仍然存在                  |
| elasticsearch-river-mysql | 版本太旧，不再考虑                                           | 2012年12月13日后便不再更新                                   |
| go-mysql-elasticsearch    | 1.国内作者开发<br/>2.能实现同步增删改查操作                  | 1.仍处于开发阶段，相对不稳定<br/>2.没有日志，不方便排查问题和查看同步结果 |
| logstash-input-jdbc       | 1.能实现同步增、改操作的增量、全量数据同步<br/>2.版本更新快、相对稳定<br/>3.集成在LogStash，作为ES生态固有插件的一部分，易用 | 不能实现同步删除操作mysql删除了，ES仍然存在                  |

*严格意义上elasticsearch-jdbc已经不是第三方插件，已经成为独立的第三方工具。*

##### 分析

插件比较多，经过对比分析选择`logstash-input-jdbc`较为合适，版本更新快、稳定，而且作为LogStash的一款插件，而且现在LogStash的版本已经默认集成了该插件，后来要使用日志可视化（ELK）也会用到LogStash。虽然有不能实现同步删除的操作，但是如果数据库字段采用的逻辑删除，则可以完美避免这个问题，逻辑删除刚好符合我们的场景。



#### 方案四：canal(alibaba)

[canal](https://github.com/alibaba/canal)——mysql数据库binlog的增量订阅&消费组件。

##### 分析

推荐使用场景 canal适用于对于Mysql和Elasticsearch数据实时增、删、改要求高的业务场景。 实时场景要求不高的业务场景，logstash-input-jdbc也能满足。

建议，做好选型甄别。

###### 优点

1. 基于binlog实现，支持增删改同步实现
2. 阿里开源，网上有很多具体实现方案
3. SpringBoot引入canal客户端，canal可监听数据变化，做相应的业务处理

###### 缺点

1. 不支持全量同步



### logstash-input-jdbc实现mysql与ES的数据同步

LogStash在5.x版本已经默认集成logstash-input-jdbc插件，新版本的LogStash不用再单独下载插件，直接使用LogStash就可以完成mysql与ES的数据同步。

下载完成LogStash，编写`jdbc.conf`文件（可以自定义命名）。`jdbc.conf`我放在了LogStash的`config`的目录下（可以更换）。

LogStash官网下载速度比较慢，可以考虑从国内镜像下载： [https://www.newbe.pro/Mirrors/Mirrors-Logstash/](https://www.newbe.pro/Mirrors/Mirrors-Logstash/)

#### 全量同步

```shell
input{
  jdbc{
    # mysql连接地址
    jdbc_connection_string => "jdbc:mysql://localhost:3306/smpe?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true"
    # mysql账号
    jdbc_user => "root"
    # mysql密码
    jdbc_password => "159357asd"
    # Java连接mysql的驱动jar包（最好与数据库的版本对应）
    jdbc_driver_library => "D:\javaweb\Elasticsearch\logstash-7.10.0\mysql\mysql-connector-java-8.0.12.jar"
    # mysql连接驱动名
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    # 是否支持分页
    jdbc_paging_enabled => "true"
    # 每页显示的条数
    jdbc_page_size => "100"
	# 设置时区
	jdbc_default_timezone => "Asia/Shanghai"
	# 具体的sql，将哪些数据同步到es
    statement => "select id,name,title from sys_menu"
    # 可以将statement换成statement_filepath将sql语句写到额外的文件里
    # statement_filepath => "D:\javaweb\Elasticsearch\logstash-7.10.0\config\jdbc.sql"
    # 同步的频率/时间 第一位是分钟 不设置就是1分钟执行一次
    schedule => "* * * * *"
  }
}
output{
  elasticsearch {
    # es的ip和端口
    hosts => "127.0.0.1:9200"
    # 同步的索引
    index => "menu"
    # id
    document_id => "%{id}"
  }
  stdout{
    codec => json_lines
  }
}
```



#### 增量同步

增量同步主要改动以下项

```shell
# 如果使用更新时间来追踪增量，那么值设置为false；如果为其他字段追踪，如id，值设置为true
use_column_value => true
# 追踪增量的字段
tracking_column => id
# 设置记录上一次的增量的值
record_last_run => true
# 将上一次增量的值存储的文件
last_run_metadata_path => "D:\javaweb\Elasticsearch\logstash-7.10.0\config\.logstash_jdbc_last_run"
# 增量同步，sql后跟where子句，即"where 追踪字段 > :sql_last_value"
statement => "select id,name,title from sys_menu where id > :sql_last_value"
```



增量同步完全文件`jdbc.conf`

```shell
input{
  jdbc{
    # mysql连接地址
    jdbc_connection_string => "jdbc:mysql://localhost:3306/smpe?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true"
    # mysql账号
    jdbc_user => "root"
    # mysql密码
    jdbc_password => "159357asd"
    # Java连接mysql的驱动jar包（最好与数据库的版本对应）
    jdbc_driver_library => "D:\javaweb\Elasticsearch\logstash-7.10.0\mysql\mysql-connector-java-8.0.12.jar"
    # mysql连接驱动名
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    # 是否支持分页
    jdbc_paging_enabled => "true"
    # 每页显示的条数
    jdbc_page_size => "100"
	# 设置时区
	jdbc_default_timezone => "Asia/Shanghai"
	# 如果使用更新时间来追踪增量，那么值设置为false；如果为其他字段追踪，如id，值设置为true
	use_column_value => true
	# 追踪增量的字段
	tracking_column => id
	# 设置记录上一次的增量的值
	record_last_run => true
	# 将上一次增量的值存储的文件
	last_run_metadata_path => "D:\javaweb\Elasticsearch\logstash-7.10.0\config\.logstash_jdbc_last_run"
	# 增量同步，sql后跟where子句，即"where 追踪字段 > :sql_last_value"
	statement => "select id,name,title from sys_menu where id > :sql_last_value"
	# 同步的频率/时间 第一位是分钟 不设置就是1分钟执行一次
    schedule => "* * * * *"
  }
}
output{
  elasticsearch {
    # es的ip和端口
    hosts => "127.0.0.1:9200"
    # 同步的索引
    index => "menu"
    # id
    document_id => "%{id}"
  }
  stdout{
    codec => json_lines
  }
}
```



配置完成`jdbc.conf`后，进入`bin`目录使用命令启动LogStash（启动前要先启动ES）

```shell
 logstash -f ../config/jdbc.conf
```



### canal实现mysql与ES的数据同步

下载`canal.deployer`

修改`conf/example/instance.properties`文件，主要修改以下几处：

- `canal.instance.master.address`：数据库地址，例如127.0.0.1:3306
- `canal.instance.dbUsername`：数据库用户
- `canal.instance.dbPassword`：数据库密码

完整`conf/example/instance.properties`文件：

```properties
#################################################
## mysql serverId , v1.0.26+ will autoGen
# canal.instance.mysql.slaveId=0

# enable gtid use true/false
canal.instance.gtidon=false

# position info
canal.instance.master.address=127.0.0.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
canal.instance.dbUsername=root
canal.instance.dbPassword=159357asd
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
#canal.instance.filter.black.regex=mysql\\.slave_.*
canal.instance.filter.black.regex=
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch

# mq config
canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#canal.mq.dynamicTopicPartitionNum=test.*:4,mycanal:6
#################################################

```



下载`canal.adapter`

修改`conf/application.yml`文件，主要修改

- server.port:canal-adapter端口号
- canal.conf.canalServerHost:canal-server地址和ip
- canal.conf.srcDataSources.defaultDS.url:数据库地址
- canal.conf.srcDataSources.defaultDS.username:数据库用户名
- canal.conf.srcDataSources.defaultDS.password:数据库密码
- canal.conf.canalAdapters.groups.outerAdapters.hosts:es主机地址,tcp端口

完整文件如下：

```yml
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp #tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:
    # kafka consumer
    kafka.bootstrap.servers: 127.0.0.1:9092
    kafka.enable.auto.commit: false
    kafka.auto.commit.interval.ms: 1000
    kafka.auto.offset.reset: latest
    kafka.request.timeout.ms: 40000
    kafka.session.timeout.ms: 30000
    kafka.isolation.level: read_committed
    kafka.max.poll.records: 1000
    # rocketMQ consumer
    rocketmq.namespace:
    rocketmq.namesrv.addr: 127.0.0.1:9876
    rocketmq.batch.size: 1000
    rocketmq.enable.message.trace: false
    rocketmq.customized.trace.topic:
    rocketmq.access.channel:
    rocketmq.subscribe.filter:
    # rabbitMQ consumer
    rabbitmq.host:
    rabbitmq.virtual.host:
    rabbitmq.username:
    rabbitmq.password:
    rabbitmq.resource.ownerId:

  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.0.0.1:3306/smpe?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true
      username: root
      password: 159357asd
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
#      - name: rdb
#        key: mysql1
#        properties:
#          jdbc.driverClassName: com.mysql.jdbc.Driver
#          jdbc.url: jdbc:mysql://127.0.0.1:3306/mytest2?useUnicode=true
#          jdbc.username: root
#          jdbc.password: 121212
#      - name: rdb
#        key: oracle1
#        properties:
#          jdbc.driverClassName: oracle.jdbc.OracleDriver
#          jdbc.url: jdbc:oracle:thin:@localhost:49161:XE
#          jdbc.username: mytest
#          jdbc.password: m121212
#      - name: rdb
#        key: postgres1
#        properties:
#          jdbc.driverClassName: org.postgresql.Driver
#          jdbc.url: jdbc:postgresql://localhost:5432/postgres
#          jdbc.username: postgres
#          jdbc.password: 121212
#          threads: 1
#          commitSize: 3000
#      - name: hbase
#        properties:
#          hbase.zookeeper.quorum: 127.0.0.1
#          hbase.zookeeper.property.clientPort: 2181
#          zookeeper.znode.parent: /hbase
      - name: es7
        key: example
        hosts: 127.0.0.1:9200 # 127.0.0.1:9200 for rest mode
        properties:
          mode: rest #transport or rest
#          # security.auth: test:123456 #  only used for rest mode
          cluster.name: elasticsearch
#        - name: kudu
#          key: kudu
#          properties:
#            kudu.master.address: 127.0.0.1 # ',' split multi address
```

另外需要配置`conf/es/*.yml`文件，adapter将会自动加载conf / es下的所有.yml结尾的配置文件。以`test.yml`为例：

```yml
dataSourceKey: defaultDS
destination: example
groupId:
esMapping:
  _index: menu
  _type: _doc
  _id: _id
#  upsert: true
  sql: "select id as _id,name,title from sys_menu "
#  etlCondition: "where update_time>='{0}'" #etl的条件参数，可以将之前没能同步的数据同步，数据量大的话可以用logstash
  commitBatch: 3000
```



然后分别在`canal.deployer`和`canal.adapter`的bin目录启动deployer和adapter



测试增删改mysql数据，也可以同步数据到ES。



###  对比LogStash和Canal同步mysql和ES的数据

|               | LogStash                 | Canal                                                     |
| ------------- | ------------------------ | --------------------------------------------------------- |
| 全量同步      | 支持                     | 不支持                                                    |
| 增量同步      | 支持                     | 支持                                                      |
| mysql删除同步 | 不支持（可采用逻辑删除） | 支持                                                      |
| ES7.x兼容性   | 好                       | 目前不友好（mysql增删改可以监听到，但是数据不能同步到ES） |
| 实时性        | 较低                     | 较高                                                      |

如果选用ES6.x，那么这两种方案其实都可行。

如果选用ES7.X，那么目前LogStash会合适一些，Canal对ES7.X的兼容性还并不好，如果canalv1.1.5版本发布稳定版了可以再考虑使用。



