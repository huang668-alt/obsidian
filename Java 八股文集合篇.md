### 1. Collection接口和Collections类的区别是什么？

- **`Collection` 是一个接口（Interface）：**  它是 Java 集合框架的**根接口**之一。
    
    - 它定义了单列集合（如列表、集合）的通用行为规范（例如 `add()`, `remove()`, `size()`, `iterator()` 等）。
        
    - 它不能被直接实例化，常见的子接口包括 `List`、`Set` 和 `Queue`。
        
- **`Collections` 是一个工具类（Utility Class）：**
    
    - 它属于 `java.util` 包，是一个**特殊的类**。
        
    - 它的构造方法被 `private` 私有化了，无法被继承或实例化。
        
    - 它内部包含的全部是 **`static` 静态方法**，专门用来操作或返回 `Collection` 集合。


### 2. HashMap的扩容机制是怎样的？
1. **触发条件：** 
	 - 当 `HashMap` 中存储的键值对数量（`size`）超过了一个**阈值**时，就会触发扩容。这个阈值通常是 `当前容量 * 加载因子（loadFactor，默认0.75）`。
2. **扩容过程：**
	- **创建新数组：** 创建一个**新的、更大的数组**，通常是原数组容量的**两倍**。
	- **重新计算哈希与迁移 (Rehashing)：** 遍历旧数组中的**每一个**键值对，**重新计算**它们在新数组中的位置（因为数组长度变了，哈希映射的索引也会变），然后将它们放入新数组的对应位置。这个过程很重要，因为同一个元素在新旧数组中的桶（bucket）索引很可能不同。
3. **目的：**
	- **减少哈希冲突：** 通过扩大数组容量，使得键值对能更均匀地分布，从而减少哈希冲突，保持 `HashMap` 的查找、插入等操作的效率（接近 O(1)）。
	- **保持性能：** 如果不扩容，当元素越来越多时，链表（或红黑树）会越来越长，性能会下降。
### 3. ArrayList扩容机制是什么？

**当 ArrayList 实际元素个数（size）超过当前数组容量（elementData.length）时，会触发扩容，新容量 = 旧容量 + 旧容量/2（即 1.5 倍）**，并把旧数组元素复制到新数组中。

ArrayList 的扩容机制是：

- 默认初始容量 10（第一次 add 时分配）
- 每次容量不够用时，新容量 = 旧容量 + 旧容量/2（即 oldCapacity * 1.5）
- 使用 Arrays.copyOf() 进行数组复制（底层是 System.arraycopy()）
- 容量增长呈 1.5 倍关系（不是 2 倍，与 HashMap 不同）

**额外容易问到的点**

- **为什么是 1.5 倍而不是 2 倍？**  
    1.5 倍在空间利用率和扩容频率之间取得了较好的平衡，2 倍虽然扩容次数更少，但浪费空间更多。
- **是否线程安全？**  
    ArrayList 扩容过程**非线程安全**（多线程同时 add 可能导致数据丢失或异常）。
- **性能开销最大的是哪一步？**  
    **数组复制**（System.arraycopy），属于 O(n) 操作，所以尽量在创建时指定大概容量：`new ArrayList<>(1000)`

### 4. HashMap的put方法的流程是怎样的？

1. **计算哈希值和索引：**

	- 对传入的 `key` 计算哈希值，并通过扰动函数和位运算确定其在底层数组中的**存储位置**（索引）。

2. **判断位置是否为空：**

	- 如果该索引位置**为空**，直接创建新结点并放入。
	- 如果该索引位置**不为空**（发生**哈希冲突**），则需要进一步处理。

3. **处理哈希冲突：**

	- **Key已存在：** 如果链表/红黑树中已存在相同的 `key`，则放弃添加元素，更新其对应的 `value`，并返回旧值。
	- **Key不存在：** 在链表末尾添加新结点。
	- **链表转红黑树：** 如果链表长度达到阈值（默认为 8），会尝试将链表转换为红黑树，以提高查找效率。

4. **扩容检查：**

- 元素添加成功后，会检查 `HashMap` 的当前元素数量 `size` 是否超过了扩容阈值 `threshold`。如果超过，则进行扩容操作，将底层数组的容量翻倍，并重新计算所有元素的索引位置。

### 5. ⭐⭐️️⭐️️HashMap 和 Hashtable 的区别

- **线程是否安全：**`HashMap` 是非线程安全的，`Hashtable` 是线程安全的,因为 `Hashtable` 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
- **效率：** 因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰，不要在代码中使用它；
- **对 Null key 和 Null value 的支持：**`HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
- **初始容量大小和每次扩充容量大小的不同：**
	 - 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 `2n+1` 。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 `2` 倍。
	 - 创建时如果给定了容量初始值，那么 `Hashtable` 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
- **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间（后文中我会结合源码对这一过程进行分析）。`Hashtable` 没有这样的机制。
- **哈希函数的实现**：`HashMap` 对哈希值进行了高位和低位的混合扰动处理以减少冲突，而 `Hashtable` 直接使用键的 `hashCode()` 值。
### 6. HashMap 和 HashSet 区别

`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

