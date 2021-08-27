## 

Bean的生命周期

### 创建

class -> 实例化 -> 对象（刚new出来的对象） -> 属性填充 -> 初始化`afterPropertiesSet()` -> AOP -> Bean对象（Aop后可能不是同一个对象）





## Spring Bean 容器

#### 是什么？

Spring 包含并管理应用对象的配置和生命周期，在这个意义上它是一种用于承载对象的容器，你可以配置你的每个 Bean 对象是如何被创建的，这些 Bean 可以创建一个单独的实例或者每次需要时都生成一个新的实例，以及它们是如何相互关联构建和使用的。

如果一个 Bean 对象交给 Spring 容器管理，那么这个 Bean 对象就应该以类似零件的方式被拆解后存放到 Bean 的定义中，这样相当于一种把对象解耦的操作，可以由 Spring 更加容易的管理，就像处理循环依赖等操作。

当一个 Bean 对象被定义存放以后，再由 Spring 统一进行装配，这个过程包括 Bean 的初始化、属性填充等，最终我们就可以完整的使用一个 Bean 实例化后的对象了。

#### 数据结构

Bean容器需要一种可以用于存放和名称索引式的数据结构，所以选择 HashMap 是最合适不过的。

> HashMap 是一种基于扰动函数、负载因子、红黑树转换等技术内容，形成的拉链寻址的数据结构，它能让数据更加散列的分布在哈希桶以及碰撞时形成的链表和红黑树上。它的数据结构会尽可能最大限度的让整个数据读取的复杂度在 O(1) ~ O(Logn) ~O(n)之间，当然在极端情况下也会有 O(n) 链表查找数据较多的情况。不过我们经过10万数据的扰动函数再寻址验证测试，数据会均匀的散列在各个哈希桶索引上，所以 HashMap 非常适合用在 Spring Bean 的容器实现上。

####  Bean 的定义、注册、获取

