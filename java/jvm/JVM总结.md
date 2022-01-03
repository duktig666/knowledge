# JVM运行时数据区

![运行时数据区内存结构图](https://gitee.com/koala010/typora/raw/master/img/20210621153304.png)



![JVM内存结构](https://gitee.com/koala010/typora/raw/master/img/20210810171112.png)



线程共享方法区和堆，独占虚拟机栈、本地方法栈和程序计数器。

## 程序计数器

程序计数器是一块较小的内存空间，可以看做当前线程执行的字节码的行号指示器。

**作用：**

1. **字节码解释器通过改变程序计数器来依次读取指令，从而实现流程控制**。如：顺序选择、选择、循环、异常处理。
2. **在多线程的情况下，程序计数器记录当前线程的执行位置，以便线程切换回来可以得知上次的执行位置**。（问题：线程为什么独占程序计数器的答案）

**注意：**

- **程序计数器是唯一一个在《Java虚拟机规范》中没有规定任何`OutOfMemoryError`情况的区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**
- 如果正在执行的是Java方法，记录的是正在执行的虚拟机字节码指令地址；如果正在执行的是**本地（Native）方法**，这个**计数器值则应为空（`Undefined`）**。

## 虚拟机栈

描述的是Java方法执行的线程内存模型，每个方法会创建一个**栈帧**，栈帧中存放**局部变量表、操作数栈、动态链接、方法出口**等信息。

**方法/函数如何调用？**

Java 栈可用类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java 方法有两种返回方式：

1. return 语句。
2. 抛出异常。

不管哪种返回方式都会导致栈帧被弹出。

每个方法被调用直至执行完毕的过程，对应一个栈帧在虚拟机栈入栈和出栈的过程。

*（栈内存一般指局部变量表）*

### **局部变量表**：

局部变量表存放了编译期中各种的**基本数据类型**、**对象引用**（并不是对象本身，可能是指向对象起始地址的一个引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置信息）和**returnAddress类型**（指向一条字节码指令的地址）。

- 64位的long和double类型的数据会占用2个局部变量空间，其余的数据类型只占用1个
- 局部变量表所需内存编译期完成分配，进入一个方法后，栈分配多少内存是固定的，运行期间不会改变。

OOM

《Java虚拟机规范》规定，

1）如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出`StackOverflowError`异常。
2）如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出`OutOfMemoryError`异常。

但是HotSpot虚拟机的选择是不支持扩展，所以除非在创建线程申请内存时就因无法获得足够内存而出现
`OutOfMemoryError`异常，否则在线程运行时是不会因为扩展而导致内存溢出的，只会因为栈容量无法容纳新的栈帧而导致`StackOverflowError`异常。



## 本地方法栈

- 和虚拟机栈类似，两者的区别就是**虚拟机栈是为虚拟机执行java方法服务**，**本地方法栈为虚拟机执行native方法服务**。

- **HotSpot虚拟机不区分虚拟机栈和本地方法栈**（合二为一）。

## 堆

- Java 虚拟机所管理的内存中最大的一块，Java 堆是**所有线程共享的一块内存区域**，在虚拟机启动时创建。

- **此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

- 堆是垃圾收集器管理的主要区域，因此也被称为“GC堆”

- JAVA堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可。
- 可通过参数 -Xmx -Xms 来指定运行时堆内存的大小，在Java堆中没有内存完成实例分配，并且也无法扩展时，会抛`OutOfMemoryError`异常。

## 方法区

- 方法区也是线程共享区，用于存储【虚拟机加载的**类信息**（类的版本、字段、方法、接口），**常量**，**静态变量**，**即时编译器编译后的代码缓存**等数据】

**方法区和永久代的关系**：

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。 **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

**JDK8移除了永久代，改为元空间代替。为什么？**

因为永久代有上限，导致Java应用更容易遇到内存溢出问题。

内存回收的目标主要针对 常量池的回收 和 类型的卸载。

### 运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分，用于**存放编译期生成的各种字面量与符号引用**（属于类信息的一部分）。

《Java虚拟机规范》并没有做任何细节要求，可以由供应商自己实现。

运行时常量池与Class文件常量池对比 最大特征是**具备动态性**。运行时可以将新的常量放入池中，典型的有`String`类的`intern()`方法。

> `String`类的`intern()`方法：返回字符串对象的规范化表示形式。
>
> 一个初始时为空的字符串池，它由类 String 私有地维护。
>
> **当调用 intern 方法时，如果池已经包含一个等于此 String 对象的字符串（该对象由 `equals(Object)` 方法确定），则返回池中的字符串。否则，将此 String 对象添加到池中，并且返回此 String 对象的引用。**
>
> 它遵循对于任何两个字符串 s 和 t，当且仅当 `s.equals(t)` 为 `true` 时，`s.intern() == t.intern() `才为 `true`。

### 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGenCopy to clipboardErrorCopied
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小Copy to clipboardErrorCopied
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

### 为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 呢?

> 考虑到HotSpot未来的发展，在JDK 6的时候HotSpot开发团队就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区的计划了[1]，到了JDK 7的HotSpot，已经把原本放在永久代的字符串常量池、静态变量等移出，而到了JDK 8，终于完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间（Meta-space）来代替，把JDK 7中永久代还剩余的内容（主要是类型信息）全部移到元空间中。

1. 整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

   > 当元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

   你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

2. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。

3. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

## 直接内存

**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 `OutOfMemoryError` 异常出现**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

一般配置虚拟机参数时会根据实际内存去设置-Xmx等信息，但经常忽略直接内存，导致总内存大于物理内存限制，动态扩容时出现`OutOfMemoryError` 异常。

> JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

# HotSpot虚拟机

总结HotSpot虚拟机在Java堆中对象分配、布局和访问的全过程。

## 对象的创建过程

![Java创建对象的过程](https://gitee.com/koala010/typora/raw/master/img/20210621161216.png)

### Step1:类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

### Step2:分配内存

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

**内存分配的两种方式：**

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的。

![image-20210621161720591](https://gitee.com/koala010/typora/raw/master/img/20210621161720.png)

**内存分配并发问题：**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

### Step3:初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都**初始化为零值（不包括对象头）**，这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

### Step4:设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 这些信息存放在**对象头**中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

### Step5:执行 init 方法

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

## 对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：**对象头**、**实例数据**和**对齐填充**。

### 对象头

**Hotspot 虚拟机的对象头包括两部分信息**：

1. 用于存储**对象自身的运行时数据**（哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等）。官方称这部分为“Mark Word”。
2. **类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。
3. 特殊情况：如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通
   Java对象的元数据信息确定Java对象的大小，但是如果数组的长度是不确定的，将无法通过元数据中的
   信息推断出数组的大小。

![HotSpot虚拟机对象头Mark Word](https://cos.duktig.cn/typora/202109242103520.png)

### 实例数据

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

### 对齐填充部分

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 

因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。



### 对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有**① 使用句柄**和**② 直接指针**两种：

**句柄访问：** 如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

优点：reference中存储的是稳定句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference本身不需要被修改。

![通过句柄访问对象](https://gitee.com/koala010/typora/raw/master/img/20210621162654.png)

**直接指针访问**： 如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址。

优点：速度更快，它节省了一次指针定位的时间开销，由于对象访问在Java中非常频繁，因此这类开销积少成多也是一项极为可观的执行成本。

![通过直接指针访问对象](https://gitee.com/koala010/typora/raw/master/img/20210621162734.png)

HotSpot而言，它主要使用**直接指针访问方式**进行对象访问（有例外情况，如果使用了Shenandoah收集器的话也会有一次额外的转发）



# OutOfMemoryError异常

## Java堆溢出

Java堆用来不断创建对象实例，只要不断创建对象，并且避免垃圾回收机制清除这些对象（**保证GC Roots到对象之间有可达路径**
**来避免垃圾回收机制清除这些对象**），那么随着对象数量增加，超出总容量及最大容量限制后就会产生内存溢出异常。

限制Java堆的大小为20MB，不可扩展（将堆的最小值-Xms参数与最大值-Xmx参数设置为一样即可避免堆自动扩展），通过参数`-XX：+HeapDumpOnOutOfMemoryError`可以让虚拟机在出现内存溢出异常的时候Dump出当前的内存堆转储快照以便进行事后分析。

```java
/**
 * description:Java堆内存溢出异常测试
 * VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 *
 * @author RenShiWei
 * Date: 2021/6/20 17:26
 **/
public class HeapOOM {

    static class OOMObject {}

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

结果：

![image-20210620173433707](https://gitee.com/koala010/typora/raw/master/img/ Java堆溢出结果.png)

Java堆内存的`OutOfMemoryError`异常是实际应用中最常见的内存溢出异常情况。出现Java堆内存溢出时，异常堆栈信息“`java.lang.OutOfMemoryError`”会跟随进一步提示“`Java heap space`”。

**解决**：

解决这个异常，需要通过**内存映像分析工具**对Dump出来的堆转储快照进行分析。

第一步确定导致内存OMM的对象是否必要，即确定到底是**内存泄漏**，还是**内存溢出**。

如果是**内存泄漏**，可进一步通过工具**查看泄漏对象到GC Roots的引用链**，找到泄漏对象是通过怎样的引用路径、与哪些GC Roots相关联，才导致垃圾收集器无法回收它们，根据泄漏对象的类型信息以及它到GC Roots引用链的信息，一般可以比较准确地定位到这些对象创建的位置，进而找出产生内存泄漏的代码的具体位置。

如果是**内存溢出**，换句话说就是**内存中的对象确实都是必须存活的**，那就应当**检查Java虚拟机的堆参数（-Xmx与-Xms）设置**，与机器的内存对比，看看是否还有向上调整的空间。再**从代码上检查是否存在某些对象生命周期过长、持有状态时间过长、存储结构设计不合理等情况，尽量减少程序运行期的内存消耗**。

### Java的内存泄漏和内存溢出的区别

内存溢出就是你要求分配的内存超出了系统能给你的，系统不能满足需求，于是产生溢出。

Java内存泄漏就是没有及时清理内存垃圾，导致系统无法再给你提供内存资源（内存资源耗尽）。

例子：

- Java内存泄露是说程序逻辑问题，造成申请的内存无法释放，这样的话无论多少内存，早晚都会被占用光的。最简单的例子就是死循环了，由于程序判断错误导经常发生此事。
- Java内存泄漏是指在堆上分配的内存没有被释放，从而失去对其控制。这样会造成程序能使用的内存越来越少，导致系统运行速度减慢，严重情况会使程序当掉。

- 关于内存溢出有点出入。比如说你申请了一个integer，但给它存了long才能存下的数，那就是内存溢出。

关于内存泄漏与内存溢出参考：https://blog.csdn.net/sinat_35512245/article/details/54866068



## 虚拟机栈和本地方法栈溢出

由于HotSpot虚拟机中并不区分虚拟机栈和本地方法栈，因此对于HotSpot来说，-Xoss参数（设置本地方法栈大小）虽然存在，但实际上是没有任何效果的，栈容量只能由`-Xss`参数来设定。关于虚拟机栈和本地方法栈，在《Java虚拟机规范》中描述了两种异常：

1）**如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出`StackOverflowError`异常**。

2）**如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出`OutOfMemoryError`异常**。

> 《Java虚拟机规范》明确允许Java虚拟机实现自行选择是否支持栈的动态扩展，而HotSpot虚拟机的选择是不支持扩展，所以除非在创建线程申请内存时就因无法获得足够内存而出现`OutOfMemoryError`异常，否则在线程运行时是不会因为扩展而导致内存溢出的，只会因为栈容量无法容纳新的栈帧而导致`StackOverflowError`异常。

为了验证这点，我们可以做两个实验，先将实验范围限制在单线程中操作，尝试下面两种行为是否能让HotSpot虚拟机产生OutOfMemoryError异常：

**实例一：使用`-Xss`参数减少栈内存容量。**
结果：抛出`StackOverflowError`异常，异常出现时输出的堆栈深度相应缩小

```java
/**
 * description:VM Args：-Xss128k
 * 减少内存容量，测试虚拟机栈和本地方法栈溢出
 *
 * @author RenShiWei
 * Date: 2021/6/21 10:27
 **/
public class StackSOF {

    private int stackLength = 1;

    /**
     * 无限递归压栈，出现栈溢出现象
     */
    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) throws Throwable {
        StackSOF oom = new StackSOF();
        try {
            oom.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack length:" + oom.stackLength);
            throw e;
        }
    }

}
```

结果：

![image-20210621103925493](https://gitee.com/koala010/typora/raw/master/img/20210621103925.png)

**实例二：定义了大量的本地变量，增大此方法帧中本地变量表的长度。**
结果：抛出`StackOverflowError`异常，异常出现时输出的堆栈深度相应缩小。

```java
/**
 * description:定义了大量的本地变量，增大此方法帧中本地变量表的长度，测试虚拟机栈和本地方法栈的溢出
 *
 * @author RenShiWei
 * Date: 2021/6/21 10:37
 **/
public class StackSOF2 {

    private static int stackLength = 0;

    /**
     * 定义大量的变量，并且赋值
     */
    public static void test() {
        long unused1, unused2, unused3, unused4, unused5,
                unused6, unused7, unused8, unused9, unused10,
                unused11, unused12, unused13, unused14, unused15,
                unused16, unused17, unused18, unused19, unused20,
                unused21, unused22, unused23, unused24, unused25,
                unused26, unused27, unused28, unused29, unused30,
                unused31, unused32, unused33, unused34, unused35,
                unused36, unused37, unused38, unused39, unused40,
                unused41, unused42, unused43, unused44, unused45,
                unused46, unused47, unused48, unused49, unused50,
                unused51, unused52, unused53, unused54, unused55,
                unused56, unused57, unused58, unused59, unused60,
                unused61, unused62, unused63, unused64, unused65,
                unused66, unused67, unused68, unused69, unused70,
                unused71, unused72, unused73, unused74, unused75,
                unused76, unused77, unused78, unused79, unused80,
                unused81, unused82, unused83, unused84, unused85,
                unused86, unused87, unused88, unused89, unused90,
                unused91, unused92, unused93, unused94, unused95,
                unused96, unused97, unused98, unused99, unused100;

        stackLength++;
        test();

        unused1 = unused2 = unused3 = unused4 = unused5 =
                unused6 = unused7 = unused8 = unused9 = unused10 =
                        unused11 = unused12 = unused13 = unused14 = unused15 =
                                unused16 = unused17 = unused18 = unused19 = unused20 =
                                        unused21 = unused22 = unused23 = unused24 = unused25 =
                                                unused26 = unused27 = unused28 = unused29 = unused30 =
                                                        unused31 = unused32 = unused33 = unused34 = unused35 =
                                                                unused36 = unused37 = unused38 = unused39 = unused40 =
                                                                        unused41 = unused42 = unused43 = unused44 =
                                                                                unused45 =
                                                                                        unused46 = unused47 = unused48 =
                                                                                                unused49 = unused50 =
                                                                                                        unused51 =
                                                                                                                unused52 =
                                                                                                                        unused53 = unused54 =
                                                                                                                                unused55 =
                                                                                                                                        unused56 = unused57 =
                                                                                                                                                unused58 =
                                                                                                                                                        unused59 = unused60 =
                                                                                                                                                                unused61 =
                                                                                                                                                                        unused62 = unused63 = unused64 = unused65 =
                                                                                                                                                                                unused66 = unused67 = unused68 = unused69 = unused70 =
                                                                                                                                                                                        unused71 = unused72 = unused73 = unused74 = unused75 =
                                                                                                                                                                                                unused76 = unused77 = unused78 = unused79 = unused80 =
                                                                                                                                                                                                        unused81 = unused82 = unused83 = unused84 = unused85 =
                                                                                                                                                                                                                unused86 = unused87 = unused88 = unused89 = unused90 =
                                                                                                                                                                                                                        unused91 = unused92 = unused93 = unused94 = unused95 =
                                                                                                                                                                                                                                unused96 = unused97 = unused98 = unused99 = unused100 = 0;
    }

    public static void main(String[] args) {
        try {
            test();
        } catch (Error e) {
            System.out.println("stack length:" + stackLength);
            throw e;
        }
    }

}
```

![image-20210621104016532](https://gitee.com/koala010/typora/raw/master/img/20210621104016.png)

如果是远古时代的Classic虚拟机，这款虚拟机可以支持动态扩展栈内存的容量，定义大量的本地变量会产生OutOfMemoryError而不是StackOverflowError异常。

结果表明：**无论是由于栈帧太大还是虚拟机栈容量太小，当新的栈帧内存无法分配的时候，HotSpot虚拟机抛出的都是StackOverflowError异常**。

## 方法区和运行时常量池溢出

方法区的主要职责是用于存放类型的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。对于这部分区域的测试，基本的思路是运行时产大量的类去填满方法区，直到溢出为止。

直接使用Java SE API也可以动态产生类（如反射时的GeneratedConstructorAccessor和动态代理等）

借助CGLib使得方法区出现内存溢出异常：

```java
/**
 * VM Args：-XX:PermSize=10M -XX:MaxPermSize=10M
 * @author zzm
 */
public class JavaMethodAreaOOM {

    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws
                    return proxy.invokeSuper(obj, args);
                }
            });
            enhancer.create();
        }
    }

    static class OOMObject {
    }
}
```

**在JDK7的环境中测试**：

```java
Caused by: java.lang.OutOfMemoryError: PermGen space
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClassCond(ClassLoader.java:632)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:616)
    ... 8 more
