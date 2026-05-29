### 如何保证变量的可见性？
+ **最轻量级**是使用 `**volatile**`** 关键字**，它通过底层插入**内存屏障**，强制触发 CPU 缓存一致性协议，使修改立刻落盘主内存并令其他核心缓存失效；
+ **锁机制**可以使用 `**synchronized**`** 块** 或 `**ReentrantLock**`，它们利用了 JMM 规定在‘进入同步块清空缓存（工作区的缓存）、退出同步块强制刷回主内存’的锁协议来保卫可见性；
+ **开箱即用**则推荐 JUC 包下的 **Atomic 原子类**（如 `AtomicInteger`）或 **并发容器**，其内部利用 `volatile` 字段配合底层硬件级 **CAS 指令**，实现了无锁高并发下的可见性安全。”



### 如何禁止指令重排序？
 在 Java 中，禁止指令重排序主要依靠给变量加上 `volatile` 关键字，其底层物理本质是通过编译器在字节码中插入‘内存屏障（Memory Barrier）’，从而死锁了 CPU 硬件和编译器的优化重排路径，确保代码按物理先后顺序死板执行。



### volatile 可以保证原子性么？
`volatile` 关键字能保证变量的可见性，但不能保证对变量的操作是原子性的。



### 什么是悲观锁？
 悲观锁是一种‘先加锁、再访问’的排他性防御策略，它认为并发冲突必然发生，因此在线程修改数据前，会强制调用操作系统的互斥锁（如 `synchronized` 或 `ReentrantLock`）将资源完全死锁，让其他竞争线程全部在门外挂起排队，其底层代价是伴随着极其沉重的 CPU 上下文切换开销。  



+ **底层的物理本质**：在 Java 中，当你敲下 `synchronized` 这种悲观锁时，JVM 会在后台调用操作系统的底层指令（在 Linux 下是 `**pthread_mutex_lock**`）。
+ **代价所在**：一旦锁被霸占，其他没抢到锁的线程会被操作系统强行从“运行状态（Running）”切换到“阻塞状态（Blocked）”，放进等待队列。当锁释放时，再把它们唤醒。这种**线程状态的频繁切换，需要操作系统在内核态与用户态之间来回倒腾 CPU 寄存器和计数器（上下文切换）**，在高并发下会导致系统非常沉重。



### 什么是乐观锁？
 乐观锁是一种‘先不加锁、直接尝试更新’的非阻塞性冲突检测策略，它乐观地认为并发冲突很少发生，因此在读取数据时不加任何锁，而是在数据最终‘落盘落库’的瞬间，利用 CAS 硬件指令或版本号（Version）进行冲突检测，若中途被别人偷跑修改则直接宣告失败或进行人肉重试（自旋），其底层彻底避免了操作系统的互斥锁开销。  



+ **底层的物理本质（CAS机制）**：在 Java 中，乐观锁最核心的底层支柱是 **CAS（Compare And Swap，比较并交换）**。它是一个由 CPU 硬件芯片直接支持的**原子指令**（在 Intel CPU 上对应 `**cmpxchg**`** 指令**）。它在修改数据时，会拿着你手里的旧预期值（A）去和内存里的当前值（B）对齐，如果 $A == B$，说明中途没人动过，瞬间把新值（C）写入；如果 $A \neq B$，说明有人插队了，直接拒绝写入。
+ **代价所在（自旋空转）**：乐观锁虽然彻底免去了操作系统**线程上下文切换**的沉重代价，但如果在高并发、写多读少的极端“踩踏”场景下，线程由于不断更新失败，会陷入死循环不断去尝试（**自旋**），这会**疯狂压榨并白白耗尽 CPU 的算力**。因此，乐观锁非常适合**读多写少**的轻量级并发场景。



### 如何实现乐观锁？
乐观锁一般会使用**版本号机**制或 **CAS 算法**实现，CAS 算法相对来说更多一些，这里需要格外注意。

**版本号机制：**

一般是在数据表中加上一个数据版本号 `version` 字段，表示数据被修改的次数。当数据被修改时，`version` 值会加一。当线程 A 要更新数据值时，在读取数据的同时也会读取 `version` 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的 `version` 值相等时才更新，否则重试更新操作，直到更新成功。

**CAS 算法：**

CAS 的全称是 **Compare And Swap（比较与交换）** ，用于实现乐观锁，被广泛应用于各大框架中。CAS 的思想很简单，就是用一个预期值和要更新的变量值进行比较，两值相等才会进行更新。

CAS 是一个原子操作，底层依赖于一条 CPU 的原子指令。

> **原子操作** 即最小不可拆分的操作，也就是说操作一旦开始，就不能被打断，直到操作完成。
>

CAS 涉及到三个操作数：

+ **V**：要更新的变量值(Var)
+ **E**：预期值(Expected)
+ **N**：拟写入的新值(New)

当且仅当 V 的值等于 E 时，CAS 通过原子方式用新值 N 来更新 V 的值。如果不等，说明已经有其它线程更新了 V，则当前线程放弃更新。

### Java 中 CAS 是如何实现的？
在 Java 中，实现 CAS（Compare-And-Swap, 比较并交换）操作的一个关键类是`Unsafe`。

`Unsafe`类位于`sun.misc`包下，是一个提供低级别、不安全操作的类。由于其强大的功能和潜在的危险性，它通常用于 JVM 内部或一些需要极高性能和底层访问的库中，而不推荐普通开发者在应用程序中使用。

`sun.misc`包下的`Unsafe`类提供了`compareAndSwapObject`、`compareAndSwapInt`、`compareAndSwapLong`方法来实现的对`Object`、`int`、`long`类型的 CAS 操作：

```java
/**
 * 以原子方式更新对象字段的值。
 *
 * @param o        要操作的对象
 * @param offset   对象字段的内存偏移量
 * @param expected 期望的旧值
 * @param x        要设置的新值
 * @return 如果值被成功更新，则返回 true；否则返回 false
 */
boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);

/**
 * 以原子方式更新 int 类型的对象字段的值。
 */
boolean compareAndSwapInt(Object o, long offset, int expected, int x);

/**
 * 以原子方式更新 long 类型的对象字段的值。
 */
boolean compareAndSwapLong(Object o, long offset, long expected, long x);
```

`Unsafe`类中的 CAS 方法是`native`方法。`native`关键字表明这些方法是用本地代码（通常是 C 或 C++）实现的，而不是用 Java 实现的。这些方法直接调用底层的硬件指令来实现原子操作。也就是说，Java 语言并没有直接用 Java 实现 CAS，而是通过 C++ 内联汇编的形式实现的（通过 JNI 调用）。因此，CAS 的具体实现与操作系统以及 CPU 密切相关。

由于 CAS 操作可能会因为并发冲突而失败，因此通常会与`while`循环搭配使用，在失败后不断重试，直到操作成功。这就是 **自旋锁机制** 。



### CAS 算法存在哪些问题？


####  经典 ABA 问题：瞒天过海
+ **物理画面**：线程 1 准备用 CAS 把变量从 `A` 改成 `X`。就在它刚拿到 `A` 还没动手的一瞬间，线程 2 突然插队，把变量从 `A` 变成了 `B`，紧接着又把 `B` 变回了 `A`。当线程 1 终于执行 CAS 指令时，低头一看内存里确实是 `A`（$A == A$），直接宣告成功落盘。
+ **灾难隐患**：虽然数值没变，但**该变量绑定的物理状态或者引用对象的内部属性可能已经面目全非了**（比如链表中的头节点被换过，会导致整个拓扑结构断裂）。
+ **官方解法**：在 Java 中，使用 `**AtomicStampedReference**`（加版本号/时间戳）或 `**AtomicMarkableReference**`（加布尔标记位）。它不仅比对‘值’，还强制比对‘版本号’（从 `A,1` 变成 `B,2` 再变成 `A,3`），版本号对不上，直接物理拒绝。

#### 自旋时间过长：疯狂空转
+ **物理画面**：乐观锁在冲突时会选择不挂起线程，而是原地死循环人肉重试（自旋）。如果高并发下有上百个线程同时抢写同一个变量，每次只有一个幸运儿成功，剩下 99 个线程会全部在 CPU 核心里疯狂打转。
+ **灾难隐患**：这会**直接把 CPU 的利用率瞬间飙升到 100%**，产生恐怖的功耗和热量，却不干任何有价值的业务，严重拖垮整台服务器的吞吐量。
+ **官方解法**：
    - 引入**自适应自旋锁**（JVM 在后台根据历史成功率动态调整死循环次数，超过限制就强行降级为悲观锁挂起）。
    - 使用 `**LongAdder**` 替代 `AtomicLong`（利用**分段锁/热点分离**的思想，把一个大变量拆成一个 `Cell[]` 数组，让多个线程分散去改不同的格子里，最后求和，彻底分流高并发踩踏）。

#### 只能保证单个变量的原子操作：孤军奋战
+ **物理画面**：CPU 的底层指令（如 `cmpxchg`）在物理芯片上一次只能锁死一个内存地址。如果你想同时修改 `name` 和 `age` 两个变量，并且要求它们要么同时成功要么同时失败，普通的 `AtomicInteger` 根本无能为力。
+ **官方解法**：在 Java 中，如果你必须用 CAS 绑死多个变量，需要把这些变量统一封装进一个正牌的 Java 对象（Object）中，然后使用 `**AtomicReference**`（原子引用类）来直接对整个对象的物理内存地址进行一键 CAS 替换。

### synchronized 是什么？有什么用？
`synchronized` 是 Java 中由 JVM 虚拟机原生实现的互斥锁，也是最正统的悲观锁和同步机制；它核心用来解决多线程并发下的‘原子性、可见性和有序性’灾难，通过死锁共享资源，确保同一时刻只有一个线程能够踏入受保护的代码块，从而死守线程安全。 

### 如何使用 synchronized？
`synchronized` 关键字的使用方式主要有下面 3 种：

1. 修饰实例方法
2. 修饰静态方法
3. 修饰代码块

**1、修饰实例方法** （锁当前对象实例）

给当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁** 。

```java
synchronized void method() {
    //业务代码
}
```

**2、修饰静态方法** （锁当前类）

给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。

这是因为静态成员不属于任何一个实例对象，归整个类所有，不依赖于类的特定实例，被类的所有实例共享。

```java
synchronized static void method() {
    //业务代码
}
```

静态 `synchronized` 方法和非静态 `synchronized` 方法之间的调用互斥么？不互斥！如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁。

**3、修饰代码块** （锁指定对象/类）

对括号里指定的对象/类加锁：

+ `synchronized(object)` 表示进入同步代码块前要获得 **给定对象的锁**。
+ `synchronized(类.class)` 表示进入同步代码块前要获得 **给定 Class 的锁**

```java
synchronized(this) {
    //业务代码
}
```

**总结：**

+ `synchronized` 关键字加到 `static` 静态方法和 `synchronized(class)` 代码块上都是是给 Class 类上锁；
+ `synchronized` 关键字加到实例方法上是给对象实例上锁；
+ 尽量不要使用 `synchronized(String a)` 因为 JVM 中，字符串常量池具有缓存功能。

### 构造方法可以用 synchronized 修饰么？
构造方法不能使用 synchronized 关键字修饰。不过，可以在构造方法内部使用 synchronized 代码块。

另外，构造方法本身是线程安全的，但如果在构造方法中涉及到共享资源的操作，就需要采取适当的同步措施来保证整个构造过程的线程安全。

### ⭐️synchronized 底层原理了解吗？
`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取而代之的是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。

**不过，两者的本质都是对对象监视器 monitor 的获取。**

### JDK1.6 之后的 synchronized 底层做了哪些优化？锁升级原理了解吗？
在 Java 6 之后， `synchronized` 引入了大量的优化如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多（JDK18 中，偏向锁已经被彻底废弃，前面已经提到过了）。

锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

### synchronized 的偏向锁为什么被废弃了？
在 JDK15 中，偏向锁被默认关闭（仍然可以使用 `-XX:+UseBiasedLocking` 启用偏向锁），在 JDK18 中，偏向锁已经被彻底废弃（无法通过命令行打开）。

在官方声明中，主要原因有两个方面：

+ **性能收益不明显：**

偏向锁是 HotSpot 虚拟机的一项优化技术，可以提升单线程对同步代码块的访问性能。

受益于偏向锁的应用程序通常使用了早期的 Java 集合 API，例如 HashTable、Vector，在这些集合类中通过 synchronized 来控制同步，这样在单线程频繁访问时，通过偏向锁会减少同步开销。

随着 JDK 的发展，出现了 ConcurrentHashMap 高性能的集合类，在集合类内部进行了许多性能优化，此时偏向锁带来的性能收益就不明显了。

偏向锁仅仅在单线程访问同步代码块的场景中可以获得性能收益。

如果存在多线程竞争，就需要 **撤销偏向锁** ，这个操作的性能开销是比较昂贵的。偏向锁的撤销需要等待进入到全局安全点（safe point），该状态下所有线程都是暂停的，此时去检查线程状态并进行偏向锁的撤销。

+ **JVM 内部代码维护成本太高：**

偏向锁将许多复杂代码引入到同步子系统，并且对其他的 HotSpot 组件也具有侵入性。这种复杂性为理解代码、系统重构带来了困难，因此， OpenJDK 官方希望禁用、废弃并删除偏向锁。

### ⭐️synchronized 和 volatile 有什么区别？
`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

