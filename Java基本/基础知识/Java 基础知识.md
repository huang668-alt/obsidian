## Java 语法基石
### - 基本语法

### - 流程控制

### - 数组
#### - 数组的核心特质
- **连续的内存空间**：数组在内存中申请的空间必须是死死挨在一起的。
- **相同的数据类型**：数组中的所有元素，其占用的字节大小必须完全一致。
- **长度一旦确定，不可更改**：系统在内存中划定这块区域后，它的大小就固定了。如果需要扩容，只能创建新数组并把数据复制过去。

### - 核心机制
[[Java八股文JVM篇]]

## 面向对象编程

## - 类与对象

## - 三大特性

## - 高级类特性
### - 抽象类（`abstract`）

#### - 1. 编译期与字节码层面的底层表现

当我们写下一个抽象类并编译它时，JVM 到底看到了什么？

假设我们有如下代码：

``` Java
public abstract class Animal {
    private String name;
    
    // 抽象类可以有构造方法
    public Animal(String name) {
        this.name = name;
    }

    // 抽象方法：没有方法体
    public abstract void makeSound();
}
```

##### 1. 编译后的标志位（ACC_ABSTRACT）

使用 `javap -v Animal.class` 反编译字节码，你会看到该类的类修饰符（flags）中包含一个核心标志：

> **`flags: (0x0421) ACC_PUBLIC, ACC_SUPER, ACC_ABSTRACT`**

- **`ACC_ABSTRACT`**：这就是 JVM 识别抽象类的底层印记。当你在代码中尝试 `new Animal()` 时，Java 编译器（javac）和 JVM 的类加载验证器一旦检测到这个标志位，就会直接抛出编译错误或 `InstantiationError` 异常，**从底层逻辑上直接锁死实例化路径**。
##### 2. 为什么抽象类一定要有构造方法？

很多人会疑惑：“既然抽象类不能被 `new`，那它里面为什么允许、甚至必须有构造方法（默认为无参构造）？”

我们来看看子类 `Dog` 的字节码：
``` Java
public class Dog extends Animal {
    public Dog(String name) {
        super(name); // 显式或隐式调用父类构造
    }
    @Override
    public void makeSound() {
        System.out.println("汪汪");
    }
}
```
反编译 `Dog` 的构造方法字节码，你会看到第一行指令必然是：

> **`invokespecial #1 // Method Animal."<init>":(Ljava/lang/String;)V`**

`invokespecial` 指令用于调用实例初始化方法（即构造方法 `<init>`）。 这揭示了底层的堆内存初始化顺序：**在堆内存中创建子类对象时，必须先调用父类（抽象类）的构造方法，在子类对象的内存空间中初始化父类定义的成员变量（如 `name`），然后才执行子类本身的构造逻辑。**
#### - 2. JVM 运行期的内存模型与虚方法表（VMT）

要理解抽象方法是如何在底层被执行的，必须引入 JVM 的虚方法表（Virtual Method Table，简称 vtable）机制。
- **虚方法表（vtable）是类共享的，而不是对象共享的。**

##### 1. 对象的内存布局

当我们在堆内存中创建一个子类对象 `Animal dog = new Dog("旺财");` 时，它的内存布局主要由三部分组成：

1. **对象头（Object Header）**：包含 Mark Word（锁状态、GC 分代年龄）和 **Klass Pointer（类型指针）**。Klass Pointer 指向方法区中 `Dog` 类的元数据。
    
2. **实例数据（Instance Data）**：**包含从抽象父类 `Animal` 继承来的 `name` 属性**，以及 `Dog` 类自身的属性。
    
3. **对齐填充（Padding）**。

##### 2. 动态绑定与虚方法表

在 JVM 中，绝大多数普通方法和抽象方法都属于**虚方法**。JVM 在类加载的“链接”阶段，会为每个类在方法区创建一个**虚方法表（VMT）**。这个表本质上是一个指针数组，里面记录了当前类所有可调用的方法在内存中的绝对入口地址。

- 抽象类 `Animal` 的虚方法表中，`makeSound()` 对应的值是一个**空指针（或者指向一个引发异常的特殊陷阱地址）**，因为它是抽象的，没有代码实现。
    
- 当子类 `Dog` 加载时，它重写了 `makeSound()`。JVM 会将 `Dog` 类的虚方法表中相同索引位置的指针，**替换（覆写）为 `Dog.makeSound()` 在内存中的真实操作码首地址**。

当执行代码：
``` Java
dog.makeSound();
```

