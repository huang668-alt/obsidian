
## 一、 什么是 Fork/Join 框架？

**Fork/Join** 是 Java 7 引入的并发计算框架，主要用于执行**分治任务（Divide-and-Conquer）**。它能够把一个大任务拆分成多个小任务并行执行，最后把小任务的结果合并成最终结果。

- **Fork（拆分）**：开启一个子线程并行执行拆分出的子任务。
    
- **Join（合并）**：等待子任务执行结束，并获取（合并）其结果。
    

## 二、 核心底层机制：工作窃取算法（Work-Stealing）

Fork/Join 框架能实现高效并发的核心在于 **Work-Stealing（工作窃取）算法**。

### 1. 为什么需要工作窃取？

在多线程环境中，如果把任务平均分给不同的线程，由于每个任务的复杂度不同，有些线程很快就做完了（处于空闲状态），而有些线程还在苦苦加班。这就会导致 CPU 资源的浪费。

### 2. 工作窃取是如何运行的？

- 每个工作线程（ForkJoinWorkerThread）都有一个双端队列（Deque）来存放自己的任务。
    
- 当某个线程把自己的队列任务做完后，它不会闲着，而是去**窥探**其他线程的队列。
    
- 它会从其他忙碌线程的队列尾部（Tail）偷偷“窃取”一个任务来帮它执行。
    
- **无锁优化**：本地线程自己消费任务时是从队列**头部（Head）**拿，而窃取线程从**尾部**拿，这样大大减少了线程之间的锁竞争（避免了首尾冲突）。
    

## 三、 核心 API 体系

### 1. 核心线程池：`ForkJoinPool`

负责管理工作线程和任务队列，类似于普通的 `ThreadPoolExecutor`，但内部实现了工作窃取算法。

- 可以通过 `new ForkJoinPool(parallelism)` 指定并行度。
    
- 通常使用通用池：`ForkJoinPool.commonPool()`。
    

### 2. 核心任务类：`ForkJoinTask<V>`

我们要执行的任务必须是 `ForkJoinTask` 的子类。在实际开发中，我们通常不需要直接继承它，而是继承它的两个抽象子类：

|**抽象子类**|**是否有返回值**|**适用场景**|
|---|---|---|
|**`RecursiveTask<V>`**|**有**返回值|比如：分治求和、斐波那契数列、大数据聚合计算|
|**`RecursiveAction`**|**无**返回值|比如：大规模数组排序、多线程批量修改数据|

## 四、 核心代码模板

编写 Fork/Join 任务的代码逻辑非常固定，通常遵循以下伪代码模板：

``` java
if (任务足 够小) {
    直接执行并返回结果;
} else {
    拆分成子任务 1 (fork)
    拆分成子任务 2 (fork)
    等待子任务结果并合并 (join)
}
```

## 五、 实战示例：大数组并行求和

以下是一个使用 `RecursiveTask<Long>` 实现对 1 亿个元素的数组进行并行求和的完整示例：

``` java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class ForkJoinSumDemo {

    // 定义任务拆分的阈值（当前设置为：当数组长度小于 10w 时不再拆分，直接循环求和）
    private static final int THRESHOLD = 100_000;

    // 1. 定义求和任务类，继承 RecursiveTask 
    static class SumTask extends RecursiveTask<Long> {
        private final long[] numbers;
        private final int start;
        private final int end;

        public SumTask(long[] numbers, int start, int end) {
            this.numbers = numbers;
            this.start = start;
            this.end = end;
        }

        @Override
        protected Long compute() {
            int length = end - start;
            
            // 如果任务足够小，直接进行序列化计算
            if (length <= THRESHOLD) {
                long sum = 0;
                for (int i = start; i < end; i++) {
                    sum += numbers[i];
                }
                return sum;
            }

            // 否则，进行拆分 (Fork)
            int middle = start + (length / 2);
            SumTask leftTask = new SumTask(numbers, start, middle);
            SumTask rightTask = new SumTask(numbers, middle, end);

            // 提交子任务并行执行
            leftTask.fork(); 
            rightTask.fork();

            // 等待子任务结果并合并 (Join)
            long leftResult = leftTask.join();
            long rightResult = rightTask.join();

            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        // 初始化一个包含 1 亿个元素的数组
        long[] numbers = new long[100_000_000];
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = i + 1;
        }

        // 2. 创建 ForkJoinPool 线程池
        ForkJoinPool pool = ForkJoinPool.commonPool();

        // 3. 提交总任务
        SumTask mainTask = new SumTask(numbers, 0, numbers.length);
        long startTime = System.currentTimeMillis();
        
        long result = pool.invoke(mainTask); // invoke 是同步阻塞等待结果
        
        long endTime = System.currentTimeMillis();

        System.out.println("计算结果: " + result);
        System.out.println("耗时: " + (endTime - startTime) + " ms");
    }
}
```

## 六、 Fork/Join 的最佳实践与注意事项

> ⚠️ **踩坑警示录**

1. **避免在 `compute()` 中使用阻塞操作**：Fork/Join 线程池的线程数量是有限的（默认等于 CPU 核心数）。如果在任务中进行了 I/O 阻塞、`Thread.sleep()` 或等锁操作，会导致整个池里的线程被耗尽，工作窃取机制直接瘫痪。
    
2. **正确使用 fork() 和 join() 的顺序**：
    
    - ❌ **错误做法**：`left.fork(); long res1 = left.join(); right.fork(); long res2 = right.join();`
        
        - _原因_：这会导致线程刚拆分就立马阻塞等待（Join），直接把并行变成了串行。
            
    - **正确做法**：先全部 `fork()`，最后再统一 `join()`。
        
3. **计算好拆分阈值（Threshold）**：
    
    - 拆得**太空（阈值太大）**：无法充分利用多核 CPU。
        
    - 拆得**太细（阈值太小）**：任务子项过多，导致对象创建和线程切换的系统开销远大于计算本身的开销，反而变慢。
        
4. **Java 8 的 Stream API 衍生**：Java 8 的 `parallelStream()`（并行流）底层就是直接基于 Fork/Join 的 `commonPool()` 实现的。如果你只是做简单的大数据并行过滤、映射和聚合，直接用 `parallelStream()` 会更方便。