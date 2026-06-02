
## 定义与体系结构

- 在 `Java` 中，集合（`Collection`） 是一种用于存储、管理多个数据元素的数据结构，它可以动态地存储一组相关的对象，解决了数组长度固定、类型单一的局限性。`Java` 集合框架（`Java Collection Framework, JCF`）提供了一套完整的集合接口、实现类和工具类，使得数据操作更加便捷和高效。
- 我们可以将集合大体分为两类，分别是单列集合（`Collection`）和双列集合（`Map`）。

## Collection

`Collection`代表单列集合，每个元素（数据）只包含一个值。  
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1765279311402-5a9c3a6f-97f8-4424-ab49-469f38e6df84.png)

注意：虚边框的是接口，实边框的是实现类。

**特点**：

- `List`系列集合：添加的元素是有序、可重复、有索引。

- `ArrayList、LinkedList`：有序、可重复、有索引。

- `Set`系列集合：添加的元素的无序、不重复、无索引。

- `HashSet`：无序、不重复、无索引。
- `LinkedHashSet`：有序、不重复、无索引。
- `TreeSet`：按图大小默认升序排序、不重复、无索引。

### 💯常用功能

`Collection`是单列集合的祖宗，它规定的方法（功能）是全部单列集合都会继承的。

| 方法名                                 | 说明               |
| ----------------------------------- | ---------------- |
| public boolean add(E e)             | 把给定的对象添加到当前集合中   |
| public void clear( )                | 清空集合中所有的元素       |
| public boolean remove(E e)          | 把给定的对象在当前集合中删除   |
| public boolean contains(Object obj) | 判断当前集合中是否包含给定的对象 |
| public boolean isEmpty()            | 判断当前集合是否为空       |
| public int size()                   | 返回集合中元素的个数       |
| public Object[ ] toArray()          | 把集合中的元素，存储到数组中   |

### 💯遍历方式

1. **迭代器遍历**

迭代器是用来遍历集合的专用方式（数组没有迭代器），在`java`中迭代器的代表是`Iterator`。

- `Collection`集合获取迭代器的方法

|   |   |
|---|---|
|方法名称|说明|
|Iterator< E > iterator()|返回集合中的迭代器对象，该迭代器对象默认指向当前集合的第一个元素（即索引为0的元素 ）|

- Iterator迭代器中的常用方法

|   |   |
|---|---|
|方法名称|说明|
|boolean hasNext()|询问当前位置是否有元素存在，存在返回`true`<br><br>，不存在返回`false`|
|E next()|获取当前位置的元素，并同时将迭代器对象指向下一个元素处|

示例：

``` Java
public class Test {
    public static void main(String[] args) {
        Collection<String> list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        list.add("java");
        list.add("C++");

        Iterator<String> it = list.iterator();
        while (it.hasNext()){
            String ele = it.next();
            System.out.println(ele);
        }
    }
}
```

2. **增强**`**for**`**循环**

- 增强`for`可以用来遍历集合或数组。
- 增强`for`遍历集合，本质就是迭代器遍历集合的简化写法。

格式：

```
for (元素的数据类型 变量名 : 数组或者集合){

}
```

示例：

```
public class Test {
    public static void main(String[] args) {
        Collection<String> list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        list.add("java");
        list.add("C++");

        for(String s:list){
            System.out.println(s);
        }

        String [] arr = {"hello","world","java","C++"};
        for(String s:arr){
            System.out.println(s);
        }
    }
}
```

3. `**Lambda**`**表达式**

- 得益于JDK 8开始的新技术`Lambda`表达式，提供了一种更简单、更直接的方式来遍历集合。

需要使用`Collection`的如下方法来完成：

|   |   |
|---|---|
|方法名称|说明|
|default void forEach(Consumer<? super T> action)|结合Lambda遍历集合|

示例：

```
public class Test {
    public static void main(String[] args) {
        Collection<String> list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        list.add("java");
        list.add("C++");

        list.forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });

        list.forEach(s -> System.out.println(s));
        list.forEach(System.out::println);
    }
}
```

4. **三种遍历的区别**

前置知识： 遍历集合的同时又存在增删集合元素的行为时可能出现业务异常，这种现象被称之为并发修改异常问题。

解决这一问题的方案：

- 如果集合支持索引，可以使用for循环遍历，每删除数据后做`i--;`或者可以倒着遍历。

```
for(int i=0; i<list.size(); i++){
    if(条件){
        list.remove(i);
        i--;
    }
}

for(int i=list.size()-1; i>=0; i--){
    if(条件){
        list.remove(i);
    }
}
```

- 可以使用迭代器遍历，并用迭代器提供的删除方法删除数据（例如：`it.remove();`）。

```
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String item = it.next();
    if(条件){
        it.remove();  
    }
}
```

