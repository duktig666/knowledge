# 背景

上篇文章介绍了 解决**分布式ID问题**的各种方案，详情可参看上篇文章：[分布式ID常用方案——UUID、MySQL、Redis、ZooKeeper、雪花算法、美团Leaf……](https://duktig.cn/archives/85/)

本篇文章着重介绍 **Redis生成分布式ID**

# Redis实现分布式ID分析

关于分布式ID的基本要求和背景不在赘述。

## 为什么使用Redis可以解决分布式ID问题？

> Redis的所有命令操作都是单线程的，本身提供像 `incr` 和 `increby` 这样的自增原子命令，所以能保证生成的 ID 肯定是唯一有序的。

## 优缺点分析

优点：

- 不依赖于数据库，灵活方便，且性能优于数据库；
- 数字ID天然排序，对分页或者需要排序的结果很有帮助。

缺点：

- 如果系统中没有Redis，还需要引入新的组件，增加系统复杂度；
- 需要编码和配置的工作量比较大。

考虑到单节点的性能瓶颈，可以使用 Redis 集群来获取更高的吞吐量。使用 Redis 集群也可以方式单点故障的问题。

## 实现思路

使用 `incr` 和 `increby` 这样的自增原子命令本身已经可以实现类似于MySQL自增有序，但这样可能在某些特定的情况下存在一些问题：

- ID自增，有数据安全的隐患，比如：订单id，可推测一天大概的订单量。
- 无具体的含义，有时可能想知道生成唯一ID的时间（对时间敏感的业务可不掺杂时间）。

### 思路一

所以一般会掺入一些自定义的逻辑实现，比如：

【**年份 + 当天距当年第多少天 + 天数 + 小时 + redis自增**】

这样的唯一ID比较常用，也比较易实现。但也有一些别的问题，比如：

- 和时间高度耦合
- 不能根据不同的业务进行区分

### 思路二（本文采用的方案）

这次我们采用另一种算法思路来实现：

【**业务前缀标识 + 时间 + 递增序列 + 随机数**】

这样的优点是：

- 可以根据不同的业务，采用不同的分布式ID生成的规则
- 具有更强的随机性，在自增的基础上进一步引入随机数，来提供同一时刻的并发量（Redis为单线程可不考虑，但若是别的实现方案，这是一个不错的优化）。



# 实现

## 1. SpringBoot+Redis基本环境搭建

### 1.1 引入依赖

```xml
<!--    hutool的java开发工具包    -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
</dependency>
<!-- SpringBoot Boot Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
</dependency>
<!--lombok插件-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<!--Spring boot 测试-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 1.2 编写application.yml

```yaml
server:
  port: 8000

spring:
  jackson:
    time-zone: Asia/Shanghai
    serialization:
      # Date返回前端转时间戳 但不能解决LocalDateTime转时间戳（JacksonCustomizerConfig类解决）
      write-dates-as-timestamps: true

  data:
    redis:
      repositories:
        enabled: false
  aop:
    proxy-target-class: true

  redis:
    #数据库索引
    database: 0
    host: 127.0.0.1
    port: 6379
    password:
    #连接超时时间（ms）
    timeout: 5000
    jedis:
      pool:
        # 连接池最大连接数（使用负值表示没有限制）
        max-active: -1
        # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1
```

### 1.3 编写启动类

```java
@SpringBootApplication
public class RedisApplication {
    public static void main(String[] args) {
        SpringApplication.run(RedisApplication.class, args);
    }
}
```

### 1.4 配置Redis

```java
@Slf4j
@Configuration
@EnableCaching
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
public class RedisConfig extends CachingConfigurerSupport {

    /**
     * 设置 redis 数据默认过期时间，默认2小时
     * 设置@cacheable 序列化方式
     */
    @Bean
    public RedisCacheConfiguration redisCacheConfiguration() {
        FastJsonRedisSerializer<Object> fastJsonRedisSerializer = new FastJsonRedisSerializer<>(Object.class);
        RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig();
        configuration =
                configuration.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(fastJsonRedisSerializer)).entryTtl(Duration.ofHours(2));
        return configuration;
    }

    @SuppressWarnings("all")
    @Bean(name = "redisTemplate")
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        //序列化
        FastJsonRedisSerializer<Object> fastJsonRedisSerializer = new FastJsonRedisSerializer<>(Object.class);
        // value值的序列化采用fastJsonRedisSerializer
        template.setValueSerializer(fastJsonRedisSerializer);
        template.setHashValueSerializer(fastJsonRedisSerializer);
        // 全局开启AutoType，这里方便开发，使用全局的方式
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        // 建议使用这种方式，小范围指定白名单
        // ParserConfig.getGlobalInstance().addAccept("me.zhengjie.domain");
        // key的序列化采用StringRedisSerializer
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        template.afterPropertiesSet();
        return template;
    }

    /**
     * 自定义缓存key生成策略，默认将使用该策略
     */
    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return (target, method, params) -> {
            Map<String, Object> container = new HashMap<>(3);
            Class<?> targetClassClass = target.getClass();
            // 类地址
            container.put("class", targetClassClass.toGenericString());
            // 方法名称
            container.put("methodName", method.getName());
            // 包名称
            container.put("package", targetClassClass.getPackage());
            // 参数列表
            for (int i = 0; i < params.length; i++) {
                container.put(String.valueOf(i), params[i]);
            }
            // 转为JSON字符串
            String jsonString = JSON.toJSONString(container);
            // 做SHA256 Hash计算，得到一个SHA256摘要作为Key
            return DigestUtil.sha256(jsonString);
        };
    }

    @Bean
    @Override
    public CacheErrorHandler errorHandler() {
        // 异常处理，当Redis发生异常时，打印日志，但是程序正常走
        log.info("初始化 -> [{}]", "Redis CacheErrorHandler");
        return new CacheErrorHandler() {
            @Override
            public void handleCacheGetError(RuntimeException e, Cache cache, Object key) {
                log.error("Redis occur handleCacheGetError：key -> [{}]", key, e);
            }

            @Override
            public void handleCachePutError(RuntimeException e, Cache cache, Object key, Object value) {
                log.error("Redis occur handleCachePutError：key -> [{}]；value -> [{}]", key, value, e);
            }

            @Override
            public void handleCacheEvictError(RuntimeException e, Cache cache, Object key) {
                log.error("Redis occur handleCacheEvictError：key -> [{}]", key, e);
            }

            @Override
            public void handleCacheClearError(RuntimeException e, Cache cache) {
                log.error("Redis occur handleCacheClearError：", e);
            }
        };
    }

}

