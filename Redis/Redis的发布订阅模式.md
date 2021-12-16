

本文源码参看：[https://github.com/duktig666/learn-example/tree/5586febea31c2fb368e19fbdba11ed08afd463e0/Redis/src/main/java/cn/duktig/pubsub](https://github.com/duktig666/learn-example/tree/5586febea31c2fb368e19fbdba11ed08afd463e0/Redis/src/main/java/cn/duktig/pubsub)

# Redis的发布订阅模式

## Redis发布订阅概述

Redis 发布订阅 (publish/subscribe) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 客户端可以订阅任意数量的频道。

介绍：

- PUBLISH 命令向通道发送信息，此客户端称为publisher 发布者；
- SUBSCRIBE 向命令通道订阅信息，此客户端称为subscriber 订阅者；
- redis 中 发布订阅模块的名字叫着 PubSub，也就是 PublisherSubscriber；
- 一个发布者向一个通道发送消息，订阅者可以向多个通道订阅消息；当发布者向通道发布消息后，如果有订阅者订阅该通道，订阅者就会收到消息。

**发布订阅相关的命令**：

| 命令                                                         | 描述                               |
| :----------------------------------------------------------- | :--------------------------------- |
| [Redis Unsubscribe 命令](https://www.redis.net.cn/order/3637.html) | 指退订给定的频道。                 |
| [Redis Subscribe 命令](https://www.redis.net.cn/order/3636.html) | 订阅给定的一个或多个频道的信息。   |
| [Redis Pubsub 命令](https://www.redis.net.cn/order/3633.html) | 查看订阅与发布系统状态。           |
| [Redis Punsubscribe 命令](https://www.redis.net.cn/order/3635.html) | 退订所有给定模式的频道。           |
| [Redis Publish 命令](https://www.redis.net.cn/order/3634.html) | 将信息发送到指定的频道。           |
| [Redis Psubscribe 命令](https://www.redis.net.cn/order/3632.html) | 订阅一个或多个符合给定模式的频道。 |

## 发布订阅演示

subscribe/publish

![订阅](https://cos.duktig.cn/typora/202111102124676.png)

![发布](https://cos.duktig.cn/typora/202111102124186.png)

psubscribe/publish

![按模式订阅](https://cos.duktig.cn/typora/202111102124604.png)

![按模式发布](https://cos.duktig.cn/typora/202111102124218.png)

## Redis发布订阅模式 与 消息中间件 进行对比

### 可靠性

Redis虽然可以实现发布订阅，其功能与常见的消息中间件类似（例如RabbitMQ），但是 **Redis的发布订阅模式不支持持久化**，而且发布者发布一条消息，没有对应的消费者时，消息会丢失。

而RabbitMQ具有消息消费的确认机制，发布者发布一条消息，一直在队列中，直到消息被消费。

### 实时性

Redis作为高效的缓存服务器，基于内存，发布的消息不需要持久化，具备更高的实时性。

### 消费者的负载均衡

rabbitmq队列可以被多个消费者同时监控消费，但是每一条消息只能被消费一次，由于rabbitmq的消费确认机制，因此它能够根据消费者的消费能力而调整它的负载；

**redis发布订阅模式**，一个队列可以被多个消费者同时订阅，当有消息到达时，会将该消息依次发送给每个订阅者；

### 持久性

**redis**：redis的持久化是针对于整个redis缓存的内容，它有RDB和AOF两种持久化方式（redis持久化方式，后续更新），可以将整个redis实例持久化到磁盘，以此来做数据备份，防止异常情况下导致数据丢失。

**rabbitmq**：队列，消息都可以选择性持久化，持久化粒度更小，更灵活；

### 队列监控

**rabbitmq实现了后台监控平台**，可以在该平台上看到所有创建的队列的详细情况，良好的后台管理平台可以方面我们更好的使用；**redis没有所谓的监控平台**。

### 总结

**redis**： 轻量级，低延迟，高并发，低可靠性；

**rabbitmq**：重量级，高可靠，异步，不保证实时；

rabbitmq是一个专门的AMQP协议队列，他的优势就在于提供可靠的队列服务，并且可做到异步，而redis主要是用于缓存的，redis的发布订阅模块，可用于实现及时性，且可靠性低的功能。

## SpringBoot整合Redis实现发布订阅模式

*SpringBoot整合Redis的默认配置略，详情参看本文相关源码。*

1、定义订阅者接受消息的接口

目的：使接受方法通用，方便后边配置适配器

```java
@Component
public interface RedisMsg {

    /**
     * Redis订阅者接受消息的接口
     *
     * @param message 订阅的消息
     */
    void receiveMessage(String message);

}
```

2、定义两个订阅者

```java
public class RedisChannelSub implements RedisMsg {

    @Override
    public void receiveMessage(String message) {
        //注意通道调用的方法名要和 RedisPubSubConfig 的listenerAdapter的 MessageListenerAdapter 参数2相同
        System.out.println("这是RedisChannelSub" + "-----" + message);
    }

}
```



```java
public class RedisPmpSub implements RedisMsg {

    /**
     * 接收消息的方法
     *
     * @param message 订阅消息
     */
    @Override
    public void receiveMessage(String message) {
        //注意通道调用的方法名要和RedisConfig2的listenerAdapter的MessageListenerAdapter参数2相同
        System.out.println("这是RedisPmpSub---" + message);
    }

}
```

3、定义订阅相关配置

```java
@Configuration
public class RedisPubSubConfig {

    /**
     * Redis消息监听器容器
     *
     * @param connectionFactory /
     * @return /
     */
    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        //订阅了一个叫pmp和channel 的通道，多通道
        container.addMessageListener(listenerAdapter(new RedisPmpSub()), new PatternTopic("pmp"));
        container.addMessageListener(listenerAdapter(new RedisChannelSub()), new PatternTopic("channel"));
        //这个container 可以添加多个 messageListener
        return container;
    }

    /**
     * 配置消息接收处理类
     *
     * @param redisMsg 自定义消息接收类
     * @return Redis的监听适配器
     */
    @Bean
    @Scope("prototype")
    MessageListenerAdapter listenerAdapter(RedisMsg redisMsg) {
        //这个地方 是给messageListenerAdapter 传入一个消息接受的处理器，利用反射的方法调用“receiveMessage”
        //也有好几个重载方法，这边默认调用处理器的方法 叫handleMessage 可以自己到源码里面看
        //注意2个通道调用的方法都要为receiveMessage
        return new MessageListenerAdapter(redisMsg, "receiveMessage");
    }

}
```

4、定义发布者

这里使用定时发布（当然也可以根据业务情况触发消息的发布，比如使用接口触发）

```java
@EnableScheduling
@Component
public class TestScheduleRedisPublishController {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 向redis消息队列index通道发布消息
     */
    @Scheduled(fixedRate = 2000)
    public void sendMessage() {
        stringRedisTemplate.convertAndSend("pmp", String.valueOf(Math.random()));
        stringRedisTemplate.convertAndSend("channel", String.valueOf(Math.random()));
    }

}
```

5、启动程序后的结果

![结果](https://cos.duktig.cn/typora/202112141953282.png)

可以看到两个订阅者，都可以正常的接收消息。



## 参看

- [springboot入门--springboot集成redis实现消息发布订阅模式-双通道](https://blog.csdn.net/llll234/article/details/80966952)
- [redis发布订阅模式](https://zhuanlan.zhihu.com/p/184948451)
- [redis发布订阅模式用做消息队列和rabbitmq的区别](https://blog.csdn.net/weixin_34061042/article/details/93027056)

