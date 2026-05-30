![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752250683710-1e2e1079-4c31-481f-a617-ec263abff337.png)

# <font style="color:rgb(79, 79, 79);">一、内存结构</font>
### 1、程序计数器（PC Register）
#### 1，定义
Program Counter Register 程序计数器（寄存器）

作用：程序计数器（PC）就像一个路标，时刻指向 CPU 下一步该去执行哪一行 Java 代码指令

特点：是线程私有的，不会存在内存溢出。

#### 2，作用
+ <font style="color:rgba(0, 0, 0, 0.75);">解释器会解释指令为机器码交给 cpu 执行，程序计数器会记录下一条指令的地址行号，这样下一次解释器会从程序计数器拿到指令然后进行解释执行。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">多线程的环境下，如果两个线程发生了上下文切换，那么程序计数器会记录线程下一行指令的地址行号，以便于接着往下执行。</font>

### <font style="color:rgb(79, 79, 79);">2、虚拟机栈（JVM Stacks）</font>
#### <font style="color:rgb(79, 79, 79);">1，定义</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">每个线程运行需要的内存空间，称为虚拟机栈</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">每个栈由多个栈帧（Frame）组成，对应着每次调用方法时所占用的内存</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">每个线程只能有一个活动栈帧，对应着当前正在执行的方法（最顶部的栈帧）</font>

##### 1.1，问题辨析
1.垃圾回收是否涉及栈内存？
不会。栈内存是方法调用产生的，方法调用结束后会弹出栈。

2.栈内存分配是越大越好吗？
<font style="color:rgba(0, 0, 0, 0.75);">不是。因为物理内存是一定的，栈内存越大，可以支持更多的递归调用，但是可执行的线程数就会越少。</font>

**<font style="color:rgb(15, 17, 21);">物理内存的分配：</font>**
<font style="color:rgb(15, 17, 21);">对于一个给定的系统（物理内存总量固定），线程数（N）和每个线程的栈大小（S）之间存在一个制约关系：</font>`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">可用物理内存 ≈ N × S + 其他内存开销`

3.<font style="color:rgba(0, 0, 0, 0.75);">局部变量是否线程安全？</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">如果方法内部的变量没有逃离方法的作用访问，它是线程安全的。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">如果是局部变量引用了对象，并逃离了方法的访问，那就要考虑线程安全问题。</font>

`逃离指的是：局部变量的引用被传递到了方法外部，可能被其他线程访问到。`

#### 2，栈内存溢出
导致栈内存溢出的原因：

<font style="color:rgb(77, 77, 77);">栈帧过大、过多（递归无法正确停止）、或者第三方类库操作，都有可能造成栈内存溢出 java.lang.stackOverflowError ，使用 -Xss256k 指定栈内存大小！</font>

#### 3，栈帧的结构
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752391679141-cbcc1f87-9b9b-425e-a863-f06b4c377d0f.png)

##### 1.局部变量表（Local Variable Table）
+ 也叫本地变量表，用于存储方法的**局部变量**，包括**方法参数**和**方法内部定义的变量**。局部变量表以变量槽（Slot）为单位，每个Slot可以存储一个基本数据类型（如 int、float、boolean等）或对象引用（reference）。对于long和double类型的数据，需要占用两个连续的Slot。局部变量表的容量在编译时确定，并存储在方法的字节码中。

javap -v 类名.class 可查看局部变量。或者在idea中装jclasslib插件

```java
public void example(int a, int b) {
    int c = a + b;
}
//局部变量表：a（Slot 0）、b（Slot 1）、c（Slot 2）。
```

##### 2.操作数栈（Operand Stack）
+ **用于存储方法执行过程中的操作数和中间结果**。操作数栈是一个后进先出（`LIFO`）的栈结构。字节码指令从操作数栈中取出操作数，执行计算后将结果压入栈中。操作数栈的深度在编译时确定，并存储在方法的字节码中。

``` java
public int add(int a, int b) {
    return a + b;
}
```

``` Plain
字节码指令：
iload_1  // 将局部变量表 Slot 1 的值（a）压入操作数栈
iload_2  // 将局部变量表 Slot 2 的值（b）压入操作数栈
iadd     // 弹出栈顶的两个值相加，将结果压入栈中
ireturn  // 返回栈顶的值
```

+ 图解

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752391778685-f98a4794-f55a-4b56-962e-8a5ed38e0d81.png)

##### 3.动态链接（Dynamic Linking）


动态链接（Dynamic Linking）是栈帧中非常核心的一个物理指针。

一句话通俗解释：**它就像是方法栈帧里的“实时 GPS 导航”，当代码运行到需要调用其他方法（尤其是多态、重写方法）时，它能瞬间帮程序精准定位到内存中真正该执行的那段代码。**

为了让你彻底明白，我们把它拆解为以下三个最核心的物理本质：

###### 1. 常量池里的“符号引用”是什么？

在 Java 把代码编译成 `.class` 字节码文件时，所有的变量名、方法名、类名都还没分配真正的内存地址，它们此时只是一个代号，老老实实地呆在 Class 文件的常量池里。 比如你调用一个方法，在字节码里只是一行字：`类名.方法名:(参数类型)`。这就叫**符号引用**。 当程序跑起来后，这堆符号数据就会被加载到内存的方法区中，叫做**运行时常量池**。

###### 2. 动态链接在干什么？

每一个方法被调用时，JVM 都会在虚拟机栈里为它丢一个“栈帧”进去。这个栈帧里包含一个**特殊的物理指针**，死死地指向运行时常量池中这个方法本身的符号引用。

有些方法在编译时就能确定（比如静态方法、私有方法、构造方法），它们的符号引用在类加载时就直接变成内存指针了（这叫**静态解析**）。

但是 Java 拥有强大的“多态”和“方法重写”特性，很多方法在编译时是根本无法确定具体要执行哪段代码的。

- **经典案例**：你设计了一个父类 `Animal`（动物），里面有 `eat()` 方法。子类 `Dog`（狗）重写了 `eat()` 让它吃肉。
    
- **代码写着**：`Animal myAnimal = new Dog(); myAnimal.eat();`
    
- **编译时的尴尬**：在字节码里，JVM 只知道 `myAnimal` 是 `Animal` 类型，符号引用指向的是 `Animal.eat()`，它根本不知道运行的时候具体是哪只动物。
    
- **运行时的动态解析**：当代码真正跑起来，执行到这一行时，当前方法的栈帧就会通过**动态链接**这个 GPS 导航，去常量池里顺藤摸瓜，识别出当前运行的实际对象是 `Dog`，从而把原本模糊的 `Animal.eat()` 符号引用，**动态地翻译并替换**为内存中 `Dog.eat()` 字节码的真实物理内存地址。
    

这个在**程序运行期间**将符号引用转化为直接物理引用的过程，就叫做**动态链接**。

###### 3. 如果没有它会怎么样？

如果没有这个指向常量池的动态链接指针，栈帧就会变成一个“断了线的风筝”或“瞎子”。 当它在执行过程中遇到多态调用，或者需要动态寻找子类实现、读取类变量时，它根本不知道去内存的哪个角落找代码，Java 赖以生存的面相对象高级特性（多态、虚方法调用等）在底层物理上将彻底瘫痪。

##### 4.方法返回地址（`Return Address`）

**方法返回地址（Return Address）**（在有些资料里也叫**方法出口**），是 JVM 虚拟机栈中每一个栈帧（Stack Frame）里专门用来“记录后路”的物理区域。

一句话通俗解释它的意思： **它就像是程序运行时的“回迁地址标签”——告诉 JVM 无论当前方法是正常顺利地执行完，还是中途踩雷崩溃了，下一步都该回到调用者的哪一行代码去交差并继续运行。**

###### 🧬 为什么需要它？（物理协作场景）

Java 程序在运行时，方法之间是层层嵌套调用的。 假设你的代码中 `a()` 方法在执行到第 5 行时，调用了 `b()` 方法。

1. JVM 会立刻暂停 `a()`，并为 `b()` 创建一个全新的栈帧（**[栈帧b]**）压入栈顶。
    
2. 此时，**[栈帧b]** 内部的“方法返回地址”区域就会被立刻写入一个数据——**记录下 `a()` 方法第 6 行（即调用处的下一条指令）的内存地址**。
    
3. 这样一来，当 `b()` 自己在工位（栈顶）干完活之后，JVM 只要瞅一眼这个“返回地址”，就能立刻知道该把程序切回给 `a()` 的哪一行继续跑。

###### 🎬 方法退出时的两种物理命运

根据 `b()` 方法执行情况的不同，方法返回地址在退出时会触发两种完全不同的运行机制：

1. 正常完成出口（Normal Completion）

- **触发条件**：`b()` 方法内部的代码顺理成章地全部跑完，或者遇到了诸如 `return`（对应字节码中的 `ireturn`、`areturn` 等）指令。
    
- **物理行为**：JVM 会读取 **[栈帧b]** 里存的返回地址，把 `b()` 的程序计数器（PC）重置为调用者的下一条指令地址。同时，如果 `b()` 有返回值，会把它强行压入 `a()` 的操作数栈。最后，**[栈帧b] 弹出并销毁**，程序无缝回到 `a()` 接着往下推进。


2. 异常完成出口（Abrupt Completion）

- **触发条件**：`b()` 方法在执行时突然“踩雷”（例如发生了除以 0、空指针、数组越界等异常），且在这个方法内部**没有任何 `try-catch` 代码块**能降服这个异常。
    
- **物理行为**：此时，`b()` 属于非正常死亡，它存的那个“正常下一条指令地址”直接作废。它的返回地址将由**异常处理器表（Exception Table）来暴力决定。JVM 会去查当前方法的异常表，如果没有匹配的捕获逻辑，[栈帧b] 会直接被无情弹出并销毁，且不会给 `a()` 返回任何返回值**。这个异常会被物理抛给调用者 `a()`，让 `a()` 看看能不能接住，如果一路没人接，整个线程就会直接崩溃。

##### 5.附加信息（Additional Information）
存储一些与实现相关的附加信息，例如调试信息、性能监控数据、虚拟机版本信息，栈帧的高度等。这部分内容不是JVM规范强制要求的，具体实现可能有所不同。例如，HotSpot虚拟机可能会在这里存储一些与即时编译（JIT）相关的信息。

### 3.本地方法栈（Nation Method Stacks）
<font style="color:rgb(77, 77, 77);">一些带有 native 关键字的方法就是需要 JAVA 去调用本地的C或者C++方法，因为 JAVA 有时候没法直接和操作系统底层交互，所以需要用到本地方法栈，为本地方法提供空间，服务于带 native 关键字的方法。</font>

### 4，堆（Heap）（线程共享的区域）
#### 1、 堆内存的定义与本质

- **物理定义**：通过 `new` 关键字创建的所有对象实例以及数组，都会在堆内存中物理分配空间。
    
- **物理本质**：堆是 JVM 中专门用来“存放生命周期、大小不确定的动态数据”的工作场。当你写下 `User user = new User();` 时，变量 `user`（引用）作为一个指针保存在局部变量表（栈帧）里，而它指向的那个庞大的、真实的 `User` 对象实体，则稳稳地坐在堆内存里。

#### 2、 堆内存的核心特点（深度扩充）

##### 1. 线程共享，需考虑线程安全问题

- **为什么共享**：与每个线程独享的“虚拟机栈”不同，堆内存全家桶对 JVM 内部的所有线程都是无条件开放的。
    
- **安全风险**：如果线程 A 和线程 B 同时去修改堆内存中同一个对象的同一个属性，就会发生**数据覆盖或读写冲突（并发安全问题）**。因此，在多线程环境下访问堆中共享的数据时，必须使用 `synchronized`、锁（Lock）或者原子类（Atomic）来确保线程安全。


##### 2. 物理连续性

- 堆内存在逻辑上是连续的，但在物理内存（RAM）上**不需要连续**。它就像一块大蛋糕，JVM 会通过各种算法把它切分成零散的格子来塞数据。


#### 3、 核心拓展：多讲一点垃圾回收（GC）机制

你提到“**GC 的核心是回收不再被程序使用的内存空间**”，这句话切中了 GC 的本质。我们可以从 **“怎么算不再使用”**、**“堆的内部划分（分代）”** 以及 **“GC 是怎么干活的”** 三个物理维度来深度剖析：

##### 1. 怎么判断对象“不再被程序使用”？

JVM 绝对不会盲目杀掉一个对象，它主要通过可达性分析算法（Reachability Analysis）来判死刑：

- JVM 会设定一类绝对处于存活状态的“核心骨干”作为起点，叫做 **GC Roots**（比如栈帧里正在使用的局部变量、方法区里的静态变量）。
    
- 从这些 GC Roots 开始顺着引用物理向下搜索。如果一个堆内部的对象，**没有任何一条引用链**能连接到 GC Roots（也就是说，从核心变量出发根本找不到它了），那么这个对象就被判定为“孤立死物”，在下一次 GC 时就会被无情回收。
    

##### 2. 为什么堆内存要分代？（分而治之）

如果堆内存是一锅粥，每次 GC 都把整锅粥翻一遍去寻找死对象，效率会极低。Java 官方发现了一个铁律：**98% 的 Java 对象都是“短命鬼”**（比如一个方法里临时 `new` 出来运算的 String，方法结束它就死了）。 为了提高效率，堆内存被物理划分为两大阵营：

###### ① 新生代（Young Generation）

- **定位**：新创建的对象、短命的对象聚集地。
    
- **内部结构**：细分为 **Eden（伊甸园区）**、**From Survivor 0（幸存区0）**、**To Survivor 1（幸存区1）**，默认比例是 `8:1:1`。
    
- **运作逻辑**：
    
    1. 绝大多数新对象在 **Eden** 诞生。
        
    2. 当 Eden 满了，触发一次轻量级的垃圾回收，叫做 **Minor GC / Young GC**。
        
    3. 存活下来的幸运儿会被复制到 **Survivor 0**，并给对象贴上一个标签：年龄 = 1。
        
    4. 下一次 GC 时，Eden 和 Survivor 0 存活的对象会被一起复制到 **Survivor 1**，存活对象年龄 + 1。
        
    5. 这个过程就像两块 Survivor 抹布来回擦，对象在两个区之间飞速复制，绝大多数短命对象在这个阶段就被消灭了。

###### ② 老年代（Old Generation）

- **定位**：存放生命周期很长的、或者体积非常庞大的“百岁老人”。
    
- **晋升机制**：如果一个对象在新生代来回复制了足足 **15次**（默认年龄阈值是15，可以通过 `-XX:MaxTenuringThreshold` 调整）依然活得好好的，它就会被物理“晋升”移居到**老年代**。
    
- **运作逻辑**：老年代空间大，当老年代满了或者放不下新晋升的大对象时，就会触发极为沉重的垃圾回收，叫做 **Major GC / Full GC**。

##### 3. Full GC：高并发系统的“致命杀手”

当老年代要进行 Full GC 时，为了防止在清理过程中程序继续制造垃圾导致混乱，JVM 往往会触发一个物理行为叫做 **STW（Stop-The-World）**。

- **物理代价**：它会把除了 GC 线程以外的所有用户业务线程**全部物理暂停、强制卡死**。
    
- **后果**：如果你的 Full GC 耗时很长（比如几秒甚至十几秒），用户端的表现就是界面转圈、卡顿、请求超时。因此，JVM 调优的核心目标之一，就是**尽量减少 Full GC 的发生频率和耗时**，让对象死在新生代，别去污染老年代。

#### 4，堆内存溢出

**堆内存溢出（Heap OutOfMemoryError，简称 OOM）**是 Java 开发中最常见的严重错误之一。当程序运行时，需要创建新对象，但**堆内存中已经没有足够的物理空间分配给新对象，且经过垃圾回收（GC）后也无法释放出更多空间**，JVM 就会抛出： `java.lang.Object: java.lang.OutOfMemoryError: Java heap space`

在 JVM 底层，导致堆 OOM 的本质原因可以分为两种情况：

1.  内存泄漏（Memory Leak）

- **物理本质**：很多对象在程序中**已经不再被业务逻辑使用了**，但是它们仍然被某些生命周期极长的核心变量引用着（即通过“可达性分析算法”依然能连到 GC Roots）。
    
- **后果**：垃圾回收器（GC）不敢物理回收这些对象，导致它们像钉子户一样长期霸占堆内存，随着时间推移，堆内存被活活耗尽。
    

2.  内存溢出（Memory Overflow）

- **物理本质**：程序本身并没有写错，所有对象确实都在被高频使用，但**业务体量瞬间暴涨**，或者创建的对象实在太大，超过了物理堆内存的上限。
    
- **后果**：即使 GC 拼命干活（触发了 Full GC），也实在挤不出空间来塞新对象，只能报错。

#### 5，堆内存诊断
1. jps 工具（Java Virtual Machine Process Status Tool）

- **核心作用**：**查看当前系统中有哪些 Java 进程**。它类似于 Linux 系统中的 `ps` 命令，但它是专门用来过滤并列出 JVM 实例进程的。
    
- **常用命令**：
    
    - `jps`：仅显示本地 JVM 进程的 ID 和程序主类名。
        
    - `jps -l`：输出主类的完整包名，或者如果进程执行的是 JAR 包，则输出 JAR 文件的完整路径。
        
    - `jps -v`：列出 JVM 进程启动时的 JVM 参数（例如 `-Xms`, `-Xmx` 等）。
        

2. jmap 工具（Java Memory Map）

- **核心作用**：**查看堆内存占用情况（只能查看某一刻的快照数据）**。它是一个静态的内存映射工具，用于生成堆转储快照（Dump 文件）或物理查看内存中对象的存活分布。
    
- **常用命令**：
    
    - `jmap -heap <进程id>`：查看指定 Java 进程的**当前堆内存配置及占用情况**（包括新生代、老年代的内存总量、已使用量等）。
        
    - `jmap -histo <进程id>`：显示堆中对象的统计信息，包括类名、实例数量和合计占用内存大小。
        
    - `jmap -dump:format=b,file=heap.hprof <进程id>`：物理导出当前时刻的**堆内存快照（Dump 文件）**，用于后续分析内存泄漏。
        

3. jconsole 工具（Java Monitoring and Management Console）

- **核心作用**：**图形界面的、多功能的、可连续监测的工具**。
    
- **特点说明**：
    
    - 它是 JDK 自带的基于 GUI（图形用户界面）的监测控制台。
        
    - 通过 RMI（远程方法调用）物理连接到本地或远程的 JVM，能够**连续不断地**以折线图形式监测系统的内存趋势、线程数量、CPU 占用率以及类加载情况。
        
    - 在排查**死锁（Deadlock）**、线程阻塞和内存随时间缓慢爬升（内存泄漏）时非常直观。
        

