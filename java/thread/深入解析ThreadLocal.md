## ThreadLocal基础

ThreadLocal`用于解决线程内资源共享问题，可以提供线程局部变量，每个线程`Thread`拥有一份自己的**副本变量**，多个线程互不干扰。

`ThreadLocal`主要作用是将数据放入当前线程`Tread`类成员变量的Map中。`ThreadLocal`不管理和存储任何数据，只是作为数据和Map之间的桥梁。

执行流程：数据->`ThreadLocal`->`currentThread()`->`Map`

Map的key存`ThreadLocal`对象，value存数据值。每个`Thread`的Map只对当前线程可见，其他线程不能访问。

当前线程销毁，Map销毁，Map中的数据没有被引用和使用，随时可以被GC回收。

如果没有在`Thread`的`Map`中存储`ThreadLocal`对应的值，`get()`方法返回`null`。可以继承并覆盖`initialValue()`方法，设置初始值。



## ThreadLocal的数据结构

![ThreadLocal的数据结构](https://cos.duktig.cn/typora/202109141421470.png)

![ThreadLocal的数据结构](https://cos.duktig.cn/typora/202109141421487.png)

- `Thread`类有一个类型为`ThreadLocal.ThreadLocalMap`的实例变量`threadLocals`，也就是说每个线程有一个自己的`ThreadLocalMap`。
- `ThreadLocalMap`有自己的独立实现，可以简单地将它的`key`视作`ThreadLocal`，`value`为代码中放入的值（实际上`key`并不是`ThreadLocal`本身，而是它的一个**弱引用**）。
- 每个线程在往`ThreadLocal`里放值的时候，都会往自己的`ThreadLocalMap`里存，读也是以`ThreadLocal`作为引用，在自己的`map`里找对应的`key`，从而实现了**线程隔离**。
- `Entry`，这里没用再打开，其实它是一个弱引用实现，`static class Entry extends WeakReference>`。这说明只要没用强引用存 在，发生 **GC** 时就会被垃圾回收。
- `ThreadLocalMap`有点类似`HashMap`的结构，只是`HashMap`是由**数组+链表**实现的，而`ThreadLocalMap`中并没有**链表**结构。另外由于这里不同于 HashMap 的数据结构，发生哈希碰撞不会存成链表或红黑树，而是使用**线性探测法**进行存储。也就是**同一个下标位置发生冲突时，则+1 向后寻 址，直到找到空位置或垃圾回收位置进行存储**。
- 数据元素采用哈希散列方式进行存储，不过这里的散列使用的是 **斐波那契 （Fibonacci）散列法**。



## ThreadLocal的Hash算法

既然是`Map`结构，那么`ThreadLocalMap`当然也要实现自己的`hash`算法来解决散列表数组冲突问题——斐波那契（Fibonacci）散列法 + 线性探测法。

```java
int i = key.threadLocalHashCode & (len-1);
```

`ThreadLocalMap`中`hash`算法很简单，这里`i`就是当前 key 在散列表中对应的数组下标位置。

这里最关键的就是`threadLocalHashCode`值的计算，`ThreadLocal`中有一个属性为`HASH_INCREMENT = 0x61c88647`

```java
public class ThreadLocal<T> {
    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode = new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;

    // 计算哈希
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    //获取下标
    int i = key.threadLocalHashCode & (len-1);
}
```

每当创建一个`ThreadLocal`对象，这个`ThreadLocal.nextHashCode` 这个值就会增长 `0x61c88647` 。

### 神秘的数字 `0x61c88647`

**这是什么方式计算哈希？**

其实计算哈希的方式，绝不止是我们平常看到 String 获取哈希值的一种方式，还包括；`除法散列法`、`平方散列法`、`斐波那契（Fibonacci）散列法`、`随机数法`等。

而 `ThreadLocal` 使用的就是 `斐波那契（Fibonacci）散列法` + 线性探测法存储数据到数组结构中。之所以使用斐波那契数列，是为了让数据更加散列，减少哈希碰撞。具体来自数学公式的计算求值，**公式**：`f(k) = ((k * 2654435769) >> X) << Y`对于常见的32位整数而言，也就是 `f(k) = (k * 2654435769) >> 28`

**这个数字怎么来的？**

其实这是一个哈希值的黄金分割点，也就是 `0.618`，你还记得你学过的数学吗？计算方式如下；

```java
// 黄金分割点：(√5 - 1) / 2 = 0.6180339887     1.618:1 == 1:0.618
System.out.println(BigDecimal.valueOf(Math.pow(2, 32) * 0.6180339887).intValue());      //-1640531527
```

- 学过数学都应该知道，黄金分割点是，`(√5 - 1) / 2`，取10位近似 `0.6180339887`。
- 之后用 2 ^32^ * 0.6180339887，得到的结果是：`-1640531527`，也就是 16 进制的，0x61c88647。*这个数呢也就是这么来的*.

### 验证 斐波那契（Fibonacci）散列法

```java
public class ThreadLocalHashDemo {

