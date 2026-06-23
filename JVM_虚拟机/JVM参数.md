# JVM参数
## 堆内存相关
> Java 堆（Java Heap）是 JVM 所管理的内存中最大的一块区域，**所有线程共享**，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都要在堆上分配内存。**
>

### 设置堆内存大小 (-Xms 和 -Xmx)
根据应用程序的实际需求设置初始和最大堆内存大小，是性能调优中最常见的实践之一。**推荐显式设置这两个参数，并且通常建议将它们设置为相同的值**，以避免运行时堆内存的动态调整带来的性能开销。

使用以下参数进行设置：

```plain
-Xms<heap size>[unit]  # 设置 JVM 初始堆大小
-Xmx<heap size>[unit]  # 设置 JVM 最大堆大小
```

+ `<heap size>`: 指定内存的具体数值。
+ `[unit]`: 指定内存的单位，如 g (GB)、m (MB)、k (KB)。

**示例：** 将 JVM 的初始堆和最大堆都设置为 4GB：

### 设置新生代内存大小 (Young Generation)
在堆总可用内存配置完成之后，第二大影响因素是为 `Young Generation` 在堆内存所占的比例。默认情况下，YG 的最小大小为 **1310 MB**，最大大小为 **无限制**。

可以通过以下两种方式设置新生代内存大小：

**1.通过**`-XX:NewSize`**和**`-XX:MaxNewSize`**指定**

```plain
-XX:NewSize=<young size>[unit]    # 设置新生代初始大小
-XX:MaxNewSize=<young size>[unit] # 设置新生代最大大小
```

**示例：** 设置新生代最小 512MB，最大 1024MB：

```plain
-XX:NewSize=512m -XX:MaxNewSize=1024m
```

**2.通过**`**-Xmn<young size>[unit]**`**指定**

**示例：** 将新生代大小固定为 512MB：

GC 调优策略中很重要的一条经验总结是这样说的：

> 尽量让新创建的对象在新生代分配内存并被回收，因为 Minor GC 的成本通常远低于 Full GC。通过分析 GC 日志，判断新生代空间分配是否合理。如果大量新对象过早进入老年代（Promotion），可以适当通过 `-Xmn` 或 -`XX:NewSize/-XX:MaxNewSize` 调整新生代大小，目标是最大限度地减少对象直接进入老年代的情况。
>

另外，你还可以通过 `**-XX:NewRatio=<int>**` 参数来设置**老年代与新生代（不含 Survivor 区）的内存大小比例**。

例如，`-XX:NewRatio=2` （默认值）表示老年代 : 新生代 = 2 : 1。即新生代占整个堆大小的 1/3。

