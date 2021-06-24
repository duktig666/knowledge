# Java相关面试题

## 1. HashMap相关面试题

[HashMap底层实现原理（上）](https://zhuanlan.zhihu.com/p/28501879) （通俗易懂）

[JavaGuide-HashMap(JDK1.8)源码+底层数据结构分析](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90) 

### 1.1 HashMap为什么使用红黑树？红黑树的临界点为什么是8？

在JDK1.8中加入了红黑树是为了防止**哈希表碰撞攻击，当链表链长度为8时，及时转成红黑树，提高map的效率。**

参考：

- [为什么hashMap引入了红黑树而不是其他结构](https://www.cnblogs.com/wq-9/articles/14202773.html)
- [为什么HashMap链表存储结构转换为红黑树的长度临界点是8?](https://www.sunjianbo.com/why-threshold-is-8/)  （简单总结，并且回答了为什么是8）
- [面试官：Hashmap链表长度为8时转换成红黑树，你知道为什么是8吗](https://blog.csdn.net/kyle_wu_/article/details/113578055)



### 1.2 一致性哈希

参考：

- [白话解析：一致性哈希算法 consistent hashing](https://www.zsythink.net/archives/1182) （通俗易懂，讲的太棒了）
- [一致性哈希算法（consistent hashing）](https://zhuanlan.zhihu.com/p/129049724) 





