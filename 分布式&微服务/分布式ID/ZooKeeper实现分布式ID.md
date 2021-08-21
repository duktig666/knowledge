# 背景

上篇文章介绍了 解决**分布式ID问题**的各种方案，详情可参看上篇文章：[分布式ID常用方案——UUID、MySQL、Redis、ZooKeeper、雪花算法、美团Leaf……](https://duktig.cn/archives/85/)

可参看：[Redis构建分布式唯一ID生成器](https://duktig.cn/archives/88/)

本篇文章着重介绍 **ZooKeeper生成分布式ID**



源码参看：[https://github.com/duktig666/distributed-programme](https://github.com/duktig666/distributed-programme)

# ZooKeeper实现分布式ID分析

**ZooKeeper分布式ID生成,原理是利用ZooKeeper的临时有序节点，生成全局唯一的ID**。

多个客户端同时创建同一节点，zk保证了能有序的创建，创建成功并返回的path类似于/root/generateid0000000001酱紫的，可以看到是顺序有规律的，能较好的解决这个问题，缺点是，会依赖于zk。

# 实现

## 1. 环境搭建

### 1.1 引入依赖

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.7</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.8.2</version>
</dependency>
<!--    curator   start -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.3.0</version>
</dependency>
<!--    curator  end  -->
<!--lombok插件-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

## 2. ZooKeeper实现分布式ID

### 2.1 代码实现

```java
/**
 * description:zk实现分布式id
 *
 * @author RenShiWei
 * Date: 2021/8/20 9:20
 **/
@Slf4j
public class ZookeeperUniqueID implements Watcher {

    /** 计数器对象 */
    public static CountDownLatch countDownLatch = new CountDownLatch(1);

    /** 连接对象 */
    public static ZooKeeper zooKeeper;

    private final String IP = "127.0.0.1:2181";

    /** 用户生成序号的节点 */
    private final String DEFAULT_PATH = "/uniqueId";
    /** 根节点 */
    private final String ROOT_PATH = "/uniqueId";

    public ZookeeperUniqueID() {
        try {
            zooKeeper = new ZooKeeper(IP, 6000, this);
            //等待zk正常连接后，往下走程序
            countDownLatch.await();
            // 判断根节点是否存在
            Stat stat = zooKeeper.exists(ROOT_PATH, false);
            if (stat == null) {
                // 创建一下根节点
                zooKeeper.create(ROOT_PATH, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        try {
            //EventType = None时
            if (watchedEvent.getType() == Watcher.Event.EventType.None) {
                if (watchedEvent.getState() == Watcher.Event.KeeperState.SyncConnected) {
                    log.info("连接成功");
                    countDownLatch.countDown();
                } else if (watchedEvent.getState() == Watcher.Event.KeeperState.Disconnected) {
                    log.info("断开连接");
                } else if (watchedEvent.getState() == Watcher.Event.KeeperState.Expired) {
                    log.info("会话超时");
                    // 超时后服务器端已经将连接释放，需要重新连接服务器端
                    zooKeeper = new ZooKeeper(IP, 6000, this);
                } else if (watchedEvent.getState() == Watcher.Event.KeeperState.AuthFailed) {
                    log.info("认证失败");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 根据 zk 临时有序节点，生成唯一ID
     *
     * @return 唯一ID
     */
    public String getUniqueId() {
        String path = "";
        //创建临时有序节点
        try {
            path = zooKeeper.create(ROOT_PATH + DEFAULT_PATH, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);
            log.info("zk创建临时有序节点：{}", path);
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
        //截取defaultPath的长度
        return path.substring(DEFAULT_PATH.length());
    }

}

```

### 2.2 测试

```java
public class ZookeeperUniqueIDTest {

    /**
     * 测试 zk 生成唯一ID
     */
    @Test
    public void getUniqueId() {
        ZookeeperUniqueID zookeeperUniqueID = new ZookeeperUniqueID();
        for (int i = 0; i < 10; i++) {
            System.out.println("分布式唯一ID:" + zookeeperUniqueID.getUniqueId());
        }
    }

    /**
     * 测试 zk 生成唯一ID（多线程下）
     */
    @Test
    public void getUniqueIdForMore() {
        ZookeeperUniqueID zookeeperUniqueID = new ZookeeperUniqueID();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + zookeeperUniqueID.getUniqueId());
            }
        }, "thread-0").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + zookeeperUniqueID.getUniqueId());
            }
        }, "thread-1").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + zookeeperUniqueID.getUniqueId());
            }
        }, "thread-2").start();

        //睡眠，用以保证有充足的时间执行
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

}
```

### 2.3 结果

**代码执行结果**：

```Java
 2021-08-20 20:45:59,091 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - 连接成功
 2021-08-20 20:45:59,133 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000060
 thread-2分布式唯一ID:0000000060
2021-08-20 20:45:59,133 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000061
 thread-0分布式唯一ID:0000000061
2021-08-20 20:45:59,135 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000062
 thread-1分布式唯一ID:0000000062
2021-08-20 20:45:59,153 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000063
 thread-2分布式唯一ID:0000000063
2021-08-20 20:45:59,153 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000064
 thread-0分布式唯一ID:0000000064
2021-08-20 20:45:59,177 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000065
 thread-1分布式唯一ID:0000000065
2021-08-20 20:45:59,197 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000066
 thread-2分布式唯一ID:0000000066
2021-08-20 20:45:59,197 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000067
 thread-0分布式唯一ID:0000000067
2021-08-20 20:45:59,221 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000068
 thread-1分布式唯一ID:0000000068
2021-08-20 20:45:59,241 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000069
 thread-2分布式唯一ID:0000000069
2021-08-20 20:45:59,242 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000070
 thread-0分布式唯一ID:0000000070
2021-08-20 20:45:59,265 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000072
 thread-2分布式唯一ID:0000000072
2021-08-20 20:45:59,265 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000073
 thread-0分布式唯一ID:0000000073
2021-08-20 20:45:59,265 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000071
 thread-1分布式唯一ID:0000000071
2021-08-20 20:45:59,289 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000074
 thread-2分布式唯一ID:0000000074
2021-08-20 20:45:59,321 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000075
 thread-0分布式唯一ID:0000000075
2021-08-20 20:45:59,321 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000076
 thread-1分布式唯一ID:0000000076
2021-08-20 20:45:59,323 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000077
 thread-2分布式唯一ID:0000000077
2021-08-20 20:45:59,342 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000078
 thread-0分布式唯一ID:0000000078
2021-08-20 20:45:59,342 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000079
 thread-1分布式唯一ID:0000000079
2021-08-20 20:45:59,343 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000080
 thread-2分布式唯一ID:0000000080
2021-08-20 20:45:59,376 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000081
 thread-0分布式唯一ID:0000000081
2021-08-20 20:45:59,399 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000082
 thread-1分布式唯一ID:0000000082
2021-08-20 20:45:59,400 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000083
 thread-2分布式唯一ID:0000000083
2021-08-20 20:45:59,400 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000084
 thread-0分布式唯一ID:0000000084
2021-08-20 20:45:59,421 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000085
 thread-1分布式唯一ID:0000000085
2021-08-20 20:45:59,422 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000086
 thread-2分布式唯一ID:0000000086
2021-08-20 20:45:59,422 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000087
 thread-0分布式唯一ID:0000000087
2021-08-20 20:45:59,440 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000088
 thread-1分布式唯一ID:0000000088
2021-08-20 20:45:59,510 INFO [cn.duktig.learn.id.ZookeeperUniqueID] - zk创建临时有序节点：/uniqueId/uniqueId0000000089
 thread-1分布式唯一ID:0000000089
```

**Zookeeper Client 查看节点数据**：

![Zookeeper Client 节点数据](https://cos.duktig.cn/typora/20210820204818.png)

**分析**：

- 所有ID唯一，无重复
- 多线程环境下可保证ID自增

### 2.4 优化空间

同Redis实现分布式ID中的分析一致，这样强一致的递增，在很多的情况下并不一定适用，存在以下可能的问题：

- ID自增，有数据安全的隐患，比如：订单id，可推测一天大概的订单量。
- 无具体的含义，有时可能想知道生成唯一ID的时间（对时间敏感的业务可不掺杂时间）。

> 目前就先不做扩展了，如果想做扩展，可参考我那篇Redis的实现，原理基本都差不多，只是将中间那段有序序列替换了而已。

### 2.5 原生ZooKeeper的API实现的一些问题

1. 会话连接是异步的，需要自己去处理。比如使用 CountDownLatch
2. Watch 需要重复注册，不然就不能生效
3. 开发的复杂性还是比较高的
4. 不支持多节点删除和创建，需要自己去递归。

因此可以使用ZooKeeper的框架——Curator。

## 3. Curator实现分布式ID

### 3.1 简介

> Curator是Netflix公司开源的一套ZooKeeper客户端框架，提供了一套易用性和可读性更强的Fluent风格的客户端API框架。为ZooKeeper客户端框架提供了一些比较普遍的、开箱即用的、分布式开发用的解决方案，例如Recipe、共享锁服务、Master选举机制和分布式计算器等，帮助开发者避免了“重复造轮子”的无效开发工作。
>
> Guava is to Java that Curator to ZooKeeper
>
> 更多Curator介绍参考：《Zookeeper开源客户端框架Curator简介》[https://www.iteye.com/blog/macrochen-1366136](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.iteye.com%2Fblog%2Fmacrochen-1366136)

### 3.2 代码实现

```java
/**
 * description:利用 Curator框架 生成唯一id （底层使用zk）
 *
 * @author RenShiWei
 * Date: 2021/8/20 10:05
 **/
@Slf4j
public class CuratorUniqueID {

    private static CuratorFramework curatorFrameworkClient;

    private static RetryPolicy retryPolicy;

    private static final String IP = "127.0.0.1:2181";

    private static String ROOT = "/uniqueId-curator";

    private static String NODE_NAME = "/uniqueId";

    static {
        retryPolicy = new ExponentialBackoffRetry(1000, 3);
        curatorFrameworkClient = CuratorFrameworkFactory
                .builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .connectionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .build();
        curatorFrameworkClient.start();
        try {
            //请先判断父节点/root节点是否存在
            Stat stat = curatorFrameworkClient.checkExists().forPath(ROOT);
            if (stat == null) {
                curatorFrameworkClient.create().withMode(CreateMode.PERSISTENT).forPath(ROOT, null);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 生成唯一id
     *
     * @return 唯一id
     */
    public static String generateId() {
        String backPath = "";
        String fullPath = ROOT.concat(NODE_NAME);
        try {
            // 关键点：创建临时顺序节点
            backPath = curatorFrameworkClient.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(fullPath,
                    null);
            log.info("zk创建临时有序节点：{}", backPath);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return backPath.substring(ROOT.length() + NODE_NAME.length());
    }
    
}
```

### 3.3 多线程测试

```java
public class CuratorUniqueIDTest {

    /**
     * 测试 Curator 生成分布式id
     */
    @Test
    public void generateId() {
        String id = CuratorUniqueID.generateId();
        System.out.println(id);
    }

    /**
     * 测试 Curator 生成唯一ID（多线程下）
     */
    @Test
    public void getUniqueIdForMore() {
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + CuratorUniqueID.generateId());
            }
        }, "thread-0").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + CuratorUniqueID.generateId());
            }
        }, "thread-1").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + CuratorUniqueID.generateId());
            }
        }, "thread-2").start();

        //睡眠，用以保证有充足的时间执行
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
}
```

**代码结果**：

```java
2021-08-20 21:17:39,104 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000061
 thread-2分布式唯一ID:0000000061
2021-08-20 21:17:39,106 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000062
 thread-0分布式唯一ID:0000000062
2021-08-20 21:17:39,107 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000063
 thread-1分布式唯一ID:0000000063
2021-08-20 21:17:39,131 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000064
 thread-2分布式唯一ID:0000000064
2021-08-20 21:17:39,134 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000065
 thread-0分布式唯一ID:0000000065
2021-08-20 21:17:39,147 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000066
 thread-1分布式唯一ID:0000000066
2021-08-20 21:17:39,182 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000067
 thread-2分布式唯一ID:0000000067
2021-08-20 21:17:39,183 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000068
 thread-0分布式唯一ID:0000000068
2021-08-20 21:17:39,211 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000069
 thread-1分布式唯一ID:0000000069
2021-08-20 21:17:39,219 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000070
 thread-2分布式唯一ID:0000000070
2021-08-20 21:17:39,220 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000071
 thread-0分布式唯一ID:0000000071
2021-08-20 21:17:39,258 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000072
 thread-1分布式唯一ID:0000000072
2021-08-20 21:17:39,258 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000073
 thread-2分布式唯一ID:0000000073
2021-08-20 21:17:39,259 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000074
 thread-0分布式唯一ID:0000000074
2021-08-20 21:17:39,283 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000075
 thread-1分布式唯一ID:0000000075
2021-08-20 21:17:39,283 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000077
 thread-0分布式唯一ID:0000000077
2021-08-20 21:17:39,283 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000076
 thread-2分布式唯一ID:0000000076
2021-08-20 21:17:39,313 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000078
 thread-1分布式唯一ID:0000000078
2021-08-20 21:17:39,339 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000079
 thread-0分布式唯一ID:0000000079
2021-08-20 21:17:39,341 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000080
 thread-2分布式唯一ID:0000000080
2021-08-20 21:17:39,341 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000081
 thread-1分布式唯一ID:0000000081
2021-08-20 21:17:39,368 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000082
 thread-0分布式唯一ID:0000000082
2021-08-20 21:17:39,369 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000083
 thread-2分布式唯一ID:0000000083
2021-08-20 21:17:39,373 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000084
 thread-1分布式唯一ID:0000000084
2021-08-20 21:17:39,400 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000085
 thread-0分布式唯一ID:0000000085
2021-08-20 21:17:39,400 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000086
 thread-2分布式唯一ID:0000000086
2021-08-20 21:17:39,401 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000087
 thread-1分布式唯一ID:0000000087
2021-08-20 21:17:39,421 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000088
 thread-0分布式唯一ID:0000000088
2021-08-20 21:17:39,423 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000089
 thread-2分布式唯一ID:0000000089
2021-08-20 21:17:39,423 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000000090
 thread-1分布式唯一ID:0000000090
```

**Zookeeper Client 查看节点数据**：

![Zookeeper Client节点数据](https://cos.duktig.cn/typora/20210820212017.png)

### 3.4 单线程生成10万个分布式ID测速

**ZooKeeper单机环境下，使用单线程，生成10万分布式ID测速**。

代码：

```java
/**
     * 单机ZooKeeper
     * 单线程生成10W个 分布式ID 测速
     * 大约为：3060085 ms  大约为 51min
     */
@Test
public void generateUniqueIdForMore() {
    LocalDateTime startTime = LocalDateTime.now();
    for (int i = 0; i < 100000; i++) {
        String id = CuratorUniqueID.generateId();
        System.out.println(id);
    }
    LocalDateTime endTime = LocalDateTime.now();
    // 计算时间差值
    long minutes = Duration.between(startTime, endTime).toMillis();
    // 输出
    System.out.println("生成10万个分布式id所用的时间：" + minutes + " ms");
}
```

结果：

```java
……
2021-08-21 10:29:28,568 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000100087
 0000100087
2021-08-21 10:29:28,603 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000100088
 0000100088
2021-08-21 10:29:28,645 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000100089
 0000100089
2021-08-21 10:29:28,681 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000100090
 0000100090
    
生成10万个分布式id所用的时间：3060085 ms
```

### 3.5 线程池10个线程生成10万个分布式ID测速

**ZooKeeper单机环境下，使用线程池开10个线程，生成10万分布式ID测速**。

```java
/**
     * 单机ZooKeeper
     * 线程池开10个线程生成10W个 分布式ID 测速
     * 大约为：3073690 ms  基本和单线程环境一致
     */
@Test
public void generateUniqueIdForThreadPoolExecutor() {
    ThreadPoolExecutor threadPoolExecutor = null;

    //创建线程池
    threadPoolExecutor = new ThreadPoolExecutor(10,
                                                20,
                                                10,
                                                TimeUnit.SECONDS,
                                                new LinkedBlockingQueue<>(20),
                                                new ThreadPoolExecutor.CallerRunsPolicy());
    LocalDateTime startTime = LocalDateTime.now();
    for (int i = 0; i < 100000; i++) {
        FutureTask<String> futureTask = new FutureTask<>(new ThreadPoolTask());
        threadPoolExecutor.execute(futureTask);
        try {
            String id = futureTask.get();
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
    threadPoolExecutor.shutdown();
    LocalDateTime endTime = LocalDateTime.now();
    // 计算时间差值
    long minutes = Duration.between(startTime, endTime).toMillis();
    // 输出
    System.out.println("线程池 生成10万个分布式id所用的时间：" + minutes + " ms");

}

static class ThreadPoolTask implements Callable<String> {

    @Override
    public String call() {
        String id = CuratorUniqueID.generateId();
        System.out.println(Thread.currentThread().getName() + "---" + id);
        return id;
    }

}
```

结果：

```java
……
2021-08-21 11:24:31,328 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200077
 pool-1-thread-5---0000200077
2021-08-21 11:24:31,353 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200078
 pool-1-thread-4---0000200078
2021-08-21 11:24:31,384 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200079
 pool-1-thread-2---0000200079
2021-08-21 11:24:31,408 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200080
 pool-1-thread-3---0000200080
2021-08-21 11:24:31,439 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200081
 pool-1-thread-8---0000200081
2021-08-21 11:24:31,463 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200082
 pool-1-thread-7---0000200082
2021-08-21 11:24:31,543 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200083
 pool-1-thread-1---0000200083
2021-08-21 11:24:31,574 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200084
 pool-1-thread-6---0000200084
2021-08-21 11:24:31,605 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200085
 pool-1-thread-10---0000200085
2021-08-21 11:24:31,629 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200086
 pool-1-thread-9---0000200086
2021-08-21 11:24:31,649 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200087
 pool-1-thread-5---0000200087
2021-08-21 11:24:31,674 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200088
 pool-1-thread-4---0000200088
2021-08-21 11:24:31,694 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200089
 pool-1-thread-2---0000200089
2021-08-21 11:24:31,718 INFO [cn.duktig.learn.id.CuratorUniqueID] - zk创建临时有序节点：/uniqueId-curator/uniqueId0000200090
 pool-1-thread-3---0000200090

线程池 生成10万个分布式id所用的时间：3073690 ms
```

## 4. Redis与ZooKeeper实现分布式ID对比

环境：单机的Redis和单机的ZooKeeper进行测试

|                                  | Redis     | ZooKeeper                        |
| -------------------------------- | --------- | -------------------------------- |
| 单线程10万分布式ID               | 110353 ms | 3060085 ms  大约为 51min         |
| 线程池开10个线程生成10万分布式ID | 106959 ms | 3073690 ms  基本和单线程环境一致 |



## 5. 小结

ZooKeeper实现分布式ID更适合并发量小，对数据强一致要求高的场景。相对来说，因为主从数据同步的原因，效率略低，如果对性能要求较高的话，可以考虑使用Redis。

关于具体的分布式ID选型，可以参考上一篇文章：[分布式ID常用方案——UUID、MySQL、Redis、ZooKeeper、雪花算法、美团Leaf……](https://duktig.cn/archives/85/)