4. jvisualvm 工具（All-in-One Java Troubleshooting Tool）

- **核心作用**：**威力加强版的、现代化的多功能图形化分析工具（高级连续监测与诊断）**。
    
- **特点说明**：
    
    - 它是 jconsole 的完美升级替代品，也是 JDK 曾经自带的最强大的性能分析神器（新版本 JDK 中已将其独立为 VisualVM 项目）。
        
    - **核心超能力**：
        
        1. **实时监控**：连续动态监测 CPU、堆、元空间、线程和类加载的曲线。
            
        2. **生成与分析 Dump**：可以直接在界面上一键物理抓取当前时刻的堆快照（Heap Dump）或线程快照（Thread Dump），并直接在内置的图形界面里人肉分析是哪行代码、哪个大对象卡死了内存。
            
        3. **性能采样（Profiler）**：可以实时监控各个 Java 方法的执行耗时和内存分配占用，帮你精准抓出系统性能的“瓶颈（CPU 消耗大户）”。

### 5，方法区
#### 1，定义

方法区是 JVM 在内存中专门划分出来的一个**线程共享**的区域。它主要用于存储**已被虚拟机加载的类型信息、常量、静态变量、以及即时编译器（JIT）编译后的代码缓存**等数据。

如果用一句话来通俗地总结它的本质：**堆内存（Heap）里存放的是“产品（具体实例对象）”，而方法区里存放的则是“生产产品的全套图纸（类模板）”以及整个工厂共享的“公共固定资产”。**

##### <font style="color:rgb(0, 0, 0);">1， 关键特性</font>
+ **<font style="color:rgb(0, 0, 0) !important;">各线程共享</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：所有线程都能够访问方法区中的数据。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">内存不连续</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：其内存空间可以是不连续的，只要逻辑上是连续的即可。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">可扩展</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：方法区的大小能够根据需求进行扩展。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">存在内存溢出风险</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：当方法区无法满足内存分配需求时，就会抛出</font>`OutOfMemoryError`<font style="color:rgba(0, 0, 0, 0.85) !important;">异常。</font>

##### <font style="color:rgb(0, 0, 0);">2， 内部存储内容</font>

1. **类型信息（Type Information）**

这部分就像是这个类的“户口本”，记录了它的社会关系和基本属性：

- **类的完整名称**：`com.example.demo.UserService`（全限定名）。
    
- **直接父类的完整名称**：`java.lang.Object`（虽然代码没写，但 Java 默认继承它）。
    
- **实现的接口列表**：`com.example.demo.BaseService`。
    
- **类的修饰符**：修饰符的二进制掩码，代表它是 `public` 和 `final` 的（这意味着它不能被继承）。

2. **字段与方法信息（Field & Method Information）**

这部分记录了类内部的“成员名单”和“具体技能（字节码）”：

- **字段信息**：
    
    - 名称：`userName`，类型：`java.lang.String`，修饰符：`private`。
        
- **方法信息**：
    
    - 名称：`login`，返回类型：`void`，修饰符：`public`。
        
    - **核心附加**：`login()` 方法内部真正的**底层字节码指令**（如 `aload_0`, `invokevirtual` 等）也保存在这里，等待被执行引擎读取。
        

3. **运行时常量池（Runtime Constant Pool）**

这部分是代码运行的“动力源”和“索引表”：

- **字面量**：代码中的文本常量 `"Welcome"` 在类加载时会被物理塞进这里。
    
- **符号引用**：代码里调用的 `System.out.println` 方法、`String` 类名，在编译时由于不知道具体内存地址，都会变成一堆符号（如 `#4 = Methodref #12.#23`）存在这里，用来支持后续的**动态链接**。
    
- **动态性扩展（`String.intern()`）**：
    
    - 正如你提到的，常量池不是死板的。如果在运行期间执行了 `String s = new String("ABC").intern();`，JVM 会去检查常量池里有没有 `"ABC"`。如果没有，它会物理地把 `"ABC"` 的引用或对象动态追加到常量池中。

4. **静态变量（Static Variables）**

这部分存放的是全家共享的“固定资产”：

- 变量 `totalUsers` 带有 `static` 修饰符，所以它**不属于任何一个 `new UserService()` 出来的对象**。
    
- 它在类加载的“准备阶段”就会在方法区被物理分配空间，并初始化为默认值 `0`，在“初始化阶段”被正式赋值为 `100`。所有线程和对象访问的都是方法区里的这同一块物理空间。

5. **JIT 编译后的代码缓存（Code Cache）**

这部分是系统的“加速器”：

- 如果这个 `login()` 方法是一个高频核心业务，被调用了成千上万次，JVM 的 **JIT（即时编译器）** 就会介入。
    
- JIT 会把 `login()` 的字节码直接物理优化并编译成 CPU 能直接听懂的**本地机器码（Native Machine Code）**，然后缓存在方法区的代码缓存区。下次再调用 `login()` 时，CPU 连字节码都不用解释了，直接起飞运行机器码，速度大幅提升。

##### 3， 方法区与永久代、元空间的关系

**① 永久代是堆的“寄生虫”，上限极低且难预测**

在 JDK 7 以前，永久代和 Java 堆在物理内存上是**连续的一整块**。

- **痛点**：永久代的大小在 JVM 启动时就必须死锁（通过 `-XX:MaxPermSize`）。但是，一个系统运行时到底会加载多少个类，是非常难以精准预测的。
    
- **后果**：随着现代企业级开发中 Spring、CGLIB 动态代理、MyBatis 等框架大量在运行期动态生成字节码类，永久代非常容易被直接撑爆，从而频繁抛出恶心且无法自愈的 `OutOfMemoryError: PermGen space`。

**② 元空间解放思想，拥抱操作系统的“本地内存”**

从 JDK 8 开始，官方彻底把方法区从 JVM 堆内存里剥离了出来，改用**元空间**，并直接物理搬迁到了操作系统的本地内存（Native Memory）空间中。

- **物理优势**：元空间不再占用 JVM 内存，它的默认上限直接变成了**你这台服务器的整块物理内存大小**。只要服务器的 RAM 没被吸干，元空间几乎再也不会因为动态加载类过多而轻易发生 OOM 崩溃。

#### 2，组成
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752305656167-f1b4c847-2933-4463-90d3-570d3c60dcec.png)

#### 3，方法区内存溢出

+ 1.8 之前会导致永久代内存溢出
```Plain
java.lang.OutOfMemoryError: PermGen space
```
    - 使用 -XX:MaxPermSize=8m 指定永久代内存大小。

+ 1.8 之后会导致元空间内存溢出
```Plain
java.lang.OutOfMemoryError: Metaspace
```
    - 使用 -XX:MaxMetaspaceSize=8m 指定元空间大小。

#### 4，常量池

为了让你对这个底层的“代码 GPS 基站”有更透彻的物理理解，我们把它们拆解为：**Class文件的常量池是什么**、**它是如何物理转换为运行时常量池的**，以及通过一个**真实的字节码案例**来还原它的运转流程。

##### 1. 静态的“类文件常量池”（Constant Pool）

- **物理位置**：存在于硬盘上的 `*.class` 字节码文件中。
    
- **本质作用**：**它是一张专属于该类的“静态索引表”**。
    
- **为什么需要它**：Java 源代码在编译成字节码时，为了节省空间，不可能在每条指令里都完整地写一遍“`java/lang/System`”、“`println`”这样长长的字符串。于是，编译器把这些类名、方法名、参数类型以及你在代码里写的数字和文本字面量，统一提取出来物理排成一个表格。在具体的指令里，只需要用一个简短的索引代号（比如 `#2`、`#3`）去指向这个表格即可。这时的指针叫做**符号引用**。
##### 2. 动态的“运行时常量池”（Runtime Constant Pool）

- **物理位置**：存在于内存（方法区/元空间）中。
    
- **本质作用**：**它是程序跑起来以后的“动态地址导航仪”**。
    
- **惊心动魄的转换（类加载阶段）**： 当 JVM 读取硬盘上的 `*.class` 文件并加载到内存中时，会把 Class 文件里的那张静态常量池表“搬”进方法区的运行时常量池中。
    
    - **核心飞跃（解析）**：在搬运和运行的过程中，JVM 会触发一个关键的物理动作——**符号引用转化为直接引用**。原本呆板的代号（如 `#2` 代表的 `a()` 方法符号），会被 JVM 动态翻译并替换为 `a()` 方法在内存中**真正的物理内存地址指针**。
        
    - **具备动态性**：正如你前面提到的，运行时常量池在程序运行期间是可以动态追加新常量的（例如通过 `String.intern()` 物理向里面塞入新的字符串引用）。

##### 3. 核心总结
- **Class文件常量池**：是写在书（硬盘文件）最前面的**静态目录索引**。
    
- **运行时常量池**：是把目录输入到了**电子导航仪（内存）中，并且把目录名直接变成了可以点击跳转的超链接（真实物理内存地址）**。
    
- 虚拟机指令（如 `ldc`、`invokevirtual`）自身不包含复杂的名称信息，它们只是一个瘦小的“快递员”，每次干活都必须拿着手里的单号（索引），去运行时常量池这个大仓库里核对，才能知道自己拿的到底是什么数据、要送去哪个方法地址。

#### 5，StringTable特性

##### 1. StringTable 的物理本质：它是个 HashTable

在 JVM 底层（C++ 实现中），StringTable 的本质是一个**固定大小的哈希表（HashTable）**，里面存储的是结构体，映射着堆中的 String 对象。

- **不能重复**：既然是哈希表，就具备 HashTable 的特性——它的 Key 是字符串的字面量（经 Hash 计算），Value 是堆中对应 String 对象的物理引用。这就天然决定了 StringTable 内部**不可能存在两个一模一样的字符串引用**。
    
- **性能敏感**：如果 StringTable 里的桶（Buckets）太少，而往里塞的字符串太多，就会发生严重的哈希冲突，导致链表变长。这会直接导致 `String.intern()` 或者字面量查找速度变慢。在 JDK 8 中，可以通过 `-XX:StringTableSize` 参数来物理调节它的大小（默认通常是 60013）。

##### 2. StringTable 的三大核心特性

###### ① 延迟加载（懒加载）

Java 代码中的字面量（如 `String s = "abc";`）在编译后，只是呆在 Class 文件的常量池里。当程序跑起来，方法被调用、执行到这一行代码时，JVM 才会去运行时常量池里把这个符号变成真正的字符串对象，并尝试物理放入 StringTable。**不用到它，它就不会提前进池子。**

###### ② 字面量自动入池

当你直接写 `String s1 = "hello";` 和 `String s2 = "hello";` 时：

1. JVM 执行到 `s1`，发现 StringTable 里没有 `"hello"`，于是就在堆中创建一个 `"hello"` 对象，并把它的引用登记到 StringTable 中，返回给 `s1`。
    
2. 执行到 `s2` 时，JVM 去 StringTable 一查：“哟，已经登记过 `"hello"` 了”，于是**直接把刚才那个对象的引用抛给 `s2`**。
    
3. 结果就是：`s1 == s2` 为 `true`，物理上它们指向堆里的同一个对象。

###### ③ 拼接动作的底层分化（高频考点）

- **常量拼接**：`String s = "a" + "b";`
    
    - **物理行为**：这是编译期优化。在编译时，编译器看它们都是写死的常量，会直接把它们融合成 `"ab"`。运行时直接去 StringTable 里找 `"ab"`。
        
- **变量拼接**：`String s1 = "a"; String s2 = s1 + "b";`
    
    - **物理行为**：由于引入了变量 `s1`，编译器无法预知未来。底层会物理物理创建一个 `StringBuilder` 对象，调用 `append("a")`，再调用 `append("b")`，最后执行 `toString()`。**`StringBuilder.toString()` 内部是直接 `new String("ab")` 出来的，这个新创建的 `"ab"` 对象绝对不会自动放入 StringTable！** 它老老实实呆在普通的堆内存里。

#### 6，核心运作机制：`String.intern()` 的主动入池

`intern()` 是 StringTable 留给程序员的一把物理操作武器。它的作用是：**强行让一个呆在普通堆内存里的字符串对象，去 StringTable 登记备案。**

由于 JDK 版本的变革，它的物理行为在不同版本中有着巨大的区别（以经典面试题为例）：

``` java
String s = new String("a") + new String("b"); // s 指向普通的堆内存 "ab"，此时 StringTable 里没有 "ab"
s.intern(); // 主动入池
String s2 = "ab"; // 去池里找
System.out.println(s == s2); 
```

###### 在 JDK 1.6 中的命运：`false`

- **物理行为**：JDK 1.6 的 StringTable 还在永久代。当调用 `s.intern()` 时，发现池子里没有 `"ab"`，JVM 会在永久代的 StringTable 里**复制一个全新的 `"ab"` 对象**，把新对象的引用存入池中。
    
- **结果**：`s` 指向堆（老家），`s2` 指向永久代（池子），它们是两个完全不同的物理地址，所以是 `false`。

###### 在 JDK 1.7 及以上（JDK 8/11/17等）的命运：`true`

- **物理行为**：此时 StringTable 被搬到了堆内存中。当调用 `s.intern()` 时，发现池子里没有 `"ab"`。为了节省内存，JVM 不再复制对象了，而是**直接把 `s` 在普通堆中的物理内存地址（引用）登记到 StringTable 中**。
    
- **结果**：当执行 `s2 = "ab"` 时，去池子里一查，发现登记的地址就是 `s` 的地址。因此 `s2` 直接拿到了 `s` 的地址。两者完全对等，返回 `true`！

##### 4. StringTable 的历史演变与垃圾回收

这也是很多人容易误解的地方：**StringTable 会触发垃圾回收（GC）吗？**

- **JDK 1.6 及以前**：StringTable 寄生在方法区的**永久代**。永久代的垃圾回收频率极低（只有 Full GC 才会动它）。如果程序高频拼接、生成大量临时字符串并调用 `intern()`，永久代会瞬间被撑爆，抛出 `OOM: PermGen space`。
    
- **JDK 1.7 开始**：官方把 StringTable **物理搬迁到了普通的 Java 堆内存（Heap）中**。
    
    - **重大物理红利**：进了堆内存，StringTable 里的字符串引用就和普通对象一样，**无条件接受堆内存 Minor GC 和 Full GC 的管辖**。当系统内存吃紧，而池子里的某些字符串已经没有任何地方引用它们时，垃圾回收器会无情地将它们从 StringTable 中清理掉，彻底根治了早期因字符串引起的内存溢出顽疾。


#### 8，StringTable 垃圾回收
##### 1. `-Xmx10m`

- **核心作用**：**手动物理锁定 Java 堆（Heap）内存的最大上限为 10MB。**
    
- **深度解析**：`-Xmx` 是 `Memory Maximum` 的物理缩写。在生产环境中，JVM 默认的最大堆内存通常是物理机内存的 $1/4$。通过显式将其限制为 `10m` 这样一个极小的数值，我们在测试环境可以非常轻松地人为模拟出堆内存溢出（OOM: Java heap space）的极端场景，用来验证代码在高负载下的健壮性或测试垃圾回收器的极限性能。
    

##### 2. `-XX:+PrintStringTableStatistics`

- **核心作用**：**在程序退出（JVM 关闭）的刹那，物理打印出整个字符串常量池（StringTable）的统计报告。**
    
- **打印出的核心数据**：
    
    - `Number of buckets`：StringTable 这个哈希表底层的**桶（Buckets）总个数**（默认通常是 60013）。
        
    - `Number of entries`：池子里当前**一共登记备案了多少个唯一的字符串对象引用**。
        
    - `Total footprint`：这些字符串在内存中一共**物理蚕食了多少字节的内存空间**。
        
- **应用场景**：当你怀疑代码中高频调用的 `String.intern()` 导致了内存飙升，或者怀疑字符串太多引发了哈希冲突、拉低了运行速度时，开启这个参数能让你一眼看清 StringTable 的底层健康状况。
    
- _(注：在 JDK 9 及以后的现代化版本中，该参数已被整合或废弃，通常使用 `-Xlog:stringtable*=debug` 来替代。)_
    

##### 3. `-XX:+PrintGCDetails`

- **核心作用**：**开启极其详细的垃圾回收（GC）日志打印模式。**
    
- **深度解析**：它不仅告诉你发生了一次 GC，还会像显微镜一样，把垃圾回收发生时**新生代（Eden区、Survivor区）、老年代以及元空间**在回收前后的具体物理内存变化、垃圾回收所消耗的精准时间（用户CPU时间/系统CPU时间）悉数打印在控制台。
    
- **应用场景**：系统调优的核心依据。通过它，你可以精准算出年轻代晋升老年代的速率，看清到底是因为 Eden 区瞬间塞满触发了 Minor GC，还是因为老年代钉子户太多逼出了沉重的 Full GC。
    

##### 4. `-verbose:gc`

- **核心作用**：**打印最基础、最直观的垃圾回收（GC）摘要信息。**
    
- **深度解析**：与 `-XX:+PrintGCDetails` 的长篇大论不同，`-verbose:gc` 属于“言简意赅型”。它通常只会输出一行简短的日志，物理记录下：**这是第几次 GC、它是 Minor GC 还是 Full GC、回收前堆内存大小、回收后堆内存大小、以及这次回收总共卡死了多少毫秒（耗时时间）**。
    
- **应用场景**：适合挂在生产环境进行长期轻量级监控。因为它输出的信息少，对系统性能的额外损耗几乎为零，同时又能让运维人员一眼盯住系统 GC 的高低频次和耗时趋势，防止系统因垃圾回收卡死（STW）而导致接口超时。

#### <font style="color:rgb(77, 77, 77);">9，StringTable 性能调优</font>

这两个关于 **StringTable 性能调优** 的核心策略，直接切中了 JVM 字符串处理的底层物理本质。为了帮你深度复习，我将这两大调优武器的底层物理逻辑和应用场景为你进行高标准复述：

##### 武器一：通过调整桶个数优化速度（`-XX:StringTableSize`）

- **物理本质复述**：
    
    StringTable 在 JVM 底层的实现是一个**哈希表（HashTable）**。既然是哈希表，数据定位的速度就极其依赖**桶（Buckets）的数量**。
    
    - **低配置的硬伤**：如果你的系统里有数百万个唯一的字符串，但 StringTable 的桶数量设置得太小（比如早期的默认值很低），就会发生极其严重的**哈希冲突**。大量的字符串会被物理挂在同一个桶的链表（或红黑树）后面。这会导致每次调用 `intern()` 或遇到新字面量时，JVM 都要花费大量的时间去遍历长长的链表进行比对，**导致系统运行速度呈断崖式下跌**。
        
    - **调优的物理动作**：通过参数 `-XX:StringTableSize=桶个数` 物理增加桶的数量（规范要求最低必须设置在 `1009` 以上，现代项目通常会设置为 `100003` 或更高）。
        