+ `volatile` 关键字是线程同步的轻量级实现，所以 `volatile`性能肯定比`synchronized`关键字要好 。但是 `volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块 。
+ `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。
+ `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。

### ReentrantLock 是什么？
`ReentrantLock` 实现了 `Lock` 接口，是一个可重入且独占式的锁，和 `synchronized` 关键字类似。不过，`**ReentrantLock**`** 更灵活、更强大，增加了轮询、超时、中断、公平锁和非公平锁等高级功能。**

```java
public class ReentrantLock implements Lock, java.io.Serializable {}
```

`ReentrantLock` 默认使用非公平锁，也可以通过构造器来显式的指定使用公平锁。

```java
// 传入一个 boolean 值，true 时为公平锁，false 时为非公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

### 公平锁和非公平锁有什么区别？
+ **公平锁** : 锁被释放之后，先申请的线程先得到锁。性能较差一些，因为公平锁为了保证时间上的绝对顺序，上下文切换更频繁。
+ **非公平锁**：锁被释放之后，后申请的线程可能会先获取到锁，是随机或者按照其他优先级排序的。性能更好，但可能会导致某些线程永远无法获取到锁。

### ⭐️synchronized 和 ReentrantLock 有什么区别？
**两者都是可重入锁：**

**可重入锁** 也叫递归锁，指的是线程可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果是不可重入锁的话，就会造成死锁。

JDK 提供的所有现成的 `Lock` 实现类，包括 `synchronized` 关键字锁都是可重入的。

在下面的代码中，`method1()` 和 `method2()`都被 `synchronized` 关键字修饰，`method1()`调用了`method2()`。

```java
public class SynchronizedDemo {
    public synchronized void method1() {
        System.out.println("方法1");
        method2();
    }

    public synchronized void method2() {
        System.out.println("方法2");
    }
}
```

由于 `synchronized`锁是可重入的，同一个线程在调用`method1()` 时可以直接获得当前对象的锁，执行 `method2()` 的时候可以再次获取这个对象的锁，不会产生死锁问题。假如`synchronized`是不可重入锁的话，由于该对象的锁已被当前线程所持有且无法释放，这就导致线程在执行 `method2()`时获取锁失败，会出现死锁问题。

**synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API：**

`synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。

`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 `lock()` 和 `unlock()` 方法配合 `try/finally` 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

**ReentrantLock 比 synchronized 增加了一些高级功能：**

相比`synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：

+ **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说当前线程在等待获取锁的过程中，如果其他线程中断当前线程「 `interrupt()` 」，当前线程就会抛出 `InterruptedException` 异常，可以捕捉该异常进行相应处理。
+ **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来指定是否是公平的。
+ **通知机制更强大**：`ReentrantLock` 通过绑定多个 `Condition` 对象，可以实现分组唤醒和选择性通知。这解决了 `synchronized` 只能随机唤醒或全部唤醒的效率问题，为复杂的线程协作场景提供了强大的支持。
+ **支持超时** ：`ReentrantLock` 提供了 `tryLock(timeout)` 的方法，可以指定等待获取锁的最长等待时间，如果超过了等待时间，就会获取锁失败，不会一直等待。



关于 `Condition`接口的补充：

> `Condition`是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个`Lock`对象中可以创建多个`Condition`实例（即对象监视器），**线程对象可以注册在指定的**`**Condition**`**中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用**`**notify()/notifyAll()**`**方法进行通知时，被通知的线程是由 JVM 选择的，用**`**ReentrantLock**`**类结合**`**Condition**`**实例可以实现“选择性通知”** ，这个功能非常重要，而且是 `Condition` 接口默认提供的。而`synchronized`关键字就相当于整个 `Lock` 对象中只有一个`Condition`实例，所有的线程都注册在它一个身上。如果执行`notifyAll()`方法的话就会通知所有处于等待状态的线程，这样会造成很大的效率问题。而`Condition`实例的`signalAll()`方法，只会唤醒注册在该`Condition`实例中的所有等待线程。
>

关于 **等待可中断** 的补充：

> `lockInterruptibly()` 会让获取锁的线程在阻塞等待的过程中可以响应中断，即当前线程在获取锁的时候，发现锁被其他线程持有，就会阻塞等待。
>
> 在阻塞等待的过程中，如果其他线程中断当前线程 `interrupt()` ，就会抛出 `InterruptedException` 异常，可以捕获该异常，做一些处理操作。
>
> 为了更好理解这个方法，借用 Stack Overflow 上的一个案例，可以更好地理解 `lockInterruptibly()` 可以响应中断：
>
> 输出：
>

```java
public class MyRentrantlock {
    Thread t = new Thread() {
        @Override
        public void run() {
            ReentrantLock r = new ReentrantLock();
            // 1.1、第一次尝试获取锁，可以获取成功
            r.lock();

            // 1.2、此时锁的重入次数为 1
            System.out.println("lock() : lock count :" + r.getHoldCount());

            // 2、中断当前线程，通过 Thread.currentThread().isInterrupted() 可以看到当前线程的中断状态为 true
            interrupt();
            System.out.println("Current thread is intrupted");

            // 3.1、尝试获取锁，可以成功获取
            r.tryLock();
            // 3.2、此时锁的重入次数为 2
            System.out.println("tryLock() on intrupted thread lock count :" + r.getHoldCount());
            try {
                // 4、打印线程的中断状态为 true，那么调用 lockInterruptibly() 方法就会抛出 InterruptedException 异常
                System.out.println("Current Thread isInterrupted:" + Thread.currentThread().isInterrupted());
                r.lockInterruptibly();
                System.out.println("lockInterruptibly() --NOt executable statement" + r.getHoldCount());
            } catch (InterruptedException e) {
                r.lock();
                System.out.println("Error");
            } finally {
                r.unlock();
            }

            // 5、打印锁的重入次数，可以发现 lockInterruptibly() 方法并没有成功获取到锁
            System.out.println("lockInterruptibly() not able to Acqurie lock: lock count :" + r.getHoldCount());

            r.unlock();
            System.out.println("lock count :" + r.getHoldCount());
            r.unlock();
            System.out.println("lock count :" + r.getHoldCount());
        }
    };
    public static void main(String str[]) {
        MyRentrantlock m = new MyRentrantlock();
        m.t.start();
    }
}
```

```plain
lock() : lock count :1
Current thread is intrupted
tryLock() on intrupted thread lock count :2
Current Thread isInterrupted:true
Error
lockInterruptibly() not able to Acqurie lock: lock count :2
lock count :1
lock count :0
```

关于 **支持超时** 的补充：

> **为什么需要 **`**tryLock(timeout)**`** 这个功能呢？**
>
> `tryLock(timeout)` 方法尝试在指定的超时时间内获取锁。如果成功获取锁，则返回 `true`；如果在锁可用之前超时，则返回 `false`。此功能在以下几种场景中非常有用：
>
> + **防止死锁：** 在复杂的锁场景中，`tryLock(timeout)` 可以通过允许线程在合理的时间内放弃并重试来帮助防止死锁。
> + **提高响应速度：** 防止线程无限期阻塞。
> + **处理时间敏感的操作：** 对于具有严格时间限制的操作，`tryLock(timeout)` 允许线程在无法及时获取锁时继续执行替代操作。
>



### 可中断锁和不可中断锁有什么区别？
 两者的核心区别在于‘面对长时间等锁时，能否接受外界信号主动放弃排队’：不可中断锁（如 `synchronized`）一旦抢锁失败，只能在内核态死等，期间对任何中断信号免疫，极易在发生死锁时导致线程永久卡死；而可中断锁（如 `ReentrantLock` 的 `lockInterruptibly()`）在等锁期间允许其他线程向其发送中断信号，从而让其立刻清醒、放弃排队并抛出异常，提供了高并发下的‘逃生通道’。  

| 维度 | 可中断锁 (Interruptible Lock) | 不可中断锁 (Uninterruptible Lock) |
| --- | --- | --- |
| 是否响应interrupt | 是，收到中断信号后会立即抛出 `InterruptedException`<br/> 并退出等待 | 否，即使收到中断信号也会继续等待，直到成功获取锁 |
| 典型实现方式 | ReentrantLock 的 `lockInterruptibly()`<br/> 方法 ReentrantLock 的 `tryLock(long timeout, TimeUnit)` | synchronized 关键字 ReentrantLock 的普通 `lock()`<br/> 方法 |
| 线程被中断后的行为 | 抛出异常，线程可以立即做清理、退出、切换逻辑等操作 | 中断标志位会被设置，但线程仍然阻塞在锁等待上 |
| 适用场景 | 需要响应取消操作、避免死锁风险、任务可取消的场景（如线程池任务） | 对响应中断不敏感、逻辑简单、对性能要求极高的场景 |
| 缺点 | 代码需要处理 InterruptedException，稍复杂 | 线程可能永久阻塞（如果持有锁的线程死循环/长时间不释放） |


### ReentrantReadWriteLock 是什么？
`ReentrantReadWriteLock` 实现了 `ReadWriteLock` ，是一个可重入的读写锁，既可以保证多个线程同时读的效率，同时又可以保证有写入操作时的线程安全。

+ 一般锁进行并发控制的规则：读读互斥、读写互斥、写写互斥。
+ 读写锁进行并发控制的规则：读读不互斥、读写互斥、写写互斥（只有读读不互斥）。

`ReentrantReadWriteLock` 其实是两把锁，一把是 `WriteLock` (写锁)，一把是 `ReadLock`（读锁） 。读锁是共享锁，写锁是独占锁。读锁可以被同时读，可以同时被多个线程持有，而写锁最多只能同时被一个线程持有。

`ReentrantReadWriteLock` 也支持公平锁和非公平锁，默认使用非公平锁，可以通过构造器来显式地指定。

```java
// 传入一个 boolean 值，true 时为公平锁，false 时为非公平锁
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

### ReentrantReadWriteLock 适合什么场景？
由于 `ReentrantReadWriteLock` 既可以保证多个线程同时读的效率，同时又可以保证有写入操作时的线程安全。因此，在读多写少的情况下，使用 `ReentrantReadWriteLock` 能够明显提升系统性能。

### 共享锁和独占锁有什么区别？
+ **共享锁**：一把锁可以被多个线程同时获得。
+ **独占锁**：一把锁只能被一个线程获得。

### 线程持有读锁还能获取写锁吗？
+ **读锁 → 写锁**：不允许，会死锁/永久阻塞（JDK 故意设计）
+ **写锁 → 读锁**：允许，且推荐（锁降级）

### 读锁为什么不能升级为写锁？
写锁可以降级为读锁，但是读锁却不能升级为写锁。这是因为读锁升级为写锁会引起线程的争夺，毕竟写锁属于是独占锁，这样的话，会影响性能。

另外，还可能会有死锁问题发生。举个例子：假设两个线程的读锁都想升级写锁，则需要对方都释放自己锁，而双方都不释放，就会产生死锁。

### StampedLock 是什么？
`StampedLock` 是 JDK 1.8 引入的性能更好的读写锁，<font style="background-color:#FBDE28;">不可重入且不支持条件变量 </font>`Condition`。

不同于一般的 `Lock` 类，`StampedLock` 并不是直接实现 `Lock`或 `ReadWriteLock`接口，而是基于 **CLH 锁** 独立实现的（AQS 也是基于这玩意）。

```java
public class StampedLock implements java.io.Serializable {
}
```

`StampedLock` 提供了三种模式的读写控制模式：读锁、写锁和乐观读。

