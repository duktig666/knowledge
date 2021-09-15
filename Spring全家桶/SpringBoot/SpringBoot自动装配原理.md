## 背景

使用过 Spring 的小伙伴，一定有被 XML 配置统治的恐惧。即使 Spring 后面引入了基于注解的配置，我们在开启某些 Spring 特性或者引入第三方依赖的时候，还是需要用 XML 或 Java 进行显式配置。

但是，SpringBoot 项目，我们只需要添加相关依赖，无需配置，通过启动的 `main` 方法即可。并且，我们通过 SpringBoot 的全局配置文件 `application.properties`或`application.yml`即可对项目进行设置比如更换端口号，配置 JPA 属性等等。

**为什么 SpringBoot 使用起来这么酸爽呢？** 这得益于其自动装配。**自动装配可以说是 SpringBoot 的核心，那究竟什么是自动装配呢？**

<br>

> 真实面试体验：
>
> 在 **Java后端** 的面试时，SpringBoot也是会经常性被问到的，最常见的两个问题：
>
> 1. **SpringBoot对比传统的SpringMVC的开发，有什么不同**？ ——①**简化配置，开箱即用** ②**约定大于配置**
> 2. **SpringBoot是如何实现简化配置、自动加载配置的？SpringBoot的Starter是如何实现的**？
>
> 其实第二个问题，都可以回归到本文的主题 “**SpringBoot 自动装配原理**” 。

## 什么是 SpringBoot 的自动装配？

我们现在提到自动装配的时候，一般会和 Spring Boot 联系在一起。但是，实际上 Spring Framework 早就实现了这个功能。Spring Boot 只是在其基础上，通过 **SPI** 的方式，做了进一步优化。

> **SPI** ，全称为 Service Provider Interface(服务提供者接口)，是一种服务发现机制。它通过在classpath路径下的`META-INF/services`文件夹查找文件，自动加载文件中所定义的类。

> SpringBoot 定义了一套接口规范，这套规范规定：**SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器**（此处涉及到 JVM 类加载机制与 Spring 的容器知识），**并执行类中定义的各种操作**。对于外部 jar 来说，只需要按照 SpringBoot 定义的标准，就能将自己的功能装置进 SpringBoot。

没有 Spring Boot 的情况下，如果我们需要引入第三方依赖，需要手动配置，非常麻烦。但是，Spring Boot 中，我们直接引入一个 starter 即可。引入 starter 之后，我们通过少量注解和一些简单的配置就能使用第三方组件提供的功能了。

在我看来，自动装配可以简单理解为：**通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能。**

## SpringBoot 是如何实现自动装配的？