底层字节码是：

> **`invokevirtual #4 // Method Animal.makeSound:()V`**

**`invokevirtual` 的执行全流程（最底层逻辑）：**

1. JVM 顺着栈帧中的 `dog` 引用，找到堆内存中的 `Dog` 对象实体。
    
2. 通过对象头里的 **Klass Pointer**，定位到方法区中 `Dog` 类的元数据及它的**虚方法表**。
    
3. 根据方法签名，在虚方法表中通过固定的偏移量（Offset）直接拿到 `Dog.makeSound()` 的实际内存地址。
    
4. 直接跳转到该地址执行字节码。

这就是为什么通过父类抽象引用能够精准调用到子类具体实现的底层底层真相：**在运行期，JVM 根本不看声明的类型（Animal），它通过对象头的指针直接去查实际对象（Dog）的虚方法表。**

### - 接口（`interface`）

#### - 1. 编译后的标志位（ACC_INTERFACE & ACC_ABSTRACT）

当我们写下一个接口并用 `javac` 编译时：
``` java
public interface Flyable {
    void fly();
}
```

使用 `javap -v Flyable.class` 查看其字节码，你会看到类修饰符（flags）呈现出这样的底层印记：

> **`flags: (0x0601) ACC_PUBLIC, ACC_INTERFACE, ACC_ABSTRACT`**

- **`ACC_INTERFACE`**：明确告诉 JVM，这**不是**一个普通的类拓扑结构，而是一个特殊的接口类型。
    
- **`ACC_ABSTRACT`**：是的，**在 JVM 眼里，所有接口天然都是抽象的**，哪怕你没写 `abstract` 关键字。
#### - 2. 字段的底层固化

在接口中，你声明的任何变量（如 `int SPEED = 100;`），编译器都会在编译时毫不留情地强行加上三大补丁：`public static final`。 在字节码中，它不属于任何对象个体，而是作为**类变量**直接锁死在方法区的常量池中。
接口绝对没有能力在堆内存的对象实体中开辟空间来维护“ **动态状态** ”（接口是一张纯粹的“行为纸条”，它没有自己的实体，所以它没办法在每个独立的对象身上“挖一个专属的格子”来存只属于这个对象的数据。）

##### 1. 什么是“动态状态”？

在面向对象里，“状态”就是变量（属性），而“动态状态”指的是**每个对象的属性值可以各自独立、随时改变**。 比如：

- 迪迦奥特曼的生命值（`hp`）是 100，当前能量（`energy`）是 80；
    
- 赛罗奥特曼的生命值（`hp`）是 200，当前能量（`energy`）是 50。

这种“同一个变量，在不同对象身上值不一样，且能各自修改”的特征，就叫维护动态状态。



#### - 3. 为什么接口不能用 vtable 解决？（槽位冲突大灾难）

前面我们讲过，普通类和抽象类多态的底层基石是 **虚方法表（vtable）**。 当调用父类方法时，JVM 为什么能“盲数”偏移量（Offset）直接拿到地址？因为 Java 是**单继承**的！子类重写父类方法时，方法在 `vtable` 里的**索引槽位（Index）是死死固定且完全一致的**。

但是，**接口支持多实现！** 这直接粉碎了 `vtable` 的固定偏移量神话。

#### - 4. JVM 的解法：引入接口方法表（itable）

为了解决多实现的槽位冲突，JVM 在类的方法区元数据中，除了 `vtable` 之外，又专门开辟了一块全新的区域——**接口方法表（Interface Method Table，简称 itable）**。

`itable` 的结构由两部分组成：

1. **偏移量部分（Offset Table）**：记录当前类实现了哪些接口，以及这些接口的方法列表在内存里的起点。
    
2. **方法实现部分（Method Table）**：真正记录重写后的方法执行代码的绝对内存地址。

``` java
Flyable f = new Bird();
f.fly();
```
其底层的字节码指令不再是 `invokevirtual`，而变成了：

> **`invokeinterface #2, 1 // InterfaceMethod Flyable.fly:()V`**

**`invokeinterface` 的底层寻址全流程：**

1. JVM 顺着栈里的引用 `f` 找到堆里的 `Bird` 对象，通过对象头里的 **Klass Pointer** 杀进方法区 `Bird` 类的元数据。
    
2. 进入 **`itable`** 区域，开始**动态扫描**偏移量表，人肉搜索：“让我看看这个类实现的接口里，哪个是 `Flyable` 接口？”
    
