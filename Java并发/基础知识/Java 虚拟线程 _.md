## Java 虚拟线程的发展
Java 虚拟线程最早源自 Project Loom，Oracle 在 2017 年正式提出。JDK19 引入预览功能，最后在 JDK21 正式启用。

作为对比，近些年兴起的语言中，Go 从一开始就使用 goroutine，由 go runtime 调度，所以 go 天生就支持这种非常轻量级的并发。

而作为更接近系统编程的 Rust，虽然它没有像 go 一样内建虚拟线程，但是提供了 async/await 的规范，由社区的 tokio、aync-std 等实现 executor，也间接实现了这种协作式调度的并发，它已经属于事实上的标准，Tokio 之于 Rust 有如 Spring 之于 Java。

在 Java 生态中，Reactive 的一些库，是间接实现了轻量级的线程调度的，但是近些年 Reactive Programing 并没有得到大面积的推广，它对新手非常不友好，学习路线陡峭，代码调试非常困难，同时代码可读性也一般。

## 为什么 Java 迟迟没有支持虚拟线程
### 历史原因
+ Java 一开始的线程模型就是系统线程映射模型：Thread 对应一个原生操作系统线程。
+ 硬件和 JVM 技术条件有限的情况下，做轻量级线程会让垃圾回收、同步、调度、异常处理等问题复杂化。
+ Java 生态太庞大，几乎所有框架、库、监控工具等都建立在现有线程模型之上，属实有点改不动。

### Java 已有线程模型的优缺点
Java 的线程池设计 ThreadPoolExecutor 非常成熟，能满足绝大多数场景。同时配合 ForkJoinPool 在 CPU 密集任务中表现很好。

当然它的缺点也是非常明显的：

+ 线程栈空间固定，默认 1M 左右（可以通过启动参数 -Xss 指定）创建大量线程需要消耗大量的内存。
+ 上下文切换开销比较高，如果线程数在几千这个数量级的时候，调度成本会比较高。

当前的互联网环境，我们希望每个 JVM 实例，能承载高并发请求，现在传统的线程模型就会显得过于重。

## Java 虚拟线程池设计
虚拟线程目前是可选项，我们在创建线程或线程池的时候，要主动选择是创建传统的线程还是虚拟线程。

```plain
Thread vt = Thread.ofVirtual() // 如果要创建传统的线程，调用 Thread.ofPlatform()
                  .name("my-virtual-thread", 0) // 可选命名
                  .start(() -> {
                      System.out.println("Hello from virtual thread!");
                  });

// 或者我们也可以使用 Executors 中的新 API 来创建由虚拟线程组成的线程池
// 这个线程池，每次提交任务，都会创建新的虚拟线程来执行任务
ExecutorService executorService = Executors.newVirtualThreadPerTaskExecutor();
```

Java 虚拟线程的核心思想无非就是解耦 Java 线程与操作系统线程，在中间引入载体线程，由载体线程来负责调度任务。

载体线程（Carrier Thread）还是我们理解的原来 Java 中的普通线程，它和操作系统线程一一对应。

载体线程由 ForkJoinPool 负责创建管理，默认地 JVM 会创建和 CPU 核心数一样多的线程出来作为载体线程池。

```plain
┌─────────────────────────────────────────────────┐
│              Virtual Threads                    │
│  VT1    VT2    VT3    ...    VTn                │
└─────┬─────┬─────┬───────────┬───────────────────┘
      │     │     │           │
      │     │     │           │
┌─────▼─────▼─────▼───────────▼───────────────────┐
│         ForkJoinPool (Carrier Threads)          │
│    CT1         CT2         CT3      ...   CTm   │
└─────┬─────────┬───────────┬─────────────────────┘
      │         │           │
      │         │           │
┌─────▼─────────▼───────────▼─────────────────────┐
│              Operating System                   │
│       Native Threads (1:1 mapping)              │
└─────────────────────────────────────────────────┘
```

虚拟线程的任务运行在载体线程上，当任务到达阻塞点的时候（比如调用了 Thread 的 sleep() 方法），JVM 会把它挂起，把这个载体线程释放出来，这样这个载体线程就可以去干别的虚拟线程的活了，等条件满足以后，再调度回来，再运行在某个载体线程上。

> 我觉得把虚拟线程理解为任务会更合理一些，它虽然实现了 Thread 的 API，但是它只是看起来像是一个 Thread，实际上它只负责管理一个 Runnable，由真正的 Thread 来调度它。
>

我们可以从源码中看到下面经过简化的 VirtualThread 内部结构：