| `HashMap`                      | `HashSet`                                                                        |
| ------------------------------ | -------------------------------------------------------------------------------- |
| 实现了 `Map` 接口                   | 实现 `Set` 接口                                                                      |
| 存储键值对                          | 仅存储对象                                                                            |
| 调用 `put()`向 map 中添加元素          | 调用 `add()`方法向 `Set`中添加元素                                                         |
| `HashMap`使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode`值，对于两个对象来说 `hashcode`可能相同，所以`equals()`方法用来判断对象的相等性 |

### 7. ⭐️HashMap 和 TreeMap 区别

 1. 底层数据结构不同

- **HashMap：** 基于 **哈希表**（数组 + 链表 + 红黑树）。
    
- **TreeMap：** 基于 **红黑树**（自平衡的二叉查找树）。
    

 2. 元素是否有序

- **HashMap：** **无序**。元素的存储位置由哈希值决定，遍历出来的顺序与插入顺序无关。
    
- **TreeMap：** **有序**。默认按 Key 的升序（自然顺序）排列，也可以传入自定义的 `Comparator` 比较器来决定顺序。
    

 3. 性能与时间复杂度

- **HashMap：** 性能更高。理想情况下，增删改查的时间复杂度为 **$O(1)$**。
    
- **TreeMap：** 性能稍逊。由于每次增删都需要调整红黑树以保持平衡，时间复杂度为 **$O(\log n)$**。
    

 4. 对 Null 值的支持

- **HashMap：** **支持 `null`**。允许有一个 Key 为 `null`，多个 Value 为 `null`。
    
- **TreeMap：** **Key 绝不能为 `null`**（会抛出 `NullPointerException`），因为红黑树需要调用 Key 的 `compareTo()` 方法进行比较排序；但 Value 可以为 `null`。
    

 5. 查找接口的功能扩展

- **HashMap：** 只有最基础的键值对存取功能。
    
- **TreeMap：** 实现了 `NavigableMap` 接口，提供了很多**范围查找**和**边界查找**的绝活（例如查找比某个 Key 大的最小 Key：`higherKey()`，或者获取子集合：`subMap()`）。
    

 6. 核心应用场景

- **HashMap：** 绝大多数场景下的首选，适用于**单纯的高效数据缓存、快速查找**。
    
- **TreeMap：** 适用于**需要对 Key 进行排序**，或者需要做**区间/范围统计**的特殊业务场景（如商品价格区间分类、按时间戳排序的日志）。

### 8. HashSet 如何检查重复?

既然元素变成了 `HashMap` 的 Key，那么 `HashSet` 检查重复的逻辑就完全等同于 `HashMap` 元素的去重逻辑（**先文斗，后武斗**）：

1. **第一步：计算并比对 `hashCode`** 当加入新元素时，首先计算该元素的 `hashCode` 值，并通过扰动函数找到对应的桶位。
    
    - 如果该桶位没有其他元素，说明绝不重复，直接存入。
        
    - 如果该桶位有元素（发生哈希冲突），则进入第二步。
        
2. **第二步：通过 `equals` 做精准判断** 如果两个元素的 `hashCode` 相同，`HashSet` 会调用 `equals()` 方法去逐个字段比对它们的内容。
    
    - 如果 `equals()` 返回 `true`：说明这两个元素**完全重复**，`HashMap` 会用新的 Value 覆盖旧的 Value（对 `HashSet` 来说就是拒绝加入，保持原样），并返回 `false`。
        
    - 如果 `equals()` 返回 `false`：说明只是碰巧哈希冲突了，内容并不重复，会以链表或红黑树的形式在尾部插入该元素。

### 9. ⭐⭐️️⭐️HashMap 的底层实现

#### JDK1.8 之前

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。HashMap 通过 key 的 `hashcode` 经过扰动函数处理过后得到 hash 值，然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

`HashMap` 中的扰动函数（`hash` 方法）是用来优化哈希值的分布。通过对原始的 `hashCode()` 进行额外处理，扰动函数可以减小由于糟糕的 `hashCode()` 实现导致的碰撞，从而提高数据的分布均匀性。

**JDK 1.8 HashMap 的 hash 方法源码:**

JDK 1.8 的 hash 方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

``` Java
static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

对比一下 JDK1.7 的 HashMap 的 hash 方法源码.