```

**方法区溢出也是一种常见的内存溢出异常，一个类如果要被垃圾收集器回收，要达成的条件是比较苛刻的**。

在经常运行时生成大量动态类的应用场景里，就应该特别关注这些类的回收状况。这类场景除了之前提到的程序使用了**CGLib字节码增强**和动态语言外，常见的还有：**大量JSP或动态产生JSP文件的应用（JSP第一次运行时需要编译为Java类）**等。

**在JDK 8以后，元空间替代永久代。在默认设置下，很难再迫使虚拟机产生方法区的溢出异常了**。不过HotSpot还是提供了一些参数作为元空间的防御措施，主要包括：

- `-XX：MaxMetaspaceSize`：设置元空间最大值，默认是-1，即不限制，或者说只受限于本地内存大小。
- `-XX：MetaspaceSize`：指定元空间的初始空间大小，以字节为单位，达到该值就会触发垃圾收集进行类型卸载，同时收集器会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过-XX：MaxMetaspaceSize（如果设置了的话）的情况下，适当提高该值。
- `-XX：MinMetaspaceFreeRatio`：作用是在垃圾收集之后控制最小的元空间剩余容量的百分比，可减少因为元空间不足导致的垃圾收集的频率。类似的还有`-XX：Max-MetaspaceFreeRatio`，用于控制最大的元空间剩余容量的百分比。

## 本机直接内存溢出

 直接内存（Direct Memory）的容量大小可通过`-XX：MaxDirectMemorySize`参数来指定，如果不去指定，则默认与Java堆最大值（由-Xmx指定）一致。

```java
Exception in thread "main" java.lang.OutOfMemoryError
    at sun.misc.Unsafe.allocateMemory(Native Method)
    at org.fenixsoft.oom.DMOOM.main(DMOOM.java:20)
```

由直接内存导致的内存溢出，一个明显的特征是在Heap Dump文件中不会看见有什么明显的异常情况，如果读者发现内存溢出之后产生的Dump文件很小，而程序中又直接或间接使用了DirectMemory（典型的间接使用就是NIO），那就可以考虑重点检查一下直接内存方面的原因了



# String类和常量池

## String 类和常量池

**String 对象的两种创建方式：**

```java
String str1 = "abcd";//先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；
String str2 = new String("abcd");//堆中创建一个新的对象
String str3 = new String("abcd");//堆中创建一个新的对象
System.out.println(str1==str2);//false
System.out.println(str2==str3);//false
```

这两种不同的创建方法是有差别的。

- 第一种方式是在常量池中拿对象；
- 第二种方式是直接在堆内存空间创建一个新的对象。

记住一点：**只要使用 new 方法，便需要创建新的对象。**

![String-Pool-Java](https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/2019-3String-Pool-Java1-450x249.png)

**String 类型的常量池比较特殊。它的主要使用方法有两种：**

1. 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
2. 如果不是用双引号声明的 String 对象，可以使用 String 提供的 `intern()` 方法。`String.intern()` 是一个 Native 方法，它的作用是：如果运行时常量池中已经包含一个等于此 String 对象内容的字符串，则返回常量池中该字符串的引用；如果没有，JDK1.7 之前（不包含 1.7）的处理方式是在常量池中创建与此 String 内容相同的字符串，并返回常量池中创建的字符串的引用，JDK1.7 以及之后的处理方式是在常量池中记录此字符串的引用，并返回该引用。

> 在JDK 6中，intern()方法会把首次遇到的字符串实例复制到永久代的字符串常量池中存储，返回的也是永久代里面这个字符串实例的引用。
>
> JDK 7（以及部分其他虚拟机，例如JRockit）的intern()方法实现就不需要再拷贝字符串的实例到永久代了，既然字符串常量池已经移到Java堆中，那只需要在常量池里记录一下首次出现的实例引用即可。

JDK8 :

```java
String s1 = "计算机";
String s2 = s1.intern();
String s3 = "计算机";
System.out.println(s2);//计算机
System.out.println(s1 == s2);//true
System.out.println(s3 == s2);//true，因为两个都是常量池中的 String 对象
```

**字符串拼接:**

```java
String str1 = "str";
String str2 = "ing";

String str3 = "str" + "ing";//常量池中的对象
String str4 = str1 + str2; //在堆上创建的新的对象
String str5 = "string";//常量池中的对象
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

![字符串拼接](https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%8B%BC%E6%8E%A5-%E5%B8%B8%E9%87%8F%E6%B1%A02.png)

尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 StringBuilder 或者 StringBuffer。

## `String s1 = new String("abc");`这句话创建了几个字符串对象？

**将创建 1 或 2 个字符串。如果池中已存在字符串常量“abc”，则只会在堆空间创建一个字符串常量“abc”。如果池中没有字符串常量“abc”，那么它将首先在池中创建，然后在堆空间中创建，因此将创建总共 2 个字符串对象。**

**验证：**

```java
String s1 = new String("abc");// 堆内存的地址值
String s2 = "abc";
System.out.println(s1 == s2);// 输出 false,因为一个是堆内存，一个是常量池的内存，故两者是不同的。
System.out.println(s1.equals(s2));// 输出 true
```

**结果：**

```
false
true
```

##  8种基本类型的包装类和常量池

**Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character 创建了数值在[0,127]范围的缓存数据，Boolean 直接返回 True Or False。如果超出对应范围仍然会去创建新的对象。** 为啥把缓存设置为[-128，127]区间？（[参见 issue/461](https://github.com/Snailclimb/JavaGuide/issues/461)）性能和资源之间的权衡。

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}

private static class CharacterCache {
    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}
```

两种浮点数类型的包装类 Float,Double 并没有实现常量池技术。

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true
Integer i11 = 333;
Integer i22 = 333;
System.out.println(i11 == i22);// 输出 false
Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```

**Integer 缓存源代码：**

```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

**应用场景：**

1. `Integer i1=40`；Java 在编译的时候会直接将代码封装成 `Integer i1=Integer.valueOf(40);`，从而使用常量池中的对象。
2. `Integer i1 = new Integer(40);`这种情况下会创建新的对象。

```java
  Integer i1 = 40;
  Integer i2 = new Integer(40);
  System.out.println(i1==i2);//输出 false
```

**Integer 比较更丰富的一个例子:**

```java
  Integer i1 = 40;
  Integer i2 = 40;
  Integer i3 = 0;
  Integer i4 = new Integer(40);
  Integer i5 = new Integer(40);
  Integer i6 = new Integer(0);

  System.out.println("i1=i2   " + (i1 == i2));
  System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
  System.out.println("i1=i4   " + (i1 == i4));
  System.out.println("i4=i5   " + (i4 == i5));
  System.out.println("i4=i5+i6   " + (i4 == i5 + i6));
  System.out.println("40=i5+i6   " + (40 == i5 + i6));