- **调优结果**：
    
    桶数量增加后，哈希表的碰撞概率暴降，字符串能够均匀地平铺在各个桶中。JVM 找字符串时几乎可以做到 $O(1)$ 的秒级定位，**大幅减少字符串放入和查找串池所消耗的时间**。

##### 武器二：通过 `intern()` 方法主动去重，暴降内存占用

- **物理本质复述**：
    
    在处理海量数据的业务场景中（例如从数据库、Excel、RPC 接口或外部文本中读取上百万条用户地址、省市区名称、状态编码等），由于这些字符串在代码里是通过网络或流动态读取的，它们**默认只会被 `new` 出来放在普通的堆内存中**。
    
    - **不调优的惨状**：如果 100 万个用户对象里有 80 万个都写着 `"北京市"`，那么堆内存里就会物理存在 80 万个独立的、内容一模一样的 String 对象，造成严重的**内存大面积浪费**。
        
    - **调优的物理动作**：在读取到字符串后，不直接对其进行引用，而是主动调用 `s = s.intern();`。
        
        1. JVM 拿着这个字符串去 StringTable 里登记。
            
        2. 如果池子里已经有内容相同的字符串引用了，`intern()` 会直接返回池子里那个老对象的地址，而刚刚动态 `new` 出来的那个临时对象就会变成无人引用的死物，在下一次 Minor GC 时被快速回收。
            
- **调优结果**：
    
    这 80 万个用户对象最终会**物理上共同指向 StringTable 里的同一个 `"北京市"` 对象**。通过让重复数据主动入池去重，可以在不改变任何业务逻辑的前提下，让整个系统的**堆内存占用量暴降数倍甚至数十倍**，彻底远离堆内存溢出（OOM）的风险。

### 直接内存

在 JVM 的内存管理和高并发性能调优中，**直接内存（Direct Memory）** 是一个经常被提及的“性能外挂”。它非常特殊，因为**它不属于 JVM 虚拟机运行时数据区的一部分，也不是 JVM 规范中定义的内存区域**。

一句话通俗解释它的本质：**它是 JVM 绕过自己的“院子”（堆内存），直接向操作系统的外层“大院”（系统物理内存）申请和使用的无机密核心存储区。**

为了让你彻底吃透直接内存，我们从**为什么需要它（核心痛点）、底层物理运转机制（零拷贝）、垃圾回收机制**以及**如何参数控制与防范溢出**四个维度进行深度拆解：

#### 1. 核心痛点：为什么需要直接内存？

在传统的 Java 阻塞 I/O（BIO）中，当我们要把硬盘上的一张图片或者一段文件通过网络发送出去，或者从网络读取数据时，数据在内存中需要经历一场**极其低效的“物理搬运”**：

1. **第一次拷贝**：操作系统先把硬盘数据读入到操作系统的**内核缓冲区（Kernel Buffer）**。
    
2. **第二次拷贝**：JVM 无法直接读取内核缓冲区的数据，它必须把数据从内核缓冲区拷贝到 Java 的堆内存（Heap）中，Java 代码才能处理。
    
3. **第三次拷贝**：当你要发送数据时，JVM 再把堆内存里的数据，拷贝到操作系统的Socket 缓冲区（Socket Buffer）中。

**致命伤**：数据在内存里来回复制，不仅白白浪费了宝贵的 CPU 算力，还让堆内存里瞬间充斥着大量临时的字节数组，引发高频的垃圾回收（GC）卡顿。

#### 2. 物理运转机制：基于 NIO 的“零拷贝（Zero-Copy）”

为了解决上述痛点，从 **JDK 1.4 开始引入了 NIO（New Input/Output）**，并带来了一个核心武器：**`DirectByteBuffer`（直接字节缓冲区）**。

当你调用 `ByteBuffer.allocateDirect(size)` 时，神奇的物理现象发生了：

- **直接向系统要空间**：JVM 通过底层的本地方法（Native Method），直接在操作系统的本地内存（Native Memory）中划分出一块物理区域。
    
- **物理映射**：这块区域对操作系统内核和 Java 程序是**共同开放、直接映射**的。
    
- **零拷贝的飞跃**：此时，操作系统的内核缓冲区和 Java 读写缓冲区共享同一块物理地址。数据从硬盘读入这块直接内存后，Java 程序可以直接在这个“公共工位”上进行读写，随后操作系统直接将其发送到网卡。

**带来的红利**：物理上**少了一次大内存块的拷贝动作**，I/O 性能迎来了质的飞跃。这就是为什么高性能网络通信框架 **Netty**、大数据组件 **Kafka** 和 **Spark** 内部大量使用直接内存的原因。

#### 3. 悬疑问题：直接内存受垃圾回收（GC）管辖吗？

很多人认为直接内存在 JVM 外面，JVM 的垃圾回收器（GC）就管不到它，其实**只对了一半**。

- **物理上确实管不到**：新生代和老年代的 GC 算法（如复制算法、标记清除）是绝对无法物理触碰直接内存的。
    
- **解铃还须系铃人（虚引用与 Cleaner 机制）**： 当你在 Java 代码中 `ByteBuffer.allocateDirect(10 * 1024 * 1024)` 申请了 10MB 的直接内存时，Java 堆内存里会物理诞生一个非常瘦小的引用对象，叫 **`DirectByteBuffer`**。这个小对象内部绑定了一个 **`Cleaner`（虚引用清洁工）**。
    
    1. 当你的业务代码结束，堆里的这个 `DirectByteBuffer` 小对象变成死物、没有任何人引用它时。
        
    2. JVM 触发 Minor GC 或 Full GC，把堆里的这个 `DirectByteBuffer` 小对象**回收销毁**。
        
    3. 在它倒下的瞬间，会物理触发绑定的 `Cleaner` 对象的 `clean()` 方法。
        
    4. `clean()` 方法内部会调用底层的 C++ 源码 `Unsafe.freeMemory()`，**主动向操作系统管家递交申请，把外面的 10MB 直接内存物理释放掉**。

**高危避坑警告**： 直接内存的释放极度依赖堆内存对象的回收。如果你的堆内存设置得很大（比如 16GB），导致 JVM 隔了很久才触发一次 GC。那么在两次 GC 之间，外面的直接内存可能已经被 Netty 等框架申请了几个 GB 却因为堆内对象还没死而**一直无法释放**，最终导致系统的物理内存被活活耗尽！

#### 4. 核心参数控制与内存溢出（OOM）

直接内存既然是内存，就绝对存在溢出的风险。如果直接内存没有空间了，JVM 就会无情抛出： `java.lang.OutOfMemoryError: Direct buffer memory`

#### 🛠️ 企业级核心控制参数

- **`-XX:MaxDirectMemorySize=大小`**： 用于手动锁死直接内存的最大物理上限（例如 `-XX:MaxDirectMemorySize=512m`）。
    
    - _注意_：如果你在启动项目时**没有手动指定**这个参数，JVM 默认允许直接内存的最大上限将**直接等于你设置的 Java 堆最大值（即 `-Xmx` 的值）**。

# 二<font style="color:rgb(79, 79, 79);">、</font>垃圾回收
### 1、介绍
+ Java中为了简化对象的释放，引入了自动的垃圾回收（Garbage Collection简称GC）机制。通过垃圾回收器来对不再使用的对象完成自动的回收，垃圾回收器主要负责对堆上的内存进行回收。其他很多现代语言比如c#、Python、Go都拥有自己的垃圾回收器。
+ 线程不共享的部分，都是伴随着线程的创建而创建，线程的销毁而销毁。因此线程不共享的程序计数器、虚拟机栈、本地方法栈中没有垃圾回收。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752392194464-1e71c9d8-08b2-4be1-ae98-947dee5719fe.png)

+ JVM（Java虚拟机）的垃圾回收机制（Garbage Collection，GC）是Java内存管理的核心部分。它负责自动回收不再使用的对象，释放内存空间，从而避免内存泄漏和手动内存管理的复杂性，垃圾收集主要是针对堆和方法区进行。
+ 垃圾：是指程序中不再被引用的对象。这些对象占用的内存可以被回收。
+ Stop-the-world：意味着JVM由于要执行GC而停止了应用程序的执行，并且这种情形会在任何一种GC算法中发生，
+ 当Stop-the-world发生时，除了GC所需的线程以外，所有线程都处于等待状态直到GC任务完成。事实上，GC优化很多时候就是指减少Stop-the-world发生的时阅。从而使系统具有高吞吐，低停顿的特点。

### 2、方法区垃圾回收
#### 1，类的生命周期
1. **<font style="color:rgb(0, 0, 0) !important;">加载（Loading）</font>**
    - **<font style="color:rgb(0, 0, 0) !important;">过程</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：通过类的全限定名来获取定义此类的二进制字节流（如.class 文件、网络、动态生成等），并将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构，同时在内存中生成一个代表这个类的</font>`<font style="color:rgb(0, 0, 0);">java.lang.Class</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">对象，作为方法区这个类的各种数据的访问入口。</font>
    - **<font style="color:rgb(0, 0, 0) !important;">注意</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：加载阶段与连接阶段的部分动作（如一部分字节码文件格式验证动作）是交叉进行的，但加载阶段的开始时间要先于连接阶段。</font>
2. **<font style="color:rgb(0, 0, 0) !important;">连接（Linking）</font>**
    - **<font style="color:rgb(0, 0, 0) !important;">验证（Verification）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证内容包括文件格式验证、元数据验证、字节码验证、符号引用验证等。</font>
    - **<font style="color:rgb(0, 0, 0) !important;">准备（Preparation）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：为类变量（被 static 修饰的变量）分配内存并设置类变量初始值（如 0、null、false 等默认值），但不包括实例变量（实例变量会在对象实例化时随着对象一起分配在堆中）。如果类变量被 final 修饰且为基本类型或字符串常量，则会在准备阶段直接赋值为指定的值。</font>
    - **<font style="color:rgb(0, 0, 0) !important;">解析（Resolution）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：虚拟机将常量池内的符号引用替换为直接引用的过程。符号引用以一组符号来描述所引用的目标，直接引用是直接指向目标的指针、相对偏移量或一个能间接定位到目标的句柄。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符等 7 类符号引用进行。</font>
3. **<font style="color:rgb(0, 0, 0) !important;">初始化（Initialization）</font>**
    - **<font style="color:rgb(0, 0, 0) !important;">触发时机</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：类的初始化阶段是类加载过程的最后一步，在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划去初始化类变量和其他资源。以下情况会触发类的初始化：</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">遇到 new、getstatic、putstatic 或 invokestatic 这四条字节码指令时（如使用 new 关键字实例化对象、读取或设置一个类的静态字段（被 final 修饰已在编译期把结果放入常量池的静态字段除外）、调用一个类的静态方法）。</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">使用 java.lang.reflect 包的方法对类进行反射调用的时候。</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">当虚拟机启动时，用户需要指定一个要执行的主类（包含 main () 方法的那个类），虚拟机会先初始化这个主类。</font>
    - **<font style="color:rgb(0, 0, 0) !important;">执行内容</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：执行类构造器</font>`<font style="color:rgb(0, 0, 0);"><clinit>()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">方法的过程。</font>`<font style="color:rgb(0, 0, 0);"><clinit>()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static {} 块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序决定的。静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但不能访问。</font>
4. **<font style="color:rgb(0, 0, 0) !important;">使用（Using）</font>**
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">类初始化完成后，就可以在程序中使用该类的实例、静态方法、静态字段等。</font>
5. **<font style="color:rgb(0, 0, 0) !important;">卸载（Unloading）</font>**
    - **<font style="color:rgb(0, 0, 0) !important;">条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：类的卸载即该类的 Class 对象被垃圾回收。要实现类的卸载，需要满足以下三个条件：</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">该类的所有实例都已经被垃圾回收。</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">加载该类的类加载器已经被垃圾回收。</font>
        * <font style="color:rgba(0, 0, 0, 0.85) !important;">该类对应的</font>`<font style="color:rgb(0, 0, 0);">java.lang.Class</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。</font>
    - **<font style="color:rgb(0, 0, 0) !important;">过程</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：当满足上述条件时，Java 虚拟机可能会卸载该类，将其占用的内存空间回收。</font>

#### 2，对象的生命周期
**<font style="color:rgb(0, 0, 0) !important;">1. 创建阶段（Creation）</font>**

<font style="color:rgba(0, 0, 0, 0.85) !important;">对象的创建是生命周期的起点，主要完成内存分配、初始化等操作，具体步骤因语言而异（以 Java 为例）：</font>

+ **<font style="color:rgb(0, 0, 0) !important;">类加载检查</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：当使用</font>`<font style="color:rgb(0, 0, 0);">new</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">关键字或反射等方式创建对象时，虚拟机首先检查该类是否已加载、连接和初始化。若未加载，则先执行类的加载过程。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">内存分配</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：为对象在堆内存中分配存储空间，包括对象的实例变量、对象头（存储哈希码、GC 分代年龄等信息）等。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">初始化零值</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：将分配到的内存空间初始化为零值（如 int 为 0，引用类型为 null），确保对象字段在未显式赋值时也有默认值。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">设置对象头</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：填充对象头信息，包括所属类的元数据指针、哈希码、GC 标记等。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">执行构造方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：调用类的构造方法（</font>`<font style="color:rgb(0, 0, 0);">constructor</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">），对对象进行初始化（如为实例变量赋值、执行构造代码块等），最终生成一个可用的对象。</font>

**<font style="color:rgb(0, 0, 0) !important;">2. 使用阶段（Usage）</font>**

<font style="color:rgba(0, 0, 0, 0.85) !important;">对象创建后进入使用阶段，此时对象被程序引用并参与各种操作：</font>

+ **<font style="color:rgb(0, 0, 0) !important;">被引用</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：对象通过引用变量（如 Java 中的对象引用）被访问，参与方法调用、属性读写、作为参数传递等。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">状态变化</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：对象的实例变量可能被修改，方法的执行会改变对象的状态（如一个</font>`<font style="color:rgb(0, 0, 0);">User</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">对象的</font>`<font style="color:rgb(0, 0, 0);">age</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">属性从 20 变为 21）。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">跨作用域传递</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：对象可在不同方法、类或线程间传递，只要存在有效引用，就不会被回收。</font>

**<font style="color:rgb(0, 0, 0) !important;">3. 不可见阶段（Invisibility）</font>**

<font style="color:rgba(0, 0, 0, 0.85) !important;">当对象不再被程序中的任何活跃引用指向时，进入不可见阶段：</font>

+ **<font style="color:rgb(0, 0, 0) !important;">表现</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：在当前作用域内，没有任何引用变量指向该对象（如局部变量超出作用域被销毁，或引用被赋值为</font>`<font style="color:rgb(0, 0, 0);">null</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">）。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">注意</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：此时对象仍可能被其他作用域的引用持有（如被静态变量或其他线程的引用指向），因此不一定会被回收。</font>

**<font style="color:rgb(0, 0, 0) !important;">4. 不可达阶段（Unreachability）</font>**

<font style="color:rgba(0, 0, 0, 0.85) !important;">当对象不再被任何可达的引用链指向时，进入不可达阶段：</font>

+ **<font style="color:rgb(0, 0, 0) !important;">判断标准</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：从根对象（如虚拟机栈中的引用、静态变量、常量池引用等）出发，无法通过引用链到达该对象。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">结果</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：对象成为垃圾回收器的回收候选，但此时尚未被回收，仍占用内存。</font>

**<font style="color:rgb(0, 0, 0) !important;">5. 收集阶段（Collection）</font>**

<font style="color:rgba(0, 0, 0, 0.85) !important;">垃圾回收器（GC）发现不可达对象后，对其进行处理：</font>

+ **<font style="color:rgb(0, 0, 0) !important;">回收前操前</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：若对象重写了</font>`<font style="color:rgb(0, 0, 0);">finalize()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">方法（Java），虚拟机会先调用该方法（仅一次），给对象最后一次 “自救” 机会（如重新建立引用）。若对象在</font>`<font style="color:rgb(0, 0, 0);">finalize()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">中成功被引用，则会重新变为可达状态，避免被回收。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">内存回收</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：若对象未 “自救” 或未重写</font>`<font style="color:rgb(0, 0, 0);">finalize()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">，垃圾回收器会释放其占用的堆内存，清理对象相关资源（如释放文件句柄、网络连接等，需手动在</font>`<font style="color:rgb(0, 0, 0);">close()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">或</font>`<font style="color:rgb(0, 0, 0);">try-with-resources</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">中处理）。</font>

**<font style="color:rgb(0, 0, 0) !important;">6. 终结阶段（Finalization）</font>**

<font style="color:rgba(0, 0, 0, 0.85) !important;">对象被彻底回收后，生命周期结束：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">内存空间被释放，对象的所有信息从内存中清除，无法再被访问。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">若对象关联了外部资源（如数据库连接），需确保在回收前已释放，否则可能导致资源泄漏。</font>

### <font style="color:rgb(79, 79, 79);">3、判断垃圾的方法</font>
#### <font style="color:rgb(79, 79, 79);">1，引用计数法</font>
引用计数法是另一种判断对象是否为垃圾的方法，但其在Java中并未被使用。它对每个对象保存一个整型的引用计数器属性，用于记录对象被引用的情况。对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器就减1；当对象A的引用计数器的值为0，即表示对象A不可能再被使用，可进行回收（Java没有采用）。

优点：

+ 回收没有延迟性，无需等到内存不够的时候才开始回收，运行时根据对象计数器是否为0，可以直接回收
+ 在垃圾回收过程中，应用无需挂起；如果申请内存时，内存不足，则立刻报OOM 错误
+ 区域性，更新对象的计数器时，只是影响到该对象，不会扫描全部对象

缺点：

+ 每次对象被引用时，都需要去更新计数器，有一点时间开销
+ 浪费CPU资源，即使内存够用，仍然在运行时进行计数器的统计。
+ 无法解决循环引用问题，会引发内存泄露（最大的缺点）

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752393712031-cccfbcea-53a5-47a5-b00d-0d28823dee92.png)

#### 2，三色标记<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(0, 0, 0, 0.04);">（Tri-color marking）</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">三色标记（Tri-color marking）是一种用于垃圾回收（Garbage Collection, GC）的算法，属于追踪式垃圾回收的范畴。其核心思想是通过三种颜色标记对象的不同状态，以此来识别哪些对象是存活的、哪些是可以被回收的，从而在垃圾回收过程中高效地管理内存。</font>

##### <font style="color:rgb(0, 0, 0);">1，三色标记的基本原理</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">三色标记算法将对象标记为以下三种颜色</font>