``` java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。

所谓 **“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263293-9ebb70db-8dbe-45ef-b985-e8f02091a182.png)

#### JDK1.8 之后

相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树。

这样做的目的是减少搜索时间：链表的查询效率为 O(n)（n 是链表的长度），红黑树是一种自平衡二叉搜索树，其查询效率为 O(log n)。当链表较短时，O(n) 和 O(log n) 的性能差异不明显。但当链表变长时，查询性能会显著下降。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263299-d504026f-66de-40cf-99eb-09680f4515aa.png)

**为什么优先扩容而非直接转为红黑树？**

数组扩容能减少哈希冲突的发生概率（即将元素重新分散到新的、更大的数组中），这在多数情况下比直接转换为红黑树更高效。

红黑树需要保持自平衡，维护成本较高。并且，过早引入红黑树反而会增加复杂度。

**为什么选择阈值 8 和 64？**

1. 泊松分布表明，链表长度达到 8 的概率极低（小于千万分之一）。在绝大多数情况下，链表长度都不会超过 8。阈值设置为 8，可以保证性能和空间效率的平衡。
2. 数组长度阈值 64 同样是经过实践验证的经验值。在小数组中扩容成本低，优先扩容可以避免过早引入红黑树。数组大小达到 64 时，冲突概率较高，此时红黑树的性能优势开始显现。

TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

我们来结合源码分析一下 `HashMap` 链表到红黑树的转换。

**1、** `**putVal**` **方法中执行链表转红黑树的判断逻辑。**

链表的长度大于 8 的时候，就执行 `treeifyBin` （转换红黑树）的逻辑。

```Java
// 遍历链表
for (int binCount = 0; ; ++binCount) {
    // 遍历到链表最后一个节点
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        // 如果链表元素个数大于TREEIFY_THRESHOLD（8）
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            // 红黑树转换（并不会直接转换成红黑树）
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

**2、**`treeifyBin` **方法中判断是否真的转换为红黑树。**

```Java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 判断当前数组的长度是否小于 64
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // 如果当前数组的长度小于 64，那么会选择先进行数组扩容
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 否则才将列表转换为红黑树

        TreeNode<K,V> hd = null, tl = null;
        do {
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
            hd.treeify(tab);
    }
}
```

将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树。

### 10. ⭐️HashMap 的长度为什么是 2 的幂次方

HashMap 容量设计为 2 的幂次方，主要为了能用高效的位与运算 & (capacity-1) 替代取模运算，同时在扩容翻倍时能通过位判断快速、均匀地重新散列元素，显著提升性能。

### 11. ⭐️HashMap 多线程操作导致死循环问题

 1. 明确前提：发生在 JDK 1.7（头插法）

首先向面试官划定范围：死循环问题**只发生在 JDK 1.7 及以前的版本**。

- **根本诱因：** JDK 1.7 在扩容迁移元素时使用的是**头插法**。
    
- **副作用：** 头插法会导致新链表**反转**旧链表元素的顺序（例如原本是 `A -> B`，迁移后变成 `B -> A`）。

 2. 核心成因：多线程并发扩容

当两个或多个线程同时对 `HashMap` 进行扩容（`resize`）时，死循环就可能发生：

1. **线程 1 记录指针：** 线程 1 准备迁移元素，刚记录好当前节点 `e = A` 和下一个节点 `next = B`，此时它的时间片用完，被挂起。
    
2. **线程 2 后来居上：** 线程 2 获得时间片并顺利完成了整个扩容。因为是头插法，堆内存中的链表结构被反转成了 **`B.next = A`**。
    
3. **线程 1 被唤醒复活：** 线程 1 恢复运行，它脑子里的记忆依然是 `e = A`，`next = B`。但在第一轮和第二轮循环迁移后，由于堆内存的物理结构已经被线程 2 改变，最终会导致 **`A.next = B` 且 `B.next = A`**。


> **结果：** 环形链表正式形成。后续只要有读操作（如 `get()`）在这个桶位查找一个不存在的 Key，就会陷入死循环，导致 **CPU 飙升到 100%**。

 3. JDK 1.8 是如何解决的？

JDK 1.8 彻底解决了死循环问题，主要做了两点重构：

- **改用尾插法：** 扩容迁移时保持元素原本的先后顺序，不再反转。
    
- **高低位链表：** 通过 `(e.hash & oldCap) == 0` 将原链表拆分为高位和低位两组，直接整体平移到新数组。


由于多线程下顺序不再颠倒，**环形链表彻底成为历史**。

 4. 终极避坑（面试加分反问）

**不要等面试官追问，主动说出这句话顶格加分：**

> “虽然 JDK 1.8 解决了死循环问题，但 `HashMap` 在多线程下依然是不安全的，仍然会出现 **数据覆盖丢失** 和 **size计算不准** 的问题。所以，并发场景下请务必使用 **`ConcurrentHashMap`**。”

### 头插法

 **什么是头插法**？

头插法就是：**新元素不排队，直接插到链表的最前面，成为新的头节点。**

 **它的致命问题**

因为总是插到最前面，扩容时会导致**链表顺序彻底反转**（比如原本是 `A -> B`，扩容后变成 `B -> A`）。

 **怎么导致死循环的**？

1. **多线程并发扩容**时，线程 1 刚记下 `A -> B` 的结构就被挂起了。
    
2. 线程 2 后来居上完成了扩容，用头插法把结构反转成了 `B -> A`。
    
3. 线程 1 醒来后，按照错误的记忆继续用头插法迁移，结果把 `A` 的下一个节点指向了 `B`，而内存里 `B` 已经指向了 `A`。

**最终结果：** `A` 指向 `B`，`B` 又指向 `A`，形成**环形链表**。后续 `get()` 查找时就会陷入死循环，CPU 瞬间飙到 100%。

### 12. ⭐️HashMap 为什么线程不安全？

`HashMap` 不是线程安全的。在多线程环境下对 `HashMap` 进行并发写操作，可能会导致两种主要问题：

1. **数据丢失**：并发 `put` 操作可能导致一个线程的写入被另一个线程覆盖。
2. **无限循环**：在 JDK 7 及以前的版本中，并发扩容时，由于头插法可能导致链表形成环，从而在 `get` 操作时引发无限循环，CPU 飙升至 100%。

数据丢失这个在 JDK1.7 和 JDK 1.8 中都存在，这里以 JDK 1.8 为例进行介绍。

JDK 1.8 后，在 `HashMap` 中，多个键值对可能会被分配到同一个桶（bucket），并以链表或红黑树的形式存储。多个线程对 `HashMap` 的 `put` 操作会导致线程不安全，具体来说会有数据覆盖的风险。

举个例子：

- 两个线程 1,2 同时进行 put 操作，并且发生了哈希冲突（hash 函数计算出的插入下标是相同的）。
- 不同的线程可能在不同的时间片获得 CPU 执行的机会，当前线程 1 执行完哈希冲突判断后，由于时间片耗尽挂起。线程 2 先完成了插入操作。
- 随后，线程 1 获得时间片，由于之前已经进行过 hash 碰撞的判断，所有此时会直接进行插入，这就导致线程 2 插入的数据被线程 1 覆盖了。

还有一种情况是这两个线程同时 `put` 操作导致 `size` 的值不正确，进而导致数据覆盖的问题：

1. 线程 1 执行 `if(++size > threshold)` 判断时，假设获得 `size` 的值为 10，由于时间片耗尽挂起。
2. 线程 2 也执行 `if(++size > threshold)` 判断，获得 `size` 的值也为 10，并将元素插入到该桶位中，并将 `size` 的值更新为 11。
3. 随后，线程 1 获得时间片，它也将元素放入桶位中，并将 size 的值更新为 11。
4. 线程 1、2 都执行了一次 `put` 操作，但是 `size` 的值只增加了 1，也就导致实际上只有一个元素被添加到了 `HashMap` 中。

### 13. HashMap 常见的遍历方式?

[[HashMap 的 7 种遍历方式与性能分析！]]

### 14. ⭐️ConcurrentHashMap 和 Hashtable 的区别

`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：** 
	- JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，在 JDK1.8 中采用的数据结构跟 `HashMap` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
- **实现线程安全的方式（重要）：**
	- 在 JDK1.7 的时候，`ConcurrentHashMap` 对整个桶数组进行了分割分段(`Segment`，分段锁)，每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
	- 到了 JDK1.8 的时候，`ConcurrentHashMap` 已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 `synchronized` 锁做了很多优化） 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本；
	- `Hashtable`**(同一把锁)** :使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

下面，我们再来看看两者底层数据结构的对比图。

**Hashtable** :

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263303-abc2ca56-80d2-4047-aa89-9def16ad29a6.png)


**JDK1.7 的 ConcurrentHashMap**：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263770-055e8ddf-3c17-4673-8415-8733fe380bd1.png)

`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成。

`Segment` 数组中的每个元素包含一个 `HashEntry` 数组，每个 `HashEntry` 数组属于链表结构。

**JDK1.8 的 ConcurrentHashMap**：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263860-8893578e-a845-4a33-9455-3c809cabbb23.png)

