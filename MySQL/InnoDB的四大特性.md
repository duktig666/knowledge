### InnoDB四大特性

#### 1、插入缓冲（insert buffer）

> 插入缓冲（Insert Buffer/Change Buffer）：提升插入性能，change buffering是insert buffer的加强，insert buffer只针对insert有效，change buffering对insert、delete、update(delete+insert)、purge都有效

使用插入缓冲的条件：

- 非聚集索引（辅助索引）
- 非唯一索引

Change buffer是作为buffer pool中的一部分存在。`Innodb_change_buffering`参数缓存所对应的操作(update会被认为是delete+insert)：

- `all`: 默认值，缓存insert, delete, purges操作
- `none`: 不缓存
- `inserts`: 缓存insert操作
- `deletes`: 缓存delete操作
- `changes`: 缓存insert和delete操作
- `purges`: 缓存后台执行的物理删除操作

`innodb_change_buffer_max_size`参数：控制使用的大小，默认25%，最大可设置50%，如果mysql实例中有大量的修改操作，可考虑增大该参数；

对满足插入缓存条件的插入，每一次的插入不是写到索引页中，而是：

1. 会先判断插入的非聚集索引页是否在缓冲池中，如果在直接插入；
2. 如果不在，则先放到insert buffer中，再按照一定的频率进行合并操作，再写会磁盘；
3. 通常可以将多个插入合并到一个操作中，目的是为了减少随机IO带来的性能损耗；

这样通常能将多个插入合并到一个操作中，目的还是为了**减少随机IO带来性能损耗**。

**上面提过在一定频率下进行合并，那所谓的频率是什么条件**？

1. 辅助索引页被读取到缓冲池中。正常的select先检查Insert Buffer是否有该非聚集索引页存在，若有则合并插入。
2. 辅助索引页没有可用空间。空间小于1/32页的大小，则会强制合并操作。
3. Master Thread 每秒和每10秒的合并操作。

insert buffer的数据结构是一颗B+树；

- 全局只有一颗insert buffer B+树，负责对所有表的辅助索引进行insert buffer；
- 这颗B+树放在共享表空间中，试图通过独立表空间ibd文件恢复表中数据时，往往会导致check table失败，因为表中的辅助索引中的数据可能还在insert buffer中，也就是共享表空间中，所以ibd文件恢复后，还需要repair table操作来重建表上所有的辅助索引；

#### 2、二次写（double write）

1. doublewrite缓存位于系统表空间的存储区域，用来缓存innodb的数据页从innodb buffer pool中flush之后并写入到数据文件之前；
2. 当操作系统或数据库进程在数据页写入磁盘的过程中崩溃，可以在doublewrite缓存中找到数据页的备份，用来执行crash恢复；
3. 数据页写入到doublewrite缓存的动作所需要的io消耗要小于写入到数据文件的消耗，因为此写入操作会以一次大的连续块的方式写入；

![二次写过程](https://gitee.com/koala010/typora/raw/master/img/20210627104446.png)

从上图可知：

1. 内存中doublewrite buffer大小2M；物理磁盘上共享表空间中连续的128个页，也就是2个区（extent）大小同样为2M
2. 对缓冲池脏页进行刷新时，不是直接写磁盘。流程：
   1. 通过memcpy()函数将脏页先复制到内存中的doublewrite buffer
   2. 通过doublewrite分两次，每次1M顺序的写入共享表空间的物理磁盘上。这个过程中，doublewrite页是连续的，因此这个过程是顺序的，所以开销并不大；
   3. 完成doublewrite页的写入后，再将doublewrite buffer中的页写入各个表空间文件中，此时写入是离散的，可能会较慢；
   4. 如果操作系统在第三步的过程中发生了崩溃，在恢复过程中，可以从共享表空间中的doublewrite中找到该页的一个副本，将其复制到表空间文件中，再应用重做日志；

#### 3、自适应hash索引（ahi）

> innodb存储引擎会监控对表上二级索引的查找，如果发现某二级索引被频繁访问，此索引成为热数据，建立hash索引以提升查询速度，此建立是自动建立哈希索引，故称为自适应哈希索引（adaptive hash index）。

该属性通过`innodb_adapitve_hash_index`开启，也可以通过`—skip-innodb_adaptive_hash_index`参数关闭

注意事项：

- 自适应哈希索引会占用innodb buffer pool
- 只适合搜索等值（=）的查询，对于范围查找等操作，是不能使用的
- 极端情况下，自适应hash索引才有比较大的意义，可以降低逻辑读

#### 4、预读(read ahead)

> **extent 定义**：表空间（tablespace 中的一组 page）

InnoDB使用两种预读算法来提高I/O性能：线性预读（linear read-ahead）和随机预读（randomread-ahead）。

- 线性预读：以extent为单位，将下一个extent提前读取到buffer pool中；
- 随机预读：以extent中的page为单位，将当前extent中的剩余的page提前读取到buffer pool中；

线性预读一个重要参数：innodb_read_ahead_threshold，控制什么时间（访问extent中多少页的阈值）触发预读；

- 默认：56，范围：0～64，值越高，访问模式检查越严格；
- 没有该变量之前，当访问到extent最后一个page时，innodb会决定是否将下一个extent放入到buffer pool中；

随机预读说明：

- 当同一个extent的一些page在buffer pool中发现时，innodb会将extent中剩余page一并读取到buffer pool中；
- 随机预读给innodb code带来一些不必要的复杂性，性能上也不稳定，在5.5版本已经废弃，如果启用，需要修改变量：innodb_random_read_ahead为ON；

参考：[InnoDB四大特性](https://zhuanlan.zhihu.com/p/109528131)