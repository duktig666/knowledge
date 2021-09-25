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