### 设置永久代/元空间大小 (PermGen/Metaspace)
**从 Java 8 开始，如果我们没有指定 Metaspace 的大小，随着更多类的创建，虚拟机会耗尽所有可用的系统内存（永久代并不会出现这种情况）。**

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```plain
-XX:PermSize=N #方法区 (永久代) 初始大小
-XX:MaxPermSize=N #方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

**JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。**

下面是一些常用参数：

```plain
-XX:MetaspaceSize=N #设置 Metaspace 的初始大小（是一个常见的误区，后面会解释）
-XX:MaxMetaspaceSize=N #设置 Metaspace 的最大大小
```

**🐛**** 修正（参见：**[**issue#1947**](https://github.com/Snailclimb/JavaGuide/issues/1947)**）**：

**1、**`**-XX:MetaspaceSize**`** 并非初始容量：** Metaspace 的初始容量并不是 `-XX:MetaspaceSize` 设置，无论 `-XX:MetaspaceSize` 配置什么值，对于 64 位 JVM，元空间的初始容量通常是一个固定的较小值（Oracle 文档提到约 12MB 到 20MB 之间，实际观察约 20.8MB）。

可以参考 Oracle 官方文档 [Other Considerations](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/considerations.html) 中提到的：

> Specify a higher value for the option MetaspaceSize to avoid early garbage collections induced for class metadata. The amount of class metadata allocated for an application is application-dependent and general guidelines do not exist for the selection of MetaspaceSize. The default size of MetaspaceSize is platform-dependent and ranges from 12 MB to about 20 MB.
>
> MetaspaceSize 的默认大小取决于平台，范围从 12 MB 到大约 20 MB。
>

另外，还可以看一下这个试验：[JVM 参数 MetaspaceSize 的误解](https://mp.weixin.qq.com/s/jqfppqqd98DfAJHZhFbmxA)。

**2、扩容与 Full GC：** 当 Metaspace 的使用量增长并首次达到`-XX:MetaspaceSize` 指定的阈值时，会触发一次 Full GC。在此之后，JVM 会动态调整这个触发 GC 的阈值。如果元空间继续增长，每次达到新的阈值需要扩容时，仍然可能触发 Full GC（具体行为与垃圾收集器和版本有关）。垃圾搜集器内部是根据变量 `_capacity_until_GC`来判断 Metaspace 区域是否达到阈值的，初始化代码如下所示：

```plain
void MetaspaceGC::initialize() {
  // Set the high-water mark to MaxMetapaceSize during VM initialization since
  // we can't do a GC during initialization.
  _capacity_until_GC = MaxMetaspaceSize;
}
```

**3、**`**-XX:MaxMetaspaceSize**`** 的重要性：****如果不显式设置 -**`**XX:MaxMetaspaceSize**`**，元空间的最大大小理论上受限于可用的本地内存。在极端情况下（如类加载器泄漏导致不断加载类），这确实****可能耗尽大量本地内存**。因此，**强烈建议设置一个合理的 **`**-XX:MaxMetaspaceSize**`** 上限**，以防止对系统造成影响。

相关阅读：[issue 更正：MaxMetaspaceSize 如果不指定大小的话，不会耗尽内存 #1204](https://github.com/Snailclimb/JavaGuide/issues/1204) 。

## 垃圾收集相关
### 选择垃圾回收器
选择合适的垃圾收集器（Garbage Collector, GC）对于应用的吞吐量和响应延迟至关重要。关于垃圾收集算法和收集器的详细介绍，可以看笔者写的这篇：[JVM 垃圾回收详解（重点）](https://javaguide.cn/java/jvm/jvm-garbage-collection.html)。

JVM 提供了多种 GC 实现，适用于不同的场景：

+ **Serial GC (串行垃圾收集器):** 单线程执行 GC，适用于客户端模式或单核 CPU 环境。参数：`-XX:+UseSerialGC`。
+ **Parallel GC (并行垃圾收集器):** 多线程执行新生代 GC (Minor GC)，以及可选的多线程执行老年代 GC (Full GC，通过 `-XX:+UseParallelOldGC`)。关注吞吐量，是 JDK 8 的默认 GC。参数：`-XX:+UseParallelGC`。
+ **CMS GC (Concurrent Mark Sweep 并发标记清除收集器):** 以获取最短回收停顿时间为目标，大部分 GC 阶段可与用户线程并发执行。适用于对响应时间要求高的应用。在 JDK 9 中被标记为弃用，JDK 14 中被移除。参数：`-XX:+UseConcMarkSweepGC`。
+ **G1 GC (Garbage-First Garbage Collector):** JDK 9 及之后版本的默认 GC。将堆划分为多个 Region，兼顾吞吐量和停顿时间，试图在可预测的停顿时间内完成 GC。参数：`-XX:+UseG1GC`。
+ **ZGC:** 更新的低延迟 GC，目标是将 GC 停顿时间控制在几毫秒甚至亚毫秒级别，需要较新版本的 JDK 支持。参数（具体参数可能随版本变化）：`-XX:+UseZGC`、`-XX:+UseShenandoahGC`。

### GC 日志记录
在生产环境或进行 GC 问题排查时，**务必开启 GC 日志记录**。详细的 GC 日志是分析和解决 GC 问题的关键依据。

以下是一些推荐配置的 GC 日志参数（适用于 JDK 8/11 等常见版本）：

```plain
# --- 推荐的基础配置 ---
# 打印详细 GC 信息
-XX:+PrintGCDetails
# 打印 GC 发生的时间戳 (相对于 JVM 启动时间)
# -XX:+PrintGCTimeStamps
# 打印 GC 发生的日期和时间 (更常用)
-XX:+PrintGCDateStamps
# 指定 GC 日志文件的输出路径，%t 可以输出日期时间戳
-Xloggc:/path/to/gc-%t.log

# --- 推荐的进阶配置 ---
# 打印对象年龄分布 (有助于判断对象晋升老年代的情况)
-XX:+PrintTenuringDistribution
# 在 GC 前后打印堆信息
-XX:+PrintHeapAtGC
# 打印各种类型引用 (强/软/弱/虚) 的处理信息
-XX:+PrintReferenceGC
# 打印应用暂停时间 (Stop-The-World, STW)
-XX:+PrintGCApplicationStoppedTime

