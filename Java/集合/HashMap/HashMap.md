# HashMap

$HashMap$ 主要用来存放键值对，它基于哈希表的 $Map$ 接口实现，是常用的 $Java$ 集合之一，是非线程安全的。 

`HashMap` 可以存储 $null$ 的 $key$ 和 $value$ ，但 $null$ 作为键只能有一个， $null$ 作为值可以有多个。

$JDK1.8$ 之前 $HashMap$ 由数组+链表组成的，数组是 $HashMap$ 的主体，链表则主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。 $JDK1.8$ 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于或等于阈值（默认为 $8$ ）（将链表转换成红黑树之前会进行判断，如果当前数组的长度小于 $64$ ，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

`HashMap` 默认的初始化大小为 $16$ 。之后每次扩充，容量变为原来的 $2$ 倍。并且， `HashMap` 总是使用 $2$ 的幂作为哈希表的大小。

## 数据结构分析

### $JDK1.8$ 之前

$JDK1.8$ 之前 $HashMap$ 底层是**数组和链表**结合在一起使用也就是**链表散列**。

$HashMap$ 通过 $key$ 的 $hashCode$ 经过扰动函数处理过后得到 $hash$ 值，然后通过 `(n-1) & hash` 判断当前元素存放的位置（这里的 $n$ 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 $hash$ 值以及 $key$ 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

所谓扰动函数指的就是 $HashMap$ 的 $hash$ 方法。使用 $hash$ 方法也就是扰动函数是为了防止一些实现比较差的 $hashCode()$ 方法，换句话说使用扰动函数之后可以减少碰撞。

$JDK1.8$ 的 $hash$ 方法相比于 $JDK1.7$ $hash$ 方法更加简化，但是原理不变。

```java
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

对比一下 $JDK1.7$ 的 $HashMap$ 的 $hash$ 方法源码：

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

```

相比于 $JDK1.8$ 的 $hash$ 方法， $JDK1.7$ 的 $hash$ 方法的性能会稍微差一点点，因为毕竟扰动了 $4$ 次。

所谓“拉链法”就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突（计算出的哈希值相同），则将冲突的值加到链表中即可。即遍历链表比对 $key$ 来找到目标值。

### $JDK1.8$ 之后

相比于之前的版本， $JDK1.8$ 以后在解决哈希冲突时有了较大的变化。

当链表长度大于阈值（默认为 $8$ ）时，会首先调用 `treeifyBin()` 方法。这个方法会根据 $HashMap$ 数组来决定是否转换为红黑树。只有当数组长度大于或者等于 $64$ 的情况下，才会执行转换红黑树操作，以减少搜索时间（因为链表过长，遍历链表查找时可能比红黑树平均查找时间 $O(logn$)$ 花费时间多）。否则，就是只是执行 `resize()` 方法对数组扩容。

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于等于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 当桶(bucket)上的结点数小于等于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小容量
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table;
    // 一个包含了映射中所有键值对的集合视图
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;
    // 阈值(容量*负载因子) 当实际大小超过阈值时，会进行扩容
    int threshold;
    // 负载因子
    final float loadFactor;
}

```