+ **写锁**：独占锁，一把锁只能被一个线程获得。当一个线程获取写锁后，其他请求读锁和写锁的线程必须等待。类似于 `ReentrantReadWriteLock` 的写锁，不过这里的写锁是不可重入的。
+ **读锁** （悲观读）：共享锁，没有线程获取写锁的情况下，多个线程可以同时持有读锁。如果己经有线程持有写锁，则其他线程请求获取该读锁会被阻塞。类似于 `ReentrantReadWriteLock` 的读锁，不过这里的读锁是不可重入的。
+ **乐观读**：允许多个线程获取乐观读以及读锁。同时允许一个写线程获取写锁。

另外，`StampedLock` 还支持这三种锁在一定条件下进行相互转换 。

```java
long tryConvertToWriteLock(long stamp){}
long tryConvertToReadLock(long stamp){}
long tryConvertToOptimisticRead(long stamp){}
```

`StampedLock` 在获取锁的时候会返回一个 long 型的数据戳，该数据戳用于稍后的锁释放参数，如果返回的数据戳为 0 则表示锁获取失败。当前线程持有了锁再次获取锁还是会返回一个新的数据戳，这也是`StampedLock`不可重入的原因。

```java
// 写锁
public long writeLock() {
    long s, next;  // bypass acquireWrite in fully unlocked case only
    return ((((s = state) & ABITS) == 0L &&
             U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
            next : acquireWrite(false, 0L));
}
// 读锁
public long readLock() {
    long s = state, next;  // bypass acquireRead on common uncontended case
    return ((whead == wtail && (s & ABITS) < RFULL &&
             U.compareAndSwapLong(this, STATE, s, next = s + RUNIT)) ?
            next : acquireRead(false, 0L));
}
// 乐观读
public long tryOptimisticRead() {
    long s;
    return (((s = state) & WBIT) == 0L) ? (s & SBITS) : 0L;
}
```

### StampedLock 的性能为什么更好？
`StampedLock` 性能更好的核心原因在于它开创性地引入了‘乐观读（Optimistic Reading）’机制：它在读取数据时完全不施加任何传统的 CPU 锁，而是通过一个‘邮戳（Stamp）’版本号进行快速的版本校验；这种‘零锁开销’的特技，彻底打破了读写锁中‘读锁与写锁互斥’的性能瓶颈，让读操作在没有写冲突时能跑出接近无锁的极限速度。  

### StampedLock 适合什么场景？
和 `ReentrantReadWriteLock` 一样，`StampedLock` 同样适合<font style="background-color:#FBDE28;">读多写少</font>的业务场景，可以作为 `ReentrantReadWriteLock`的替代品，性能更好。

不过，需要注意的是`StampedLock`不可重入，不支持条件变量 `Condition`，对中断操作支持也不友好（使用不当容易导致 CPU 飙升）。如果你需要用到 `ReentrantLock` 的一些高级性能，就不太建议使用 `StampedLock` 了。

### StampedLock 的底层原理了解吗？
`StampedLock` 在底层完全抛弃了传统的 AQS 框架，而是人肉维护了一个 64 位的 `state` 状态变量和一个基于 CLH 双向链表的等待队列；它巧妙地将 `state` 变量切分为‘高 57 位表示写锁版本号’与‘低 7 位表示悲观读锁计数’，配合底层硬件级 CAS 指令，实现了在‘乐观读’时完全不修改内存状态、不加任何锁，从而跑出了极限的并发性能。  

### ⭐️什么是线程和进程?
**何为进程?**

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程。

如下图所示，在 Windows 中通过查看任务管理器的方式，我们就可以清楚看到 Windows 当前运行的进程（`.exe` 文件的运行）。

**何为线程?**

线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间做切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

Java 程序天生就是多线程程序，

### Java 线程和操作系统的线程有啥区别？
JDK 1.2 之前，Java 线程是基于绿色线程（Green Threads）实现的，这是一种用户级线程（用户线程），也就是说 JVM 自己模拟了多线程的运行，而不依赖于操作系统。由于绿色线程和原生线程比起来在使用时有一些限制（比如绿色线程不能直接使用操作系统提供的功能如异步 I/O、只能在一个内核线程上运行无法利用多核），在 JDK 1.2 及以后，Java 线程改为基于原生线程（Native Threads）实现，也就是说 JVM 直接使用操作系统原生的内核级线程（内核线程）来实现 Java 线程，由操作系统内核进行线程的调度和管理。

我们上面提到了用户线程和内核线程，考虑到很多读者不太了解二者的区别，这里简单介绍一下：

+ 用户线程：由用户空间程序管理和调度的线程，运行在用户空间（专门给应用程序使用）。
+ 内核线程：由操作系统内核管理和调度的线程，运行在内核空间（只有内核程序可以访问）。

顺便简单总结一下用户线程和内核线程的区别和特点：用户线程创建和切换成本低，但不可以利用多核。内核态线程，创建和切换成本高，可以利用多核。

一句话概括 Java 线程和操作系统线程的关系：**现在的 Java 线程的本质其实就是操作系统的线程**。

线程模型是用户线程和内核线程之间的关联方式，常见的线程模型有这三种：

1. 一对一（一个用户线程对应一个内核线程）
2. 多对一（多个用户线程映射到一个内核线程）
3. 多对多（多个用户线程映射到多个内核线程）

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329652914-fdb81823-dc4b-4d6d-a041-b1d670788332.png)

在 Windows 和 Linux 等主流操作系统中，Java 线程采用的是一对一的线程模型，也就是一个 Java 线程对应一个系统内核线程。Solaris 系统是一个特例（Solaris 系统本身就支持多对多的线程模型），HotSpot VM 在 Solaris 上支持多对多和一对一。

### ⭐️请简要描述线程与进程的关系,区别及优缺点？
下图是 Java 内存区域，通过下图我们从 JVM 的角度来说一下线程和进程之间的关系。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329652983-622113ef-1b9b-47e8-b639-285a948ce866.png)

从上图可以看出：一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)****资源，但是每个线程有自己的****程序计数器**、**虚拟机栈** 和 **本地方法栈**。

**总结：** 线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反。

### 如何创建线程？
一般来说，创建线程有很多种方式，例如继承`Thread`类、实现`Runnable`接口、实现`Callable`接口、使用线程池、使用`CompletableFuture`类等等。

不过，这些方式其实并没有真正创建出线程。准确点来说，这些都属于是在 Java 代码中使用多线程的方法。

严格来说，Java 就只有一种方式可以创建线程，那就是通过`new Thread().start()`创建。不管是哪种方式，最终还是依赖于`new Thread().start()`。

### ⭐️说说线程的生命周期和状态?
Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态：

+ NEW: 初始状态，线程被创建出来但没有被调用 `start()` 。
+ RUNNABLE: 运行状态，线程被调用了 `start()`等待运行的状态。
    - RUNNING：运行中状态。
    - READY：就绪状态。
+ BLOCKED：阻塞状态，需要等待锁释放。
+ WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。
+ TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
+ TERMINATED：终止状态，表示该线程已经运行完毕。

线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329653004-7aa7e493-b788-4dd5-9dcb-c7ef7c4f726f.png)

由上图可以看出：

+ 线程创建之后它将处于 **NEW（新建）** 状态，调用 `start()` 方法后开始运行，线程这时候处于 **READY（可运行）** 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 **RUNNING（运行）** 状态。
+ 当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）** 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态。
+ **TIMED_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将线程置于 TIMED_WAITING 状态。当超时时间结束后，线程将会返回到 RUNNABLE 状态。
+ 当线程进入 `synchronized` 方法/块或者调用 `wait` 后（被 `notify`）重新进入 `synchronized` 方法/块，但是锁被其它线程占有，这个时候线程就会进入 **BLOCKED（阻塞）** 状态。
+ 线程在执行完了 `run()`方法之后将会进入到 **TERMINATED（终止）** 状态。

### 什么是线程上下文切换?
线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说到过的程序计数器，栈信息等。当出现如下情况的时候，线程会从占用 CPU 状态中退出。

+ 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等。
+ 时间片用完，因为操作系统要防止一个线程或者进程长时间占用 CPU 导致其他线程或者进程饿死。
+ 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
+ 被终止或结束运行

这其中前三种都会发生线程切换，线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是所谓的 **上下文切换**。

上下文切换是现代操作系统的基本功能，因其每次需要保存信息恢复信息，这将会占用 CPU，内存等系统资源进行处理，也就意味着效率会有一定损耗，如果频繁切换就会造成整体效率低下。

### Thread#sleep() 方法和 Object#wait() 方法对比
**共同点**：两者都可以暂停线程的执行。

**区别**：

+ `**sleep()**`** 方法没有释放锁，而 **`**wait()**`** 方法释放了锁** 。
+ `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
+ `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。`sleep()`方法执行完成后，线程会自动苏醒，或者也可以使用 `wait(long timeout)` 超时后线程会自动苏醒。
+ `sleep()` 是 `Thread` 类的静态本地方法，`wait()` 则是 `Object` 类的本地方法。

### 为什么 wait() 方法不定义在 Thread 中？
`wait()` 是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。每个对象（`Object`）都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入 WAITING 状态，自然是要操作对应的对象（`Object`）而非当前的线程（`Thread`）。

类似的问题：**为什么 **`**sleep()**`** 方法定义在 **`**Thread**`** 中？**

因为 `sleep()` 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁。

### 可以直接调用 Thread 类的 run 方法吗？


new 一个 `Thread`，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个普通方法在调用该方法的线程去执行，所以这并不是多线程工作。

**总结：调用 **`**start()**`** 方法方可启动线程并使线程进入就绪状态，直接执行 **`**run()**`** 方法的话不会以多线程的方式执行。**

### 并发与并行的区别
+ **并发**：两个及两个以上的作业在同一 **时间段** 内执行。
+ **并行**：两个及两个以上的作业在同一 **时刻** 执行。

最关键的点是：是否是 **同时** 执行。

### 同步和异步的区别
+ **同步**：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。
+ **异步**：调用在发出之后，不用等待返回结果，该调用直接返回。

### ⭐️为什么要使用多线程?
先从总体上来说：

+ **从计算机底层来说：** 线程可以比作是轻量级的进程，是程序执行的最小单位，线程间的切换和调度的成本远远小于进程。另外，多核 CPU 时代意味着多个线程可以同时运行，这减少了线程上下文切换的开销。
+ **从当代互联网发展趋势来说：** 现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

再深入到计算机底层来探讨：

+ **单核时代**：在单核时代多线程主要是为了提高单进程利用 CPU 和 IO 系统的效率。 假设只运行了一个 Java 进程的情况，当我们请求 IO 的时候，如果 Java 进程中只有一个线程，此线程被 IO 阻塞则整个进程被阻塞。CPU 和 IO 设备只有一个在运行，那么可以简单地说系统整体效率只有 50%。当使用多线程的时候，一个线程被 IO 阻塞，其他线程还可以继续使用 CPU。从而提高了 Java 进程利用系统资源的整体效率。
+ **多核时代**: 多核时代多线程主要是为了提高进程利用多核 CPU 的能力。举个例子：假如我们要计算一个复杂的任务，我们只用一个线程的话，不论系统有几个 CPU 核心，都只会有一个 CPU 核心被利用到。而创建多个线程，这些线程可以被映射到底层多个 CPU 核心上执行，在任务中的多个线程没有资源竞争的情况下，任务执行的效率会有显著性的提高，约等于（单核时执行时间/CPU 核心数）。

### ⭐️单核 CPU 支持 Java 多线程吗？
单核 CPU 是支持 Java 多线程的。操作系统通过时间片轮转的方式，将 CPU 的时间分配给不同的线程。尽管单核 CPU 一次只能执行一个任务，但通过快速在多个线程之间切换，可以让用户感觉多个任务是同时进行的。

### ⭐️单核 CPU 上运行多个线程效率一定会高吗？
单核 CPU 同时运行多个线程的效率是否会高，取决于线程的类型和任务的性质。一般来说，有两种类型的线程：

1. **CPU 密集型**：CPU 密集型的线程主要进行计算和逻辑处理，需要占用大量的 CPU 资源。
2. **IO 密集型**：IO 密集型的线程主要进行输入输出操作，如读写文件、网络通信等，需要等待 IO 设备的响应，而不占用太多的 CPU 资源。

在单核 CPU 上，同一时刻只能有一个线程在运行，其他线程需要等待 CPU 的时间片分配。如果线程是 CPU 密集型的，那么多个线程同时运行会导致频繁的线程切换，增加了系统的开销，降低了效率。如果线程是 IO 密集型的，那么多个线程同时运行可以利用 CPU 在等待 IO 时的空闲时间，提高了效率。

因此，对于单核 CPU 来说，如果任务是 CPU 密集型的，那么开很多线程会影响效率；如果任务是 IO 密集型的，那么开很多线程会提高效率。当然，这里的“很多”也要适度，不能超过系统能够承受的上限。