# --- GC 日志文件滚动配置 ---
# 启用 GC 日志文件滚动
-XX:+UseGCLogFileRotation
# 设置滚动日志文件的数量 (例如，保留最近 14 个)
-XX:NumberOfGCLogFiles=14
# 设置每个日志文件的最大大小 (例如，50MB)
-XX:GCLogFileSize=50M

# --- 可选的辅助诊断配置 ---
# 打印安全点 (Safepoint) 统计信息 (有助于分析 STW 原因)
# -XX:+PrintSafepointStatistics
# -XX:PrintSafepointStatisticsCount=1
```

**注意:** JDK 9 及之后版本引入了统一的 JVM 日志框架 (`-Xlog`)，配置方式有所不同，但上述 `-Xloggc` 和滚动参数通常仍然兼容或有对应的新参数。

## 处理 OOM
对于大型应用程序来说，面对内存不足错误是非常常见的，这反过来会导致应用程序崩溃。这是一个非常关键的场景，很难通过复制来解决这个问题。

这就是为什么 JVM 提供了一些参数，这些参数将堆内存转储到一个物理文件中，以后可以用来查找泄漏:

```plain
# 在发生 OOM 时生成堆转储文件
-XX:+HeapDumpOnOutOfMemoryError

# 指定堆转储文件的输出路径。<pid> 会被替换为进程 ID
-XX:HeapDumpPath=/path/to/heapdump/java_pid<pid>.hprof
# 示例：-XX:HeapDumpPath=/data/dumps/

# (可选) 在发生 OOM 时执行指定的命令或脚本
# 例如，发送告警通知或尝试重启服务（需谨慎使用）
# -XX:OnOutOfMemoryError="<command> <args>"
# 示例：-XX:OnOutOfMemoryError="sh /path/to/notify.sh"

# (可选) 启用 GC 开销限制检查
# 如果 GC 时间占总时间比例过高（默认 98%）且回收效果甚微（默认小于 2% 堆内存），
# 会提前抛出 OOM，防止应用长时间卡死在 GC 中。
-XX:+UseGCOverheadLimit
```

## 其他常用参数
+ `-server`: 明确启用 Server 模式的 HotSpot VM。（在 64 位 JVM 上通常是默认值）。
+ `-XX:+UseStringDeduplication`: (JDK 8u20+) 尝试识别并共享底层 `char[]` 数组相同的 String 对象，以减少内存占用。适用于存在大量重复字符串的场景。
+ `-XX:SurvivorRatio=<ratio>`: 设置 Eden 区与单个 Survivor 区的大小比例。例如 `-XX:SurvivorRatio=8` 表示 Eden:Survivor = 8:1。
+ `-XX:MaxTenuringThreshold=<threshold>`: 设置对象从新生代晋升到老年代的最大年龄阈值（对象每经历一次 Minor GC 且存活，年龄加 1）。默认值通常是 15。
+ `-XX:+DisableExplicitGC`: 禁止代码中显式调用 `System.gc()`。推荐开启，避免人为触发不必要的 Full GC。
+ `-XX:+UseLargePages`: (需要操作系统支持) 尝试使用大内存页（如 2MB 而非 4KB），可能提升内存密集型应用的性能，但需谨慎测试。
+ -`XX:MinHeapFreeRatio=<percent> / -XX:MaxHeapFreeRatio=<percent>`: 控制 GC 后堆内存保持空闲的最小/最大百分比，用于动态调整堆大小（如果 `-Xms` 和 `-Xmx` 不相等）。通常建议将 `-Xms` 和 `-Xmx` 设为一致，避免调整开销。

**注意：** 以下参数在现代 JVM 版本中可能已**弃用、移除或默认开启且无需手动设置**：

+ `-XX:+UseLWPSynchronization`: 较旧的同步策略选项，现代 JVM 通常有更优化的实现。
+ `-XX:LargePageSizeInBytes`: 通常由 `-XX:+UseLargePages` 自动确定或通过 OS 配置。
+ `-XX:+UseStringCache`: 已被移除。
+ `-XX:+UseCompressedStrings`: 已被 Java 9 及之后默认开启的 Compact Strings 特性取代。
+ `-XX:+OptimizeStringConcat`: 字符串连接优化（invokedynamic）在 Java 9 及之后是默认行为。

# JDK 命令行工具
这些命令在 JDK 安装目录下的 bin 目录下：

+ `**jps**` (JVM Process Status）: 类似 UNIX 的 `ps` 命令。用于查看所有 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息；
+ `**jstat**`（JVM Statistics Monitoring Tool）: 用于收集 HotSpot 虚拟机各方面的运行数据;
+ `**jinfo**` (Configuration Info for Java) : Configuration Info for Java,显示虚拟机配置信息;
+ `**jmap**` (Memory Map for Java) : 生成堆转储快照;
+ `**jhat**` (JVM Heap Dump Browser) : 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果。JDK9 移除了 jhat；
+ `**jstack**` (Stack Trace for Java) : 生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。

### jps:查看所有 Java 进程
`jps`(JVM Process Status) 命令类似 UNIX 的 `ps` 命令。

`jps`：显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一 ID（Local Virtual Machine Identifier,LVMID）。`jps -q`：只输出进程的本地虚拟机唯一 ID。

```plain
C:\Users\SnailClimb>jps
7360 NettyClient2
17396
7972 Launcher
16504 Jps
17340 NettyServer
```

`jps -l`:输出主类的全名，如果进程执行的是 Jar 包，输出 Jar 路径。

```plain
C:\Users\SnailClimb>jps -l
7360 firstNettyDemo.NettyClient2
17396
7972 org.jetbrains.jps.cmdline.Launcher
16492 sun.tools.jps.Jps
17340 firstNettyDemo.NettyServer
```

`jps -v`：输出虚拟机进程启动时 JVM 参数。

`jps -m`：输出传递给 Java 进程 main() 函数的参数。

### jstat: 监视虚拟机各种运行状态信息
jstat（JVM Statistics Monitoring Tool） 使用于监视虚拟机各种运行状态信息的命令行工具。 它可以显示本地或者远程（需要远程主机提供 RMI 支持）虚拟机进程中的类信息、内存、垃圾收集、JIT 编译等运行数据，在没有 GUI，只提供了纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的首选工具。

`**jstat**`** 命令使用格式：**

```plain
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

