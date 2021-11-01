> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。
>
> 本文相关源码参看：[https://github.com/duktig666/learn-example/tree/8f925bf26956ca13bb0c5f358177f4c8faab4498/java-core/src/main/java/thread](https://github.com/duktig666/learn-example/tree/8f925bf26956ca13bb0c5f358177f4c8faab4498/java-core/src/main/java/thread)

# Lock锁

## Lock锁的使用

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
} finally {
    lock.unlock();
}
```

在finally块中释放锁，目的是保证在获取到锁之后，最终能够被释放。

不要将获取锁的过程写在try块中，因为如果在获取锁（自定义锁的实现）时发生了异常，异常抛出的同时，也会导致锁无故释放。

## Lock接口提供的synchronized关键字所不具备的主要特性

![Lock接口提供的synchronized关键字所不具备的主要特性](https://cos.duktig.cn/typora/202110160911861.png)

`Lock`的主要API：

![Lock的主要API](https://cos.duktig.cn/typora/202110160912526.png)



# AQS

## 什么是AQS？

> 队列同步器AbstractQueuedSynchronizer，是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。
>
> **同步器的主要使用方式**：继承，子类通过继承同步器并实现它的抽象方法来管理同步状态，它**独占式**和**共享式**两种方式获取同步状态，多线程并发情况下通过CAS的方式修改同步状态。
>
> 同步器的设计基于**模板方法模式**：
>
> 1. 继承同步器并重写指定的方法
> 2. 将同步器组合在自定义同步组件的实现中
> 3. 调用同步器提供的模板方法
> 4. 模板方法将会调用使用者重写的方法

## 为什么要使用AQS？

锁是面向使用者的，它定义了使用者与锁交互的接口（比如可以允许两个线程并行访问），隐藏了实现细节；

同步器面向的是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

锁和同步器很好地隔离了使用者和实现者所需关注的领域。

## AQS的API

### 同步器可重写的方法

![AQS 可重写的方法](https://cos.duktig.cn/typora/202110160950056.png)

### 同步器提供的模板方法

同步器提供的模板方法基本上分为3类：独占式获取与释放同步状态、共享式获取与释放同步状态和查询同步队列中的等待线程情况。

![AQS提供的模板方法](https://cos.duktig.cn/typora/202110160951509.png)

## AQS实战——自定义同步组件

独占锁（互斥锁）Mutex是一个自定义同步组件，它**在同一时刻只允许一个线程占有锁**。

### 实现思路

继承同步器的思路：

- Mutex中定义了一个静态内部类，该内部类继承了同步器并实现了独占式获取和释放同步状态。
- `tryAcquire(int acquires)`：如果经过CAS设置成功（同步状态设置为1），则代表获取了同步状态。存在并发抢占，所以需要使用CAS来获取锁（`compareAndSetState`）。
- `tryRelease(int releases)`：只是将同步状态重置为0。因为同一时刻只有一个线程可以获取到锁，不能存在并发问题，所以直接使用`setState`即可，不用使用CAS。

锁Mutex的思路：

- 用户使用Mutex时并不会直接和内部同步器的实现打交道，而是调用Mutex提供的方法。
- 在Mutex的实现中，调用同步器的模板方法即可，大大降低了实现一个**可靠自定义同步组件**的门槛。

### 代码实现

```java
/**
 * description:自定义同步组件（AQS原理）——互斥锁：同一个时刻只允许有一个锁占有线程
 *
 * @author RenShiWei
 * Date: 2021/7/5 21:25
 **/
public class Mutex implements Lock {

    /**
     * 静态内部类自定义AQS同步器
     */
    private static class Sync extends AbstractQueuedSynchronizer {