先看一下 SpringBoot 的核心注解 `SpringBootApplication` 。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
<1.>@SpringBootConfiguration
<2.>@ComponentScan
<3.>@EnableAutoConfiguration
public @interface SpringBootApplication {

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //实际上它也是一个配置类
public @interface SpringBootConfiguration {
}
```

大概可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：**启用 SpringBoot 的自动配置机制**
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解**默认会扫描启动类所在的包下所有的类** ，可以自定义不扫描某些 bean。

`@EnableAutoConfiguration` 是实现自动装配的重要注解，我们以从这个注解入手。

### @EnableAutoConfiguration：实现自动装配的核心注解

`EnableAutoConfiguration` 只是一个简单地注解，自动装配核心功能的实现实际是通过 `AutoConfigurationImportSelector`类。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //作用：将main包下的所有组件注册到容器中
@Import({AutoConfigurationImportSelector.class}) //加载自动装配类 xxxAutoconfiguration
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

我们现在重点分析下`AutoConfigurationImportSelector` 类到底做了什么？

### AutoConfigurationImportSelector：加载自动装配类

`AutoConfigurationImportSelector`类的继承体系如下：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);
}
```

可以看出，`AutoConfigurationImportSelector` 类实现了 `ImportSelector`接口，也就实现了这个接口中的 `selectImports`方法，该方法主要用于**获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中**。

```java
private static final String[] NO_IMPORTS = new String[0];

public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>.判断自动装配开关是否打开
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
          //<2>.获取所有需要装配的bean
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
```

这里我们需要重点关注一下`getAutoConfigurationEntry()`方法，这个方法主要负责**获取所有候选的配置**。

![getAutoConfigurationEntry()方法 调用链路](https://cos.duktig.cn/typora/202109152021540.image)

`getAutoConfigurationEntry()`的源码：

```java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
        //<1>. 判断自动装配开关是否打开
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //<2>. 用于获取EnableAutoConfiguration注解 中的 exclude 和 excludeName
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //<3>. 获取需要自动装配的所有配置类，读取META-INF/spring.factories
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //<4>. 按需加载配置
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

**第 1 步**:

判断自动装配开关是否打开。默认`spring.boot.enableautoconfiguration=true`，可在 `application.properties` 或 `application.yml` 中设置

![第 1 步](https://cos.duktig.cn/typora/202109152024727.png)

**第 2 步** ：

用于获取`EnableAutoConfiguration`注解中的 `exclude` 和 `excludeName`。

![第 2 步](https://cos.duktig.cn/typora/202109152024799.png)

**第 3 步**：

获取需要自动装配的所有配置类，读取`META-INF/spring.factories`

```java
spring-boot/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories
```

![第 3-1 步](https://cos.duktig.cn/typora/202109152044629.png)

从下图可以看到这个文件的配置内容都被我们读取到了。`XXXAutoConfiguration`的作用就是按需加载组件。

![第 3-2 步](https://cos.duktig.cn/typora/202109152044161.png)

不光是这个依赖下的`META-INF/spring.factories`被读取到，所有 Spring Boot Starter 下的`META-INF/spring.factories`都会被读取到。

所以，你可以清楚滴看到， druid 数据库连接池的 Spring Boot Starter 就创建了`META-INF/spring.factories`文件。

如果，我们自己要创建一个 Spring Boot Starter，这一步是必不可少的。

![第 3-3 步](https://cos.duktig.cn/typora/202109152045328.png)

**第 4 步** ：

到这里可能面试官会问你:“`spring.factories`中这么多配置，每次启动都要全部加载么？”。

很明显，这是不现实的。我们 debug 到后面你会发现，`configurations` 的值变小了。
![第 4 步](https://cos.duktig.cn/typora/202109152046247.png)

因为，这一步有经历了一遍筛选，`@ConditionalOnXXX` 中的所有条件都满足，该类才会生效。

```java
@Configuration
// 检查相关的类：RabbitTemplate 和 Channel是否存在
// 存在才会加载
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
}
```

有兴趣的可以详细了解下 Spring Boot 提供的条件注解：

- `@ConditionalOnBean`：当容器里有指定 Bean 的条件下
- `@ConditionalOnMissingBean`：当容器里没有指定 Bean 的情况下
- `@ConditionalOnSingleCandidate`：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
- `@ConditionalOnClass`：当类路径下有指定类的条件下
- `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
- `@ConditionalOnProperty`：指定的属性是否有指定的值
- `@ConditionalOnResource`：类路径是否有指定的值
- `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
- `@ConditionalOnJava`：基于 Java 版本作为判断条件
- `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
- `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
- `@ConditionalOnWebApplication`：当前项目是 Web 项 目的条件下

## 如何实现一个Starter？

### 第一步：创建 **自定义starter** 工程`threadpool-spring-boot-starter`

**引入依赖**：

```xml
<dependencies>
    <!--    springboot启动器    -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

### 第二步：创建配置类，并注入到Spring容器中

**创建`Author`类，方便后续测试**

后续测试：另一个工程启动后，输出作者信息。

```java
public class Author {

    private String name;

    private Integer age;

    private String email;

    public Author() {
    }

    public Author(String name, Integer age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "Author{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", email='" + email + '\'' +
                '}';
    }
}
```

**创建配置`ThreadPoolAutoConfiguration`类，并注入到Spring容器中**：

```java
@Configuration
public class ThreadPoolAutoConfiguration {

    /**
     * ConditionalOnClass 需要此项目存在 ThreadPoolExecutor 类，该类为JDK自带，一定成立
     *
     * @return 线程池对象
     */
    @Bean
    @ConditionalOnClass(ThreadPoolExecutor.class)
    public ThreadPoolExecutor myThreadPool() {
        return new ThreadPoolExecutor(10, 10, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(100));
    }

    /**
     * 定义作者信息
     */
    @Bean
    @ConditionalOnClass(Author.class)
    public Author author() {
        return new Author("duktig", 23, "duktig666@163.com");
    }

}
```

### 第三步：创建`resources/META-INF/spring.factories` 文件，设置需要自动配置的类

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    cn.duktig.threadpool.config.ThreadPoolAutoConfiguration
```

### 第四步：新建工程，并引入自定义starter工程`threadpool-spring-boot-starter`的依赖

```xml
<dependencies>
    <!--    springboot启动器    -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <!--Spring boot 测试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--    自定义线程启动器   -->
    <dependency>
        <groupId>cn.duktig</groupId>
        <artifactId>threadpool-spring-boot-starter</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

启动类编写，省略

### 测试

```java
@SpringBootTest
public class MyStarterTest {

    @Autowired
    private Author author;

    @Autowired
    private ThreadPoolExecutor myThreadPool;

    @Test
    public void testGetThreadPoolInfo() {
        System.out.println("核心线程数" + myThreadPool.getCorePoolSize());
        System.out.println("阻塞队列大小" + myThreadPool.getQueue().size());
    }

    @Test
    public void testGetStarterAuthorInfo() {
        System.out.println(author);
    }

}
```

**结果**：

```java
核心线程数10
最大线程数10
```

```java
Author{name='duktig', age=23, email='duktig666@163.com'}
```

从结果上看，我们**在自定义starter的工程中，自定义的配置，在新工程中可以被自动装配**。

测试通过。

## 总结

1. Spring Boot 通过`@EnableAutoConfiguration`开启自动装配
2. 通过 SpringFactoriesLoader 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配
3. 自动配置类其实就是通过`@Conditional`按需加载的配置类
4. 想要其生效必须引入`spring-boot-starter-xxx`包实现起步依赖

## 参看：

- [淘宝一面：“说一下 Spring Boot 自动装配原理呗？” ](https://www.cnblogs.com/javaguide/p/springboot-auto-config.html) （详细分析了SpringBoot自动装配的原理）
- [SpringBoot 究竟是如何跑起来的?](https://zhuanlan.zhihu.com/p/54146400) （文章很好，分析的很调理。但缺少一些总结，跑起来的步骤是什么没有细说。）
- [SpringBoot：认认真真梳理一遍自动装配原理 ](https://zhuanlan.zhihu.com/p/95217578) 

