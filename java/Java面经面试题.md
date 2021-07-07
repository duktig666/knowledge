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
- [源码分析---HashMap中链表和红黑树的转换阈值](https://blog.csdn.net/cg2258911936/article/details/103402684)

### 1.2 一致性哈希

参考：

- [白话解析：一致性哈希算法 consistent hashing](https://www.zsythink.net/archives/1182) （通俗易懂，讲的太棒了）
- [一致性哈希算法（consistent hashing）](https://zhuanlan.zhihu.com/p/129049724) 

### 1.3 HashCode 为什么使用 31 作为乘数？

// 获取 hashCode "abc".hashCode();

```java
public int hashCode() {
	int h = hash;
	if (h == 0 && value.length > 0) {
		char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
	}
	return h;
}
```

在获取 hashCode 的源码中可以看到，有一个固定值 31，在 for 循环每次执行时进行乘积计算，循环后的公式如下； `s[0]*31^(n-1) + s[1]*31^(n-2) + ... +s[n-1]` 那么这里为什么选择 31 作为乘积值呢？原因如下：

1. 31 是一个奇质数，如果选择偶数会导致乘积运算时数据溢出。
2. 在二进制中，2 个 5 次方是 32，那么也就是 `31 * i == (i << 5) - i`。这主要是说乘积运算可以使用位移提升性能，同时目前的 JVM 虚拟机 也会自动支持此类的优化。
3. 超过 5 千个单词计算 hashCode， 这个 hashCode 的运算使用 31、33、37、39 和 41 作为乘积，得到的碰撞 结果，31 被使用就很正常了。
   1. 31的碰撞概率很小，比较稳定，而且不超过int的取值范围
   2. hash的散列比较均匀

参看：[面经手册 · 第2篇《数据结构，HashCode为什么使用31作为乘数？》](https://bugstack.cn/interview/2020/08/04/%E9%9D%A2%E7%BB%8F%E6%89%8B%E5%86%8C-%E7%AC%AC2%E7%AF%87-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84-HashCode%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A831%E4%BD%9C%E4%B8%BA%E4%B9%98%E6%95%B0.html)

### 1.3





## 2. 多线程常见面试题

### 2.1 基础

参看：[Java多线程基础](https://duktig.cn/archives/31/)

### 2.2 synchroniced关键字

参看：[深入理解synchroniced关键字](https://duktig.cn/archives/42/)

### 2.3 volatile关键字

参看：[深入理解volatile关键字](https://duktig.cn/archives/46/)

### 2.4 ThreadLocal

参看：[万字详解ThreadLocal关键字](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/%E4%B8%87%E5%AD%97%E8%AF%A6%E8%A7%A3ThreadLocal%E5%85%B3%E9%94%AE%E5%AD%97)

