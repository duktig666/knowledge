作者：duktig

博客：http://duktig.cn/

> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。

# Spring基础

## 1.什么是Spring？

Spring是**一个轻量级Java开发框架**，最根本的使命是**解决企业级应用开发的复杂性，即简化Java开发**。即为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。

Spring负责基础架构，因此Java开发者可以专注于应用程序的开发。

它的成功来自于理念，而不是技术，它最为核心的理念是IoC （控制反转）和AOP （面向切面编程），其中IoC 是Spring的基础，而AOP 则是其重要的功能，最为典型的当属数据库事务的使用。

## 2.Spring的优缺点？

优点

- 方便解耦，简化开发

  Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理。

- AOP编程的支持

  Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。

- 声明式事务的支持

  只需要通过配置就可以完成对事务的管理，而无需手动编程。

- 方便程序的测试

  Spring对Junit4支持，可以通过注解方便的测试Spring程序。

- 方便集成各种优秀框架

  Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）。

- 降低JavaEE API的使用难度

  Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低。

缺点

- Spring明明一个很轻量级的框架，却给人感觉大而全
- Spring依赖反射，反射影响性能
- 使用门槛升高，入门Spring需要较长时间

# Spring IOC

## 1. 谈谈你对IOC的理解

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**

**IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value），Map 中存放的是各种对象。**

## 2.IOC实现的原理

依赖倒置

依赖倒置原则——把原本的高层建筑依赖底层建筑“倒置”过来，变成底层建筑依赖高层建筑。**高层建筑决定需要什么，底层去实现这样的需求，但是高层并不用管底层是怎么实现的**。

控制反转

就是依赖倒置原则的一种代码设计的思路。具体采用的方法就是所谓的依赖注入（Dependency Injection）。