1. **<font style="color:rgb(0, 0, 0) !important;">白色</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：表示对象尚未被垃圾回收器访问到。在初始阶段，所有对象都是白色的。如果在标记结束后，对象仍然是白色的，这意味着它不可达，会被视为垃圾并回收。</font>
2. **<font style="color:rgb(0, 0, 0) !important;">灰色</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：表示对象已经被垃圾回收器访问过，但它的某些引用还未被处理。灰色对象是白色对象和黑色对象之间的中间状态。垃圾回收器会从灰色对象出发，继续遍历其引用的对象。</font>
3. **<font style="color:rgb(0, 0, 0) !important;">黑色</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：表示对象已经被垃圾回收器访问过，并且它的所有引用都已经被处理完毕。黑色对象的所有引用对象都已经被标记为灰色或黑色，因此黑色对象不会再被重新处理。</font>

##### <font style="color:rgb(0, 0, 0);">2，三色标记的工作流程</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">三色标记算法的工作流程可以分为以下几个步骤：</font>

1. **<font style="color:rgb(0, 0, 0) !important;">初始标记</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：垃圾回收器从根对象（如栈、静态变量等）开始，将直接可达的对象标记为灰色，并将这些灰色对象放入待处理队列。</font>
2. **<font style="color:rgb(0, 0, 0) !important;">并发标记</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：垃圾回收器从灰色对象队列中取出对象，将其标记为黑色，并将其引用的所有白色对象标记为灰色，然后将这些新标记的灰色对象加入队列。这个过程会持续进行，直到灰色对象队列为空。</font>
3. **<font style="color:rgb(0, 0, 0) !important;">最终标记</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：在并发标记阶段，可能会有新的引用关系被创建或修改，导致一些应该被标记的对象没有被标记（即漏标问题）。最终标记阶段会处理这些增量更新，确保所有存活对象都被正确标记。</font>
4. **<font style="color:rgb(0, 0, 0) !important;">垃圾回收</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：标记完成后，所有白色对象都被视为垃圾，垃圾回收器会回收它们占用的内存空间。</font>

###### <font style="color:rgb(0, 0, 0) !important;">漏标问题</font>
###### **<font style="color:rgb(0, 0, 0) !important;">1，漏标问题的产生原因</font>**
<font style="color:rgba(0, 0, 0, 0.85) !important;">三色标记算法允许垃圾回收线程与应用程序线程并发执行（即标记过程中，用户线程仍在修改对象的引用关系）。这种并发特性可能导致：  
</font>**<font style="color:rgb(0, 0, 0) !important;">原本应该被标记为存活的对象，最终被标记为白色（垃圾）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">，即 “漏标”。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">2，漏标问题的触发条件</font>**
<font style="color:rgba(0, 0, 0, 0.85) !important;">漏标需要同时满足以下两个条件（缺一不可）：</font>  


1. **<font style="color:rgb(0, 0, 0) !important;">新增引用</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：一个</font>**<font style="color:rgb(0, 0, 0) !important;">黑色对象</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">（已完成标记，其引用被认为已处理完毕）新增了对一个</font>**<font style="color:rgb(0, 0, 0) !important;">白色对象</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">的引用。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">黑色对象的引用被视为 “已处理”，垃圾回收器不会再遍历其引用，因此新增的白色对象引用可能被忽略。</font>
2. **<font style="color:rgb(0, 0, 0) !important;">引用删除</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：所有</font>**<font style="color:rgb(0, 0, 0) !important;">灰色对象</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">（正在处理其引用的对象）对该白色对象的直接或间接引用</font>**<font style="color:rgb(0, 0, 0) !important;">全部被删除</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">灰色对象是标记过程的 “桥梁”，若它们对白色对象的引用被删除，白色对象就失去了被标记的唯一途径，最终保持白色。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">3，举例说明</font>**
<font style="color:rgba(0, 0, 0, 0.85) !important;">假设初始状态：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">根对象引用灰色对象 A（A 的引用正在处理），A 引用白色对象 B（未被标记）。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">并发过程中发生两个操作：</font>

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">用户线程删除 A 对 B 的引用（满足条件 2：灰色对象 A 对 B 的引用消失）。</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">用户线程让黑色对象 C（已完成标记）新增对 B 的引用（满足条件 1：黑色对象 C 引用 B）。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">结果：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">垃圾回收器不会再处理黑色对象 C 的引用，也不会再通过 A 遍历到 B，导致 B 始终为白色，被误判为垃圾。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">4，漏标问题的影响</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">程序崩溃</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：被误标的存活对象被回收后，应用程序线程若继续访问该对象，会导致空指针异常（NPE）或数据错误。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">这是三色标记算法必须解决的核心问题，否则并发垃圾回收无法保证正确性。</font>

#### 3，可达性分析算法
通过一系列称为GC Roots的根对象作为起点，从这些根对象出发，遍历所有可达的对象，未被遍历到的对象则被视为垃圾。可达性分析算法的分析工作必须在一个保障一致性的快照中进行，否则结果的准确性无法保证，这也是导致GC进行时必须Stop The World的一个原因。

可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索走过的路径称为引用链，如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象，在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象

流程：

+ 确定GC Roots：GC Roots是一组不会被回收的根对象，
+ 遍历对象图：从GC Roots出发，递归遍历所有可达的对象，并标记这些对象为存活对象。
+ 清除不可达对象：未被标记的对象则被视为不可达对象，可以被回收。

### <font style="color:rgb(79, 79, 79);">4，四种引用</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752406926360-1bf8e6b9-ba8d-4513-b604-d2babaabce61.png)

1. **强引用**

只有所有 GC Roots 对象都不通过【强引用】引用该对象，该对象才能被垃圾回收

2. **软引用（SoftReference）**

仅有软引用引用该对象时，在垃圾回收后，内存仍不足时会再次出发垃圾回收，回收软引用对象

可以配合引用队列来释放软引用自身

3. **弱引用（WeakReference）**

仅有弱引用引用该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象

可以配合引用队列来释放弱引用自身

4. **虚引用（PhantomReference）**

必须配合引用队列使用，主要配合 ByteBuffer 使用，被引用对象回收时，会将虚引用入队，

由 Reference Handler 线程调用虚引用相关方法释放直接内存

5. **终结器引用（FinalReference）**

无需手动编码，但其内部配合引用队列使用，在垃圾回收时，终结器引用入队（被引用对象暂时没有被回收），再由 Finalizer 线程通过终结器引用找到被引用对象并调用它的 finalize 方法，第二次 GC 时才能回收被引用对象。

### 5，<font style="color:rgb(79, 79, 79);">垃圾回收算法</font>
#### <font style="color:rgb(79, 79, 79);">1，标记清除</font>
<font style="color:rgb(77, 77, 77);">定义：Mark Sweep</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">速度较快</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">会产生内存碎片</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752410940262-36212219-d99f-4ccd-8e67-d47d545f3de6.png)

#### <font style="color:rgb(79, 79, 79);">2，标记整理</font>
<font style="color:rgb(77, 77, 77);">Mark Compact</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">速度慢</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">没有内存碎片</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752410952703-d98ac62d-cfca-4af1-a645-998ac10736e4.png)

#### <font style="color:rgb(79, 79, 79);">3，复制</font>
<font style="color:rgb(77, 77, 77);">Copy</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">不会有内存碎片</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">需要占用两倍内存空间</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752410969713-28636dd2-30da-42b0-a819-fd36fe4a3e16.png)

### <font style="color:rgb(79, 79, 79);">6，分代垃圾回收</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752411856549-e8bf6ff5-67b8-4080-bfe5-5e6158d38474.png)

+ 新创建的对象首先分配在 eden 区
+ 新生代空间不足时，触发 minor gc ，eden 区 和 from 区存活的对象使用 - copy 复制到 to 中，存活的对象年龄加一，然后交换 from to
+ minor gc 会引发 stop the world，暂停其他线程，等垃圾回收结束后，恢复用户线程运行
+ 当幸存区对象的寿命超过阈值时，会晋升到老年代，最大的寿命是 15（4bit）
+ 当老年代空间不足时，会先触发 minor gc，如果空间仍然不足，那么就触发 full fc ，停止的时间更长！

4、垃圾回收器

#### 1，相关概念
+ 并行收集：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。
+ 并发收集：指用户线程与垃圾收集线程同时工作（不一定是并行的可能会交替执行）。用户程序在继续运行，而垃圾收集程序运行在另一个 CPU 上
+ 吞吐量：即 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值（吞吐量 = 运行用户代码时间 / ( 运行用户代码时间 + 垃圾收集时间 )），也就是。例如：虚拟机共运行 100 分钟，垃圾收集器花掉 1 分钟，那么吞吐量就是 99% 。

#### <font style="color:rgb(79, 79, 79);">2，串行</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">单线程。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">堆内存较少，适合个人电脑。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752432898394-1bbcf226-c309-49b1-be12-8d86b51371c0.png)

安全点：让其他线程都在这个点停下来，以免垃圾回收时移动对象地址，使得其他线程找不到被移动的对象

因为是串行的，所以只有一个垃圾回收线程。且在该线程执行回收工作时，其他线程进入阻塞状态

**Serial 收集器**

Serial 收集器是最基本的、发展历史最悠久的收集器

特点：单线程、简单高效（与其他收集器的单线程相比），采用复制算法。对于限定单个 CPU 的环境来说，Serial 收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。收集器进行垃圾回收时，必须暂停其他所有的工作线程，直到它结束（Stop The World）！

**ParNew 收集器**

ParNew 收集器其实就是 Serial 收集器的多线程版本

特点：多线程、ParNew 收集器默认开启的收集线程数与CPU的数量相同，在 CPU 非常多的环境中，可以使用 -XX:ParallelGCThreads 参数来限制垃圾收集的线程数。和 Serial 收集器一样存在 Stop The World 问题

**Serial Old 收集器**

Serial Old 是 Serial 收集器的老年代版本

特点：同样是单线程收集器，采用标记-整理算法

#### <font style="color:rgb(79, 79, 79);">2，吞吐量优先</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">多线程。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">堆内存较大，多核 cpu。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">让单位时间内，STW 的时间最短。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752432989626-96553f1a-e2d5-4355-a21d-638d16a27748.png)

`**Parallel Scavenge**`** 收集器：**与吞吐量关系密切，故也称为吞吐量优先收集器。

特点：属于新生代收集器也是采用复制算法的收集器（用到了新生代的幸存区），又是并行的多线程收集器（与 ParNew 收集器类似）。该收集器的目标是达到一个可控制的吞吐量。还有一个值得关注的点是：GC自适应调节策略（与 ParNew 收集器最重要的一个区别）。

**GC自适应调节策略：**Parallel Scavenge 收集器可设置 -XX:+UseAdptiveSizePolicy 参数。

当开关打开时不需要手动指定新生代的大小（-Xmn）、Eden 与 Survivor 区的比例（-XX:SurvivorRation）、

晋升老年代的对象年龄（-XX:PretenureSizeThreshold）等，虚拟机会根据系统的运行状况收集性能监控信息，动态设置这些参数以提供最优的停顿时间和最高的吞吐量，这种调节方式称为 GC 的自适应调节策略。

`**Parallel Scavenge**`** 收集器使用两个参数控制吞吐量：**

XX:MaxGCPauseMillis=ms 控制最大的垃圾收集停顿时间（默认200ms）

XX:GCTimeRatio=rario 直接设置吞吐量的大小

`**Parallel Old**`** 收集器：**是 Parallel Scavenge 收集器的老年代版本

特点：多线程，采用标记-整理算法（老年代没有幸存区）。

#### <font style="color:rgb(79, 79, 79);">3，响应时间优先</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">多线程。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">堆内存较大，多核 cpu。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">尽可能让 STW 的单次时间最短。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752433080211-b170b870-e69f-4350-9120-f07b2148dc7e.png)

**CMS 收集器：**Concurrent Mark Sweep，一种以获取最短回收停顿时间为目标的老年代收集器

**特点：**基于标记-清除算法实现。并发收集、低停顿，但是会产生内存碎片。

**应用场景：**适用于注重服务的响应速度，希望系统停顿时间最短，给用户带来更好的体验等场景下。如 web 程序、b/s 服务。

**CMS 收集器的运行过程分为下列4步：**

**初始标记：**标记 GC Roots 能直接到的对象。速度很快但是仍存在 Stop The World 问题。

**并发标记：**进行 GC Roots Tracing 的过程，找出存活对象且用户线程可并发执行。

**重新标记：**为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录。仍然存在 Stop The World 问题

**并发清除：**对标记的对象进行清除回收，清除的过程中，可能任然会有新的垃圾产生，这些垃圾就叫浮动垃圾，如果当用户需要存入一个很大的对象时，新生代放不下去，老年代由于浮动垃圾过多，就会退化为 serial Old 收集器，将老年代垃圾进行标记-整理，当然这也是很耗费时间的！

CMS 收集器的内存回收过程是与用户线程一起并发执行的，可以搭配 ParNew 收集器（多线程，新生代，复制算法）与 Serial Old 收集器（单线程，老年代，标记-整理算法）使用。

##### CMS退化
<font style="color:rgba(0, 0, 0, 0.85) !important;">CMS（Concurrent Mark Sweep）在以下情况下会退化为</font>**<font style="color:rgb(0, 0, 0) !important;">Serial Old 收集器</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">进行 Full GC，导致长时间的 STW（Stop The World）停顿：</font>

###### **<font style="color:rgb(0, 0, 0) !important;">1. 并发模式失败（Concurrent Mode Failure）</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">触发条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">在 CMS 的</font>**<font style="color:rgb(0, 0, 0) !important;">并发标记</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">或</font>**<font style="color:rgb(0, 0, 0) !important;">并发清除</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">阶段，用户线程持续创建新对象，导致</font>**<font style="color:rgb(0, 0, 0) !important;">老年代空间不足以容纳晋升的对象</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">原因</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">CMS 的并发收集速度跟不上对象晋升到老年代的速度，例如：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">新生代对象存活率过高，晋升对象过多。</font>
    - `<font style="color:rgb(0, 0, 0);">-XX:CMSInitiatingOccupancyFraction</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">参数设置过高，导致 CMS 启动过晚。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">解决方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">降低</font>`<font style="color:rgb(0, 0, 0);">CMSInitiatingOccupancyFraction</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（如设为 60%），提前触发 CMS GC，预留更多空间。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">2. 晋升失败（Promotion Failure）</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">触发条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">新生代对象晋升到老年代时，老年代剩余空间</font>**<font style="color:rgb(0, 0, 0) !important;">不足以容纳该对象</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">，或因</font>**<font style="color:rgb(0, 0, 0) !important;">内存碎片</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">导致无法找到连续空间。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">原因</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">CMS 采用</font>**<font style="color:rgb(0, 0, 0) !important;">标记 - 清除算法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">，会产生内存碎片，导致大对象无法分配。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">浮动垃圾（Floating Garbage）过多，占用老年代空间。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">解决方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">启用</font>`<font style="color:rgb(0, 0, 0);">-XX:+UseCMSCompactAtFullCollection</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（默认开启），在 Full GC 时进行内存整理。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">增加</font>`<font style="color:rgb(0, 0, 0);">-XX:CMSFullGCsBeforeCompaction</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">参数（如设为 0），每次 Full GC 后都进行整理。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">3. 大对象直接分配到老年代</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">触发条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">当创建的</font>**<font style="color:rgb(0, 0, 0) !important;">大对象</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">（如数组、大集合）超过</font>`<font style="color:rgb(0, 0, 0);">-XX:PretenureSizeThreshold</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">阈值时，会直接在老年代分配。若此时老年代空间不足，触发 Full GC。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">解决方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">调整</font>`<font style="color:rgb(0, 0, 0);">PretenureSizeThreshold</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">参数，避免过小对象直接进入老年代。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">增加堆内存或老年代空间比例。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">4. 显示调用 System.gc ()</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">触发条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">代码中显式调用</font>`<font style="color:rgb(0, 0, 0);">System.gc()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">，且未通过</font>`<font style="color:rgb(0, 0, 0);">-XX:+DisableExplicitGC</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">禁用时，CMS 会触发 Full GC。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">解决方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">禁止代码中使用</font>`<font style="color:rgb(0, 0, 0);">System.gc()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">启用</font>`<font style="color:rgb(0, 0, 0);">-XX:+DisableExplicitGC</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">参数忽略显式 GC 请求。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">5. Metaspace 空间不足</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">触发条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">JDK 8 + 中，元空间（Metaspace）存储类元数据，若元空间不足，会触发 Full GC。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">原因</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">动态生成大量类（如反射、CGLIB 代理）。</font>
    - `<font style="color:rgb(0, 0, 0);">-XX:MetaspaceSize</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">或</font>`<font style="color:rgb(0, 0, 0);">-XX:MaxMetaspaceSize</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">设置过小。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">解决方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">增加元空间大小：</font>`<font style="color:rgb(0, 0, 0);">-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">。</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">使用</font>`<font style="color:rgb(0, 0, 0);">-XX:+CMSClassUnloadingEnabled</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">允许 CMS 回收无用类。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">6. 初始标记失败</font>**
