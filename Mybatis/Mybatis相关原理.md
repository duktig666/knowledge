## Mybatis的一级缓存和二级缓存

mybatis的的**一级缓存是SqlSession级别的缓存**，一级缓存**缓存的是对象**，当SqlSession提交、关闭以及其他的更新数据库的操作发生后，一级缓存就会清空。

二级缓存是SqlSessionFactory级别的缓存，同一个SqlSessionFactory产生的SqlSession都共享一个二级缓存，二级缓存中存储的是数据，当命中二级缓存时，通过存储的数据构造对象返回。

mybatis默认开启一级缓存，二级缓存需要手动开启。

查询数据的时候，查询的流程是**二级缓存>一级缓存>数据库**。



参看：

- [Mybatis的一级缓存和二级缓存的理解以及用法](https://www.cnblogs.com/hopeofthevillage/p/11427438.html)
- [mybatis一级缓存二级缓存 ](https://www.cnblogs.com/happyflyingpig/p/7739749.html)