/**
 * Value 序列化
 *
 * @param <T>
 * @author /
 */
class FastJsonRedisSerializer<T> implements RedisSerializer<T> {

    private final Class<T> clazz;

    FastJsonRedisSerializer(Class<T> clazz) {
        super();
        this.clazz = clazz;
    }

    @Override
    public byte[] serialize(T t) {
        if (t == null) {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(StandardCharsets.UTF_8);
    }

    @Override
    public T deserialize(byte[] bytes) {
        if (bytes == null || bytes.length <= 0) {
            return null;
        }
        String str = new String(bytes, StandardCharsets.UTF_8);
        return JSON.parseObject(str, clazz);
    }

}

/**
 * 重写序列化器
 *
 * @author /
 */
class StringRedisSerializer implements RedisSerializer<Object> {

    private final Charset charset;

    StringRedisSerializer() {
        this(StandardCharsets.UTF_8);
    }

    private StringRedisSerializer(Charset charset) {
        Assert.notNull(charset, "Charset must not be null!");
        this.charset = charset;
    }

    @Override
    public String deserialize(byte[] bytes) {
        return (bytes == null ? null : new String(bytes, charset));
    }

    @Override
    public byte[] serialize(Object object) {
        String string = JSON.toJSONString(object);
        if (StrUtil.isBlank(string)) {
            return null;
        }
        string = string.replace("\"", "");
        return string.getBytes(charset);
    }
}
```

### 1.5 编写Redis工具类，用以操作Redis

由于篇幅太长，仅展示和分布式ID相关的方法，详情可参看源码。

```java
	//---------------------redis  incr/decr 相关----------------------------

    /**
     * 设置自增/自减初始值
     *
     * @param key
     * @param value
     * @param timeout
     * @param unit
     */
    public void setAtomicValue(String key, int value, long timeout, TimeUnit unit) {
        RedisAtomicLong redisAtomicLong = new RedisAtomicLong(key, redisTemplate.getConnectionFactory(), value);
        redisAtomicLong.expire(timeout, unit);
    }

    /**
     * 在redis中自增并获取数据
     *
     * @param key 键
     * @return 自增后的值
     */
    public long incr(String key) {
        RedisAtomicLong redisAtomicLong = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
        return redisAtomicLong.incrementAndGet();
    }

    /**
     * 在redis中自增并获取数据，并设置过期时间
     *
     * @param key     键
     * @param timeout 过期时间
     * @param unit    过期时间单位
     * @return 自增后的值
     */
    public long incr(String key, long timeout, TimeUnit unit) {
        RedisAtomicLong redisAtomicLong = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
        redisAtomicLong.expire(timeout, unit);
        return redisAtomicLong.incrementAndGet();
    }

