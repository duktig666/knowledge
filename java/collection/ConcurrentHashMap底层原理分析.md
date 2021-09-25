> 作者：duktig
>
> 博客：[https://duktig.cn](https://duktig.cn)  （文章首发）
>
> 优秀还努力。愿你付出甘之如饴，所得归于欢喜。

# 背景

上篇文章分析了`HashMap`的原理，但是其不能保证线程安全，所以`ConcurrentHashMap`作为线程安全的`Map` ，它的使用频率也是很高。也是面试中的常考的点。

> 面试官：面试开始，那我们就先聊聊Java基础吧
>
> 面试官：你先说说`HashMap`的底层数据结构和原理吧
>
> 我：`HashMap`使用了 数组+链表+红黑树。然后怎么解决hash冲突，怎么扩容和缩容……
>
> 面试官：`HashMap`怎么减少hash碰撞？
> 我：通过负载因子……
>
> 面试官：`HashMap`是线程不安全的，那么它会带来是么问题呢？
>
> 我：……
>
> 面试官：线程安全的`Map`我们通常使用`ConcurrentHashMap`，那么说说他的底层数据结构和原理吧？
>
> 我：瞬间蒙圈！！！
>
> *然后说到线程安全，马上又聊到多线程和高并发……*

有的面试官一般会一步步地挖掘你的知识深度，看看你到底能达到什么地步。

# 多线程下的 HashMap 有什么问题？

- 在jdk1.7中，在多线程环境下，扩容时会造成死循环（尾插法引起环形链）或数据丢失。
- 在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。

## jdk1.7多线程环境下HashMap问题

在jdk1.7多线程环境下HashMap容易出现死循环和数据丢失，这里我们先用代码来模拟出现死循环的情况：

```java
public class HashMapTest {

    public static void main(String[] args) {
        HashMapThread thread0 = new HashMapThread();
        HashMapThread thread1 = new HashMapThread();
        HashMapThread thread2 = new HashMapThread();
        HashMapThread thread3 = new HashMapThread();
        HashMapThread thread4 = new HashMapThread();
        thread0.start();
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
    }
}

class HashMapThread extends Thread {
    private static AtomicInteger ai = new AtomicInteger();
    private static Map<Integer, Integer> map = new HashMap<>();

    @Override
    public void run() {
        while (ai.get() < 1000000) {
            map.put(ai.get(), ai.get());
            ai.incrementAndGet();
        }
    }
}
```

上述代码比较简单，就是开多个线程不断进行put操作，并且HashMap与AtomicInteger都是全局共享的。在多运行几次该代码后，出现死循环情形。

我们着重分析为什么会出现死循环的情况，通过`jps`和`jstack`命名查看死循环情况，结果如下：

![jdk1.8 HashMap死循环堆栈情况](https://cos.duktig.cn/typora/202109252012771.jpeg)

从堆栈信息中可以看到出现死循环的位置，通过该信息可明确知道死循环发生在HashMap的扩容函数中，根源在**`transfer`函数**中，jdk1.7中HashMap的`transfer`函数如下：

```java
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

在对table进行扩容到newTable后，需要将原来数据转移到newTable中，注意10-12行代码，这里可以看出在转移元素的过程中，使用的是**头插法**，也就是**链表的顺序会翻转**，这里也是形成死循环的关键点。

## **jdk1.8多线程环境下HashMap问题**

在jdk8中对HashMap进行了优化，在发生hash碰撞，不再采用头插法方式，而是直接插入链表尾部，因此不会出现环形链表的情况，但是在多线程的情况下仍然不安全。

这里我们看jdk8中HashMap的put操作源码：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) // 如果没有hash碰撞则直接插入元素
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

这是jdk1.8中HashMap中put操作的主函数， 注意第6行代码，如果没有hash碰撞则会直接插入元素。如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给**覆盖**，发生线程不安全。

详细分析参看：[阿里Java一面：说说HashMap线程不安全的体现，这TM什么人间疾苦！](https://zhuanlan.zhihu.com/p/174226955)

# 怎样保证线程安全，为什么选用 ConcurrentHashMap？

**怎样保证线程安全？**

首先能想到的就是`Hashtable`了，但是它的 `put` 方法和 `get` 方法是通过加`synchronized`关键字来实现的，锁住了整个 table，效率低下，因此 并不适合高并发场景。

也许，你还会想起来一个集合工具类 `Collections`，生成一个`SynchronizedMap`。其实，它和 Hashtable 差不多，同样的原因，锁住整张表，效率低下。

所以，思考一下，既然锁住整张表的话，并发效率低下，那我**把整张表分成 N 个部分，并使元素尽量均匀的分布到每个部分中，分别给他们加锁，互相之间并不影响**，这种方式岂不是更好 。这就是在 JDK1.7 中 `ConcurrentHashMap` 采用的方案，被叫做**锁分段技术**，每个部分就是一个 Segment（段）。

但是，在JDK1.8中，完全重构了，采用的是 **`Synchronized` + `CAS`** ，**把锁的粒度进一步降低，而放弃了 Segment 分段**。（此时的 Synchronized 已经升级了，效率得到了很大提升，锁升级可以了解一下）

# ConcurrentHashMap的数据结构

## ConcurrentHashMap 1.7

在 JDK1.7中，本质上还是采用链表+数组的形式存储键值对的。但是，为了提高并发，把原来的整个 table 划分为 n 个 `Segment` 。所以，从整体来看，它是一个由 `Segment` 组成的数组。然后，每个 `Segment` 里边是由 `HashEntry` 组成的数组，每个 `HashEntry`之间又可以形成链表。我们可以把每个 `Segment` 看成是一个小的 `HashMap`，其内部结构和 `HashMap` 是一模一样的，每一个 `HashMap` 的内部可以进行扩容。但是 `Segment` 的个数一旦**初始化就不能改变**，默认 `Segment` 的个数是 16 个，你也可以认为 `ConcurrentHashMap` 默认支持最多 16 个线程并发。

![ConcurrentHashMap 1.7存储结构](https://cos.duktig.cn/typora/202109252101050.png)

当对某个 Segment 加锁时，并不会影响到其他 Segment 的读写。每个 Segment 内部自己操作自己的数据。这样一来，我们要做的就是**尽可能的让元素均匀的分布在不同的 Segment中**。最理想的状态是，所有执行的线程操作的元素都是不同的 Segment，这样就可以降低锁的竞争。

## ConcurrentHashMap 1.8

可以发现 Java8 的 `ConcurrentHashMap` 相对于 Java7 来说变化比较大，不再是之前的 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。当冲突链表达到一定长度时，链表会转换成红黑树。

![Java8 ConcurrentHashMap 存储结构](https://cos.duktig.cn/typora/202109252157388.png)

不再使用分段锁，而是给数组中的每一个头节点（为了方便，以后都叫桶）都加锁，锁的粒度降低了。并且，用的是 `Synchronized `锁。

底层存储结构和 1.8 的 HashMap 是一样的，不同的就是，多了一些并发的处理。



# ConcurrentHashMap 1.8 如何保证线程安全？

## 初始化 initTable

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    //循环判断表是否为空，直到初始化成功为止。
    while ((tab = table) == null || tab.length == 0) {
        //sizeCtl 这个值有很多情况，默认值为0，
        //当为 -1 时，说明有其它线程正在对表进行初始化操作
        //当表初始化成功后，又会把它设置为扩容阈值
        //当为一个小于 -1 的负数，用来表示当前有几个线程正在帮助扩容(后边细讲)
        if ((sc = sizeCtl) < 0)
            //若 sc 小于0，其实在这里就是-1，因为此时表是空的，不会发生扩容，sc只能为正数或者-1
            //因此，当前线程放弃 CPU 时间片，只是自旋。
            Thread.yield(); // lost initialization race; just spin
        //通过 CAS 把 sc 的值设置为-1，表明当前线程正在进行表的初始化，其它失败的线程就会自旋
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                //重新检查一下表是否为空
                if ((tab = table) == null || tab.length == 0) {
                    //如果sc大于0，则为sc，否则返回默认容量 16。
                    //当调用有参构造创建 Map 时，sc的值是大于0的。
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    //创建数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    //n减去 1/4 n ，即为 0.75n ，表示扩容阈值
                    sc = n - (n >>> 2);
                }
            } finally {
                //更新 sizeCtl 为扩容阈值
                sizeCtl = sc;
            }
            //若当前线程初始化表成功，则跳出循环。其它自旋的线程因为判断数组不为空，也会停止自旋
            break;
        }
    }
    return tab;
}
```

从源码中可以发现 `ConcurrentHashMap` 的初始化是通过**自旋和 CAS** 操作完成的。里面需要注意的是变量 `sizeCtl` ，它的值决定着当前的初始化状态。

1. `-1` 说明正在初始化
2. `-N` 说明有N-1个线程正在进行扩容
3. 如果 table 没有初始化，表示 table 初始化大小
4. 如果 table已经初始化，表示 table 容量。

## put()方法

总结`put`过程：

1. 根据 key 计算出 hashcode 。
2. 判断是否需要进行初始化。
3. 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
4. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
5. 如果都不满足，则利用`synchronized` 锁写入数据。
6. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

源码：

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    //可以看到，在并发情况下，key 和 value 都是不支持为空的。
    if (key == null || value == null) throw new NullPointerException();
    //这里和1.8 HashMap 的hash 方法大同小异，只是多了一个操作，如下
    //( h ^ (h >>> 16)) & HASH_BITS;  HASH_BITS = 0x7fffffff;
    // 0x7fffffff ，二进制为 0111 1111 1111 1111 1111 1111 1111 1111 。
    //所以，hash值除了做了高低位异或运算，还多了一步，保证最高位的 1 个 bit 位总是0。
    //这里，我并没有明白它的意图，仅仅是保证计算出来的hash值不超过 Integer 最大值，且不为负数吗。
    //同 HashMap 的hash 方法对比一下，会发现连源码注释都是相同的，并没有多说明其它的。
    //我个人认为意义不大，因为最后 hash 是为了和 capacity -1 做与运算，而 capacity 最大值为 1<<30，
    //即 0100 0000 0000 0000 0000 0000 0000 0000 ，减1为 0011 1111 1111 1111 1111 1111 1111 1111。
    //即使 hash 最高位为 1(无所谓0)，也不影响最后的结果，最高位也总会是0.
    int hash = spread(key.hashCode());
    //用来计算当前链表上的元素个数
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //如果表为空，则说明还未初始化。
        if (tab == null || (n = tab.length) == 0)
            //初始化表，只有一个线程可以初始化成功。
            tab = initTable();
        //若表已经初始化，则找到当前 key 所在的桶，并且判断是否为空
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //若当前桶为空，则通过 CAS 原子操作，把新节点插入到此位置，
            //这保证了只有一个线程可以 CAS 成功，其它线程都会失败。
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //若所在桶不为空，则判断节点的 hash 值是否为 MOVED（值是-1）
        else if ((fh = f.hash) == MOVED)
            //若为-1，说明当前数组正在进行扩容，则需要当前线程帮忙迁移数据
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //这里用加同步锁的方式，来保证线程安全，给桶中第一个节点对象加锁
            synchronized (f) {
                //recheck 一下，保证当前桶的第一个节点无变化，后边很多这样类似的操作，不再赘述
                if (tabAt(tab, i) == f) {
                    //如果hash值大于等于0，说明是正常的链表结构
                    if (fh >= 0) {
                        binCount = 1;
                        //从头结点开始遍历，每遍历一次，binCount计数加1
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果找到了和当前 key 相同的节点，则用新值替换旧值
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            //若遍历到了尾结点，则把新节点尾插进去
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //否则判断是否是树节点。这里提一下，TreeBin只是头结点对TreeNode的再封装
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //注意下，这个判断是在同步锁外部，因为 treeifyBin内部也有同步锁，并不影响
            if (binCount != 0) {
                //如果节点个数大于等于 8，则转化为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                //把旧节点值返回
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //给元素个数加 1，并有可能会触发扩容，比较复杂，稍后细讲
    addCount(1L, binCount);
    return null;
}
```

## get()方法

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // key 所在的 hash 位置
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果指定位置元素存在，头结点hash值相同
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                // key hash 值相等，key值相同，直接返回元素 value
                return e.val;
        }
        else if (eh < 0)
            // 头结点hash值小于0，说明正在扩容或者是红黑树，find查找
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            // 是链表，遍历查找
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

总结一下 get 过程：

1. 根据 hash 值计算位置。
2. 查找到指定位置，如果头节点就是要找的，直接返回它的 value.
3. 如果头节点 hash 值小于 0 ，说明正在扩容或者是红黑树，查找之。
4. 如果是链表，遍历查找之。

# 总结

**HashMap线程不安全体现在**：

- 在jdk1.7中，在多线程环境下，扩容时会造成死循环（尾插法引起环形链）或数据丢失。
- 在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。

**ConcurrentHashMap如何解决线程安全问题？**

在JDK1.7中，使用**锁分段技术**实现线程安全。**把整张表分成 N 个部分，并使元素尽量均匀的分布到每个部分中，分别给他们加锁，互相之间并不影响**，每个部分就是一个 Segment（段）。

在JDK1.8中，完全重构了，采用的是 **`Synchronized` + `CAS`** ，**把锁的粒度进一步降低，而放弃了 Segment 分段**。（此时的 Synchronized 已经升级了，效率得到了很大提升，锁升级可以了解一下）

**`put`过程**

1. 根据 key 计算出 hashcode 。
2. 判断是否需要进行初始化。
3. 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
4. 如果当前位置的 `hashcode == MOVED == -1`，则需要进行扩容。
5. 如果都不满足，则利用`synchronized` 锁写入数据。
6. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。



*更加详细的分析，参看文末的文章，分析的都比较好。*

# 参看

- [阿里Java一面：说说HashMap线程不安全的体现，这TM什么人间疾苦！](https://zhuanlan.zhihu.com/p/174226955) （HashMap线程不安全的体现分析的比较透彻）
- [我就知道面试官接下来要问我 ConcurrentHashMap 底层原理了](https://mp.weixin.qq.com/s/My4P_BBXDnAGX1gh630ZKw) 
- [ConcurrentHashMap源码+底层数据结构分析（JavaGuide）](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ConcurrentHashMap%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90)