        /**
         * 当前同步器是否处于占用状态（因为只有一个线程可以占用，所以state为1时代表占用）
         */
        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        /**
         * 独占式获取锁
         * 当状态为0时，说明没有线程占用；因为存在多线程并发抢占锁，所以使用CAS操作进行获取锁
         */
        @Override
        public boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, 1)) {
                // 设置那个线程获取到这个锁
                super.setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        /**
         * 释放锁，将同步状态设置为0
         * 只有一个线程可以获取到锁，释放锁时不存在并发抢占，因此不需要CAS操作
         */
        @Override
        protected boolean tryRelease(int releases) {
            if (getState() == 0) {
                throw new IllegalMonitorStateException();
            }
            // 将锁的持有者置空
            setExclusiveOwnerThread(null);
            // 设置同步状态
            setState(0);
            return true;
        }

        /**
         * @return 一个Condition，每个condition都包含了一个condition队列
         */
        Condition newCondition() { return new ConditionObject(); }

    }

    /**
     * 仅需要将操作代理到Sync上即可
     */
    private final Sync sync = new Sync();

    /**
     * 获取锁
     */
    @Override
    public void lock() {
        sync.acquire(1);
    }

    /**
     * 可中断的获取锁
     *
     * @throws InterruptedException 当前线程被中断，抛出中断异常
     */
    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    /**
     * 非阻塞获取锁
     *
     * @return 获取失败，立即返回false
     */
    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    /**
     * 中断、非阻塞、超时 获取锁
     */
    @Override
    public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

    /**
     * 释放锁
     */
    @Override
    public void unlock() {
        sync.release(1);
    }

    /**
     * @return lock的 Condition对象
     */
    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }

    /**
     * 独占模式下 同步器是否被占用（一般表示，是否被当前线程所占用）
     */
    public boolean isLocked() { return sync.isHeldExclusively(); }

    /**
     * @return 等待队列上是否有线程
     */
    public boolean hasQueuedThreads() { return sync.hasQueuedThreads(); }

}
```

## AQS的原理分析

从实现角度分析同步器是如何完成线程同步的，主要包括：同步队列、独占式同步状态获取与释放、共享式同步状态获取与释放以及超时获取同步状态等同步器的核心数据结构与模板方法。

### 同步队列

同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成为一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。

同步队列中的节点（Node）用来保存获取同步状态失败的线程引用、等待状态以及前驱和后继节点，节点的属性类型与名称以及描述如下表所示：

![同步队列的一个几点](https://cos.duktig.cn/typora/202110161407505.png)

同步器拥有首节点（head）和尾节点（tail），没有成功获取同步状态的线程将会成为节点加入该队列的尾部，同步队列的基本结构如图：

![同步队列的基本结构](https://cos.duktig.cn/typora/202110161414525.png)



**插入尾节点时**，可能存在并发问题，所以需要使用CAS设置尾节点的方法：`compareAndSetTail(Node expect,Node update)`，`expect`代表尾节点，`update`代表当前节点。

**获取同步状态**：

同步队列遵循FIFO，首节点是获取同步状态成功的节点，首节点的线程在释放同步状态时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点，该过程如下图：

![获取同步状态成功的节点示意图](https://cos.duktig.cn/typora/202110161414652.png)

设置首节点是通过获取同步状态成功的线程来完成的，**由于只有一个线程能够成功获取到同步状态**，因此设置头节点的方法并不需要使用CAS来保证，它只需要**将首节点设置成为原首节点的后继节点并断开原首节点的next引用即可**。

### 独占式同步状态获取与释放

独占式同步状态获取流程，也就是acquire(int arg)方法调用流程，如图：

![独占式获取同步状态](https://cos.duktig.cn/typora/202110161433684.png)

获取独占式同步状态：

1. 同步器维护一个同步队列，获取同步状态成功直接返回。
2. **获取同步状态失败的线程会被构造成节点，循环CAS（死循环式自旋保证节点正确添加）加入到队列尾部**。
3. 节点自旋加入尾部成功后，**如果是头结点尝试获取同步状态**，随后退出自旋，其他线程唤醒依靠前驱节点出队或阻塞的线程被中断。
4. 移出队列（或停止自旋）的条件是前驱节点为头节点且成功获取了同步状态。

释放独占式同步状态：

1. 同步器调用`tryRelease(int arg)`方法释放同步状态。
2. 唤醒头节点的后继节点。



### 共享式同步状态获取与释放

共享式获取与独占式获取最主要的区别：**同一时刻能否有多个线程同时获取到同步状态**。

以文件读写为例，共享式同步状态支持 **并发读和独占写**，具体不同如下图：

 ![共享式与独占式访问资源的对比](https://cos.duktig.cn/typora/202110161547964.png)

在共享式获取的自旋过程中，如果当前节点的前驱为头节点时，尝试获取同步状态，如果返回值大于等于0，表示该次获取同步状态成功并从自旋过程中退出。

与独占式一样，共享式获取在释放同步状态后，将会唤醒后续处于等待状态的节点。与独占式不同的是，释放同步状态要保证线程安全（释放同步状态的操作会同时来自多个线程），需要使用CAS操作。

### 独占式超时获取同步状态

#### 响应中断的同步状态获取

在Java 5之前，当一个线程获取不到锁而被阻塞在`synchronized`之外时，对该线程进行中断操作，此时该线程的中断标志位会被修改，但线程依旧会阻塞在`synchronized`上，等待着获取锁。

在Java 5中，同步器提供了`acquireInterruptibly(int arg)`方法，这个方法在等待获取同步状态时，如果当前线程被中断，会立刻返回，并抛出`InterruptedException`。

#### 超时的同步状态获取

超时获取同步状态过程可以被视作响应中断获取同步状态过程的“增强版”。

`doAcquireNanos(int arg,long nanosTimeout)`方法在支持响应中断的基础上，增加了超时获取的特性。

> 针对超时获取，主要需要计算出需要睡眠的时间间隔nanosTimeout，为了防止过早通知，nanosTimeout计算公式为：nanosTimeout-=now-lastTime，其中now为当前唤醒时间，lastTime为上次唤醒时间，如果nanosTimeout大于0则表示超时时间未到，需要继续睡眠nanosTimeout纳秒，反之，表示已经超时。
>
> 如果nanosTimeout小于等于spinForTimeoutThreshold（1000纳秒）时，将不会使该线程进行超时等待，而是进入快速的自旋过程。原因于，非常短的超时等待无法做到十分精确，如果这时再进行超时等待，相反会让nanosTimeout的超时从整体上表现得反而不精确。因此，在超时非常短的场景下，同步器会进入无条件的快速自旋。

具体过程如下：

![超时的同步状态获取流程](https://cos.duktig.cn/typora/202110161619650.png)

# ReetrantLock

## 什么是ReetrantLock？

> 重入锁ReentrantLock，顾名思义，就是支持重进入的锁，它表示该锁能够支持一个线程对资源的重复加锁。除此之外，该锁的还支持获取锁时的公平和非公平性选择。

### 锁为什么要支持可重入？

**如果锁不支持可重入，再次调用`lock()`方法，则该线程将会被自己所阻塞**。即调用`tryAcquire(int acquires)`方法时获取同步状态返回了false，标识当前锁已被持有，导致该线程被阻塞。

### 锁的公平性问题

如果在绝对时间上，**先对锁进行获取的请求一定先被满足，那么这个锁是公平的**，反之，是不公平的。

公平的获取锁，也就是等待时间最长的线程最优先获取锁，也可以说锁获取是顺序的。ReentrantLock提供了一个构造函数，能够控制锁是否是公平的。

公平的锁机制往往没有非公平的效率高，但是**公平锁能够减少“饥饿”发生的概率**，等待越久的请求越是能够得到优先满足。

## 重入锁实现原理

重进入是指**任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞**。

该特性的实现需要解决以下两个问题：

1. **线程再次获取锁**。锁需要去识别获取锁的线程是否为当前占据锁的线程，如果是，则再次成功获取。
2. **锁的最终释放**。线程重复n次获取了锁，随后在第n次释放该锁后，其他线程能够获取到该锁。锁的最终释放要求锁对于获取进行计数自增，计数表示当前锁被重复获取的次数，而锁被释放时，计数自减，当计数等于0时表示锁已经成功释放。

### 重入锁加锁

以非公平性（默认的）实现为例分析重入锁的实现，重入锁的加锁逻辑：

- 通过判断当前线程是否为获取锁的线程来决定获取操作是否成功
- 如果是获取锁的线程再次请求，则将同步状态值进行增加并返回true，表示获取同步状态成功。
- 成功获取锁的线程再次获取锁，只是增加了同步状态值（这也就要求ReentrantLock在释放同步状态时减少同步状态值）

代码分析：

> `ReentrantLock`的`nonfairTryAcquire`方法
>
> ```java
> final boolean nonfairTryAcquire(int acquires) {
>        final Thread current = Thread.currentThread();
>        int c = getState();
>        // c为0，表示第一次获取锁
>        if (c == 0) {
>            if (compareAndSetState(0, acquires)) {
>                setExclusiveOwnerThread(current);
>                return true;
>            }
>            // 不是第一次获取先判断当前线程是否为持有锁的线程
>        } else if (current == getExclusiveOwnerThread()) {
>            int nextc = c + acquires;
>            if (nextc < 0)
>                throw new Error("Maximum lock count exceeded");
>            // 设置增加的同步状态
>            setState(nextc);
>            // 返回true表示加锁成功
>            return true;
>        }
>        return false;
> }
> ```



### 重入锁释放

重入锁的释放逻辑：

- 如果该锁被获取了n次，那么前(n-1)次`tryRelease(int releases)`方法必须返回`false`，而只有同步状态完全释放了，才能返回`true`。

- 该方法将同步状态是否为0作为最终释放的条件，当同步状态为0时，将占有线程设置为`null`，并返回`true`，表示释放成功。

代码分析：

> `ReentrantLock`的`tryRelease`方法
>
> ```java
> protected final boolean tryRelease(int releases) {
>     int c = getState() - releases;
>     if (Thread.currentThread() != getExclusiveOwnerThread())
>             throw new IllegalMonitorStateException();
>     boolean free = false;
>     if (c == 0) {
>             free = true;
>             setExclusiveOwnerThread(null);
>     }
>     setState(c);
>     return free;
> }
> ```

## 公平与非公平获取锁的区别

> **公平锁**：采用**先到先得**的策略，每次获取锁时都会检查队列里有没有排队等待的线程，没有才尝试获取锁，如果有就将当前线程追加到队列中。
>
> **非公平锁**：采用有机会插队的策略，一个线程获取锁之前要先尝试获取锁而不是在队列中等待，如果获取锁成功，则说明线程虽然是后启动的，但先获得了锁，这就是“作弊插队”的效果。如果没有获取锁成功，那么将自身追加到队列中进行等待。

#### 公平锁的实现分析

公平性与否是针对获取锁而言的，**如果一个锁是公平的，那么锁的获取顺序就应该符合请求的绝对时间顺序，也就是FIFO**。

> 介绍的`nonfairTryAcquire(int acquires)`方法，对于非公平锁，只要CAS设置同步状态成功，则表示当前线程获取了锁，而公平锁则不同，如代码：
>
> ```java
> protected final boolean tryAcquire(int acquires) {
>        final Thread current = Thread.currentThread();
>        int c = getState();
>        if (c == 0) {
>            if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
>                setExclusiveOwnerThread(current);
>                return true;
>            }
>        } else if (current == getExclusiveOwnerThread()) {
>            int nextc = c + acquires;
>            if (nextc < 0)
>                throw new Error("Maximum lock count exceeded");
>            setState(nextc);
>            return true;
>        }
>        return false;
> }
> ```

该方法与`nonfairTryAcquire(int acquires)`比较，唯一不同的位置为判断条件多了`hasQueuedPredecessors()`方法，即**加入了同步队列中当前节点是否有前驱节点的判断**，如果该方法返回true，则表示有线程比当前线程更早地请求获取锁，因此需要等待前驱线程获取并释放锁之后才能继续获取锁。

#### 为什么非公平锁会出现线程连续获取锁的情况呢？

`nonfairTryAcquire(int acquires)`方法，当一个线程请求锁时，只要获取了同步状态即成功获取锁。在这个前提下，**刚释放锁的线程再次获取同步状态的几率会非常大**，使得其他线程只能在同步队列中等待。



#### 非公平性锁可能使线程“饥饿”，为什么它又被设定成默认的实现呢？

如果把每次不同线程获取到锁定义为1次切换，公平性锁在测试中进行了10次切换，而非公平性锁只有5次切换，这说明**非公平性锁的开销更小**。

下面运行测试用例（测试环境：ubuntuserver 14.04 i5-34708GB，测试场景：10个线程，每个线程获取100000次锁），通过vmstat统计测试运行时系统线程上下文切换的次数，运行结果如表：

![公平锁和非公平锁 上下文切换次数比较](https://cos.duktig.cn/typora/202110161709694.png)

在测试中公平性锁与非公平性锁相比，总耗时是其94.3倍，总切换次数是其133倍。

可以看出，**公平性锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换**。**非公平性锁虽然可能造成线程“饥饿”，但极少的线程切换，保证了其更大的吞吐量**。

# ReentrantReadWriteLock

## 什么是读写锁？

ReentrantLock是排它锁，在同一时刻只允许一个线程进行访问。而**读写锁在同一时刻可以允许多个读线程访问，但是在写线程访问时，所有的读**
**线程和其他写线程均被阻塞**。

读写锁维护了一对锁，一个读锁和一个写锁，通过分离读锁和写锁，使得并发性相比一般的排他锁有了很大提升。

## 为什么要使用读写锁？

除了保证写操作对读操作的可见性以及并发性的提升之外，读写锁能够简化读写交互场景的编程方式。

> 以一个场景为例：假设在程序中定义一个共享的用作缓存数据结构，它大部分时间提供读服务（例如查询和搜索），而写操作占有的时间很少，但是写操作完成之后的更新需要对后续的读服务可见。

**在没有读写锁支持的**（Java 5之前）时候实现方式：

使用Java的等待通知机制，就是当写操作开始时，所有晚于写操作的读操作均会进入等待状态，只有写操作完成并进行通知之后，所有等待的读操作才能继续执行（写操作之间依靠synchronized关键进行同步）。

这样做的目的是使读操作能读取到正确的数据，不会出现脏读。

**使用读写锁实现**：

只需要在读操作时获取读锁，写操作时获取写锁即可。

当写锁被获取到时，后续（非当前写操作线程）的读写操作都会被阻塞，写锁释放之后，所有操作继续执行，编程方式相对于使用等待通知机制的实现方式而言，变得简单明了。

**使用场景**：

一般情况下，读写锁的性能都会比排它锁好，因为大多数场景读是多于写的。在读多于写的情况下，读写锁能够提供比排它锁更好的**并发性**和**吞吐量**。

## ReentrantReadWriteLock的使用

### 读写锁的特性

![ReentrantReadWriteLock的特性](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211016172530817.png)

### 读写锁的接口

ReadWriteLock仅定义了获取读锁和写锁的两个方法，即`readLock()`方法和`writeLock()`方法，而其实现——ReentrantReadWriteLock，除了接口方法之外，还提供了一些便于外界监控其内部工作状态的方法，这些方法以及描述如表：

![ReentrantReadWriteLock 部分API](https://cos.duktig.cn/typora/202110161729546.png)



### 读写锁示例

利用读写锁实现HashMap线程安全的缓存。

代码如下：

```java
/**
 * description: 读写锁缓存的实现（ReentrantReadWriteLock示例）
 *
 * @author RenShiWei
 * Date: 2021/10/16 17:30
 * blog: https://duktig.cn/
 * github知识库: https://github.com/duktig666/knowledge
 **/
