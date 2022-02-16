# HashCode

## 1. HashCode 为什么使用 31 作为乘数？

**hashCode的计算逻辑中，为什么是31作为乘数**？

![image-20210703205523786](https://gitee.com/koala010/typora/raw/master/img/20210703205530.png)

在获取`hashCode`的源码中可以看到，有一个固定值`31`，在for循环每次执行时进行乘积计算，循环后的公式如下； `s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`。

原因：

- 31 是一个奇质数，如果选择偶数会导致乘积运算时数据溢出。

- 在二进制中，2个5次方是32，那么也就是 `31 * i == (i << 5) - i`。这主要是说乘积运算可以使用位移提升性能，同时目前的JVM虚拟机也会自动支持此类的优化。
- 超过 5 千个单词计算 hashCode， 这个 hashCode 的运算使用 31、33、37、39 和 41 作为乘积，得到的碰撞 结果，31 被使用就很正常了。
  - 31的碰撞概率很小，比较稳定，而且不超过int的取值范围。*199的碰撞概率更小，这就相当于一排奇数的茅坑量多，自然会减少碰撞。**但这个范围值已经远超过int的取值范围了，如果用此数作为乘数，又返回int值，就会丢失数据信息**。*
  - hash的散列比较均匀

# HashMap

## 2. HashMap的基本特点

- HashMap 主要用来存放键值对，它基于哈希表的 Map 接口实现，是常用的 Java 集合之一，是非线程安全的，在多线程环境下可能会存在问题。
- HashMap 允许 null 键和 null 值，在计算哈键的哈希值时，null 键哈希值为 0。null键只能有一个，null值可以有多个。
- HashMap 并不保 证键值对的顺序，这意味着在进行某些操作后，键值对的顺序可能会发生变化。
- `HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。并且， `HashMap` 总是使用 2 的幂作为哈希表的大小。
- JDK1.8 之前 HashMap 由 数组+链表 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。 JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

## 3. 实现一个简单的HashMap

**问题**： 假设我们有一组 7 个字符串，需要存放到数组中，但要求在获取每个元 素的时候时间复杂度是 O(1)。也就是说你不能通过循环遍历的方式进行获取， 而是要定位到数组 ID 直接获取相应的元素。

**方案：**如果说我们需要通过ID从数组中获取元素，那么就需要把每个字符串都计算出一个在数组中的位置ID。*字符串获取ID你能想到什么方式？* 一个字符串最直接的获取跟数字相关的信息就是HashCode，可HashCode的取值范围太大了`[-2147483648, 2147483647]`，不可能直接使用。那么就需要使用HashCode与数组长度做与运算，得到一个可以在数组中出现的位置。如果说有两个元素得到同样的ID，那么这个数组ID下就存放两个字符串。

```java
// 初始化一组字符串
List<String> list = new ArrayList<>();
list.add("jlkk");
list.add("lopi");
list.add("小傅哥");
list.add("e4we");
list.add("alpo");
list.add("yhjk");
list.add("plop");

// 定义要存放的数组
String[] tab = new String[8];

// 循环存放
for (String key : list) {
    int idx = key.hashCode() & (tab.length - 1);  // 计算索引位置
    System.out.println(String.format("key值=%s Idx=%d", key, idx));
    if (null == tab[idx]) {
        tab[idx] = key;
        continue;
    }
    tab[idx] = tab[idx] + "->" + key;
}
// 输出测试结果
System.out.println(JSON.toJSONString(tab));
```

**代码实现思路**：

1. 初始化一组字符串集合，这里初始化了7个。
2. 定义一个数组用于存放字符串，注意这里的长度是8，也就是2的倍数。这样的数组长度才会出现一个 `0111` 除高位以外都是1的特征，也是为了散列。
3. 接下来就是循环存放数据，计算出每个字符串在数组中的位置。`key.hashCode() & (tab.length - 1)`。
4. 在字符串存放到数组的过程，如果遇到相同的元素，进行连接操作`模拟链表的过程`。
5. 最后输出存放结果。

**测试结果**

```java
key值=jlkk Idx=2
key值=lopi Idx=4
key值=小傅哥 Idx=7
key值=e4we Idx=5
key值=alpo Idx=2
key值=yhjk Idx=0
key值=plop Idx=5
测试结果：["yhjk",null,"jlkk->alpo",null,"lopi","e4we->plop",null,"小傅哥"]
```

- 在测试结果首先是计算出每个元素在数组的Idx，也有出现重复的位置。
- 最后是测试结果的输出，1、3、6，位置是空的，2、5，位置有两个元素被链接起来`e4we->plop`。
- 这就达到了我们一个最基本的要求，将串元素散列存放到数组中，最后通过字符串元素的索引ID进行获取对应字符串。这样是HashMap的一个最基本原理，有了这个基础后面就会更容易理解HashMap的源码实现。

1. 初始化一组字符串集合，这里初始化了7个。
2. 定义一个数组用于存放字符串，注意这里的长度是8，也就是2的倍数。这样的数组长度才会出现一个 `0111` 除高位以外都是1的特征，也是为了散列。
3. 接下来就是循环存放数据，计算出每个字符串在数组中的位置。`key.hashCode() & (tab.length - 1)`。
4. 在字符串存放到数组的过程，如果遇到相同的元素，进行连接操作`模拟链表的过程`。
5. 最后输出存放结果。

**测试结果**

```
key值=jlkk Idx=2
key值=lopi Idx=4
key值=小傅哥 Idx=7
key值=e4we Idx=5
key值=alpo Idx=2
key值=yhjk Idx=0
key值=plop Idx=5
测试结果：["yhjk",null,"jlkk->alpo",null,"lopi","e4we->plop",null,"小傅哥"]
```

- 在测试结果首先是计算出每个元素在数组的Idx，也有出现重复的位置。
- 最后是测试结果的输出，1、3、6，位置是空的，2、5，位置有两个元素被链接起来`e4we->plop`。
- 这就达到了我们一个最基本的要求，将串元素散列存放到数组中，最后通过字符串元素的索引ID进行获取对应字符串。这样是HashMap的一个最基本原理，有了这个基础后面就会更容易理解HashMap的源码实现。

**Hash散列示意图**

如果上面的测试结果不能在你的头脑中很好的建立出一个数据结构，那么可以看以下这张散列示意图，方便理解

![7个元素的hash散列示意图](https://gitee.com/koala010/typora/raw/master/img/20210703210928.png)

黄色的索引ID是没有元素存放、绿色的索引ID存放了一个元素、红色的索引ID存放了两个元素

**简单的HashMap有哪些问题**？

以上实现了一个简单的HashMap，或者说还算不上HashMap，只能算做一个散列数据存放的雏形。但这样的一个数据结构放在实际使用中，会有哪些问题呢？

1. 在获取索引ID的计算公式中，需要数组长度是2的倍数，那么**怎么进行初始化这个数组大小**？
2. 数组越小碰撞的越大，数组越大碰撞的越小，**时间与空间如何取舍**？
3. 目前存放7个元素，已经有两个位置都存放了2个字符串，那么**链表越来越长怎么优化**？链表越长意味着检索效率越低
4. 随着元素的不断添加，数组长度不足扩容时，**怎么把原有的元素，拆分到新的位置上去**？
5. 这里所有的元素存放都需要获取一个索引位置，而如果元素的位置不够散列碰撞严重，那么就失去了散列表存放的意义，没有达到预期的性能

以上这些问题可以从HashMap的`扰动函数`、`初始化容量`、`负载因子`、`扩容方法`以及`链表和红黑树转换`的角度考虑。

## 4. 扰动函数

### 4.1 Java8中的扰动函数

在HashMap存放元素时候有这样一段代码来处理哈希值，这是`java 8`的散列值扰动函数，用于优化散列效果：

```java
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### 4.2 为什么使用扰动函数？

理论上来说字符串的`hashCode`是一个int类型值，可以直接作为数组下标，且不会出现碰撞。但是这个`hashCode`的取值范围是`[-2147483648, 2147483647]`，有将近40亿的长度，不可能把数组初始化的这么大，内存是放不下的。

所以HashMap默认大小是16个长度，即 `DEFAULT_INITIAL_CAPACITY = 1 << 4`，所以获取的Hash值并不能直接作为下标使用，需要与数组长度进行取模运算得到一个下标值。

hashMap源码这里不只是直接获取哈希值，还进行了一次扰动计算，`(h = key.hashCode()) ^ (h >>> 16)`。把哈希值右移16位，也就正好是自己长度的一半，之后与原哈希值做异或运算，这样就混合了原哈希值中的高位和低位，增大了**随机性**。计算方式如下图：

![扰动函数计算过程](https://gitee.com/koala010/typora/raw/master/img/20210703212320.png)

总结：**使用扰动函数就是为了增加随机性，让数据元素更加均衡的散列，减少碰撞**。

### 4.3 Java7中的扰动函数

对比一下 JDK1.7 的 HashMap 的 hash 方法源码.

```java
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为**毕竟扰动了 4 次**。

### 4.4 HashMap 的容量必须是 2 的 N 次方，这是为什么？

计算索引位置的公式为：`(n - 1) & hash`，当 n 为 2 的 N 次方时，n - 1 为低位全是 1 的值，此时任何值跟 n - 1 进行 & 运算的结果为该值的低 N 位，达到了和取模同样的效果，实现了均匀分布。实际上，这个设计就是基于公式：`x mod 2^n = x & (2^n - 1)`，因为 & 运算比 mod 具有更高的效率。

## 5. 初始化容量和负载因子

散列数组需要一个2的倍数的长度，因为只有2的倍数在减1的时候，才会出现`01111`这样的值。

那么这里就有一个问题，我们在初始化HashMap的时候，如果传一个17个的值`new HashMap<>(17);`，它会怎么处理呢？

### 5.1 寻找2的倍数的最小值

在HashMap的初始化中，有这样一段方法；

```Java
public HashMap(int initialCapacity, float loadFactor) {
    ...
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

- 阀值`threshold`，通过方法`tableSizeFor`进行计算，是根据初始化来计算的。
- 这个方法也就是要寻找比初始值大的，最小的那个2进制数值。比如传了17，我应该找到的是32。

计算阀值大小的方法；

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

- MAXIMUM_CAPACITY = 1 « 30，这个是临界范围，也就是最大的Map集合。
- 乍一看可能有点晕😵怎么都在向右移位1、2、4、8、16，这主要是为了把二进制的各个位置都填上1，当二进制的各个位置都是1以后，就是一个标准的2的倍数减1了，最后把结果加1再返回即可。

图示过程如下：

<img src="https://gitee.com/koala010/typora/raw/master/img/20210703212959.png" alt="初始化容量为2的倍数最小值的过程" style="zoom:80%;" />

### 5.2 负载因子

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

**负载因子是做什么的？**

在HashMap中，**负载因子决定了数据量多少了以后进行扩容**。*这里要提到上面做的HashMap例子，我们准备了7个元素，但是最后还有3个位置空余，2个位置存放了2个元素。* 所以可能即使你**数据比数组容量大时也是不一定能正正好好的把数组占满的，而是在某些小标位置出现了大量的碰撞，只能在同一个位置用链表存放，那么这样就失去了Map数组的性能**。

所以，要选择一个合理的大小下进行扩容，**默认值0.75就是说当阀值容量占了3/4时赶紧扩容，减少Hash碰撞**。

同时0.75是一个默认构造值，在创建HashMap也可以调整，比如你希望用更多的空间换取时间，可以把负载因子调的更小一些，减少碰撞。

### 5.3 扩容元素拆分

为什么扩容？因为数组长度不足了。那**扩容最直接的问题，就是需要把元素拆分到新的数组中**。拆分元素的过程中，原jdk1.7中会需要重新计算哈希值，但是到jdk1.8中已经进行优化，不在需要重新计算，提升了拆分的性能。

#### 5.3.1 测试数据，得出Java8的元素拆分规律

```java
@Test
public void test_hashMap() {
    List<String> list = new ArrayList<>();
    list.add("jlkk");
    list.add("lopi");
    list.add("jmdw");
    list.add("e4we");
    list.add("io98");
    list.add("nmhg");
    list.add("vfg6");
    list.add("gfrt");
    list.add("alpo");
    list.add("vfbh");
    list.add("bnhj");
    list.add("zuio");
    list.add("iu8e");
    list.add("yhjk");
    list.add("plop");
    list.add("dd0p");
    for (String key : list) {
        int hash = key.hashCode() ^ (key.hashCode() >>> 16);
        System.out.println("字符串：" + key + " \tIdx(16)：" + ((16 - 1) & hash) + " \tBit值：" + Integer.toBinaryString(hash) + " - " + Integer.toBinaryString(hash & 16) + " \t\tIdx(32)：" + ((
        System.out.println(Integer.toBinaryString(key.hashCode()) +" "+ Integer.toBinaryString(hash) + " " + Integer.toBinaryString((32 - 1) & hash));
    }
}
```

**测试结果**

```java
字符串：jlkk 	Idx(16)：3 	Bit值：1100011101001000010011 - 10000 		Idx(32)：19
1100011101001000100010 1100011101001000010011 10011
字符串：lopi 	Idx(16)：14 	Bit值：1100101100011010001110 - 0 		Idx(32)：14
1100101100011010111100 1100101100011010001110 1110
字符串：jmdw 	Idx(16)：7 	Bit值：1100011101010100100111 - 0 		Idx(32)：7
1100011101010100010110 1100011101010100100111 111
字符串：e4we 	Idx(16)：3 	Bit值：1011101011101101010011 - 10000 		Idx(32)：19
1011101011101101111101 1011101011101101010011 10011
字符串：io98 	Idx(16)：4 	Bit值：1100010110001011110100 - 10000 		Idx(32)：20
1100010110001011000101 1100010110001011110100 10100
字符串：nmhg 	Idx(16)：13 	Bit值：1100111010011011001101 - 0 		Idx(32)：13
1100111010011011111110 1100111010011011001101 1101
字符串：vfg6 	Idx(16)：8 	Bit值：1101110010111101101000 - 0 		Idx(32)：8
1101110010111101011111 1101110010111101101000 1000
字符串：gfrt 	Idx(16)：1 	Bit值：1100000101111101010001 - 10000 		Idx(32)：17
1100000101111101100001 1100000101111101010001 10001
字符串：alpo 	Idx(16)：7 	Bit值：1011011011101101000111 - 0 		Idx(32)：7
1011011011101101101010 1011011011101101000111 111
字符串：vfbh 	Idx(16)：1 	Bit值：1101110010111011000001 - 0 		Idx(32)：1
1101110010111011110110 1101110010111011000001 1
字符串：bnhj 	Idx(16)：0 	Bit值：1011100011011001100000 - 0 		Idx(32)：0
1011100011011001001110 1011100011011001100000 0
字符串：zuio 	Idx(16)：8 	Bit值：1110010011100110011000 - 10000 		Idx(32)：24
1110010011100110100001 1110010011100110011000 11000
字符串：iu8e 	Idx(16)：8 	Bit值：1100010111100101101000 - 0 		Idx(32)：8
1100010111100101011001 1100010111100101101000 1000
字符串：yhjk 	Idx(16)：8 	Bit值：1110001001010010101000 - 0 		Idx(32)：8
1110001001010010010000 1110001001010010101000 1000
字符串：plop 	Idx(16)：9 	Bit值：1101001000110011101001 - 0 		Idx(32)：9
1101001000110011011101 1101001000110011101001 1001
字符串：dd0p 	Idx(16)：14 	Bit值：1011101111001011101110 - 0 		Idx(32)：14
1011101111001011000000 1011101111001011101110 1110
```

**分析**：

- 这里随机使用一些字符串计算他们分别在16位长度和32位长度数组下的索引分配情况，看哪些数据被重新路由到了新的地址。
- 观察出一个非常重要的信息，**原哈希值与扩容新增出来的长度16，进行&运算，如果值等于0，则下标位置不变。如果不为0，那么新的位置则是原来位置上加16**。（这个规律很重要）
- 这样一来，就不需要在重新计算每一个数组中元素的哈希值了。

#### 5.3.2 数据迁移过程

![数据迁移过程](https://gitee.com/koala010/typora/raw/master/img/20210703214025.png)

上图表示原16位长度数组元素转移的过程，其中黄色区域元素`zuio`因计算结果 `hash & oldCap` 不为1，则被迁移到下标位置24（24由`8+16`得出）。同时还是用重新计算哈希值的方式验证了，确实分配到24的位置，因为这是在二进制计算中补1的过程，所以可以通过上面简化的方式确定哈希值的位置。

## 6. 插入操作分析

### 6.1 HashMap插入流程分析

![HashMap插入流程分析](https://gitee.com/koala010/typora/raw/master/img/20210703214620.png)

以上就是HashMap中一个数据插入的整体流程，包括了**计算下标、何时扩容、何时链表转红黑树**等问题，具体流程分析如下：

1. 根据key值，通过扰动函数`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);`获取新的哈希值。

2. 判断tab是否位空或者长度为0，如果是则进行**初始化（扩容）**操作。

   ```java
   if ((tab = table) == null || (n = tab.length) == 0)
       n = (tab = resize()).length;
   ```

3. 如果哈希值计算的数组下标没有数据，直接插入；如果有数据并且key相等，那么覆盖数据。下标：`tab[i = (n - 1) & hash`。

4. 如果key不相等（hash冲突），判断tab[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点。

5. 如果链表中插入节点的时候，链表长度是否大于等于8。否插入链表;是则执行`treeifyBin(tab, hash);`。

   `treeifyBin`,是一个链表转树的方法，但不是所有的链表长度为8后都会转成树，还需要判断存放key值的数组桶长度是否小于64 `MIN_TREEIFY_CAPACITY`。如果小于则需要扩容，扩容后链表上的数据会被拆分散列的相应的桶节点上，也就把链表长度缩短了。反之则进行链表转红黑树。

6. 最后所有元素处理完成后，判断是否超过阈值`threshold`；超过则扩容。

### 6.2 `put`方法源码分析

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 初始化桶数组 table，table 被延迟到插入新数据时再进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果桶中不包含键值对节点引用，则将新键值对节点的引用存入桶中即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 如果键的值以及节点 hash 等于链表中的第一个键值对节点时，则将 e 指向该键值对
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
            
        // 如果桶中的引用类型为 TreeNode，则调用红黑树的插入方法
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 对链表进行遍历，并统计链表长度
            for (int binCount = 0; ; ++binCount) {
                // 链表中不包含要插入的键值对节点时，则将该节点接在链表的最后
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果链表长度大于或等于树化阈值，则进行树化操作
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                
                // 条件为 true，表示当前链表包含要插入的键值对，终止遍历
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // 判断要插入的键值对是否存在 HashMap 中
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // onlyIfAbsent 表示是否仅在 oldValue 为 null 的情况下更新键值对的值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 键值对数量超过阈值时，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### 6.3 扩容机制

HashMap是基于数组+链表和红黑树实现的，但用于存放key值得的数组桶的长度是固定的，由初始化决定。

随着数据的插入数量增加以及负载因子的作用下，就需要扩容来存放更多的数据。而扩容中有一个非常重要的点，就是jdk1.8中的优化操作，可以不需要再重新计算每一个元素的哈希值。这一点可参看上文的[扩容元素拆分](#5.3 扩容元素拆分)。

**扩容的代码**：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // Cap 是 capacity 的缩写，容量。如果容量不为空，则说明已经初始化。
    if (oldCap > 0) {
        // 如果容量达到最大1 << 30则不再扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        
        // 按旧容量和阀值的2倍计算新容量和阀值
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
    
        // initial capacity was placed in threshold 翻译过来的意思，如下；
        // 初始化时，将 threshold 的值赋值给 newCap，
        // HashMap 使用 threshold 变量暂时保存 initialCapacity 参数的值
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        // 这一部分也是，源代码中也有相应的英文注释
        // 调用无参构造方法时，数组桶数组容量为默认容量 1 << 4; aka 16
        // 阀值；是默认容量与负载因子的乘积，0.75
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    
    // newThr为0，则使用阀值公式计算容量
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    
    @SuppressWarnings({"rawtypes","unchecked"})
        // 初始化数组桶，用于存放key
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 如果旧数组桶，oldCap有值，则遍历将键值映射到新数组桶中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 这里split，是红黑树拆分操作。在重新映射时操作的。
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 这里是链表，如果当前是按照链表存放的，则将链表节点按原顺序进行分组{这里有专门的文章介绍，如何不需要重新计算哈希值进行拆分《HashMap核心知识，扰动函数、负载因子、扩容链表拆分，深度学习》}
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    
                    // 将分组后的链表映射到桶中
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

**分析**：

1. 扩容时计算出新的newCap、newThr，这是两个单词的缩写，一个是Capacity ，另一个是阀Threshold
2. newCap用于创新的数组桶 `new Node[newCap];`
3. 随着扩容后，原来那些因为哈希碰撞，存放成链表和红黑树的元素，都需要进行拆分存放到新的位置中。

### 6.4 链表转红黑树

#### 6.4.1 为什么使用8为链表转红黑树的阈值？

HashMap这种散列表的数据结构，最大的性能在于可以O(1)时间复杂度定位到元素，但因为哈希碰撞不得已在一个下标里存放多组数据，那么jdk1.8之前的设计只是采用链表的方式进行存放，如果需要从链表中定位到数据时间复杂度就是O(n)，链表越长性能越差。

因为在jdk1.8中把过长的链表也就是8个，优化为自平衡的红黑树结构，以此让定位元素的时间复杂度优化近似于O(logn)，这样来提升元素查找的效率。但也不是完全抛弃链表，因为在元素相对不多的情况下，链表的插入速度更快，所以综合考虑下设定阈值为8才进行红黑树转换操作。

#### 6.4.2 链表转红黑树的过程

![链表转红黑树的过程](https://gitee.com/koala010/typora/raw/master/img/20210703221222.png)

链表到底怎么转换成红黑树，以及具体代码的实现分析，在后续的文章中更新。

#### 6.4.3 HashMap链表树化源码

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 这块就是我们上面提到的，不一定树化还可能只是扩容。主要桶数组容量是否小于64 MIN_TREEIFY_CAPACITY 
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
    	// 又是单词缩写；hd = head (头部)，tl = tile (结尾)
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 将普通节点转换为树节点，但此时还不是红黑树，也就是说还不一定平衡
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            // 转红黑树操作，这里需要循环比较，染色、旋转。关于红黑树，在下一章节详细讲解
            hd.treeify(tab);
    }
}
```

这一部分链表树化的操作因为封装的原因并不复杂，复杂点在于下一层的红黑树转换上。

**分析**：

1. 链表树化的条件有两点；链表长度大于等于8、桶容量大于64，否则只是扩容，不会树化。
2. 链表树化的过程中是先由链表转换为树节点，此时的树可能不是一颗平衡树。同时在树转换过程中会记录链表的顺序，`tl.next = p`，这主要方便后续树转链表和拆分更方便。
3. 链表转换成树完成后，在进行红黑树的转换。

### 6.5 红黑树转链表

在链表转换树的过程中，记录了原有链表的顺序。那么，在红黑树转链表时候，直接把TreeNode转换为Node即可，源码如下：

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    // 遍历TreeNode
    for (Node<K,V> q = this; q != null; q = q.next) {
    	// TreeNode替换Node
        Node<K,V> p = map.replacementNode(q, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }
    return hd;
}

// 替换方法
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
```

因为记录了链表关系，所以替换过程很容易。所以好的数据结构可以让操作变得更加容易。

## 7. 查找操作分析

### 7.1 查找操作流程

<img src="https://gitee.com/koala010/typora/raw/master/img/20210703221824.png" alt="查找操作流程图" style="zoom:80%;" />

### 7.2 代码分析

```java
public V get(Object key) {
    Node<K,V> e;
    // 同样需要经过扰动函数计算哈希值
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 判断桶数组的是否为空和长度值
    if ((tab = table) != null && (n = tab.length) > 0 &&
        // 计算下标，哈希值与数组长度-1
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // TreeNode 节点直接调用红黑树的查找方法，时间复杂度O(logn)
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 如果是链表就依次遍历查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

**分析**：

1. 扰动函数的使用，获取新的哈希值，这在上一章节已经讲过
2. 下标的计算，同样也介绍过 `tab[(n - 1) & hash]) `
3. 确定了桶数组下标位置，接下来就是对红黑树和链表进行查找和遍历操作了

## 8. 删除操作分析

```java
 public V remove(Object key) {
     Node<K,V> e;
     return (e = removeNode(hash(key), key, null, false, true)) == null ?
         null : e.value;
 }
 
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    // 定位桶数组中的下标位置，index = (n - 1) & hash
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 如果键的值与链表第一个节点相等，则将 node 指向该节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 树节点，调用红黑树的查找方法，定位节点。
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 遍历链表，找到待删除节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        
        // 删除节点，以及红黑树需要修复，因为删除后会破坏平衡性。链表的删除更加简单。
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
} 
```

**分析**：

- 删除的操作比较简单，没有太多的复杂的逻辑。
- 红黑树的操作因为被包装了，使用上也是很容易。

## 9. 遍历操作分析

### 9.1 遍历操作的无序性

HashMap中的遍历也是非常常用的API方法，包括；

**KeySet**

```java
 for (String key : map.keySet()) {
     System.out.print(key + " ");
 }
```

**EntrySet**

```java
 for (HashMap.Entry entry : map.entrySet()) {
     System.out.print(entry + " ");
 }
```

从方法上以及日常使用都知道，KeySet是遍历是无序的，但每次使用不同方式遍历包括`keys.iterator()`，它们遍历的结果是固定的。

那么从实现的角度来看，这些种遍历都是从散列表中的链表和红黑树获取集合值，那么他们有一个什么固定的规律吗？

### 9.2 代码测试

测试的场景和前提；

1. 这里我们要设定一个既有红黑树又有链表结构的数据场景
2. 为了可以有这样的数据结构，我们最好把HashMap的初始长度设定为64，避免在链表超过8位后扩容，而是直接让其转换为红黑树。
3. 找到18个元素，分别放在不同节点(这些数据通过程序计算得来)；
   1. 桶数组02节点：24、46、68
   2. 桶数组07节点：29
   3. 桶数组12节点：150、172、194、271、293、370、392、491、590

**代码测试**

```java
@Test
public void test_Iterator() {
    Map<String, String> map = new HashMap<String, String>(64);
    map.put("24", "Idx：2");
    map.put("46", "Idx：2");
    map.put("68", "Idx：2");
    map.put("29", "Idx：7");
    map.put("150", "Idx：12");
    map.put("172", "Idx：12");
    map.put("194", "Idx：12");
    map.put("271", "Idx：12");
    System.out.println("排序01：");
    for (String key : map.keySet()) {
        System.out.print(key + " ");
    }
    
    map.put("293", "Idx：12");
    map.put("370", "Idx：12");
    map.put("392", "Idx：12");
    map.put("491", "Idx：12");
    map.put("590", "Idx：12");
    System.out.println("\n\n排序02：");
    for (String key : map.keySet()) {
        System.out.print(key + " ");
    }    
    
    map.remove("293");
    map.remove("370");
    map.remove("392");
    map.remove("491");
    map.remove("590");
    System.out.println("\n\n排序03：");
    for (String key : map.keySet()) {
        System.out.print(key + " ");
    }
    
}
```

这段代码分别测试了三种场景，如下；

1. 添加元素，在HashMap还是只链表结构时，输出测试结果01
2. 添加元素，在HashMap转换为红黑树时候，输出测试结果02
3. 删除元素，在HashMap转换为链表结构时，输出测试结果03

```java
排序01：
24 46 68 29 150 172 194 271 

排序02：
24 46 68 29 271 150 172 194 293 370 392 491 590 

排序03：
24 46 68 29 172 271 150 194 
```

从map.keySet()测试结果可以看到，如下信息：

**01情况下，排序定位哈希值下标和链表信息**

![只是链表结构时的遍历信息](https://gitee.com/koala010/typora/raw/master/img/20210703225247.png)

**02情况下，因为链表转换为红黑树，树根会移动到数组头部。`moveRootToFront()方法`**

![链表转红黑树的遍历信息](https://gitee.com/koala010/typora/raw/master/img/20210703225254.png)

**03情况下，因为删除了部分元素，红黑树退化成链表**

![红黑树转链表后的遍历信息](https://gitee.com/koala010/typora/raw/master/img/20210703225307.png)



## 参看

- [面试阿里，HashMap 这一篇就够了](https://blog.csdn.net/v123411739/article/details/106324537)