![Bean 的定义、注册、获取](https://cos.duktig.cn/typora/202108241416774.png)





- 定义：BeanDefinition，可能这是你在查阅 Spring 源码时经常看到的一个类，例如它会包括 singleton、prototype、BeanClassName 等。但目前我们初步实现会更加简单的处理，只定义一个 Object 类型用于存放对象。
- 注册：这个过程就相当于我们把数据存放到 HashMap 中，只不过现在 HashMap 存放的是定义了的 Bean 的对象信息。
- 获取：最后就是获取对象，Bean 的名字就是key，Spring 容器初始化好 Bean 以后，就可以直接获取了。

![Bean 的定义、注册、获取](https://cos.duktig.cn/typora/202108241526124.png)

- 首先我们需要定义 BeanFactory 这样一个 Bean 工厂，提供 Bean 的获取方法 `getBean(String name)`，之后这个 Bean 工厂接口由抽象类 AbstractBeanFactory 实现。这样使用[模板模式](https://bugstack.cn/itstack-demo-design/2020/07/07/重学-Java-设计模式-实战模板模式.html)的设计方式，可以统一收口通用核心方法的调用逻辑和标准定义，也就很好的控制了后续的实现者不用关心调用逻辑，按照统一方式执行。那么类的继承者只需要关心具体方法的逻辑实现即可。
- 那么在继承抽象类 AbstractBeanFactory 后的 AbstractAutowireCapableBeanFactory 就可以实现相应的抽象方法了，因为 AbstractAutowireCapableBeanFactory 本身也是一个抽象类，所以它只会实现属于自己的抽象方法，其他抽象方法由继承 AbstractAutowireCapableBeanFactory 的类实现。这里就体现了类实现过程中的各司其职，你只需要关心属于你的内容，不是你的内容，不要参与。*这一部分内容我们会在代码里有具体的体现*
- 另外这里还有块非常重要的知识点，就是关于单例 SingletonBeanRegistry 的接口定义实现，而 DefaultSingletonBeanRegistry 对接口实现后，会被抽象类 AbstractBeanFactory 继承。现在 AbstractBeanFactory 就是一个非常完整且强大的抽象类了，也能非常好的体现出它对模板模式的抽象定义。





![Bean的依赖关系](https://cos.duktig.cn/typora/202108241538591.png)





- `BeanFactory` 的定义由 `AbstractBeanFactory` 抽象类实现接口的 `getBean` 方法
- 而 `AbstractBeanFactory` 又继承了实现了 `SingletonBeanRegistry` 的`DefaultSingletonBeanRegistry` 类。这样 **`AbstractBeanFactory` 抽象类就具备了单例 Bean 的注册功能**。
- `AbstractBeanFactory` 中又定义了两个抽象方法：`getBeanDefinition(String beanName)`、`createBean(String beanName, BeanDefinition beanDefinition)` ，而这两个抽象方法分别由 `DefaultListableBeanFactory`、`AbstractAutowireCapableBeanFactory` 实现。
- 最终 `DefaultListableBeanFactory` 还会继承抽象类 `AbstractAutowireCapableBeanFactory` 也就可以调用抽象类中的 `createBean` 方法了。





考虑构造器有参数实例化Bean的情况

![构造器实例化bean的选择](https://cos.duktig.cn/typora/202108241526686.png)

- 参考 Spring Bean 容器源码的实现方式，在 BeanFactory 中添加 `Object getBean(String name, Object... args)` 接口，这样就可以在获取 Bean 时把构造函数的入参信息传递进去了。
- 另外一个核心的内容是使用什么方式来创建含有构造函数的 Bean 对象呢？这里有两种方式可以选择，一个是基于 Java 本身自带的方法 `DeclaredConstructor`，另外一个是使用 Cglib 来动态创建 Bean 对象。*Cglib 是基于字节码框架 ASM 实现，所以你也可以直接通过 ASM 操作指令码来创建对象*



![Spring Bean 容器类关系](https://cos.duktig.cn/typora/202108241537576.png)

 InstantiationStrategy 实例化策略接口，以及补充相应的 getBean 入参信息，让外部调用时可以传递构造函数的入参并顺利实例化。

- **SimpleInstantiationStrategy** JDK实例化
- **CglibSubclassingInstantiationStrategy** CGLIB实例化



属性填充

![image-20210824155936499](https://cos.duktig.cn/typora/202108241559266.png)



- 属性填充要在类实例化创建之后，也就是需要在 `AbstractAutowireCapableBeanFactory` 的 createBean 方法中添加 `applyPropertyValues` 操作。
- 由于我们需要在创建Bean时候填充属性操作，那么就需要在 bean 定义 BeanDefinition 类中，添加 PropertyValues 信息。
- 另外是填充属性信息还包括了 Bean 的对象类型，也就是需要再定义一个 BeanReference，里面其实就是一个简单的 Bean 名称，在具体的实例化操作时进行递归创建和填充。

![属性填充](https://cos.duktig.cn/typora/202108241602086.png)

- `BeanReference`(类引用)、`PropertyValue`(属性值)、`PropertyValues`(属性集合)，分别用于类和其他类型属性填充操作。
- 另外改动的类主要是 `AbstractAutowireCapableBeanFactory`，在 createBean 中补全属性填充部分。



读取、解析、注册Bean

![image-20210824161739815](https://cos.duktig.cn/typora/202108241617064.png)



- 资源加载器属于相对独立的部分，它位于 Spring 框架核心包下的IO实现内容，主要用于处理Class、本地和云环境中的文件信息。
- 当资源可以加载后，接下来就是解析和注册 Bean 到 Spring 中的操作，这部分实现需要和 DefaultListableBeanFactory 核心类结合起来，因为你所有的解析后的注册动作，都会把 Bean 定义信息放入到这个类中。
- 那么在实现的时候就设计好接口的实现层级关系，包括我们需要定义出 Bean 定义的读取接口 `BeanDefinitionReader` 以及做好对应的实现类，在实现类中完成对 Bean 对象的解析和注册。



![资源加载](https://cos.duktig.cn/typora/202108241632538.png)

- BeanFactory，已经存在的 Bean 工厂接口用于获取 Bean 对象，这次新增加了按照类型获取 Bean 的方法：`<T> T getBean(String name, Class<T> requiredType)`
- ListableBeanFactory，是一个扩展 Bean 工厂接口的接口，新增加了 `getBeansOfType`、`getBeanDefinitionNames()` 方法，在 Spring 源码中还有其他扩展方法。
- HierarchicalBeanFactory，在 Spring 源码中它提供了可以获取父类 BeanFactory 方法，属于是一种扩展工厂的层次子接口。*Sub-interface implemented by bean factories that can be part of a hierarchy.*
- AutowireCapableBeanFactory，是一个自动化处理Bean工厂配置的接口，目前案例工程中还没有做相应的实现，后续逐步完善。
- ConfigurableBeanFactory，可获取 BeanPostProcessor、BeanClassLoader等的一个配置化接口。
- ConfigurableListableBeanFactory，提供分析和修改Bean以及预先实例化的操作接口，不过目前只有一个 getBeanDefinition 方法。







![spring-7-03](https://cos.duktig.cn/typora/202108242148355.png)



![image-20210824220841641](https://cos.duktig.cn/typora/202108242208856.png)



![image-20210825081728869](https://cos.duktig.cn/typora/202108250817714.png)

- 整个的实现过程包括了两部分，一个解决单例还是原型对象，另外一个处理 FactoryBean 类型对象创建过程中关于获取具体调用对象的 `getObject` 操作。
- `SCOPE_SINGLETON`、`SCOPE_PROTOTYPE`，对象类型的创建获取方式，主要区分在于 `AbstractAutowireCapableBeanFactory#createBean` 创建完成对象后是否放入到内存中，如果不放入则每次获取都会重新创建。
- createBean 执行对象创建、属性填充、依赖加载、前置后置处理、初始化等操作后，就要开始做执行判断整个对象是否是一个 FactoryBean 对象，如果是这样的对象，就需要再继续执行获取 FactoryBean 具体对象中的 `getObject` 对象了。整个 getBean 过程中都会新增一个单例类型的判断`factory.isSingleton()`，用于决定是否使用内存存放对象信息。