+ **<font style="color:rgb(0, 0, 0) !important;">触发条件</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">在 CMS 的</font>**<font style="color:rgb(0, 0, 0) !important;">初始标记阶段</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">（STW），若老年代空间不足，无法完成根对象的标记，退化为 Serial Old。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">解决方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">同 “并发模式失败”，调整</font>`<font style="color:rgb(0, 0, 0);">CMSInitiatingOccupancyFraction</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">参数。</font>

###### **<font style="color:rgb(0, 0, 0) !important;">总结</font>**
<font style="color:rgba(0, 0, 0, 0.85) !important;">CMS 退化为 Serial Old 的核心原因是</font>**<font style="color:rgb(0, 0, 0) !important;">老年代空间不足</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">或</font>**<font style="color:rgb(0, 0, 0) !important;">内存碎片</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">，导致无法在并发模式下完成垃圾回收。优化方向包括：</font>

1. **<font style="color:rgb(0, 0, 0) !important;">提前触发 CMS</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：降低</font>`<font style="color:rgb(0, 0, 0);">CMSInitiatingOccupancyFraction</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">。</font>
2. **<font style="color:rgb(0, 0, 0) !important;">减少内存碎片</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：启用</font>`<font style="color:rgb(0, 0, 0);">-XX:+UseCMSCompactAtFullCollection</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">。</font>
3. **<font style="color:rgb(0, 0, 0) !important;">控制对象晋升</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：避免大对象直接进入老年代，减少浮动垃圾。</font>
4. **<font style="color:rgb(0, 0, 0) !important;">监控与调优</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：通过工具（如 G1GCViewer、jstat）监控 GC 日志，及时调整参数。</font>

#### 4，<font style="color:rgb(79, 79, 79);">G1 收集器</font>
**<font style="color:rgb(77, 77, 77);">定义：</font>**<font style="color:rgb(77, 77, 77);"> Garbage First</font>

<font style="color:rgba(0, 0, 0, 0.85);">G1 将堆内存划分为多个大小相等的独立</font>**<font style="color:rgb(0, 0, 0) !important;">Region</font>**<font style="color:rgba(0, 0, 0, 0.85);">（默认 1MB~32MB），每个 Region 可动态标记为</font>**<font style="color:rgb(0, 0, 0) !important;">Eden 区（E）、Survivor 区（S）或老年代（O）</font>**<font style="color:rgba(0, 0, 0, 0.85);">。</font>

**<font style="color:rgb(77, 77, 77);">适用场景：</font>**

+ <font style="color:rgba(0, 0, 0, 0.75);">同时注重吞吐量和低延迟（响应时间）</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">超大堆内存（内存大的），会将堆内存划分为多个大小相等的区域</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">整体上是标记-整理算法，两个区域之间是复制算法</font>

**<font style="color:rgb(77, 77, 77);">相关参数：</font>**  
<font style="color:rgb(77, 77, 77);">JDK8 并不是默认开启的，所需要参数开启</font>

```plain
-XX:+UseG1GC
-XX:G1HeapRegionSize=size
-XX:MaxGCPauseMillis=time
```

##### <font style="color:rgb(79, 79, 79);">G1 垃圾回收阶段</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752481229021-7c01b6e5-a293-43ce-9cc1-458860ce349b.png)

**Young Collection**：对新生代垃圾收集

**Young Collection + Concurrent Mark**：如果老年代内存到达一定的阈值了，新生代垃圾收集同时会执行一些并发的标记。

**Mixed Collection**：会对新生代 + 老年代 + 幸存区等进行混合收集，然后收集结束，会重新进入新生代收集。

##### <font style="color:rgb(79, 79, 79);">Young Collection</font>
**<font style="color:rgb(77, 77, 77);">新生代存在 STW：</font>**  
<font style="color:rgb(77, 77, 77);">分代是按对象的生命周期划分，分区则是将堆空间划分连续几个不同小区间，每一个小区间独立回收，可以控制一次回收多少个小区间，方便控制 GC 产生的停顿时间！</font>  
<font style="color:rgb(77, 77, 77);">E：eden，S：幸存区，O：老年代</font>  
<font style="color:rgb(77, 77, 77);">新生代收集会产生 STW ！</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/gif/58546395/1752481317146-24ee4d10-c174-4a44-b8bd-582eeeb73a88.gif)

##### <font style="color:rgb(79, 79, 79);">Young Collection + CM</font>
<font style="color:rgb(77, 77, 77);">在 Young GC 时会进行 GC Root 的初始化标记。</font><font style="color:rgb(77, 77, 77);">老年代占用堆空间比例达到阈值时，进行并发标记（不会STW），由下面的 JVM 参数决定 -</font>`<font style="color:rgb(77, 77, 77);">XX:InitiatingHeapOccupancyPercent=percent （默认45%）</font>`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752481426024-87a3f3a9-b6a9-4984-b4d8-e1c841b57f17.png)

##### <font style="color:rgb(79, 79, 79);">Mixed Collection</font>
会对 E S O 进行全面的回收

+ 最终标记会 STW
+ 拷贝存活会 STW

-`XX:MaxGCPauseMills=xxms `用于指定最长的停顿时间！

问：为什么有的老年代被拷贝了，有的没拷贝？

因为指定了最大停顿时间，如果对所有老年代都进行回收，耗时可能过高。为了保证时间不超过设定的停顿时间，会回收最有价值的老年代（回收后，能够得到更多内存）。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752481485704-9bb03d06-cfd0-4d7e-8726-c30901634565.png)

##### Full GC
+ <font style="color:rgb(0, 0, 0) !important;">老年代空间不足</font><font style="color:rgba(0, 0, 0, 0.85) !important;">：当对象晋升到老年代时，老年代没有足够空间容纳。</font>
+ 如果垃圾产生速度慢于垃圾回收速度，不会触发 Full GC，还是并发地进行清理
+ 如果垃圾产生速度快于垃圾回收速度，便会触发 Full GC，然后退化成 serial Old 收集器串行的收集，就会导致停顿的时候长。

##### <font style="color:rgb(79, 79, 79);">Young Collection 跨代引用</font>
**<font style="color:rgb(15, 17, 21);">Young Collection 跨代引用</font>**<font style="color:rgb(15, 17, 21);"> 指的是在 </font>**<font style="color:rgb(15, 17, 21);">年轻代</font>**<font style="color:rgb(15, 17, 21);"> 中的对象，被 </font>**<font style="color:rgb(15, 17, 21);">老年代</font>**<font style="color:rgb(15, 17, 21);"> 中的对象所引用。这种引用关系是年轻代垃圾回收（Minor GC）时的一个“麻烦”，因为它会让被引用的年轻代对象不能被回收。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752483691441-fe08675c-2259-4c0e-9b60-f0bd441993b7.png)

**<font style="color:rgb(15, 17, 21);">卡表（Card Table）和记忆集（Remembered Set）</font>**

+ **<font style="color:rgb(15, 17, 21);">记忆集</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">=</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">“需求”</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">（我需要知道谁引用了我的对象）</font>
+ **<font style="color:rgb(15, 17, 21);">卡表</font>**<font style="color:rgb(15, 17, 21);"> = </font>**<font style="color:rgb(15, 17, 21);">“解决方案之一”</font>**<font style="color:rgb(15, 17, 21);"> （我用一个字节数组按块来记录）</font>
+ **<font style="color:rgb(15, 17, 21);">记忆集（Remembered Set）</font>**存在于E中，用于保存新生代对象对应的脏卡。
+ **脏卡**：O 被划分为多个区域（一个区域512K），如果该区域引用了新生代对象，则该区域被称为脏卡。
+ 在引用变更时通过 post-write barried + dirty card queue。
+ concurrent refinement threads 更新 Remembered Set。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752483748670-965a7987-06a1-46e3-ba1a-ef40951256d8.png)

##### <font style="color:rgb(79, 79, 79);">Remark（</font><font style="color:rgb(77, 77, 77);">重新标记</font><font style="color:rgb(79, 79, 79);">）</font><font style="color:rgb(77, 77, 77);"></font>
<font style="color:rgb(77, 77, 77);">在垃圾回收时，收集器处理对象的过程中</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">黑色：已被处理，需要保留的</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">灰色：正在处理中的</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">白色：还未处理的</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752483764687-1e5dea6f-1d8f-43fa-9973-4663a44b0021.png)

但是在并发标记过程中，有可能 A 被处理了以后未引用 C ，但该处理过程还未结束，在处理过程结束之前 A 引用了 C ，这时就会用到 remark 。

过程如下：

之前 C 未被引用，这时 A 引用了 C ，就会给 C 加一个写屏障，写屏障的指令会被执行，将 C 放入一个队列当中，并将 C 变为 处理中状态

在并发标记阶段结束以后，重新标记阶段会 STW ，然后将放在该队列中的对象重新处理，发现有强引用引用它，就会处理它，由灰色变成黑色。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752483799231-5f6359db-7b16-4423-804d-2216ad209fc4.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752483818868-a2996ed0-4b87-4050-a77f-e8586bf4c3ff.png)

##### JDK 8u20 字符串去重
**过程：**

1. 将所有新分配的字符串（底层是 char[] ）放入一个队列。
2. 当新生代回收时，G1 并发检查是否有重复的字符串。
3. 如果字符串的值一样，就让他们引用同一个字符串对象。

注意：

+ 其与 String.intern() 的区别
+ String.intern() 关注的是字符串对象
+ 字符串去重关注的是 char[]
+ 在 JVM 内部，使用了不同的字符串标

**优点与缺点**

优点：节省了大量内存

缺点：新生代回收时间略微增加，导致略微多占用 CPU

##### <font style="color:rgb(79, 79, 79);">JDK 8u40 并发标记类卸载</font>
<font style="color:rgb(77, 77, 77);">在并发标记阶段结束以后，就能知道哪些类不再被使用。如果一个类加载器的所有类都不在使用，则卸载它所加载的所有类。</font>

##### <font style="color:rgb(79, 79, 79);">JDK 8u60 回收巨型对象</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">一个对象大于region的一半时，就称为巨型对象</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">G1不会对巨型对象进行拷贝</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">回收时被优先考虑</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">G1会跟踪老年代所有incoming引用，如果老年代incoming引用为0的巨型对象就可以在新生代垃圾回收时处理掉</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752483983490-65741e9b-45e1-4022-a8ad-17401bb201e2.png)

#### <font style="color:rgb(79, 79, 79);">5、垃圾回收调优</font>
<font style="color:rgb(77, 77, 77);">查看虚拟机参数命令</font>

```powershell
D:\JavaJDK1.8\bin\java  -XX:+PrintFlagsFinal -version | findstr "GC"
```

可以根据参数去查询具体的信息

**1，调优领域**

+ 内存
+ 锁竞争
+ cpu 占用
+ io
+ gc

**2，确定目标**

低延迟/高吞吐量？ 选择合适的GC

+ CMS(<font style="color:rgb(15, 17, 21);">并发标记清除收集器</font>), G1(<font style="color:rgb(15, 17, 21);">G1垃圾回收器</font>), ZGC(<font style="color:rgb(15, 17, 21);">Z垃圾回收器</font>)
+ ParallelGC(<font style="color:rgb(15, 17, 21);">并行垃圾回收器</font>)
+ Zing(<font style="color:rgb(15, 17, 21);">商业JVM平台</font>)

**3，最快的 GC就是不GC**

首先排除减少因为自身编写的代码而引发的内存问题

查看 Full GC 前后的内存占用，考虑以下几个问题

+ 数据是不是太多？
+ 数据表示是否太臃肿
+ 对象大小 16 Integer 24 int 4
+ 是否存在内存泄漏
+ 第三方缓存实现

**4，新生代调优**

新生代的特点：

+ 所有的 new 操作分配内存都是非常廉价的
+ TLAB thread-lcoal allocation buffer(**<font style="color:rgb(15, 17, 21);">线程本地分配缓冲区</font>**)
+ 死亡对象回收零代价
+ 大部分对象用过即死（朝生夕死）
+ Minor GC 所用时间远小于 Full GC

**5，新生代内存越大越好么？**

不是

新生代内存太小：频繁触发 Minor GC ，会 STW ，会使得吞吐量下降

新生代内存太大：老年代内存占比有所降低，会更频繁地触发 Full GC。而且触发 Minor GC 时，清理新生代所花费的时间会更长

**新生代内存设置为内容纳[并发量*(请求-响应)]的数据为宜**

**幸存区需要能够保存 当前活跃对象+需要晋升的对象**

晋升阈值配置得当，让长时间存活的对象尽快晋升

```java
-XX:MaxTenuringThreshold=threshold
-XX:+PrintTenuringDistrubution
```

**<font style="color:rgb(79, 79, 79);">5，老年代调优</font>**

<font style="color:rgb(77, 77, 77);">以 CMS 为例：</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">CMS 的老年代内存越大越好</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">先尝试不做调优，如果没有 Full GC 那么已经，否者先尝试调优新生代。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">观察发现 Full GC 时老年代内存占用，将老年代内存预设调大 1/4 ~ 1/3</font>

```java
-XX:CMSInitiatingOccupancyFraction=percent
```

**<font style="color:rgb(79, 79, 79);">6，案例</font>**

<font style="color:rgb(77, 77, 77);">案例1：Full GC 和 Minor GC 频繁</font>

<font style="color:rgb(77, 77, 77);">新生代空间小，让一些没有到达晋升标准的对象到达老年代使其老年代空间紧张，频繁发生垃圾回收。  
</font><font style="color:rgb(77, 77, 77);">案例2：请求高峰期发生 Full GC，单次暂停时间特别长（CMS）</font>

<font style="color:rgb(77, 77, 77);">在并发表标记前对新生代进行垃圾回收。  
</font><font style="color:rgb(77, 77, 77);">案例3：老年代充裕情况下，发生 Full GC（jdk1.7）</font>

<font style="color:rgb(77, 77, 77);">永久代空间不够。</font>

# <font style="color:rgb(77, 77, 77);">三</font><font style="color:rgb(79, 79, 79);">、</font><font style="color:rgb(77, 77, 77);">类加载和字节码技术</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752571631433-e48016f7-7bbc-4fe1-88a6-f733eb6c12e5.png)

### <font style="color:rgb(79, 79, 79);">1、类文件结构</font>
<font style="color:rgb(77, 77, 77);">通过 javac 类名.java 编译 java 文件后，会生成一个 .class 的文件！  
</font><font style="color:rgb(77, 77, 77);">以下是字节码文件：</font>

```plain
0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 09 
0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b 07 
0000040 00 1c 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29 
0000060 56 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e 
0000100 75 6d 62 65 72 54 61 62 6c 65 01 00 12 4c 6f 63 
0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 01 
0000140 00 04 74 68 69 73 01 00 1d 4c 63 6e 2f 69 74 63 
0000160 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f 
0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e 01 00 16 
0000220 28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 
0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 01 00 13 
0000260 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 
0000300 6e 67 3b 01 00 10 4d 65 74 68 6f 64 50 61 72 61 
0000320 6d 65 74 65 72 73 01 00 0a 53 6f 75 72 63 65 46 
0000340 69 6c 65 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64
0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d 0c 00 1e 
0000400 00 1f 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64 
0000420 07 00 20 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74 
0000440 63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 
0000460 6f 57 6f 72 6c 64 01 00 10 6a 61 76 61 2f 6c 61 
0000500 6e 67 2f 4f 62 6a 65 63 74 01 00 10 6a 61 76 61 
0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d 01 00 03 6f 
0000540 75 74 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72 
0000560 69 6e 74 53 74 72 65 61 6d 3b 01 00 13 6a 61 76 
0000600 61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d 
0000620 01 00 07 70 72 69 6e 74 6c 6e 01 00 15 28 4c 6a 
0000640 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b 
0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 00 01 
0000700 00 07 00 08 00 01 00 09 00 00 00 2f 00 01 00 01 
0000720 00 00 00 05 2a b7 00 01 b1 00 00 00 02 00 0a 00 
0000740 00 00 06 00 01 00 00 00 04 00 0b 00 00 00 0c 00 
0000760 01 00 00 00 05 00 0c 00 0d 00 00 00 09 00 0e 00 
0001000 0f 00 02 00 09 00 00 00 37 00 02 00 01 00 00 00 
0001020 09 b2 00 02 12 03 b6 00 04 b1 00 00 00 02 00 0a 
0001040 00 00 00 0a 00 02 00 00 00 06 00 08 00 07 00 0b 
0001060 00 00 00 0c 00 01 00 00 00 09 00 10 00 11 00 00 
0001100 00 12 00 00 00 05 01 00 10 00 00 00 01 00 13 00 
0001120 00 00 02 00 14
```

<font style="color:rgb(77, 77, 77);">根据 JVM 规范，类文件结构如下：</font>

```java
u4 			   magic
u2             minor_version;    
u2             major_version;    
u2             constant_pool_count;    
cp_info        constant_pool[constant_pool_count-1];    
u2             access_flags;    
u2             this_class;    
u2             super_class;   
u2             interfaces_count;    
u2             interfaces[interfaces_count];   
u2             fields_count;    
field_info     fields[fields_count];   
u2             methods_count;    
method_info    methods[methods_count];    
u2             attributes_count;    
attribute_info attributes[attributes_count];