public class ReadWriteLockCache {

    static Map<String, Object> map = new HashMap<String, Object>();
    static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    static Lock r = rwl.readLock();
    static Lock w = rwl.writeLock();

    /**
     * 获取一个key对应的value
     */
    public static Object get(String key) {
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }

    /**
     * 设置key对应的value，并返回旧的value
     *
     * @param key   /
     * @param value /
     * @return 旧的value
     */
    public static Object put(String key, Object value) {
        w.lock();
        try {
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }

    /**
     * 清空所有的内容
     */
    public static void clear() {
        w.lock();
        try {
            map.clear();
        } finally {
            w.unlock();
        }
    }

}
```

## ReentrantReadWriteLock原理分析

分析 `ReentrantReadWriteLock` 的实现，主要包括：读写状态的设计、写锁的获取与释放、读锁的获取与释放以及锁降级。

### 读写状态的设计

#### 读写锁同步状态的设计

读写锁同样依赖自定义同步器来实现同步功能，而**读写状态就是其同步器的同步状态**。

在`ReentrantLock`中自定义同步器的实现，同步状态表示**锁被一个线程重复获取的次数**；而读写锁的自定义同步器需要**在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态**。

如果在一个整型变量上维护多种状态，就一定需要“**按位切割使用**”这个变量，读写锁将变量切分成了两个部分，**高16位表示读，低16位表示写**，划分方式如图：

![读写锁状态的划分方式](https://cos.duktig.cn/typora/202110161737766.png)

当前同步状态表示一个线程已经获取了写锁，且重进入了两次，同时也连续获取了两次读锁。

#### 读写锁是如何迅速确定读和写各自的状态？

通过位运算。

假设当前同步状态值为S，写状态等于S&0x0000FFFF（将高16位全部抹去），读状态等于S>>>16（无符号补0右移16位）。当写状态增加1时，等于S+1，当读状态增加1时，等于S+(1<<16)，也就是S+0x00010000。

根据状态的划分能得出一个推论：S不等于0时，当写状态（S&0x0000FFFF）等于0时，则读状态（S>>>16）大于0，即读锁已被获取。



### 写锁的获取与释放

写锁是一个**支持重进入的排它锁**。

如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态。

> ReentrantReadWriteLock的tryAcquire方法：
>
> ```java
> protected final boolean tryAcquire(int acquires) {
>        Thread current = Thread.currentThread();
>        int c = getState();
>        int w = exclusiveCount(c);
>        if (c != 0) {
>            // 存在读锁或者当前获取线程不是已经获取写锁的线程
>            if (w == 0 || current != getExclusiveOwnerThread())
>                return false;
>            if (w + exclusiveCount(acquires) > MAX_COUNT)
>                throw new Error("Maximum lock count exceeded");
>            setState(c + acquires);
>            return true;
>        }
>        if (writerShouldBlock() || !compareAndSetState(c, c + acquires)) {
>            return false;
>        }
>        setExclusiveOwnerThread(current);
>        return true;
> }
> ```

该方法除了重入条件（当前线程为获取了写锁的线程）之外，增加了一个读锁是否存在的判断。

如果存在读锁，则写锁不能被获取，原因在于：读写锁要**确保写锁的操作对读锁可见**，如果允许读锁在已被获取的情况下对写锁的获取，那么正在运行的其他读线程就无法感知到当前写线程的操作。

写锁的释放与ReentrantLock的释放过程基本类似。

### 读锁的获取与释放

读锁是一个支持重进入的共享锁，它能够被多个线程同时获取，**在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取**，而所做的也只是（线程安全的）增加读状态。

**读状态是所有线程获取读锁次数的总和**，而**每个线程各自获取读锁的次数只能选择保存在ThreadLocal中**，由线程自身维护。

> 获取读锁的实现从Java 5到Java 6变得复杂许多，主要原因是新增了一些功能，例如`getReadHoldCount()`方法，作用是返回当前线程获取读锁的次数。如下代码做了删减，保留必要部分。
>
> ```java
> protected final int tryAcquireShared(int unused) {
>     for (;;) {
>         int c = getState();
>         int nextc = c + (1 << 16);
>         if (nextc < c)
>             throw new Error("Maximum lock count exceeded");
>         if (exclusiveCount(c) != 0 && owner != Thread.currentThread())
>             return -1;
>         if (compareAndSetState(c, nextc)) 
>             return 1;
>     }
> }
> ```

在 `tryAcquireShared(int unused)` 方法中，如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态。如果当前线程获取了写锁或者写锁未被获取，则当前线程（线程安全，依靠CAS保证）增加读状态，成功获取读锁。

读锁的每次释放（线程安全的，可能有多个读线程同时释放读锁）均减少读状态，减少的值是（1<<16）。

### 锁降级

锁降级指的是**写锁降级成为读锁**。

如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。**锁降级是指把持住（当前拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁的过程**。

> 锁降级示例：
>
> ```java
> public void processData() {
>     readLock.lock();
>     if (!update) {
>         // 必须先释放读锁
>         readLock.unlock();
>         // 锁降级从写锁获取到开始
>         writeLock.lock();
>         try {
>             if (!update) {
>                 // 准备数据的流程（略）
>                 update = true;
>             }
>             readLock.lock();
>         } finally {
>             writeLock.unlock();
>         }
>         // 锁降级完成，写锁降级为读锁
>     }
>     try {
> 
> 
>         // 使用数据的流程（略）
>     } finally {
>         readLock.unlock();
>     }
> }
> ```
>
> 当数据发生变更后，update变量（布尔类型且volatile修饰）被设置为false，此时所有访问processData()方法的线程都能够感知到变化，但只有一个线程能够获取到写锁，其他线程会被阻塞在读锁和写锁的lock()方法上。当前线程获取写锁完成数据准备之后，再获取读锁，随后释放写锁，完成锁降级。

**锁降级中读锁的获取是否必要呢？**

必要的。主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程（记作线程T）获取了写锁并修改了数据，那么当前线程无法感知线程T的数据更新。

如果当前线程获取读锁，即遵循锁降级的步骤，则线程T将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁进行数据更新。

**RentrantReadWriteLock是否支持锁升级（把持读锁、获取写锁，最后释放读锁的过程）？**

不支持。目的也是保证数据可见性，如**果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的**。



# Condition接口

## 什么是Condition？

任意一个Java对象，都拥有一组监视器方法（定义在java.lang.Object上），主要包括wait()、wait(long timeout)、notify()以及notifyAll()方法，这些方法与synchronized同步关键字配合，可以实现**等待/通知模式**。Condition接口也提供了类似Object的监视器方法，与Lock配合可以实现等通知模式，但是这两者在使用方式以及功能特性上还是有差别的。具体如下：

![Object的监视器方法与Condition接口的对比](https://cos.duktig.cn/typora/202110162008580.png)

## 如何使用Condition？

Condition定义了等待/通知两种类型的方法，当前线程调用这些方法时，需要提前获取到Condition对象关联的锁。Condition对象是由Lock对象（调用Lock对象的`newCondition()`方法）创建出来的，换句话说，Condition是依赖Lock对象的。

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

public void conditionWait() throws InterruptedException {
    lock.lock();
    try {
        condition.await();
    } finally {
        lock.unlock();
    }
}

public void conditionSignal() throws InterruptedException {
    lock.lock();
    try {
        condition.signal();
    } finally {
        lock.unlock();
    }
}
```

Condition定义的（部分）方法以及描述如下：

![Condition定义的（部分）方法](https://cos.duktig.cn/typora/202110162013958.png)

获取一个Condition必须通过Lock的`newCondition()`方法。

## Condition实战——实现阻塞队列

```java
/**
 * description:数组来实现简单的阻塞队列
 * <p>
 * 使用 ReentrantLock + Condition
 *
 * @author RenShiWei
 * Date: 2021/8/6 17:06
 **/
public class ArrayBlockQueue<T> {

    /** 存放元素的数组 */
    private T[] data;

    /** 实际的元素个数 */
    private int count;

    /** 队首指针 */
    private int head;

    /** 队尾指针 */
    private int tail;

    private Lock lock = new ReentrantLock();
    private Condition notFullCondition = lock.newCondition();
    private Condition notEmptyCondition = lock.newCondition();

    @SuppressWarnings("unchecked")
    public ArrayBlockQueue(int len) {
        data = (T[]) new Object[len];
    }

    /**
     * 阻塞队列入队
     *
     * @param t 入队元素
     */
    public void put(T t) {
        lock.lock();
        try {
            while (count >= data.length) {
                try {
                    System.out.println(Thread.currentThread().getName() + "队列已满，put操作阻塞线程");
                    notFullCondition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            this.data[tail] = t;
            this.tail = (tail + 1) % this.data.length;
            count++;
            notEmptyCondition.signal();
        } finally {
            lock.unlock();
        }
    }

    /**
     * 阻塞队列出队
     *
     * @return 出队元素
     */
    public T take() {
        lock.lock();//获取锁不能写在try块中，如果发生异常，锁会被释放
        try {
            while (count == 0) {
                try {
                    System.out.println(Thread.currentThread().getName() + "队列已空，take操作阻塞线程");
                    notEmptyCondition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = data[head];
            this.head = (head + 1) % data.length;
            count--;
            notFullCondition.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 测试
     */
    @SuppressWarnings("all")
    public static void main(String[] args) {
        ArrayBlockQueue<Integer> blockingQueue = new ArrayBlockQueue<>(10);
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 9; i++) {
                blockingQueue.put(i);
                System.out.println("put-num:" + i);
            }
        });
        try {
            thread.start();
            //等待thread执行完毕后再进行操作
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                Integer num = blockingQueue.take();
                System.out.println(Thread.currentThread().getName() + "-take-num:" + num);
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 15; i++) {
                Integer num = blockingQueue.take();
                System.out.println(Thread.currentThread().getName() + "-take-num:" + num);
            }
        }).start();

    }

}
```

## Condition的原理分析

ConditionObject是同步器`AbstractQueuedSynchronizer`的内部类，因为Condition的操作需要获取相关联的锁，所以作为同步器的内部类也较为合理。

**每个Condition对象都包含着一个队列（以下称为等待队列）**，该队列是Condition对象实现等待/通知功能的关键。

Condition的实现，主要包括：等待队列、等待和通知。

### 等待队列

等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在Condition对象上等待的线程。**如果一个线程调用了  `Condition.await()` 方法，那么该线程将会释放锁、构造成节点加入等待队列并进入等待状态**。

> 事实上，节点的定义复用了同步器中节点的定义，也就是说，**同步队列和等待队列中节点类型都是同步器的静态内部类** `AbstractQueuedSynchronizer.Node`。

一个Condition包含一个等待队列，Condition拥有首节点（firstWaiter）和尾节点（lastWaiter）。当前线程调用`Condition.await()`方法，将会以当前线程构造节点，并将节点从尾部加入等待队列，等待队列的基本结构如图:

![等待队列的基本结构](https://cos.duktig.cn/typora/202110162018219.png)

上述**节点引用更新的过程并没有使用CAS保证**，原因在于调用await()方法的线程必定是获取了锁的线程，也就是说**该过程是由锁来保证线程安全的**。

在Object的监视器模型上（synchronized），一个对象拥有一个同步队列和等待队列，而并发包中的Lock（更确切地说是同步器）拥有**一个同步队列和多个等待队列**，其对应关系如图：

![同步队列与等待队列](https://cos.duktig.cn/typora/202110162020261.png)

### 等待机制

调用Condition的`await()`方法（或者以await开头的方法），会**使当前线程进入等待队列并释放锁，同时线程状态变为等待状态**。

当从`await()`方法返回时，当前线程一定获取了Condition相关联的锁。

如果从队列（同步队列和等待队列）的角度看`await()`方法，**当调用`await()`方法时，相当于同步队列的首节点（获取了锁的节点）移动到Condition的等待队列中**。

> ConditionObject的await方法
>
> ```java
> public final void await() throws InterruptedException {
>     if (Thread.interrupted())
>         throw new InterruptedException();
>     // 当前线程加入等待队列
>     Node node = addConditionWaiter();
>     // 释放同步状态，也就是释放锁
>     int savedState = fullyRelease(node);
>     int interruptMode = 0;
>     while (!isOnSyncQueue(node)) {
>         LockSupport.park(this);
>         if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
>             break;
>     }
>     if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
>         interruptMode = REINTERRUPT;
>     if (node.nextWaiter != null)
>         unlinkCancelledWaiters();
>     if (interruptMode != 0)
>         reportInterruptAfterWait(interruptMode);
> }
> ```

调用该方法的线程成功获取了锁的线程，也就是同步队列中的首节点，该方法会将当前线程构造成节点并加入等待队列中，然后释放同步状态，唤醒同步队列中的后继节点，然后当前线程会进入等待状态。

如果从队列的角度去看，当前线程加入Condition的等待队列，该过程如图：

![当前线程加入等待队列](https://cos.duktig.cn/typora/202110162023047.png)



如图所示，同步队列的首节点并不会直接加入等待队列，而是通过`addConditionWaiter()`方法把当前线程构造成一个新的节点并将其加入等待队列中。

### 通知机制

调用Condition的signal()方法，将会唤醒在等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移到同步队列中。

> ConditionObject的signal方法
>
> ```java
> public final void signal() {
>     if (!isHeldExclusively())
>         throw new IllegalMonitorStateException();
>     Node first = firstWaiter;
>     if (first != null)
>         doSignal(first);
> }
> ```

调用该方法的前置条件是当前线程必须获取了锁，可以看到`signal()`方法进行了`isHeldExclusively()`检查，也就是当前线程必须是获取了锁的线程。接着获取等待队列的首节点，将其移动到同步队列并使用LockSupport唤醒节点中的线程。

节点从等待队列移动到同步队列的过程如图:

![节点从等待队列移动到同步队列](https://cos.duktig.cn/typora/202110162025326.png)

通过调用同步器的`enq(Node node)`方法，等待队列中的头节点线程安全地移动到同步队列。当节点移动到同步队列后，当前线程再使用LockSupport唤醒该节点的线程。

被唤醒后的线程，将从`await()`方法中的while循环中退出(`isOnSyncQueue(Node node)`方法返回`true`，节点已经在同步队列中），进而调用同步器的`acquireQueued()`方法加入到获取同步状态的竞争中。

成功获取同步状态（或者说锁）之后，被唤醒的线程将从先前调用的`await()`方法返回，此时该线程已经成功地获取了锁。

Condition的`signalAll()`方法，相当于对等待队列中的每个节点均执行一次`signal()`方法，效果就是将等待队列中所有节点全部移动到同步队列中，并唤醒每个节点的线程。

总结：

1. 等待队列中的头节点线程安全地移动到同步队列，然后被唤醒。
2. 被唤醒后的线程，从`await()`方法中的while循环中退出，并进行同步状态的竞争
3. 成功获取到同步状态后，被唤醒的线程将从先前调用的`await()`方法返回。

