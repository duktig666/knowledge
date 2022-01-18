# RabbitMQ总结

## 1. RabbitMQ的消息模型

### "Hello World"（基本）消息模型

生产者将消息发送到队列，消费者从队列中获取消息，队列是存储消息的缓冲区。

![基本消息模型](https://gitee.com/koala010/typora/raw/master/img/20210701201946.png)

### Work模型

工作队列或者竞争消费者模式。

当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。此时就可以使用work模型:让多个消费者绑定到一个队列，共同消费队列中的消息。队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。

**一个消息只能被一个消费者消费**。

可以将平均消费，配置成能者多劳的模式。

![Work消息模型](https://gitee.com/koala010/typora/raw/master/img/20210701203022.png)

### Publish/Subscribe（订阅）模型——FanOut

- 1个生产者对应多个消费者
- 每一个消费者都有自己的队列。
- 每个队列都要绑定到交换机。
- 生产者只能将消息发送到交换机，交换机把消息发送给绑定过的所有队列。
- 队列的消费者都可以拿到消息，实现**一个消息被多个消费者获取**的目的。

X（Exchanges）：交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

![订阅模型-FanOut](https://gitee.com/koala010/typora/raw/master/img/20210701203221.png)

### Routing模型—Direct

在FanOut模式中，一个消息会被所有订阅的队列都消费。在某些场景下，需要不同的消息被不同的队列消费，这是要使用Direct类型的交换机。

在Direct下：

- 交换机与指定RoutingKey的队列绑定
- 消息发送方向交换机发送时，必须指定消息的RoutingKey
- 交换机将消息发送给指定的RoutingKey的队列，只有队列RoutingKey和消息的RoutingKey一致的才会接到消息

![订阅模型-Routing](https://gitee.com/koala010/typora/raw/master/img/20210701202927.png)

### Topic模型

与Direct类似，只是可以使用通配符来决定将消息发送给那些队列。

- `*`匹配到一个单词
- `#`匹配到多个单词



![订阅模型-Topic](https://gitee.com/koala010/typora/raw/master/img/20210701203342.png)

具体使用可参看：[rabbitmq五种消息模型整理](https://www.cnblogs.com/ifme/p/12024064.html)

## 2. RabbitMQ与Kafla对比

参看：[面试官：RabbitMQ 和 Kafka选哪个？](https://zhuanlan.zhihu.com/p/161224418)

## 3. RabbitMQ常见面试题

[如果面试问RabbitMQ，你可以吊打他 ！](https://zhuanlan.zhihu.com/p/62087283)

## 4. 为什么使用消息队列？

[为什么使用消息队列？](https://zhuanlan.zhihu.com/p/372485966)

## 5.RabbitMQ实现延时队列？

使用RabbitMQ来实现延迟任务必须先了解RabbitMQ的两个概念：**消息的TTL** 和 **死信Exchange**，通过这两者的组合来实现上述需求。

### 消息的TTL（Time To Live）

消息的TTL就是消息的存活时间。RabbitMQ可以对队列和消息分别设置TTL。对队列设置就是队列没有消费者连着的保留时间，也可以对每一个单独的消息做单独的设置。超过了这个时间，我们认为这个消息就死了，称之为死信。如果队列设置了，消息也设置了，那么会取小的。所以一个消息如果被路由到不同的队列中，这个消息死亡的时间有可能不一样（不同的队列设置）。这里单讲单个消息的TTL，因为它才是实现延迟任务的关键。

可以通过设置消息的expiration字段或者x-message-ttl属性来设置时间，两者是一样的效果。只是expiration字段是字符串参数，所以要写个int类型的字符串：

![img](https://cos.duktig.cn/typora/202201101007749.png)

当上面的消息扔到队列中后，过了3分钟，如果没有被消费，它就死了。不会被消费者消费到。这个消息后面的，没有“死掉”的消息对顶上来，被消费者消费。死信在队列中并不会被删除和释放，它会被统计到队列的消息数中去。单靠死信还不能实现延迟任务，还要靠Dead Letter Exchange。

### 死信交换机（Dead Letter Exchanges）

Exchage的概念在这里就不在赘述。一个消息在满足如下条件下，会进死信路由，记住这里是路由而不是队列，一个路由可以对应很多队列。

1. 一个消息被Consumer拒收了，并且reject方法的参数里requeue是false。也就是说不会被再次放在队列里，被其他消费者使用。

2. 上面的消息的TTL到了，消息过期了。

3. 队列的长度限制满了。排在前面的消息会被丢弃或者扔到死信路由上。

Dead Letter Exchange其实就是一种普通的exchange，和创建其他exchange没有两样。只是在某一个设置Dead Letter Exchange的队列中有消息过期了，会自动触发消息的转发，发送到Dead Letter Exchange中去。

### 延时队列的实现

延迟任务通过消息的TTL和Dead Letter Exchange来实现。我们需要建立2个队列，一个用于发送消息，一个用于消息过期后的转发目标队列。

![image-20220110101328411](https://cos.duktig.cn/typora/202201101013728.png)

生产者输出消息到Queue1，并且这个消息是设置有有效时间的，比如3分钟。消息会在Queue1中等待3分钟，如果没有消费者收掉的话，它就是被转发到Queue2，Queue2有消费者，收到，处理延迟任务。

[rabbitmq的延迟消息队列实现 ](https://www.cnblogs.com/yinfengjiujian/p/9204600.html)