    private static final int HASH_INCREMENT = 0x61c88647;

    public static void main(String[] args) {
        int hashCode = 0;
        for (int i = 0; i < 16; i++) {
            hashCode = i * HASH_INCREMENT + HASH_INCREMENT;
            int idx = hashCode & 15;
            System.out.println("斐波那契散列：" + idx + " 普通散列：" + (String.valueOf(i).hashCode() & 15));
        }
    }
    
}
```

结果：

```
斐波那契散列：7 普通散列：0
斐波那契散列：14 普通散列：1
斐波那契散列：5 普通散列：2
斐波那契散列：12 普通散列：3
斐波那契散列：3 普通散列：4
斐波那契散列：10 普通散列：5
斐波那契散列：1 普通散列：6
斐波那契散列：8 普通散列：7
斐波那契散列：15 普通散列：8
斐波那契散列：6 普通散列：9
斐波那契散列：13 普通散列：15
斐波那契散列：4 普通散列：0
斐波那契散列：11 普通散列：1
斐波那契散列：2 普通散列：2
斐波那契散列：9 普通散列：3
斐波那契散列：0 普通散列：4
```

斐波那契散列的非常均匀，普通散列到15个以后已经开发生碰撞。这也就是斐波那契散列的魅力，**减少碰撞也就可以让数据存储的更加分散，获取数据的时间复杂度基本保持在O(1)**。

## ThreadLocal的Hash冲突

> **注明：** 下面所有示例图中，**绿色块**`Entry`代表**正常数据**，**灰色块**代表`Entry`的`key`值为`null`，**已被垃圾回收**。**白色块**表示`Entry`为`null`。

虽然`ThreadLocalMap`中使用了**黄金分割数来**作为`hash`计算因子，大大减少了`Hash`冲突的概率，但是仍然会存在冲突。

`HashMap`中解决冲突的方法是在数组上构造一个**链表**结构，冲突的数据挂载到链表上，如果链表长度超过一定数量则会转化成**红黑树**。而 `ThreadLocalMap` 中并没有链表结构，所以这里不能使用 `HashMap` 解决冲突的方式了。

![ThreadLocal的Hash冲突](https://cos.duktig.cn/typora/202109141507366.png)

如上图所示，如果我们插入一个`value=27`的数据，通过 `hash` 计算后应该落入第 4 个槽位中，而槽位 4 已经有了 `Entry` 数据。

此时就会线性向后查找，一直找到 `Entry` 为 `null` 的槽位才会停止查找，将当前元素放入此槽位中。当然迭代过程中还有其他的情况，比如遇到了 `Entry` 不为 `null` 且 `key` 值相等的情况，还有 `Entry` 中的 `key` 值为 `null` 的情况等等都会有不同的处理，后面会一一详细讲解。

这里还画了一个`Entry`中的`key`为`null`的数据（**Entry=2 的灰色块数据**），因为`key`值是**弱引用**类型，所以会有这种数据存在。在`set`过程中，如果遇到了`key`过期的`Entry`数据，实际上是会进行一轮**探测式清理**操作的，具体操作方式后面会讲到。

## ThreadLocal的源码解析

### ThreadLocal 初始化解析

```
new ThreadLocal<>()
```

初始化的过程也很简单，可以按照自己需要的泛型进行设置。但在 `ThreadLocal` 的源码中有一点非常重要，就是获取 `threadLocal` 的哈希值的获取——`threadLocalHashCode`。

```java
private final int threadLocalHashCode = nextHashCode();

/**
 * Returns the next hash code.
 */
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

**只要实例化一个 `ThreadLocal` ，就会获取一个相应的哈希值**。测试例子如下：

```java
/**
 * 测试 ThreadLocal 每次初始化 产生新的 hashcode
 */
@Test
public void testThreadLocalHashCode() throws Exception {
    for (int i = 0; i < 5; i++) {
        ThreadLocal<Object> objectThreadLocal = new ThreadLocal<>();
        Field threadLocalHashCode = objectThreadLocal.getClass().getDeclaredField("threadLocalHashCode");
        threadLocalHashCode.setAccessible(true);
        System.out.println("objectThreadLocal：" + threadLocalHashCode.get(objectThreadLocal));
    }
}
```