比如 `jstat -gc -h3 31736 1000 10`表示分析进程 id 为 31736 的 gc 情况，每隔 1000ms 打印一次记录，打印 10 次停止，每 3 行后打印指标头部。

**常见的 option 如下：**

+ `jstat -class vmid`：显示 ClassLoader 的相关信息；
+ `jstat -compiler vmid`：显示 JIT 编译的相关信息；
+ `jstat -gc vmid`：显示与 GC 相关的堆信息；
+ `jstat -gccapacity vmid`：显示各个代的容量及使用情况；
+ `jstat -gcnew vmid`：显示新生代信息；
+ `jstat -gcnewcapcacity vmid`：显示新生代大小与使用情况；
+ `jstat -gcold vmid`：显示老年代和永久代的行为统计，从 jdk1.8 开始,该选项仅表示老年代，因为永久代被移除了；
+ `jstat -gcoldcapacity vmid`：显示老年代的大小；
+ `jstat -gcpermcapacity vmid`：显示永久代大小，从 jdk1.8 开始,该选项不存在了，因为永久代被移除了；
+ `jstat -gcutil vmid`：显示垃圾收集信息；

另外，加上 `-t`参数可以在输出信息上加一个 Timestamp 列，显示程序的运行时间。

### jinfo: 实时地查看和调整虚拟机各项参数
`jinfo vmid` :输出当前 jvm 进程的全部参数和系统属性 (第一部分是系统的属性，第二部分是 JVM 的参数)。

`jinfo -flag name vmid` :输出对应名称的参数的具体值。比如输出 MaxHeapSize、查看当前 jvm 进程是否开启打印 GC 日志 ( `-XX:PrintGCDetails` :详细 GC 日志模式，这两个都是默认关闭的)。

```plain
C:\Users\SnailClimb>jinfo  -flag MaxHeapSize 17340
-XX:MaxHeapSize=2124414976
C:\Users\SnailClimb>jinfo  -flag PrintGC 17340
-XX:-PrintGC
```

使用 jinfo 可以在不重启虚拟机的情况下，可以动态的修改 jvm 的参数。尤其在线上的环境特别有用,请看下面的例子：

`jinfo -flag [+|-]name vmid` 开启或者关闭对应名称的参数。

```plain
C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:-PrintGC

C:\Users\SnailClimb>jinfo  -flag  +PrintGC 17340

C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:+PrintGC
```