    /**
     * 在redis中自增指定步长并获取数据
     *
     * @param key       键
     * @param increment 步长
     * @return 自增后的值
     */
    public long incr(String key, int increment) {
        RedisAtomicLong redisAtomicLong = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
        return redisAtomicLong.addAndGet(increment);
    }

    /**
     * 在redis中自增指定步长并获取数据，并设置过期时间
     *
     * @param key       键
     * @param increment 步长
     * @param timeout   过期时间
     * @param unit      过期时间单位
     * @return 自增后的值
     */
    public long incr(String key, int increment, long timeout, TimeUnit unit) {
        RedisAtomicLong redisAtomicLong = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
        redisAtomicLong.expire(timeout, unit);
        return redisAtomicLong.addAndGet(increment);
    }
```



## 2. 分布式唯一ID服务搭建

### 2.1 编写必要的常量类

这些常量也可放到`application.yml`去实现。

```java
/**
 * description:唯一id 常量
 *
 * @author RenShiWei
 * Date: 2021/8/20 11:26
 **/
public class UniqueIDConstants {

    /**
     * 单号流水号缓存Key前缀
     */
    public static final String SERIAL_CACHE_PREFIX = "UNIQUE_CACHE_";

    /**
     * 单号流水号yyMMdd前缀
     */
    public static final String SERIAL_YYMMDD_PREFIX = "yyMMdd";

    /**
     * 单号流水号yyyyMMdd前缀
     */
    public static final String SERIAL_YYYYMMDD_PREFIX = "yyyyMMdd";

    /**
     * 默认缓存天数
     */
    public static final int DEFAULT_CACHE_DAYS = 7;

}

```

### 2.2 提供枚举类

**为了实现通用性，采用不同的枚举对应不同的分布式唯一ID的生成规则。**

```java
/**
 * description:唯一ID枚举类
 *
 * @author RenShiWei
 * Date: 2021/8/20 11:16
 **/
@Getter
@AllArgsConstructor
public enum UniqueIDEnum {

    /**
     * 测试单号
     */
    TS_ORDER("YF", "yyyyMMdd", 7, 3, 20),
    ;

    /**
     * 单号前缀
     * 为空时填""
     */
    private String prefix;

    /**
     * 时间格式表达式
     * 例如：yyyyMMdd
     */
    private String datePattern;

    /**
     * 流水号长度
     */
    private Integer serialLength;

    /**
     * 随机数长度
     */
    private Integer randomLength;

    /**
     * 总长度
     */
    private Integer totalLength;

}

```

### 2.3 提供工具类

编写工具类，用来生成特定的唯一ID的规则。

```java
/**
 * description:唯一ID生成工具类
 *
 * @author RenShiWei
 * Date: 2021/8/20 11:22
 **/
public class RedisUniqueIDUtil {

    /**
     * 生成单号前缀：自定义前缀 + 一定格式的时间
     *
     * @param uniqueIdEnum 自定义的枚举
     * @return 单号前缀
     */
    public static String getFormNoPrefix(UniqueIDEnum uniqueIdEnum) {
        //格式化时间
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(uniqueIdEnum.getDatePattern());
        StringBuffer sb = new StringBuffer();
        sb.append(uniqueIdEnum.getPrefix());
        sb.append(formatter.format(LocalDateTime.now()));
        return sb.toString();
    }

    /**
     * 构建流水号缓存Key
     *
     * @param serialPrefix 流水号前缀
     * @return 流水号缓存Key
     */
    public static String getCacheKey(String serialPrefix) {
        return UniqueIDConstants.SERIAL_CACHE_PREFIX.concat(serialPrefix);
    }

    /**
     * 补全流水号
     *
     * @param serialPrefix      单号前缀
     * @param incrementalSerial 当天自增流水号
     */
    public static String completionSerial(String serialPrefix, Long incrementalSerial, UniqueIDEnum uniqueIdEnum) {
        StringBuffer sb = new StringBuffer(serialPrefix);
        //需要补0的长度=流水号长度 -当日自增计数长度
        int length = uniqueIdEnum.getSerialLength() - String.valueOf(incrementalSerial).length();
        //补零
        for (int i = 0; i < length; i++) {
            sb.append("0");
        }
        //redis当日自增数
        sb.append(incrementalSerial);
        return sb.toString();
    }