```

#### 1）魔数
u4 magic

对应字节码文件的 0~3 个字节

0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 09

ca fe ba be ：意思是 .class 文件，不同的东西有不同的魔数，比如 jpg、png 图片等！

#### <font style="color:rgb(79, 79, 79);">2）版本</font>
<font style="color:rgb(77, 77, 77);">u2 minor_version;  
</font><font style="color:rgb(77, 77, 77);">u2 major_version;  
</font><font style="color:rgb(77, 77, 77);">0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 09  
</font><font style="color:rgb(77, 77, 77);">00 00 00 34：34H（16进制） = 52（10进制），代表JDK8</font>

### <font style="color:rgb(79, 79, 79);">2、字节码指令</font>
#### <font style="color:rgb(79, 79, 79);">1）javap 工具</font>
<font style="color:rgb(77, 77, 77);">Java 中提供了 javap 工具来反编译 class 文件</font>

```java
javap -v D:Demo.class
```

#### <font style="color:rgb(79, 79, 79);">2）图解方法执行流程</font>
```java
public class Demo3_1 {    
	public static void main(String[] args) {        
		int a = 10;        
		int b = Short.MAX_VALUE + 1;        
		int c = a + b;        
		System.out.println(c);   
    } 
}
```

**<font style="color:rgb(77, 77, 77);">常量池载入运行时常量池</font>**  
<font style="color:rgb(77, 77, 77);">常量池也属于方法区，只不过这里单独提出来了</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573189113-0ca4dfdd-aa0f-42b7-887f-9670d3293898.png)

**<font style="color:rgb(77, 77, 77);">方法字节码载入方法区</font>****  
**<font style="color:rgb(77, 77, 77);">（stack=2，locals=4） 对应操作数栈有 2 个空间（每个空间 4 个字节），局部变量表中有 4 个槽位。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573211069-dd5c6076-9034-4912-9eee-256469406b91.png)

**<font style="color:rgb(77, 77, 77);">执行引擎开始执行字节码</font>**  
**<font style="color:rgb(77, 77, 77);">bipush 10</font>**

<font style="color:rgb(77, 77, 77);">将一个 byte 压入操作数栈（其长度会补齐 4 个字节），类似的指令还有</font>

+ <font style="color:rgb(77, 77, 77);">sipush 将一个 short 压入操作数栈（其长度会补齐 4 个字节）</font>
+ <font style="color:rgb(77, 77, 77);">ldc 将一个 int 压入操作数栈</font>
+ <font style="color:rgb(77, 77, 77);">ldc2_w 将一个 long 压入操作数栈（分两次压入，因为 long 是 8 个字节）</font>
+ <font style="color:rgb(77, 77, 77);">这里小的数字都是和字节码指令存在一起，超过 short 范围的数字存入了常量池</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573275607-24c5f81e-08a5-4500-ad63-0b2f2629e1fc.png)

**<font style="color:rgb(77, 77, 77);">istore 1</font>**  
<font style="color:rgb(77, 77, 77);">将操作数栈栈顶元素弹出，放入局部变量表的 slot 1 中</font>  
<font style="color:rgb(77, 77, 77);">对应代码中的 a = 10</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573290573-2a8b9b7d-e95d-426c-8095-fb252bdc5ac0.png)

ldc #3

读取运行时常量池中 #3 ，即 32768 (超过 short 最大值范围的数会被放到运行时常量池中)，将其加载到操作数栈中

注意 Short.MAX_VALUE 是 32767，所以 32768 = Short.MAX_VALUE + 1 实际是在编译期间计算好的。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573327288-c4b60c21-0ca6-4b0c-9b86-2149a8dad70b.png)

**<font style="color:rgb(77, 77, 77);">istore 2</font>**  
<font style="color:rgb(77, 77, 77);">将操作数栈中的元素弹出，放到局部变量表的 2 号位置</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573340111-ff9589e8-df3e-42d4-882a-e4d580005e6c.png)

**<font style="color:rgb(77, 77, 77);">iload1 iload2</font>**  
<font style="color:rgb(77, 77, 77);">将局部变量表中 1 号位置和 2 号位置的元素放入操作数栈中。因为只能在操作数栈中执行运算操作</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573356231-343c1fe4-5d89-4330-a469-fb3f2893e087.png)

**<font style="color:rgb(77, 77, 77);">iadd</font>**  
<font style="color:rgb(77, 77, 77);">将操作数栈中的两个元素弹出栈并相加，结果在压入操作数栈中。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573370486-7775f08e-ec24-462d-a656-6ebd50c215f7.png)

**<font style="color:rgb(77, 77, 77);">istore 3</font>**  
<font style="color:rgb(77, 77, 77);">将操作数栈中的元素弹出，放入局部变量表的3号位置。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573395383-57bb41a5-6105-44ba-9ee1-3a671d8e89f6.png)

**<font style="color:rgb(77, 77, 77);">getstatic #4</font>**  
<font style="color:rgb(77, 77, 77);">在运行时常量池中找到 #4 ，发现是一个对象，在堆内存中找到该对象，并将其引用放入操作数栈中</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573456404-10d4d53e-e79a-4c8f-9a0b-324e0f8fb7d3.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573440588-e79cfc49-f693-4da2-b29f-2ec8001e38ab.png)

**<font style="color:rgb(77, 77, 77);">iload 3</font>**  
<font style="color:rgb(77, 77, 77);">将局部变量表中 3 号位置的元素压入操作数栈中。</font>  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573467979-96b65eb8-7051-4f74-9425-b53fc18f9bbc.png)

**<font style="color:rgb(77, 77, 77);">invokevirtual #5</font>**  
<font style="color:rgb(77, 77, 77);">找到常量池 #5 项，定位到方法区 java/io/PrintStream.println:(I)V 方法</font>  
<font style="color:rgb(77, 77, 77);">生成新的栈帧（分配 locals、stack等）</font>  
<font style="color:rgb(77, 77, 77);">传递参数，执行新栈帧中的字节码</font>  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573476952-ea3fb75b-1186-4dd2-bfaa-4c4f1d36886c.png)

**<font style="color:rgb(77, 77, 77);">invokevirtual #5</font>**  
<font style="color:rgb(77, 77, 77);">找到常量池 #5 项，定位到方法区 java/io/PrintStream.println:(I)V 方法</font>  
<font style="color:rgb(77, 77, 77);">生成新的栈帧（分配 locals、stack等）</font>  
<font style="color:rgb(77, 77, 77);">传递参数，执行新栈帧中的字节码</font>  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752573489641-98a45ea6-2159-44b0-8798-3e4cf3b878e2.png)

**<font style="color:rgb(77, 77, 77);">return</font>**  
<font style="color:rgb(77, 77, 77);">完成 main 方法调用，弹出 main 栈帧，程序结束</font>

#### <font style="color:rgb(79, 79, 79);">3）通过字节码指令分析问题</font>
```java
public class Code_11_ByteCodeTest {
    public static void main(String[] args) {
        int i = 0;
        int x = 0;
        while (i < 10) {
            x = x++;
            i++;
        }
        System.out.println(x); // 0
    }
}
```

<font style="color:rgb(77, 77, 77);">为什么最终的 x 结果为 0 呢？ 通过分析字节码指令即可知晓</font>

```java
Code:
     stack=2, locals=3, args_size=1	// 操作数栈分配2个空间，局部变量表分配 3 个空间
        0: iconst_0	             // 准备一个常数 0
        1: istore_1	             // 将常数 0 放入局部变量表的 1 号槽位 i = 0
        2: iconst_0	             // 准备一个常数 0
        3: istore_2	             // 将常数 0 放入局部变量的 2 号槽位 x = 0	
        4: iload_1		         // 将局部变量表 1 号槽位的数放入操作数栈中
        5: bipush        10	     // 将数字 10 放入操作数栈中，此时操作数栈中有 2 个数
        7: if_icmpge     21	     // 比较操作数栈中的两个数，如果下面的数大于上面的数，就跳转到 21 。这里的比较是将两个数做减法。因为涉及运算操作，所以会将两个数弹出操作数栈来进行运算。运算结束后操作数栈为空
       10: iload_2		         // 将局部变量 2 号槽位的数放入操作数栈中，放入的值是 0 
       11: iinc          2, 1	 // 将局部变量 2 号槽位的数加 1 ，自增后，槽位中的值为 1 
       14: istore_2	             //将操作数栈中的数放入到局部变量表的 2 号槽位，2 号槽位的值又变为了0
       15: iinc          1, 1    // 1 号槽位的值自增 1 
       18: goto          4       // 跳转到第4条指令
       21: getstatic     #2      // Field java/lang/System.out:Ljava/io/PrintStream;
       24: iload_2
       25: invokevirtual #3      // Method java/io/PrintStream.println:(I)V
       28: return

```

#### <font style="color:rgb(79, 79, 79);">4）构造方法</font>
**<font style="color:rgb(77, 77, 77);">cinit()V</font>**

```java
public class Code_12_CinitTest {
	static int i = 10;
	static {
		i = 20;
	}
	static {
		i = 30;
	}
	public static void main(String[] args) {
		System.out.println(i); // 30
	}
}
```

<font style="color:rgb(77, 77, 77);">编译器会按从上至下的顺序，收集所有 static 静态代码块和静态成员赋值的代码，合并为一个特殊的方法 cinit()V ：</font>

```java
stack=1, locals=0, args_size=0
         0: bipush        10
         2: putstatic     #3                  // Field i:I
         5: bipush        20
         7: putstatic     #3                  // Field i:I
        10: bipush        30
        12: putstatic     #3                  // Field i:I
        15: return
```

**<font style="color:rgb(77, 77, 77);">init()V</font>**

```java
public class Code_13_InitTest {

    private String a = "s1";

    {
        b = 20;
    }

    private int b = 10;

    {
        a = "s2";
    }

    public Code_13_InitTest(String a, int b) {
        this.a = a;
        this.b = b;
    }

    public static void main(String[] args) {
        Code_13_InitTest d = new Code_13_InitTest("s3", 30);
        System.out.println(d.a);
        System.out.println(d.b);
    }

}
```

<font style="color:rgb(77, 77, 77);">编译器会按从上至下的顺序，收集所有 {} 代码块和成员变量赋值的代码，形成新的构造方法，但原始构造方法内的代码总是在后.</font>

```java
Code:
     stack=2, locals=3, args_size=3
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: aload_0
        5: ldc           #2                  // String s1
        7: putfield      #3                  // Field a:Ljava/lang/String;
       10: aload_0
       11: bipush        20
       13: putfield      #4                  // Field b:I
       16: aload_0
       17: bipush        10
       19: putfield      #4                  // Field b:I
       22: aload_0
       23: ldc           #5                  // String s2
       25: putfield      #3                  // Field a:Ljava/lang/String;
       // 原始构造方法在最后执行
       28: aload_0
       29: aload_1
       30: putfield      #3                  // Field a:Ljava/lang/String;
       33: aload_0
       34: iload_2
       35: putfield      #4                  // Field b:I
       38: return
```

#### <font style="color:rgb(79, 79, 79);">5）方法调用</font>
```java
public class Code_14_MethodTest {

    public Code_14_MethodTest() {

    }

    private void test1() {

    }

    private final void test2() {

    }

    public void test3() {

    }

    public static void test4() {

    }

    public static void main(String[] args) {
        Code_14_MethodTest obj = new Code_14_MethodTest();
        obj.test1();
        obj.test2();
        obj.test3();
        Code_14_MethodTest.test4();
    }
}
```

**不同方法在调用时，对应的虚拟机指令有所区别。**

+ 私有、构造、被 final 修饰的方法，在调用时都使用 invokespecial 指令
+ 普通成员方法在调用时，使用 invokevirtual 指令。因为编译期间无法确定该方法的内容，只有在运行期间才能确定
+ 静态方法在调用时使用 invokestatic 指令

```java
Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  //
         3: dup // 复制一份对象地址压入操作数栈中
         4: invokespecial #3                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: invokespecial #4                  // Method test1:()V
        12: aload_1
        13: invokespecial #5                  // Method test2:()V
        16: aload_1
        17: invokevirtual #6                  // Method test3:()V
        20: invokestatic  #7                  // Method test4:()V
        23: return
```

+ new 是创建【对象】，给对象分配堆内存，执行成功会将【对象引用】压入操作数栈
+ dup 是赋值操作数栈栈顶的内容，本例即为【对象引用】，为什么需要两份引用呢，一个是要配合 invokespecial 调用该对象的构造方法 “init”: ()V （会消耗掉栈顶一个引用），另一个要 配合 astore_1 赋值给局部变量
+ 终方法（ﬁnal），私有方法（private），构造方法都是由 invokespecial 指令来调用，属于静态绑定
+ 普通成员方法是由 invokevirtual 调用，属于动态绑定，即支持多态 成员方法与静态方法调用的另一个区别是，执行方法前是否需要【对象引用】

#### 6）多态原理
因为普通成员方法需要在运行时才能确定具体的内容，所以虚拟机需要调用 invokevirtual 指令

在执行 invokevirtual 指令时，经历了以下几个步骤

+ 先通过栈帧中对象的引用找到对象
+ 分析对象头，找到对象实际的 Class
+ Class 结构中有 vtable
+ 查询 vtable 找到方法的具体地址
+ 执行方法的字节码

#### <font style="color:rgb(79, 79, 79);">7）异常处理</font>
**<font style="color:rgb(77, 77, 77);">try-catch</font>**

```java
public class Code_15_TryCatchTest {
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        }catch (Exception e) {
            i = 20;
        }
    }
}
```

<font style="color:rgb(77, 77, 77);">对应字节码指令</font>

```java
Code:
     stack=1, locals=3, args_size=1
        0: iconst_0
        1: istore_1
        2: bipush        10
        4: istore_1
        5: goto          12
        8: astore_2
        9: bipush        20
       11: istore_1
       12: return
     //多出来一个异常表
     Exception table:
        from    to  target type
            2     5     8   Class java/lang/Exception
```

+ 可以看到多出来一个 Exception table 的结构，[from, to) 是前闭后开（也就是检测 2~4 行）的检测范围，一旦这个范围内的字节码执行出现异常，则通过 type 匹配异常类型，如果一致，进入 target 所指示行号
+ 8 行的字节码指令 astore_2 是将异常对象引用存入局部变量表的 2 号位置（为 e ）

**<font style="color:rgb(77, 77, 77);">多个 single-catch</font>**

```java
public class Code_16_MultipleCatchTest {
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        }catch (ArithmeticException e) {
            i = 20;
        }catch (Exception e) {
            i = 30;
        }
    }
}
```

<font style="color:rgb(77, 77, 77);">对应的字节码</font>

```java
Code:
     stack=1, locals=3, args_size=1
        0: iconst_0
        1: istore_1
        2: bipush        10
        4: istore_1
        5: goto          19
        8: astore_2
        9: bipush        20
       11: istore_1
       12: goto          19
       15: astore_2
       16: bipush        30
       18: istore_1
       19: return
     Exception table:
        from    to  target type
            2     5     8   Class java/lang/ArithmeticException
            2     5    15   Class java/lang/Exception
```

+ <font style="color:rgba(0, 0, 0, 0.75);">因为异常出现时，只能进入 Exception table 中一个分支，所以局部变量表 slot 2 位置被共用</font>

**<font style="color:rgb(77, 77, 77);">finally</font>**

```java
public class Code_17_FinallyTest {
    
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        } catch (Exception e) {
            i = 20;
        } finally {
            i = 30;
        }
    }
}
```

<font style="color:rgb(77, 77, 77);">对应字节码</font>

```java
Code:
     stack=1, locals=4, args_size=1
        0: iconst_0
        1: istore_1
        // try块
        2: bipush        10
        4: istore_1
        // try块执行完后，会执行finally    
        5: bipush        30
        7: istore_1
        8: goto          27
       // catch块     
       11: astore_2 // 异常信息放入局部变量表的2号槽位
       12: bipush        20
       14: istore_1
       // catch块执行完后，会执行finally        
       15: bipush        30
       17: istore_1
       18: goto          27
       // 出现异常，但未被 Exception 捕获，会抛出其他异常，这时也需要执行 finally 块中的代码   
       21: astore_3
       22: bipush        30
       24: istore_1
       25: aload_3
       26: athrow  // 抛出异常
       27: return
     Exception table:
        from    to  target type
            2     5    11   Class java/lang/Exception
            2     5    21   any
           11    15    21   any
```

<font style="color:rgb(77, 77, 77);">可以看到 ﬁnally 中的代码被复制了 3 份，分别放入 try 流程，catch 流程以及 catch 剩余的异常类型流程</font>  
<font style="color:rgb(77, 77, 77);">注意：虽然从字节码指令看来，每个块中都有 finally 块，但是 finally 块中的代码只会被执行一次</font>

**<font style="color:rgb(77, 77, 77);">finally 中的 return</font>**

```java
public class Code_18_FinallyReturnTest {

    public static void main(String[] args) {
        int i = Code_18_FinallyReturnTest.test();
        // 结果为 20
        System.out.println(i);
    }

    public static int test() {
        int i;
        try {
            i = 10;
            return i;
        } finally {
            i = 20;
            return i;
        }
    }
}
```

<font style="color:rgb(77, 77, 77);">对应字节码</font>

```java
Code:
     stack=1, locals=3, args_size=0
        0: bipush        10
        2: istore_0
        3: iload_0
        4: istore_1  // 暂存返回值
        5: bipush        20
        7: istore_0
        8: iload_0
        9: ireturn	// ireturn 会返回操作数栈顶的整型值 20
       // 如果出现异常，还是会执行finally 块中的内容，没有抛出异常
       10: astore_2
       11: bipush        20
       13: istore_0
       14: iload_0
       15: ireturn	// 这里没有 athrow 了，也就是如果在 finally 块中如果有返回操作的话，且 try 块中出现异常，会吞掉异常！
     Exception table:
        from    to  target type
            0     5    10   any
```

+ 由于 ﬁnally 中的 ireturn 被插入了所有可能的流程，因此返回结果肯定以ﬁnally的为准
+ 至于字节码中第 2 行，似乎没啥用，且留个伏笔，看下个例子
+ 跟上例中的 ﬁnally 相比，发现没有 athrow 了，这告诉我们：如果在 ﬁnally 中出现了 return，会吞掉异常
+ 所以不要在finally中进行返回操作

**<font style="color:rgb(77, 77, 77);">被吞掉的异常</font>**

```java
public static int test() {
      int i;
      try {
         i = 10;
         //  这里应该会抛出异常
         i = i/0;
         return i;
      } finally {
         i = 20;
         return i;
      }
   }
```

<font style="color:rgb(77, 77, 77);">会发现打印结果为 20 ，并未抛出异常</font>

**<font style="color:rgb(77, 77, 77);">finally 不带 return</font>**

```java
public static int test() {
		int i = 10;
		try {
			return i;
		} finally {
			i = 20;
		}
	}
```

<font style="color:rgb(77, 77, 77);">对应字节码</font>

```java
Code:
     stack=1, locals=3, args_size=0
        0: bipush        10
        2: istore_0 // 赋值给i 10
        3: iload_0	// 加载到操作数栈顶
        4: istore_1 // 加载到局部变量表的1号位置
        5: bipush        20
        7: istore_0 // 赋值给i 20
        8: iload_1 // 加载局部变量表1号位置的数10到操作数栈
        9: ireturn // 返回操作数栈顶元素 10
       10: astore_2
       11: bipush        20
       13: istore_0
       14: aload_2 // 加载异常
       15: athrow // 抛出异常
     Exception table:
        from    to  target type
            3     5    10   any
```

#### <font style="color:rgb(79, 79, 79);">8）Synchronized</font>
```java
public class Code_19_SyncTest {

    public static void main(String[] args) {
        Object lock = new Object();
        synchronized (lock) {
            System.out.println("ok");
        }
    }
}
```

<font style="color:rgb(77, 77, 77);">对应字节码</font>

```java
Code:
      stack=2, locals=4, args_size=1
         0: new           #2                  // class java/lang/Object
         3: dup // 复制一份栈顶，然后压入栈中。用于函数消耗
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V
         7: astore_1 // 将栈顶的对象地址方法 局部变量表中 1 中
         8: aload_1 // 加载到操作数栈
         9: dup // 复制一份，放到操作数栈，用于加锁时消耗
        10: astore_2 // 将操作数栈顶元素弹出，暂存到局部变量表的 2 号槽位。这时操作数栈中有一份对象的引用
        11: monitorenter // 加锁
        12: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        15: ldc           #4                  // String ok
        17: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        20: aload_2 // 加载对象到栈顶
        21: monitorexit // 释放锁
        22: goto          30
        // 异常情况的解决方案 释放锁！
        25: astore_3
        26: aload_2
        27: monitorexit
        28: aload_3
        29: athrow
        30: return
        // 异常表！
      Exception table:
         from    to  target type
            12    22    25   any
            25    28    25   any