JDK1.8 的 `ConcurrentHashMap` 不再是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用 `TreeNode`。当冲突链表达到一定长度时，链表会转换成红黑树。

`TreeNode`是存储红黑树节点，被`TreeBin`包装。`TreeBin`通过`root`属性维护红黑树的根结点，因为红黑树在旋转的时候，根结点可能会被它原来的子节点替换掉，在这个时间点，如果有其他线程要写这棵红黑树就会发生线程不安全问题，所以在 `ConcurrentHashMap` 中`TreeBin`通过`waiter`属性维护当前使用这棵红黑树的线程，来防止其他线程的进入。

``` Java
static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
...
}
```

### 15. ⭐️ConcurrentHashMap 线程安全的具体实现方式/底层具体实现

#### JDK1.8 之前

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263860-f899193b-257e-449a-b3f5-1f1c12b5377a.png)

首先将数据分为一段一段（这个“段”就是 `Segment`）的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

`**ConcurrentHashMap**` **是由** `**Segment**` **数组结构和** `**HashEntry**` **数组结构组成**。

`Segment` 继承了 `ReentrantLock`,所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。

```
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组，`Segment` 的个数一旦**初始化就不能改变**。 `Segment` 数组的大小默认是 16，也就是说默认可以同时支持 16 个线程并发写。

`Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构，一个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素，每个 `Segment` 守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁。也就是说，对同一 `Segment` 的并发写入会被阻塞，不同 `Segment` 的写入是可以并发执行的。

#### JDK1.8 之后

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118263868-1912b0b7-db0e-4d5e-838e-619930f8deb7.png)

Java 8 几乎完全重写了 `ConcurrentHashMap`，代码量从原来 Java 7 中的 1000 多行，变成了现在的 6000 多行。

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 `Node + CAS + synchronized` 来保证并发安全。数据结构跟 `HashMap` 1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）。