    /**
     * 补全随机数
     *
     * @param serialWithPrefix 当前单号
     * @param uniqueIdEnum     单号生成枚举
     */
    public static String completionRandom(String serialWithPrefix, UniqueIDEnum uniqueIdEnum) {
        StringBuffer sb = new StringBuffer(serialWithPrefix);
        //随机数长度
        int length = uniqueIdEnum.getRandomLength();
        if (length > 0) {
            Random random = new Random();
            for (int i = 0; i < length; i++) {
                //十以内随机数补全
                sb.append(random.nextInt(10));
            }
        }
        return sb.toString();
    }

}

```

### 2.4 提供分布式ID的接口

```java
public interface IRedisUniqueIDService {

    /**
     * 根据单号枚举 生成唯一单号
     *
     * @param uniqueIdEnum 单号枚举
     * @return 唯一单号
     */
    String generateUniqueId(UniqueIDEnum uniqueIdEnum);

}
```

### 2.5 提供接口实现

```java
/**
 * description:唯一id生成服务
 *
 * @author RenShiWei
 * Date: 2021/8/20 11:38
 **/
@Service
@RequiredArgsConstructor
public class RedisUniqueIDServiceImpl implements IRedisUniqueIDService {

    private final RedisUtils redisUtils;

    /**
     * 根据单号枚举 生成唯一单号
     *
     * @param uniqueIdEnum 单号枚举
     * @return 唯一单号
     */
    @Override
    public String generateUniqueId(UniqueIDEnum uniqueIdEnum) {
        //获得单号前缀 格式 固定前缀 +时间前缀 示例 ：YF20190101
        String prefix = RedisUniqueIDUtil.getFormNoPrefix(uniqueIdEnum);
        //获得缓存key
        String cacheKey = RedisUniqueIDUtil.getCacheKey(prefix);
        //获得当日自增数，并设置时间
        Long incrementalSerial = redisUtils.incr(cacheKey, UniqueIDConstants.DEFAULT_CACHE_DAYS, TimeUnit.DAYS);
        //组合单号并补全流水号
        String serialWithPrefix = RedisUniqueIDUtil.completionSerial(prefix, incrementalSerial, uniqueIdEnum);
        //补全随机数
        return RedisUniqueIDUtil.completionRandom(serialWithPrefix, uniqueIdEnum);
    }

}
```



## 3. 测试

### 3.1 测试代码

```java
/**
 * description: 测试Redis生成唯一ID
 *
 * @author RenShiWei
 * Date: 2021/08/20 17:26
 **/
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Slf4j
public class IRedisUniqueIDServiceTest {

    @Autowired
    private IRedisUniqueIDService redisUniqueIDService;

    /**
     * 测试生成唯一id
     */
    @Test
    public void generateUniqueId() {
        String uniqueId = redisUniqueIDService.generateUniqueId(UniqueIDEnum.TS_ORDER);
        System.out.println("唯一id：" + uniqueId);
    }

    /**
     * 多线程环境下，测试生成唯一ID
     */
    @Test
    public void generateUniqueIdForThread() {
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + redisUniqueIDService.generateUniqueId(UniqueIDEnum.TS_ORDER));
            }
        }, "thread-0").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + redisUniqueIDService.generateUniqueId(UniqueIDEnum.TS_ORDER));
            }
        }, "thread-1").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "分布式唯一ID:" + redisUniqueIDService.generateUniqueId(UniqueIDEnum.TS_ORDER));
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

### 3.2 单线程下测试

**测试结果**：

```java
唯一id：YF202108200000001793
```

**分析**：

- `YF`为具体业务的前缀
- `20210820`为当前时间（若不想掺杂时间，可去掉或更换别的逻辑）
- `0000001`为递增序列（长度可在枚举类中配置）
- `793`随机数（随机数的位数可配置）

### 3.3 多线程测试

**测试结果**：

```java
thread-1分布式唯一ID:YF202108200000003085
thread-0分布式唯一ID:YF202108200000004932
thread-2分布式唯一ID:YF202108200000002144
thread-0分布式唯一ID:YF202108200000005912
thread-2分布式唯一ID:YF202108200000007478
thread-1分布式唯一ID:YF202108200000006075
thread-0分布式唯一ID:YF202108200000008986
thread-2分布式唯一ID:YF202108200000009319
thread-1分布式唯一ID:YF202108200000010986
thread-0分布式唯一ID:YF202108200000011753
thread-2分布式唯一ID:YF202108200000012119
thread-1分布式唯一ID:YF202108200000013123
thread-0分布式唯一ID:YF202108200000014994
thread-2分布式唯一ID:YF202108200000015649
thread-1分布式唯一ID:YF202108200000016526
thread-0分布式唯一ID:YF202108200000018049
thread-2分布式唯一ID:YF202108200000017602
thread-0分布式唯一ID:YF202108200000019267
thread-1分布式唯一ID:YF202108200000020557
thread-2分布式唯一ID:YF202108200000021312
thread-1分布式唯一ID:YF202108200000022026
thread-0分布式唯一ID:YF202108200000023409
thread-1分布式唯一ID:YF202108200000024375
thread-0分布式唯一ID:YF202108200000025967
thread-1分布式唯一ID:YF202108200000026413
thread-1分布式唯一ID:YF202108200000027283
thread-0分布式唯一ID:YF202108200000028386
thread-2分布式唯一ID:YF202108200000029551
thread-2分布式唯一ID:YF202108200000030513
thread-2分布式唯一ID:YF202108200000031485
```

