# 前瞻

## 真实面试题1：什么是微服务？（说说你对微服务的理解）













































## 真实面试题2：SpringMVC、SpringBoot、SpringCloud的联系和区别？

















































## 真实面试题3：SpringCloud的优势和缺点？

















































## 架构演变史















































### 思考+讨论1：Dubbo属于哪一个架构？



















































## 思考+讨论2：关于射击项目中对Gateway使用的思考——使用SpringBoot+Dubbo+zookeeper架构时，有必要使用Gateway吗？

当时面试官问，你们项目使用gaetway使用的Dubbo那一套、SpringCloud、SpringCloudAlibaba



















































## 真实面试题+思考+讨论3：模块间调用是随便调的还是有一定的规则？

业务互相支持，简单

业务链  业务隔离

设计模式













































## 经验总结——学习一样技术首先要掌握哪些？

从这些面试中总结了一些规律，很多面试时会问“说说你对xxx的理解/了解吧？”你该怎样回答？

所以学习一项技术，首先知道它是什么？解决了什么问题？有什么优缺点？比较重要



提问：回顾上一次分享——Gateway它是什么？解决了什么问题？有什么优缺点？与同等技术相比有什么不同？



> 本项目提供了一个构建在 Spring 生态系统之上的 API 网关，包括：Spring 5、Spring Boot 2 和 Project Reactor。Spring Cloud Gateway 旨在提供一种简单而有效的方式来路由到 API 并为它们提供交叉关注点，例如：安全性、监控/指标和弹性。









也不一定是死记概念。回答出自己的理解。











# 动态路由

## 什么是动态路由？

> 动态路由是与静态路由相对的一个概念，指路由器能够根据路由器之间的交换的特定路由信息自动地建立自己的路由表，并且能够根据链路和节点的变化适时地进行自动调整。当网络中节点或节点间的链路发生故障，或存在其它可用路由时，动态路由可以自行选择最佳的可用路由并继续转发报文。
>
> ——摘自百度百科

动态路由机制的运作依赖路由器的两个基本功能：对路由表的维护；路由器之间适时的路由信息交换。

## 为什么要有动态路由？（解决了什么样的问题）

静态路由和动态路由 http/lb



## 动态路由的实现

Gateway已经默认提供来动态路由策略，即使用 lb 协议进行访问。

Gateway从注册中心读取可用的服务列表，然后采用自己的过滤拦截策略和负载均衡策略进行访问。

Gateway使用SpringCloudLoadBan