### 使用多线程可能带来什么问题?
并发编程的目的就是为了能提高程序的执行效率进而提高程序的运行速度，但是并发编程并不总是能提高程序运行速度的，而且并发编程可能会遇到很多问题，比如：内存泄漏、死锁、线程不安全等等。

### 如何理解线程安全和不安全？
线程安全和不安全是在多线程环境下对于同一份数据的访问是否能够保证其正确性和一致性的描述。

+ 线程安全指的是在多线程环境下，对于同一份数据，不管有多少个线程同时访问，都能保证这份数据的正确性和一致性。
+ 线程不安全则表示在多线程环境下，对于同一份数据，多个线程同时访问时可能会导致数据混乱、错误或者丢失。

### 什么是线程死锁？
线程死锁描述的是这样一种情况：多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。

如下图所示，线程 A 持有资源 2，线程 B 持有资源 1，他们同时都想申请对方的资源，所以这两个线程就会互相等待而进入死锁状态。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329653147-d0cac374-cfab-43ee-aea5-db0af3648d39.png)

下面通过一个例子来说明线程死锁,代码模拟了上图的死锁的情况 (代码来源于《并发编程之美》)：

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

Output

```plain
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

线程 A 通过 `synchronized (resource1)` 获得 `resource1` 的监视器锁，然后通过 `Thread.sleep(1000);` 让线程 A 休眠 1s，为的是让线程 B 得到执行然后获取到 resource2 的监视器锁。线程 A 和线程 B 休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。

上面的例子符合产生死锁的四个必要条件：

1. **互斥条件**：该资源任意一个时刻只由一个线程占用。
2. **请求与保持条件**：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. **不剥夺条件**：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. **循环等待条件**：若干线程之间形成一种头尾相接的循环等待资源关系。

### 如何检测死锁？
+ 使用`jmap`、`jstack`等命令查看 JVM 线程栈和堆内存的情况。如果有死锁，`jstack` 的输出中通常会有 `Found one Java-level deadlock:`的字样，后面会跟着死锁相关的线程信息。另外，实际项目中还可以搭配使用`top`、`df`、`free`等命令查看操作系统的基本情况，出现死锁可能会导致 CPU、内存等资源消耗过高。
+ 采用 VisualVM、JConsole 等工具进行排查。

这里以 JConsole 工具为例进行演示。

首先，我们要找到 JDK 的 bin 目录，找到 jconsole 并双击打开。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329653242-6d6e8fb8-b712-4a69-8b19-0618165550c8.png)

对于 MAC 用户来说，可以通过 `/usr/libexec/java_home -V`查看 JDK 安装目录，找到后通过 `open . + 文件夹地址`打开即可。例如，我本地的某个 JDK 的路径是：

```java
open . /Users/guide/Library/Java/JavaVirtualMachines/corretto-1.8.0_252/Contents/Home
```

打开 jconsole 后，连接对应的程序，然后进入线程界面选择检测死锁即可！

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329653314-71f46a01-082c-475e-8103-193f73fe6ea6.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329653250-b42df6a8-cdef-4341-9711-fcda970215b1.png)

### 如何预防和避免线程死锁?
**如何预防死锁？** 破坏死锁的产生的必要条件即可：

1. **破坏请求与保持条件**：一次性申请所有的资源。
2. **破坏不剥夺条件**：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
3. **破坏循环等待条件**：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

**如何避免死锁？**

避免死锁就是在资源分配时，借助于算法（比如银行家算法）对资源分配进行计算评估，使其进入安全状态。

> **安全状态** 指的是系统能够按照某种线程推进顺序（P1、P2、P3……Pn）来为每个线程分配所需资源，直到满足每个线程对资源的最大需求，使每个线程都可顺利完成。称 `<P1、P2、P3.....Pn>` 序列为安全序列。
>

我们对线程 2 的代码修改成下面这样就不会产生死锁了。

```java
new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 2").start();
```

输出：

```plain
Thread[线程 1,5,main]get resource1
Thread[线程 1,5,main]waiting get resource2
Thread[线程 1,5,main]get resource2
Thread[线程 2,5,main]get resource1
Thread[线程 2,5,main]waiting get resource2
Thread[线程 2,5,main]get resource2

Process finished with exit code 0
```

我们分析一下上面的代码为什么避免了死锁的发生?

线程 1 首先获得到 resource1 的监视器锁,这时候线程 2 就获取不到了。然后线程 1 再去获取 resource2 的监视器锁，可以获取到。然后线程 1 释放了对 resource1、resource2 的监视器锁的占用，线程 2 获取到就可以执行了。这样就破坏了循环等待条件，因此避免了死锁。

### ThreadLocal 是什么？


`ThreadLocal` 是 Java 提供的一种‘线程本地变量’机制；它的核心作用是为每一个线程提供一份完全独立的变量副本，让每个线程只能操作属于自己的数据。其物理本质是通过各个线程肚子里私有的 `ThreadLocalMap` 来存储数据，从而在无锁状态下完美实现了高并发下的线程资源隔离与线程安全。  



### ThreadLocal 有什么用？


`ThreadLocal` 的核心作用是实现‘线程内的数据共享’与‘线程间的资源隔离’；

它最经典的用途有两大场景：

一是高并发下跨层级传递上下文（如存放全局用户信息、TraceId），省去了方法间层层传参的痛苦；	

二是解决线程安全问题，通过为每个线程分发一份独立的非线程安全资源副本（如 `SimpleDateFormat`、数据库连接 `Connection`），让它们在零锁开销下各跑各的。  

### ⭐️ThreadLocal 原理了解吗？


`**ThreadLocal**`** 的底层原理是：每个线程（**`**Thread**`** 对象）内部都偷偷绑定着一个名为 **`**threadLocals**`** 的 **`**ThreadLocalMap**`** 属性；当你调用 **`**threadLocal.set(value)**`** 时，本质上是把当前 **`**ThreadLocal**`** 实例作为 Key，把你传入的数据作为 Value，直接存进**当前线程**私有的那个 Map 里。由于数据在物理上就存在属于线程自己的堆栈内存中，各个线程之间天然做到了完美的资源隔离。**

****

很多人在直觉上会误以为：`ThreadLocal` 内部维护了一个巨大的全局 Map，然后用 `Thread` 作为 Key 去查数据。**这是完全错误的，而且会带来灾难性的锁竞争。**

**现代 Java 的设计完全反了过来：**

+ **每一个线程对象 **`**Thread**`** 内部，都有一个单独的实例变量：**`**ThreadLocal.ThreadLocalMap threadLocals;**`**。**
+ **这个 Map 并不存在 **`**ThreadLocal**`** 里，而是****牢牢长在每个线程自己的肚子里****。**
+ `**ThreadLocal**`** 本质上只是一个“中间人兼通行证”。你调用它的 **`**set()**`** 或 **`**get()**`**，它只是帮你去当前线程的肚子里把那个 Map 抠出来，然后把数据存进去或取出来。**



`ThreadLocalMap` 是 `ThreadLocal` 的一个静态内部类，它虽然叫 Map，但并没有实现 Java 官方的 `Map` 接口，而是 JVM 团队为了极致性能纯手工打造的：

+ **哈希冲突的解决（线性探测法）**：传统的 `HashMap` 发生哈希冲突时使用的是“链表 + 红黑树”。而 `ThreadLocalMap` 采用的是极其死板但也极快的**线性探测法（Linear Probing）**——如果算出来的格子被别人占了，它不会挂链表，而是直接顺着数组往后挪一格，看看有没有空位，直到找到空位为止。
+ **弱引用（WeakReference）的关键设计**：Map 里面存储数据的最小单元是一个个的 `Entry` 节点。精妙之处在于，这个 `Entry` 继承了 `WeakReference<ThreadLocal<?>>`。**也就是说，Map 里的 Key（即 **`**ThreadLocal**`** 实例本身）是被弱引用修饰的，而 Value 则是被强引用修饰的。**

### ⭐️ThreadLocal 内存泄露问题是怎么导致的？
```java
public void doSomething() {
    ThreadLocal<User> local = new ThreadLocal<>();
    local.set(new User("张三")); 
    // 方法执行完毕，局部变量 local 被销毁了
}
```

#### 🚨 物理死亡链路：
1. 当方法执行完毕，栈帧销毁，指向 `ThreadLocal` 的强引用 `local` 瞬间断开。
2. 此时，因为线程肚子里 `Entry` 的 Key 是**弱引用**，在下一次垃圾回收（GC）时，JVM 看到这个 `ThreadLocal` 已经没有强引用护体了，会**无情地将其直接回收**。
3. 结果导致：当前线程的 Map 里，出现了一个 `**Key = null**`** 对应的 **`**Value = User("张三")**` 的奇葩 Entry。
4. **灾难发生**：如果这个线程是**线程池里的核心核心线程**（生命周期极其漫长，几乎与 JVM 同生共死），那么这个线程就会一直活着。由于 Key 已经变成了 `null`，你再也无法通过任何正常的 `threadLocal.get()` 拿到这个 `User` 变量；但由于强引用链条（`Thread -> ThreadLocalMap -> Entry -> Value`）依然存在，**这个 **`**User**`** 对象永远无法被 GC 回收，它就这么无声无息地死锁在内存里，造成了恐怖的内存泄漏**。

### ⭐️如何跨线程传递 ThreadLocal 的值？


#### 方案一：JDK 自带的 `InheritableThreadLocal`（仅限父子线程）
如果你只是在主线程中显式地 `new Thread()` 创建一个子线程，想要把主线程的上下文传过去，用它最合适。

```java
public class ParentChildDemo {
    // 1. 声明一个 InheritableThreadLocal
    private static final ThreadLocal<String> context = new InheritableThreadLocal<>();

    public static void main(String[] args) {
        context.set("Agent-007 的机密数据");

        // 2. 显式创建一个子线程
        new Thread(() -> {
            // 子线程竟然奇迹般地拿到了主线程的值！
            System.out.println("子线程读取: " + context.get()); 
        }).start();
    }
}
```

+ **底层的物理本质**：打开 `Thread` 类的源码，你会发现除了 `threadLocals` 之外，每个线程肚子里面还长着另一个变量：`ThreadLocalMap inheritableThreadLocals;`。
+ **拷贝的时机**：当你在主线程执行 `new Thread()` 的瞬间，子线程的初始化构造方法（`init` 方法）会被调用。JVM 在底层会下一道命令：**检查父线程的 **`**inheritableThreadLocals**`** 是否有值，如果有，就把父线程的 Map 完整地复制一份，塞进子线程的 **`**inheritableThreadLocals**`** 肚子里。**

##### 🚨 `InheritableThreadLocal` 的致命死穴：遇上线程池直接瘫痪
在企业级生产环境中，**绝对禁止盲目显式 **`**new Thread()**`，所有异步任务必须提交给**线程池（ThreadPoolExecutor）**。这时候，`InheritableThreadLocal` 会引发恐怖的**业务数据错乱 Bug**。

+ **瘫痪的原因（线程复用）**： 线程池的核心思想是**线程复用**。
    1. 当**主线程 A** 第一次向线程池提交任务时，线程池创建了**核心线程 1**。此时核心线程 1 顺利触发了上面说的构造方法拷贝，拿到了线程 A 的值。
    2. 任务执行完，核心线程 1 **不销毁**，留在池子里。
    3. 过了几秒，**主线程 B** 来了，把上下文改成了“Agent-008”。它再次向线程池提交任务。
    4. 此时线程池直接复用了闲置的**核心线程 1**。**因为核心线程 1 早在一开始就创建好了，绝对不会重新执行构造方法！** 5. **灾难发生**：核心线程 1 肚子里留着的依然是好几秒前线程 A 的“Agent-007”，它完全感知不到线程 B 的更新。这就发生了**严重的上下文丢失与数据踩踏**！

#### 方案二：阿里巴巴开源的 `TransmittableThreadLocal` (TTL) —— 线程池的终极救星
为了彻底解决线程池复用导致的上下文丢失问题，阿里巴巴开源了 `**TransmittableThreadLocal**`** (TTL)**，它长在了现代大厂微服务架构的骨架上。

```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>transmittable-thread-local</artifactId>
  <version>2.14.2</version>
</dependency>
```

```java
public class ThreadPoolTTLDemo {
    // 1. 使用 TTL 声明变量
    private static final TransmittableThreadLocal<String> traceId = new TransmittableThreadLocal<>();
    // 2. 声明一个普通线程池
    private static final ExecutorService executor = Executors.newFixedThreadPool(2);
    // 3. 革命性动作：用 TTL 官方提供的工具，把普通线程池人肉“包装”一下
    private static final Executor ttlExecutor = TtlExecutors.getTtlExecutor(executor);

