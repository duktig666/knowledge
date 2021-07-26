# 疑问

## Spring Boot 之 spring.factories

在Spring中也有一种类似与Java SPI的加载机制。它在META-INF/spring.factories文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。
这种自定义的SPI机制是Spring Boot Starter实现的基础。

参看：

- [Spring Boot 之 spring.factories](https://www.cnblogs.com/huanghzm/p/12217630.html) 
- [Spring Boot的扩展机制之Spring Factories](https://blog.csdn.net/lvoyee/article/details/82017057)