```plain
public class VirtualThread extends BaseVirtualThread {
    private volatile int state;           // 线程状态
    private volatile boolean interrupted; // 中断标记
    private Continuation cont;            // 续体对象
    private volatile Thread carrierThread; // 载体线程
    private volatile Parker parker;       
}
```

这里面的核心是 Continuation，很多地方都把它翻译成**续体**，它实现了执行上下文的保存与恢复。

下面是它里面最重要的两个方法：

```plain
public class Continuation {
  
  // 运行直到yield或将任务执行完成
  public final void run() {...}
  
  // 保存当前执行状态并让出控制权
  public static boolean yield(ContinuationScope scope) {...}
}
```

JVM 会在碰到阻塞点的时候，执行 Continuation 的 yield() 方法，释放载体线程，然后在条件满足以后，重新被调度回来执行 run() 方法恢复虚拟线程的执行。

比如代码执行到 Thread.sleep(3000)，此时会调用 yield 方法来释放载体线程，然后调度器会在 3s 后重新调度这个任务，寻找一个合适的载体线程继续推进该任务。

如果某个任务，始终都没有阻塞点，那么它自然就会一直占用着一个载体线程。

在 JDK21 之前，我们代码中执行的各种阻塞操作，都是依赖于操作系统的，比如 sleep, wait, park 等等，这些方法都是 native 方法，都依赖于操作系统的本地方法，比如下面的这些 API，全都是 native 方法：

```plain
// Thread.java
public static native void sleep(long millis) throws InterruptedException;

// Object.java
public final native void wait(long timeoutMillis) throws InterruptedException;

// Unsafe
public native void unpark(Object thread);
public native void park(boolean isAbsolute, long time);
```

但是当我们切换到虚拟线程以后，这些系统调用显然就不能这么用了，否则会导致任务固定在载体线程上，载体线程不能去处理其他的任务，这就是虚拟线程的 **pinning** 现象。所以，为了引入虚拟线程，JDK 需要重新实现所有这些阻塞操作。比如下面是 JDK21 中的 sleep 实现：

```plain
private static void sleepNanos(long nanos) throws InterruptedException {
    ThreadSleepEvent event = beforeSleep(nanos);
    try {
        if (currentThread() instanceof VirtualThread vthread) {
            vthread.sleepNanos(nanos); // JDK 自己管理 parker，不使用操作系统 API
        } else {
            sleepNanos0(nanos); // native 方法
        }
    } finally {
        afterSleep(event);
    }
}

private static native void sleepNanos0(long nanos) throws InterruptedException;
```

对于其他的比如 Unsafe 里面的 park, IO 操作里面的 read, write 等方法，JDK 也都做了重新实现，以适配虚拟线程。

另外，我们需要看到，之所以引入虚拟线程的整体效率可能会比传统的线程模型高，是有几方面的原因的：

+ 系统线程使用固定的栈空间，每创建一个线程，就大约需要 1MB 的栈空间（可通过 -Xss1M 调节），而虚拟线程使用分段栈，按需扩容，初始仅需要几 KB 的栈空间。
+ 另一个就是线程切换的成本差异。传统线程模型里面，线程切换涉及到用户态到内核态的转换，在内核态执行线程切换，保存/恢复上下文，在高并发场景下，每次执行内核调用，成本可能在几百纳秒到几微秒之间。而虚拟线程的切换始终在用户态上，挂起/恢复线程操作的是 Java 对象，所以非常快，大概在几十到几百纳秒之间。

当然，我们也要清楚，如果是 CPU 密集型应用，线程切换开销几乎忽略不计。

### 虚拟线程状态转换
虚拟线程的线程状态非常多，大家可以通过源码找到所有可能的状态：