    public static void main(String[] args) {
        traceId.set("Trace-001");
        ttlExecutor.execute(() -> {
            // 哪怕线程被无限复用，这里每次都能完美拿到提交任务那一瞬间的最新的 TraceId！
            System.out.println("异步任务读取 TraceId: " + traceId.get());
        });
    }
}
```

##### 🛠️ TTL 完美的底层黑魔法：抓取（Capture） ➡️ 放回（Replay） ➡️ 恢复（Restore）
TTL 为什么能在复用的线程池里来去自如？因为它把变量拷贝的时机，从不可控的“线程创建时”，硬生生推迟到了“任务提交与执行的那一瞬间”。

它的底层切面运转画面包含极其精妙的三步大招：

1. **第一步：抓取（Capture）** 当主线程调用 `ttlExecutor.execute(runnable)` 准备提交任务的温存瞬间，TTL 会在主线程侧暗中启动切面，**把当前主线程里所有的 TTL 变量打包截个图（抓取快照）**，并死死绑定在这个被提交的 `Runnable` 任务对象身上。
2. **第二步：回放（Replay）** 当核心线程池里的某个旧线程终于排到队，把这个任务抓过去准备执行的 `**run()**`** 方法前一刻**： TTL 会强行介入，**把刚才绑定在任务身上的快照数据，人肉强行灌进这个旧线程的肚子里（复写它的 Map）**。这就叫数据回放。这时候，旧线程瞬间清醒，拿到了最新值，高高兴兴跑完业务。
3. **第三步：恢复（Restore）** 任务刚一执行完、**在 **`**run()**`** 方法退出的那一瞬间**： TTL 执行最后的善后清理，**把这个旧线程肚子里刚才灌进去的数据无情抹去，彻底还原成它进门前的原始状态（恢复旧状态）**。这样可以确保这个线程干净利落地回到池子里，绝对不会污染下一个倒霉的任务。



### 什么是线程池?
线程池就是管理一系列线程的资源池。当有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。

### ⭐️为什么要用线程池？
池化技术想必大家已经屡见不鲜了，线程池、数据库连接池、HTTP 连接池等等都是对这个思想的应用。池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。

线程池提供了一种限制和管理资源（包括执行一个任务）的方式。 每个线程池还维护一些基本统计信息，例如已完成任务的数量。使用线程池主要带来以下几个好处：

1. **降低资源消耗**：线程池里的线程是可以重复利用的。一旦线程完成了某个任务，它不会立即销毁，而是回到池子里等待下一个任务。这就避免了频繁创建和销毁线程带来的开销。
2. **提高响应速度**：因为线程池里通常会维护一定数量的核心线程（或者说“常驻工人”），任务来了之后，可以直接交给这些已经存在的、空闲的线程去执行，省去了创建线程的时间，任务能够更快地得到处理。
3. **提高线程的可管理性**：线程池允许我们统一管理池中的线程。我们可以配置线程池的大小（核心线程数、最大线程数）、任务队列的类型和大小、拒绝策略等。这样就能控制并发线程的总量，防止资源耗尽，保证系统的稳定性。同时，线程池通常也提供了监控接口，方便我们了解线程池的运行状态（比如有多少活跃线程、多少任务在排队等），便于调优。

### 如何创建线程池？
**方式一：通过 **`**ThreadPoolExecutor**`** 构造函数直接创建 (推荐)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329694115-6e07d331-f752-489d-852f-5f12d0749157.png)

这是最推荐的方式，因为它允许开发者明确指定线程池的核心参数，对线程池的运行行为有更精细的控制，从而避免资源耗尽的风险。

**方式二：通过 **`**Executors**`** 工具类创建 (不推荐用于生产环境)**

`Executors`工具类提供的创建线程池的方法如下图所示：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329694061-fd58916c-9e8b-4551-85a5-e1314526178a.png)

可以看出，通过`Executors`工具类可以创建多种类型的线程池，包括：

+ `FixedThreadPool`：固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
+ `SingleThreadExecutor`： 只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
+ `CachedThreadPool`： 可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。
+ `ScheduledThreadPool`：给定的延迟后运行任务或者定期执行任务的线程池。

### ⭐️为什么不推荐使用内置线程池？


#### 🚨 隐患一：`newFixedThreadPool` 和 `newSingleThreadExecutor`（死穴：无界队列）
这两种线程池一个叫“固定大小线程池”，一个叫“单线程池”。我们直接打开 JDK 源码，看看它们的底层参数：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
return new ThreadPoolExecutor(nThreads, nThreads,
                              0L, TimeUnit.MILLISECONDS,
                              new LinkedBlockingQueue<Runnable>()); // 致命死穴在这里！
}
```

##### 🧬 底层物理画面与灾难演线：
1. `LinkedBlockingQueue` 如果在创建时不指定大小，它默认的容量是 `**Integer.MAX_VALUE**`（即 $2^{31}-1$，接近 21 亿）。这在物理层面上相当于一个**无界队列（无限大队列）**。
2. 当大促流量激增，或者下游数据库出现短暂拥堵时，线程池里的核心线程全在超负荷工作。
3. 此时源源不断进来的新异步任务抢不到核心线程，就会被全部塞进这个 `LinkedBlockingQueue` 队列里排队。
4. **灾难发生**：由于队列几乎无限大，它可以像黑洞一样疯狂吞噬任务。随着几十万、上百万个任务对象在堆内存中堆积，**JVM 的堆内存会被瞬间榨干，直接向操作系统抛出 **`**java.lang.OutOfMemoryError: Java heap space**`** 崩溃闪退**。此时，整个微服务彻底瘫痪。

#### 🚨 隐患二：`newCachedThreadPool` 和 `newScheduledThreadPool`（死穴：无界线程数）
这两种线程池一个叫“缓存线程池”（号称可以无限扩容），一个叫“定时任务线程池”。我们同样来看它的 JDK 源码：

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, // 致命死穴在这里！
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

##### 🧬 底层物理画面与灾难演线：
1. 它的最大线程数（`maximumPoolSize`）被设置成了 `**Integer.MAX_VALUE**`。也就是说，它允许操作系统创建接近 21 亿个线程。
2. `SynchronousQueue` 是一个不存储元素的特殊队列，来一个任务就必须立刻扔给一个线程去跑。
3. 假设你的业务并发极高，一秒钟进来了 5000 个请求。由于核心线程不够或为 0，线程池为了不丢弃任务，会开始**疯狂地向操作系统申请创建新的物理线程**。
4. **灾难发生**：在 Linux/Windows 操作系统中，**创建线程是非常昂贵的物理行为**（每个线程默认要分配 1MB 的栈内存 `Xss`）。当系统线程数暴增到几千上万个时，不仅会**耗尽操作系统的物理内存导致 OOM**，更恐怖的是，CPU 此时什么业务都不干了，全部算力都浪费在成千上万个线程之间的上下文切换（Context Switch）上。服务器的 CPU 利用率会瞬间飙升到 100%，系统完全卡死卡崩溃。



### ⭐️线程池常见参数有哪些？如何解释？
```java
/**
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
    public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                              int maximumPoolSize,//线程池的最大线程数
                              long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                               ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

`ThreadPoolExecutor` 3 个最重要的参数：

+ `corePoolSize` : 任务队列未达到队列容量时，最大可以同时运行的线程数量。
+ `maximumPoolSize` : 任务队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
+ `workQueue`: 新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

`ThreadPoolExecutor`其他常见参数 :

+ `keepAliveTime`:当线程池中的线程数量大于 `corePoolSize` ，即有非核心线程（线程池中核心线程以外的线程）时，这些非核心线程空闲后不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁。
+ `unit` : `keepAliveTime` 参数的时间单位。
+ `threadFactory` :executor 创建新线程的时候会用到。
+ `handler` :拒绝策略（后面会单独详细介绍一下）。

### 线程池的核心线程会被回收吗？
`ThreadPoolExecutor` 默认不会回收核心线程，即使它们已经空闲了。这是为了减少创建线程的开销，因为核心线程通常是要长期保持活跃的。但是，如果线程池是被用于周期性使用的场景，且频率不高（周期之间有明显的空闲时间），可以考虑将 `allowCoreThreadTimeOut(boolean value)` 方法的参数设置为 `true`，这样就会回收空闲（时间间隔由 `keepAliveTime` 指定）的核心线程了。

```java
public void allowCoreThreadTimeOut(boolean value) {
    // 核心线程的 keepAliveTime 必须大于 0 才能启用超时机制
    if (value && keepAliveTime <= 0) {
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");
    }
    // 设置 allowCoreThreadTimeOut 的值
    if (value != allowCoreThreadTimeOut) {
        allowCoreThreadTimeOut = value;
        // 如果启用了超时机制，清理所有空闲的线程，包括核心线程
        if (value) {
            interruptIdleWorkers();
        }
    }
}
```

### 核心线程空闲时处于什么状态？
核心线程空闲时，其状态分为以下两种情况：

+ **设置了核心线程的存活时间** ：核心线程在空闲时，会处于 `WAITING` 状态，等待获取任务。如果阻塞等待的时间超过了核心线程存活时间，则该线程会退出工作，将该线程从线程池的工作线程集合中移除，线程状态变为 `TERMINATED` 状态。
+ **没有设置核心线程的存活时间** ：核心线程在空闲时，会一直处于 `WAITING` 状态，等待获取任务，核心线程会一直存活在线程池中。

### ⭐️线程池的拒绝策略有哪些？
如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任务时，`ThreadPoolExecutor` 定义一些策略:

+ `ThreadPoolExecutor.AbortPolicy`：抛出 `RejectedExecutionException`来拒绝新任务的处理。
+ `ThreadPoolExecutor.CallerRunsPolicy`：调用执行者自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果你的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
+ `ThreadPoolExecutor.DiscardPolicy`：不处理新任务，直接丢弃掉。
+ `ThreadPoolExecutor.DiscardOldestPolicy`：此策略将丢弃最早的未处理的任务请求。

### 如果不允许丢弃任务，应该选择哪个拒绝策略？
根据上面对线程池拒绝策略的介绍，相信大家很容易能够得出答案是：`CallerRunsPolicy` 。

### CallerRunsPolicy 拒绝策略有什么风险？如何解决？
| 风险点 | 具体表现 | 严重程度 | 最容易出问题的场景 |
| --- | --- | --- | --- |
| **阻塞/卡住调用线程** | 调用线程（通常是 Tomcat/Netty IO线程、Spring MVC 主线程、定时任务线程等）被长时间占用，导致整个应用响应变慢甚至假死 | ★★★★★ | Web 服务、RPC 服务、消息消费主线程使用 |
| **级联阻塞 / 雪崩** | 如果多个上游线程同时触发 CallerRuns，调用线程集体被耗时任务阻塞，整个调用链路都慢下来 | ★★★★ | 高并发场景、微服务间调用、网关转发任务 |
| **死锁风险** | 任务内部又提交子任务到**同一个线程池**，而调用线程正被父任务阻塞 → 子任务无法执行 → 死锁 | ★★★★ | 任务嵌套提交（最经典的坑） |
| **主线程 / 关键线程饥饿** | 主线程 / 事件循环线程 / NIO 线程被阻塞，影响心跳、定时调度、连接管理等 | ★★★★ | 使用 Executors.newFixedThreadPool() + CallerRunsPolicy 的常见错误用法 |
| **吞吐量急剧下降** | 本来想异步的任务变成同步，整体 TPS / QPS 崩盘 | ★★★ | 峰值流量远超线程池容量 |
| **线程池失去意义** | 极端情况下退化成“单线程串行执行”，并发优势荡然无存 | ★★★ | 耗时任务占比高 |


按优先级排序，从最推荐开始：

1. **优先使用 AbortPolicy + 监控 + 扩容 / 限流**（最安全、最常见做法）
    - 让线程池抛 RejectedExecutionException
    - 上层捕获异常 → 降级 / 重试 / 限流 / 告诉调用方“系统忙”
    - 配合 Sentinel、Resilience4j、Hystrix 等限流熔断
2. **自定义拒绝策略**（最灵活）
    - 记录日志 + 降级处理
    - 异步重试（放到另一个无界队列或延迟队列）
    - 计数器 + 告警

```java
public class MyRejectPolicy implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // 记录、告警、业务降级...
        log.error("线程池饱和，任务被拒绝: {}", r);
        // 可选：放到另一个备份线程池 / MQ / 数据库重放
    }
}
```

3. **业务上避免任务嵌套提交到同一个池**
    - 使用不同的线程池：父任务池 + 子任务池
    - 或使用 ForkJoinPool / CompletableFuture 的默认 common pool（但注意 common pool 也有阻塞风险）
4. **合理评估线程池参数**
    - 队列别设太小（给一定缓冲）
    - 最大线程数适当放大（但别无限制）
    - 核心线程数 ≈ CPU核数 ~ 2×CPU核数（IO密集型可更大）
5. **极端场景可用 DiscardOldestPolicy**（丢最老的任务，保留新任务）
    - 适合“新数据更重要”的场景（如实时行情、监控上报）

### 线程池常用的阻塞队列有哪些？
| Executors 工厂方法 | 实际使用的队列类型 | 容量特点 | maximumPoolSize 是否有效 | 风险 / 适用场景 |
| --- | --- | --- | --- | --- |
| newFixedThreadPool() | LinkedBlockingQueue | 无界 | 无效（恒等于 core） | 任务积压 → OOM 风险；适合稳定负载 |
| newSingleThreadExecutor() | LinkedBlockingQueue | 无界 | 无效（恒等于 1） | 同上，单线程顺序执行 |
| newCachedThreadPool() | SynchronousQueue | 0（同步） | 有效（可达 Integer.MAX_VALUE） | 短任务多 → 线程爆炸风险；适合突发短任务 |
| newScheduledThreadPool() | DelayedWorkQueue（自定义堆） | 无界（自动扩容） | 无效（恒等于 core） | 定时/周期任务专用 |


### ⭐️线程池处理任务的流程了解吗？
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329695279-d606be22-c8f3-45e5-a76c-fa50a1e6ba31.png)

1. 如果当前运行的线程数小于核心线程数，那么就会新建一个线程来执行任务。
2. 如果当前运行的线程数等于或大于核心线程数，但是小于最大线程数，那么就把该任务放入到任务队列里等待执行。
3. 如果向任务队列投放任务失败（任务队列已经满了），但是当前运行的线程数是小于最大线程数的，就新建一个线程来执行任务。
4. 如果当前运行的线程数已经等同于最大线程数了，新建线程将会使当前运行的线程超出最大线程数，那么当前任务会被拒绝，拒绝策略会调用`RejectedExecutionHandler.rejectedExecution()`方法。

### 线程池在提交任务前，可以提前创建线程吗？
答案是可以的！

+ `ThreadPoolExecutor` 提供了两个方法帮助我们在提交任务之前，完成核心线程的创建，从而实现线程池预热的效果：
+ `prestartCoreThread()`:启动一个线程，等待任务，如果已达到核心线程数，这个方法返回 false，否则返回 true；
+ `prestartAllCoreThreads()`:启动所有的核心线程，并返回启动成功的核心线程数。

### ⭐️线程池中线程异常后，销毁还是复用？
直接说结论，需要分两种情况：

+ **使用**`**execute()**`**提交任务**：当任务通过`execute()`提交到线程池并在执行过程中抛出异常时，如果这个异常没有在任务内被捕获，那么该异常会导致当前线程终止，并且异常会被打印到控制台或日志文件中。线程池会检测到这种线程终止，并创建一个新线程来替换它，从而保持配置的线程数不变。
+ **使用**`**submit()**`**提交任务**：对于通过`submit()`提交的任务，如果在任务执行中发生异常，这个异常不会直接打印出来。相反，异常会被封装在由`submit()`返回的`Future`对象中。当调用`Future.get()`方法时，可以捕获到一个`ExecutionException`。在这种情况下，线程不会因为异常而终止，它会继续存在于线程池中，准备执行后续的任务。

简单来说：使用`execute()`时，未捕获异常导致线程终止，线程池创建新线程替代；使用`submit()`时，异常被封装在`Future`中，线程继续复用。

这种设计允许`submit()`提供更灵活的错误处理机制，因为它允许调用者决定如何处理异常，而`execute()`则适用于那些不需要关注执行结果的场景。

### ⭐️如何给线程池命名？
初始化线程池的时候需要显示命名（设置线程池名称前缀），有利于定位问题。

默认情况下创建的线程名字类似 `pool-1-thread-n` 这样的，没有业务含义，不利于我们定位问题。

给线程池里的线程命名通常有下面两种方式：

**1、利用 guava 的 **`**ThreadFactoryBuilder**`

```java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
                        .setNameFormat(threadNamePrefix + "-%d")
                        .setDaemon(true).build();
ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit.MINUTES, workQueue, threadFactory);
```

**2、自己实现 **`**ThreadFactory**`**。**

```java
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 线程工厂，它设置线程名称，有利于我们定位问题。
 */