Java 8 中，锁粒度更细，`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升。

### 16. ⭐️JDK 1.7 和 JDK 1.8 的 ConcurrentHashMap 实现有什么不同？

- **线程安全实现方式**：JDK 1.7 采用 `Segment` 分段锁来保证安全， `Segment` 是继承自 `ReentrantLock`。JDK1.8 放弃了 `Segment` 分段锁的设计，采用 `Node + CAS + synchronized` 保证线程安全，锁粒度更细，`synchronized` 只锁定当前链表或红黑二叉树的首节点。
- **Hash 碰撞解决方法** : JDK 1.7 采用拉链法，JDK1.8 采用拉链法结合红黑树（链表长度超过一定阈值时，将链表转换为红黑树）。
- **并发度**：JDK 1.7 最大并发度是 Segment 的个数，默认是 16。JDK 1.8 最大并发度是 Node 数组的大小，并发度更大。

### 17. ConcurrentHashMap 为什么 key 和 value 不能为 null？

`ConcurrentHashMap` 不允许 key 和 value 为 null，是为了在并发环境下消除 get(key) == null 的歧义性：

返回 null 只能表示“key 不存在”，而不能是“key 存在但 value 是 null”。

这样用户代码更安全，避免了在并发场景下因“先 get 再 containsKey”导致的竞态条件。

这是 Doug Lea 的有意设计取舍，牺牲了 null 的支持，换来了更清晰、更可靠的并发语义。

### 18. ⭐️ConcurrentHashMap 能保证复合操作的原子性吗？

 1. 什么是复合操作？（翻车现场）

所谓的复合操作，就是“先读后写”（Read-Modify-Write）。比如：**“如果不存在则放入”**，或者“先获取旧值，在此基础上加 1，再放回去”。

来看这段经典的错误代码：


``` Java
// ❌ 线程不安全！
if (!map.containsKey("key")) { // 操作 1：读
    map.put("key", "value");   // 操作 2：写
}
```

**为什么不安全？** 虽然 `containsKey` 和 `put` 各自都是线程安全的，但在多线程环境下，可能线程 A 刚执行完 `containsKey` 发现不存在，还没来得及 `put`，线程 B 就抢先一步 `put` 成功了。随后线程 A 继续执行 `put`，就会把线程 B 的数据**覆盖掉**。

 2. 解决方案一：使用 CHM 提供的原子复合方法

为了解决这个问题，`ConcurrentHashMap` 官方在内部提供了一系列**原子性的复合方法**。在开发中，我们应该优先使用这些方法来代替手写的 `if-else`：

- **`putIfAbsent(K key, V value)`**：如果 Key 不存在则放入，存在则不放入。完美替代上面的错误示范。
    
- **`computeIfPresent(K key, BiFunction remappingFunction)`**：Key 存在时，计算并更新其 Value。
    
- **`computeIfAbsent(K key, Function mappingFunction)`**：Key 不存在时，计算并插入其 Value（常用于实现本地缓存）。
    
- **`replace(K key, V oldValue, V newValue)`**：CAS 机制，只有当当前值等于 `oldValue` 时，才替换为 `newValue`。
    

 3. 解决方案二：显式加锁（当业务极其复杂时）

如果你的复合操作非常复杂，包含了多个不同的 Map 或者其他业务逻辑，光靠上面单个 Key 的原子方法搞不定，就必须**显式加锁**：


```
// 使用 synchronized 或 ReentrantLock 将整个复合操作打包成一个原子整体
synchronized(map) {
    if (!map.containsKey("key")) {
        map.put("key", "value");
    }
}
```

> **⚠️ 注意：** 一旦加上 `synchronized(map)`，`ConcurrentHashMap` 内部的分段锁/CAS 优势就会失效，退化成类似 `Hashtable` 的全局大锁，性能会大幅下降。因此非必要不推荐。


### 19. ⭐️说说 List, Set, Queue, Map 四者的区别？

 1. List (对付顺序的好帮手)

- **面试加分点：** 熟练掌握 **`ArrayList`**（底层是动态数组，查询快 $O(1)$，增删慢）和 **`LinkedList`**（底层是双向链表，查询慢，增删快 $O(1)$）的底层区别与选型场景。
    

 2. Set (注重独一无二的性质)

- **面试加分点：** 必须要能说出 **`HashSet` 检查重复的底层原理**（前面我们聊过，它其实就是套壳了 `HashMap` 的 Key，利用 `hashCode()` 和 `equals()` 来去重）。
    

 3. Queue (实现排队功能的叫号机)

- **面试加分点：** 除了普通的 FIFO（先进先出），面试中常考它的变向延伸：
    
    - **`Deque`（双端队列）：** 头部尾部都能进出，常用来代替 `Stack` 实现栈的操作（Java 官方推荐用 `ArrayDeque` 代替老旧的 `Stack` 类）。
        
    - **`PriorityQueue`（优先级队列）：** 它不是按时间排队，而是按元素的“大小/优先级”排队（底层是二叉堆），常用于 Top-K 算法问题。
        

 4. Map (用 key 来搜索的专家)

- **面试加分点：** 它是整个集合框架面试的重灾区。不仅要懂 **`HashMap`** 的拉链法、红黑树转化和多线程安全问题，还要知道如果需要 Key 有序就选 **`TreeMap`**，如果需要保证插入顺序就选 **`LinkedHashMap`**。

### 20. 如何选用集合?

我们主要根据集合的特点来选择合适的集合。比如：

- 我们需要根据键值获取到元素值时就选用 `Map` 接口下的集合，需要排序时选择 `TreeMap`,不需要排序时就选择 `HashMap`,需要保证线程安全就选用 `ConcurrentHashMap`。

- 我们只需要存放元素值时，就选择实现`Collection` 接口的集合，需要保证元素唯一时选择实现 `Set` 接口的集合比如 `TreeSet` 或 `HashSet`，不需要就选择实现 `List` 接口的比如 `ArrayList` 或 `LinkedList`，然后再根据实现这些接口的集合的特点来选用。

### 21. 为什么要使用集合？

当我们需要存储一组类型相同的数据时，数组是最常用且最基本的容器之一。但是，使用数组存储对象存在一些不足之处，因为在实际开发中，存储的数据类型多种多样且数量不确定。这时，Java 集合就派上用场了。与数组相比，Java 集合提供了更灵活、更有效的方法来存储多个数据对象。Java 集合框架中的各种集合类和接口可以存储不同类型和数量的对象，同时还具有多样化的操作方式。相较于数组，Java 集合的优势在于它们的大小可变、支持泛型、具有内建算法等。总的来说，Java 集合提高了数据的存储和处理灵活性，可以更好地适应现代软件开发中多样化的数据需求，并支持高质量的代码编写。

### 22. ⭐️ArrayList 和 Array（数组）的区别？

`ArrayList` 内部基于动态数组实现，比 `Array`（静态数组） 使用起来更加灵活：

- `ArrayList`会根据实际存储的元素动态地扩容或缩容，而 `Array` 被创建之后就不能改变它的长度了。
- `ArrayList` 允许你使用泛型来确保类型安全，`Array` 则不可以。
- `ArrayList` 中只能存储对象。对于基本类型数据，需要使用其对应的包装类（如 Integer、Double 等）。`Array` 可以直接存储基本类型数据，也可以存储对象。
- `ArrayList` 支持插入、删除、遍历等常见操作，并且提供了丰富的 API 操作方法，比如 `add()`、`remove()`等。`Array` 只是一个固定长度的数组，只能按照下标访问其中的元素，不具备动态添加、删除元素的能力。
- `ArrayList`创建时不需要指定大小，而`Array`创建时必须指定大小。

### 23. ArrayList 和 Vector 的区别?（了解即可）

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]`存储，适用于频繁的查找工作，线程不安全 。

- `Vector` 是 `List` 的古老实现类，底层使用`Object[]` 存储，线程安全。

### 24. Vector 和 Stack 的区别?（了解即可）

- `Vector` 和 `Stack` 两者都是线程安全的，都是使用 `synchronized` 关键字进行同步处理。

- `Stack` 继承自 `Vector`，是一个后进先出的栈，而 `Vector` 是一个列表。

### 25. ArrayList 可以添加 null 值吗？

`ArrayList` 中可以存储任何类型的对象，包括 `null` 值。不过，不建议向`ArrayList` 中添加 `null` 值， `null` 值无意义，会让代码难以维护比如忘记做判空处理就会导致空指针异常。

### 26. ⭐️ArrayList 插入和删除元素的时间复杂度？

对于插入：

- 头部插入：由于需要将所有元素都依次向后移动一个位置，因此时间复杂度是 O(n)。
- 尾部插入：当 `ArrayList` 的容量未达到极限时，往列表末尾插入元素的时间复杂度是 O(1)，因为它只需要在数组末尾添加一个元素即可；当容量已达到极限并且需要扩容时，则需要执行一次 O(n) 的操作将原数组复制到新的更大的数组中，然后再执行 O(1) 的操作添加元素。
- 指定位置插入：需要将目标位置之后的所有元素都向后移动一个位置，然后再把新元素放入指定位置。这个过程需要移动平均 n/2 个元素，因此时间复杂度为 O(n)。

对于删除：