注意：增强`for`循环/`Lambda`遍历均不能解决并发修改异常问题，因此它们只适合做数据的遍历，不适合同时做增删操作，这就是三种遍历的区别。

### 💯List集合

`List`系列集合特点：有序、可重复、有索引。

- `ArrayList`：有序、可重复、有索引。
- `LinkedList`：有序、可重复、有索引。

`ArrayList`与`LinkedList`特点相同，但底层实现不同，适合的场景不同。

1. `List`集合继承了`Collection`的功能，同时也拥有自己的特有功能，这些功能都与索引有关。

|   |   |
|---|---|
|方法名称|说明|
|void add(int index,E element)|在此集合中的指定位置插入指定的元素|
|E remove(int index)|删除指定索引处的元素，返回被删除的元素|
|E set(int index,E element)|修改指定索引处的元素，返回被修改的元素|
|E get(int index)|返回指定索引处的元素|

```
import java.util.ArrayList;
import java.util.List;

public class ListExample {
    public static void main(String[] args) {

        List<String> fruits = new ArrayList<>();

        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Cherry");

        fruits.add(1, "Orange"); 

        String fruitAtIndex2 = fruits.get(2);
        System.out.println("索引2的元素: " + fruitAtIndex2);

        fruits.set(0, "Mango");
        System.out.println("修改后的List: " + fruits);

        for (String fruit : fruits) {
            System.out.println(fruit);
        }

        fruits.remove(3);
        System.out.println("删除后的List: " + fruits);
    }
}
```

1. `ArrayList`底层是基于数组存储数据的。

特点：

- 查询速度快（注意：是根据索引查询数据块）：查询数据通过地址值和索引定位，查询任意数据耗时相同。
- 增删数据效率低：可能需要把后面很多的数据进行前移。

2. `LinkedList`底层是基于双链表存储数据的。

特点：

- 链表中的数据是一个一个独立的结点组成的，结点在内存中是不连续的，每个结点包含数据值和下一个结点的地址。
- 查询慢，无论查询哪个数据都要从头开始找。
- 链表增删相对快。
- `LinkedList`新增了很多首尾操作的特定方法：

|   |   |
|---|---|
|方法名称|说明|
|public void addFirst(E e)|在该列表开头插入指定的元素|
|public void addLast(E e)|将指定的元素追加到该列表的末尾|
|public E getFirst()|返回此列表中的第一个元素|
|public E getLast()|返回此列表中的最后一个元素|
|public E removeFirst()|从此列表中删除并返回第一个元素|
|public E removeLast()|从此列表中删除并返回最后一个元素|

`LinkedList`可以用来实现栈和队列。

（1）**实现栈（LIFO结构）**  
栈是一种后进先出（Last-In-First-Out）的数据结构，`LinkedList`可以通过以下方式模拟栈操作：

- 压栈（Push）：使用`addFirst()`方法在链表头部插入元素

```
LinkedList<Integer> stack = new LinkedList<>();
stack.addFirst(1);  
stack.addFirst(2);
```

- 弹栈（Pop）：使用`removeFirst()`方法移除并返回头部元素

```
int top = stack.removeFirst();
```

- 查看栈顶（Peek）：使用`getFirst()`方法
- 典型应用场景：函数调用栈、浏览器后退功能、表达式求值等

（2）**实现队列（FIFO结构）**  
队列是一种先进先出（First-In-First-Out）的数据结构，可以通过以下方式实现：

- 入队（Enqueue）：使用`addLast()`方法在尾部添加元素

```
LinkedList<String> queue = new LinkedList<>();
queue.addLast("A");  
queue.addLast("B");
```

- 出队（Dequeue）：使用`removeFirst()`方法从头部移除元素

```
String first = queue.removeFirst();
```

- 查看队首元素：使用`getFirst()`方法
- 典型应用场景：消息队列、多线程任务调度、BFS算法实现等

（3）**实现双端队列（Deque）**  
`LinkedList`本身就实现了`Deque`接口，因此可以直接作为双端队列使用：

```
Deque<Integer> deque = new LinkedList<>();
deque.addFirst(1);    
deque.addLast(2);     
int front = deque.removeFirst();  
int rear = deque.removeLast();
```

（4） **性能考虑**

- 相比`ArrayList`，`LinkedList`在头部插入/删除操作上有O(1)的时间复杂度
- 但随机访问需要O(n)时间，因为需要遍历链表
- 在内存使用上，每个元素需要额外的空间存储前后节点引用

（5） **线程安全注意事项**  
标准`LinkedList`实现不是线程安全的，多线程环境下建议使用：

```
List<String> syncList = Collections.synchronizedList(new LinkedList<>());
```

通过这些实现方式，`LinkedList`可以灵活地满足不同场景下栈和队列的需求，特别是当需要频繁在数据结构两端进行操作时，其性能优势尤为明显。

