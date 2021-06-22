# Java相关面试题

## 1. HashMap相关面试题

[HashMap底层实现原理（上）](https://zhuanlan.zhihu.com/p/28501879) （通俗易懂）

### 1.1 HashMap为什么使用红黑树？红黑树的临界点为什么是8？

在JDK1.8中加入了红黑树是为了防止**哈希表碰撞攻击，当链表链长度为8时，及时转成红黑树，提高map的效率。**

参考：

- [为什么hashMap引入了红黑树而不是其他结构](https://www.cnblogs.com/wq-9/articles/14202773.html)
- [为什么HashMap链表存储结构转换为红黑树的长度临界点是8?](https://www.sunjianbo.com/why-threshold-is-8/) 
- [面试官：Hashmap链表长度为8时转换成红黑树，你知道为什么是8吗](https://blog.csdn.net/kyle_wu_/article/details/113578055)