### jmap:生成堆转储快照
`jmap`（Memory Map for Java）命令用于生成堆转储快照。 如果不使用 `jmap` 命令，要想获取 Java 堆转储，可以使用 `“-XX:+HeapDumpOnOutOfMemoryError”` 参数，可以让虚拟机在 OOM 异常出现之后自动生成 dump 文件，Linux 命令下可以通过 `kill -3` 发送进程退出信号也能拿到 dump 文件。

`jmap` 的作用并不仅仅是为了获取 dump 文件，它还可以查询 finalizer 执行队列、Java 堆和永久代的详细信息，如空间使用率、当前使用的是哪种收集器等。和`jinfo`一样，`jmap`有不少功能在 Windows 平台下也是受限制的。

示例：将指定应用程序的堆快照输出到桌面。后面，可以通过 jhat、Visual VM 等工具分析该堆文件。

```plain
C:\Users\SnailClimb>jmap -dump:format=b,file=C:\Users\SnailClimb\Desktop\heap.hprof 17340
Dumping heap to C:\Users\SnailClimb\Desktop\heap.hprof ...
Heap dump file created
```

### jhat: 分析 heapdump 文件
`**jhat**` 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果。

```plain
C:\Users\SnailClimb>jhat C:\Users\SnailClimb\Desktop\heap.hprof
Reading from C:\Users\SnailClimb\Desktop\heap.hprof...
Dump file created Sat May 04 12:30:31 CST 2019
Snapshot read, resolving...
Resolving 131419 objects...
Chasing references, expect 26 dots..........................
Eliminating duplicate references..........................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

访问 [http://localhost:7000/](http://localhost:7000/)

注意⚠️：JDK9 移除了 jhat（[JEP 241: Remove the jhat Tool](https://openjdk.org/jeps/241)），你可以使用其替代品 Eclipse Memory Analyzer Tool (MAT) 和 VisualVM，这也是官方所推荐的。

### jstack :生成虚拟机当前时刻的线程快照
`jstack`（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合.

生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过`jstack`来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。

**下面是一个线程死锁的代码。我们下面会通过 **`**jstack**`** 命令进行死锁检查，输出死锁信息，找到发生死锁的线程。**

```plain
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

线程 A 通过 synchronized (resource1) 获得 resource1 的监视器锁，然后通过`Thread.sleep(1000);`让线程 A 休眠 1s 为的是让线程 B 得到执行然后获取到 resource2 的监视器锁。线程 A 和线程 B 休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。

**通过 **`**jstack**`** 命令分析：**

```plain
C:\Users\SnailClimb>jps
13792 KotlinCompileDaemon
7360 NettyClient2
17396
7972 Launcher
8932 Launcher
9256 DeadLockDemo
10764 Jps
17340 NettyServer

C:\Users\SnailClimb>jstack 9256
```

输出的部分内容如下：