```

结果：

```
i1=i2   true
i1=i2+i3   true
i1=i4   false
i4=i5   false
i4=i5+i6   true
40=i5+i6   true
```

解释：

语句 `i4 == i5 + i6`，因为+这个操作符不适用于 Integer 对象，首先 i5 和 i6 进行自动拆箱操作，进行数值相加，即 i4 == 40。然后 Integer 对象无法与数值进行直接比较，所以 i4 自动拆箱转为 int 值 40，最终这条语句转为 40 == 40 进行数值比较。

# 垃圾回收机制

## **为什么需要了解垃圾回收机制**？

当需要排查各种内存溢出、内存泄漏问题时，当垃圾收集成为系统达到更高并发量的瓶颈时，我们就必须对这些“自动化”的技术实施必要的监控和调节。

## **哪些区域需要垃圾回收**？

程序计数器、虚拟机栈和本地方法栈同线程生命周期一致，方法/线程技结束，内存直接回收，无需过多考虑。

**堆和方法区有着不确定性**：一个接口多个实现类所需内存不一样，一个方法不同的条件和分支所需内存也可能不一样。只有在运行期间才知道程序会创建哪些对象、多少个对象。这部分内存分配和回收是动态，所以需要关注内存的分配与回收。

## 如何判断对象需要被回收？

### 引用计数算法

> 给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加 1；当引用失效，计数器就减 1；任何时候计数器为 0 的对象就是不可能再被使用的。

**这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题。**

> 对象objA和objB都有字段`instance`，赋值令 `objA.instance=objB` 及 `objB.instance=objA` ，除此之外，这两个对象再无任何引用，实际上这两个对象已经不可能再被访问，但是它们因为互相引用着对方，导致它们的引用计数都不为零，引用计数算法也就无法回收它们。

### 可达性分析算法

> 通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的。

![image-20210622161600979](https://gitee.com/koala010/typora/raw/master/img/ 可达性分析算法20210622161601.png)

**可作为 GC Roots 的对象**：

- **在虚拟机栈（栈帧中的本地变量表）中引用的对象**，譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
- **在方法区中类静态属性引用的对象**，譬如Java类的引用类型静态变量。
- **在方法区中常量引用的对象**，譬如字符串常量池（String Table）里的引用。
- **在本地方法栈中JNI（即通常所说的Native方法）引用的对象**。
- **Java虚拟机内部的引用**，如基本数据类型对应的Class对象，一些常驻的异常对象（比如NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器。
- **所有被同步锁（synchronized关键字）持有的对象**。
- **反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等**。



### **利用可达性分析算法，如何判断一个对象死亡**？

即使在可达性分析算法中判定为不可达的对象，也不是“非死不可”的，这时候它们暂时还处于**“缓刑”阶段**，要真正宣告一个对象死亡，至少要经历两次标记过程。

第一次标记：对象没有与`GC Roots`相连时，判断该对象的`finalize`方法有没有被覆盖过，或者有没有被虚拟机执行过。如果没有，则直接被回收；如果执行过，对象被放置进`F-Queue`队列中，进行第二次标记。

第二次标记：如果对象在`finalize`关联上了`GC Roots`，在队列中移除（只能关联一次）；如果没有，被回收。

## Java中的引用

JDK1.2 之前，Java 中引用的定义很传统：如果 reference 类型的数据存储的数值代表的是另一块内存的起始地址，就称这块内存代表一个引用。

JDK1.2 以后，Java 对引用的概念进行了扩充，将引用分为强引用、软引用、弱引用、虚引用四种（引用强度逐渐减弱）。

![Java四种引用的结构](https://gitee.com/koala010/typora/raw/master/img/Java四种引用的结构20210622164831.png)

**1、强引用**

强引用是最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似`Objectobj=new Object()`这种引用关系。

**只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。**

**2、软引用**（`SoftReference`）

描述有些**还有用但并非必需的对象**。

**在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围进行二次回收**。如果这次回收还没有足够的内存，才会抛出内存溢出异常。

**3、弱引用**（`WeakReference`）

描述非必需对象。**被弱引用关联的对象只能生存到下一次垃圾回收之前**，垃圾收集器工作之后，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。

**4、虚引用**（`PhantomReference`）

这个引用存在的**唯一目的就是在这个对象被收集器回收时收到一个系统通知**，**被虚引用关联的对象，和其生存时间完全没关系**。

关于应用的使用案例参考：[Java的四种引用详解与使用案例](https://blog.csdn.net/TJtulong/article/details/104879688)

提到Java引用及垃圾回收机制，当我联想到了以前的一个说法，**在链表删除一个节点的时候，将这个节点置为null，以方便下次垃圾回收机制删除**，这个操作是否有意义？

通常情况下没有什么意义，除非在特定的一些情况下：

1 同一个方法中
2 定义了一个大对象(小对象没有意义)
3 之后跟着一个非常耗时的操作.
4 没有满足JIT编译条件

 上面4个条件缺一不可,把obj显式设置成null才是有意义的。

参考：

- [在Java中将对象分配为null会影响垃圾回收吗？](https://www.itranslater.com/qa/details/2126692746223158272)
- [java中将对象赋值为null，对垃圾回收有用吗？](https://blog.csdn.net/qq_42945742/article/details/84107531)



## 方法区的回收

《Java虚拟机规范》中提到过可以不要求虚拟机在方法区中实现垃圾收集。

方法区垃圾收集的“性价比”通常也是比较低的：在Java堆中，尤其是在新生代中，对常规应用进行一次垃圾收集通常可以回收70%至99%的内存空间，相比之下，方法区回收囿于苛刻的判定条件，其区域垃圾收集的回收成果往往远低于此。

**方法区的垃圾收集主要回收两部分内容：废弃的常量和不再使用的类型**。

> 举个常量池中字面量回收的例子，假如一个字符串“java”曾经进入常量池中，但是当前系统又没有任何一个字符串对象的值是“java”，换句话说，已经没有任何字符串对象引用常量池中的“java”常量，且虚拟机中也没有其他地方引用这个字面量。如果在这时发生内存回收，而且
> 垃圾收集器判断确有必要的话，这个“java”常量就将会被系统清理出常量池。常量池中其他类（接口）、方法、字段的符号引用也与此似。

判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
- 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。
- ·该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

**在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力**。



## 垃圾回收算法

### 分代收集理论

建立在两个分代假说之上：

1）**弱分代假说（Weak Generational Hypothesis）：绝大多数对象都是朝生夕灭的**。

2）**强分代假说（Strong Generational Hypothesis）：熬过越多次垃圾收集过程的对象就越难以消亡**。

收集器应该将Java堆划分出不同的区域，然后将回收对象依据其年龄（年龄即对象熬过垃圾收集过程的次数）分配到不同的区域之中存储。

因为对象之间会存在跨代引用，进行一次Minor GC，但新生代对象可能被老年代引用，不得不在GC Roots之外再遍历老年代确保可达性分析的正确性，反之一样。可能会给内存回收带来很大的性能负担。

**跨代引用假说（Intergenerational Reference Hypothesis）：跨代引用相对于同代引用来说仅占极少数**。隐含推论：存在互相引用关系的两个对象，是应该倾向于同时生存或者同时消亡的。

需在新生代上建立一个全局的数据结构，把老年代划分成若干小块，标识出老年代的哪一块内存会存在跨代引用，可以缩小扫描范围。

### 垃圾回收类型

- 部分收集（Partial GC）：指目标不是完整收集整个Java堆的垃圾收集，其中又分为：
  - 新生代收集（Minor GC/Young GC）：指目标只是新生代的垃圾收集。
  - 老年代收集（Major GC/Old GC）：指目标只是老年代的垃圾收集。目前只有CMS收集器会有单独收集老年代的行为。另外请注意“Major GC”这个说法现在有点混淆，在不同资料上常有不同所指，读者需按上下文区分到底是指老年代的收集还是整堆收集。
  - 混合收集（Mixed GC）：指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有G1收集器会有这种行为。
- 整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集。

### 标记-清除算法（Mark-Sweep）

**标记需回收/存活对象，统一清除标记的对象**。

<img src="https://gitee.com/koala010/typora/raw/master/img/标记-清除算法20210622174750.png" alt="image-20210622174750725" style="zoom:67%;" />

**在CMS回收器中用于老年代**。

**缺点**：

- **执行效率不稳定**。对象越多，效率越低。
- **内存空间的碎片化问题**。

### 标记-复制算法（Mark-Copying）

**“半区复制”，每次只用一块内存，存活对象复制到另一块，已经使用的那一块直接清除**。

<img src="https://gitee.com/koala010/typora/raw/master/img/标记-复制算法20210622175418.png" alt="image-20210622175417983" style="zoom:67%;" />

一般用于**新生代**

优点：无内存碎片化问题。

缺点：对象存活率高时，大量复制，效率低。

#### 新生代Eden和Survivor比例问题

> 现在的商用Java虚拟机大多都优先采用了这种收集算法去回收新生代，IBM公司曾有一项专门研究对新生代“朝生夕灭”的特点做了更量化的诠释——**新生代中的对象有98%熬不过第一轮收集**。因此并不需要按照1∶1的比例来划分新生代的内存空间。
>
> **“Appel式回收”**：Appel式回收的具体做法是把新生代分为一块较大的Eden空间和两块较小的Survivor空间，每次分配内存只使用Eden和其中一块Survivor。**发生垃圾搜集时，将Eden和Survivor中仍然存活的对象一次性复制到另外一块Survivor空间上，然后直接清理掉Eden和已用过的那块Survivor空间**。

Eden区是一块，Survivor区是两块。Eden区和Survivor区的比例是8：1：1。

98%的对象可被回收仅仅是“普通场景”下测得的数据，任何人都没有办法百分百保证每次回收都只有不多于10%的对象存活，因此Appel式回收还有一个充当罕见情况的“逃生门”的安全设计。当Survivor空间不足以容纳一次Minor GC之后存活的对象时，就需要依赖其他内存区域（实际上大多就是老年代）进行**分配担保（Handle Promotion）**。

**如果另外一块Survivor空间没有足够空间存放上一次新生代收集下来的存活对象，这些对象便将通过分配担保机制直接进入老年代**。

参考：[新生代Eden与两个Survivor区的解释](https://blog.csdn.net/lojze_ly/article/details/49456255)

### 标记-整理算法（Mark-Compact）

标记存活对象，让所有存活对象移动到一端，然后清理所有的边界以外的内存。

<img src="https://gitee.com/koala010/typora/raw/master/img/ 标记-整理算法（Mark-Compact）.png" alt="image-20210622180525483" style="zoom:67%;" />

一般用于**老年代**。

**是否移动对象都有弊端，移动内存回收比较复杂，不移动内存分配比较复杂（空间碎片化）**。

一种方案：**虚拟机平时采用标记-清除算法，暂时容忍碎片存在，直到碎片影响大到内存分配时，在进行标记-整理算法，可获得规整的内存空间**。

关于**标记-整理算法的可回收对象与存活对象如何移动？**，可参考：[垃圾回收算法——标记—整理回收](https://blog.csdn.net/luliuliu1234/article/details/104058259) （了解）

## HotSpot实现垃圾回收的算法细节（了解）

> 了解这部分内容是很好理解垃圾回收器工作原理的前提，但是这部分内容往往比较难以理解，需要结合垃圾收集器反复推敲。
>
> 这部分主要讲解垃圾收集中遇到的哪些问题，以及如何解决的？
>
> *简单总结，详细内容参看：《深入理解Java虚拟机（第三版）》*

### 根节点枚举

固定可作为GC Roots的节点主要在全局性的引用（例如常量或类静态属性）与执行上下文（例如栈帧中的本地变量表）中，尽管目标明确，但查找过程要做到高效并非一件容易的事情，现在Java应用越做越庞大，光是方法区的大小就常有数百上千兆，里面的类、常量等更是恒河沙数，若要逐个检查以这里为起源的引用肯定得消耗不少时间。

迄今为止，**所有收集器在根节点枚举这一步骤时都是必须暂停用户线程的**，因此毫无疑问根节点枚举与之前提及的整理内存碎片一样会面临相似的“Stop The World”的困扰。

**垃圾收集过程必须停顿所有用户线程的其中一个重要原因**：

现在可达性分析算法耗时最长的查找引用链的过程已经可以做到与用户线程一起并发，但根节点枚举始终还是必须在一个能保障一致性的快照中才得以进行——这里**“一致性”的意思是整个枚举期间执行子系统看起来就像被冻结在某个时间点上，不会出现分析过程中，根节点集合的对象引用关系还在不断变化的情况**，若这点不能满足的话，分析结果准确性也就无法保证。

由于目前主流Java虚拟机使用的都是**准确式垃圾收集**，所以**当用户线程停顿下来之后，其实并不需要一个不漏地检查完所有执行上下文和全局的引用位置，虚拟机应当是有办法直接得到哪些地方存放着对象引用的**。

> 在HotSpot的解决方案里，使用OopMap的数据结构来达到这个目的。一旦类加载动作完成的时候，HotSpot就会把对象内什么偏移量上是什么类型的数据计算出来，在即时编译过程中，也会在特定的位置记录下栈里和寄存器里哪些位置是引用。这样收集器在扫描时就可以直接得知这些信息了，并不需要真正一个不漏地从方法区等GC Roots开始查找。

总结：

1. **所有收集器在根节点枚举这一步骤时都是必须暂停用户线程的**
2. **在HotSpot中，使用OopMap的数据结构可以，当用户线程停顿下来之后，使虚拟机直接得到哪些地方存放着对象引用，而不需要一个不漏地检查完所有执行上下文和全局的引用位置。**
3. OopMap的目的：**使HotSpot可以快速准确地完成GC Roots枚举**

### 安全点

在OopMap的协助下，HotSpot可以快速准确地完成GC Roots枚举，**但是可能导致引用关系变化，或者说导致OopMap内容变化的指令非常多，如果为每一条指令都生成对应的OopMap，那将会需要大量的额外存储空间，成本高昂**。

在“特定的位置”记录了这些信息，这些位置被称为安全点（Safepoint），有了安全点的设定，也就决定了用户程序执行时并非在代码指令流的任意位置都能够停顿下来开始垃圾收集，而是**强制要求必须执行到达安全点后才能够暂停**。

**安全点的选定既不能太少以至于让收集器等待时间过长，也不能太过频繁以至于过分增大运行时的内存负荷**。安全点位置的选取基本上是以“**是否具有让程序长时间执行的特征**”为标准进行选定的，因为每条指令执行的时间都非常短暂，程序不太可能因为指令流长度太长这样的原因而长时间执行，“长时间执行”的最明显特征就是**指令序列的复用**，例如方法调用、循环跳转、异常跳转等功能的指令才会产生安全点。

另外一个问题：**如何在垃圾收集发生时让所有线程（这里其实不包括执行JNI调用的线程）都跑到最近的安全点，然后停顿下来？**

这里有两种方案可供选择：抢先式中断（Preemptive Suspension）和**主动式中断**（Voluntary Suspension，一般采用的方案）。

> **主动式中断**的思想是当垃圾收集需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志位，各个线程执行过程时会不停地主动去轮询这个标志，一旦发现中断标志为真时就自己在最近的安全点上主动中断挂起。轮询标志的地方和安全点是重合的，另外还要加上所有创建对象和其他需要在Java堆上分配内存的地方，这是为了检查是否即将要发生垃圾收集，避免没有足够内存分配新对象。

总结：

1. 安全点的目的：**在“特定的位置”记录OopMap需要记录的信息，解决每一个指令生成对应OopMap的高昂存储空间问题**。即**解决如何停顿用户线程，让虚拟机进入垃圾回收状态的问题**。
2. 如何选取安全点位置：以“**是否具有让程序长时间执行的特征**”为标准（即指令复用），例如方法调用、循环跳转、异常跳转等功能的指令才会产生安全点。
3. 如何使垃圾收集时线程跑到最近的安全点：**主动式中断**。

### 安全区域

使用安全点的设计似乎已经完美解决如何停顿用户线程，让虚拟机进入垃圾回收状态的问题了。安全点机制保证了程序执行时，在不太长的时间内就会遇到可进入垃圾收集过程的安全点。但是，**程序“不执行”的时候呢？**所谓的程序不执行就是**没有分配处理器时间**，典型的场景便是***用户线程处于Sleep状态或者Blocked状态，这时候线程无法响应虚拟机的中断请求，不能再走到安全的地方去中断挂起自己，虚拟机也显然不可能持续等待线程重新被激活分配处理器时间***。对于这种情况，就必须引入安全区域（Safe Region）来解决。

**安全区域是指能够确保在某一段代码片段之中，引用关系不会发生变化**，因此，在这个区域中任意地方开始垃圾收集都是安全的。我们也可以把安全区域看作被扩展拉伸了的安全点。

> 流程：
>
> 当用户线程执行到安全区域里面的代码时，首先会标识自己已经进入了安全区域，那样当这段时间里虚拟机要发起垃圾收集时就不必去管这些已声明自己在安全区域内的线程了。
>
> 当线程要离开安全区域时，它要检查虚拟机是否已经完成了根节点枚举（或者垃圾收集过程中其他需要暂停用户线程的
> 阶段），如果完成了，那线程就当作没事发生过，继续执行；否则它就必须一直等待，直到收到可以离开安全区域的信号为止。

总结：

1. 引入安全区域的目的：**解决程序不执行时（没有分配处理器时间），线程无法响应虚拟机的中断请求挂起自己的问题**。
2. **安全区域是指能够确保在某一段代码片段之中，引用关系不会发生变化，保证在这个区域中任意地方开始垃圾收集都是安全的。**

### 记忆集与卡表

垃圾收集器**在新生代中建立了名为记忆集**（Remembered Set）的数据结构，用以**避免把整个老年代加进GC Roots扫描范围**。

记忆集是一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构，卡表是其实现。

### 写屏障

记忆集来缩减GC Roots扫描范围的问题，但还没有**解决卡表元素如何维护的问题，例如它们何时变脏、谁来把它们变脏等**，使用写屏障。

### 并发的可达性分析

可达性分析算法理论上要求全过程都基于一个能保障一致性的快照中才能够进行分析，这意味着必须全程冻结用户线程的运行。

 根节点枚举 在各种优化下时间固定，但是“标记”阶段，也就是**GC Roots再继续往下遍历对象图时，停顿时间就必定会与Java堆容量直接成正比例关系**。如果这个阶段会随着堆变大而等比例增加停顿时间，其影响就会波及几乎所有的垃圾收集器。

使用并发扫描可以，大大提升“标记”阶段的速度，但是可能存在**对象消失问题**。

解决：CMS是基于增量更新来做并发标记的，G1、Shenandoah则是用原始快照来实现。

这部分详细参看：《深入理解Java虚拟机（第三版）》

## Young GC 和 Full GC 在什么情况下触发？

- Minor GC：新生代的 GC
- Major GC：老年代的 GC
- Full GC：整堆收集，收集整个 Java 堆和方法区的垃圾收集

对象优先在新生代Eden区中分配，如果Eden区没有足够的空间时，就会触发一次young gc

Full gc的触发条件有多个，FULL GC的时候会STOP THE WORLD。

- 在执行Young gc之前，JVM会进行空间分配担保——如果老年代的连续空间小于新生代对象的总大小（或历次晋升的平均大小），则触发一次full gc。如果大于历次晋升的平均大小，那么会进行一次Minor GC，尽管这次Minor GC是有风险的。

- 显式调用System.gc()方法时，系统建议执行，不是必然执行；

- 老年代空间不足

- 方法区（元空间）空间不足

  

## 三色标记法

### 三色标记原理

颜色含义：

- 白色：还没有搜索过的对象（标记结束后，白色对象会被当成垃圾对象）
- 灰色：正在搜索的对象 
- 黑色：搜索完成的对象（不会当成垃圾对象，不会被GC） 

假设现在有白、灰、黑三个集合（表示当前对象的颜色），其遍历访问过程为：

1. 初始时，所有对象都在【白色集合】中；
2. 将 GC Roots 直接引用到的对象挪到 【灰色集合】中；
3. 从灰色集合中获取对象：
   3.1. 将本对象引用到的其他对象全部挪到 【灰色集合】中；
   3.2. 将本对象挪到【黑色集合】里面。
4. 重复步骤3，直至【灰色集合】为空时结束。
5. 结束后，仍在【白色集合】的对象即为 GC Roots 不可达，可以进行回收。

> 注：如果标记结束后对象仍为白色，意味着已经“找不到”该对象在哪了，不可能会再被重新引用。

![img](https://cos.duktig.cn/typora/202112292135030.gif)

当 Stop The World （以下简称 STW）时，对象间的引用是不会发生变化的，可以轻松完成标记。

而当需要支持并发标记时，即标记期间应用线程还在继续跑，对象间的引用可能发生变化，多标和漏标的情况就有可能发生。

### 多标——浮动垃圾

假设已经遍历到 E（变为灰色了），此时应用执行了 objD.fieldE = null (D > E 的引用断开)：

![img](https://cos.duktig.cn/typora/202112292142247.png)

此刻之后，对象 E/F/G 是“应该”被回收的。然而因为 E 已经变为灰色了，其仍会被当作存活对象继续遍历下去。最终的结果是：这部分对象仍会被标记为存活，即本轮 GC 不会回收这部分内存。

这部分本应该回收 但是没有回收到的内存，被称之为“浮动垃圾”。浮动垃圾并不会影响应用程序的正确性，只是需要等到下一轮垃圾回收中才被清除。

另外，针对并发标记开始后的新对象，通常的做法是直接全部当成黑色，本轮不会进行清除。这部分对象期间可能会变为垃圾，这也算是浮动垃圾的一部分。

### 漏标-读写屏障

假设 GC 线程已经遍历到 E（变为灰色了），此时应用线程先执行了：

```
var G = objE.fieldG; 
objE.fieldG = null;  // 灰色E 断开引用 白色G 
objD.fieldG = G;  // 黑色D 引用 白色G
```

![img](https://cos.duktig.cn/typora/202112292143458.png)

此时切回 GC 线程继续跑，因为 E 已经没有对 G 的引用了，所以不会将 G 放到灰色集合；尽管因为 D 重新引用了 G，但因为 D 已经是黑色了，不会再重新做遍历处理。

最终导致的结果是：G 会一直停留在白色集合中，最后被当作垃圾进行清除。这直接影响到了应用程序的正确性，是不可接受的。

不难分析，漏标只有同时满足以下两个条件时才会发生：

1. 灰色对象断开了白色对象的引用（直接或间接的引用）；即灰色对象原来成员变量的引用发生了变化。
2. 黑色对象重新引用了该白色对象；即黑色对象成员变量增加了新的引用。

从代码的角度看：

```
var G = objE.fieldG; // 1.读
objE.fieldG = null;  // 2.写
objD.fieldG = G;     // 3.写
```

1. 读取对象 E 的成员变量 fieldG 的引用值，即对象 G；
2. 对象 E 往其成员变量 fieldG，写入 null值。
3. 对象 D 往其成员变量 fieldG，写入对象 G ；

我们只要在上面这三步中的任意一步中做一些“手脚”，将对象 G 记录起来，然后作为灰色对象再进行遍历即可。比如放到一个特定的集合，等初始的 GC Roots 遍历完（并发标记），该集合的对象遍历即可（重新标记）。

> 重新标记是需要 STW 的，因为应用程序一直在跑的话，该集合可能会一直增加新的对象，导致永远都跑不完。当然，并发标记期间也可以将该集合中的大部分先跑了，从而缩短重新标记 STW 的时间，这个是优化问题了。

写屏障用于拦截第二和第三步；而读屏障则是拦截第一步。它们的拦截的目的很简单：就是在读写前后，将对象 G 给记录下来。

### 写屏障

给某个对象的成员变量赋值时，其底层代码大概长这样：

```
/**
* @param field 某对象的成员变量，如 D.fieldG
* @param new_value 新值，如 null
*/
void oop_field_store(oop* field, oop new_value) { 
    *field = new_value; // 赋值操作
} 
```

所谓的写屏障，其实就是指在赋值操作前后，加入一些处理（可以参考AOP的概念），读屏障的含义也类似。

```
void oop_field_store(oop* field, oop new_value) {  
    pre_write_barrier(field); // 写屏障-写前操作
    *field = new_value; 
    post_write_barrier(field, value);  // 写屏障-写后操作
}
```

#### 写屏障 + SATB

当对象 E 的成员变量的引用发生变化时（objE.fieldG = null;），我们可以利用写屏障，将 E 原来成员变量的引用对象 G 记录下来：

```
void pre_write_barrier(oop* field) {
    oop old_value = *field; // 获取旧值
    remark_set.add(old_value); // 记录 原来的引用对象
}
```

当原来成员变量的引用发生变化之前，记录下原来的引用对象。

这种做法的思路是：尝试保留开始时的对象图，即原始快照（Snapshot At The Beginning，SATB），当某个时刻 的 GC Roots 确定后，当时的对象图就已经确定了。

比如 当时 D 是引用着 G 的，那后续的标记也应该是按照这个时刻的对象图走（D 引用着 G）。如果期间发生变化，则可以记录起来，保证标记依然按照原本的视图来。

> SATB 破坏了条件一：【灰色对象断开了白色对象的引用】，从而保证了不会漏标。

### 写屏障 + 增量更新

当对象 D 的成员变量的引用发生变化时（objD.fieldG = G;），我们可以利用写屏障，将 D 新的成员变量引用对象 G 记录下来：

```
void post_write_barrier(oop* field, oop new_value) {  
  if($gc_phase == GC_CONCURRENT_MARK && !isMarkd(field)) {
      remark_set.add(new_value); // 记录新引用的对象
  }
}
```

当有新引用插入进来时，记录下新的引用对象。

这种做法的思路是：不要求保留原始快照，而是针对新增的引用，将其记录下来等待遍历，即增量更新（Incremental Update）。

> 增量更新破坏了条件二：【黑色对象重新引用了该白色对象】，从而保证了不会漏标。

### 读屏障

```
oop oop_field_load(oop* field) {
    pre_load_barrier(field); // 读屏障-读取前操作
    return *field;
}
```

读屏障是直接针对第一步：var G = objE.fieldG;，当读取成员变量时，一律记录下来：

```
void pre_load_barrier(oop* field, oop old_value) {  
  if($gc_phase == GC_CONCURRENT_MARK && !isMarkd(field)) {
      oop old_value = *field;
      remark_set.add(old_value); // 记录读取到的对象
  }
}
```

这种做法是保守的，但也是安全的。因为条件二中【黑色对象重新引用了该白色对象】，重新引用的前提是：得获取到该白色对象，此时已经读屏障就发挥作用了。

### 三色标记法与现代垃圾回收器

现代追踪式（可达性分析）的垃圾回收器几乎都借鉴了三色标记的算法思想，尽管实现的方式不尽相同：比如白色/黑色集合一般都不会出现（但是有其他体现颜色的地方）、灰色集合可以通过栈/队列/缓存日志等方式进行实现、遍历方式可以是广度/深度遍历等等。

对于读写屏障，以Java HotSpot VM 为例，其并发标记时对漏标的处理方案如下：

- CMS：写屏障 + 增量更新
- G1：写屏障 + SATB
- ZGC：读屏障



参看：[JVM系列十六（三色标记法与读写屏障）](https://www.cnblogs.com/jmcui/p/14165601.html)

# 垃圾收集器

## 经典的垃圾收集器

![垃圾收集器总结](https://gitee.com/koala010/typora/raw/master/img/垃圾收集器总结.png)

《Java虚拟机规范》中对垃圾收集器应该如何实现并没有做出任何规定。

**jdk1.8默认使用ParallelGC。新生代采用的是Parallel Scavenge，老年代Parallel Old。**

使用ParNew（标记复制、并行、作用于新生代） + CMS的垃圾收集器（标记清除、并行、作用于老年代），追求响应速度优先，其适用于多CPU环境的Server模式的互联网或者B/S业务。

如果追求吞吐量优先，应用在后台运算并不需要太多交互场景的，可采用Parallel（标记复制、并行、作用于新生代） + Parallel Old 的垃圾收集器（标记整理、并行、作用于老年代）

> **“Stop The World”**：**虚拟机在后台自动发起和自动完成的，在用户不可知、不可控的情况下把用户的正常工作的用户线程全部停掉**。这对很多应用来说都是不能接受的。

### Serial收集器

新生代  单线程   复制算法

优点：

- 与其他单线程垃圾收集器相比简单高效
- 对于内存受限的环境，额外内存消耗最小
- 对于单核或者核心数较少的环境来说，其没有线程交互开销，专心做垃圾收集可以获得最高的单线程收集效率

使用场景：

Serial收集器对于运行在**客户端模式下的虚拟机**来说是一个很好的选择。

> 在用户桌面的应用场景以及近年来流行的部分微服务应用中，分配给虚拟机管理的内存一般来说并不会特别大，收集几十兆甚至一两百兆的新生代（仅仅是指新生代使用的内存，桌面应用甚少超过这个容量），垃圾收集的停顿时间完全可以控制在十几、几十毫秒，最多一
> 百多毫秒以内，只要不是频繁发生收集，这点停顿时间对许多用户来说是完全可以接受的。

Serial/Serial Old收集器运行示意图：

![Serial/Serial Old收集器运行示意图](https://cos.duktig.cn/typora/202110021731339.png)




### ParNew收集器

ParNew收集器实质上是Serial收集器的多线程并行版本。

ParNew收集器是激活CMS后（使用-XX：+UseConcMarkSweepGC选项）的默认新生代收集器。

新生代 多线程   复制算法。

**JDK9以前ParNew + CMS 适用于 服务端模式下的垃圾收集器组合**。

ParNew/Serial Old收集器运行示意图：

![ParNew/Serial Old收集器运行示意图](https://cos.duktig.cn/typora/202110021732241.png)

### Parallel Scavenge收集器

新生代 多线程   复制算法

CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是**达到一个可控制的吞吐**
**量**（Throughput）。所谓吞吐量就是处理器用于运行用户代码的时间与处理器总消耗时间的比值。

停顿时间越短就越适合需要与用户交互或需要保证服务响应质量的程序，良好的响应速度能提升用户体验；而高吞吐量则可以最高效率地利用处理器资源，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的分析任务。

> Parallel Scavenge收集器提供了两个参数用于精确控制吞吐量，分别是控制最大垃圾收集停顿时间的`-XX：MaxGCPauseMillis`参数以及直接设置吞吐量大小的`XX：GCTimeRatio`参数。
>
> **-XX：MaxGCPauseMillis参数允许的值是一个大于0的毫秒数，收集器将尽力保证内存回收花费的时间不超过用户设定值**。垃圾收集停顿时间缩短是以牺牲吞吐量和新生代空间为代价换取的：系统把新生代调得小一些，收集300MB新生代肯定比收集500MB快，但这也直接导致垃圾收集发生得更频繁，原来10秒收集一次、每次停顿100毫秒，现在变成5秒收集一次、每次停顿70毫秒。停顿时间的确在下降，但吞吐量也降下来了。
>
> -XX：GCTimeRatio参数的值则应当是一个大于0小于100的整数，也就是垃圾收集时间占总时间的比率，相当于吞吐量的倒数。譬如把此参数设置为19，那允许的最大垃圾收集时间就占总时间的5%，即1/(1+19)），默认值为99，即允许最大1%（即1/(1+99)）的垃圾收集时间。

Parallel Scavenge/Parallel Old收集器运行示意图：

![Parallel Scavenge/Parallel Old收集器运行示意图](https://cos.duktig.cn/typora/202110021732331.png)

### Serial Old收集器

Serial Old是Serial收集器的老年代版本。

老年代 单线程   整理算法

**使用场景**：

这个收集器的主要意义也是供客户端模式下的HotSpot虚拟机使用。

如果在服务端模式下，它也可能有两种用途：

1. 在JDK 5以及之前的版本中与Parallel Scavenge收集器搭配使用[1]，
2. 作为CMS收集器发生失败时的后备预案，在并发收集发生Concurrent Mode Failure时使用。



### Parallel Old收集器

Parallel Old是Parallel Scavenge收集器的老年代版本。

老年代 多线程   整理算法

直到Parallel Old收集器出现后，“吞吐量优先”收集器终于有了比较名副其实的搭配组合，**在注重吞吐量或者处理器资源较为稀缺的场合，都可以优先考虑Parallel Scavenge / Parallel Old收集器这个组合**。



### CMS收集器

> **CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器**。这是因为CMS收集器工作时，GC工作线程与用户线程可以`并发`执行，以此来达到降低收集停顿时间的目的。

使用场景：集中在 **互联网网站或者基于浏览器的B/S系统的服务端** 上，这类应用通常都会较为关注服务的响应速度，希望系统停顿时间尽可能短，以给用户带来良好的交互体验。

CMS收集器仅作用于**老年代**的收集，是基于`标记-清除算法`的，它的运作过程分为4个步骤：

1. 初始标记（CMS initial mark）：仅仅只是**标记一下GC Roots能直接关联到的对象**，速度很快，需要“Stop The World”。
2. 并发标记（CMS concurrent mark）：是**从GC Roots的直接关联对象开始遍历整个对象图的过程**，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。
3. 重新标记（CMS remark）：为了**修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录**，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。此阶段也需要“Stop The World”。
4. 并发清除（CMS concurrent sweep）：**清理删除掉标记阶段判断的已经死亡的对象**。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

> CMS以流水线方式拆分了收集周期，将耗时长的操作单元保持与应用线程并发执行。只将那些必需STW才能执行的操作单元单独拎出来，控制这些单元在恰当的时机运行，并能保证仅需短暂的时间就可以完成。这样，在整个收集周期内，只有**两次短暂的暂停（初始标记和重新标记）**，**达到了近似并发的目的**。

由于整个过程中耗时最长的并发标记和并发清除过程收集器线程都可以与用户线程一起工作。所以，从总体上来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。通过下图可以比较清楚地看到CMS收集器的运作步骤中并发和需要停顿的时间：

![Concurrent Mark Sweep收集器运行示意图](https://cos.duktig.cn/typora/202110021739818.png)

CMS收集器**优点**：并发收集、低停顿。

CMS收集器**缺点**：

- **CMS收集器对CPU资源非常敏感**。面向并发设计的程序都对CPU资源比较敏感。在并发阶段，它虽然不会导致用户线程停顿，但会因为占用了一部分线程（或者说CPU资源）而导致应用程序变慢，总吞吐量会降低。
- **CMS收集器无法处理浮动垃圾（Floating Garbage）**。
  - 由于CMS并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生。这一部分垃圾出现在标记过程之后，CMS无法再当次收集中处理掉它们，只好留待下一次GC时再清理掉。
  - 这一部分垃圾就被称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用。
- **CMS收集器是基于标记-清除算法，收集结束时可能存在大量空间碎片**。碎片过多时，将会给大对象分配带来很大麻烦，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象（所以未提供CMS的新生代版本）。



### Garbage First收集器 (G1)

#### G1简介

Garbage First（简称G1）收集器是垃圾收集器技术发展历史上的里程碑式的成果，它开创了**收集器面向局部收集的设计思路**和**基于Region的内存布局形式**。

G1是一款主要面向服务端应用的垃圾收集器。

G1 GC切分堆内存为多个区间（Region），从而避免很多GC操作在整个Java堆或者整个年轻代进行。G1 GC只关注你有没有存货对象，都会被回收并放入可用的Region队列。G1 GC是基于Region的GC，**适用于大内存机器。即使内存很大，Region扫描，性能还是很高的**。

> JDK 9发布之日，G1宣告取代Parallel Scavenge加Parallel Old组合，成为服务端模式下的默认垃圾收集器，而CMS则沦落至被声明为不推荐使用（Deprecate）的收集器。如果对JDK 9及以上版本的HotSpot虚拟机使用参数 `-XX：+UseConcMarkSweepGC` 来开启CMS收集器的话，用户会收到一个警告信息，提示CMS未来将会被废弃：
>
> ```java
> Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely
> ```

#### G1特点

- **并行与并发**：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短Stop-the-world停顿的时间，部分其他收集器原来需要停顿Java线程执行的GC操作，G1收集器仍然可以通过**并发**的方式让Java程序继续运行。
- 分代收集：G1能够自己管理不同分代内已创建对象和新对象的收集。
- 空间整合：与CMS的标记-清除算法不同，G1从整体来看是基于**标记-整理算法**实现的收集器，从局部（两个Region之间）上来看是基于“**复制**”算法实现的。但无论如何，这两种算法都意味着G1运作期间不会产生内存空间碎片，收集后能提供规整的可用内存。**这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC**。
- 可预测的停顿：这是G1相对于CMS的一个优势，降低停顿时间是G1和CMS共同的关注点。

#### **G1的目标：**

作为CMS收集器的替代者和继承人，设计者们希望做出一款能够建立起**“停顿时间模型”**（PausePrediction Model）的收集器，**停顿时间模型的意思是能够支持指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间大概率不超过N毫秒这样的目标**，这几乎已经是实时Java（RTSJ）的中软实时垃圾收集器特征了。

#### **怎么做才能实现G1的目标呢？**

1、首先要有一个思想上的改变

在G1收集器出现之前的所有其他收集器，包括CMS在内，垃圾收集的目标范围要么是整个新生代（Minor GC），要么就是整个老年代（Major GC），再要么就是整个Java堆（Full GC）。

而G1跳出了这个樊笼，它可以**面向堆内存任何部分来组成回收集（Collection Set，一般简称CSet）进行回收**，衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是G1收集器的**Mixed GC模式**。

2、G1开创的**基于Region的堆内存布局**是它能够实现这个目标的关键

虽然G1也仍是遵循分代收集理论设计的，但其堆内存的布局与其他收集器有非常明显的差异：

**G1不再坚持固定大小以及固定数量的分代区域划分，而是把连续的Java堆划分为多个大小相等的独立区域（Region），每一个Region都可以根据需要，扮演新生代的Eden空间、Survivor空间，或者老年代空间**。收集器能够对扮演不同角色的Region采用不同的策略去处理，这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果。

Region中还有一类特殊的Humongous区域，专门用来存储大对象。**G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象**。

每个Region的大小可以通过参数`-XX：G1HeapRegionSize`设定，取值范围为1MB～32MB，且应为2的N次幂。而对于那些超过了整个Region容量的超级大对象，将会被存放在N个连续的Humongous Region之中，G1的大多数行为都把Humongous Region作为老年代的一部分来进行看待。

**3、建立可预测的停顿时间模型**

将Region作为单次回收的最小单元，即每次收集到的内存空间都是Region大小的整数倍，这样可以**有计划地避免在整个Java堆中进行全区域的垃圾收集**。

具体的处理思路是让G1收集器去跟踪各个Region里面的垃圾堆积的“价值”大小，价值即回收所获得的空间大小以及回收所需时间的经验值，然后在后台维护一个优先级列表，每次根据用户设定允许的收集停顿时间（使用参数-XX：MaxGCPauseMillis指定，默认值是200毫秒），优先处理回收价值收益最大的那些Region，这也就是“Garbage First”名字的由来。这种使用Region划分内存空间，以及具有优先级的区域回收方式，保证了G1收集器在有限的时间内获取尽可能高的收集效率。

#### G1收集器Region分区示意图

![G1收集器Region分区示意图](https://cos.duktig.cn/typora/202110021758946.png)

#### G1将堆内存“化整为零”的“解题思路”，遇到的难题

从2004年Sun实验室发表第一篇关于G1的论文后一直拖到2012年4月JDK 7 Update 4发布，用将近10年时间才倒腾出能够商用的G1收集器来。G1收集器至少有（不限于）以下这些关键的细节问题需要妥善解决：

**1、将Java堆分成多个独立Region后，Region里面存在的跨Region引用对象如何解决？**

使用记忆集避免全堆作为GC Roots扫描，但在G1收集器上记忆集的应用复杂很多。

它的每个Region都维护有自己的记忆集，这些记忆集会记录下别的Region指向自己的指针，并标记这些指针分别在哪些卡页的范围之内。

G1的记忆集在存储结构的本质上是一种**哈希表**，**Key是别的Region的起始地址，Value是一个集合，里面存储的元素是卡表的索引号**。这
种“双向”的卡表结构（卡表是“我指向谁”，这种结构还记录了“谁指向我”）比原来的卡表实现起来更复杂，同时由于Region数量比传统收集器的分代数量明显要多得多，因此**G1收集器要比其他的传统垃圾收集器有着更高的内存占用负担**。

根据经验，**G1至少要耗费大约相当于Java堆容量10%至20%的额外内存来维持收集器工作**。

**2、在并发标记阶段如何保证收集线程与用户线程互不干扰地运行？**

**首先要解决的是用户线程改变对象引用关系时，必须保证其不能打破原本的对象图结构，导致标记结果出现错误。**

解决办法：

CMS收集器采用增量更新算法实现，而G1收集器则是通过原始快照（SATB）算法来实现的。

**回收过程中新创建对象的内存分配上，程序要继续运行会持续创建新对象。**

解决办法：

G1为每一个Region设计了两个名为TAMS（Top at Mark Start）的指针，把Region中的一部分空间划分出来用于并发回收过程中的新对象分配，并发回收时新分配的对象地址都必须要在这两个指针位置以上。G1收集器默认在这个地址以上的对象是被隐式标记过的，即默认它们是存活的，不纳入回收范围。与CMS中的“Concurrent Mode Failure”失败会导致Full GC类似，如果内存回收的速度赶不上内存分配的速度，**G1收集器也要被迫冻结用户线程执行，导致Full GC而产生长时间“Stop The World”**。

**3、怎样建立起可靠的停顿预测模型？**

用户通过 `-XX：MaxGCPauseMillis` 参数指定的停顿时间只意味着垃圾收集发生之前的期望值，但G1收集器要怎么做才能满足用户的期望呢？

G1收集器的停顿预测模型是以**衰减均值**（Decaying Average）为理论基础来实现的。

在垃圾收集过程中，G1收集器会记录每个Region的回收耗时、每个Region记忆集里的脏卡数量等各个可测量的步骤花费的成本，并分析得出平均值、标准偏差、置信度等统计信息。这里强调的“衰减平均值”是指它会比普通的平均值更容易受到新数据的影响，平均值代表整体平均状态，但衰减平均值更准确地代表“最近的”平均状态。

换句话说，**Region的统计状态越新越能决定其回收的价值**。然后通过这些信息预测现在开始回收的话，**由哪些Region组成回收集才可以在不超过期望停顿时间的约束下获得最高的收益**。

#### G1的运作过程

如果我们不去计算用户线程运行过程中的动作（如使用写屏障维护记忆集的操作），G1收集器运作过程大致可划分为以下四个步骤：

1. **初始标记（Initial Marking）**：**仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS指针的值，让下一阶段用户线程并发运行时，能正确地在可用的Region中分配新对象**。这个阶段需要停顿线程，但耗时很短，而且是借用进行Minor GC的时候同步完成的，所以G1收集器在这个阶段实际并没有额外的停顿。
2. **并发标记（Concurrent Marking）**：**从GC Root开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象**。这阶段耗时较长，但可与用户程序并发执行。当对象图扫描完成以后，还要重新处理SATB记录下的在并发时有引用变动的对象。
3. **最终标记（Final Marking）**：**对用户线程做另一个短暂的暂停，用于处理并发阶段结束后仍遗留下来的最后那少量的SATB记录**。
4. **筛选回收（Live Data Counting and Evacuation）**：**负责更新Region的统计数据，对各个Region的回收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，可以自由选择任意多个Region构成回收集，然后把决定回收的那一部分Region的存活对象复制到空的Region中，再清理掉整个旧Region的全部空间。**这里的操作涉及存活对象的移动，是必须暂停用户线程，由多条收集器线程并行完成的。

G1收集器运行示意图：

![G1收集器运行示意图](https://cos.duktig.cn/typora/202110021813564.png)

## *CMS和G1的区别

### 区别

**区别一： 使用范围不一样**

CMS收集器是老年代的收集器，可以配合新生代的Serial和ParNew收集器一起使用

G1收集器收集范围是老年代和新生代。不需要结合其他收集器使用

**区别二： STW的时间**

CMS收集器以最小的停顿时间为目标的收集器。

G1收集器可预测垃圾回收的停顿时间（建立可预测的停顿时间模型）

**区别三： 垃圾碎片**

CMS收集器是使用“标记-清除”算法进行的垃圾回收，容易产生内存碎片

G1收集器使用的是“标记-整理”算法，进行了空间整合，降低了内存空间碎片。

**区别四： 垃圾回收的过程不一样**

CMS收集器                      G1收集器

1. 初始标记                   1. 初始标记
2. 并发标记                   2. 并发标记
3. 重新标记                   3. 最终标记
4. 并发清除                   4. 筛选回收

### G1的优劣势分析

**G1的优势**：

1、最大停顿时间、分Region的内存布局、按收益动态确定回收集这些创新性设计带来的优势

2、算法层面的优势

与CMS的“标记-清除”算法不同，G1从整体来看是基于“标记-整理”算法实现的收集器，但从局部（两个Region之间）上看又是基于“标记-复制”算法实现，无论如何，这两种算法都意味着**G1运作期间不会产生内存空间碎片，垃圾收集完成之后能提供规整的可用内存**。

这种特性有利于程序长时间运行，在程序为大对象分配内存时不容易因无法找到连续内存空间而提前触发下一次收集

**G1的弱势**：

在用户程序运行过程中，G1无论是为了垃圾收集产生的内存占用（Footprint）还是程序运行时的额外执行负载（Overload）都要比CMS要高。

> **就内存占用来说**，虽然G1和CMS都使用卡表来处理跨代指针，但G1的卡表实现更为复杂，而且堆中每个Region，无论扮演的是新生代还是老年代角色，都必须有一份卡表，这导致G1的记忆集（和其他内存消耗）可能会占整个堆容量的20%乃至更多的内存空间；相比起来CMS的卡表就相当简单，只有唯一一份，而且只需要处理老年代到新生代的引用，反过来则不需要，由于新生代的对象具有朝生夕灭的不稳定性，引用变化频繁，能省下这个区域的维护开销是很划算的。
>
> **在执行负载的角度上**，同样由于两个收集器各自的细节实现特点导致了用户程序运行时的负载会有不同，譬如它们都使用到写屏障，CMS用写后屏障来更新维护卡表；而G1除了使用写后屏障来进行同样的（由于G1的卡表结构复杂，其实是更烦琐的）卡表维护操作外，为了实现原始快照搜索（SATB）算法，还需要使用写前屏障来跟踪并发时的指针变化情况。相比起增量更新算法，原始快照搜索能够减少并发标记和重新标记阶段的消耗，避免CMS那样在最终标记阶段停顿时间过长的缺点，但是在用户程序运行过程中确实会产生由跟踪引用变化带来的额外负担。由于G1对写屏障的复杂操作要比CMS消耗更多的运算资源，所以CMS的写屏障实现是直接的同步操作，而G1就不得不将其实现为类似于消息队列的结构，把写前屏障和写后屏障中要做的事情都放到队列里，然后再异步处理。



**总览**

|                      | CMS                          | G1                                                           |
| :------------------- | ---------------------------- | ------------------------------------------------------------ |
| JDK版本              |                              |                                                              |
| 回收算法             | 标记—清除                    | 标记—整理（标记—复制）                                       |
| 运行环境             | 针对70G以内的堆内存          | 可针对好几百G的大内存                                        |
| 回收区域             | 老年代                       | 新生代和老年代                                               |
| 内存布局             | 传统连续的新生代和老年代区域 | Region(将新生代和老年代切分成Region，默认一个Region 1 M,默认2048块)<br/>MIN_REGION_SIZE：允许的最小的REGION_SIZE，即1M，不可能比1M还小；<br/>MAX_REGION_SIZE：允许的最大的REGION_SIZE，即32M，不可能比32M更大；<br/>限制最大REGION_SIZE是为了考虑GC时的清理效果； |
| 浮动垃圾             | 是                           | 否                                                           |
| 内存碎片             | 是                           | 否                                                           |
| 全堆扫描             | 是                           | 否                                                           |
| 回收时间可控         | 否                           | 是                                                           |
| 对象进入老年代的年龄 |                              |                                                              |
| 空间动态调整         | 否                           | 是（新生代5%-60%动态调整，一般不需求指定）                   |
| 调优参数             | 多（近百个）                 | 少（十几个）                                                 |

## 低延迟的垃圾收集器

垃圾收集器的三项重要指标：**内存占用、吞吐量、延迟**。

**内存的扩大，对低延迟反而会带来负面效果**：虚拟机要回收完整的1TB的堆内存，要比回收1GB的堆内存耗费更多的时间。

> CMS和G1分别使用增量更新和原始快照技术，实现了标记阶段的并发，不会因管理的堆内存变大，要标记的对象变多而导致停顿时间随之增长。但是对于标记阶段之后的处理，仍未得到妥善解决。
>
> CMS使用标记-清除算法，虽然避免了整理阶段收集器带来的停顿，但是清除算法不论如何优化改进，在设计原理上避免不了空间碎片的产生，随着空间碎片不断淤积最终依然逃不过“Stop TheWorld”的命运。
>
> G1虽然可以按更小的粒度进行回收，从而抑制整理阶段出现时间过长的停顿，但毕竟也还是要暂停的。

### Shenandoah收集器

Shenandoah是一款只有OpenJDK才会包含，而OracleJDK里反而不存在的收集器。

目标是实现一种能在任何堆内存大小下都可以把垃圾收集的停顿时间限制在十毫秒以内的垃圾收集器，该目标意味着相比CMS和G1，Shenandoah不仅要进行并发的垃圾标记，还要并发地进行对象清理后的整理动作。

### ZGC收集器

> ZGC（“Z”并非什么专业名词的缩写，这款收集器的名字就叫作Z Garbage Collector）是一款在JDK 11中新加入的具有实验性质[1]的低延迟垃圾收集器，是由Oracle公司研发的。2018年Oracle创建了JEP 333将ZGC提交给OpenJDK，推动其进入OpenJDK 11的发布清单之中。

ZGC和Shenandoah的目标是高度相似的，都希望在**尽可能对吞吐量影响不太大的前提下，实现在任意堆内存大小下都可以把垃圾收集的停顿时间限制在十毫秒以内的低延迟**。

## 如何选择垃圾收集器？

### 主要因素

1. 应用程序的主要关注点是什么？
   1. 如果是数据分析、科学计算类的任务，目标是能尽快算出结果，那吞吐量就是主要关注点；
   2. 如果是SLA应用，那停顿时间直接影响服务质量，严重的甚至会导致事务超时，这样延迟就是主要关注点；
   3. 如果是客户端应用或者嵌入式应用，那垃圾收集的内存占用则是不可忽视的。
2. 运行应用的基础设施如何？
3. 使用JDK的发行商是什么？版本号是多少？

### 如何选择？

> 使用ParNew（标记复制、并行、作用于新生代） + CMS的垃圾收集器（标记清除、并行、作用于老年代），追求响应速度优先，其适用于多CPU环境的Server模式的互联网或者B/S业务。
>
> 如果追求吞吐量优先，应用在后台运算并不需要太多交互场景的，可采用Parallel（标记复制、并行、作用于新生代） + Parallel Old 的垃圾收集器（标记整理、并行、作用于老年代）

如果是直接面向用户提供服务的B/S系统，延迟时间是主要关注点。

- 有充足的预算但没有太多调优经验，可以选择商业性解决方案，Zing VM可以使用C4收集器
- 能够掌控软硬件型号，使用较新的版本，同时又特别注重延迟，那ZGC很值得尝试
- 遗留系统，软硬件基础设施和JDK版本都比较落后，对于大概4GB到6GB以下的堆内存，CMS一般能处理得比较好，而对于更大的堆内存，可重点考察一下G1。

如果是面向计算，没有太多交互，注重高吞吐量，可以使用Java8默认提供的垃圾收集器——Parallel Scavenge收集器和Parallel Old。

# 内存分配与回收策略

内存分配与回收策略（即垃圾回收过程）

- **对象优先在Eden分配**。大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。
- **大对象直接进入老年代**。大对象就是指需要大量连续内存空间的Java对象，最典型的大对象便是那种很长的字符串，或者元素数量很庞大的数组。这样做的目的就是避免在Eden区及两个Survivor区之间来回复制，产生大量的内存复制操作。
- **长期存活的对象将进入老年代**。
  - HotSpot虚拟机中多数收集器都采用了分代收集来管理堆内存，那内存回收时就必须能决策哪些存活对象应当放在新生代，哪些存活对象放在老年代中。为做到这点，虚拟机给每个对象定义了一个对象年龄（Age）计数器，存储在对象头中（详见第2章）。对象通常在Eden区里诞生，如果经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，该对象会被移动到Survivor空间中，并且将其对象
    年龄设为1岁。对象在Survivor区中每熬过一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15），就会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数`-XX：MaxTenuringThreshold`设置。
- **动态对象年龄判定**。
  - 为了能更好地适应不同程序的内存状况，HotSpot虚拟机并不是永远要求对象的年龄必须达到`-XX：MaxTenuringThreshold`才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到`-XX：MaxTenuringThreshold`中要求的年龄。
- **空间分配担保**。
  - 在发生Minor GC之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次Minor GC可以确保是安全的。如果不成立，则虚拟机会先查看`-XX：HandlePromotionFailure`参数的设置值是否允许担保失败（Handle Promotion Failure）；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者`-XX：HandlePromotionFailure`设置不允许冒险，那这时就要改为进行一次Full GC。

（参考：深入理解Java虚拟机第三版的3.8节）

关于垃圾收集机制的详细内容可参考：[JVM垃圾回收机制](https://www.cnblogs.com/hexinwei1/p/9525737.html)

# 类加载过程

系统加载 Class 类型的文件主要三步：**加载->连接->初始化**。连接过程又可分为三步：**验证->准备->解析**。

如下图：

![类加载过程](https://gitee.com/koala010/typora/raw/master/img/20210701103844.png)

## 加载

在加载阶段，JVM主要完成下面三件事情：

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口

虚拟机规范上面这 3 点并不具体，因此是非常灵活的。比如："通过全类名获取定义此类的二进制字节流" 并没有指明具体从哪里获取、怎样获取。比如：比较常见的就是从 `ZIP` 包中读取（日后出现的 `JAR`、`EAR`、`WAR` 格式的基础）、其他文件生成（典型应用就是 `JSP`）等等。

**一个非数组类的加载阶段（加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，这一步可以去完成，还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的 `loadClass()` 方法）。数组类型不通过类加载器创建，它由 Java 虚拟机直接创建。**

加载阶段和连接阶段的部分内容是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了。

> 说明：除了在加载阶段用户应用程序可以通过自定义类加载器的方式局部参与外，其余动作都完全由Java虚拟机来主导控制。

## 连接

#### 验证

##### 1、文件格式验证

主要验证内容：

- 是否以魔数`0xCAFEBABE`开头。
- 主、次版本号是否在当前Java虚拟机接受范围之内。
- 常量池的常量中是否有不被支持的常量类型（检查常量tag标志）。
- 指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量。
- `CONSTANT_Utf8_info`型的常量中是否有不符合UTF-8编码的数据。
- Class文件中各个部分及文件本身是否有被删除的或附加的其他信息。
- ……

该验证阶段的主要目的是**保证输入的字节流能正确地解析并存储于方法区之内，格式上符合描述一个Java类型信息的要求**。这阶段的验证是基于二进制字节流进行的，只有通过了这个阶段的验证之后，这段字节流才被允许进入Java虚拟机内存的方法区中进行存储，所以后面的三个验证阶段全部是基于方法区的存储结构上进行的，不会再直接读取、操作字节流了。

##### 2、元数据验证

这个阶段是**对字节码描述的信息进行语义分析**。主要验证内容：

- 这个类是否有父类（除了java.lang.Object之外，所有的类都应当有父类）。
- 这个类的父类是否继承了不允许被继承的类（被final修饰的类）。
- 如果这个类不是抽象类，是否实现了其父类或接口之中要求实现的所有方法。
- 类中的字段、方法是否与父类产生矛盾（例如覆盖了父类的final字段，或者出现不符合规则的方法重载，例如方法参数都一致，但返回值类型却不同等）。

##### 3、字节码验证

第三阶段是整个验证过程中最复杂的一个阶段，主要目的是**通过数据流分析和控制流分析，确定程序语义是合法的、符合逻辑的**。主要验证内容：

- 保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作，例如不会出现类似于“在操作栈放置了一个int类型的数据，使用时却按long类型来加载入本地变量表中”这样的情况。
- 保证任何跳转指令都不会跳转到方法体以外的字节码指令上。
- 保证方法体中的类型转换总是有效的，例如可以把一个子类对象赋值给父类数据类型，这是安全的，但是把父类对象赋值给子类数据类型，甚至把对象赋值给与它毫无继承关系、完全不相干的一个数据类型，则是危险和不合法的。

元数据验证段对元数据信息中的数据类型校验完毕以后，这阶段就要对类的方法体（Class文件中的Code属性）进行校验分析，保证被校验类的方法在运行时不会做出危害虚拟机安全的行为。

##### 4、符号引用验证

校验行为发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在连接的第三阶段——解析阶段中发生。目的：**确保解析动作正确执行**。主要验证内容：

- 符号引用中通过字符串描述的全限定名是否能找到对应的类。
- 在指定类中是否存在符合方法的字段描述符及简单名称所描述的方法和字段。
- 符号引用中的类、字段、方法的可访问性（`private`、`protected`、`public`、`<package>`）是否可被当前类访问

#### 准备

**准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段**。这一阶段并不包括实例变量，实例变量的初始化随着对象一块分配在Java堆中。

> 从概念上讲，这些变量所使用的内存都应当在方法区中进行分配，但必须注意到方法区本身是一个逻辑上的区域，在JDK 7及之前，HotSpot使用永久代来实现方法区时，实现是完全符合这种逻辑概念的；而在JDK 8及之后，类变量则会随着Class对象一起存放在Java堆中，这时候“类变量在方法区”就完全是一种对逻辑概念的表述了。

**这里所设置的初始值"通常情况"下是数据类型默认的零值**（如 0、0L、null、false 等），比如我们定义了public static int value=111 ，那么 value 变量在准备阶段的初始值就是 0 而不是 111（初始化阶段才会赋值）。特殊情况：比如给 value 变量加上了 final 关键字public static final int value=111 ，那么准备阶段 value 的值就被赋值为 111。

#### 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符 7 类符号引用进行。

> 符号引用就是一组符号来描述目标，可以是任何字面量。

> **直接引用**就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。在程序实际运行时，只有符号引用是不够的，举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用。

综上，解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，也就是得到类或者字段、方法在内存中的指针或者偏移量。

对类/接口、字段、方法、接口方法的解析过程，可参看《深入理解Java虚拟机（第3版）》。

## 初始化

初始化阶段是执行初始化方法 `<clinit> ()`方法的过程，是类加载的最后一步，这一步 JVM 才开始真正执行类中定义的 Java 程序代码(字节码)。

> 说明： `<clinit> ()`方法是编译之后自动生成的。

对于`<clinit> ()` 方法的调用，虚拟机会自己确保其在多线程环境中的安全性。因为 `<clinit> ()` 方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起多个进程阻塞，并且这种阻塞很难被发现。

对于初始化阶段，虚拟机严格规范了有且只有 以下情况下，必须对类进行初始化(只有主动去使用类才会初始化类)：

1. 当遇到 new 、 getstatic、putstatic 或 invokestatic 这 4 条直接码指令时，比如 new 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。

   当 jvm 执行 new 指令时会初始化类。即当程序创建一个类的实例对象。

   当 jvm 执行 getstatic 指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。

   当 jvm 执行 putstatic 指令时会初始化类。即程序给类的静态变量赋值。

   当 jvm 执行 invokestatic 指令时会初始化类。即程序调用类的静态方法。

2. 使用 java.lang.reflect 包的方法对类进行反射调用时如 Class.forname("..."), newInstance() 等等。如果类没初始化，需要触发其初始化。

3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。

4. 当虚拟机启动时，用户需要定义一个要执行的主类 (包含 main 方法的那个类)，虚拟机会先初始化这个类。

5. MethodHandle 和 VarHandle 可以看作是轻量级的反射调用机制，而要想使用这 2 个调用， 就必须先使用 findStaticVarHandle 来初始化要调用的类。

6. 当一个接口中定义了 JDK8 新加入的默认方法（被 default 关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。

## 补充：卸载

卸载类即该类的 Class 对象被 GC。

卸载类需要满足 3 个要求:

1. 该类的所有的实例对象都已被 GC，也就是说堆不存在该类的实例对象。
2. 该类没有在其他任何地方被引用
3. 该类的类加载器的实例已被 GC

所以，在 JVM 生命周期内，由 jvm 自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。

只要想通一点就好了，jdk 自带的 `BootstrapClassLoader`, `ExtClassLoader`, `AppClassLoader` 负责加载 jdk 提供的类，所以它们(类加载器的实例)肯定不会被回收。而我们自定义的类加载器的实例是可以被回收的，所以使用我们自定义加载器加载的类是可以被卸载掉的。



# 类加载器和双亲委派模型

所有的类都由类加载器加载，加载的作用就是将 .class文件加载到内存。

> Java虚拟机设计团队有意把类加载阶段中的“通过一个类的全限定名来获取描述该类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需的类。实现这个动作的代码被称为“类加载器”（Class Loader）。

每一个类加载器，都拥有一个独立的类名称空间比较两个类是否“相等”。只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个Java虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。

这里所指的“相等”，包括代表类的Class对象的`equals()`方法、`isAssignableFrom()`方法、`isInstance()`方法的返回结果，也包括了使用`instanceof`关键字做对象所属关系判定等各种情况。如果没有注意到类加载器的影响，在某些情况下可能会产生具有迷惑性的结果，

## 类加载器总结

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1. **BootstrapClassLoader(启动类加载器)** ：最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。
2. **ExtensionClassLoader(扩展类加载器)** ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。
3. **AppClassLoader(应用程序类加载器)** ：面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。

## 双亲委派模型

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。

### **双亲委派模型的工作过程**

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

![双亲委派模型](https://gitee.com/koala010/typora/raw/master/img/20210701154740.png)

### 好处

使用双亲委派模型来组织类加载器之间的关系，一个显而易见的好处就是Java中的类随着它的类加载器一起具备了一种带有优先级的层次关系。

双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类。

### 如何破坏双亲委派模型？

自定义加载器的话，需要继承 `ClassLoader` 。如果我们不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写 `loadClass()` 方法

### 自定义类加载器

除了 `BootstrapClassLoader` 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`。如果我们要自定义自己的类加载器，很明显需要继承 `ClassLoader`。