```plain
/*
     * Virtual thread state transitions:
     *
     *      NEW -> STARTED         // Thread.start, schedule to run
     *  STARTED -> TERMINATED      // failed to start
     *  STARTED -> RUNNING         // first run
     *  RUNNING -> TERMINATED      // done
     *
     *  RUNNING -> PARKING         // Thread parking with LockSupport.park
     *  PARKING -> PARKED          // cont.yield successful, parked indefinitely
     *  PARKING -> PINNED          // cont.yield failed, parked indefinitely on carrier
     *   PARKED -> UNPARKED        // unparked, may be scheduled to continue
     *   PINNED -> RUNNING         // unparked, continue execution on same carrier
     * UNPARKED -> RUNNING         // continue execution after park
     *
     *       RUNNING -> TIMED_PARKING   // Thread parking with LockSupport.parkNanos
     * TIMED_PARKING -> TIMED_PARKED    // cont.yield successful, timed-parked
     * TIMED_PARKING -> TIMED_PINNED    // cont.yield failed, timed-parked on carrier
     *  TIMED_PARKED -> UNPARKED        // unparked, may be scheduled to continue
     *  TIMED_PINNED -> RUNNING         // unparked, continue execution on same carrier
     *
     *   RUNNING -> BLOCKING       // blocking on monitor enter
     *  BLOCKING -> BLOCKED        // blocked on monitor enter
     *   BLOCKED -> UNBLOCKED      // unblocked, may be scheduled to continue
     * UNBLOCKED -> RUNNING        // continue execution after blocked on monitor enter
     *
     *   RUNNING -> WAITING        // transitional state during wait on monitor
     *   WAITING -> WAIT           // waiting on monitor
     *      WAIT -> BLOCKED        // notified, waiting to be unblocked by monitor owner
     *      WAIT -> UNBLOCKED      // timed-out/interrupted
     *
     *       RUNNING -> TIMED_WAITING   // transition state during timed-waiting on monitor
     * TIMED_WAITING -> TIMED_WAIT      // timed-waiting on monitor
     *    TIMED_WAIT -> BLOCKED         // notified, waiting to be unblocked by monitor owner
     *    TIMED_WAIT -> UNBLOCKED       // timed-out/interrupted
     *
     *  RUNNING -> YIELDING        // Thread.yield
     * YIELDING -> YIELDED         // cont.yield successful, may be scheduled to continue
     * YIELDING -> RUNNING         // cont.yield failed
     *  YIELDED -> RUNNING         // continue execution after Thread.yield
     */
```

但是核心的几个转换大致如下：

```plain
NEW → STARTED → (RUNNING ⇄ PARKED) → TERMINATED
           ↓
        PINNED (特殊状态)
```

+ **RUNNING**: 正在载体线程上执行
+ **PARKED**: 被阻塞，已从载体线程卸载
+ **PINNED**: 被固定在载体线程上（无法卸载）

### 不合适使用虚拟线程的场景
CPU 密集型应用

大量使用了 synchronized 或者 JNI

大量使用了 ThreadLocal 的代码，不适合直接切换到使用虚拟线程

## 使用虚拟线程需要注意的问题
对于 web 应用来说，过去我们一直基于线程池来考虑问题，我们的 request 都来自于 Tomcat 或者 Dubbo 的线程池，但是以后我们会淡化线程池的存在，以后我们的代码基本都是跑在虚拟线程上。

我们需要特别关注的是，不能让虚拟线程 pinning 在载体线程上，因为我们前面说过，载体线程是稀缺资源，数量等于 CPU 核心数，如果虚拟线程 pinning 在载体线程上，可能很快载体线程就会被消耗完，导致所有的任务都得不到调度。

### synchronized 代码块
由于 synchronized 使用的是操作系统的互斥锁 mutex，这导致执行到 synchronized 代码块的时候，如果线程进入到队列中排队，等待获取监视器锁，此时载体线程没法被释放出来。所以我们应该尽量使用其他的 JDK 工具替代，比如使用 ReentrantLock，它有更丰富的功能，ReentrantLock 中的 await 方法已经改造过，如果获取不到锁，进入到等待的时候，可以 yield 载体线程。

```plain
// ❌ 错误做法
// 尽量不要再使用监视器锁
Object monitor = new Object();
synchronized(monitor) { // pinning
    // slowCall();
}

// ✅ 正确做法
// 使用 ReentrantLock 代替
ReentrantLock reentrantLock = new ReentrantLock();
reentrantLock.lock();
try {
    //
} finally {
    reentrantLock.unlock();
}
```

### JDBC 驱动的问题
需要注意的是，MySQL 官方的 MySQL Connector/J 这个驱动，一直以来使用了大量的 synchronized 来保证线程安全性，导致我们在请求 MySQL 的过程中虚拟线程会 pin 在载体线程上。

MySQL 的另一个变种 MariaDB 的官方驱动也存在同样的问题。

好在社区已经因为 Project Loom，做了大量的优化，把大量的 synchronized 替换成了 ReentrantLock，以更友好地支持虚拟线程，所以我们要尽量使用新版本的驱动。