```

### 3、编译期处理(语法糖)
所谓的 **语法糖** ，其实就是指 java 编译器把 .java 源码编译为 .class 字节码的过程中，自动生成和转换的一些代码，主要是为了减轻程序员的负担，算是 java 编译器给我们的一个额外福利

注意，以下代码的分析，借助了 javap 工具，idea 的反编译功能，idea 插件 jclasslib 等工具。另外， 编译器转换的结果直接就是 class 字节码，只是为了便于阅读，给出了 几乎等价 的 java 源码方式，并不是编译器还会转换出中间的 java 源码，切记。

#### <font style="color:rgb(79, 79, 79);">1）默认构造器</font>
```java
public class Candy1 {

}
```

<font style="color:rgb(77, 77, 77);">经过编译期优化后</font>

```java
public class Candy1 {
   // 这个无参构造器是java编译器帮我们加上的
   public Candy1() {
      // 即调用父类 Object 的无参构造方法，即调用 java/lang/Object." <init>":()V
      super();
   }
}
```

#### <font style="color:rgb(79, 79, 79);">2）自动拆装箱</font>
<font style="color:rgb(77, 77, 77);">基本类型和其包装类型的相互转换过程，称为拆装箱  
</font><font style="color:rgb(77, 77, 77);">在 JDK 5 以后，它们的转换可以在编译期自动完成</font>

```java
public class Candy2 {
   public static void main(String[] args) {
      Integer x = 1;
      int y = x;
   }
}
```

<font style="color:rgb(77, 77, 77);">转换过程如下</font>

```java
public class Candy2 {
   public static void main(String[] args) {
      // 基本类型赋值给包装类型，称为装箱
      Integer x = Integer.valueOf(1);
      // 包装类型赋值给基本类型，称谓拆箱
      int y = x.intValue();
   }
}
```

#### <font style="color:rgb(79, 79, 79);">3）泛型集合取值</font>
<font style="color:rgb(77, 77, 77);">泛型也是在 JDK 5 开始加入的特性，但 java 在编译泛型代码后会执行泛型擦除的动作，即泛型信息在编译为字节码之后就丢失了，实际的类型都当做了 Object 类型来处理：</font>

```java
public class Candy3 {
   public static void main(String[] args) {
      List<Integer> list = new ArrayList<>();
      list.add(10);
      Integer x = list.get(0);
   }
}
```

<font style="color:rgb(77, 77, 77);">对应字节码</font>

```java
Code:
    stack=2, locals=3, args_size=1
       0: new           #2                  // class java/util/ArrayList
       3: dup
       4: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
       7: astore_1
       8: aload_1
       9: bipush        10
      11: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      // 这里进行了泛型擦除，实际调用的是add(Objcet o)
      14: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
      19: pop
      20: aload_1
      21: iconst_0
      // 这里也进行了泛型擦除，实际调用的是get(Object o)   
      22: invokeinterface #6,  2            // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object;
// 这里进行了类型转换，将 Object 转换成了 Integer
      27: checkcast     #7                  // class java/lang/Integer
      30: astore_2
      31: return
```

<font style="color:rgb(77, 77, 77);">所以调用 get 函数取值时，有一个类型转换的操作</font>

```java
Integer x = (Integer) list.get(0);
```

<font style="color:rgb(77, 77, 77);">如果要将返回结果赋值给一个 int 类型的变量，则还有自动拆箱的操作</font>

```java
int x = (Integer) list.get(0).intValue();
```

<font style="color:rgb(77, 77, 77);">使用反射可以得到，参数的类型以及泛型类型。泛型反射代码如下：</font>

```java

    public static void main(String[] args) throws NoSuchMethodException {
        // 1. 拿到方法
        Method method = Code_20_ReflectTest.class.getMethod("test", List.class, Map.class);
        // 2. 得到泛型参数的类型信息
        Type[] types = method.getGenericParameterTypes();
        for(Type type : types) {
            // 3. 判断参数类型是否，带泛型的类型。
            if(type instanceof ParameterizedType) {
                ParameterizedType parameterizedType = (ParameterizedType) type;
                // 4. 得到原始类型
                System.out.println("原始类型 - " + parameterizedType.getRawType());
                // 5. 拿到泛型类型
                Type[] arguments = parameterizedType.getActualTypeArguments();
                for(int i = 0; i < arguments.length; i++) {
                    System.out.printf("泛型参数[%d] - %s\n", i, arguments[i]);
                }
            }
        }
    }
    public Set<Integer> test(List<String> list, Map<Integer, Object> map) {
        return null;
    }
```

<font style="color:rgb(77, 77, 77);">输出：</font>

```java
原始类型 - interface java.util.List
泛型参数[0] - class java.lang.String
原始类型 - interface java.util.Map
泛型参数[0] - class java.lang.Integer
泛型参数[1] - class java.lang.Object
```

#### <font style="color:rgb(79, 79, 79);">4）可变参数</font>
<font style="color:rgb(77, 77, 77);">可变参数也是 JDK 5 开始加入的新特性： 例如：</font>

```java
public class Candy4 {
   public static void foo(String... args) {
      // 将 args 赋值给 arr ，可以看出 String... 实际就是 String[]  
      String[] arr = args;
      System.out.println(arr.length);
   }

   public static void main(String[] args) {
      foo("hello", "world");
   }
}
```

<font style="color:rgb(77, 77, 77);">可变参数 String… args 其实是一个 String[] args ，从代码中的赋值语句中就可以看出来。 同 样 java 编译器会在编译期间将上述代码变换为：</font>

```java
public class Candy4 {
   public Candy4 {}
   public static void foo(String[] args) {
      String[] arr = args;
      System.out.println(arr.length);
   }

   public static void main(String[] args) {
      foo(new String[]);
   }
}
```

<font style="color:rgb(77, 77, 77);">注意，如果调用的是 foo() ，即未传递参数时，等价代码为 foo(new String[]{}) ，创建了一个空数组，而不是直接传递的 null .</font>

#### <font style="color:rgb(79, 79, 79);">5）foreach 循环</font>
<font style="color:rgb(77, 77, 77);">仍是 JDK 5 开始引入的语法糖，数组的循环：</font>

```java
public class Candy5 {
	public static void main(String[] args) {
        // 数组赋初值的简化写法也是一种语法糖。
		int[] arr = {1, 2, 3, 4, 5};
		for(int x : arr) {
			System.out.println(x);
		}
	}
}
```

<font style="color:rgb(77, 77, 77);">编译器会帮我们转换为</font>

```java
public class Candy5 {
    public Candy5() {}

	public static void main(String[] args) {
		int[] arr = new int[]{1, 2, 3, 4, 5};
		for(int i = 0; i < arr.length; ++i) {
			int x = arr[i];
			System.out.println(x);
		}
	}
}
```

<font style="color:rgb(77, 77, 77);">如果是集合使用 foreach</font>

```java
public class Candy5 {
   public static void main(String[] args) {
      List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
      for (Integer x : list) {
         System.out.println(x);
      }
   }
```

<font style="color:rgb(77, 77, 77);">集合要使用 foreach ，需要该集合类实现了 Iterable 接口，因为集合的遍历需要用到迭代器 Iterator.</font>

```java
public class Candy5 {
    public Candy5(){}
    
   public static void main(String[] args) {
      List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
      // 获得该集合的迭代器
      Iterator<Integer> iterator = list.iterator();
      while(iterator.hasNext()) {
         Integer x = iterator.next();
         System.out.println(x);
      }
   }
}
```

#### <font style="color:rgb(79, 79, 79);">6）switch 字符串</font>
<font style="color:rgb(77, 77, 77);">从 JDK 7 开始，switch 可以作用于字符串和枚举类，这个功能其实也是语法糖，例如：</font>

```java
public class Cnady6 {
   public static void main(String[] args) {
      String str = "hello";
      switch (str) {
         case "hello" :
            System.out.println("h");
            break;
         case "world" :
            System.out.println("w");
            break;
         default:
            break;
      }
   }
}
```

<font style="color:rgb(77, 77, 77);">在编译器中执行的操作</font>

```java
public class Candy6 {
   public Candy6() {
      
   }
   public static void main(String[] args) {
      String str = "hello";
      int x = -1;
      // 通过字符串的 hashCode + value 来判断是否匹配
      switch (str.hashCode()) {
         // hello 的 hashCode
         case 99162322 :
            // 再次比较，因为字符串的 hashCode 有可能相等
            if(str.equals("hello")) {
               x = 0;
            }
            break;
         // world 的 hashCode
         case 11331880 :
            if(str.equals("world")) {
               x = 1;
            }
            break;
         default:
            break;
      }

      // 用第二个 switch 在进行输出判断
      switch (x) {
         case 0:
            System.out.println("h");
            break;
         case 1:
            System.out.println("w");
            break;
         default:
            break;
      }
   }
}
```

<font style="color:rgb(77, 77, 77);">过程说明：</font>

<font style="color:rgba(0, 0, 0, 0.75);">在编译期间，单个的 switch 被分为了两个</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">第一个用来匹配字符串，并给 x 赋值</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">字符串的匹配用到了字符串的 hashCode ，还用到了 equals 方法</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">使用 hashCode 是为了提高比较效率，使用 equals 是防止有 hashCode 冲突（如 BM 和 C .）</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">第二个用来根据x的值来决定输出语句</font>

#### <font style="color:rgb(79, 79, 79);">7）switch 枚举</font>
```java
enum SEX {
   MALE, FEMALE;
}
public class Candy7 {
   public static void main(String[] args) {
      SEX sex = SEX.MALE;
      switch (sex) {
         case MALE:
            System.out.println("man");
            break;
         case FEMALE:
            System.out.println("woman");
            break;
         default:
            break;
      }
   }
}
```

<font style="color:rgb(77, 77, 77);">编译器中执行的代码如下</font>

```java
enum SEX {
   MALE, FEMALE;
}

public class Candy7 {
   /**     
    * 定义一个合成类（仅 jvm 使用，对我们不可见）     
    * 用来映射枚举的 ordinal 与数组元素的关系     
    * 枚举的 ordinal 表示枚举对象的序号，从 0 开始     
    * 即 MALE 的 ordinal()=0，FEMALE 的 ordinal()=1     
    */ 
   static class $MAP {
      // 数组大小即为枚举元素个数，里面存放了 case 用于比较的数字
      static int[] map = new int[2];
      static {
         // ordinal 即枚举元素对应所在的位置，MALE 为 0 ，FEMALE 为 1
         map[SEX.MALE.ordinal()] = 1;
         map[SEX.FEMALE.ordinal()] = 2;
      }
   }

   public static void main(String[] args) {
      SEX sex = SEX.MALE;
      // 将对应位置枚举元素的值赋给 x ，用于 case 操作
      int x = $MAP.map[sex.ordinal()];
      switch (x) {
         case 1:
            System.out.println("man");
            break;
         case 2:
            System.out.println("woman");
            break;
         default:
            break;
      }
   }
}
```

#### <font style="color:rgb(79, 79, 79);">8）枚举类</font>
<font style="color:rgb(77, 77, 77);">JDK 7 新增了枚举类，以前面的性别枚举为例：</font>

```java
enum SEX {
   MALE, FEMALE;
}
```

<font style="color:rgb(77, 77, 77);">转换后的代码</font>

```java
public final class Sex extends Enum<Sex> {   
   // 对应枚举类中的元素
   public static final Sex MALE;    
   public static final Sex FEMALE;    
   private static final Sex[] $VALUES;
   
    static {       
    	// 调用构造函数，传入枚举元素的值及 ordinal
    	MALE = new Sex("MALE", 0);    
        FEMALE = new Sex("FEMALE", 1);   
        $VALUES = new Sex[]{MALE, FEMALE}; 
   }
 	
   // 调用父类中的方法
    private Sex(String name, int ordinal) {     
        super(name, ordinal);    
    }
   
    public static Sex[] values() {  
        return $VALUES.clone();  
    }
    public static Sex valueOf(String name) { 
        return Enum.valueOf(Sex.class, name);  
    } 
   
}

```

#### <font style="color:rgb(79, 79, 79);">9）try-with-resources</font>
<font style="color:rgb(77, 77, 77);">JDK 7 开始新增了对需要关闭的资源处理的特殊语法，‘try-with-resources’</font>

```java
try(资源变量 = 创建资源对象) {
	
} catch() {

}
```

其中资源对象需要实现 AutoCloseable 接口，例如 InputStream 、 OutputStream 、 Connection 、 Statement 、 ResultSet 等接口都实现了 AutoCloseable ，使用 try-with- resources 可以不用写 finally 语句块，编译器会帮助生成关闭资源代码，例如：

```java
public class Candy9 { 
	public static void main(String[] args) {
		try(InputStream is = new FileInputStream("d:\\1.txt")){	
			System.out.println(is); 
		} catch (IOException e) { 
			e.printStackTrace(); 
		} 
	} 
}
```

<font style="color:rgb(77, 77, 77);">会被转换为：</font>

```java
public class Candy9 { 
    
    public Candy9() { }
   
    public static void main(String[] args) { 
        try {
            InputStream is = new FileInputStream("d:\\1.txt");
            Throwable t = null; 
            try {
                System.out.println(is); 
            } catch (Throwable e1) { 
                // t 是我们代码出现的异常 
                t = e1; 
                throw e1; 
            } finally {
                // 判断了资源不为空 
                if (is != null) { 
                    // 如果我们代码有异常
                    if (t != null) { 
                        try {
                            is.close(); 
                        } catch (Throwable e2) { 
                            // 如果 close 出现异常，作为被压制异常添加
                            t.addSuppressed(e2); 
                        } 
                    } else { 
                        // 如果我们代码没有异常，close 出现的异常就是最后 catch 块中的 e 
                        is.close(); 
                    } 
                } 
            } 
        } catch (IOException e) {
            e.printStackTrace(); 
        } 
    }
}
```

<font style="color:rgb(77, 77, 77);">为什么要设计一个 addSuppressed(Throwable e) （添加被压制异常）的方法呢？是为了防止异常信息的丢失（想想 try-with-resources 生成的 fianlly 中如果抛出了异常）：</font>

```java
public class Test6 { 
	public static void main(String[] args) { 
		try (MyResource resource = new MyResource()) { 
			int i = 1/0; 
		} catch (Exception e) { 
			e.printStackTrace(); 
		} 
	} 
}
class MyResource implements AutoCloseable { 
	public void close() throws Exception { 
		throw new Exception("close 异常"); 
	} 
}
```

<font style="color:rgb(77, 77, 77);">输出：</font>

```java
java.lang.ArithmeticException: / by zero 
	at test.Test6.main(Test6.java:7) 
	Suppressed: java.lang.Exception: close 异常 
		at test.MyResource.close(Test6.java:18) 
		at test.Test6.main(Test6.java:6)
```

#### <font style="color:rgb(79, 79, 79);">10）方法重写时的桥接方法</font>
<font style="color:rgb(77, 77, 77);">我们都知道，方法重写时对返回值分两种情况：  
</font><font style="color:rgb(77, 77, 77);">- 父子类的返回值完全一致  
</font><font style="color:rgb(77, 77, 77);">- 子类返回值可以是父类返回值的子类（比较绕口，见下面的例子）</font>

```java
class A { 
	public Number m() { 
		return 1; 
	} 
}
class B extends A { 
	@Override 
	// 子类 m 方法的返回值是 Integer 是父类 m 方法返回值 Number 的子类 	
	public Integer m() { 
		return 2; 
	} 
}
```

<font style="color:rgb(77, 77, 77);">对于子类，java 编译器会做如下处理：</font>

```java
class B extends A { 
	public Integer m() { 
		return 2; 
	}
	// 此方法才是真正重写了父类 public Number m() 方法 
	public synthetic bridge Number m() { 
		// 调用 public Integer m() 
		return m(); 
	} 
}
```

<font style="color:rgb(77, 77, 77);">其中桥接方法比较特殊，仅对 java 虚拟机可见，并且与原来的 public Integer m() 没有命名冲突，可以</font>  
<font style="color:rgb(77, 77, 77);">用下面反射代码来验证：</font>

```java
public static void main(String[] args) {
        for(Method m : B.class.getDeclaredMethods()) {
            System.out.println(m);
        }
    }
```

<font style="color:rgb(77, 77, 77);">结果：</font>

```plain
public java.lang.Integer cn.ali.jvm.test.B.m()
public java.lang.Number cn.ali.jvm.test.B.m()
```

#### <font style="color:rgb(79, 79, 79);">11）匿名内部类</font>
```java
public class Candy10 {
    public static void main(String[] args) {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("running...");
            }
        };
    }
}
```

<font style="color:rgb(77, 77, 77);">转换后的代码</font>

```java
public class Candy10 {
    public static void main(String[] args) {
        // 用额外创建的类来创建匿名内部类对象
        Runnable runnable = new Candy10$1();
    }
}

// 创建了一个额外的类，实现了 Runnable 接口
final class Candy10$1 implements Runnable {
    public Demo8$1() {}

    @Override
    public void run() {
        System.out.println("running...");
    }
}
```

<font style="color:rgb(77, 77, 77);">引用局部变量的匿名内部类，源代码</font>

```java
public class Candy11 { 
    public static void test(final int x) { 
        Runnable runnable = new Runnable() { 
            @Override 
            public void run() { 	
                System.out.println("ok:" + x); 
            } 
        }; 
    } 
}
```

<font style="color:rgb(77, 77, 77);">转换后代码：</font>

```java
// 额外生成的类 
final class Candy11$1 implements Runnable { 
    int val$x; 
    Candy11$1(int x) { 
        this.val$x = x; 
    }
    public void run() { 
        System.out.println("ok:" + this.val$x); 
    } 
}

