# Redis常见的数据结构（数据类型）

## String

### 基本介绍

String是Redis最基本的类型，一个key对应一个value，**一个Redis中字符串value最多可以是512M**。

String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

使用场景：常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

### 数据结构

String的数据结构为**简单动态字符串**(Simple Dynamic String，缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用**预分配冗余空间的方式来减少内存的频繁分配**。

![String数据结构](https://cos.duktig.cn/typora/202111091645431.png)

如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。

当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

## List

### 基本介绍

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。它是单键多值的。

它的底层实际是个**双向链表**，**对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差**。

![image-20211109164542487](https://cos.duktig.cn/typora/202111091645367.png)

**应用场景:** 发布与订阅或者说消息队列、慢查询。

### 数据结构

List的数据结构为快速链表quickList。

- 首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。
- 当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

![image-20211109164705085](https://cos.duktig.cn/typora/202111091647949.png)

Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

#### linkedList

与`Java`中的`LinkedList`类似，`Redis`中的`linkedList`是一个双向链表，也是由一个个节点组成的。`Redis`中借助`C`语言实现的链表节点结构如下所示：

```
//定义链表节点的结构体 
typedf struct listNode{
    //前一个节点
    struct listNode *prev;
    //后一个节点
    struct listNode *next;
    //当前节点的值的指针
    void *value;
}listNode;
```

`pre`指向前一个节点，`next`指针指向后一个节点，`value`保存着当前节点对应的数据对象。`listNode`的示意图如下所示：

![img](https://cos.duktig.cn/typora/202111091749843.png)

链表的结构如下：

```
typedf struct list{
    //头指针
    listNode *head;
    //尾指针
    listNode *tail;
    //节点拷贝函数
    void *(*dup)(void *ptr);
    //释放节点函数
    void *(*free)(void *ptr);
    //判断两个节点是否相等的函数
    int (*match)(void *ptr,void *key);
    //链表长度
    unsigned long len;
}
```

`head`指向链表的头节点，`tail`指向链表的尾节点，`dup`函数用于链表转移复制时对节点`value`拷贝的一个实现，一般情况下使用**等号**足以，但在某些特殊情况下可能会用到节点转移函数，默认可以给这个函数赋值`NULL`即表示使用等号进行节点转移。`free`函数用于释放一个节点所占用的内存空间，默认赋值`NULL`的话，即使用`Redis`自带的`zfree`函数进行内存空间释放。`match`函数是用来比较两个链表节点的`value`值是否相等，相等返回1，不等返回0。`len`表示这个链表共有多少个节点，这样就可以在`O(1)`的时间复杂度内获得链表的长度。

链表的结构如下所示：

![img](https://cos.duktig.cn/typora/202111091750815.png)

#### ZipList

`Redis`的`zipList`结构如下所示：

```c
typedf struct ziplist<T>{
    //压缩列表占用字符数
    int32 zlbytes;
    //最后一个元素距离起始位置的偏移量，用于快速定位最后一个节点
    int32 zltail_offset;
    //元素个数
    int16 zllength;
    //元素内容
    T[] entries;
    //结束位 0xFF
    int8 zlend;
}ziplist
```

`zipList`的结构如下所示：

![ziplist](https://cos.duktig.cn/typora/202111091737927.png)

注意到`zltail_offset`这个参数，有了这个参数就可以快速定位到最后一个`entry`节点的位置，然后开始倒序遍历，也就是说`zipList`支持双向遍历。

下面是`entry`的结构：

```
typede struct entry{
    //前一个entry的长度
    int<var> prelen;
    //元素类型编码
    int<var> encoding;
    //元素内容
    optional byte[] content;
}entry
```

`prelen`保存的是前一个`entry`节点的长度，这样在倒序遍历时就可以通过这个参数定位到上一个`entry`的位置。`encoding`保存了`content`的编码类型。`content`则是保存的元素内容，它是`optional`类型的，表示这个字段是可选的。当`content`是很小的整数时，它会内联到`content`字段的尾部。`entry`结构的示意图如下所示：

![entry](https://cos.duktig.cn/typora/202111091738937.png)

好了，那现在我们思考一个问题，为什么有了`linkedList`还有设计一个`zipList`呢？就像`zipList`的名字一样，它是一个压缩列表，是为了节约内存而开发的。相比于`linkedList`，其少了`pre`和`next`两个指针。在`Redis`中，`pre`和`next`指针就要占用16个字节（64位系统的一个指针就是8个字节）。另外，`linkedList`的每个节点的内存都是单独分配，加剧内存的碎片化，影响内存的管理效率。与之相对的是，`zipList`是由连续的内存组成的，这样一来，由于内存是连续的，就减少了许多内存碎片和指针的内存占用，进而节约了内存。

`zipList`遍历时，先根据`zlbytes`和`zltail_offset`定位到最后一个`entry`的位置，然后再根据最后一个`entry`里的`prelen`时确定前一个`entry`的位置。

**连锁更新**

上面说到了，`entry`中有一个`prelen`字段，它的长度要么是1个字节，要么都是5个字节：

- 前一个节点的长度小于254个字节，则`prelen`长度为1字节；
- 前一个节点的长度大于254字节，则`prelen`长度为5字节；

假设现在有一组压缩列表，长度都在250~253字节之间，突然新增一个`entry`节点，这个`entry`节点长度大于等于254字节。由于新的`entry`节点大于等于254字节，这个`entry`节点的`prelen`为5个字节，随后会导致其余的所有`entry`节点的`prelen`增大为5字节。

![连锁更新](https://cos.duktig.cn/typora/202111091740545.png)

同样地，删除操作也会导致出现**连锁更新**这种情况，假设在某一时刻，插入一个长度大于等于254个字节的`entry`节点，同时删除其后面的一个长度小于254个字节的`entry`节点，由于小于254的`entry`节点的删除，大于等于254个字节的`entry`节点将会与后面小于254个字节的`entry`节点相连，此时就与新增一个长度大于等于254个字节的`entry`节点时的情况一样，将会发生连续更新。发生连续更新时，`Redis`需要不断地对压缩列表进行**内存分配工作**，直到结束。

#### quickList

在`Redis`3.2版本之后，`list`的底层实现方式又多了一种，`quickList`。`qucikList`是由`zipList`和双向链表`linkedList`组成的混合体。它将`linkedList`按段切分，每一段使用`zipList`来紧凑存储，多个`zipList`之间使用双向指针串接起来。示意图如下所示：

![quicklist](https://cos.duktig.cn/typora/202111091743491.png)

节点`quickListNode`的定义如下：

```c
typedf struct quicklistNode{
    //前一个节点
    quicklistNode* prev;
    //后一个节点
    quicklistNode* next;
    //压缩列表
    ziplist* zl;	
    //ziplist大小
    int32 size;		
    //ziplist 中元素数量
    int16 count;
    //编码形式 存储 ziplist 还是进行 LZF 压缩储存的zipList
    int2 encoding;			
    ...
}quickListNode
```

`quickList`的定义如下所示：

```c
typedf struct quicklist{
    //指向头结点
    quicklistNode* head;
    //指向尾节点
    quicklistNode* tail;
    //元素总数
    long count;
    //quicklistNode节点的个数
    int nodes;	
    //压缩算法深度
    int compressDepth;		
    ...
}quickList
```

上述代码简单地表示了`quickList`的大致结构，为了进一步节约空间，`Redis`还会对`zipList`进行压缩存储，使用**LZF**算法进行压缩，可以选择压缩深度。

#### 每个zipList可以存储多少个元素？

想要了解这个问题，就得打开`redis.conf`文件了。在`DVANCED CONFIG`下面有着清晰的记载。

```
# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
list-max-ziplist-size -2
```

`quickList`内部默认单个`zipList`长度为**8k**字节，即`list-max-ziplist-size`的值设置为**-2**，超出了这个阈值，就会重新生成一个`zipList`来存储数据。根据注释可知，性能最好的时候就是就是`list-max-ziplist-size`为**-1**和**-2**，即分别是**4kb和8kb**的时候，当然，这个值也可以被设置为正数，当`list-max-ziplist-szie`为**正数n**时，表示每个`quickList`节点上的`zipList`最多包含**n个**数据项。

#### 压缩深度

上面提到过，`quickList`中可以使用压缩算法对`zipList`进行进一步的压缩，这个算法就是**[LZF算法](https://blog.csdn.net/u012319493/article/details/83653860)**，这是一种无损压缩算法，具体可以参考上面的链接。使用压缩算法对`zipList`进行压缩后，`zipList`的结构如下所示：

```
typedf struct ziplist_compressed{
    //元素个数
    int32 size;
    //元素内容
    byte[] compressed_data
}
```

此时`quickList`的示意图如下所示：

![img](https://cos.duktig.cn/typora/202111091748141.png)]

当然，在`redis.conf`文件中的`DVANCED CONFIG`下面也可以对压缩深度进行配置。

```
# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0
```

`list-compress-depth`这个参数表示**一个`quickList`两端不被压缩的节点个数。**需要注意的是，这里的节点个数是指`quicklist`双向链表的节点个数，而不是指`ziplist`里面的数据项个数。实际上，一个`quicklist`节点上的`ziplist`，如果被压缩，就是整体被压缩的。

- `quickList`默认的压缩深度为**0**，也就是不开启压缩
- 当`list-compress-depth`为1，表示`quickList`的两端各有1个节点不进行压缩，中间结点进行压缩；
- 当`list-compress-depth`为2，表示`quickList`的首尾2个节点不进行压缩，中间结点进行压缩；
- 以此类推

从上面可以看出，对于`quickList`来说，其首尾两个节点永远不会被压缩。

## Hash

### 基本介绍

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。类似Java里面的 `Map<String,Object>`

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储，主要如下：

![image-20211109165533101](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20211109165533101.png)

**应用场景:** 系统中对象数据的存储。

### 数据结构

Hash类型对应的数据结构是两种：ziplist（压缩列表），dict（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用dict。

### Dict

字典`dict`作为一种常用的数据结构，`C`语言内部并不具备，因而`Redis`的开发人员自己设计和开发了`Redis`中的`dict`结构。

#### dict

其定义如下：

```c
typedf struct dict{
    dictType *type;//类型特定函数，包括一些自定义函数，这些函数使得key和
                   //value能够存储
    void *private;//私有数据
    dictht ht[2];//两张hash表 
    int rehashidx;//rehash索引，字典没有进行rehash时，此值为-1
    unsigned long iterators; //正在迭代的迭代器数量
}dict;
```

- `type`和`private`这两个属性是为了实现字典多态而设置的，当字典中存放着不同类型的值，对应的一些复制，比较函数也不一样，这两个属性配合起来可以实现多态的方法调用；
- `ht[2]`，两个`hash`表
- `rehashidx`，这是一个辅助变量，用于记录`rehash`过程的进度，以及是否正在进行`rehash`等信息，当此值为**-1**时，表示该`dict`此时没有`rehash`过程
- `iterators`，记录此时`dict`有几个迭代器正在进行遍历过程

#### **dictht**

由上面可以看出，`dict`本质上是对哈希表`dictht`的一个简单封装，`dictht`的定义如下所示：

```c
typedf struct dictht{
    dictEntry **table;//存储数据的数组 二维
    unsigned long size;//数组的大小
    unsigned long sizemask;//哈希表的大小的掩码，用于计算索引值，总是等于 
                           //size-1
    unsigned long used;//// 哈希表中中元素个数
}dictht;
```

`table`是一个`dictEntry`类型的数组，用于真正存储数据；`size`表示`table`这个数组的大小；`sizemask`用于计算索引位置，且总是等于`size-1`；`used`表示`dictht`中已有的节点数量，其示意图如下所示：
![img](https://cos.duktig.cn/typora/202111091726815.png)

#### **dictEntry**

上面分析`dictht`时说到，真正存储数据的结构是`dictEntry`数组，其结构定义如下：

```c
typedf struct dictEntry{
    void *key;//键
    union{
        void val;
        unit64_t u64;
        int64_t s64;
        double d;
    }v;//值
    struct dictEntry *next；//指向下一个节点的指针
}dictEntry;
```

其示意图如下所示：

![image-20200831175713878](https://cos.duktig.cn/typora/202111091727359.png)

最后整个`dict`的结构示意图如上所示：

上图是一个没有处于`rehash`状态下的字典`dict`，整个`dict`中有两个哈希表`dictht`，其中一个哈希表存储数据，另一个哈希表为空。

#### **扩容和缩容**

当哈希表中元素数量逐渐增加时，此时产生`hash冲突`的概率逐渐增大，且由于`dict`也是采用**拉链法**解决`hash冲突`的，随着`hash冲突`概率上升，链表会越来越长，这就会导致查找效率下降。相反，当元素不断减少时，元素占用`dict`的空间就越少，出于对内存的极致利用，此时就需要进行缩容操作。

既然说到扩容和缩容，熟悉`Java`集合的小伙伴是不是想到了什么。不错，那就是**负载因子**。**负载因子一般用于描述集合当前被填充的程度**。在`Redis`的字典`dict`中，**负责因子=哈希表中已保存节点数量/哈希表的大小**，即：

```
load factor = ht[0].used / ht[0].size
```

`Redis`中，三条关于扩容和缩容的规则：

- 没有执行BGSAVE和BGREWRITEAOF指令的情况下，哈希表的负载因子大于等于1时进行扩容；
- 正在执行BGSAVE和BGREWRITEAOF指令的情况下，哈希表的负载因大于等于5时进行扩容；
- 负载因子小于0.1时，`Redis`自动开始对哈希表进行收缩操作；

其中，扩容和缩容的数量大小也有一定的规则：

- 扩容：**扩容后的`dictEntry`数组数量为第一个大于等于`ht[0].used*2`的`2^n`**；
- 缩容：**缩容后的`dictEntry`数组数量为第一个大于等于`ht[0].used`的`2^n`**；

## Set

### 基本介绍

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的无序集合。它 **底层其实是一个value为null的hash表** ，所以添加，删除，查找的 **复杂度都是O(1)**。

使用场景：可以基于 set 轻易实现**交集、并集、差集**的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。

### 数据结构

Set数据结构是dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

## ZSet

### 基本介绍

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员都关联了一个**评分**（score）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。**集合的成员是唯一的，但是评分可以是重复的** 。

因为元素是有序的，所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

**应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

### 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构 `Map<String, Double>`，可以给每一个元素value赋予一个权重score，另一方面它又类似于`TreeSet`，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构：

1. hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

2. 跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

zset也有两种不同的实现，分别是`zipList`和`skipList`：

- zipList：满足以下两个条件
  - `[score,value]`键值对数量少于128个；
  - 每个元素的长度小于64字节；
- skipList：不满足以上两个条件时使用跳表、组合了hash和skipList
  - `hash`用来存储`value`到`score`的映射，这样就可以在`O(1)`时间内找到`value`对应的分数；
  - `skipList`按照**从小到大**的顺序存储分数
  - `skipList`每个元素的值都是`[socre,value]`对

### 跳跃表

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。

Redis采用的是跳跃表。跳表可以保证增、删、查等操作时的时间复杂度为`O(logN)`，这个性能可以与平衡树相媲美，但实现方式上却更加简单，唯一美中不足的就是跳表占用的空间比较大，其实就是一种**空间换时间**的思想。

实例：对比有序链表和跳跃表，从链表中查询出51

（1）  有序链表：

![image-20211109171136112](https://cos.duktig.cn/typora/202111091711455.png)

要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到。共需要6次比较。

（2）  跳跃表

![image-20211109171219713](https://cos.duktig.cn/typora/202111091712526.png)



- 从第2层开始，1节点比51节点小，向后比较。
- 21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层
- 在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下
- 在第0层，51节点为要查找的节点，节点被找到，共查找4次。

从此可以看出跳跃表比有序链表效率要高。

## Bitmaps

### 基本介绍

现代计算机用二进制（位） 作为信息的基础单位， 1个字节等于8位， 例如“abc”字符串是由3个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的ASCII码分别是97、 98、 99， 对应的二进制分别是01100001、 01100010和01100011，如下图

![image-20211111100250796](https://cos.duktig.cn/typora/202111111002687.png)                             

合理地使用操作位能够有效地提高内存使用率和开发效率。

   Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1）  Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2）  Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

 ![image-20211111100301872](https://cos.duktig.cn/typora/202111111003757.png)

### 相关命令

#### 1、setbit 

设置Bitmaps中某个偏移量的值（0或1）(offset:偏移量从0开始)

```sh
setbit<key><offset><value>
```

每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id。

设置键的第offset个位的值（从0算起） ， 假设现在有20个用户，userid=1， 6， 11， 15， 19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图:

![image-20211111100540833](https://cos.duktig.cn/typora/202111111005756.png)

unique:users:20201106代表2020-11-06这天的独立访问用户的Bitmaps.

![image-20211111100601628](https://cos.duktig.cn/typora/202111111006438.png)

> 很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。
>
> 在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。

#### 2、getbit

获取Bitmaps中某个偏移量的值（偏移量不存在，也是返回0）

```sh
getbit<key><offset>
```

#### 3、bitcount

统计**字符串**被设置为1的bit数。

一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，start、end 是指bit组的字节的下标数，二者皆包含。

```sh
# 统计字符串从start字节到end字节比特值为1的数量
bitcount<key>[start end] 
```

#### 4、bitop

```sh
bitop and(or/not/xor) <destkey> [key…]
```

bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。

实例：

```sh
# 2020-11-04 日访问网站的userid=1,2,5,9。
setbit unique:users:20201104 1 1
setbit unique:users:20201104 2 1
setbit unique:users:20201104 5 1
setbit unique:users:20201104 9 1

#2020-11-03 日访问网站的userid=0,1,4,9。
setbit unique:users:20201103 0 1
setbit unique:users:20201103 1 1
setbit unique:users:20201103 4 1
setbit unique:users:20201103 9 1
```

计算出两天都访问过网站的用户数量:`bitop and unique:users:20201103 unique:users:20201104`

计算出任意一天都访问过网站的用户数量（例如月活跃就是类似这种） ， 可以使用or求并集：`bitop or unique:users:20201103 unique:users:20201104`

### Bitmaps与set对比

假设网站有1亿用户， 每天独立访问的用户有5千万， 如果每天用集合类型和Bitmaps分别存储活跃用户可以得到表：

![image-20211111101502391](https://cos.duktig.cn/typora/202111111015921.png)

![image-20211111101542846](https://cos.duktig.cn/typora/202111111015690.png)

很明显， 这种情况下使用Bitmaps能节省很多的内存空间， 尤其是随着时间推移节省的内存还是非常可观的。

但Bitmaps并不是万金油， 假如该网站每天的独立访问用户很少， 例如只有10万（大量的僵尸用户） ， 那么两者的对比如下表所示， 很显然， 这时候使用Bitmaps就不太合适了， 因为基本上大部分位都是0。

![image-20211111101625476](https://cos.duktig.cn/typora/202111111016925.png)

## HyperLogLog

### 基本介绍

在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站PV（PageView页面访问量）,可以使用Redis的incr、incrby轻松实现。

但像UV（UniqueVisitor，独立访客）、独立IP数、搜索记录数等需要去重和计数的问题如何解决？这种 **求集合中不重复元素个数** 的问题称为 **基数** 问题。解决基数问题有很多种方案：

（1）数据存储在MySQL表中，使用distinct count计算不重复个数

（2）使用Redis提供的hash、set、bitmaps等数据结构来处理

以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。

**能否能够降低一定的精度来平衡存储空间？**Redis推出了HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，**HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。**

在 Redis 里面，**每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数**。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

**什么是基数?**

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

### 相关命令

#### 1、pfadd

添加指定元素到 HyperLogLog 中

```sh
pfadd <key>< element> [element ...]   
```

将所有元素添加到指定HyperLogLog数据结构中。如果执行命令后HLL估计的近似基数发生变化，则返回1，否则返回0。

![image-20211111102059470](https://cos.duktig.cn/typora/202111111021159.png)

#### 2、pfcount

**计算HLL的近似基数**，可以计算多个HLL，比如用HLL存储每天的UV，计算一周的UV可以使用7天的UV合并计算即可

```sh
pfcount<key> [key ...] 
```

#### 3、pfmerge

将一个或多个HLL合并后的结果存储在另一个HLL中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得

```sh
pfmerge<destkey><sourcekey> [sourcekey ...] 
```

## Geospatial

### 基本介绍

Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。

### 相关命令

| 命令                                                         | 描述                                                      |
| :----------------------------------------------------------- | :-------------------------------------------------------- |
| [Redis GEOHASH 命令](https://www.redis.net.cn/order/3687.html) | 返回一个或多个位置元素的 Geohash 表示                     |
| [Redis GEOPOS 命令](https://www.redis.net.cn/order/3688.html) | 从key里返回所有给定位置元素的位置（经度和纬度）           |
| [Redis GEODIST 命令](https://www.redis.net.cn/order/3686.html) | 返回两个给定位置之间的距离                                |
| [Redis GEORADIUS 命令](https://www.redis.net.cn/order/3689.html) | 以给定的经纬度为中心， 找出某一半径内的元素               |
| [Redis GEOADD 命令](https://www.redis.net.cn/order/3685.html) | 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中 |
| [Redis GEORADIUSBYMEMBER 命令](https://www.redis.net.cn/order/3690.html) | 找出位于指定范围内的元素，中心点是由给定的位置元素决定    |

## 参看：

- [【尚硅谷】Redis 6 入门到精通 超详细 教程](https://www.bilibili.com/video/BV1Rv41177Af?p=4)
- [Redis底层数据结构之List](https://www.cnblogs.com/reecelin/p/13358432.html)
- [Redis底层数据结构之hash](https://www.cnblogs.com/reecelin/p/13362104.html)
- [Redis底层数据结构之string](https://www.cnblogs.com/reecelin/p/13352694.html)
- [Redis底层数据结构之set](https://www.cnblogs.com/reecelin/p/13364089.html)
- [Redis底层数据结构之 zset](https://www.cnblogs.com/reecelin/p/13368374.html)