### 💯Set集合

`Set`集合特点：无序：添加数据的顺序和获取出的数据顺序不一致；不重复；无索引；

- HashSet：无序、不重复、无索引。（底层原理：数组+链表+红黑树）
- LinkedHashSet：有序、不重复、无索引。（底层原理：数组+双链表+红黑树）
- TreeSet：排序、不重复、无索引。（底层原理：红黑树）

注意：`Set`要用到的常用的方法，基本上就是`Collection`提供的，自己几乎没有额外新增一些功能。JDK8开始，当链表长度超过8，且数组长度>=64时，`HashSet`自动将链表转为红黑树。

1. `HashSet`集合元素的去重操作

- 去重原理 `HashSet`的去重机制主要通过以下方式实现：

- 当向`HashSet`添加元素时，会先计算元素的`hashCode`值
- 根据`hashCode`值确定元素的存储位置
- 如果该位置已有元素，则调用`equals()`方法比较
- 如果`equals()`返回`true`，则视为重复元素，不会被添加

- 如果希望`Set`集合认为2个内容一样的对象是重复的，必须重写对象的`hashCode()`和`equals()`方法。(自定义对象去重复)

```
class Student {
    private String id;
    private String name;
    
    @Override
    public int hashCode() {

        return Objects.hash(id, name);
    }

    @Override
    public boolean equals(Object obj) {

    }
}

Set<Student> studentSet = new HashSet<>();
studentSet.add(new Student("001", "王五"));
studentSet.add(new Student("001", "王五"));
```

1. `TreeSet`底层是基于红黑树（Red-Black Tree）实现的有序集合，它能够保证元素始终处于排序状态。红黑树是一种自平衡的二叉查找树，通过特定的着色规则和旋转操作来维护树的平衡，确保插入、删除和查找操作的时间复杂度都是O(log n)。

**注意**：

- 对于数值类型：如`Integer`、`Double`等，`TreeSet`默认按照数值本身的大小进行升序排序。例如：`[3, 1, 2]`会被排序为`[1, 2, 3]`。
- 对于字符串类型：默认按照字符串的首字符的Unicode编号升序排序。例如：`["banana", "apple", "cherry"]`会被排序为`["apple", "banana", "cherry"]`。
- 对于自定义类型：如`Student`对象，`TreeSet`默认是无法直接排序的，因为集合无法确定如何比较两个自定义对象的大小。

**解决自定义类型排序的两种方案**：

1. **实现**`**Comparable**`**接口**：

- 让自定义类实现`Comparable`接口，并重写`compareTo()`方法，定义对象的自然排序规则。
- 示例：

```
class Student implements Comparable<Student> {
    private String name;
    private int age;

    @Override
    public int compareTo(Student other) {
        return this.age - other.age; 
    }
}
```

`this`是比较者，`other`是被比较者。如果要左边大于右边，就返回正整数；如果要左边小于右边，就返回负整数；如果要两边相等，就返回0。

1. **使用**`**Comparator**`**比较器**：

- 在创建`TreeSet`时传入一个自定义的`Comparator`比较器对象，定义临时的排序规则。
- 示例：

```
TreeSet<Student> students = new TreeSet<>(new Comparator<Student>() {
    @Override
    public int compare(Student s1, Student s2) {
        return s1.getName().compareTo(s2.getName()); 
    }
});
```

## Map

`Map`代表双列集合，每个元素包含两个值（键值对）。

- `Map`集合也叫做“键值对集合”，格式：`key1=value1,key2=value2,key3=value3,...`
- `Map`集合的所有键是不允许重复的，但值可以重复，键和值是一一对应的，每一个键只能找到自己对应的值。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1765279311297-c1864a96-357d-4fff-980f-c90240d0b658.png)

注意：虚边框的是接口，实边框的是实现类。

**特点**：  
`Map`系列集合的特点都是由键决定的，值只是一个附属品，值是不做要求的。

- `HashMap`（键决定特点）：无序、不重复、无索引。（用的最多）
- `LinkedHashMap`（键决定特点）：有序、不重复、无索引。
- `TreeMap`（键决定特点）：按照大小默认升序排序、不重复、无索引。

### 💯常用方法

`Map`是双列集合的祖宗，它的功能是全部双列集合都可以继承来使用的。

|   |   |
|---|---|
|方法名称|说明|
|public V put(K key,v value)|添加元素|
|public int size()|获取集合的大小|
|public void clear()|清空集合|
|public boolean isEmpty()|判断集合是否为空，为空返回true，反之返回false|
|public V get(Object key)|根据键获取对应值|
|public V remove(Object key)|根据键删除整个元素|
|public boolean containsKey(Object key)|判断是否包含某个键|
|public boolean containsValue(Object value)|判断是否包含某个值|
|public Set< K> keySet()|获取全部键的集合|
|public Collection< V> values()|获取Map集合的全部值|