因为 `threadLocalHashCode` ，是一个私有属性，所以我们实例化后使用**反射**的方式进行获取哈希值。

结果：

```text
objectThreadLocal：1253254570
objectThreadLocal：-1401181199
objectThreadLocal：239350328
objectThreadLocal：1879881855
objectThreadLocal：-774553914
```

这个值的获取，也就是计算 `ThreadLocalMap`，存储数据时，`ThreadLocal` 的数组下标。只要是这同一个对象，在`set`、`get`时，就可以设置和获取对应的值。

### `ThreadLocalMap.set()` 解析

```
new ThreadLocal<>().set("duktig");
```

设置元素的方法，也就这么一句代码。但设置元素的流程却涉及的比较多。往`ThreadLocalMap`中`set`数据（**新增**或者**更新**数据）分为好几种情况。

1. 情况 1，待插入的下标，是空位置直接插入。
2.  情况 2，待插入的下标，不为空，key 相同，直接更新 
3. 情况 3，待插入的下标，不为空，key 不相同，线性探测法寻址
4. 情况 4，不为空，key 不相同，碰到过期 key。其实情况 4，遇到的是弱引用发生 GC 时，产生的情况。碰到这种情况，ThreadLocal 会进行探测清理过期 key。

#### `ThreadLocalMap.set()` 四种情况解析

**第一种情况：** 通过`hash`计算后的槽位对应的`Entry`数据为空：