public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final String name;

    /**
     * 创建一个带名字的线程池生产工厂
     */
    public NamingThreadFactory(String name) {
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
        return t;
    }
}
```

### 如何设定线程池的大小？
很多人甚至可能都会觉得把线程池配置过大一点比较好！我觉得这明显是有问题的。就拿我们生活中非常常见的一例子来说：**并不是人多就能把事情做好，增加了沟通交流成本。你本来一件事情只需要 3 个人做，你硬是拉来了 6 个人，会提升做事效率嘛？我想并不会。** 线程数量过多的影响也是和我们分配多少人做事情一样，对于多线程这个场景来说主要是增加了**上下文切换**成本。不清楚什么是上下文切换的话，可以看我下面的介绍。

> 上下文切换：
>
> 多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。
>
> 上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。
>
> Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。
>

类比于现实世界中的人类通过合作做某件事情，我们可以肯定的一点是线程池大小设置过大或者过小都会有问题，合适的才是最好。

+ 如果我们设置的线程池数量太小的话，如果同一时间有大量任务/请求需要处理，可能会导致大量的请求/任务在任务队列中排队等待执行，甚至会出现任务队列满了之后任务/请求无法处理的情况，或者大量任务堆积在任务队列导致 OOM。这样很明显是有问题的，CPU 根本没有得到充分利用。
+ 如果我们设置线程数量太大，大量线程可能会同时在争取 CPU 资源，这样会导致大量的上下文切换，从而增加线程的执行时间，影响了整体执行效率。

有一个简单并且适用面比较广的公式：

+ **CPU 密集型任务(N+1)：** 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1。比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。
+ **I/O 密集型任务(2N)：** 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。

**如何判断是 CPU 密集任务还是 IO 密集任务？**

CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。但凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。

> 🌈 拓展一下（参见：[issue#1737](https://github.com/Snailclimb/JavaGuide/issues/1737)）：
>
> 线程数更严谨的计算的方法应该是：`最佳线程数 = N（CPU 核心数）∗（1+WT（线程等待时间）/ST（线程计算时间））`，其中 `WT（线程等待时间）=线程运行总时间 - ST（线程计算时间）`。
>
> 线程等待时间所占比例越高，需要越多线程。线程计算时间所占比例越高，需要越少线程。
>
> 我们可以通过 JDK 自带的工具 VisualVM 来查看 `WT/ST` 比例。
>
> CPU 密集型任务的 `WT/ST` 接近或者等于 0，因此， 线程数可以设置为 N（CPU 核心数）∗（1+0）= N，和我们上面说的 N（CPU 核心数）+1 差不多。
>
> IO 密集型任务下，几乎全是线程等待时间，从理论上来说，你就可以将线程数设置为 2N（按道理来说，WT/ST 的结果应该比较大，这里选择 2N 的原因应该是为了避免创建过多线程吧）。
>

### ⭐️如何动态修改线程池的参数？
美团技术团队的思路是主要对线程池的核心参数实现自定义可配置。这三个核心参数是：

+ `**corePoolSize**`** :** 核心线程数定义了最小可以同时运行的线程数量。
+ `**maximumPoolSize**`** :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
+ `**workQueue**`**:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

**如何支持参数动态配置？** 且看 `ThreadPoolExecutor` 提供的下面这些方法。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329695955-e7aff2b9-06d2-4f1c-9bf9-b9f6ac469c69.png)

格外需要注意的是`corePoolSize`， 程序运行期间的时候，我们调用 `setCorePoolSize()`这个方法的话，线程池会首先判断当前工作线程数是否大于`corePoolSize`，如果大于的话就会回收工作线程。

如果我们的项目也想要实现这种效果的话，可以借助现成的开源项目：

+ [**Hippo4j**](https://github.com/opengoofy/hippo4j)：异步线程池框架，支持线程池动态变更&监控&报警，无需修改代码轻松引入。支持多种使用模式，轻松引入，致力于提高系统运行保障能力。
+ [**Dynamic TP**](https://github.com/dromara/dynamic-tp)：轻量级动态线程池，内置监控告警功能，集成三方中间件线程池管理，基于主流配置中心（已支持 Nacos、Apollo，Zookeeper、Consul、Etcd，可通过 SPI 自定义实现）。

### ⭐️如何设计一个能够根据任务的优先级来执行的线程池？
我们上面也提到了，不同的线程池会选用不同的阻塞队列作为任务队列，比如`FixedThreadPool` 使用的是`LinkedBlockingQueue`（有界队列），默认构造器初始的队列长度为 `Integer.MAX_VALUE` ，由于队列永远不会被放满，因此`FixedThreadPool`最多只能创建核心线程数的线程。

假如我们需要实现一个优先级任务线程池的话，那可以考虑使用 `PriorityBlockingQueue` （优先级阻塞队列）作为任务队列（`ThreadPoolExecutor` 的构造函数有一个 `workQueue` 参数可以传入任务队列）。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/jpeg/58546395/1767329697088-7f5b38d9-4f47-4d1f-af28-c8898e531fc9.jpg.jpeg)

`PriorityBlockingQueue` 是一个支持优先级的无界阻塞队列，可以看作是线程安全的 `PriorityQueue`，两者底层都是使用小顶堆形式的二叉堆，即值最小的元素优先出队。不过，`PriorityQueue` 不支持阻塞操作。

要想让 `PriorityBlockingQueue` 实现对任务的排序，传入其中的任务必须是具备排序能力的，方式有两种：

1. 提交到线程池的任务实现 `Comparable` 接口，并重写 `compareTo` 方法来指定任务之间的优先级比较规则。
2. 创建 `PriorityBlockingQueue` 时传入一个 `Comparator` 对象来指定任务之间的排序规则(推荐)。

不过，这存在一些风险和问题，比如：

+ `PriorityBlockingQueue` 是无界的，可能堆积大量的请求，从而导致 OOM。
+ 可能会导致饥饿问题，即低优先级的任务长时间得不到执行。
+ 由于需要对队列中的元素进行排序操作以及保证线程安全（并发控制采用的是可重入锁 `ReentrantLock`），因此会降低性能。

对于 OOM 这个问题的解决比较简单粗暴，就是继承`PriorityBlockingQueue` 并重写一下 `offer` 方法(入队)的逻辑，当插入的元素数量超过指定值就返回 false 。

饥饿问题这个可以通过优化设计来解决（比较麻烦），比如等待时间过长的任务会被移除并重新添加到队列中，但是优先级会被提升。

对于性能方面的影响，是没办法避免的，毕竟需要对任务进行排序操作。并且，对于大部分业务场景来说，这点性能影响是可以接受的。

### Future 类有什么用？
`Future` 类是异步思想的典型运用，主要用在一些需要执行耗时任务的场景，避免程序一直原地等待耗时任务执行完成，执行效率太低。具体来说是这样的：当我们执行某一耗时的任务时，可以将这个耗时任务交给一个子线程去异步执行，同时我们可以干点其他事情，不用傻傻等待耗时任务执行完成。等我们的事情干完后，我们再通过 `Future` 类获取到耗时任务的执行结果。这样一来，程序的执行效率就明显提高了。

这其实就是多线程中经典的 **Future 模式**，你可以将其看作是一种设计模式，核心思想是异步调用，主要用在多线程领域，并非 Java 语言独有。

在 Java 中，`Future` 类只是一个泛型接口，位于 `java.util.concurrent` 包下，其中定义了 5 个方法，主要包括下面这 4 个功能：

+ 取消任务；
+ 判断任务是否被取消;
+ 判断任务是否已经执行完成;
+ 获取任务执行结果。

```java
// V 代表了Future执行的任务返回值的类型
public interface Future<V> {
    // 取消任务执行
    // 成功取消返回 true，否则返回 false
    boolean cancel(boolean mayInterruptIfRunning);
    // 判断任务是否被取消
    boolean isCancelled();
    // 判断任务是否已经执行完成
    boolean isDone();
    // 获取任务执行结果
    V get() throws InterruptedException, ExecutionException;
    // 指定时间内没有返回计算结果就抛出 TimeOutException 异常
    V get(long timeout, TimeUnit unit)