### 💯遍历方式

1. **键找值**

先获取`Map`集合全部的键，再通过遍历键来找值。

```
Set<String> Keys = map.keySet();

for(String key:keys){
    Integer value = map.get(key);
    System.out.println(key + "=" + value);
}
```

1. **键值对**

把`Map`集合转换成`Set`集合，里面的元素类型都是键值对类型（`Map.Entry<String,Integer>`）。

```
Set<Map.Entry<String,Integer>>entries = map.entrySet();

for(Map.Entry<String,Integer>entry : entries){
    String key = entry.getKey();
    Integer value = entry.getValue();
    System.out.println(key + "=" + value);
}
```

2. **Lambda**

需要运用`Map`的如下方法：

|   |   |
|---|---|
|方法名称|说明|
|default void forEach(BiConsumer<? super K , ? super V> action)|结合Labdma遍历Map集合|

```
map.forEach((k,v)->{
    System.out.println(key + "--->" + value);
});
```

### 💯实现类详解与底层原理

1. `HashMap`集合的底层原理深入解析

`HashMap`是Java集合框架中最常用的Map实现类，其底层采用哈希表（数组+链表/红黑树）结构实现。关键特性包括：

- 初始容量默认为16，负载因子0.75
- 当链表长度超过8且数组长度≥64时，链表会转为红黑树
- 通过`hash(key.hashCode())`计算索引位置

`HashSet`与`HashMap`的关系更为明确：

```
public class HashSet<E> 
implements Set<E> {
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();  
    }

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

典型应用场景：

- 快速查找（时间复杂度O(1)）
- 去重存储
- 缓存实现

1. `LinkedHashMap`的底层实现机制

`LinkedHashMap`继承自`HashMap`，在哈希表的基础上增加了双向链表维护插入顺序或访问顺序：

```
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;  
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

有序性实现原理：

- 默认按插入顺序排序（`accessOrder=false`）
- 设置`accessOrder=true`时按访问顺序排序（适合实现LRU缓存）
- 通过重写`newNode()`方法创建带链表指针的节点

`LinkedHashSet`的实现方式：

```
public LinkedHashSet() {
    super(16, 0.75f, true);  
}
```

1. `TreeMap`的红黑树实现详解

`TreeMap`基于红黑树（自平衡二叉查找树）实现，主要特点：

- 元素默认按key的自然顺序排序
- 可通过`Comparator`自定义排序规则
- 基本操作时间复杂度为`O(log n)`

红黑树五大特性：

1. 节点是红色或黑色
2. 根节点是黑色
3. 所有叶子节点(`NIL`)是黑色
4. 红色节点的子节点必须是黑色
5. 从任一节点到其每个叶子的路径包含相同数目的黑色节点

`TreeSet`的底层实现：

```
public TreeSet() {
    this(new TreeMap<E,Object>());  
}
```

排序示例：

```
TreeMap<String, Integer> map = new TreeMap<>((a,b) -> b.length()-a.length());
map.put("apple", 1);
map.put("banana", 2);
```

## Collections 工具类

`**Collections**` **工具类常用方法**:

- 排序
- 查找,替换操作
- 同步控制(不推荐，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合)

### 排序操作

```
void reverse(List list)//反转
void shuffle(List list)//随机排序
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)//交换两个索引位置的元素
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面
```

### 查找,替换操作

```
int binarySearch(List list, Object key)//对List进行二分查找，返回索引，注意List必须是有序的
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)
int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)
void fill(List list, Object obj)//用指定的元素代替指定list中的所有元素
int frequency(Collection c, Object o)//统计元素出现次数
int indexOfSubList(List list, List target)//统计target在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target)
boolean replaceAll(List list, Object oldVal, Object newVal)//用新元素替换旧元素
```

### 同步控制

`Collections` 提供了多个`synchronizedXxx()`方法·，该方法可以将指定集合包装成线程同步的集合，从而解决多线程并发访问集合时的线程安全问题。

我们知道 `HashSet`，`TreeSet`，`ArrayList`,`LinkedList`,`HashMap`,`TreeMap` 都是线程不安全的。`Collections` 提供了多个静态方法可以把他们包装成线程同步的集合。

**最好不要用下面这些方法，效率非常低，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合。**

方法如下：

```
synchronizedCollection(Collection<T>  c) //返回指定 collection 支持的同步（线程安全的）collection。
synchronizedList(List<T> list)//返回指定列表支持的同步（线程安全的）List。
synchronizedMap(Map<K,V> m) //返回由指定映射支持的同步（线程安全的）Map。
synchronizedSet(Set<T> s) //返回指定 set 支持的同步（线程安全的）set。
```