MySQL Connector/J 9.0.0 做了优化，mariadb-java-client 在 [3.3.0](https://jira.mariadb.org/browse/CONJ-1115) 做了优化。

### synchronized 已经被解决
JDK24 对 synchronized 做了优化，所以 pinning 的问题，基本得到解决。参考：[https://openjdk.org/jeps/491](https://openjdk.org/jeps/491)

在这个 JEP 里面同时提到，目前只剩下几个情况会导致 pinning，它们几乎不会引起问题。所以我们可以认为 pinning 的问题不太需要开发者关注。

不过本文依然花了一些篇幅介绍 pinning，我觉得了解这一段历史可能也蛮有意思的。

### JNI 调用
对于 JDK 开发来说，要实现虚拟线程，就是需要把一系列的 native 方法用 Java 自己的方式来实现，因为一旦调用到 JNI 里面，JVM 就失去了 yield 任务的能力，它没法感知到任务执行的进度，没法挂起和恢复任务。

在必须调用 JNI 的场景中，如果这个调用比较耗时，我们可以搞一个 workaround 的方案，把 JNI 执行放在独立的线程中去执行，比如这样实现：

```plain
// 定义一个独立的线程池，使用的是传统的线程模型
private static final ExecutorService jniExecutor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

void callNativeSafely() throws Exception {
    Future<Integer> result = jniExecutor.submit(() -> nativeCall());
    int value = result.get(); // 虚拟线程此时挂起，不会pin载体线程
}

native int nativeCall();
```

另一个方案是使用 JDK22 开始提供的 FFM（[JEP 454: Foreign Function & Memory API](https://openjdk.org/jeps/454)），如果底层调用的是 C 库，几乎可以平替 JNI，如果是 C++ 库，暂时还不行。

### ThreadLocal
需要注意的是，ThreadLocal 的语义没有任何改变，切换到虚拟线程模型，不会有任何功能上的变化，但是我们应该改变自己的编码习惯。

因为引入虚拟线程以后，线程不再是稀缺资源，大家会到处随意创建它，它的数量可能非常大，所以我们需要注意 ThreadLocal 的使用，不合理的使用可能会导致耗费掉大量的内存资源。

比如很多序列化框架会使用 ThreadLocal 做缓存，在传统的线程模型上，它能很好地提高序列化和反序列化的性能，因为我们总是重复使用这些线程。但是切换到虚拟线程环境以后，这类缓存可能是弊大于利的，因为缓存的使用率会变得非常低，白白浪费内存资源。

ThreadLocal 是一个很好用的工具，我们不需要通过方法参数就可以进行传值，但是我们也应该看到它非常难以维护，存在几个致命的问题：

+ 代码可能在很多地方随意设值，这导致很多代码难以维护
+ 很容易忘记在使用后进行清理，这很容易造成事故，相信每一个 Java 程序员应该都写过或处理过别人写出来的这种 bug，隐晦且致命

JDK 提供了 ThreadLocal 的替代品 ScopedValue，虽然在设计思想和使用上存在差异，但是它是一个更适配虚拟线程模型的工具。

## ScopedValue
ScopedValue 有几个特点：

+ 不可变性：它的值是只读的，一旦绑定，不可修改
+ 它有明确的生命周期，被严格限制在 where(...).run(...) 作用域内。作用域结束后，值自动清理，不会发生内存泄漏。
+ 继承性，子线程（包括虚拟线程）会继承父线程的 ScopedValue，非常适合在异步处理中传递上下文信息（有使用限制，不能随意继承）

最基本的使用方式：

```plain
// 定义两个 ScopedValue    
private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();
private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

public void test() {
		ScopedValue.where(USER_ID, "user123") // 绑定 userId
		         .where(REQUEST_ID, "req456") // 绑定 requestId
		         .run(() -> {
		             // 在作用域内可以访问绑定的值
		             processRequest();
		         });
}

private static void processRequest() {
    System.out.println("处理请求 - 用户ID: " + USER_ID.get() + ", 请求ID: " + REQUEST_ID.get());
}
```

我们也可以嵌套作用域使用：

```plain
ScopedValue.where(USER_ID, "outer-user")
    .run(() -> {
        System.out.println("外层作用域 USER_ID: " + USER_ID.get());
        // 内层作用域可以重新绑定值
        ScopedValue.where(USER_ID, "inner-user")
                   .run(() -> {
                       System.out.println("内层作用域 USER_ID: " + USER_ID.get());
                   });
        // 回到外层作用域，值恢复
        System.out.println("回到外层作用域 USER_ID: " + USER_ID.get());
    });

// 输出：
// 外层作用域 USER_ID: outer-user
// 内层作用域 USER_ID: inner-user
// 回到外层作用域 USER_ID: outer-user
```

结合虚拟线程使用，这里需要使用等一下会介绍的结构化并发的 StructuredTaskScope：

```plain
ScopedValue.where(USER_ID, "main-user")
    .where(REQUEST_ID, "main-request")
    .run(() -> {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // 启动多个虚拟线程，它们都能访问相同的 ScopedValue
            var task1 = scope.fork(() -> {
                Thread.sleep(100);
                handleSubTask("Task-1");
                return "Result-1";
            });
            var task2 = scope.fork(() -> {
                Thread.sleep(150);
                handleSubTask("Task-2");
                return "Result-2";
            });
            var task3 = scope.fork(() -> {
                Thread.sleep(80);
                handleSubTask("Task-3");
                return "Result-3";
            });
            try {
                // 等待所有任务完成
                scope.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.err.println("任务被中断: " + e.getMessage());
            } catch (Exception e) {
                System.err.println("任务执行失败: " + e.getMessage());
            }
        }
    });
}

private static void handleSubTask(String taskName) {
    System.out.println(taskName + " 执行中 - 用户ID: " + USER_ID.get() +
                      ", 请求ID: " + REQUEST_ID.get() +
                      ", 线程: " + Thread.currentThread().getName());
}

// 输出：
// Task-1 执行中 - 用户ID: main-user, 请求ID: main-request, 线程: 
// Task-3 执行中 - 用户ID: main-user, 请求ID: main-request, 线程: 
// Task-2 执行中 - 用户ID: main-user, 请求ID: main-request, 线程:
```

ThreadLocal 的思想是值绑定到线程上面，而 ScopedValue 是作用域绑定，我们在代码中明确了它的作用范围，出了这个范围自动清理。

## StructuredTaskScope
传统上在 Java 中使用 ExecutorService 提交多个任务，需要手动：

+ 跟踪 Future；
+ 等待它们完成；
+ 处理中途异常；
+ 手动取消其它不需要的任务。

StructuredTaskScope 的目标就是让这些流程统一、简洁、安全。默认地，StructuredTaskScope 会对每一个任务都会使用一个独立的虚拟线程来完成。

我们先来看它的几种常见的使用模式：

**1、完成所有的子任务**

```plain
// 在 try-with-resources 里面使用
// 在 try 代码块中 (1) 使用 fork() 执行多个子任务; (2) 使用 join() 等待子任务完成; (3) 处理子任务返回值
try (var scope = new StructuredTaskScope ()) {

    StructuredTaskScope.Subtask subtask1 = scope.fork(task1);   
    StructuredTaskScope.Subtask subtask2 = scope.fork(task2);  

    scope.join();                                   

    ... process results/exceptions ...

}
```

fork()：提交子任务，返回 Future

join()：等待所有子任务完成

**2、只要有一个子任务失败就退出**

```plain
// ShutdownOnFailure<T>：只要有一个任务失败，就会退出，其余任务会被取消
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    StructuredTaskScope.Subtask<String> f1 = scope.fork(() -> task("Task1", 1));
    StructuredTaskScope.Subtask<String> f2 = scope.fork(() -> task("Task2", 2));
    // 等待任务完成或某个任务失败
    scope.join();          
    scope.throwIfFailed(); // 抛出第一个异常
    System.out.println("Result1: " + f1.get());
    System.out.println("Result2: " + f2.get());
}
```

throwIfFailed()：如果有任务异常，抛出第一个异常。

**2、只要有一个子任务成功就结束**

```plain
// ShutdownOnSuccess<T>：第一个成功结果返回，其余任务会被取消
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> slowService("A", 2));
    scope.fork(() -> slowService("B", 1));
    scope.fork(() -> slowService("C", 3));
    scope.join();  // 等待第一个成功的任务
    String result = scope.result(); // 得到结果
    System.out.println("Winner: " + result);
}
```

只要有一个任务返回结果就可以了，会取消其他任务，常见于调用多个服务取最快结果。

回过头来，我们总结下 StructuredTaskScope 有几个优点：

+ 任务和父作用域绑定，作用域退出会自动清理任务
+ 失败或成功后，能自动取消不再需要的任务
+ 结构化地收集和处理异常
+ 相比 CompletableFuture 的 API 可能会更直观一些

## 小结
目前来看，关于 Java 虚拟线程，很多功能还在不断完善中，大家不妨等待 JDK25 的放出，因为它是下一个长期支持版本。

ScopedValue 和 StructuredTaskScope 在 JDK24 中，依然处于 preview 的状态，不确定最终的 API 会不会发生改变。

（全文完）



> 来自: [Java 虚拟线程 | Javadoop](https://javadoop.com/post/virtual-thread/)
>