![image-20210803172503858](https://gitee.com/koala010/typora/raw/master/img/20210803172503.png)

但是问题是我们的配置写在yml中写死，如果我们对路由的配置需要根据线上环境进行动态配置该怎么办？

[Spring Cloud Gateway动态路由实现](https://www.jianshu.com/p/8f007bcf36ea) （Gateway上线部署分析讲的不错）

### 基于Redis实现

#### 演示

#### 测试用例

http://127.0.0.1:9001/gateway/add

```json
{
   "id":"freshman-modules-admin",
   "predicates":[
      {
         "name":"Path",
         "args":{
            "_genkey_0":"/api/admin/**"
         }
      }
   ],
   "filters":[],
   "uri":"lb://freshman-modules-admin",
   "order":0
}
```



### 基于Nacos分布式配置中心实现

#### 演示

#### 对比这两种方式





# 灰度发布

## 思考4：我们现在采用的部署方案存在哪些问题？

停机重新部署

只能全量发布，灰度发布只能使用代码实现



## 线上项目发布一般方案

1. 停机发布
2. 蓝绿部署
3. 滚动部署
4. 灰度发布

**停机发布** 这种发布一般在夜里或者进行大版本升级的时候发布，因为需要停机，所以现在大家都在研究 `Devops` 方案。

**蓝绿部署** 需要准备两个相同的环境。一个环境新版本，一个环境旧版本，通过负载均衡进行切换与回滚，目的是为了减少服务停止时间。

**滚动部署** 就是在升级过程中，并不一下子启动所有新版本，是先启动一台新版本，再停止一台老版本，然后再启动一台新版本，再停止一台老版本，直到升级完成。基于 `k8s` 的升级方案默认就是滚动部署。

**灰度发布** 也叫金丝雀发布，灰度发布中，常常按照用户设置路由权重，例如90%的用户维持使用老版本，10%的用户尝鲜新版本。不同版本应用共存，经常与A/B测试一起使用，用于测试选择多种方案。

上边介绍的几种发布方案，主要是引出我们接下来介绍的 `spring-cloud-gateway` 动态路由，我们可以基于动态路由、负载均衡和策略加载去实现 `灰度发布`。当然现在有很多开源的框架可以实现 `灰度发布`，这里只是研究学习。

## 灰度发布简介

> 灰度发布（又名金丝雀发布）是指在黑与白之间，能够平滑过渡的一种发布方式。在其上可以进行A/B testing，即让一部分用户继续用产品特性A，一部分用户开始用产品特性B，如果用户对B没有什么反对意见，那么逐步扩大范围，把所有用户都迁移到B上面来。灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。
>
> 灰度期：灰度发布开始到结束期间的这一段时间，称为灰度期。
>
> ——摘自百度百科

![灰度发布流程](https://gitee.com/koala010/typora/raw/master/img/20210803091012.png)



我们来简单的通过下图来看下，通俗的讲： 为了保证服务升级过程的平滑过渡提高客户体验，会一部分用户 一部分用户递进更新，这样生产中会同时出现多个版本的客户端，为了保证多个版本客户端的可用需要对应的多个版本的服务端版本。灰度发布就是通过一定策略保证 多个版本客户端、服务端间能够正确对应。

![灰度发布-按版本访问](https://gitee.com/koala010/typora/raw/master/img/20210802223021.png)

以下方式基于权重进行访问

![灰度发布-按权重访问](https://gitee.com/koala010/typora/raw/master/img/20210803090817.png)



### 代码演示

#### 基于version实现

#### 基于weight+version实现

测试权重选择算法



### 灰度发布的其他方案

#### nginx + lua (OpenResty)

![image-20210803160923182](https://gitee.com/koala010/typora/raw/master/img/20210803160923.png)

#### Redis+OpenResty

![image-20210803163123533](https://gitee.com/koala010/typora/raw/master/img/20210803163123.png)

[OpenResty中文官方网站](http://openresty.org/cn/)



#### [Discovery](https://github.com/Nepxion/Discovery)







## 分享一种微服务的架构思路（模块划分）

微服务的思想中有一些关键点：

- 细粒度小，甚至是单一职责
- 每个服务可以独立部署和升级
- 服务围绕业务功能拆分

所以细粒度是我们首要的思考。——对比smpe的Common模块

远程调用接口抽离

参看ruoyi架构



# 搭建微服务项目遇到的问题

## 1.抽离远程调用接口后，引入openfeign报错，找不到

报错详情：

```
Description:

Parameter 1 of constructor in cn.duktig.freshman.modules.praise.service.impl.StudentPraiseServiceImpl required a bean of type 'cn.duktig.freshman.api.student.feign.RemoteStudentsService' that could not be found.

Action:

Consider defining a bean of type 'cn.duktig.freshman.api.student.feign.RemoteStudentsService' in your configuration.
```

解决：

```java
@EnableFeignClients(basePackages = {
        "cn.duktig.freshman.api.student.feign",
        "cn.duktig.freshman.api.wx.feign"
})
```



## 2.引入openfeign，报错没有spring-cloud-starter-loadbalancer？

报错详情：

```java
Unexpected exception during bean creation; nested exception is java.lang.IllegalStateException: No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-loadbalancer?
```

解决：

```java
<!-- SpringCloud Loadbalancer -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>

        <!-- SpringCloud Alibaba Nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

https://blog.csdn.net/weixin_43556636/article/details/110653989

https://www.cnblogs.com/liaozk/p/14920650.html





## 3.springcloud 2020 gateway 503 错误代码 不能使用lb协议进行访问，但是可以使用http访问

也是spring-cloud-starter-loadbalancer的问题？参看问题2解决方案



## 4.gateway 引入 spring-boot-starter-web 报错？

报错详情：

```
Description:

Spring MVC found on classpath, which is incompatible with Spring Cloud Gateway.

Action:

Please set spring.main.web-application-type=reactive or remove spring-boot-starter-web dependency.
```



![image-20210803170018590](https://gitee.com/koala010/typora/raw/master/img/20210803170018.png)

## 5.分布式配置中心采用非默认的命名空间，不能直接获取服务的配置信息







## 参考

- [Gateway官网](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/#cors-configuration)
- [Gateway--服务网关](https://cloud.tencent.com/developer/article/1785090?from=article.detail.1620795) （比较详细）
- [Spring Cloud Gateway动态路由实现](https://www.jianshu.com/p/8f007bcf36ea) （Gateway上线部署分析讲的不错）
- [spring-cloud-gateway动态路由](https://zhuanlan.zhihu.com/p/125018436) （动态路由的实现）
- [Spring Cloud gateway 网关四 动态路由原理手把手带你飞](https://blog.csdn.net/qq_35809876/article/details/102985117) （动态路由的原理讲解）
- [gateway-nacosconfig动态配置路由原理详解](https://blog.csdn.net/he702170585/article/details/107342930) （再结合nacos config讲原理）
- [基于springcloud gateway + nacos实现灰度发布（reactive版）](https://blog.csdn.net/kingwinstar/article/details/105752725)     [源码](https://github.com/lyb-geek/gateway/blob/master/gateway-reactor-gray/src/main/java/com/github/lybgeek/loadbalancer/GrayLoadBalancer.java)
- [Spring Cloud Gateway 扩展支持多版本控制及灰度发布](https://cloud.tencent.com/developer/article/1458200)
- [SpringCloud LoadBalancer灰度策略实现](https://www.cnblogs.com/leng-leng/p/14280698.html)
- [网关 Spring Cloud Gateway 实战负载均衡(Spring Cloud Loadbalancer)](https://blog.csdn.net/abu935009066/article/details/112296435)
- [OpenResty中文官方网站](http://openresty.org/cn/)
- [Discovery](https://github.com/Nepxion/Discovery)
- [springcloud 2020 gateway 503 错误代码](https://www.cnblogs.com/sjingser1/p/14893231.html)
- [外行人都能看得懂的WebFlux，错过了血亏！](https://zhuanlan.zhihu.com/p/92460075)
- [Spring Cloud之Ribbon负载均衡（Spring Cloud 2020.0.3版）](https://www.cnblogs.com/seanRay/p/14781110.html)
- 



