![情况1，待插入的下标，是空位置直接插入](https://cos.duktig.cn/typora/202109141525395.png)

这里直接将数据放到该槽位即可。

**第二种情况：** 槽位数据不为空，`key`值与当前`ThreadLocal`通过`hash`计算获取的`key`值一致：

![情况2，待插入的下标，不为空，key 相同，直接更新](https://cos.duktig.cn/typora/202109141525180.png)

这里直接更新该槽位的数据。

**第三种情况：** 槽位数据不为空，往后遍历过程中，在找到`Entry`为`null`的槽位之前，没有遇到`key`过期的`Entry`：

![情况3，待插入的下标，不为空，key 不相同，线性探测法寻址](https://cos.duktig.cn/typora/202109141525927.png)

遍历散列数组，线性往后查找，如果找到`Entry`为`null`的槽位，则将数据放入该槽位中，或者往后遍历过程中，遇到了**key 值相等**的数据，直接更新即可。

**第四种情况：** 槽位数据不为空，往后遍历过程中，在找到`Entry`为`null`的槽位之前，遇到`key`过期的`Entry`，如下图，往后遍历过程中，一到了`index=7`的槽位数据`Entry`的`key=null`：

![情况4，不为空，key 不相同，碰到过期key，进行探测清理过期key](https://cos.duktig.cn/typora/202109141525740.png)

散列数组下标为 7 位置对应的`Entry`数据`key`为`null`，表明此数据`key`值已经被垃圾回收掉了，此时就会执行`replaceStaleEntry()`方法，该方法含义是**替换过期数据的逻辑**，以**index=7**位起点开始遍历，进行探测式数据清理工作。

**关于探测式数据清理工作如下：**

**（1）以当前过期key的位置初始化**

初始化探测式清理过期数据扫描的开始位置：`slotToExpunge = staleSlot = 7`

**（2）以当前节点位置向前迭代查找，直到碰到`Entry`为`null`结束**

以当前节点(`index=7`)向前迭代，检测是否有过期的`Entry`数据，如果有则更新`slotToExpunge`值。碰到`null`则结束探测。以上图为例`slotToExpunge`被更新为 0。

![以当前节点位置向前迭代查找，直到碰到Entry为null结束](https://cos.duktig.cn/typora/202109141807871.png)

向前迭代的操作是**为了更新探测清理过期数据的起始下标`slotToExpunge`的值**，**它是用来判断当前过期槽位`staleSlot`之前是否还有过期元素**。

**（3-1）以当前节点位置向后迭代查找——如果找到了相同 key 值的 Entry 数据**

从当前节点`staleSlot`(index=7)向后查找`key`值相等的`Entry`元素，找到后更新`Entry`的值并交换`staleSlot`元素的位置。

![以当前节点位置向后迭代查找——如果找到了相同 key 值的 Entry 数据](https://cos.duktig.cn/typora/202109141808301.png)

**（3-2）向后遍历过程中，如果没有找到相同 key 值的 Entry 数据，创建新的`Entry`，替换`table[stableSlot]`位置**

从当前节点`staleSlot`向后查找`key`值相等的`Entry`元素，直到`Entry`为`null`则停止寻找。

![向后遍历过程中，如果没有找到相同 key 值的 Entry 数据](https://cos.duktig.cn/typora/202109141808513.png)

通过上图可知，此时`table`中没有`key`值相同的`Entry`。创建新的`Entry`，替换`table[stableSlot]`位置：

![创建新的Entry，替换table[stableSlot]位置](https://cos.duktig.cn/typora/202109141808557.png)

**（4）从`staleToExpunge`位置开始向后检查过期数据，并清理。**

![从staleToExpunge位置开始向后检查过期数据，并清理](https://cos.duktig.cn/typora/202109141809201.png)

清理工作主要是有两个方法：`expungeStaleEntry()`和`cleanSomeSlots()`，具体细节后面会讲到，请继续往后看。

#### `ThreadLocalMap.set()` 源码分析

详情参看如下注释

```java
private void set(ThreadLocal<?> key, Object value) {
    // Entry，是一个弱引用对象的实现类，static class Entry extends WeakReference<ThreadLocal<?>>，所以在没有外部强引用下，会发生GC，删除key。
    Entry[] tab = table;
    int len = tab.length;
    // 斐波那契散列，计算数组下标
    int i = key.threadLocalHashCode & (len-1);
    // for循环判断元素是否存在
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        if (k == key) {
            // key相等则更新值
            e.value = value;
            return;
        }
        if (k == null) {
            // key为null，替换过期数据
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 当前下标不存在元素时，直接设置元素
    tab[i] = new Entry(key, value);
    int sz = ++size;
    // 启发式清理  
    // 如果清理工作完成后，未清理到任何数据，且size超过了阈值(数组长度的 2/3)，进行rehash()操作
    // rehash()中会先进行一轮探测式清理，清理过期key，清理完成后如果size >= threshold - threshold / 4，就会执行真正的扩容逻辑
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

**（1）计算位置**

这里会通过`key`来计算在散列表中的对应位置，然后以当前`key`对应的桶的位置向后查找，找到可以使用的桶。

```java
Entry[] tab = table;
int len = tab.length;
int i = key.threadLocalHashCode & (len-1);
```

（2）**遍历前分析**

接着就是执行`for`循环遍历，向后查找，我们先看下`nextIndex()`、`prevIndex()`方法实现：

![nextIndex()、prevIndex()方法实现](https://cos.duktig.cn/typora/202109141708618.png)

```java
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

什么情况下桶才是可以使用的呢？

1. `k = key` 说明是替换操作，可以使用
2. 碰到一个过期的桶，执行替换逻辑，占用过期桶
3. 查找过程中，碰到桶中`Entry=null`的情况，直接使用

**（3）`for`循环中的逻辑**

1. 遍历当前`key`值对应的桶中`Entry`数据为空，这说明散列数组这里没有数据冲突，跳出`for`循环，直接`set`数据到对应的桶中
2. 如果`key`值对应的桶中`Entry`数据不为空 
   1. 如果`k = key`，说明当前`set`操作是一个替换操作，做替换逻辑，直接返回 
   2. 如果`key = null`，说明当前桶位置的`Entry`是过期数据，执行`replaceStaleEntry()`方法(核心方法)，然后返回
3. `for`循环执行完毕，继续往下执行说明向后迭代的过程中遇到了`entry`为`null`的情况 
   1. 在`Entry`为`null`的桶中创建一个新的`Entry`对象 
   2. 执行`++size`操作
4. 调用`cleanSomeSlots()`做一次启发式清理工作，清理散列数组中`Entry`的`key`过期的数据 
   1. 如果清理工作完成后，未清理到任何数据，且`size`超过了阈值(数组长度的 2/3)，进行`rehash()`操作 
   2.  `rehash()`中会先进行一轮探测式清理，清理过期`key`，清理完成后如果**size >= threshold - threshold / 4**，就会执行真正的扩容逻辑(扩容逻辑往后看)

（4）**`replaceStaleEntry()`方法 分析**

`replaceStaleEntry()`方法提供替换过期数据的功能。详细分析参看如下注释：

`java.lang.ThreadLocal.ThreadLocalMap#replaceStaleEntry()`:

```java
/**
  * 替换过期数据
  *
  * @param key       ThreadLocalMap的key
  * @param value     ThreadLocalMap的值
  * @param staleSlot 当前元素的下标
  */
private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                       int staleSlot) {
    // ThreadLocalMap的数组结构
    Entry[] tab = table;
    int len = tab.length;
    Entry e;
	
    // 开始探测式清理过期数据的开始下标
    int slotToExpunge = staleSlot;
    
    // 以当前的staleSlot开始，向前迭代查找，找到没有过期的数据，for循环一直碰到Entry为null才会结束
    for (int i = prevIndex(staleSlot, len);
         (e = tab[i]) != null;
         // 如果向前找到了过期数据，更新探测清理过期数据的开始下标为 i，即slotToExpunge=i
         i = prevIndex(i, len))

        if (e.get() == null)
            slotToExpunge = i;

    // 从staleSlot向后查找，也是碰到Entry为null的桶结束
    for (int i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {

        ThreadLocal<?> k = e.get();

        // 碰到 k == key，这说明这里是替换逻辑，替换新数据并且交换当前staleSlot位置
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            // 这说明replaceStaleEntry()一开始向前查找过期数据时并未找到过期的Entry数据，接着向后查找过程中也未发现过期数据，修改开始探测式清理过期数据的下标为当前循环的 index
            if (slotToExpunge == staleSlot)
                slotToExpunge = i;
            // 启发式过期数据清理
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
            return;
        }

        // k == null说明当前遍历的Entry是一个过期数据，slotToExpunge == staleSlot说明，一开始的向前查找数据并未找到过期的Entry。  如果条件成立，则更新slotToExpunge 为当前位置，这个前提是前驱节点扫描时未发现过期数据
        if (k == null && slotToExpunge == staleSlot)
            slotToExpunge = i;
    }

    // 往后迭代的过程中如果没有找到k == key的数据，且碰到Entry为null的数据，则结束了的迭代操作。此时说明这里是一个添加的逻辑，将新的数据添加到table[staleSlot] 对应的slot中。
    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    // 最后判断除了staleSlot以外，还发现了其他过期的slot数据，就要开启清理数据的逻辑
    if (slotToExpunge != staleSlot)
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}
```



### ThreadLocalMap的扩容机制

> 只要使用到数组结构，就一定会有扩容。

### `rehash()`机制

在`ThreadLocalMap.set()`方法的最后，如果执行完启发式清理工作后，未清理到任何数据，且当前散列数组中`Entry`的数量已经达到了列表的扩容阈值`(len*2/3)`，就开始执行`rehash()`逻辑：

```java
if (!cleanSomeSlots(i, sz) && sz >= threshold)
    rehash();
```

接着看下`rehash()`具体实现：

```java
private void rehash() {
    // 清理掉当前散列表内的所有过期的数据
    expungeStaleEntries();

    // 当前散列表长度是否大于阈值*0.75
    if (size >= threshold - threshold / 4)
        // 扩容操作
        resize();
}

private void expungeStaleEntries() {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}
```

这里首先是会进行探测式清理工作，从`table`的起始位置往后清理，上面有分析清理的详细流程。清理完成之后，`table`中可能有一些`key`为`null`的`Entry`数据被清理掉，所以此时通过判断`size >= threshold - threshold / 4` 也就是`size >= threshold* 3/4` 来决定是否扩容。

我们还记得上面进行`rehash()`的阈值是`size >= threshold`，所以当面试官套路我们`ThreadLocalMap`扩容机制的时候 我们一定要说清楚这两个步骤。

![ThreadLocalMap的扩容示例](https://cos.duktig.cn/typora/202109141816837.png)

### `resize()`机制

接着看看具体的`resize()`方法，为了方便演示，我们以`oldTab.len=8`来举例：

![resize()机制](https://cos.duktig.cn/typora/202109141816884.png)

具体包括如下步骤；

1. 首先把数组长度扩容到原来的2倍，`oldLen * 2`，实例化新数组。
2. 遍历for，所有的旧数组中的元素，重新放到新数组中。
3. 在放置数组的过程中，如果发生哈希碰撞，则链式法顺延。
4. 同时这还有检测key值的操作 `if (k == null)`，方便GC。
5. 重新计算`tab`下次扩容的**阈值**

具体代码如下：

```java
private void resize() {
    // 旧数组
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    // 新数组
    Entry[] newTab = new Entry[newLen];
    int count = 0;
    // 循环遍历进行数据迁移
    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        // 条件成立 说明当前桶位值未被GC回收
        if (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                // Help the GC
                e.value = null; 
            } else {
                // 重新计算在新散列表中的桶位
                int h = k.threadLocalHashCode & (newLen - 1);
                 // 解决Hash冲突  向后遍历 直到遇到空桶位
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }
    // 设置阈值  =  散列表长度的2/3
    setThreshold(newLen);
    size = count;
    table = newTab;
}
```



### `ThreadLocalMap.get()`解析

**第一种情况：** 通过查找`key`值计算出散列表中`slot`位置，然后该`slot`位置中的`Entry.key`和查找的`key`一致，则直接返回：

![img](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/thread-local/26.png)

**第二种情况：** `slot`位置中的`Entry.key`和要查找的`key`不一致：

![img](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/thread-local/27.png)

我们以`get(ThreadLocal1)`为例，通过`hash`计算后，正确的`slot`位置应该是 4，而`index=4`的槽位已经有了数据，且`key`值不等于`ThreadLocal1`，所以需要继续往后迭代查找。

迭代到`index=5`的数据时，此时`Entry.key=null`，触发一次探测式数据回收操作，执行`expungeStaleEntry()`方法，执行完后，`index 5,8`的数据都会被回收，而`index 6,7`的数据都会前移，此时继续往后迭代，到`index = 6`的时候即找到了`key`值相等的`Entry`数据，如下图所示：

![img](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/thread-local/28.png)

**总结：**

![ThreadLocalMap.get() 情况总结](https://cos.duktig.cn/typora/202109141819161.png)

按照不同的数据元素存储情况，基本包括如下情况；

1. 直接定位到，没有哈希冲突，直接返回元素即可。
2. 没有直接定位到了，key不同，需要拉链式寻找。
3. 没有直接定位到了，key不同，拉链式寻找，遇到GC清理元素，需要探测式清理，再寻找元素。

具体代码：

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;
    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

**好了**，这部分就是获取元素的源码部分，和我们图中列举的情况是一致的。`expungeStaleEntry`，是发现有 `key == null` 时，进行清理过期元素，并把后续位置的元素，前移。

### ThreadLocal元素清理 解析

`ThreadLocalMap`的两种过期`key`数据清理方式：**探测式清理**和**启发式清理**。

####  探测式清理（expungeStaleEntry）

> 探测式清理，是以当前遇到的 GC 元素开始，向后不断的清理。直到遇到 null 为止，才停止 rehash 计算`Rehash until we encounter null`。

1. 遍历散列数组，从开始位置向后探测清理过期数据，将过期数据的`Entry`设置为`null`；
2. 沿途中碰到未过期的数据则将此数据`rehash`后重新在`table`数组中定位；
3. 如果定位的位置已经有了数据，则会将未过期的数据放到最靠近此位置的`Entry=null`的桶中，使`rehash`后的`Entry`数据距离正确的桶的位置更近一些。

**expungeStaleEntry**

```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    // 在 staleSlot 删除条目
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;
    // 重新哈希直到我们遇到 null
    Entry e;
    int i;
    
    // 以staleSlot位置往后迭代
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        
        // 如果遇到k==null的过期数据，也是清空该槽位数据，然后size--
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            // 如果key没有过期，重新计算当前key的下标位置是不是当前槽位下标位置，如果不是，那么说明产生了hash冲突,此时以新计算出来正确的槽位位置往后迭代，找到最近一个可以存放entry的位置.  
            // 经过迭代后，有过Hash冲突数据的Entry位置会更靠近正确位置，这样的话，查询的时候 效率才会更高。
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;
                // 我们必须扫描直到为空，因为多个条目可能已经过时
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

**以上**，探测式清理在获取元素中使用到； `new ThreadLocal<>().get() -> map.getEntry(this) -> getEntryAfterMiss(key, i, e) -> expungeStaleEntry(i)`

#### 启发式清理（cleanSomeSlots）

```
Heuristically scan some cells looking for stale entries.
This is invoked when either a new element is added, or
another stale one has been expunged. It performs a
logarithmic number of scans, as a balance between no
scanning (fast but retains garbage) and a number of scans
proportional to number of elements, that would find all
garbage but would cause some insertions to take O(n) time.
```

**启发式清理**，有这么一段注释，大概意思是；试探的扫描一些单元格，寻找过期元素，也就是被垃圾回收的元素。*当添加新元素或删除另一个过时元素时，将调用此函数。它执行对数扫描次数，作为不扫描（快速但保留垃圾）和与元素数量成比例的扫描次数之间的平衡，这将找到所有垃圾，但会导致一些插入花费O（n）时间。*

![启发式清理流程](https://cos.duktig.cn/typora/202109141622264.png)

具体代码：

```java
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```

while 循环中不断的右移进行寻找需要被清理的过期元素，最终都会使用 `expungeStaleEntry` 进行处理，这里还包括元素的移位。



## ThreadLocal GC之后key是否为null？

> `ThreadLocal` 的`key`是弱引用，那么在**` threadLocal.get()`**的时候,发生`GC`之后，`key`是否是`null`？

为了搞清楚这个问题，我们需要搞清楚`Java`的**四种引用类型**：

- **强引用**：我们常常new出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候
- **软引用**：使用SoftReference修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
- **弱引用**：使用WeakReference修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
- **虚引用**：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

接着再来看下代码，我们使用反射的方式来看看`GC`后`ThreadLocal`中的数据情况：

```java
public class ThreadLocalDemo {

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, InterruptedException {
        Thread t = new Thread(()->test("abc",false));
        t.start();
        t.join();
        System.out.println("--gc后--");
        Thread t2 = new Thread(() -> test("def", true));
        t2.start();
        t2.join();
    }

    private static void test(String s,boolean isGC)  {
        try {
            new ThreadLocal<>().set(s);
            if (isGC) {
                System.gc();
            }
            Thread t = Thread.currentThread();
            Class<? extends Thread> clz = t.getClass();
            Field field = clz.getDeclaredField("threadLocals");
            field.setAccessible(true);
            Object ThreadLocalMap = field.get(t);
            Class<?> tlmClass = ThreadLocalMap.getClass();
            Field tableField = tlmClass.getDeclaredField("table");
            tableField.setAccessible(true);
            Object[] arr = (Object[]) tableField.get(ThreadLocalMap);
            for (Object o : arr) {
                if (o != null) {
                    Class<?> entryClass = o.getClass();
                    Field valueField = entryClass.getDeclaredField("value");
                    Field referenceField = entryClass.getSuperclass().getSuperclass().getDeclaredField("referent");
                    valueField.setAccessible(true);
                    referenceField.setAccessible(true);
                    System.out.println(String.format("弱引用key:%s,值:%s", referenceField.get(o), valueField.get(o)));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

结果如下：

```java
弱引用key:java.lang.ThreadLocal@433619b6,值:abc
弱引用key:java.lang.ThreadLocal@418a15e3,值:java.lang.ref.SoftReference@bf97a12
--gc后--
弱引用key:null,值:def
```

![img](https://cos.duktig.cn/typora/202109141502178.png)

如图所示，因为这里创建的`ThreadLocal`并没有指向任何值，也就是没有任何引用：

```java
new ThreadLocal<>().set(s);
```

所以这里在`GC`之后，`key`就会被回收，我们看到上面`debug`中的`referent=null`, 如果**改动一下代码：**

![img](https://cos.duktig.cn/typora/202109141502850.png)

这个问题刚开始看，如果没有过多思考，**弱引用**，还有**垃圾回收**，那么肯定会觉得是`null`。

其实是不对的，因为题目说的是在做 `ThreadLocal.get()` 操作，证明其实还是有**强引用**存在的，所以 `key` 并不为 `null`，如下图所示，`ThreadLocal`的**强引用**仍然是存在的。

![image.png](https://cos.duktig.cn/typora/202109141502456.png)

如果我们的**强引用**不存在的话，那么 `key` 就会被回收，也就是会出现我们 `value` 没被回收，`key` 被回收，导致 `value` 永远存在，出现内存泄漏。

## 为什么推荐使用ThreadLocal后，调用其remove操作吗？

探测式清理，其实这也是非常耗时。为此我们在使用 ThreadLocal 一定要记得 `new ThreadLocal<>().remove();` 操作。避免弱引用发生GC后，导致内存泄漏的问题。

## InheritableThreadLocal

我们使用`ThreadLocal`的时候，在异步场景下是无法给子线程共享父线程中创建的线程副本数据的。

`InheritableThreadLocal`可以使子线程继承父线程的值。

特点：

- 子线程和父线程的不可变对象的值，互不影响。

- 子线程从父线程继承可变数据类型时，子线程可以获取**相同对象**最新改变的值。若对象不同则子父线程依旧保持自己的值。

- 重写`childValue()`方法，可对继承的值进一步进行加工。

- `childValue()`只能在创建子线程时生效，而`set()`随时可以生效。

示例：

```java
public class InheritableThreadLocalTest {

    public static void main(String[] args) {
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        ThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();
        threadLocal.set("父类数据:threadLocal");
        inheritableThreadLocal.set("父类数据:inheritableThreadLocal");

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("子线程获取父类ThreadLocal数据：" + threadLocal.get());
                System.out.println("子线程获取父类inheritableThreadLocal数据：" + inheritableThreadLocal.get());
            }
        }).start();
    }

}
```

结果：

```
子线程获取父类ThreadLocal数据：null
子线程获取父类inheritableThreadLocal数据：父类数据:inheritableThreadLocal
```

实现原理是子线程是通过在父线程中通过调用`new Thread()`方法来创建子线程，`Thread#init`方法在`Thread`的构造方法中被调用。在`init`方法中拷贝父线程数据到子线程中：

```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    this.stackSize = stackSize;
    tid = nextThreadID();
}
```

但`InheritableThreadLocal`仍然有缺陷，一般我们做异步化处理都是使用的线程池，而`InheritableThreadLocal`是在`new Thread`中的`init()`方法给赋值的，而线程池是线程复用的逻辑，所以这里会存在问题。

当然，有问题出现就会有解决问题的方案，阿里巴巴开源了一个`TransmittableThreadLocal`组件就可以解决这个问题。



## ThreadLocal一般可以用在什么场景中？

### 解决`SimpleDateFormat`的线程安全问题。

**线程不安全的`SimpleDateFormat`使用**

```java
private static SimpleDateFormat f = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

/**
 * 测试 线程不安全的 SimpleDateFormat
 */
@Test
public void testUnsafeSimpleDateFormat() {
    while (true) {
        new Thread(() -> {
            String dateStr = f.format(new Date());
            try {
                Date parseDate = f.parse(dateStr);
                String dateStrCheck = f.format(parseDate);
                boolean equals = dateStr.equals(dateStrCheck);
                if (! equals) {
                    System.out.println(equals + " " + dateStr + " " + dateStrCheck);
                } else {
                    System.out.println(equals);
                }
            } catch (ParseException e) {
                System.out.println(e.getMessage());
            }
        }).start();
    }
}
```

结果：

```
true
true
true
true
true
true
true
true
true
false 2021-09-14 21:57:59 2021-09-14 23:35:40
false 2021-09-14 21:57:59 5920-09-14 21:57:59
true
true
true
true
true
...
```

上面的结果出现了线程不安全的问题。

**使用`ThreadLocal`解决`SimpleDateFormat`的线程安全问题**

为了线程安全最直接的方式，就是每次调用都直接 `new SimpleDateFormat`。但这样的方式终究不是最好的，所以我们使用 `ThreadLocal` ，来优化这段代码。

```java
private static ThreadLocal<SimpleDateFormat> threadLocal = ThreadLocal.withInitial(() -> new SimpleDateFormat(
        "yyyy-MM-dd HH:mm:ss"));

/**
 * 测试 线程安全的 SimpleDateFormat
 */
@Test
public void testSafeSimpleDateFormat() {
    while (true) {
        new Thread(() -> {
            String dateStr = threadLocal.get().format(new Date());
            try {
                Date parseDate = threadLocal.get().parse(dateStr);
                String dateStrCheck = threadLocal.get().format(parseDate);
                boolean equals = dateStr.equals(dateStrCheck);
                if (! equals) {
                    System.out.println(equals + " " + dateStr + " " + dateStrCheck);
                } else {
                    System.out.println(equals);
                }
            } catch (ParseException e) {
                System.out.println(e.getMessage());
            }
        }).start();
    }
}
```

如上我们把 `SimpleDateFormat` ，放到 `ThreadLocal` 中进行使用，即不需要重复new对象，也避免了线程不安全问题。测试结果如下；

```java
true
true
true
true
true
true
true
...
```



### 在数据库连接上的应用

为每个线程分配一个 JDBC 连接 Connection。这样就可以保证每个线程的都在各自的 Connection 上进行数据库的操作，不会出现 A 线程关了 B线程正在使用的 Connection； 

### **非入侵全链路追踪**，或者**MDC 日志框架**。

生成linkId：

```java
public class TrackContext {

    private static final ThreadLocal<String> trackLocal = new ThreadLocal<>();

    public static void clear(){
        trackLocal.remove();
    }

    public static String getLinkId(){
        return trackLocal.get();
    }

    public static void setLinkId(String linkId){
        trackLocal.set(linkId);
    }

}
```

日志记录用的是`ELK+Logstash`，最后在`Kibana`中进行展示和检索。

现在都是分布式系统统一对外提供服务，项目间调用的关系可以通过 `traceId` 来关联，但是不同项目之间如何传递 `traceId` 呢？

这里我们使用 `org.slf4j.MDC` 来实现此功能，内部就是通过 `ThreadLocal` 来实现的，具体实现如下：

当前端发送请求到**服务 A**时，**服务 A**会生成一个类似`UUID`的`traceId`字符串，将此字符串放入当前线程的`ThreadLocal`中，在调用**服务 B**的时候，将`traceId`写入到请求的`Header`中，**服务 B**在接收请求时会先判断请求的`Header`中是否有`traceId`，如果存在则写入自己线程的`ThreadLocal`中。

![img](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/thread-local/30.png)

图中的`requestId`即为我们各个系统链路关联的`traceId`，系统间互相调用，通过这个`requestId`即可找到对应链路，这里还有会有一些其他场景：

![img](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/thread-local/31.png)

针对于这些场景，我们都可以有相应的解决方案。

