- 头部删除：由于需要将所有元素依次向前移动一个位置，因此时间复杂度是 O(n)。
- 尾部删除：当删除的元素位于列表末尾时，时间复杂度为 O(1)。
- 指定位置删除：需要将目标元素之后的所有元素向前移动一个位置以填补被删除的空白位置，因此需要移动平均 n/2 个元素，时间复杂度为 O(n)。

### 27. ⭐️LinkedList 插入和删除元素的时间复杂度？

- 头部插入/删除：只需要修改头结点的指针即可完成插入/删除操作，因此时间复杂度为 O(1)。
- 尾部插入/删除：只需要修改尾结点的指针即可完成插入/删除操作，因此时间复杂度为 O(1)。
- 指定位置插入/删除：需要先移动到指定位置，再修改指定节点的指针完成插入/删除，不过由于有头尾指针，可以从较近的指针出发，因此需要遍历平均 n/4 个元素，时间复杂度为 O(n)。

这里简单列举一个例子：假如我们要删除节点 9 的话，需要先遍历链表找到该节点。然后，再执行相应节点指针指向的更改。

具体的源码可以参考：[LinkedList 源码分析](https://javaguide.cn/java/collection/linkedlist-source-code.html) 。

![](https://cdn.nlark.com/yuque/0/2025/jpeg/58546395/1767118229825-5febadf3-dc04-4864-a120-993c9a10411d.jpg.jpeg)

LinkedList 为什么不能实现 RandomAccess 接口？

`RandomAccess` 是一个标记接口，用来表明实现该接口的类支持随机访问（即可以通过索引快速访问元素）。由于 `LinkedList` 底层数据结构是链表，内存地址不连续，只能通过指针来定位，不支持随机快速访问，所以不能实现 `RandomAccess` 接口。

### 28. ⭐️⭐⭐ArrayList 与 LinkedList 区别?

- **是否保证线程安全：**`ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
- **底层数据结构：**`ArrayList` 底层使用的是 `**Object**` **数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）
- **插入和删除是否受元素位置的影响：**

- `ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`），时间复杂度就为 O(n)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
- `LinkedList` 采用链表存储，所以在头尾插入或者删除元素不受元素位置的影响（`add(E e)`、`addFirst(E e)`、`addLast(E e)`、`removeFirst()`、 `removeLast()`），时间复杂度为 O(1)，如果是要在指定位置 `i` 插入和删除元素的话（`add(int index, E element)`，`remove(Object o)`,`remove(int index)`）， 时间复杂度为 O(n) ，因为需要先移动到指定位置再插入和删除。

- **是否支持快速随机访问：**`LinkedList` 不支持高效的随机元素访问，而 `ArrayList`（实现了 `RandomAccess` 接口） 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
- **内存空间占用：**`ArrayList` 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。

### 29. 双向链表和双向循环链表

**双向链表：** 包含两个指针，一个 prev 指向前一个节点，一个 next 指向后一个节点。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118229661-95e1dc5a-2b5b-4d4f-9d0e-6aa041df1351.png)

**双向循环链表：** 最后一个节点的 next 指向 head，而 head 的 prev 指向最后一个节点，构成一个环。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118229662-1116dd54-faa9-46d4-95ab-a58f3925b2cd.png)

### 30. RandomAccess 接口

它用于标记某个 List 实现类支持**快速随机访问**（fast random access），即通过索引（如 list.get(i)）访问元素的时间复杂度通常为常量时间 O(1)。

这个接口的目的是允许通用算法（如 Collections.binarySearch() 或其他遍历操作）根据列表类型优化性能：

- 如果列表实现了 RandomAccess（如 ArrayList），算法优先使用索引访问（for 循环更快）。
- 如果没有实现（如 LinkedList），算法优先使用迭代器（Iterator）遍历，以避免低效的随机访问（LinkedList 的 get(i) 是 O(n)）。

### 31. ⭐️集合中的 fail-fast 和 fail-safe 是什么？

`fail-fast`（快速失败）和 `fail-safe`（安全失败）是Java集合框架在处理并发修改问题时，两种截然不同的设计哲学和容错策略。

快速失败的思想即针对可能发生的异常进行提前表明故障并停止运行，通过尽早的发现和停止错误，降低故障系统级联的风险。

在`java.util`包下的大部分集合（如 `ArrayList`, `HashMap`）是不支持线程安全的，为了能够提前发现并发操作导致线程安全风险，提出通过维护一个`modCount`记录修改的次数，迭代期间通过比对预期修改次数`expectedModCount`和`modCount`是否一致来判断是否存在并发操作，从而实现快速失败，由此保证在避免在异常时执行非必要的复杂代码。

**ArrayList (fail-fast) 示例：**

``` Java
// 使用线程不安全的 ArrayList，它是一种 fail-fast 集合
      List<Integer> list = new ArrayList<>();
      CountDownLatch latch = new CountDownLatch(2);

      for (int i = 0; i < 5; i++) {
          list.add(i);
      }
      System.out.println("Initial list: " + list);

      Thread t1 = new Thread(() -> {
          try {
              for (Integer i : list) {
                  System.out.println("Iterator Thread (t1) sees: " + i);
                  Thread.sleep(100);
              }
          } catch (ConcurrentModificationException e) {
              System.err.println("!!! Iterator Thread (t1) caught ConcurrentModificationException as expected.");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              latch.countDown();
          }
      });

      Thread t2 = new Thread(() -> {
          try {
              Thread.sleep(50);
              System.out.println("-> Modifier Thread (t2) is removing element 1...");
              list.remove(Integer.valueOf(1));
              System.out.println("-> Modifier Thread (t2) finished removal.");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              latch.countDown();
          }
      });

      t1.start();
      t2.start();
      latch.await();

      System.out.println("Final list state: " + list);
```

输出：

```
Initial list: [0, 1, 2, 3, 4]
Iterator Thread (t1) sees: 0
-> Modifier Thread (t2) is removing element 1...
-> Modifier Thread (t2) finished removal.
!!! Iterator Thread (t1) caught ConcurrentModificationException as expected.
Final list state: [0, 2, 3, 4]
```

程序在线程 t2 修改列表后，线程 t1 的下一次迭代操作立刻就抛出了 `ConcurrentModificationException`。这是因为 ArrayList 的迭代器在每次 `next()` 调用时，都会检查 `modCount` 是否被改变。一旦发现集合在迭代器不知情的情况下被修改，它会立即“快速失败”，以防止在不一致的数据上继续操作导致不可预期的后果。

对此我们也给出`for`循环底层迭代器获取下一个元素时的`next`方法，可以看到其内部的`checkForComodification`具有针对修改次数比对的逻辑：

```
public E next() {
 			//检查是否存在并发修改
            checkForComodification();
            //......
            //返回下一个元素
            return (E) elementData[lastRet = i];
        }

final void checkForComodification() {
		//当前循环遍历次数和预期修改次数不一致时，就会抛出ConcurrentModificationException
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

而`fail-safe`也就是安全失败的含义，它旨在即使面对意外情况也能恢复并继续运行，这使得它特别适用于不确定或者不稳定的环境：

Fail-safe systems take a different approach, aiming to recover and continue even in the face of unexpected conditions. This makes them particularly suited for uncertain or volatile environments.

该思想常运用于并发容器，最经典的实现就是`CopyOnWriteArrayList`的实现，通过写时复制（Copy-On-Write）的思想保证在进行修改操作时复制出一份快照，基于这份快照完成添加或者删除操作后，将`CopyOnWriteArrayList`底层的数组引用指向这个新的数组空间，由此避免迭代时被并发修改所干扰所导致并发操作安全问题，当然这种做法也存在缺点，即进行遍历操作时无法获得实时结果：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118229907-d7ce8fa1-5436-459d-95c7-31bb16097958.png)

对应我们也给出`CopyOnWriteArrayList`实现`fail-safe`的核心代码，可以看到它的实现就是通过`getArray`获取数组引用然后通过`Arrays.copyOf`得到一个数组的快照，基于这个快照完成添加操作后，修改底层`array`变量指向的引用地址由此完成写时复制：

```
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
        	//获取原有数组
            Object[] elements = getArray();
            int len = elements.length;
            //基于原有数组复制出一份内存快照
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            //进行添加操作
            newElements[len] = e;
            //array指向新的数组
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

### 32. Comparable 和 Comparator 的区别

`Comparable` 接口和 `Comparator` 接口都是 Java 中用于排序的接口，它们在实现类对象之间比较大小、排序等方面发挥了重要作用：

- `Comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序
- `Comparator`接口实际上是出自 `java.util`包 它有一个`compare(Object obj1, Object obj2)`方法用来排序

一般我们需要对一个集合使用自定义排序时，我们就要重写`compareTo()`方法或`compare()`方法，当我们需要对某一个集合实现两种排序方式，比如一个 `song` 对象中的歌名和歌手名分别采用一种排序方法的话，我们可以重写`compareTo()`方法和使用自制的`Comparator`方法或者以两个 `Comparator` 来实现歌名排序和歌星名排序，第二种代表我们只能使用两个参数版的 `Collections.sort()`.

#### Comparator 定制排序

```
ArrayList<Integer> arrayList = new ArrayList<Integer>();
arrayList.add(-1);
arrayList.add(3);
arrayList.add(3);
arrayList.add(-5);
arrayList.add(7);
arrayList.add(4);
arrayList.add(-9);
arrayList.add(-7);
System.out.println("原始数组:");
System.out.println(arrayList);
// void reverse(List list)：反转
Collections.reverse(arrayList);
System.out.println("Collections.reverse(arrayList):");
System.out.println(arrayList);

// void sort(List list),按自然排序的升序排序
Collections.sort(arrayList);
System.out.println("Collections.sort(arrayList):");
System.out.println(arrayList);
// 定制排序的用法
Collections.sort(arrayList, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});
System.out.println("定制排序后：");
System.out.println(arrayList);
```

Output:

```
原始数组:
[-1, 3, 3, -5, 7, 4, -9, -7]
Collections.reverse(arrayList):
[-7, -9, 4, 7, -5, 3, 3, -1]
Collections.sort(arrayList):
[-9, -7, -5, -1, 3, 3, 4, 7]
定制排序后：
[7, 4, 3, 3, -1, -5, -7, -9]
```

#### 重写 compareTo 方法实现按年龄来排序

```
// person对象没有实现Comparable接口，所以必须实现，这样才不会出错，才可以使treemap中的数据按顺序排列
// 前面一个例子的String类已经默认实现了Comparable接口，详细可以查看String类的API文档，另外其他
// 像Integer类等都已经实现了Comparable接口，所以不需要另外实现了
public  class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     * T重写compareTo方法实现按年龄来排序
     */
    @Override
    public int compareTo(Person o) {
        if (this.age > o.getAge()) {
            return 1;
        }
        if (this.age < o.getAge()) {
            return -1;
        }
        return 0;
    }
}
```

```
public static void main(String[] args) {
        TreeMap<Person, String> pdata = new TreeMap<Person, String>();
        pdata.put(new Person("张三", 30), "zhangsan");
        pdata.put(new Person("李四", 20), "lisi");
        pdata.put(new Person("王五", 10), "wangwu");
        pdata.put(new Person("小红", 5), "xiaohong");
        // 得到key的值的同时得到key所对应的值
        Set<Person> keys = pdata.keySet();
        for (Person key : keys) {
            System.out.println(key.getAge() + "-" + key.getName());
        }
    }
```

### 33. 无序性和不可重复性的含义是什么

- 无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的。
- 不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法。

### 34. 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的。
- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。`HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）。`LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。
- 底层数据结构不同又导致这三者的应用场景不同。`HashSet` 用于不需要保证元素插入和取出顺序的场景，`LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景，`TreeSet` 用于支持对元素自定义排序规则的场景。

### 35. Queue 与 Deque 的区别

`Queue` 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循 **先进先出（FIFO）** 规则。

`Queue` 扩展了 `Collection` 的接口，根据 **因为容量问题而导致操作失败后处理方式的不同** 可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值。

|   |   |   |
|---|---|---|
|`Queue`<br><br>接口|抛出异常|返回特殊值|
|插入队尾|add(E e)|offer(E e)|
|删除队首|remove()|poll()|
|查询队首元素|element()|peek()|

`Deque` 是双端队列，在队列的两端均可以插入或删除元素。

`Deque` 扩展了 `Queue` 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：

|   |   |   |
|---|---|---|
|`Deque`<br><br>接口|抛出异常|返回特殊值|
|插入队首|addFirst(E e)|offerFirst(E e)|
|插入队尾|addLast(E e)|offerLast(E e)|
|删除队首|removeFirst()|pollFirst()|
|删除队尾|removeLast()|pollLast()|
|查询队首元素|getFirst()|peekFirst()|
|查询队尾元素|getLast()|peekLast()|

事实上，`Deque` 还提供有 `push()` 和 `pop()` 等其他方法，可用于模拟栈。

### 36. ArrayDeque 与 LinkedList 的区别

`ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口，两者都具有队列的功能，但两者有什么区别呢？

- `ArrayDeque` 是基于可变长的数组和双指针来实现，而 `LinkedList` 则通过链表来实现。
- `ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持。
- `ArrayDeque` 是在 JDK1.6 才被引入的，而`LinkedList` 早在 JDK1.2 时就已经存在。
- `ArrayDeque` 插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。

从性能的角度上，选用 `ArrayDeque` 来实现队列要比 `LinkedList` 更好。此外，`ArrayDeque` 也可以用于实现栈。

### 37. 说一说 PriorityQueue

`PriorityQueue` 是在 JDK1.5 中被引入的, 其与 `Queue` 的区别在于元素出队顺序是与优先级相关的，即总是优先级最高的元素先出队。

这里列举其相关的一些要点：

- `PriorityQueue` 利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
- `PriorityQueue` 通过堆元素的上浮和下沉，实现了在 O(logn) 的时间复杂度内插入元素和删除堆顶元素。
- `PriorityQueue` 是非线程安全的，且不支持存储 `NULL` 和 `non-comparable` 的对象。
- `PriorityQueue` 默认是小顶堆，但可以接收一个 `Comparator` 作为构造参数，从而来自定义元素优先级的先后。

`PriorityQueue` 在面试中可能更多的会出现在手撕算法的时候，典型例题包括堆排序、求第 K 大的数、带权图的遍历等，所以需要会熟练使用才行。

### 38. 什么是 BlockingQueue？

`BlockingQueue` （阻塞队列）是一个接口，继承自 `Queue`。`BlockingQueue`阻塞的原因是其支持当队列没有元素时一直阻塞，直到有元素；还支持如果队列已满，一直等到队列可以放入新元素时再放入。

```
public interface BlockingQueue<E> extends Queue<E> {
  // ...
}
```

`BlockingQueue` 常用于生产者-消费者模型中，生产者线程会向队列中添加数据，而消费者线程会从队列中取出数据进行处理。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118229888-40507ea2-3db0-4cd8-89a8-9da10624c206.png)

### 39. BlockingQueue 的实现类有哪些？

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1767118230126-a56f1e1e-4705-4cd8-a5e5-4aae890f2252.png)

Java 中常用的阻塞队列实现类有以下几种：

1. `ArrayBlockingQueue`：使用数组实现的有界阻塞队列。在创建时需要指定容量大小，并支持公平和非公平两种方式的锁访问机制。
2. `LinkedBlockingQueue`：使用单向链表实现的可选有界阻塞队列。在创建时可以指定容量大小，如果不指定则默认为`Integer.MAX_VALUE`。和`ArrayBlockingQueue`不同的是， 它仅支持非公平的锁访问机制。
3. `PriorityBlockingQueue`：支持优先级排序的无界阻塞队列。元素必须实现`Comparable`接口或者在构造函数中传入`Comparator`对象，并且不能插入 null 元素。
4. `SynchronousQueue`：同步队列，是一种不存储元素的阻塞队列。每个插入操作都必须等待对应的删除操作，反之删除操作也必须等待插入操作。因此，`SynchronousQueue`通常用于线程之间的直接传递数据。
5. `DelayQueue`：延迟队列，其中的元素只有到了其指定的延迟时间，才能够从队列中出队。
6. ……

### 40. ⭐️ArrayBlockingQueue 和 LinkedBlockingQueue 有什么区别？

`ArrayBlockingQueue` 和 `LinkedBlockingQueue` 是 Java 并发包中常用的两种阻塞队列实现，它们都是线程安全的。不过，不过它们之间也存在下面这些区别：

- 底层实现：`ArrayBlockingQueue` 基于数组实现，而 `LinkedBlockingQueue` 基于链表实现。
- 是否有界：`ArrayBlockingQueue` 是有界队列，必须在创建时指定容量大小。`LinkedBlockingQueue` 创建时可以不指定容量大小，默认是`Integer.MAX_VALUE`，也就是无界的。但也可以指定队列大小，从而成为有界的。
- 锁是否分离： `ArrayBlockingQueue`中的锁是没有分离的，即生产和消费用的是同一个锁；`LinkedBlockingQueue`中的锁是分离的，即生产用的是`putLock`，消费是`takeLock`，这样可以防止生产者和消费者线程之间的锁争夺。
- 内存占用：`ArrayBlockingQueue` 需要提前分配数组内存，而 `LinkedBlockingQueue` 则是动态分配链表节点内存。这意味着，`ArrayBlockingQueue` 在创建时就会占用一定的内存空间，且往往申请的内存比实际所用的内存更大，而`LinkedBlockingQueue` 则是根据元素的增加而逐渐占用内存空间。