3. 找到了 `Flyable` 接口的记录，由此拿到了它对应的方法表的起始基地址。
    
4. 根据 `fly()` 方法在该接口内部的相对偏移量，最终定位到 `Bird.fly()` 的真实操作码内存地址。
    
5. 跳转执行。

> **⚠️ 性能代价**： 因为 `invokevirtual`（抽象类/普通类调用）是直接通过固定索引**一步到位**的；而 `invokeinterface` 必须先在 `itable` 里**线性扫描**匹配接口，再二段跳寻址，所以在 JVM 底层，**接口调用的理论性能是略微逊色于抽象类调用的**（尽管现代 JVM 采用了内联缓存 Inline Cache 和各种激进优化，让这点差距几乎可以忽略不计）。
#### - 5. Java 8 之后的革命：default 默认方法去哪了？

``` Java
public interface Swimable {
    default void swim() {
        System.out.println("在水里划水...");
    }
}
```
**它的底层本质是什么？**

子类继承这个接口时，如果**没有**重写 `swim()`，那么子类编译后的字节码里**根本不会**生成 `swim()` 这个方法。

当执行 `swimmer.swim()` 时，JVM 在子类的 `itable` 里找不到这个方法，它会顺着接口的继承树一路向上追溯，最终在接口 `Swimable` 本身的元数据内部找到 `swim()` 的字节码入口并执行。

所以，`default` 方法的底层物理本质是：**它依然是接口级别的公共财产，由 JVM 在运行期进行向上兜底查找。它绝对没有赋予接口“维护对象实例状态”的能力。**

### - 内部类 （ `Inner Class`)

#### - 1. 成员内部类（ Non-static Inner Class ）

这是最常见的内部类，它作为外部类的“非静态成员”存在。

``` Java
public class Outer {

    private String outerData = "外部私有数据";

    public class Inner {
        public void display() {
            // 神奇魔术：直接访问外部类的私有变量
            System.out.println(outerData); 
        }
    }
    
}
```

##### 1. 编译期的惊天谎言：独立的 `.class` 文件

当你编译 `Outer.java` 时，编译器会在磁盘上狠狠地吐出两个完全独立的字节码文件：

1. `Outer.class`
    
2. `Outer$Inner.class` （中间用 `$` 符号隔开）

在 JVM 眼里，`Outer$Inner` 是一个独立的类，那它凭什么能访问 `Outer` 的私有变量？

##### 2. 底层黑魔法：隐式的引用和合成方法（Synthetic Method）

使用 `javap -v Outer$Inner.class` 查看内部类的反编译字节码，你会发现两个编译器偷偷塞进去的秘密：

- **秘密一：隐式构造函数与父类引用（`this$0`）** 内部类的构造方法被编译器强行修改了，它会隐式传入一个外部类的指针：

``` Java
// 编译器生成的实际内部类构造方法
public Outer$Inner(Outer outer) {
    this.this$0 = outer; // this$0 是一个指向外部类对象的引用
}
```
- **秘密二：合成方法（`access$000`）** 因为 `Outer$Inner` 是独立类，本来无权访问 `Outer` 的 `private` 变量。为了打破这个限制，编译器会在外部类 `Outer` 中偷偷生成一个非私有的静态“桥接方法”：

``` java
// 编译器在 Outer 类中偷偷生成的隐藏方法
static String access$000(Outer outer) {
    return outer.outerData;
}
```
当你在内部类中写 `System.out.println(outerData);` 时，底层字节码执行的其实是：`System.out.println(Outer.access$000(this.this$0));`。

##### 3. 内存分配与致命的“内存泄漏”隐患

因为成员内部类死死地抓着外部类对象的隐式引用（`this$0`），这就导致了一个极高的内存风险：

> **如果内部类的生命周期超过了外部类（例如内部类在一个长生命周期的异步线程或 Android 的 Handler 中运行），那么即使外部类（如一个销毁的 Activity）不再使用了，垃圾回收器（GC）也会因为内部类持有的这根 `this$0` 弱丝，死活无法回收外部类，导致严重的内存泄漏。**
#### - 2. 静态内部类（ Static Nested Class ）
#### - 3. 局部内部类（ Local Inner Class ）
#### - 4. 匿名内部类（ Anonymous Inner Class ）
#### -
#### -
#### -
#### -
#### -
#### -
#### -

### - 枚举（`enum`）
### - final
### - `static`

## Java 核心 API 与工具


## 进阶与底层底层交互