# SpringBoot基础

## 什么是SpringBoot？

SpringBoot主要用来简化使用Spring的难度和繁重的XML配置，它是**Spring组件的一站式解决方案**，采取了**约定优于配置**的方法。

## SpringBoot的优点

- 可以创建独立的 Spring 应用程序，并且基于其Maven或Gradle插件，可以创建可执行的JARs和WARs；
- 简化配置，开箱即用，尽可能自动配置Spring和第三方依赖库
- 直接嵌入Tmocat、Jetty等Servlet服务器
- 创建独立的Spring程序
- 提供一系列监控功能，例如指标、运行状况等的监控
- 完全不需要代码生成和XML配置

## Spring MVC 、Spring 、Spring Boot 和 Spring Cloud 有什么区别和联系？

Spring是**一个轻量级Java开发框架**，最根本的使命是**解决企业级应用开发的复杂性，即简化Java开发**。即为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。

SpringMVC是Spring基础之上的一个MVC框架，主要处理web开发的路径映射和视图渲染，属于Spring框架中WEB层开发的一部分。

Spring boot，约定优于配置，简化了spring的配置流程，是**Spring组件的一站式解决方案**。

Spring Cloud构建于Spring Boot之上，是一个关注全局的服务治理框架。

### SpringBoot开发 和 传统的 SpringMVC开发 有什么不同？

- SpringBoot简化配置，开箱即用，提升了开发效率
- SpringBoot内嵌Tomcat、Jetty等Web服务器，可独立运行；无需打war包在Tomcat服务器上运行
- SpringBoot可以快速整合第三方框架，约定大于配置，简化开发流程
- SpringBoot提供了一些列的监控功能

### SpringCloud开发相对于SpringBoot开发有什么优势和不足？

优势：

- **服务的独立部署**。每个服务都是一个独立的项目，可以独立部署，不依赖于其他服务，耦合性低。
- **服务的快速启动**。拆分之后服务启动的速度必然要比拆分之前快很多，因为依赖的库少了，代码量也少了。
- **更加适合敏捷开发**。服务拆分可以快速发布新版本，修改哪个服务只需要发布对应的服务即可，不用整体重新发布。
- **职责专一，由专门的团队负责专门的服务**。每个团队可以负责对应的业务线，服务的拆分有利于团队之间的分工。
- **服务可以动态按需扩容**。当某个服务的访问量较大时，我们只需要将这个服务扩容即可。
- **代码的复用**。每个服务都提供 REST API，所有的基础服务都必须抽出来，很多的底层实现都可以以接口方式提供。

缺点：

- 分布式部署，调用的复杂性高
- 独立的数据库，分布式事务的挑战
- 测试的难度提升
- 运维难度的提升

联系

SpringCloud大部分的功能插件都是基于SpringBoot去实现的，SpringCloud关注于全局的微服务整合和管理，将多个SpringBoot单体微服务进行整合以及管理；SpringCloud依赖于SpringBoot开发，而SpringBoot可以独立开发；

### 总结

- Spring是核心，提供了基础功能；
- Spring MVC 是基于Spring的一个 MVC 框架 ；
- Spring Boot 是为简化Spring配置的快速开发框架；
- Spring Cloud是构建在Spring Boot之上的服务治理框架。

## 开启SpringBoot 特性有哪几种方式？

#### 1. 继承spring-boot-starter-parent项目

```xml
<parent>    
	<groupId>org.springframework.boot</groupId>    
	<artifactId>spring-boot-starter-parent</artifactId>    
	<version>1.5.6.RELEASE</version>
</parent>
```

#### 2. 导入spring-boot-dependencies项目依赖

```xml
<dependencyManagement>    
    <dependencies>        
        <dependency>            
        <groupId>org.springframework.boot</groupId>            
        <artifactId>spring-boot-dependencies</artifactId>            
        <version>1.5.6.RELEASE</version>            
        <type>pom</type>            
        <scope>import</scope>        
    </dependency>
</dependencyManagement>
```



### `bootstrap.properties`和`application.properties` 有何区别 ?	

SpringBoot两个核心的配置文件：

- **bootstrap**(.yml 或者 .properties)：`boostrap` 由`父 ApplicationContext` 加载的，**比`applicaton`优先加载**，配置在应用程序上下文的引导阶段生效。一般来说我们在 SpringCloud Config 或者Nacos中会用到它。且**boostrap里面的属性不能被覆盖**；
- **application** (.yml或者.properties)：由`ApplicatonContext` 加载，用于 **SpringBoot项目的自动化配置**。

##  SpringBoot的核心注解是什么？由那些注解组成？

启动类上@SpringBootApplication是 SpringBoot 的核心注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

主要组合包含了以下 3 个注解：

- @**SpringBootConfiguration**：
  组合了 @Configuration 注解，实现配置文件的功能。
- @**EnableAutoConfiguration**：
  打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。
- @**ComponentScan**：
  Spring组件扫描。

## SpringBoot 中的监视器是什么?

**Spring boot actuator**是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTPURL访问的REST端点来检查状态。

## SpringBoot 打成的jar和普通的jar有什么区别 ?

SpringBoot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 `java -jar xxx.jar` 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。

SpringBoot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 `\BOOT-INF\classes` 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

## Spring Boot启动的时候如何运行一些特定的代码？

如果你想在Spring Boot启动的时候运行一些特定的代码，你可以实现接口 `ApplicationRunner`或者 `CommandLineRunner`，这两个接口实现方式一样，它们都只提供了一个run方法。

**CommandLineRunner**：启动获取命令行参数。

**ApplicationRunner**：启动获取应用启动的时候参数。

如果启动的时候有多个ApplicationRunner和CommandLineRunner，想控制它们的启动顺序，可以实现 `org.springframework.core.Ordered`接口或者使用 `org.springframework.core.annotation.Order`注解。

# SpringBoot原理

## SpringBoot 启动流程

1. 准备环境，根据不同的环境创建不同的Environment
2. 准备、加载上下文，为不同的环境选择不同的Spring Context，然后加载资源，配置Bean
3. 初始化，这个阶段刷新Spring Context，启动应用
4. 最后结束流程

## SpringBoot的自动装配原理（启动时都做了什么）？Starter如何实现？

> SpringBoot 定义了一套接口规范，这套规范规定：**SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器**（此处涉及到 JVM 类加载机制与 Spring 的容器知识），**并执行类中定义的各种操作**。对于外部 jar 来说，只需要按照 SpringBoot 定义的标准，就能将自己的功能装置进 SpringBoot。

1. Spring Boot 通过`@EnableAutoConfiguration`开启自动装配
2. 通过 SpringFactoriesLoader 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配
   1. 整个J2EE的整体解决方案和自动配置都在`springboot-autoconfigure`的jar包中
   2. **它将所有需要导入的组件以全类名的方式返回 ， 这些组件就会被添加到容器中 ；**
   3. 它会给容器中导入非常多的自动配置类 （`xxxAutoConfiguration`）, 就是给容器中导入这个场景需要的所有组件 ， 并配置好这些组件 
3. 通过`@Conditional`注解按需加载的配置类
4. 想要其生效必须引入`spring-boot-starter-xxx`包实现起步依赖

![SpringBoot自动配置原理 总结](https://cos.duktig.cn/typora/202109161008646.jpeg)

详情参看：[SpringBoot自动装配原理.md](./SpringBoot自动装配原理.md) 