```plain
Found one Java-level deadlock:
=============================
"线程 2":
  waiting to lock monitor 0x000000000333e668 (object 0x00000000d5efe1c0, a java.lang.Object),
  which is held by "线程 1"
"线程 1":
  waiting to lock monitor 0x000000000333be88 (object 0x00000000d5efe1d0, a java.lang.Object),
  which is held by "线程 2"

Java stack information for the threads listed above:
===================================================
"线程 2":
        at DeadLockDemo.lambda$main$1(DeadLockDemo.java:31)
        - waiting to lock <0x00000000d5efe1c0> (a java.lang.Object)
        - locked <0x00000000d5efe1d0> (a java.lang.Object)
        at DeadLockDemo$$Lambda$2/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"线程 1":
        at DeadLockDemo.lambda$main$0(DeadLockDemo.java:16)
        - waiting to lock <0x00000000d5efe1d0> (a java.lang.Object)
        - locked <0x00000000d5efe1c0> (a java.lang.Object)
        at DeadLockDemo$$Lambda$1/1324119927.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

可以看到 `jstack` 命令已经帮我们找到发生死锁的线程的具体信息。

# JDK 可视化分析工具
### JConsole:Java 监视与管理控制台
JConsole 是基于 JMX 的可视化监视、管理工具。可以很方便的监视本地及远程服务器的 java 进程的内存使用情况。你可以在控制台输入`jconsole`命令启动或者在 JDK 目录下的 bin 目录找到`jconsole.exe`然后双击启动。

#### 连接 Jconsole
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767273247007-1be24fba-4b81-4d52-bde3-6135913c6a04.png)

如果需要使用 JConsole 连接远程进程，可以在远程 Java 程序启动时加上下面这些参数:

```plain
-Djava.rmi.server.hostname=外网访问 ip 地址
-Dcom.sun.management.jmxremote.port=60001   //监控的端口号
-Dcom.sun.management.jmxremote.authenticate=false   //关闭认证
-Dcom.sun.management.jmxremote.ssl=false
```

在使用 JConsole 连接时，远程进程地址如下：

#### 查看 Java 程序概况
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767273247004-d36b6abb-4d39-4249-b940-99a233b2b9a2.png)

#### 内存监控
JConsole 可以显示当前内存的详细信息。不仅包括堆内存/非堆内存的整体信息，还可以细化到 eden 区、survivor 区等的使用情况，如下图所示。

点击右边的“执行 GC(G)”按钮可以强制应用程序执行一个 Full GC。

> + **新生代 GC（Minor GC）**:指发生新生代的垃圾收集动作，Minor GC 非常频繁，回收速度一般也比较快。
> + **老年代 GC（Major GC/Full GC）**:指发生在老年代的 GC，出现了 Major GC 经常会伴随至少一次的 Minor GC（并非绝对），Major GC 的速度一般会比 Minor GC 的慢 10 倍以上。
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767273246996-3f8bf9a3-dc76-4f78-aea9-d0d2bbc5498b.png)

#### 线程监控
类似我们前面讲的 `jstack` 命令，不过这个是可视化的。

最下面有一个"检测死锁 (D)"按钮，点击这个按钮可以自动为你找到发生死锁的线程以及它们的详细信息 。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/58546395/1767273247053-b89b6733-c394-4353-bb74-2ba41aa502f6.png)

### Visual VM:多合一故障处理工具
VisualVM 提供在 Java 虚拟机 (Java Virtual Machine, JVM) 上运行的 Java 应用程序的详细信息。在 VisualVM 的图形用户界面中，您可以方便、快捷地查看多个 Java 应用程序的相关信息。Visual VM 官网：[https://visualvm.github.io/](https://visualvm.github.io/) 。Visual VM 中文文档:[https://visualvm.github.io/documentation.html](https://visualvm.github.io/documentation.html)。

下面这段话摘自《深入理解 Java 虚拟机》。

> VisualVM（All-in-One Java Troubleshooting Tool）是到目前为止随 JDK 发布的功能最强大的运行监视和故障处理程序，官方在 VisualVM 的软件说明中写上了“All-in-One”的描述字样，预示着他除了运行监视、故障处理外，还提供了很多其他方面的功能，如性能分析（Profiling）。VisualVM 的性能分析功能甚至比起 JProfiler、YourKit 等专业且收费的 Profiling 工具都不会逊色多少，而且 VisualVM 还有一个很大的优点：不需要被监视的程序基于特殊 Agent 运行，因此他对应用程序的实际性能的影响很小，使得他可以直接应用在生产环境中。这个优点是 JProfiler、YourKit 等工具无法与之媲美的。
>

VisualVM 基于 NetBeans 平台开发，因此他一开始就具备了插件扩展功能的特性，通过插件扩展支持，VisualVM 可以做到：

+ 显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。
+ 监视应用程序的 CPU、GC、堆、方法区以及线程的信息（jstat、jstack）。
+ dump 以及分析堆转储快照（jmap、jhat）。
+ 方法级的程序运行性能分析，找到被调用最多、运行时间最长的方法。
+ 离线程序快照：收集程序的运行时配置、线程 dump、内存 dump 等信息建立一个快照，可以将快照发送开发者处进行 Bug 反馈。
+ 其他 plugins 的无限的可能性……

这里就不具体介绍 VisualVM 的使用，如果想了解的话可以看:

+ [https://visualvm.github.io/documentation.html](https://visualvm.github.io/documentation.html)
+ [https://www.ibm.com/developerworks/cn/java/j-lo-visualvm/index.html](https://www.ibm.com/developerworks/cn/java/j-lo-visualvm/index.html)

### MAT：内存分析器工具
MAT（Memory Analyzer Tool）是一款快速便捷且功能强大丰富的 JVM 堆内存离线分析工具。其通过展现 JVM 异常时所记录的运行时堆转储快照（Heap dump）状态（正常运行时也可以做堆转储分析），帮助定位内存泄漏问题或优化大内存消耗逻辑。

在遇到 OOM 和 GC 问题的时候，我一般会首选使用 MAT 分析 dump 文件在，这也是该工具应用最多的一个场景。