public class Candy11 { 
    public static void test(final int x) { 
        Runnable runnable = new Candy11$1(x); 
    } 
}
```

<font style="color:rgb(77, 77, 77);">注意：这同时解释了为什么匿名内部类引用局部变量时，局部变量必须是 final 的：因为在创建 Candy11$1 对象时，将 x 的值赋值给了 Candy11$1 对象的 值后，如果不是 final 声明的 x 值发生了改变，匿名内部类则值不一致。</font>

### <font style="color:rgb(79, 79, 79);">4、类加载阶段</font>
#### <font style="color:rgb(79, 79, 79);">1）加载</font>
+ 将类的字节码载入方法区（1.8后为元空间，在本地内存中）中，内部采用 C++ 的 instanceKlass 描述 java 类，它的重要 ﬁeld 有：
+ _java_mirror 即 java 的类镜像，例如对 String 来说，它的镜像类就是 String.class，作用是把 klass 暴露给 java 使用
    - _super 即父类
    - _ﬁelds 即成员变量
    - _methods 即方法
    - <font style="color:rgba(0, 0, 0, 0.75);">_constants 即常量池</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">_class_loader 即类加载器</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">_vtable 虚方法表</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">_itable 接口方法</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">如果这个类还有父类没有加载，先加载父类</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">加载和链接可能是交替运行的</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752691531100-7b02b045-c2e3-4931-9794-be0663185b67.png)

+ instanceKlass保存在方法区。JDK 8以后，方法区位于元空间中，而元空间又位于本地内存中
+ _java_mirror则是保存在堆内存中
+ InstanceKlass和*.class(JAVA镜像类)互相保存了对方的地址
+ 类的对象在对象头中保存了*.class的地址。让对象可以通过其找到方法区中的instanceKlass，从而获取类的各种信息

**<font style="color:rgb(77, 77, 77);">注意</font>**

+ <font style="color:rgba(0, 0, 0, 0.75);">instanceKlass 这样的【元数据】是存储在方法区（1.8 后的元空间内），但 _java_mirror 是存储在堆中</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">可以通过前面介绍的 HSDB 工具查看</font>

#### <font style="color:rgb(79, 79, 79);">2）连接</font>
**<font style="color:rgb(77, 77, 77);">验证</font>**<font style="color:rgb(77, 77, 77);">  
</font><font style="color:rgb(77, 77, 77);">验证类是否符合 JVM规范，安全性检查  
</font><font style="color:rgb(77, 77, 77);">用 UE 等支持二进制的编辑器修改 HelloWorld.class 的魔数，在控制台运行  
</font>**<font style="color:rgb(77, 77, 77);">准备</font>**<font style="color:rgb(77, 77, 77);">  
</font><font style="color:rgb(77, 77, 77);">为 static 变量分配空间，设置默认值</font>

+ <font style="color:rgb(77, 77, 77);">static 变量在 JDK 7 之前存储于 instanceKlass 末尾，从 JDK 7 开始，存储于 _java_mirror 末尾</font>
+ <font style="color:rgb(77, 77, 77);">static 变量分配空间和赋值是两个步骤，分配空间在准备阶段完成，赋值在初始化阶段完成</font>
+ <font style="color:rgb(77, 77, 77);">如果 static 变量是 final 的基本类型，以及字符串常量，那么编译阶段值就确定了，赋值在准备阶段完成</font>
+ <font style="color:rgb(77, 77, 77);">如果 static 变量是 final 的，但属于引用类型，那么赋值也会在初始化阶段完成将常量池中的符号引用解析为直接引用</font>

**解析**

+ 将常量池中的符号引用（Symbolic Reference）替换为直接引用（Direct Reference）的过程。该阶段会把一些静态方法（符号引用 如main()方法）替换为执行数据所存内存的指针或句柄（直接引用），这就是所谓的 静态链接 过程（类加载期间完成）。动态链接实在程序运行期间完成的将符号引用替换为直接引用（如 实际使用的方法math.compute()）。
+ 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要可以唯一定位到目标即可。符号引用于内存布局无关，所以所引用的对象不一定需要已经加载到内存中。各种虚拟机实现的内存布局可以不同，但是接受的符号引用必须是一致的，因为符号引用的字面量形式已经明确定义在Class文件格式中。
+ 直接引用：直接引用时直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用和虚拟机实现的内存布局相关，同一个符号引用在不同虚拟机上翻译出来的直接引用一般不会相同。如果有了直接引用，那么它一定已经存在于内存中了。

#### <font style="color:rgb(79, 79, 79);">3）初始化</font>
##### <font style="color:rgb(79, 79, 79);"><cinit>()v 方法</font>
<font style="color:rgb(77, 77, 77);">初始化即调用 <cinit>()V ，虚拟机会保证这个类的『构造方法』的线程安全</font>

##### <font style="color:rgb(79, 79, 79);">发生的时机</font>
<font style="color:rgb(77, 77, 77);">概括得说，类初始化是【懒惰的】</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">main 方法所在的类，总会被首先初始化</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">首次访问这个类的静态变量或静态方法时</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">子类初始化，如果父类还没初始化，会引发</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">子类访问父类的静态变量，只会触发父类的初始化</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">Class.forName</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">new 会导致初始化</font>

<font style="color:rgb(77, 77, 77);">不会导致类初始化的情况</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">访问类的 static final 静态常量（基本类型和字符串）不会触发初始化</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">类对象.class 不会触发初始化</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">创建该类的数组不会触发初始化</font>

```java
public class Load1 {
    static {
        System.out.println("main init");
    }
    public static void main(String[] args) throws ClassNotFoundException {
        // 1. 静态常量（基本类型和字符串）不会触发初始化
        //         System.out.println(B.b);
        // 2. 类对象.class 不会触发初始化
        //         System.out.println(B.class);
        // 3. 创建该类的数组不会触发初始化
        //         System.out.println(new B[0]);
        // 4. 不会初始化类 B，但会加载 B、A
        //         ClassLoader cl = Thread.currentThread().getContextClassLoader();
        //         cl.loadClass("cn.ali.jvm.test.classload.B");
        // 5. 不会初始化类 B，但会加载 B、A
        //         ClassLoader c2 = Thread.currentThread().getContextClassLoader();
        //         Class.forName("cn.ali.jvm.test.classload.B", false, c2);


        // 1. 首次访问这个类的静态变量或静态方法时
        //         System.out.println(A.a);
        // 2. 子类初始化，如果父类还没初始化，会引发
        //         System.out.println(B.c);
        // 3. 子类访问父类静态变量，只触发父类初始化
        //         System.out.println(B.a);
        // 4. 会初始化类 B，并先初始化类 A
        //         Class.forName("cn.ali.jvm.test.classload.B");
    }

}


class A {
    static int a = 0;
    static {
        System.out.println("a init");
    }
}
class B extends A {
    final static double b = 5.0;
    static boolean c = false;
    static {
        System.out.println("b init");
    }
}
```

### 5、类加载器
类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远超类加载阶段

对于任意一个类，都必须由加载它的类加载器和这个类本身一起共同确立其在 Java 虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。这句话可以表达得更通俗一些：比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个 Class 文件，被同一个 Java 虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等！

以JDK 8为例

| **<font style="color:rgb(79, 79, 79);">名称</font>** | **<font style="color:rgb(79, 79, 79);">加载的类</font>** | **<font style="color:rgb(79, 79, 79);">说明</font>** |
| --- | --- | --- |
| <font style="color:rgb(79, 79, 79);">Bootstrap ClassLoader（启动类加载器）</font> | <font style="color:rgb(79, 79, 79);">JAVA_HOME/jre/lib</font> | <font style="color:rgb(79, 79, 79);">无法直接访问</font> |
| <font style="color:rgb(79, 79, 79);">Extension ClassLoader(拓展类加载器)</font> | <font style="color:rgb(79, 79, 79);">JAVA_HOME/jre/lib/ext</font> | <font style="color:rgb(79, 79, 79);">上级为Bootstrap，显示为null</font> |
| <font style="color:rgb(79, 79, 79);">Application ClassLoader(应用程序类加载器)</font> | <font style="color:rgb(79, 79, 79);">classpath</font> | <font style="color:rgb(79, 79, 79);">上级为Extension</font> |
| <font style="color:rgb(79, 79, 79);">自定义类加载器</font> | <font style="color:rgb(79, 79, 79);">自定义</font> | <font style="color:rgb(79, 79, 79);">上级为Application</font> |


#### 1）启动类的加载器
可通过在控制台输入指令，使得类被启动类加器加载

#### 2）扩展类的加载器
如果 classpath 和 JAVA_HOME/jre/lib/ext 下有同名类，加载时会使用拓展类加载器加载。当应用程序类加载器发现拓展类加载器已将该同名类加载过了，则不会再次加载。

#### 3）双亲委派模式
双亲委派模式，即调用类加载器ClassLoader 的 loadClass 方法时，查找类的规则。

#### <font style="color:rgb(79, 79, 79);">4）自定义类加载器</font>
**<font style="color:rgb(77, 77, 77);">使用场景</font>**

+ <font style="color:rgba(0, 0, 0, 0.75);">想加载非 classpath 随意路径中的类文件</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">通过接口来使用实现，希望解耦时，常用在框架设计</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">这些类希望予以隔离，不同应用的同名类都可以加载，不冲突，常见于 tomcat 容器</font>

**<font style="color:rgb(77, 77, 77);">步骤</font>**

+ <font style="color:rgba(0, 0, 0, 0.75);">继承 ClassLoader 父类</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">要遵从双亲委派机制，重写 ﬁndClass 方法。不是重写 loadClass 方法，否则不会走双亲委派机制。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">读取类文件的字节码</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">调用父类的 deﬁneClass 方法来加载类</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">使用者调用该类加载器的 loadClass 方法</font>

**破坏双亲委派模式**

+ 双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前——即JDK1.2面世以前的“远古”时代
    - 建议用户重写findClass()方法，在类加载器中的loadClass()方法中也会调用该方法
+ 双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷导致的
    - 如果有基础类型又要调用回用户的代码，此时也会破坏双亲委派模式
+ 双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求而导致的
    - 这里所说的“动态性”指的是一些非常“热”门的名词：代码热替换（Hot Swap）、模块热部署（Hot Deployment）等

### <font style="color:rgb(79, 79, 79);">6、运行期优化</font>
#### <font style="color:rgb(79, 79, 79);">1）即时编译</font>
**<font style="color:rgb(77, 77, 77);">分层编译</font>**<font style="color:rgb(77, 77, 77);">  
</font><font style="color:rgb(77, 77, 77);">JVM 将执行状态分成了 5 个层次：</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">0层：解释执行，用解释器将字节码翻译为机器码</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">1层：使用 C1 即时编译器编译执行（不带 proﬁling）</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">2层：使用 C1 即时编译器编译执行（带基本的profiling）</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">3层：使用 C1 即时编译器编译执行（带完全的profiling）</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">4层：使用 C2 即时编译器编译执行</font>

<font style="color:rgb(77, 77, 77);">proﬁling 是指在运行过程中收集一些程序执行状态的数据，例如【方法的调用次数】，【循环的 回边次数】等</font>

<font style="color:rgb(77, 77, 77);">即时编译器（JIT）与解释器的区别</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">解释器</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">将字节码解释为机器码，下次即使遇到相同的字节码，仍会执行重复的解释</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">是将字节码解释为针对所有平台都通用的机器码</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">即时编译器</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">将一些字节码编译为机器码，并存入 Code Cache，下次遇到相同的代码，直接执行，无需再编译</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">根据平台类型，生成平台特定的机器码</font>

对于大部分的不常用的代码，我们无需耗费时间将其编译成机器码，而是采取解释执行的方式运行；另一方面，对于仅占据小部分的热点代码，我们则可以将其编译成机器码，以达到理想的运行速度。 执行效率上简单比较一下 Interpreter < C1 < C2，总的目标是发现热点代码（hotspot名称的由 来），并优化这些热点代码。

**逃逸分析**

逃逸分析（Escape Analysis）简单来讲就是，Java Hotspot 虚拟机可以分析新创建对象的使用范围，并决定是否在 Java 堆上分配内存的一项技术

逃逸分析的 JVM 参数如下：

开启逃逸分析：-XX:+DoEscapeAnalysis

关闭逃逸分析：-XX:-DoEscapeAnalysis

显示分析结果：-XX:+PrintEscapeAnalysis

<font style="color:rgb(77, 77, 77);">逃逸分析技术在 Java SE 6u23+ 开始支持，并默认设置为启用状态，可以不用额外加这个参数</font>

<font style="color:rgb(77, 77, 77);">对象逃逸状态</font>

<font style="color:rgb(77, 77, 77);">全局逃逸（GlobalEscape）</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">即一个对象的作用范围逃出了当前方法或者当前线程，有以下几种场景：</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">对象是一个静态变量</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">对象是一个已经发生逃逸的对象</font>
    - <font style="color:rgba(0, 0, 0, 0.75);">对象作为当前方法的返回值</font>

<font style="color:rgb(77, 77, 77);">参数逃逸（ArgEscape）</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">即一个对象被作为方法参数传递或者被参数引用，但在调用过程中不会发生全局逃逸，这个状态是通过被调方法的字节码确定的</font>

<font style="color:rgb(77, 77, 77);">没有逃逸</font>

+ <font style="color:rgba(0, 0, 0, 0.75);">即方法中的对象没有发生逃逸</font>

**逃逸分析优化**

针对上面第三点，当一个对象没有逃逸时，可以得到以下几个虚拟机的优化

**锁消除**

我们知道线程同步锁是非常牺牲性能的，当编译器确定当前对象只有当前线程使用，那么就会移除该对象的同步锁

例如，StringBuffer 和 Vector 都是用 synchronized 修饰线程安全的，但大部分情况下，它们都只是在当前线程中用到，这样编译器就会优化移除掉这些锁操作

锁消除的 JVM 参数如下：

+ 开启锁消除：-XX:+EliminateLocks
+ 关闭锁消除：-XX:-EliminateLocks

锁消除在 JDK8 中都是默认开启的，并且锁消除都要建立在逃逸分析的基础上

**<font style="color:rgb(77, 77, 77);">标量替换</font>**  
<font style="color:rgb(77, 77, 77);">首先要明白标量和聚合量，基础类型和对象的引用可以理解为标量，它们不能被进一步分解。而能被进一步分解的量就是聚合量，比如：对象</font>  
<font style="color:rgb(77, 77, 77);">对象是聚合量，它又可以被进一步分解成标量，将其成员变量分解为分散的变量，这就叫做标量替换。</font>

<font style="color:rgb(77, 77, 77);">这样，如果一个对象没有发生逃逸，那压根就不用创建它，只会在栈或者寄存器上创建它用到的成员标量，节省了内存空间，也提升了应用程序性能</font>

<font style="color:rgb(77, 77, 77);">标量替换的 JVM 参数如下：</font>

+ <font style="color:rgb(77, 77, 77);">开启标量替换：-XX:+EliminateAllocations</font>
+ <font style="color:rgb(77, 77, 77);">关闭标量替换：-XX:-EliminateAllocations</font>
+ <font style="color:rgb(77, 77, 77);">显示标量替换详情：-XX:+PrintEliminateAllocations</font>

<font style="color:rgb(77, 77, 77);">标量替换同样在 JDK8 中都是默认开启的，并且都要建立在逃逸分析的基础上</font>

**<font style="color:rgb(77, 77, 77);">栈上分配</font>**<font style="color:rgb(77, 77, 77);">  
</font><font style="color:rgb(77, 77, 77);">当对象没有发生逃逸时，该对象就可以通过标量替换分解成成员标量分配在栈内存中，和方法的生命周期一致，随着栈帧出栈时销毁，减少了 GC 压力，提高了应用程序性能</font>

**<font style="color:rgb(77, 77, 77);">方法内联</font>**

**<font style="color:rgb(77, 77, 77);">内联函数  
</font>**<font style="color:rgb(77, 77, 77);">内联函数就是在程序编译时，编译器将程序中出现的内联函数的调用表达式用内联函数的函数体来直接进行替换</font>

**<font style="color:rgb(77, 77, 77);">JVM内联函数</font>**<font style="color:rgb(77, 77, 77);">  
</font><font style="color:rgb(77, 77, 77);">C++ 是否为内联函数由自己决定，Java 由编译器决定。Java 不支持直接声明为内联函数的，如果想让他内联，你只能够向编译器提出请求: 关键字 final 修饰 用来指明那个函数是希望被 JVM 内联的，如</font>

```java
public final void doSomething() {  
    // to do something  
}
```

<font style="color:rgb(77, 77, 77);">总的来说，一般的函数都不会被当做内联函数，只有声明了final后，编译器才会考虑是不是要把你的函数变成内联函数</font>

<font style="color:rgb(77, 77, 77);">JVM内建有许多运行时优化。首先短方法更利于JVM推断。流程更明显，作用域更短，副作用也更明显。如果是长方法JVM可能直接就跪了。</font>

<font style="color:rgb(77, 77, 77);">第二个原因则更重要：方法内联</font>

<font style="color:rgb(77, 77, 77);">如果JVM监测到一些小方法被频繁的执行，它会把方法的调用替换成方法体本身，如：</font>

```java
private int add4(int x1, int x2, int x3, int x4) { 
    //这里调用了add2方法
    return add2(x1, x2) + add2(x3, x4);  
}  

private int add2(int x1, int x2) {  
    return x1 + x2;  
}
```

<font style="color:rgb(77, 77, 77);">方法调用被替换后</font>

```java
private int add4(int x1, int x2, int x3, int x4) {  
    //被替换为了方法本身
    return x1 + x2 + x3 + x4;  
}

```

#### <font style="color:rgb(79, 79, 79);">2）反射优化</font>
```java
public class Reflect1 {
    public static void foo() {
        System.out.println("foo...");
    }

    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Method foo = Demo3.class.getMethod("foo");
        for(int i = 0; i<=16; i++) {
            foo.invoke(null);
        }
    }
}

```

<font style="color:rgb(77, 77, 77);">foo.invoke 前面 0 ~ 15 次调用使用的是 MethodAccessor 的 NativeMethodAccessorImpl 实现</font>  
<font style="color:rgb(77, 77, 77);">invoke 方法源码</font>

```java
@CallerSensitive
public Object invoke(Object obj, Object... args)
    throws IllegalAccessException, IllegalArgumentException,InvocationTargetException{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, obj, modifiers);
        }
    }
    //MethodAccessor是一个接口，有3个实现类，其中有一个是抽象类
    MethodAccessor ma = methodAccessor;             // read volatile
    if (ma == null) {
        ma = acquireMethodAccessor();
    }
    return ma.invoke(obj, args);
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752828156810-f5706e78-f3b8-41d1-90ff-1a54c77fb4c2.png)

<font style="color:rgb(77, 77, 77);">会由 DelegatingMehodAccessorImpl 去调用 NativeMethodAccessorImpl</font>  
<font style="color:rgb(77, 77, 77);">NativeMethodAccessorImpl 源码</font>

```java
class NativeMethodAccessorImpl extends MethodAccessorImpl {
    private final Method method;
    private DelegatingMethodAccessorImpl parent;
    private int numInvocations;

    NativeMethodAccessorImpl(Method var1) {
        this.method = var1;
    }

    //每次进行反射调用，会让numInvocation与ReflectionFactory.inflationThreshold的值（15）进行比较，并使使得numInvocation的值加一
    //如果numInvocation>ReflectionFactory.inflationThreshold，则会调用本地方法invoke0方法
    public Object invoke(Object var1, Object[] var2) throws IllegalArgumentException, InvocationTargetException {
        if (++this.numInvocations > ReflectionFactory.inflationThreshold() && !ReflectUtil.isVMAnonymousClass(this.method.getDeclaringClass())) {
            MethodAccessorImpl var3 = (MethodAccessorImpl)(new MethodAccessorGenerator()).generateMethod(this.method.getDeclaringClass(), this.method.getName(), this.method.getParameterTypes(), this.method.getReturnType(), this.method.getExceptionTypes(), this.method.getModifiers());
            this.parent.setDelegate(var3);
        }

        return invoke0(this.method, var1, var2);
    }

    void setParent(DelegatingMethodAccessorImpl var1) {
        this.parent = var1;
    }

    private static native Object invoke0(Method var0, Object var1, Object[] var2);
}
```

```plain
//ReflectionFactory.inflationThreshold()方法的返回值
private static int inflationThreshold = 15;
```

+ 一开始if条件不满足，就会调用本地方法 invoke0
+ 随着 numInvocation 的增大，当它大于 ReflectionFactory.inflationThreshold 的值 16 时，就会本地方法访问器替换为一个运行时动态生成的访问器，来提高效率
    - 这时会从反射调用变为正常调用，即直接调用 Reflect1.foo()

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1752828206714-993163a1-03fe-4f45-87c5-09a0287d4648.png)

