## 安装

### win10安装Kafka

参看：[Win10下kafka简单安装及使用](https://blog.csdn.net/github_38482082/article/details/82112641)

**启动命令（进入kafka的安装目录）：**

启动内置zookeeper

```sh
.\bin\windows\zookeeper-server-start.bat  .\config\zookeeper.properties
```

启动kfka

```sh
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

**测试**

创建主题：

```sh
 .\bin\windows\kafka-topics.bat --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test1
```

查看主题：

```sh
.\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```



## SpringBoot 整合 Kafka

### 简单的例子

**引入关键依赖**

```xml
<!--引入kafka依赖-->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.8.2</version>
</dependency>
```

**application.yml 配置**

```yaml
server:
  port: 9001

spring:
  application:
    name: kafka-example
  kafka:
    bootstrap-servers: 127.0.0.1:9092 # kafka集群信息
    producer: # 生产者配置
      retries: 3 # 设置大于0的值，则客户端会将发送失败的记录重新发送
      batch-size: 16384 #16K
      buffer-memory: 33554432 #32M
      acks: 1
      # 指定消息key和消息体的编解码方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: kafkaTestGroup # 消费者组
      enable-auto-commit: false # 关闭自动提交
      auto-offset-reset: earliest # 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
      # 当每一条记录被消费者监听器（ListenerConsumer）处理之后提交
      # RECORD
      # 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后提交
      # BATCH
      # 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，距离上次提交时间大于TIME时提交
      # TIME
      # 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，被处理record数量大于等于COUNT时提交
      # COUNT
      # TIME |　COUNT　有一个条件满足时提交
      # COUNT_TIME
      # 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后, 手动调用Acknowledgment.acknowledge()后提交
      # MANUAL
      # 手动调用Acknowledgment.acknowledge()后立即提交，一般使用这种
      # MANUAL_IMMEDIATE
      ack-mode: manual_immediate
```

**生产者**

```java
@RestController
public class KafkaProducer {

    /** topic的名称 */
    private final static String TOPIC_NAME = "kafkaTest";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @RequestMapping("/send")
    public void send() {
        //发送功能就一行代码~
        kafkaTemplate.send(TOPIC_NAME, "key", "test message send. TIME：" + LocalDateTime.now());
    }

}
```

**消费者**

```java
@Component
public class KafkaConsumer {

    /**
     * kafka的监听器，topic为"zhTest"，消费者组为"zhTestGroup"
     */
    @KafkaListener(topics = "kafkaTest", groupId = "kafkaTestGroup")
    public void listenKafkaTestGroup(ConsumerRecord<String, String> record, Acknowledgment ack) {
        String value = record.value();
        System.out.println(value);
        System.out.println(record);
        //手动提交offset
        ack.acknowledge();
    }

    /*//配置多个消费组
    @KafkaListener(topics = "zhTest",groupId = "zhTestGroup2")
    public void listenTulingGroup(ConsumerRecord<String, String> record, Acknowledgment ack) {
        String value = record.value();
        System.out.println(value);
        System.out.println(record);
        ack.acknowledge();
    }*/

}
```

**测试**

![image-20220222215803067](https://cos.duktig.cn/typora/202202222158645.png)



参看：

- [SpringBoot集成kafka全面实战](https://blog.csdn.net/yuanlong122716/article/details/105160545/)
- [kafka学习（五）Spring Boot 整合 kafka ](https://www.cnblogs.com/riches/p/11720068.html)

