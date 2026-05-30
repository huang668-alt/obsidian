## 深入理解JVM-内存
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646908807-60a5f379-46ab-4645-86ff-acb57b7d1d32.png)  
每个方法在执行的时候，都会生成与这个方法相关的**栈帧**；  
**本地方法栈**主要用于执行本地方法—native  
**堆**是JVM管理的最大一块内存空间，线程共享，与堆相关的一个重要概念就是垃圾收集器，现代几乎所有的垃圾收集器都是采用的分代收集算法，堆空间也基于这一点进行了划分：新生代和老年代。Eben空间、From Survivor空间、Survivor空间。  
**方法区**存储元信息（常量、静态变量、class本身）。。。（永久代 Permanent Generation，从JDK1.8彻底废弃，使用元空间 meta space）  
**运行时常量池**是方法区一部分。  
**直接内存**不是由JVM管理，是由操作系统进行管理，与java nio密切相关，JVM通过DirectByteBuffer来操作直接内存。

Java对象创建的过程：

+ new关键字创建对象的3个过程：  
1.在堆内存中创建出对象的实例，通过字节码中的方法  
2.为对象的成员变量赋初值，  
3.将对象的引用返回。

**指针碰撞**（前提是堆内的空间通过一个指针进行分割，一侧是已经被占用的空间，另一侧是未被占用的空间）：当未被占用的空间被放置一个对象之后，指针就发生偏移。）  
**空闲列表**（前提是堆内存空间中已被使用或未被使用的空间是交织在一起的）：这时，虚拟机就需要一个列表来记录哪些空间是可以使用的，哪些空间是已被使用的，接下来找出可以容纳下新创建对象的且未被使用的空间，在此空间存放对象，同时修改列表上的记录）。

两种类型的垃圾回收机制：一种是回收垃圾后将剩余的对象进行移动，堆内的空间进行了分割，一部分是已被占用的空间，另一份是未被占用的空间。另外一种是回收垃圾，不进行对象移动，此时堆内的空间是离散的。

+ 对象在内存中的布局：
    1. 对象头
    2. 实例数据（类中所声明的各项信息）
    3. 对齐填充（可选）
+ 对象引用访问的两种方式：
    1. 句柄
    2. 直接指针

```java
/**
修改文件运行的vm参数：-Xms5m -Xmx5m -XX:+HeapDumpOnOutOfMemoryError
*/
public class Test1{
    public static void main(String[] args){
        List<Test1> list=new ArrayList();
        for(;;) {
            list.add(new Test1());

            //System.gc();  //主动建议垃圾回收器进行垃圾回收
        }
    }
}

运行结果：
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid51776.hprof ... 
Heap dump file created [9178675 bytes in 0.152 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
at java.util.Arrays.copyOf(<u>Arrays.java:3210</u>)
at java.util.Arrays.copyOf(<u>Arrays.java:3181</u>)
at java.util.ArrayList.grow(<u>ArrayList.java:265</u>)
at java.util.ArrayList.ensureExplicitCapacity(<u>ArrayList.java:239</u>)
at java.util.ArrayList.ensureCapacityInternal(<u>ArrayList.java:231</u>)
at java.util.ArrayList.add(<u>ArrayList.java:462</u>)
at com.hisense.Test1.main(<u>Test1.java:11</u>)

                          java_pid51776.hprof是一个转储文件
```

使用jvisualvm工具打开 java_pid51776.hprof转储文件  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646908765-e60e6a55-1713-4d86-bbce-079a75b0ca23.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646908858-af3531a7-39ba-44c0-ab10-41454ce432b8.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646908839-8879c4ac-3155-491b-92fc-e105158d1d1b.png)

+ **OutOfMemoryError**：由于内存空间不足，导致虚拟机无法为对象分配内存，垃圾回收期也不能进行回收，抛出错误。OutOfMemoryError继承了VirtualMachineError

如果打开System.gc();这行的注释，可以显式调用垃圾回收机制。

##### 虚拟机栈
```java
/**
 * 演示虚拟机栈溢出，StackOverFlow
 * 设置虚拟机栈的大小，-Xss100k 
 */
public class Test2 {
    private int length;
    public int getLength() {
        return length;
    }
    public void test() {
        length+=1;
        try{
            test();
            Thread.sleep(300);
        }catch(Exception e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) {
        Test2 t2=new Test2();
        try {
            t2.test();
        }catch(Throwable ex) {
            System.out.println(t2.getLength());
            ex.printStackTrace();
        }
    }
}

运行结果：
987

java.lang.StackOverflowError
at com.hisense.Test2.test(<u>Test2.java:13</u>)
at com.hisense.Test2.test(<u>Test2.java:14</u>)
                          at com.hisense.Test2.test(<u>Test2.java:14</u>)
                                                    at com.hisense.Test2.test(<u>Test2.java:14</u>)
                                                    ................
```

使用jvisualvm工具dump主线程  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646908835-eab75a5c-261d-4ff0-9e4b-38f8091c3eb7.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909062-2edc8fb2-3d51-4692-8f6a-25bd22f33a3d.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909135-aa7598c1-27a0-424a-8c7d-1d957b0ca394.png)

###### 线程死锁检测和与分析工具深度解析：
```java
public class DeadThread{
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                A.method();
            }
        },"A-Thread").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                B.method();
            }
        },"B-Thread").start();
        Thread.sleep(40000);
    }
}

class A {
    public static synchronized void method(){
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {  e.printStackTrace();  }
        B.method();
    }
}

class B {
    public static synchronized void method(){
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) { e.printStackTrace(); }
        A.method();
    }
}
```

