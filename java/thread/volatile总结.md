## 1. 相关概念

### 1.1 CPU缓存模型

#### 1.1.1 为什么要有CPU高速缓存？

**CPU 缓存则是为了解决 CPU 处理速度和内存处理速度不对等的问题**。

就像我们平时使用的缓存（redis），是为了解决程序处理速度和访问关系型数据库速度不对等的问题。

也可以类比外存，**内存可以看作外存的高速缓存** ，程序运行的时候我们把外存的数据复制到内存，由于内存的处理速度远远高于外存，这样提高了处理速度。

**总结**：

**CPU Cache 缓存的是内存数据用于解决 CPU 处理速度和内存不匹配的问题，内存缓存的是硬盘数据用于解决硬盘访问速度过慢的问题。**

#### 1.1.2 CPU高速缓存的工作方式

![CPU缓存模型](https://cos.duktig.cn/2021/06/18/1623987055.png)

先复制一份数据到 CPU Cache 中，当 CPU 需要用到的时候就可以直接从 CPU Cache 中读取数据，当运算完成后，再将运算得到的数据写回 Main Memory 中。

但是，这样存在 **内存缓存不一致性的问题** ！比如执行一个 i++操作的话，两个线程同时执行的话，假设两个线程从 CPU Cache 中读取的 i=1，两个线程做了 1++运算完之后再写回 Main Memory 之后 i=2，而正确结果应该是 i=3。

**CPU 为了解决内存缓存不一致性问题可以通过制定缓存一致协议或者其他手段来解决。**

### 1.2 CPU的术语定义

![CPU的术语定义](https://cos.duktig.cn/typora/202110122108366.png)

### 1.3 JMM（Java内存模型）

> JMM(Java 内存模型 Java Memory Model，简称 JMM)本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（实例字段，静态字段和构成数组对象的元素）的访问方式。

在 JDK1.2 之前，Java 的内存模型实现总是从**主存** （即共享内存）读取变量，是不需要进行特别的注意的。而在当前的 Java 内存模型下，线程可以把变量保存**本地内存** （比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成**数据的不一致** 。

![使用volatile前变量读取](https://cos.duktig.cn/2021/06/18/1624000135.png)

要解决这个问题，就需要把变量声明为 **`volatile`** ，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

所以，**`volatile` 关键字 除了防止 JVM 的指令重排 ，还有一个重要的作用就是保证变量的可见性。**

![使用volatile后变量读取](https://cos.duktig.cn/2021/06/18/1624000201.png)

JMM 关于同步的规定：

1. 线程解锁前，必须把共享变量的值刷新回主内存。
2. 线程加锁前，必须读取主内存的最新值到自己的工作内存。
3. 加锁解锁是同一把锁。

**JMM 可以保证：可见性、原子性、有序性**。

### 1.4 并发编程的三个重要特性

1. 原子性：一个或者多次操作，要么全部都执行，不会收到任何因素的干扰而中断；要么全部都不执行。
2. 可见性：当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。
3. 有序性：代码在执行的过程当中先后有序。Java在编译器及运行期间的优化，代码的执行顺序未必就是编写代码时候的顺序。



## 2. `volatile`关键字

### 2.1 `volatile`的定义

> 关键字volatile可以用来修饰字段（成员变量），就是告知程序任何对该变量的访问均需要从共享内存中获取，而对它的改变必须同步刷新回共享内存，它能保证所有线程对变量访问的可见性。

而关键字synchronized可以修饰方法或者以同步块的形式来进行使用，它主要确保多个线程在同一个时刻，只能有一个线程处于方法或者同步块中，它保证了线程对变量访问的可见性和排他性。

### 2.2 实现原理

volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为[font color="red"]它不会引起线程上下文的切换和调度[/font]。

有volatile变量修饰的共享变量进行写操作的时候会多出第二行汇编代码，Lock前缀的指令在多核处理器下会引发了两件事情。

1. **将当前处理器缓存行的数据写回到系统内存**。
2. **这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效**。

所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

### 2.3 `volatile`的特点

#### 2.3.1 保证可见性

```java
public class JMMDemo01 {

    // 如果不加volatile 程序会死循环
    // 加了volatile是可以保证可见性的
    private volatile static Integer number = 0;

    public static void main(String[] args) {
        //main线程
        //子线程1
        new Thread(()->{
            while (number==0){
            }
        }).start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //子线程2
        new Thread(()->{
            while (number==0){
            }
        }).start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        number=1;
        System.out.println(number);
    }
}
```

#### 2.3.2 不保证原子性

```java
/**
 * 不保证原子性
 * number <=2w
 * 
 */
public class VDemo02 {

    private static volatile int number = 0;

    public static void add(){
        number++; 
        //++ 不是一个原子性操作，是两个~3个操作
        //
    }

    public static void main(String[] args) {
        //理论上number  === 20000

        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                for (int j = 1; j <= 1000 ; j++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount()>2){
            //main  gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+",num="+number);
    }
}
```

**执行五次main方法结果：**

```
main,num=19052
 main,num=20000
 main,num=20000
 main,num=19722
 main,num=20000
```

number++不是一个操作，而是三个操作，所以不能保证原子性：

1. get取到number的值
2. 进行+1操作
3. put写number的值

即理论值应该是20000，但可能达不到。有一定的概率不能保证数据的正确性。

**如果不加lock和synchronized ，怎么样保证原子性？**

解决方法：使用JUC下的java.util.concurrent.atomic包下的class；即使用原子类。

![原子类的位置](https://gitee.com/koala010/typora/raw/master/img/image-20200814095936365.png)

使用原子类后的操作：

```java
public class VDemo02 {

    private static volatile AtomicInteger number = new AtomicInteger();

    public static void add(){
//        number++;
        number.incrementAndGet();  //底层是CAS保证的原子性
    }

    public static void main(String[] args) {
        //理论上number  === 20000

        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                for (int j = 1; j <= 1000 ; j++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount()>2){
            //main  gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+",num="+number);
    }
}
```

这些类的底层都直接和操作系统挂钩！是在内存中修改值。

Unsafe类是一个很特殊的存在。

![image-20200814100351230](https://gitee.com/koala010/typora/raw/master/img/image-20200814100351230.png)

#### 2.3.3 禁止指令重排（有序性）

我们写的程序，计算机并不是按照我们自己写的那样去执行的

源代码–>编译器优化重排–>指令并行也可能会重排–>内存系统也会重排–>执行

**处理器在进行指令重排的时候，会考虑数据之间的依赖性！**

```java
int x=1; //1
int y=2; //2
x=x+5;   //3
y=x*x;   //4

//我们期望的执行顺序是 1_2_3_4  可能执行的顺序会变成2134 1324
//可不可能是 4123？ 不可能的
1234567
```

可能造成的影响结果：

前提：a b x y这四个值 默认都是0

| 线程A | 线程B |
| ----- | ----- |
| x=a   | y=b   |
| b=1   | a=2   |

正常的结果： x = 0; y =0;

| 线程A | 线程B |
| ----- | ----- |
| x=a   | y=b   |
| b=1   | a=2   |

可能在线程A中会出现，先执行b=1,然后再执行x=a；

在B线程中可能会出现，先执行a=2，然后执行y=b；

那么就有可能结果如下：x=2; y=1.

**volatile可以避免指令重排：**

**volatile中会加一道内存的屏障，这个内存屏障可以保证在这个屏障中的指令顺序。**

内存屏障：CPU指令。作用：

1、保证特定的操作的执行顺序；

2、可以保证某些变量的内存可见性（利用这些特性，就可以保证volatile实现的可见性）

![内存屏障](https://gitee.com/koala010/typora/raw/master/img/image-20200814100613459.png)

## 3. `volatile`优化

JDK 7的并发包里新增一个队列集合类 `LinkedTransferQueue` ，它在使用 `volatile` 变量时，用一种追加字节的方式来优化队列出队和入队的性能。LinkedTransferQueue的代码如下：

```java
/** 队列中的头部节点 */
private transient f?inal PaddedAtomicReference<QNode> head;
/** 队列中的尾部节点 */
private transient f?inal PaddedAtomicReference<QNode> tail;
static f?inal class PaddedAtomicReference <T> extends AtomicReference T> {
     // 使用很多4个字节的引用追加到64个字节
     Object p0, p1, p2, p3, p4, p5, p6, p7, p8, p9, pa, pb, pc, pd, pe;
     PaddedAtomicReference(T r) {
        super(r);
     }
}
public class AtomicReference <V> implements java.io.Serializable {
     private volatile V value;
     // 省略其他代码
｝
```

**追加字节如何能优化性能？**

`LinkedTransferQueue ` 这个类，它使用一个内部类类型来定义队列的头节点（head）和尾节点（tail），而这个内部类`PaddedAtomicReference`相对于父类`AtomicReference`只做了一件事情，就是**将共享变量追加到64字节**。我们可以来计算下，一个对象的引用占4个字节，它追加了15个变量（共占60个字节），再加上父类的value变量，一共64个字节。

对于英特尔酷睿i7、酷睿、Atom和NetBurst，以及Core Solo和Pentium M处理器的L1、L2或L3缓存的高速缓存行是64个字节宽，不支持部分填充缓存行，**在多处理器下每个处理器都会缓存同样的头、尾节点，当一个处理器试图修改头节点时，会将整个缓存行锁定，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己高速缓存中的尾节点，而队列的入队和出队操作则需要不停修改头节点和尾节点，所以在多处理器的情况下将会严重影响到队列的入队和出队效率**。

**使用追加到64字节的方式来填满高速缓冲区的缓存行，避免头节点和尾节点加载到同一个缓存行，使头、尾节点在修改时不会互相锁定**。

**在两种场景下不应该使用volatile变量时追加到64字节**

[font color="red"]缓存行非64字节宽的处理器[/font]。如P6系列和奔腾处理器，它们的L1和L2高速缓存行是32个字节宽。

[font color="red"]共享变量不会被频繁地写[/font]。因为使用追加字节的方式需要处理器读取更多的字节到高速缓冲区，这本身就会带来一定的性能消耗，如果共享变量不被频繁写的话，锁的几率也非常小，就没必要通过追加字节的方式来避免相互锁定。





## 什么是volatile？

> 《Java语言规范第3版》中对volatile的定义如下：
>
> Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。

`volatile`是轻量级的`synchronized`，它在多处理器开发中保证了共享变量的“可见性”，禁止JVM的指令重排，但是不能保证原子性。

`volatile`比`synchronized`的使用和执行成本更低，因为它**不会引起线程上下文的切换和调度**。



## volatile的特性

可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。





## volatile 的实现原理？

### 可见性原理

有volatile变量修饰的共享变量进行写操作的时候会多出第二行汇编代码，Lock前缀的指令在多核处理器下会引发了两件事情。

1. **将当前处理器缓存行的数据写回到系统内存**。
2. **这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效**。

具体分析：

**问题1：何时将数据写到内存？**

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。

如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock**前缀的指令**，将这个变量所在缓存行的数据写回到系统内存。

**问题2：写到内存后其他线程的缓存数据还是旧的，如何解决？**

但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。

解决策略：

1. 在多处理器下，实现**缓存一致性协议**，**每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了**，当处理器发现自己缓存行对应的内存地址被修改，就会**将当前处理器的缓存行设置成无效状态**。
2. 当处理器对这个数据进行修改操作的时候，会**重新从系统内存中把数据读到处理器缓存里**。

### 禁止指令重排原理

禁止指令重排的情况：

- 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
- 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。
- 当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。

实现原理：

volatile通过添加读写屏障来禁止指令重排。

- 在每个volatile写操作的前后插入一个StoreStore屏障
- 在每个volatile读操作的后面插入一个LoadLoad屏障

![volatile 禁止指令重排的读写屏障原理](https://cos.duktig.cn/typora/202110131743529.png)











