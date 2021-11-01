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

