## HashSet的value值为什么是个Object对象呢？

HashSet 的 `add` 方法：

```java
public boolean add(E e){
    return map.put(e,PRESENT)==null;
}
```

PRESENT是什么？

```java
private static final Object PRESENT = new Object ()
```

**为什么这个要让Object作为Value存入呢？**

HashSet底层其实就是维护着 HashMap ，HashSet的 add 方法，其实是将值，作为 HashMap 的 key 存入。

而**如果HashSet的value 用null值的话，那么在删除操作的时候，可能remove方法返回的值就永远都是true了**。

即不存在时，也是null，移除时无法判断。

HashSet用对象类型作为value，来保证删除的key的value值都是不一样的，这样就可以保证上述remove方法返回的正确性。