![IOC原理](https://cos.duktig.cn/typora/202109111449502.png)

控制反转容器(IOC Container)

因为采用了依赖注入，在初始化的过程中就不可避免的会写大量的new。这里IOC容器就解决了这个问题。这个容器可以自动对你的代码进行初始化，你只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化都要亲手去写那一大段初始化的代码。

两个好处：

- 简化代码
- 无需了解细节

参看：[《Spring》IOC实现原理](https://www.jianshu.com/p/ad05cfe7868e) （依赖倒置、控制反转、依赖注入思想上分析）

## 3.IOC有什么作用（优点）？

- 管理对象的创建和依赖关系的维护（降低应用的代码量）
- 解耦，由容器维护具体对象

## 4.IOC实现机制

**工厂模式+反射机制**

示例：

```java
interface Fruit {
   public abstract void eat();
 }

class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}

class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f=null;
        try {
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```

## 5. `BeanFactory` 和 `ApplicationContext`有什么区别？

`BeanFactory`和`ApplicationContext`是Spring的两大核心接口，都可以当做Spring的容器。其中`ApplicationContext`是`BeanFactory`的子接口。

> BeanFactory 简单粗暴，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 **“低级容器”**。
>
> ApplicationContext 可以称之为 **“高级容器”**。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。

（1）依赖关系：

`BeanFactory`：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。

`ApplicationContext`接口作为`BeanFactory`的派生，除了提供`BeanFactory`所具有的功能外，还提供了更完整的框架功能：

- 继承`MessageSource`，因此支持国际化。
- 统一的资源文件访问方式。
- 提供在监听器中注册bean的事件。
- 同时加载多个配置文件。
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

（2）加载方式

**`BeanFactroy`采用的是延迟加载形式来注入Bean的**，即只有在使用到某个Bean时(调用`getBean()`)，才对该Bean进行加载实例化。

存在的问题：这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，`BeanFacotry`加载后，直至第一次使用调用`getBean`方法才会抛出异常。

**`ApplicationContext`，它是在容器启动时，一次性创建了所有的Bean**。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 `ApplicationContext`启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

**相对于基本的`BeanFactory`，`ApplicationContext` 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。**

（3）创建方式

BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

（4）注册方式

`BeanFactory`和`ApplicationContext`都支持`BeanPostProcessor`、`BeanFactoryPostProcessor`的使用，但两者之间的区别是：`BeanFactory`需要手动注册，而`ApplicationContext`则是自动注册。

![ApplicationContext UML类图](https://cos.duktig.cn/typora/202109111607037.png)

## 6. `ApplicationContext`通常的实现是什么？

1、`ClassPathXmlApplicationContext`：从 classpath 的 XML 配置文件中读取上下文，并生成
上下文定义。应用程序上下文从程序环境变量中取得。

```java
ApplicationContext context = new ClassPathXmlApplicationContext(“application.xml”);
```

2、`FileSystemXmlApplicationContext` ：由文件系统中的 XML 配置文件读取上下文。

3、`XmlWebApplicationContext`：由 Web 应用的 XML 文件读取上下文。

## 7. Spring是怎么解决循环依赖的？

首先，Spring 解决循环依赖有两个前提条件：

1. **不全是构造器方式的循环依赖**
2. **必须是单例**

基于上面的问题，我们知道Bean的生命周期，本质上解决循环依赖的问题就是三级缓存，通过三级缓存提前拿到未初始化完全的对象。

第一级缓存：用来保存实例化、初始化都完成的对象

第二级缓存：用来保存实例化完成，但是未初始化完成的对象

第三级缓存：用来保存一个对象工厂，提供一个匿名内部类，用于创建二级缓存中的对象

![三级缓存](https://cos.duktig.cn/typora/202109111624717.jpg)

假设一个简单的循环依赖场景，A、B互相依赖。

![循环依赖示例](https://cos.duktig.cn/typora/202109111624540.jpg)

A对象的创建过程：

1. 创建对象A，实例化的时候把A对象工厂放入三级缓存

![解决循环依赖流程1](https://cos.duktig.cn/typora/202109111625161.jpg)

2.A注入属性时，发现依赖B，转而去实例化B

3.同样创建对象B，注入属性时发现依赖A，依次从一级到三级缓存查询A，从三级缓存通过对象工厂拿到A，把A放入二级缓存，同时删除三级缓存中的A，此时，B已经实例化并且初始化完成，把B放入一级缓存。

![解决循环依赖流程2](https://cos.duktig.cn/typora/202109111625083.jpg)

4.接着继续创建A，顺利从一级缓存拿到实例化且初始化完成的B对象，A对象创建也完成，删除二级缓存中的A，同时把A放入一级缓存

5.最后，一级缓存中保存着实例化、初始化都完成的A、B对象

![解决循环依赖流程3](https://cos.duktig.cn/typora/202109111625001.jpg)

因此，由于把实例化和初始化的流程分开了，所以如果都是用构造器的话，就没法分离这个操作，所以都是构造器的话就无法解决循环依赖的问题了。

## Spring 启动时 循环依赖会出现什么问题？

构造器参数循环依赖

Spring容器会将每一个正在创建的 Bean 标识符放在一个**“当前创建Bean池”**中，Bean标识符在创建过程中将一直保持在这个池中，因此如果在创建Bean过程中发现自己已经在“当前创建Bean池”里时将抛出`BeanCurrentlyInCreationException`异常表示循环依赖；而对于创建完毕的Bean将从“当前创建Bean池”中清除掉。

Spring容器先创建单例StudentA，StudentA依赖StudentB，然后将A放在**“当前创建Bean池”**中，此时创建 StudentB，StudentB 依赖 StudentC ，然后将B放在**“当前创建Bean池”**中，此时创建StudentC，StudentC又依赖StudentA， 但是，此时Student已经在池中，所以会报错，因为在池中的Bean都是未初始化完的，所以会依赖错误 （初始化完的Bean会从池中移除）。

setter方式 - 单例(singleton)

Spring是先将Bean对象实例化之后再设置对象属性的

![img](https://cos.duktig.cn/typora/202112301515674.png)

**为什么用set方式就不报错了呢 ？**

我们结合上面那张图看，Spring先是用构造实例化Bean对象 ，此时Spring会将这个实例化结束的对象放到一个Map中，并且Spring提供了获取这个未设置属性的实例化对象引用的方法。 结合我们的实例来看，当Spring实例化了StudentA、StudentB、StudentC后，紧接着会去设置对象的属性，此时StudentA依赖StudentB，就会去Map中取出存在里面的单例StudentB对象，以此类推，不会出来循环的问题喽。

**为什么原型模式就报错了呢 ？**

对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。

[面试中被问Spring循环依赖的三种方式！！！ ](https://www.cnblogs.com/jajian/p/10241932.html)

## 8. 为什么要三级缓存？二级不行吗？

不可以，主要是为了生成代理对象。

因为三级缓存中放的是生成具体对象的匿名内部类，他可以生成代理对象，也可以是普通的实例对象。

使用三级缓存主要是为了**保证不管什么时候使用的都是一个对象**。

假设只有二级缓存的情况，往二级缓存中放的显示一个普通的Bean对象，`BeanPostProcessor`去生成代理对象之后，覆盖掉二级缓存中的普通Bean对象，那么**多线程环境下可能取到的对象就不一致了**。

![只有二级缓存的问题](https://cos.duktig.cn/typora/202109111629284.jpeg)

## 9.IoC 的容器构建流程

核心的构建流程如下，也就是 refresh 方法的核心内容：

![IoC 的容器构建流程](https://cos.duktig.cn/typora/202109112024701.jpeg)





# Spring AOP

## 1.谈谈你对AOP的理解

Spring AOP (Aspect-Oriented Programming:面向切面编程)基于动态代理的方式实现，如果是实现了接口的话就会使用 JDK 动态代理，反之则使用 CGLIB 代理。通过在代码的前后做一些增强处理，可以实现对业务逻辑的隔离，提高代码的模块化能力，同时也是解耦。

## 2. AOP实现的原理

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![AOP原理](https://gitee.com/koala010/typora/raw/master/img/image-20210216152837816.png)





## 3.JDK 动态代理和 CGLIB 代理有什么区别？

**（1）实现方式**

JDK 动态代理主要是针对类实现了某个接口，AOP 则会使用 JDK 动态代理。他基于反射的机制实现，生成一个实现同样接口的一个代理类，然后通过重写方法的方式，实现对代码的增强。

而如果某个类没有实现接口，AOP 则会使用 CGLIB 代理。他的底层原理是基于 ASM字节码生成框架，通过修改字节码生成成成一个子类，然后重写父类的方法，实现对代码的增强。但是因为采用的是继承，**对于final类无法进行代理**。

可以强制使用CGlib（在spring配置中加入`<aop:aspectj-autoproxy proxy-target-class="true"/>`）

**（2）速度**

jdk6之前：Cglib代理 > Jdk代理

在jdk6、jdk7、jdk8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLIB代理效率，只有当进行大量调用的时候，jdk6和jdk7比CGLIB代理效率低一点，**但是到jdk8的时候，jdk代理效率高于CGLIB代理**。

## 4. Spring AOP 和 AspectJ AOP 有什么区别？

Spring AOP 基于动态代理实现，属于**运行时增强**。

AspectJ 则属于**编译时增强**，主要有3种方式：

1. 编译时织入：指的是增强的代码和源代码我们都有，直接使用 AspectJ 编译器编译就行了，编译之后生成一个新的类，他也会作为一个正常的 Java 类装载到JVM。
2. 编译后织入：指的是代码已经被编译成 class 文件或者已经打成 jar 包，这时候要增强的话，就是编译后织入，比如你依赖了第三方的类库，又想对他增强的话，就可以通过这种方式。
3. 加载时织入：指的是在 JVM 加载类的时候进行织入。

总结下来的话，就是 **Spring AOP 只能在运行时织入，不需要单独编译，性能相比 AspectJ 编译织入的方式慢，而 AspectJ 只支持编译前后和类加载时织入，性能更好，功能更加强大**。

**如果我们的切面比较少，那么两者性能差异不大**。但是，**当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多**。

## 5. AOP 术语

AOP 领域中的特性术语：

- 通知（Advice）: AOP 框架中的增强处理。通知描述了切面何时执行以及如何执行增强处理。
- 连接点（join point）: 连接点表示应用执行过程中能够插入切面的一个点，这个点可以是方法的调用、异常的抛出。在 Spring AOP 中，连接点总是方法的调用。
- 切点（PointCut）:切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。
- 切面（Aspect）: 切面是通知和切点的结合。
- 引入（Introduction）：引入允许我们向现有的类添加新的方法或者属性。
- 织入（Weaving）: 将增强处理添加到目标对象中，并创建一个被增强的对象，这个过程就是织入。
- 目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。它通常是一个代理对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

![AOP术语](https://cos.duktig.cn/typora/202109071556064.png)

## 6. Spring通知有哪些类型？

在AOP术语中，切面的工作被称为通知，实际上是程序执行时要通过SpringAOP框架触发的代码段。

Spring切面可以应用5种类型的通知：

1. 前置通知（Before）：在目标方法被调用之前调用通知功能；
2. 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
3. 返回通知（After-returning ）：在目标方法成功执行之后调用通知；
4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
5. 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

```java
同一个aspect，不同advice的执行顺序：

①没有异常情况下的执行顺序：

around before advice
before advice
target method 执行
around after advice
after advice
afterReturning

②有异常情况下的执行顺序：

around before advice
before advice
target method 执行
around after advice
after advice
afterThrowing:异常发生
java.lang.RuntimeException: 异常发生
```

# Spring Bean

## 1. SpringBean的生命周期

**回答一**

![Bean的生命周期](https://cos.duktig.cn/typora/202109071618798.jpeg)

1. Spring 启动，查找并加载需要被 Spring 管理的 Bean，并实例化 Bean。
2. 利用依赖注入完成 Bean 中所有属性值的配置注入。
3. 如果 Bean 实现了 `BeanNameAware` 接口，则 Spring 调用 Bean 的 `setBeanName()` 方法传入当前 Bean 的 id 值。
4. 如果 Bean 实现了 `BeanFactoryAware` 接口，则 Spring 调用 `setBeanFactory()` 方法传入当前工厂实例的引用。
5. 如果 Bean 实现了 `ApplicationContextAware `接口，则 Spring 调用 `setApplicationContext()` 方法传入当前 ApplicationContext 实例的引用。
6. 如果 Bean 实现了 [`BeanPostProcessor`] 接口，则 Spring 调用该接口的预初始化方法 `postProcessBeforeInitialzation()` 对 Bean 进行加工操作，此处非常重要，**Spring 的 AOP 就是利用它实现的**。
7. 如果 Bean 实现了 `InitializingBean` 接口，则 Spring 将调用 `afterPropertiesSet()` 方法。
8. 如果在配置文件中通过 init-method 属性指定了初始化方法，则调用该初始化方法。
9. 如果 [`BeanPostProcessor` ]和 Bean 关联，则 Spring 将调用该接口的初始化方法` postProcessAfterInitialization()`。此时，Bean 已经可以被应用系统使用了。
10. 如果在 中指定了该 Bean 的作用域为 `singleton`，则将该 Bean 放入 Spring IoC 的缓存池中，触发 Spring 对该 Bean 的生命周期管理； 如果在 中指定了该 Bean 的作用域为 prototype，则将该 Bean 交给调用者，调用者管理该 Bean 的生命周期，Spring 不再管理该 Bean。
11. 如果 Bean 实现了` DisposableBean` 接口，则 Spring 会调用 `destory()` 方法销毁 Bean；如果在配置文件中通过 `destory-method` 属性指定了 Bean 的销毁方法，则 Spring 将调用该方法对 Bean 进行销毁。

**回答二**

bean 的生命周期主要有以下几个阶段，深色底的5个是比较重要的阶段：

- 实例化
- 属性填充
- 初始化
- 正常使用
- 销毁

![Bean生命周期](https://cos.duktig.cn/typora/202109111552892.jpeg)

## 2. Bean的作用域

Spring 容器在初始化一个 Bean 实例时，同时会指定该实例的作用域。Spring 5 支持以下 6 种作用域。

**singleton**

默认值，单例模式，表示在 Spring 容器中只有一个 Bean 实例，Bean 以单例的方式存在。

**prototype**

原型模式，表示每次通过 Spring 容器获取 Bean 时，容器都会创建一个 Bean 实例。

**request**

每次 HTTP 请求，容器都会创建一个 Bean 实例。该作用域只在当前 HTTP Request 内有效。

**session**

同一个 HTTP Session 共享一个 Bean 实例，不同的 Session 使用不同的 Bean 实例。该作用域仅在当前 HTTP Session 内有效。

**application**

同一个 Web 应用共享一个 Bean 实例，该作用域在当前 ServletContext 内有效。
类似于 singleton，不同的是，singleton 表示每个 IoC 容器中仅有一个 Bean 实例，而同一个 Web 应用中可能会有多个 IoC 容器，但一个 Web 应用只会有一个 ServletContext，也可以说 application 才是 Web 应用中货真价实的单例模式。

**websocket**

websocket 的作用域是 WebSocket ，即在整个 WebSocket 中有效

request、session、application、websocket 和 global Session 作用域只能在 Web 环境下使用，如果使用 ClassPathXmlApplicationContext 加载这些作用域中的任意一个的 Bean，就会抛出异常。

## 3. FactoryBean 和 BeanFactory有什么区别？

`BeanFactory` 是 Bean 的工厂， `ApplicationContext` 的父类，IOC 容器的核心，负责生产和管理 Bean 对象。最核心的功能是加载 bean，也就是 `getBean `方法，通常我们不会直接使用该接口，而是使用其子接口。

`FactoryBean` 是 Bean，可以通过实现 `FactoryBean` 接口定制实例化 Bean 的逻辑（只需要实现它的 `getObject` 方法即可），通过代理一个Bean对象，对方法前后做一些操作。

一般来说，都是通过 FactoryBean#getObject 来返回一个代理类，当我们触发调用时，会走到代理类中，从而可以在代理类中实现中间件的自定义逻辑，比如：RPC 最核心的几个功能，选址、建立连接、远程调用，还有一些自定义的监控、限流等等。

## 4. 单例bean的线程安全问题

单例 bean 存在线程安全问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。

常见的有两种解决办法：

1. 在 bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，大部分 bean 实际都是无状态（没有实例变量）的（比如 Dao、Service），这种情况下， bean 是线程安全的。

# Spring DI（依赖注入）

## 1. 什么是Spring的依赖注入？

> 所谓**依赖注入**（Dependency Injection），即组件之间的依赖关系由容器在应用系统运行期来决定，也就是由容器动态地将某种依赖关系的目标对象实例注入到应用系统中的各个关联的组件之中。

简言之，**容器在运行期动态地将有依赖关系的对象注入到关联的类中**。

## 2. `@Resource` 和 `@Autowire` 的区别

1、`@Resource` 和 `@Autowired` 都可以用来装配 bean

2、`@Autowired` 默认按类型装配，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false。

3、`@Resource` 如果指定了 name 或 type，则按指定的进行装配；如果都不指定，则优先按名称装配，当找不到与名称匹配的 bean 时才按照类型进行装配。

## 3. `@Autowire` 怎么使用名称来注入?

配合 @Qualifier 使用，如下所示：

```java
@Component
public class Test {

 @Autowired
 @Qualifier("userService")
 private UserService userService;

}
```

## 4. 什么是bean的自动装配？

在Spring框架中，在配置文件中设定bean的依赖关系是一个很好的机制，Spring 容器能够自动装配相互合作的bean，这意味着容器不需要和配置，能通过Bean工厂自动处理bean之间的协作。

这意味着 Spring可以通过向BeanFactory中注入的方式自动搞定bean之间的依赖关系。自动装配可以设置在每个bean上，也可以设定在特定的bean上。

## 5. spring 自动装配 bean 有哪些方式？

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

- no`：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。
- byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。
- byType：通过参数的数据类型进行自动装配。
- constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。
- autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。

## 6. 使用@Autowired注解自动装配的过程是怎样的？

在启动spring IoC时，容器自动装载了一个`AutowiredAnnotationBeanPostProcessor`后置处理器，当容器扫描到`@Autowied`、`@Resource`或`@Inject`时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用`@Autowired`时，首先在容器中查询对应类型的bean：

- 如果查询结果刚好为一个，就将该bean装配给`@Autowired`指定的数据；
- 如果查询的结果不止一个，那么`@Autowired`会根据名称来查找；
- 如果上述查找的结果为空，那么会抛出异常。解决方法时，使用`required=false`。



# Spring 事务

## 1. Spring事务传播机制有哪些？

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

1. **PROPAGATION_REQUIRED**：（默认值）如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务。
2. PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
3. PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
4. **PROPAGATION_REQUIRES_NEW**：创建新事务，无论当前存不存在事务，都创建新事务。
5. PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
7. PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

更多关于事务传播行为的内容请看这篇文章：[《太难了~面试官让我结合案例讲讲自己对 Spring 事务传播行为的理解。》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486668&idx=2&sn=0381e8c836442f46bdc5367170234abb&chksm=cea24307f9d5ca11c96943b3ccfa1fc70dc97dd87d9c540388581f8fe6d805ff548dff5f6b5b&token=1776990505&lang=zh_CN#rd)

## 3. `@Transactional`使用的注意事项

@Transactional 的常用配置参数

**`@Transactional` 的常用配置参数总结（只列巨额 5 个平时比较常用的）：**

| 属性名      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| propagation | 事务的传播行为，默认值为 REQUIRED，可选的值在上面介绍过      |
| isolation   | 事务的隔离级别，默认值采用 DEFAULT，可选的值在上面介绍过     |
| timeout     | 事务的超时时间，默认值为-1（不会超时）。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| readOnly    | 指定事务是否为只读事务，默认值为 false。                     |
| rollbackFor | 用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型。 |

**（1）作用范围**

1. **方法** ：推荐将注解使用于方法上，不过需要注意的是：**该注解只能应用到 public 方法上，否则不生效。**
2. **类** ：如果这个注解使用在类上的话，表明该注解对该类中所有的 public 方法都生效。
3. **接口** ：不推荐在接口上使用。

**（2）原理**

**`@Transactional` 的工作机制是基于 AOP 实现的，AOP 又是使用动态代理实现的。如果目标对象实现了接口，默认情况下会采用 JDK 的动态代理，如果目标对象没有实现了接口,会使用 CGLIB 动态代理。**

如果一个类或者一个类中的 public 方法上被标注`@Transactional` 注解的话，Spring 容器就会在启动的时候为其创建一个代理类，在调用被`@Transactional` 注解的 public 方法的时候，实际调用的是，`TransactionInterceptor` 类中的 `invoke()`方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务。

**（3）Spring AOP自调用问题**

**若同一类中的其他没有 `@Transactional` 注解的方法内部调用有 `@Transactional` 注解的方法，有`@Transactional` 注解的方法的事务会失效。**

这是由于`Spring AOP`代理的原因造成的，因为只有当 `@Transactional` 注解的方法在类以外被调用的时候，Spring 事务管理才生效。

`MyService` 类中的`method1()`调用`method2()`就会导致`method2()`的事务失效。

```java
@Service
public class MyService {

private void method1() {
     method2();
     //......
}
@Transactional
 public void method2() {
     //......
  }
}
```

解决办法就是避免同一类中自调用或者使用 AspectJ 取代 Spring AOP 代理。

**（4）异常问题**

在 `@Transactional` 注解中如果不配置`rollbackFor`属性,那么事务只会在遇到`RuntimeException`的时候才会回滚，加上 `rollbackFor=Exception.class`,可以让事务在遇到非运行时异常时也回滚。

**（5）事务的超时属性**

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。

**（6）事务的只读属性**

对于只有读取数据查询的事务，可以指定事务类型为 readonly，即只读事务。只读事务不涉及数据的修改，数据库会提供一些优化手段，适合用在有多条数据库查询操作的方法中。

拿 MySQL 的 innodb 举例子，根据官网 https://dev.mysql.com/doc/refman/5.7/en/innodb-autocommit-commit-rollback.html 描述：

> MySQL 默认对每一个新建立的连接都启用了`autocommit`模式。在该模式下，每一个发送到 MySQL 服务器的`sql`语句都会在一个单独的事务中进行处理，执行结束后会自动提交事务，并开启一个新的事务。

但是，如果你给方法加上了`Transactional`注解的话，这个方法执行的所有`sql`会被放在一个事务中。如果声明了只读事务的话，数据库就会去优化它的执行，并不会带来其他的什么收益。

如果不加`Transactional`，每条`sql`会开启一个单独的事务，中间被其它事务改了数据，都会实时读取到最新值。

分享一下关于事务只读属性，其他人的解答：

1. 如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持 SQL 执行期间的读一致性；
2. 如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询 SQL 必须保证整体的读一致性，否则，在前条 SQL 查询之后，后条 SQL 查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持

**（7）总结**

1. `@Transactional` 注解只有作用到 public 方法上事务才生效，不推荐在接口上使用；
2. 避免同一个类中调用 `@Transactional` 注解的方法，这样会导致事务失效；
3. 正确的设置 `@Transactional` 的 rollbackFor 和 `propagation` 属性，否则事务可能会回滚失败
4. 如果需要事务遇到非运行时异常也回滚，需要添加`rollbackFor=Exception.class`。（默认遇到运行时异常回滚）
5. 多条sql查询，需要保证整体读一致性，可添加只读事务。

# 其他

## 1. Spring 里用到了哪些设计模式?

- **工厂模式** : Spring 使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理模式** : Spring AOP 功能的实现。
- **单例模式** : Spring 中的 Bean 默认都是单例的。
- **模板模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** : Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

参看：[面试官:“谈谈Spring中都用到了那些设计模式?”](

## 2. Spring框架中有哪些不同类型的事件?

Spring 提供了以下5种标准的事件：

1. 上下文更新事件（`ContextRefreshedEvent`）：在调用`ConfigurableApplicationContext` 接口中的`refresh()`方法时被触发。
2. 上下文开始事件（`ContextStartedEvent`）：当容器调用`ConfigurableApplicationContext`的`Start()`方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（`ContextStoppedEvent`）：当容器调用`ConfigurableApplicationContext`的`Stop()`方法停止容器时触发该事件。
4. 上下文关闭事件（`ContextClosedEvent`）：当`ApplicationContext`被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
5. 请求处理事件（`RequestHandledEvent`）：在Web应用中，当一个http请求（request）结束触发该事件。如果一个bean实现了`ApplicationListener`接口，当一个`ApplicationEvent` 被发布以后，bean会自动被通知。

## 3. Spring注解

### `@Qualifier`注解有什么作用

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用`@Qualifier` 注解和 `@Autowired` 通过指定应该装配哪个确切的 bean 来消除歧义。

### `@Required` 注解有什么作用

这个注解表明bean的属性必须在配置的时候设置，通过一个bean定义的显式的属性值或通过自动装配，若`@Required`注解的bean属性未被设置，容器将抛出`BeanInitializationException`。示例：

```java
public class Employee {
    private String name;
    @Required
    public void setName(String name){
        this.name=name;
    }
    public string getName(){
        return name;
    }
}
```

### `@Component`, `@Controller`, `@Repository`, `@Service` 有何区别？

`@Component`：这将 java 类标记为 bean。**它是任何 Spring 管理组件的通用构造型**。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。

`@Controller`：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。

`@Service`：此注解是组件注解的特化。它不会对 `@Component` 注解提供任何其他行为。您可以在服务层类中使用 `@Service` 而不是 `@Componen`t，因为**它以更好的方式指定了意图**。

`@Repository`：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

# 参考

- [【spring大总结】希望所有看完这篇文章的C友，都能快速入门spring](https://blog.csdn.net/weixin_55932383/article/details/120088647?utm_source=app&app_version=4.14.0&code=app_1562916241&uLinkId=usr1mkqgl919blen)
- [Spring面试题（2021最新版）](https://zhuanlan.zhihu.com/p/369115360)
- [JavaGuide-Spring常见问题总结](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/Spring%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93)
- [面试题系列：Spring 夺命连环10问](https://zhuanlan.zhihu.com/p/368769721) （面试题的含金量比较高）
- [面试必问的 Spring，你懂了吗？](https://zhuanlan.zhihu.com/p/311373740)
- [《Spring》IOC实现原理](https://www.jianshu.com/p/ad05cfe7868e) （依赖倒置、控制反转、依赖注入思想上分析）
- [什么才叫懂Spring底层原理，这些面试题你都会吗](https://zhuanlan.zhihu.com/p/38484238)