使用jconsole工具查看死锁线程：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909191-b6b07d2c-d0f1-4d1a-8bff-0c9fec903ceb.png)  
再使用jvisualvm工具查看线程：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909291-761e5787-2baa-4a42-a444-0b8503b3cae9.png)  
根据工具提示，生成线程Dump：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909319-efc0435b-08a2-45dc-a6a5-bc1c49f1193b.png)  
可以发现，jconsole和jvisualvm工具得到的结论是一样的。

###### 元空间深度解析
+ Test4

```java
/**
 * 方法区产生内存溢出错误
 * ：jdk8中引入元空间，默认的初始大小为21m，如果超过21m，元空间虚拟机会进行垃圾回收，如果还不够就进行空间扩容，扩容的上限为物理内存的上限。
 * 本次测试使用cglib进行元空间内存错误演示。
 * 启动时，修改vm参数: -XX:MaxMetaspaceSize=10m。
 */
public class Test4 {
    public static void main(String[] args) {
        //程序在运行过程中，会不断的创建Test.class的子类，并放置到元空间中
        for(;;) {
            Enhancer enhancer=new Enhancer();
            enhancer.setSuperclass(Test4.class);
            enhancer.setUseCache(false);
            enhancer.setCallback((MethodInterceptor)(obj,method,arg1,proxy)-> 
                                  proxy.invokeSuper(obj, arg1));
            System.out.println("hello world");
            enhancer.create();
        }
    }
}

程序运行结果报错：
java.lang.OutOfMemoryError: Metaspace     元空间内存溢出
```

使用jvisualvm工具观察元空间的变化：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909424-f66b6636-1cdd-47b5-a79b-7a66d194ff78.png)  
-XX:MaxMetaspaceSize  
‑XX:MinMetaspaceFreeRatio  
‑XX:MaxMetaspaceFreeRatio

###### JDK自带的工具：
+ jconsole
+ jvisualvm
+ jmap -clstats PID 打印类加载器数据  
得到以下结果  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909349-00a341cc-8dc7-4b3b-aa30-627d6112b1c7.png)
+ jmap -heap PID 打印堆空间数据  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909453-ef4a1ed7-cc2f-4eb8-90eb-9000e9ff2a87.png)
+ jstat -gc LVMID 用来打印元空间的信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909535-5a601b8f-6789-43b2-a164-42ed95a625f1.png)  
其中：MC: Current Metaspace Capacity (KB); MU: Metaspace Utilization (KB)  
如果使用cgilib不断在运行期生成class，则MC和MU会不断的增加，如下图：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909601-165d6a36-3144-4882-b01b-7ca54710b840.png)
+ jps 查看 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909598-fe1e55b7-b3c1-477a-8f65-063b9d89bbd2.png)
+ jps -l  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909667-74f39dad-9a7b-43b1-afe6-cd29428daa45.png)
+ jcmd PID GC.class_stats 从jdk1.7出现的新命令，用来连接到运行的JVM并输出详尽的类元数据的柱状图。
+ jcmd -l  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909714-66477457-17d6-4582-8bc5-2dadf42cf002.png)
+ jcmd pid VM.flags 查看虚拟机启动参数  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909735-0f38d08f-1b3e-4a13-82ff-1009e341c0c7.png)
+ jcmd pid help 列出当前的java进程可以进行的操作  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909798-6b34bf2a-6792-440c-9a4f-cfa71d275688.png)
+ jcmd pid help JFR.dump ：查看具体命令的选项  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909810-29cd1a98-da64-4787-8327-986040c27617.png)
+ jcmd pid ProfCounter.print : 查看当前进程性能相关的参数  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909917-66378d9f-57b9-415b-a8ea-bfdf4dfd2a2d.png)
+ jcmd pid VM.uptime : 查看jvm的启动时长  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909913-91293428-10f6-42de-a2bc-3de9f8812a84.png)
+ jcmd pid GC.class_histogram ：查看系统中类的统计信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646909923-08ad8831-9936-445f-93e1-92f5a174fb78.png)
+ jcmd pid Thread.print : 查看线程堆栈信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910013-45b8d581-4728-4b72-b7af-d26d4f092099.png)
+ jcmd pid GC.heap_dump C:\Users\Administrator\Desktop 导出heap dump文件，可以使用jvisualvm查看
+ jcmd pid VM.system_properties ： 查看JVM的属性信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910179-98a33901-777f-4aab-af24-4fb85d6567f1.png)
+ jcmd pid VM.version : 查看目标JVM进程的版本信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910140-c659c235-33c8-4531-aa47-adb5ad92da4b.png)
+ jcmd pid VM.command_line : 查看JVM启动的命令行参数信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910150-f6e5f67e-4cc2-4f91-8ca3-8a07f6c78f17.png)
+ jstack : 查看或者导出java应用程序中线程的堆栈信息  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910157-95ac439d-c1ec-4a2b-bcf8-ffbfbc9a0376.png)  
**图形化界面工具** jmc :Java Mission Control  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910252-cc87270a-24e9-41dc-bd09-e2f5ff047408.png)  
Java飞行记录器（JFR）：Java Flight Recorder  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910498-65a8b3dd-f160-432d-8203-a049baf00b27.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910486-e77eca72-05c7-4435-a100-1e788fe573b4.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753646910589-b62432df-f0f9-4c4a-ba35-e741bb25853e.png)
+ jhat工具，利用web打开转储文件。