**分析**：

- 所有ID唯一，无重复
- 趋势递增

满足分布式ID的基本需求。

### 3.4 单线程生成10万个分布式ID测试

```java
/**
     * 单机Redis
     * 单线程生成10W个 分布式ID 测速
     * 大约为：110353 ms
     */
@Test
public void generateUniqueIdForMore() {
    LocalDateTime startTime = LocalDateTime.now();
    for (int i = 0; i < 100000; i++) {
        String id = redisUniqueIDService.generateUniqueId(UniqueIDEnum.TS_ORDER);
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
YF202108210299990446
YF202108210299991384
YF202108210299992343
YF202108210299993964
YF202108210299994746
YF202108210299995569
YF202108210299996927
YF202108210299997418
YF202108210299998532
YF202108210299999065
YF202108210300000330

生成10万个分布式id所用的时间：110353 ms
```

### 3.5 线程池开10个线程生成10万个分布式ID测试

```java
/**
     * 单机Redis
     * 线程池开10个线程生成10W个 分布式ID 测速
     * 大约为：106959 ms
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

class ThreadPoolTask implements Callable<String> {

    @Override
    public String call() {
        String id = redisUniqueIDService.generateUniqueId(UniqueIDEnum.TS_ORDER);
        System.out.println(Thread.currentThread().getName() + "---" + id);
        return id;
    }

}
```

结果：

```java
……
pool-1-thread-9---YF202108210512251586
pool-1-thread-5---YF202108210512252730
pool-1-thread-7---YF202108210512253382
pool-1-thread-8---YF202108210512254595
pool-1-thread-6---YF202108210512255710
pool-1-thread-2---YF202108210512256426
pool-1-thread-1---YF202108210512257214
pool-1-thread-3---YF202108210512258137
pool-1-thread-10---YF202108210512259590
pool-1-thread-4---YF202108210512260124
pool-1-thread-9---YF202108210512261979
pool-1-thread-5---YF202108210512262578
pool-1-thread-7---YF202108210512263373
pool-1-thread-8---YF202108210512264900
pool-1-thread-6---YF202108210512265949
pool-1-thread-2---YF202108210512266538
pool-1-thread-1---YF202108210512267429
pool-1-thread-3---YF202108210512268883
pool-1-thread-10---YF202108210512269325
pool-1-thread-4---YF202108210512270395

线程池 生成10万个分布式id所用的时间：106959 ms
```

## 4. Redis与ZooKeeper实现分布式ID对比

环境：单机的Redis和单机的ZooKeeper进行测试

|                                  | Redis     | ZooKeeper                        |
| -------------------------------- | --------- | -------------------------------- |
| 单线程10万分布式ID               | 110353 ms | 3060085 ms  大约为 51min         |
| 线程池开10个线程生成10万分布式ID | 106959 ms | 3073690 ms  基本和单线程环境一致 |

## 5. 小结

本次搭建基本实现了Redis提供分布式ID，在高并发情况下，生成分布式ID的速度远远大于ZooKeeper的方式。

因为Redis的命令执行是多线程的，所以开线程池多线程执行，几乎和单线程的效率一致。

目前是以服务的形式提供的，后续优化空间：

- 初次搭成，还可以做的更加通用
- 可以学学将服务打成jar包，放到maven仓库用来直接引用
- 可以学学怎么构建SpringBoot的启动器，做到更好的开箱即用

## 参考

- [基于Redis实现分布式单号，分布式ID（自定义规则生成）](https://blog.csdn.net/qq_38011415/article/details/85555460)
- [【redis】使用redis RedisAtomicLong生成自增的ID值](https://blog.csdn.net/smilefyx/article/details/73511243)
- [一线大厂的分布式唯一 ID 生成方案是什么样的](https://zhuanlan.zhihu.com/p/140078865)
- [基于redis的分布式ID生成器](https://developer.aliyun.com/article/6063)