        throws InterruptedException, ExecutionException, TimeoutExceptio

}
```

简单理解就是：我有一个任务，提交给了 `Future` 来处理。任务执行期间我自己可以去做任何想做的事情。并且，在这期间我还可以取消任务以及获取任务的执行状态。一段时间之后，我就可以 `Future` 那里直接取出任务执行结果。

### Callable 和 Future 有什么关系？
**Callable 是“任务定义者”**，**Future 是“任务结果的持有者和控制器”**。

| 维度 | Callable<V> | Future<V> | 关系说明 |
| --- | --- | --- | --- |
| 作用 | 定义一个**有返回值的、可抛异常的任务** | 代表一个**异步计算的结果**（承诺未来会有结果） | Callable 是任务的“蓝图”，Future 是执行后的“票据” |
| 是否有返回值 | 是，必须实现 `call()`<br/> 方法，返回 V | 是，`get()`<br/> 方法获取结果（阻塞等待） | Future 的结果就是 Callable 的 call() 返回值 |
| 是否可抛异常 | 是，`call()`<br/> 签名声明 throws Exception | 是，`get()`<br/> 会抛出 `ExecutionException`<br/>（包装原始异常） | Callable 允许抛出 checked exception，Future 帮你传递出来 |
| 是否可取消 | 本身不可取消 | 是，支持 `cancel(boolean mayInterruptIfRunning)` | Future 提供了对底层任务的控制权（取消、中断） |
| 执行方式 | 必须提交给 ExecutorService（如 submit()） | 通过 `executor.submit(Callable)`<br/> 返回 | submit(Callable) → 返回 Future；submit(Runnable) → 返回 Future（结果为 null） |
| get() 阻塞行为 | 无（call() 是同步执行的） | 会阻塞直到任务完成、抛异常或被取消 | Future 把异步执行的“等待”能力暴露出来 |
| 典型使用场景 | 需要返回结果、需要处理异常的异步任务 | 提交任务后需要获取结果、检查状态、取消任务的场景 | 几乎总是成对出现：写 Callable → submit 得到 Future → 用 Future.get() 取结果 |


### CompletableFuture 类有什么用？
让异步编程变得像写同步代码一样流畅，支持函数式链式调用、任务组合、异常处理、非阻塞回调，而不需要自己手动管理线程或阻塞等待。

`Future` 在实际使用过程中存在一些局限性，比如不支持异步任务的编排组合、获取计算结果的 `get()` 方法为阻塞调用。

Java 8 才被引入`CompletableFuture` 类可以解决`Future` 的这些缺陷。`CompletableFuture` 除了提供了更为好用和强大的 `Future` 特性之外，还提供了函数式编程、异步任务编排组合（可以将多个异步任务串联起来，组成一个完整的链式调用）等能力。

下面我们来简单看看 `CompletableFuture` 类的定义。

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}
```

可以看到，`CompletableFuture` 同时实现了 `Future` 和 `CompletionStage` 接口。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/jpeg/58546395/1767329696981-efc741d1-9432-4535-9d2a-c1de6eb7acb6.jpg.jpeg)

`CompletionStage` 接口描述了一个异步计算的阶段。很多计算可以分成多个阶段或步骤，此时可以通过它将所有步骤组合起来，形成异步计算的流水线。

`CompletionStage` 接口中的方法比较多，`CompletableFuture` 的函数式能力就是这个接口赋予的。从这个接口的方法参数你就可以发现其大量使用了 Java8 引入的函数式编程。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767329699500-961a64df-bfab-431e-8a59-f24f1e8249dc.png)

### ⭐️一个任务需要依赖另外两个任务执行完之后再执行，怎么设计？
这种任务编排场景非常适合通过`CompletableFuture`实现。这里假设要实现 T3 在 T2 和 T1 执行完后执行。

代码如下（这里为了简化代码，用到了 Hutool 的线程工具类 `ThreadUtil` 和日期时间工具类 `DateUtil`）：

```java
// T1
CompletableFuture<Void> futureT1 = CompletableFuture.runAsync(() -> {
    System.out.println("T1 is executing. Current time：" + DateUtil.now());
    // 模拟耗时操作
    ThreadUtil.sleep(1000);
});
// T2
CompletableFuture<Void> futureT2 = CompletableFuture.runAsync(() -> {
    System.out.println("T2 is executing. Current time：" + DateUtil.now());
    ThreadUtil.sleep(1000);
});

// 使用allOf()方法合并T1和T2的CompletableFuture，等待它们都完成
CompletableFuture<Void> bothCompleted = CompletableFuture.allOf(futureT1, futureT2);
// 当T1和T2都完成后，执行T3
bothCompleted.thenRunAsync(() -> System.out.println("T3 is executing after T1 and T2 have completed.Current time：" + DateUtil.now()));
// 等待所有任务完成，验证效果
ThreadUtil.sleep(3000);
```

通过 `CompletableFuture` 的 `allOf()` 这个静态方法来并行运行 T1 和 T2，当 T1 和 T2 都完成后，再执行 T3。

### ⭐️使用 CompletableFuture，有一个任务失败，如何处理异常？
使用 `CompletableFuture`的时候一定要以正确的方式进行异常处理，避免异常丢失或者出现不可控问题。

下面是一些建议：

+ 使用 `whenComplete` 方法可以在任务完成时触发回调函数，并正确地处理异常，而不是让异常被吞噬或丢失。
+ 使用 `exceptionally` 方法可以处理异常并重新抛出，以便异常能够传播到后续阶段，而不是让异常被忽略或终止。
+ 使用 `handle` 方法可以处理正常的返回结果和异常，并返回一个新的结果，而不是让异常影响正常的业务逻辑。
+ 使用 `CompletableFuture.allOf` 方法可以组合多个 `CompletableFuture`，并统一处理所有任务的异常，而不是让异常处理过于冗长或重复。
+ ……

### ⭐️在使用 CompletableFuture 的时候为什么要自定义线程池？
`CompletableFuture` 默认使用全局共享的 `ForkJoinPool.commonPool()` 作为执行器，所有未指定执行器的异步任务都会使用该线程池。这意味着应用程序、多个库或框架（如 Spring、第三方库）若都依赖 `CompletableFuture`，默认情况下它们都会共享同一个线程池。

虽然 `ForkJoinPool` 效率很高，但当同时提交大量任务时，可能会导致资源竞争和线程饥饿，进而影响系统性能。

为避免这些问题，建议为 `CompletableFuture` 提供自定义线程池，带来以下优势：

+ 隔离性：为不同任务分配独立的线程池，避免全局线程池资源争夺。
+ 资源控制：根据任务特性调整线程池大小和队列类型，优化性能表现。
+ 异常处理：通过自定义 `ThreadFactory` 更好地处理线程中的异常情况。

### AQS 是什么？
**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">AQS（AbstractQueuedSynchronizer）是 Java 并发包（java.util.concurrent）里几乎所有</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">同步工具类</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">（锁、信号量、闭锁、栅栏等）的底层核心框架</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">。 它是一个</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">抽象的队列同步器</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">，通过一个</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">volatile int state</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">（状态变量） + </font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">一个 FIFO 双向链表</font>**<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">（CLH 变种队列）来实现线程的排队、阻塞、唤醒。</font>

<font style="color:rgb(10, 10, 10);background-color:rgb(252, 252, 252);">“AQS 管排队，state 管资源，子类管规则” （AQS 负责队列和阻塞唤醒，state 表示资源状态，子类决定怎么获取/释放资源）</font>

### ⭐️AQS 的原理是什么？


 AQS（AbstractQueuedSynchronizer，抽象队列同步器）是 Java 并发包（JUC）的核心基石；它的底层原理可以概括为：‘一个被 volatile 修饰的 int 类型 state 状态位 + 一个支持多线程争抢的 CLH 双向 FIFO 虚拟排队队列 + 配合底层硬件级 CAS 原子指令’。当线程抢锁失败时，AQS 会将其包装成节点并塞入双向队列中挂起等待，在锁释放时依次唤醒，从而在无锁状态下完美构建出了高效的同步控制框架。  



#### 🧬 AQS 的三大物理核心骨架
在 Java 中，不管是 `ReentrantLock`、`Semaphore`、`CountDownLatch` 还是 `ReadWriteLock`，它们的内部都藏着一个继承了 AQS 的同步器（`Sync`）。AQS 的内部构造可以完美拆解为以下三个部分：

##### 核心状态位：`state` 变量
+ **物理本质**：AQS 肚子里最核心的资产是一个被 `volatile` 修饰的 32 位整型变量：`private volatile int state;`。
+ **状态控制**：这个 `state` 在不同的工具里代表不同的锁含义：
    - 在 `**ReentrantLock**` 中：`state = 0` 代表锁闲置；`state = 1` 代表锁被霸占；`state > 1` 代表持有锁的线程在**可重入**地疯狂敲门。
    - 在 `**Semaphore**` 中：`state` 代表当前系统还剩下多少个**可用的许可证（资源数）**。
    - 在 `**CountDownLatch**` 中：`state` 代表还需要倒计时的**门闩步数**，减到 0 时唤醒所有等待线程。

##### 硬件级防踩踏：CAS 操作
既然整个 JUC 都是为了消灭传统的、笨重的系统级互斥锁，那么当成百上千个线程同时冲过来试图把 `state` 从 0 改成 1 时，AQS 是如何保证不会发生数据撕裂的？

+ **物理本质**：它抛弃了 `synchronized`，直接调用了底层 CPU 芯片的原子指令——**CAS（Compare And Swap）**。AQS 源码中充斥着 `compareAndSetState()` 方法，它直接利用操作系统的 `Unsafe` 类，通过硬件级锁定确保**在同一万分之一秒内，绝对只有一个线程能成功修改 **`**state**`** 并宣告拿锁成功**。

##### 容纳落榜者的避难所：CLH 双向变体队列
当一个幸运儿通过 CAS 成功把 `state` 抢走之后，剩下那 99 个失败的线程该去哪里？

+ **队列结构**：AQS 内部人肉维护了一个**基于 CLH 锁结构魔改的双向 FIFO（先进先出）链表队列**。
+ **虚拟队列**：它是一个“虚拟的双向队列”，因为 AQS 肚子里其实只有两个指针：`private transient volatile Node head;`（头节点）和 `private transient volatile Node tail;`（尾节点）。真正的队列是由一个个 `Node` 节点通过 `prev`（前驱指针）和 `next`（后继指针）手拉手连起来的。



### Semaphore 有什么用？


`Semaphore`（信号量）是 Java 并发包（JUC）中极其重要的一个‘流量控制和资源限流’武器；它的核心作用是控制同一时刻能够共同访问某个特定共享资源的‘最大线程并发数’。其物理本质是通过 AQS 的共享状态（state）来维护一套‘许可证（Permit）’机制，允许指定数量的线程同时过闸，超出限额的线程则强制排队，常用于数据库连接池限流、秒杀接口防踩踏以及流量削峰场景。  



为了让你彻底看清 `Semaphore` 的运转本质，我们把它在底层的运行画面画出来：

+ **许可证池（Permits）**：初始化 `Semaphore` 时，你必须指定一个数字（比如 `new Semaphore(3)`）。这在底层意味着在 AQS 的 `state` 变量里人肉存入了数字 `3`。这 3 个数字就像是火锅店里的 3 个实体座位（许可证）。
+ **抢占座位（**`**acquire()**`**）**：当一条线程执行到 `semaphore.acquire()` 时，它会启动底层的 CAS 指令去尝试把 `state` 的数值**减 1**。
    - 如果减完后 $state \ge 0$，说明还有座位，该线程成功拿到许可证，直接放行去执行核心业务。
    - 如果减完后发现 $state < 0$，说明座位已经满了。此时 AQS 会强行阻断该线程，**把它打包成一个共享模式（Shared）的节点，扔进 AQS 的双向同步队列里挂起睡觉**。
+ **释放座位（**`**release()**`**）**：当坐在里面的线程执行完业务，调用 `semaphore.release()` 时，它会通过 CAS 把 `state`**加 1**（归还许可证）。同时，它会顺手去 AQS 队列的头部，**把正在排队憋得最久的那条线程“叫醒”**，让它顶替空出来的座位。



### Semaphore 的原理是什么？
`Semaphore` 是共享锁的一种实现，它默认构造 AQS 的 `state` 值为 `permits`，你可以将 `permits` 的值理解为许可证的数量，只有拿到许可证的线程才能执行。

调用`semaphore.acquire()` ，线程尝试获取许可证，如果 `state >= 0` 的话，则表示可以获取成功。如果获取成功的话，使用 CAS 操作去修改 `state` 的值 `state=state-1`。如果 `state<0` 的话，则表示许可证数量不足。此时会创建一个 Node 节点加入阻塞队列，挂起当前线程。

```java
/**
 *  获取1个许可证
 */
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
/**
 * 共享模式下获取许可证，获取成功则返回，失败则加入阻塞队列，挂起线程
 */
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // 尝试获取许可证，arg为获取许可证个数，当可用许可证数减当前获取的许可证数结果小于0,则创建一个节点加入阻塞队列，挂起当前线程。
    if (tryAcquireShared(arg) < 0)
      doAcquireSharedInterruptibly(arg);
}
```

调用`semaphore.release();` ，线程尝试释放许可证，并使用 CAS 操作去修改 `state` 的值 `state=state+1`。释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修改 `state` 的值 `state=state-1` ，如果 `state>=0` 则获取令牌成功，否则重新进入阻塞队列，挂起线程。

```java
// 释放一个许可证
public void release() {
    sync.releaseShared(1);
}

// 释放共享锁，同时会唤醒同步队列中的一个线程。
public final boolean releaseShared(int arg) {
    //释放共享锁
    if (tryReleaseShared(arg)) {
      //唤醒同步队列中的一个线程
      doReleaseShared();
      return true;
    }
    return false;
}
```

### CountDownLatch 有什么用？
`CountDownLatch` 允许 `count` 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。

`CountDownLatch` 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 `CountDownLatch` 使用完毕后，它不能再次被使用。

### CountDownLatch 的原理是什么？
`CountDownLatch` 是共享锁的一种实现,它默认构造 AQS 的 `state` 值为 `count`。当线程使用 `countDown()` 方法时,其实使用了`tryReleaseShared`方法以 CAS 的操作来减少 `state`,直至 `state` 为 0 。当调用 `await()` 方法的时候，如果 `state` 不为 0，那就证明任务还没有执行完毕，`await()` 方法就会一直阻塞，也就是说 `await()` 方法之后的语句不会被执行。直到`count` 个线程调用了`countDown()`使 state 值被减为 0，或者调用`await()`的线程被中断，该线程才会从阻塞中被唤醒，`await()` 方法之后的语句得到执行。

### 用过 CountDownLatch 么？什么场景下用的？
`CountDownLatch` 的作用就是 允许 count 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。之前在项目中，有一个使用多线程读取多个文件处理的场景，我用到了 `CountDownLatch` 。具体场景是下面这样的：

我们要读取处理 6 个文件，这 6 个任务都是没有执行顺序依赖的任务，但是我们需要返回给用户的时候将这几个文件的处理的结果进行统计整理。

为此我们定义了一个线程池和 count 为 6 的`CountDownLatch`对象 。使用线程池处理读取任务，每一个线程处理完之后就将 count-1，调用`CountDownLatch`对象的 `await()`方法，直到所有文件读取完之后，才会接着执行后面的逻辑。

伪代码是下面这样的：

```java
public class CountDownLatchExample1 {
    // 处理文件的数量
    private static final int threadCount = 6;

    public static void main(String[] args) throws InterruptedException {
        // 创建一个具有固定线程数量的线程池对象（推荐使用构造方法创建）
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            final int threadnum = i;
            threadPool.execute(() -> {
                try {
                    //处理文件的业务操作
                    //......
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //表示一个文件已经被完成
                    countDownLatch.countDown();
                }

            });
        }
        countDownLatch.await();
        threadPool.shutdown();
        System.out.println("finish");
    }
}
```

**有没有可以改进的地方呢？**

可以使用 `CompletableFuture` 类来改进！Java8 的 `CompletableFuture` 提供了很多对多线程友好的方法，使用它可以很方便地为我们编写多线程程序，什么异步、串行、并行或者等待所有线程执行完任务什么的都非常方便。

```java
CompletableFuture<Void> task1 =
    CompletableFuture.supplyAsync(()->{
        //自定义业务操作
    });
......
CompletableFuture<Void> task6 =
    CompletableFuture.supplyAsync(()->{
    //自定义业务操作
    });
......
CompletableFuture<Void> headerFuture=CompletableFuture.allOf(task1,.....,task6);

try {
    headerFuture.join();
} catch (Exception ex) {
    //......
}
System.out.println("all done. ");
```

上面的代码还可以继续优化，当任务过多的时候，把每一个 task 都列出来不太现实，可以考虑通过循环来添加任务。

```java
//文件夹位置
List<String> filePaths = Arrays.asList(...)
// 异步处理所有文件
List<CompletableFuture<String>> fileFutures = filePaths.stream()
    .map(filePath -> doSomeThing(filePath))
    .collect(Collectors.toList());
// 将他们合并起来
CompletableFuture<Void> allFutures = CompletableFuture.allOf(
    fileFutures.toArray(new CompletableFuture[fileFutures.size()])
);
```

### CyclicBarrier 有什么用？
`CyclicBarrier` 和 `CountDownLatch` 非常类似，它也可以实现线程间的技术等待，但是它的功能比 `CountDownLatch` 更加复杂和强大。主要应用场景和 `CountDownLatch` 类似。

> `CountDownLatch` 的实现是基于 AQS 的，而 `CyclicBarrier` 是基于 `ReentrantLock`(`ReentrantLock` 也属于 AQS 同步器)和 `Condition` 的。
>

`CyclicBarrier` 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是：让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

### CyclicBarrier 的原理是什么？
`CyclicBarrier` 内部通过一个 `count` 变量作为计数器，`count` 的初始值为 `parties` 属性的初始化值，每当一个线程到了栅栏这里了，那么就将计数器减 1。如果 count 值为 0 了，表示这是这一代最后一个线程到达栅栏，就尝试执行我们构造方法中输入的任务。

### 什么是虚拟线程？
虚拟线程是 JVM 自己管理的超轻量级线程（也叫用户态线程、纤程、类似 Go 的 goroutine），它不直接绑定操作系统线程，能以极低成本创建成千上万甚至上百万个，特别适合高并发、IO密集型场景，同时保持同步阻塞式的编程风格。

### 虚拟线程和平台线程有什么关系？
| 维度 | 平台线程 (Platform Thread) | 虚拟线程 (Virtual Thread) | 关键差异说明 |
| --- | --- | --- | --- |
| 调度者 | 操作系统内核 | JVM（用户态调度） | 虚拟线程不占用 OS 线程资源 |
| 数量上限 | 几千 ~ 上万（受 OS 和内存限制） | 百万级甚至更多（内存主要瓶颈） | 密度提升 10~1000 倍 |
| 创建/销毁成本 | 高（分配 1MB+ 栈、内核资源） | 极低（栈初始很小，按需增长，可达几十 KB） | new Thread() 几乎免费 |
| 上下文切换成本 | 高（涉及内核态切换） | 极低（JVM 内部 continuation + 挂载/卸载） | 阻塞时几乎无感 |
| 阻塞行为 | 阻塞整个 OS 线程 | 只阻塞虚拟线程 → 载体线程（carrier）立即去跑别的虚拟线程 | 核心：不浪费载体线程 |
| 编程风格 | 同步阻塞 / 线程池 | 同步阻塞（推荐） / 可与异步混用 | 代码简单，像单线程写法 |
| 典型适用场景 | CPU 密集型任务 | IO 密集型、高并发请求处理（如 Web、RPC、数据库、网络） | Web 服务吞吐量可提升数倍 |
| 守护线程默认 | 可配置 | 默认是守护线程（不可改） | 随载体线程退出 |
| 调试/监控 | 传统 thread dump 友好 | thread dump 会显示虚拟线程栈（JDK 21+ 优化） | IntelliJ/VisualVM 已支持良好 |


### 虚拟线程有什么优点和缺点？
| 维度 | 优点（Pros） | 缺点 / 风险（Cons） | 严重程度 & 缓解方式（2025–2026 当前状态） |
| --- | --- | --- | --- |
| **资源消耗** | 极低：每个虚拟线程初始栈内存很小（几 KB ~ 几十 KB），可创建 **百万级** 甚至更多 | 仍需少量内存（continuation + 元数据），百万线程仍可能吃掉几 GB 堆内存 | ★★☆☆☆ → 远低于平台线程，已不是主要问题 |
| **创建/销毁成本** | 极低（微秒级），几乎免费，**不需要线程池**（推荐 newVirtualThreadPerTaskExecutor） | — | ★★★★★ 最大亮点之一 |
| **并发规模** | 可轻松支持 **线程 per 请求 / per 任务** 模型，百万并发常见 | 载体线程（carrier，通常 CPU 核数级别）竞争激烈时会退化 | ★★★★★ 核心卖点 |
| **编程风格** | **同步阻塞式代码** 就能获得接近异步的高吞吐，代码简单、可读、可调试 | 开发者可能误用在 CPU 密集场景，导致性能退化 | ★★★★★ 最大生产力提升 |
| **IO 密集型性能** | 阻塞时自动卸载（yield），载体线程立即去执行其他虚拟线程 → 高吞吐 | — | ★★★★★ 网络、数据库、文件 IO 场景极强 |
| **CPU 密集型性能** | — | **不适合**，甚至可能比平台线程慢（频繁 continuation 切换开销 + 缺乏专用 CPU 线程） | ★★★★☆ 明确规避 |
| **Pinning 问题** | JDK 22+ 大幅缓解（synchronized + native 调用 pinning 优化） JDK 25 进一步成熟 | JDK 21 时 synchronized / native 方法内阻塞 → 载体线程被“钉死”（pinning），严重退化甚至死锁风险 | ★★☆☆☆ JDK 21 很严重，JDK 23+ 已基本可接受 |
| **线程局部变量** | Scoped Values（JDK 21+）替代 ThreadLocal，更高效 + 自动清理 | 旧代码大量 ThreadLocal → 迁移成本 + Scoped Values 还有局限性 | ★★☆☆☆ 迁移时需注意 |
| **监控 / 调试** | JDK Flight Recorder、JStack 已支持虚拟线程栈追踪（JDK 21+ 逐步完善） | 早期工具（如老版 VisualVM、thread dump）显示混乱，百万线程 dump 文件巨大 | ★★☆☆☆ 现在已较友好（IntelliJ、JMC 支持好） |
| **兼容性** | 与现有 API（ExecutorService、Lock、synchronized）无缝兼容 | 部分遗留库 / 框架（老 JDBC 驱动、某些 JNI、本地代码）仍 pinning 或不兼容 | ★★☆☆☆ 测试依赖库是关键 |
| **吞吐 vs 延迟** | 峰值吞吐大幅提升（尤其高并发 IO） | 尾部延迟（p99/p999）有时更高（调度抖动、载体线程争抢） | ★★☆☆☆ 取决于场景 |
| **学习 / 迁移成本** | 极低：改一行代码（Executors.newVirtualThreadPerTaskExecutor()）即可尝试 | 盲目替换线程池 → 隐藏 pinning / 性能倒挂 问题 | ★★☆☆☆ 先小范围灰度 |


### 如何创建虚拟线程？
```java
// 方式1：最简单，直接启动
Thread.startVirtualThread(() -> {
    // 你的业务代码，阻塞式写法也没问题
    var result = httpClient.send(request);  // 阻塞 IO 也不会卡住其他请求
    process(result);
});

// 方式2：Executors 工厂（推荐生产使用）
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100_000; i++) {  // 轻松 10 万并发
        executor.submit(() -> handleRequest());
    }
}

// 方式3：Thread.Builder
Thread.ofVirtual().name("worker-", 1).start(runnable);
```

### 虚拟线程的底层原理是什么？
+ **M:N 调度模型**
    - M 个虚拟线程 → 调度到 N 个平台线程（carrier threads）上
    - N 通常是 CPU 核数 ~ 2×核数（ForkJoinPool.commonPool 类似）
+ **Continuation（续体）**
    - JVM 内部使用 continuation 机制保存/恢复线程执行状态
    - 当虚拟线程遇到阻塞操作（IO、synchronized、Lock、Thread.sleep 等）时： → JVM 把当前 continuation “挂起”（yield） → 立即从载体线程上“卸载”（unmount） → 载体线程去执行其他就绪的虚拟线程
    - 数据就绪后 → 找个空闲载体线程 → “挂载”（mount） → 恢复 continuation 继续执行
+ **载体线程（Carrier Thread）**
    - 实际上是普通的平台线程（默认用 ForkJoinPool 管理）
    - 一个载体线程可以先后承载成千上万个虚拟线程（时间分片）



