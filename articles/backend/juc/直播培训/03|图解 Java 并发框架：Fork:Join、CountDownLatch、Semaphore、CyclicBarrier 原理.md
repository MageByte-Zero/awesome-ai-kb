在**第一章节**[《1.6w 字图解 Java 并发：多线程挑战、线程状态和通信、死锁；AQS、ReentrantLock、Condition 使用和原理》](https://mp.weixin.qq.com/s/-AY1G04J0976vDy3wT9LIg)，我们开启了 Java 高并发系列的学习，透彻理解 Java 并发编程基础，主要内容有：

1. 多线程挑战与难点
   1. 上下文切换
   2. 死锁
   3. 资源限制的挑战
   4. 什么是线程
   5. 线程的状态
   6. 线程间的通信
2. Java 各种各样的锁使用和原理
   1. syncconized 的使用和原理
   2. AQS 实现原理
   3. ReentrantLock 的使用和原理
   4. ReentrantReadWriteLock 读写锁使用和原理
   5. Condition 的使用和原理

**第二章**《1.8w 字图解 Java 并发容器： CHM、ConcurrentLinkedQueue、7 种阻塞队列的使用和原理》主要内容如下：

- `ConcurrentHashMap`的使用和原理
- `ConcurrentLinkedQueue` 的使用和原理
- `Java` 7 种阻塞队列的使用和原理详解：`ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue、DelayQueue、SynchronousQueue、LinkedTransferQueue以及LinkedBlockingDeque`

**第三章**节围绕着 Java 并发框架展开，主要内容如下：

- `Fork/Join` 框架的使用和原理
- `CountDownLatch`的使用和原理
- `CyclicBarrier`的使用和原理
- `Semaphore`的使用和原理
- `Exchanger` 的使用和原理

开搞！

## Fork/Join 框架

> Chaya：码哥，什么是 Fork/Join 框架？

Fork/Join 是 Java 7 引入的**并行计算框架**，核心思想是 **"分而治之"**。它通过以下特性解决复杂计算问题：

- **自动任务拆分**：将大任务递归拆分为子任务
- **工作窃取算法**（Work-Stealing）：最大化线程利用率
- **轻量级线程管理**：基于 `ForkJoinPool` 的优化线程池

Fork 就是把一个大任务切分为若干子任务并行的执行，Join 就是合并这些子任务的执行结果，最后得到这个大任务的结 果。

比如计算 1+2+…+10000，可以分割成 10 个子任务，每个子任务分别对 1000 个数进行求和， 最终汇总这 10 个子任务的结果。

Fork/Join 的运行流程图如下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504191922655.png)

### 工作窃取算法

> Chaya：“码哥。有任务要拆分，那必然会出现分配不均匀的情况？要如何实现负载均衡呢？”

这个问题问得好，Chaya 小姐姐。

我们设计一个工作窃取算法（Work-Stealing）来解决这个问题。每个工作线程维护一个**双端队列**（Deque）：

- **头部**：执行自己拆分出的任务（LIFO）
- **尾部**：窃取其他线程的任务（FIFO）

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504191927453.png)

工作窃取算法的优点：充分利用线程进行并行计算，减少了线程间的竞争。

**工作窃取算法的缺点**：在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并 且该算法会消耗了更多的系统资源，比如创建多个线程和多个双端队列。

### 任务拆分流程

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504191929473.png)

### 使用场景

#### 场景 1：大规模数据处理（并行排序）

**需求**：对 10 亿条数据排序，要求内存可控且充分利用多核性能。

**代码实现**：

```java
public class ParallelMergeSort extends RecursiveAction {
    private final int[] array;
    private final int start;
    private final int end;
    private static final int THRESHOLD = 1_000_000; // 拆分阈值

    @Override
    protected void compute() {
        if (end - start <= THRESHOLD) {
            Arrays.sort(array, start, end);  // 小任务直接排序
            return;
        }

        int mid = (start + end) >>> 1;
        invokeAll(
            new ParallelMergeSort(array, start, mid),
            new ParallelMergeSort(array, mid, end)
        );

        merge(array, start, mid, end);  // 合并结果
    }

    // 生产级优化：复用临时数组减少内存分配
    private void merge(int[] array, int start, int mid, int end) {
        int[] temp = ThreadLocalRandom.current().ints().toArray();
        // ... 合并逻辑 ...
    }
}

// 使用方式
ForkJoinPool pool = new ForkJoinPool(Runtime.getRuntime().availableProcessors());
int[] data = loadHugeData();
pool.invoke(new ParallelMergeSort(data, 0, data.length));
```

**性能优化点**：

1. 合理设置 `THRESHOLD`（通过压测确定最佳值）
2. 避免在递归中频繁创建临时数组
3. 使用 `ThreadLocalRandom` 保证线程安全

#### 场景 2：金融计算（蒙特卡洛模拟）

**需求**：快速计算期权定价，要求高精度且低延迟。

**代码实现**：

```java
public class MonteCarloTask extends RecursiveTask<Double> {
    private final int iterations;
    private static final int THRESHOLD = 10_000;

    @Override
    protected Double compute() {
        if (iterations <= THRESHOLD) {
            return calculateSync(); // 同步计算
        }

        MonteCarloTask left = new MonteCarloTask(iterations / 2);
        MonteCarloTask right = new MonteCarloTask(iterations / 2);
        left.fork();

        double rightResult = right.compute();
        double leftResult = left.join(); // 注意顺序：先计算再join

        return (leftResult + rightResult) / 2;
    }

    private double calculateSync() {
        double sum = 0;
        for (int i = 0; i < iterations; i++) {
            sum += randomSimulation();
        }
        return sum / iterations;
    }
}

// 生产级调用（指定超时）
ForkJoinPool pool = new ForkJoinPool(4);
MonteCarloTask task = new MonteCarloTask(1_000_000);
pool.submit(task);

try {
    double result = task.get(5, TimeUnit.SECONDS); // 严格超时控制
} catch (TimeoutException e) {
    task.cancel(true);
    // 降级策略...
}
```

#### ForkJoinPool 生产级配置

**自定义线程工厂**

```java
public class NamedForkJoinThreadFactory implements ForkJoinPool.ForkJoinWorkerThreadFactory {
    private final String namePrefix;
    private final AtomicInteger counter = new AtomicInteger(1);

    @Override
    public ForkJoinWorkerThread newThread(ForkJoinPool pool) {
        ForkJoinWorkerThread thread = new ForkJoinWorkerThread(pool) {};
        thread.setName(namePrefix + "-" + counter.getAndIncrement());
        thread.setPriority(Thread.NORM_PRIORITY);
        thread.setDaemon(false); // 生产环境必须为非守护线程
        return thread;
    }
}
```

与其他并发框架对比

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504191938927.png)

**Fork/Join 适用场景**：

1. 递归可分治的问题（排序、遍历、数学计算）
2. 严格低延迟要求的计算任务
3. 需要自动负载均衡的大规模数据处理

## CountDownLatch

CountDownLatch 是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程执行完后再执行。

例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有框架服务之后执行。

假如有这样一个需求：处理 10 万条数据，分片并行处理，全部完成后触发汇总操作。

```java
public class BatchProcessor {
    private static final int BATCH_SIZE = 1000;
    private final ExecutorService executor = Executors.newFixedThreadPool(8);

    public void process(List<Data> allData) {
        int total = allData.size();
        CountDownLatch latch = new CountDownLatch(total / BATCH_SIZE);

        for (int i = 0; i < total; i += BATCH_SIZE) {
            List<Data> batch = allData.subList(i, Math.min(i+BATCH_SIZE, total));
            executor.submit(() -> {
                try {
                    processBatch(batch);
                } finally {
                    latch.countDown(); // 确保计数减少
                }
            });
        }

        try {
            if (!latch.await(5, TimeUnit.MINUTES)) {
                throw new TimeoutException("Batch processing timeout");
            }
            generateSummaryReport();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            shutdownNow();
        }
    }

    private void processBatch(List<Data> batch) { /* ... */ }
}
```

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192220912.png)

CountDownLatch 的构造函数接收一个 int 类型的参数作为计数器，如果你想等待 N 个点完 成，这里就传入 N。

当我们调用 CountDownLatch 的 countDown 方法时，N 就会减 1，CountDownLatch 的 await 方法 会阻塞当前线程，直到 N 变成零。

用在多个线程时，只需要把这个 CountDownLatch 的引用传递到线程里即可。

如果有某个线程处理得比较慢，我们不可能让主线程一直等待，所以可以使 用另外一个带指定时间的 await 方法——`await（long time，TimeUnit unit）`，这个方法等待特定时 间后，就会不再阻塞当前线程。

### 实现原理

`CountDownLatch` 的核心实现原理是基于 `AQS`，`AQS` 全称 `AbstractQueuedSynchronizer`，是 `java.util.concurrent` 中提供的一种高效且可扩展的同步机制；

它是一种提供了原子式管理同步状态、阻塞和唤醒线程功能以及队列模型的简单框架。

除了 `CountDownLatch` 工具类，JDK 当中的 `Semaphore`、`ReentrantLock` 等工具类都是基于 `AQS` 来实现的。下面我们用 `CountDownLatch` 来分析一下 `AQS` 的实现。

在上一章[《1.6w 字图解 Java 并发：多线程挑战、线程状态和通信、死锁；AQS、ReentrantLock、Condition 使用和原理》](https://mp.weixin.qq.com/s/-AY1G04J0976vDy3wT9LIg)有详细详解 AQS。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192235622.png)

`CountDownLatch` 的源码实现，发现其实它的代码实现非常简单，算上注释也才 300+ 行代码，如果去掉注释的话代码不到 100 行，大部分方法实现都是调用的 `Sync` 这个静态内部类的实现，而 `Sync` 就是继承自 `AbstractQueuedSynchronizer`。

CountDownLatch 的 UML 类图如下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504202044879.png)

核心代码如下。

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    Sync(int count) { setState(count); } // 初始化计数器

    // 尝试获取共享锁：当 state=0 时返回 1（成功），否则返回 -1（失败）
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    // 尝试释放共享锁：CAS 递减 state，直到变为 0
    protected boolean tryReleaseShared(int releases) {
        for (;;) { // 自旋保证原子性
            int c = getState();
            if (c == 0) return false; // 已经释放完毕
            int nextc = c - 1;
            if (compareAndSetState(c, nextc)) // CAS 更新
                return nextc == 0; // 返回是否触发唤醒
        }
    }
}
```

`Sync` 重写了 `AQS` 中的 `tryAcquireShared` 和 `tryReleaseShared` 两个方法。

当调用 `CountDownLatch` 的 `awit()` 方法时，会调用内部类 `Sync` 的 `acquireSharedInterruptibly()` 方法，在这个方法中会调用 `tryAcquireShared` 方法，这个方法就是 `Sync` 重写的 `AQS` 中的方法；

调用 `countDown()` 方法原理基本类似。

#### await() 方法实现

在调用 `await()` 方法时，会直接调用 `AQS` 类的 `acquireSharedInterruptibly` 方法，在 `acquireSharedInterruptibly` 方法内部会继续调用 `Sync` 实现类中的 `tryAcquireShared` 方法，在 `tryAcquireShared` 方法中判断 `state` 变量值是否为 `0`。

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1); // 进入 AQS 核心逻辑
}

// AQS 中的实现
public final void acquireSharedInterruptibly(int arg) {
    if (Thread.interrupted()) throw new InterruptedException();
    if (tryAcquireShared(arg) < 0) // 检查 state 是否为 0
        doAcquireSharedInterruptibly(arg); // 进入阻塞队列
}
```

doAcquireSharedInterruptibly 关键步骤。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192242699.png)

**关键点**：

1. **节点入队**：通过 `addWaiter` 方法将线程封装为 `SHARED` 模式节点加入队列尾部
2. **自旋检查**：循环判断前驱节点是否是头节点（公平性保证）
3. **阻塞控制**：调用 `LockSupport.park()` 挂起线程，响应中断。

#### countDown() 方法

当执行 `CountDownLatch` 的 `countDown()` 方法，将计数器减一，也就是将 `state` 值减一，当减到 0 的时候，等待队列中的线程被释放。是调用 `AQS` 的 `releaseShared()` 方法来实现的。

```java
public void countDown() {
    sync.releaseShared(1); // 触发释放操作
}

// AQS 中的实现
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) { // CAS 递减 state
        doReleaseShared(); // 唤醒后续节点
        return true;
    }
    return false;
}
```

## CyclicBarrier

CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。

它要做的事情是，让一 组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会 开门，所有被屏障拦截的线程才会继续运行。

现实生活中我们经常会遇到这样的情景，在进行某个活动前需要等待人全部都齐了才开始。

例如吃饭时要等全家人都上座了才动筷子，旅游时要等全部人都到齐了才出发，比赛时要等运动员都上场后才开始。

CyclicBarrier 和 CountDownLatch 是不是很像，只是 CyclicBarrier 可以有不止一个栅栏，因为它的栅栏（Barrier）可以重复使用（Cyclic）。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192255855.png)

**CyclicBarrier** 是 Java 并发包中的**可重用同步屏障**，其特性包括：

- **多阶段协同**：支持多次 `await()` 的同步点
- **栅栏动作（Barrier Action）**：当所有线程抵达屏障时触发
- **自动重置**：每次所有线程通过屏障后自动复位
- **中断处理**：可响应线程中断并传播异常

### 如何使用

#### 构造方法

```java
public CyclicBarrier(int parties)
public CyclicBarrier(int parties, Runnable barrierAction)
```

解析：

- parties：传入一个**计数器值**，用来配置可以阻塞多少个线程的。
- 第二个构造方法有一个 Runnable 参数，这个对象可以在计数器值减到 0 后，发起一次调用。

例如：下面代码就会在计数器减到 0 后，打印出"回环屏障退出"。

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> System.out.println("回环屏障退出"));
```

**场景 ：多阶段分布式计算汇总**

**需求**：实时计算商品库存分布（本地仓 + 区域仓 + 全国仓），需三阶段统计数据汇总。

```java
public class InventoryComputeService {
  // 构建 3 线程等待 CyclicBarrier
    private final int PARTIES = 3;
    private final CyclicBarrier barrier = new CyclicBarrier(PARTIES, this::mergeData);
    // 保存最后的结果
    private volatile Map<String, Integer> result = new ConcurrentHashMap<>();

    public void compute() {
       // 异步执行 3 个任务，执行完成调用 barrier.await();，当所有任务完成后会执行 mergeData
        List<CompletableFuture<Void>> tasks = new ArrayList<>();
        tasks.add(computeLocalStock());
        tasks.add(computeRegionalStock());
        tasks.add(computeNationalStock());

        // 本次3 个任务任何一个计算出现异常的话，重置 barrier
        CompletableFuture.allOf(tasks.toArray(new CompletableFuture[0]))
                        .exceptionally(ex -> {
                            barrier.reset();  // 异常处理
                            return null;
                        });
    }

    private CompletableFuture<Void> computeLocalStock() {
      // 异步线程
        return CompletableFuture.runAsync(() -> {
            try {
                // 模拟计算耗时
                result.put("local", calculate(Region.LOCAL));
                // 执行完成，调用 await
                barrier.await(5, TimeUnit.SECONDS); // 超时控制
            } catch (Exception e) {
                handleException(e);
            }
        });
    }

    // computeRegionalStock/computeNationalStock 同理...

    private void mergeData() {
        lock.lock();
        try {
            System.out.println("各区域最终库存合并结果: " + result);
        } finally {
            lock.unlock();
            result.clear(); // 清空状态为下次计算准备
        }
    }
}
```

**代码核心解释**

构造方法创建等待三个线程执行完成的 CyclicBarrier，`CyclicBarrier` 与 `CountDownLatch` 最大的区别是 `CountDownLatch` 一次性的，`CyclicBarrier` 是可循环利用的。

```java
private final CyclicBarrier barrier = new CyclicBarrier(PARTIES, this::mergeData);
```

当三个线程都执行完成，会调用 `mergeData` 方法统计结果。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192320554.png)

### 实现原理

#### 核心数据结构

```java
public class CyclicBarrier {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition trip = lock.newCondition();
    private final int parties;    // 需要同步的线程数
    private final Runnable barrierCommand; // 栅栏动作
    private Generation generation = new Generation(); // 当前代

    private static class Generation {
        boolean broken = false;   // 栅栏是否破裂
    }

    // 挂起线程数计数器（每次循环递减）
    private int count;
}
```

##### CyclicBarrier 状态流转

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192324320.png)

#### await 实现

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

private int dowait(boolean timed, long nanos) throws InterruptedException, BrokenBarrierException, TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
    	// 回环屏障使用完毕或重置后，都会生成一个新的generation，这个对象可以用来让线程退出回环屏障
		final Generation g = generation;

       	// 每个进入的线程，都使计数器减1，当计数器归零后进入下面的if判断
        int index = --count;
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
            	// 如果实例化时传入了Runnable对象，则在这里调用它的run()方法
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                // 里面做了唤醒所有等待线程的操作，线程是在下面的自旋中挂起的
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        for (;;) {
        	// 此处省略的线程被interrupt的try catch
        	// 根据是否传入等待时间来判断调用哪一个方法
	        if (!timed) {
	        	// condition的await()方法，这里会暂时释放锁
	            trip.await();
	        } else if (nanos > 0L) {
			    nanos = trip.awaitNanos(nanos);
			}

			// 计数器归零后，线程退出自旋
			if (g != generation) {
				return index;
			}
        }
    } finally {
        lock.unlock();
    }
}

```

上面代码中的`trip`就是一个 Condition 对象，是`CyclicBarrier`的一个成员变量。总结一下 doWait()方法，其实做的事情还是比较简单的。

线程进入 doWait()， 先抢占到锁 lock 锁对象，并执行计数器递减 1 的操作。
递减后的计数器值不为 0，则将自己挂起在 Condition 队列中。

递减后的计数器值为 0，则调用 signalAll()唤醒所有在条件队列中的线程，并创建新的 generation 对象，让线程可以退出回环屏障。

**核心方法流程图如下。**

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192340898.png)

## Semaphore

`Semaphore`，它是一个信号量，主要作用是用来控制并发中同一个时刻执行的线程数量，可以用来做限流器，或者流程控制器。

在创建的时候会指定好它有多少个信号量，比如 `Semaphre semaphore = new Semaphore(2)`，就只有 2 个信号量。

核心功能是控制同时访问特定资源的线程数量，具有以下特性：

- **许可管理**：通过 `acquire()`/`release()` 操作许可数量
- **公平性选择**：支持公平/非公平两种模式
- **可中断**：支持带超时的许可获取
- **动态调整**：运行时修改许可数量

这个信号量可以比作是车道，每一个时刻每条车道只能允许一辆汽车通过，你可以理解为高速收费站上的收费口，每个收费口任意一时刻只能允许一辆汽车通行。

画个图来讲解一下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192345511.png)

### 如何使用

接口限流（突发流量控制）。

```java
public class ApiRateLimiter {
    // 生产级配置：许可数 = QPS阈值 * 响应时间(秒)
    private static final Semaphore SEMAPHORE = new Semaphore(500);
    private static final Timer METRIC_TIMER = new Timer(true);

    static {
        // 监控线程：每10秒打印许可使用率
        METRIC_TIMER.schedule(new TimerTask() {
            public void run() {
                double usage = (SEMAPHORE.availablePermits() / 500.0) * 100;
                log.info("API许可使用率: {0}%", 100 - usage);
            }
        }, 10_000, 10_000);
    }

    public Response handleRequest(Request request) {
        if (!SEMAPHORE.tryAcquire(50, TimeUnit.MILLISECONDS)) { // 非阻塞获取
            throw new BizException(429, "请求过于频繁");
        }

        try {
            return doBusinessLogic(request); // 核心业务逻辑
        } finally {
            SEMAPHORE.release(); // 确保释放许可
        }
    }
}
```

**生产级要点**：

1. 使用 `tryAcquire` 替代 `acquire` 避免线程阻塞
2. 通过 `finally` 保证许可释放
3. 集成监控上报（Prometheus + Grafana）.

### 实现原理

`Semaphore` 有两种模式，公平模式和非公平模式，分别对应两个内部类为 `FairSync`、`NonfairSync`，这两个子类继承了 `Sync`，都是基于之前讲解过的 `AQS` 来实现的。

#### 核心数据结构

画个图来说明一下内部的结构如下:

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504192345191.png)

`Semaphore` 的公平模式依赖于 `FairSync` 公平同步器来实现，非公平模式依赖于 `NonfairSync` 非公平同步器来实现。

其中 `FairSync`、`NonfairSync` 继承自 `Sync`，而 `Sync` 又继承自 `AQS`，这些同步器的底层都是依赖于 `AQS` 提供的机制来实现的。

```java
public class Semaphore implements java.io.Serializable {
    private final Sync sync;

    abstract static class Sync extends AbstractQueuedSynchronizer {
        Sync(int permits) { setState(permits); }
        final int getPermits() { return getState(); }
        // 非公平尝试获取许可
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 || compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        // 释放许可
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
    }

    // 公平模式实现
    static final class FairSync extends Sync {
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors()) // 检查是否有等待线程
                    return -1;
                // ...与非公平模式相同
            }
        }
    }
}
```

所以掌握 AQS 很重要啊家人们，AQS 是模板方法模式的经典运用。

这里的 `Semaphore` 实现的思路跟我们之前讲过的 `ReentrantLock` 非常的相似，包括内部类的结构都是一样的，也是有公平和非公平两种模式。

只是不同的是 `Semaphore` 是共享锁，支持多个线程同时操作；然而 `ReentrantLock` 是互斥锁，同一个时刻只允许一个线程操作。

#### 公平模式 acquire

公平模式，`Semaphore.acquire` 方法源码直接是调用 `FairSync` 的 `acquireSharedInterruptibly`，也就是进入了 AQS 的 `acquireSharedInterruptibly` 的模板方法里面了。

`java.util.concurrent.Semaphore#acquire()`源码如下。

```java
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

跳入 `AQS` 的 `acquireSharedInterruptibly` 方法。

`java.util.concurrent.locks.AbstractQueuedSynchronizer#acquireSharedInterruptibly`

```java
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted() ||
        // Semaphore.FairSync 子类实现 tryAcquireShared
        (tryAcquireShared(arg) < 0 &&
         acquire(null, arg, true, true, false, 0L) < 0))
        throw new InterruptedException();
}
```

这个方法定义了一个模板流程：

1. 先调用子类的 `tryAcquireShared` 方法获取共享锁，也就是获取信号量。
2. 如果获取信号量成功，即返回值大于等于 0，则直接返回。
3. 如果获取失败，返回值小于 0，则调用 AQS 的 `doAcquireSharedInterruptibly` 方法，进入 AQS 的等待队列里面，等待别人释放资源之后它再去获取。

这里我们画个图理解一下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504200002869.png)

**Semaphore.FairSync 子类实现 tryAcquireShared**

```java
protected int tryAcquireShared(int acquires) {
    for (;;) {
        // 这里作为公平模式，首先判断一下AQS等待队列里面
        // 有没有人在等待获取信号量，如果有人排队了，自己就不去获取了
        if (hasQueuedPredecessors())
            return -1;
        // 获取剩余的信号量资源
        int available = getState();
        // 剩余资源减去我需要的资源，是否小于0
        // 如果小于0则说明资源不够了
        // 如果大于等于0，说明资源是足够我使用的
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

上面的源码就是获取信号量的核心流程了：

1. 首先判断一下 AQS 等待队列里面是否有人在排队，如果是，则自己不尝试获取资源了，乖乖的去排队
2. 如果没有人在排队，获取一下当前剩余的信号量 `available`，然后减去自己需要的信号量 `acquires`，得到减去后的结果 `remaining`
3. 如果 remaining 小于 0，直接返回 `remaining`，说明资源不够，获取失败了，这个时候就会进入 AQS 等待队列等待。
4. 如果 `remaining` 大于等于 0，则执行 CAS 操作 `compareAndSetState` 竞争资源，如果成功了，说明自己获取信号量成功，如果失败了同样进入 AQS 等待队列。

我们画一下公平模式 `FairSync` 的 `tryAcquireShared` 流程图，以及整个公平模式的 acquire 方法的流程图：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504200006168.png)

#### 公平模式 release

看完获取，我们紧接着来看下释放，这里 `Semaphore` 的 `release` 方法直接调用 `Sync` 的 `releaseShared` 方法：

```java
  public void release() {
      sync.releaseShared(1);
  }
```

继续来分析 `releaseShared` 方法，进入到 AQS 的 `releaseShard` 释放资源的模板方法：

```java
public final boolean releaseShared(int arg) {
    // 1. 调用子类的tryReleaseShared释放资源
    if (tryReleaseShared(arg)) {
        // 释放资源成功，调用doReleaseShared唤醒等待队列中等待资源的线程
        doReleaseShared();
        return true;
    }
    return false;
}
```

这里的模板流程有：

1. 调用子类的 `tryReleaseShared` 去释放资源，即释放信号量
2. 如果释放成功了，则调用 `doReleaseShared` 唤醒 AQS 中等待资源的线程，将资源传播下去，如果释放失败，即返回小于等于 0，则直接返回。
3. 所以，这里除了 AQS 的核心模板流程之外，具体释放逻辑就是 `Sync` 的 `tryReleaseShared` 方法的源码了，我们继续来查看：

```java
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        // 这里就是将释放的信号量资源加回去而已
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        // 尝试CAS设置资源，成功直接返回，失败则进入下一循环重试
        if (compareAndSetState(current, next))
            return true;
    }
}
```

释放资源的流程图如下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504200010161.png)

## Exchanger

`Exchanger`（交换者）是一个用于线程间协作的工具类。Exchanger 用于进行线程间的数据交 换。

它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。**这两个线程通过 exchange 方法交换数据，如果第一个线程先执行 exchange()方法，它会一直等待第二个线程也 执行 exchange 方法，当两个线程都到达同步点时，这两个线程就可以交换数据，**将本线程生产 出来的数据传递给对方。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504200023530.png)

### 使用场景

这个玩意的使用场景很少很少……大家对她有个了解即可，大可不必深入。

因为存在很多局限性。

1.  **仅限两个线程**

- 超过两个线程使用同一 `Exchanger` 会导致未定义行为。
- **替代方案**：使用 `CyclicBarrier` 或 `Phaser` 实现多线程同步。

2. **阻塞风险**

- 若一方线程未到达同步点，另一线程会永久阻塞。
- **解决方案**：使用带超时的 `exchange(V x, long timeout, TimeUnit unit)`。

3. **性能瓶颈**

- 频繁交换大数据对象会导致内存和 CPU 开销。
- **优化建议**：交换轻量级对象（如引用或标识符），而非完整数据。

4. **不适用于分布式系统**

- `Exchanger` 仅限单 JVM 内的线程通信。
- **替代方案**：消息队列（如 Kafka）或 RPC 框架（如 gRPC）。

`Exchanger`在多种并发编程场景中都非常有用。例如，在遗传算法中，可以使用`Exchanger`来实现个体之间的信息交换；在管道设计中，可以使用`Exchanger`来传递数据块或任务；在游戏中，可以使用`Exchanger`来实现玩家之间的物品交易等。

如下代码，用 Exchanger 实现两个线程将交换彼此持有的字符串数据：

```java
import java.util.concurrent.Exchanger;

public class ExchangerExample {

    public static void main(String[] args) {
        // 创建一个Exchanger对象
        Exchanger<String> exchanger = new Exchanger<>();

        // 创建一个线程，它将使用"Hello"与另一个线程交换数据
        Thread producer = new Thread(() -> {
            try {
                String producedData = "Hello";
                String consumerData = exchanger.exchange(producedData);
                System.out.println("生产者线程交换后得到的数据: " + consumerData);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "生产者线程");

        // 创建一个线程，它将使用"World"与另一个线程交换数据
        Thread consumer = new Thread(() -> {
            try {
                String consumerData = "World";
                String producedData = exchanger.exchange(consumerData);
                System.out.println("消费者线程交换后得到的数据: " + producedData);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "消费者线程");

        // 启动线程
        producer.start();
        consumer.start();
    }
}
```

代码中我们创建了一个 `Exchanger` 对象，并且定义了两个线程：一个生产者线程和一个消费者线程。

生产者线程持有一个字符串 `"Hello"`，而消费者线程持有一个字符串 `"World"`。两个线程都通过调用 `exchanger.exchange()` 方法来等待交换数据。

当两个线程都到达交换点时（即都调用了 `exchange()` 方法），`Exchanger` 会确保它们安全地交换数据。交换完成后，每个线程都会得到对方原本持有的数据，并打印出来。

### 实现原理

分别从数据结构和 exchange 方法来实现流程来学习实现原理。

#### 核心数据结构

**Participant 线程本地存储**

```java
public class Exchanger<V> {
    // 每个线程持有一个Node
    private final Participant participant;

    static final class Participant extends ThreadLocal<Node> {
        public Node initialValue() { return new Node(); }
    }
}


```

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504201724097.png)

**关键作用**：

- 每个线程通过 `Participant` 持有独立的 `Node` 对象
- 避免多线程竞争同一存储位置
- 底层使用 `ThreadLocal` 实现线程隔离

**Node 交换节点设计**

```java
@sun.misc.Contended // 防止伪共享
static final class Node {
    int index;              // Arena下标
    int bound;              // 最近记录的前导边界
    int collides;           // CAS失败计数
    int hash;               // 伪随机自旋
    Object item;            // 携带的数据
    volatile Object match;  // 交换的数据
    volatile Thread parked; // 挂起的线程
}
```

**内存布局优化**：

- 使用 `@Contended` 注解填充缓存行（64 字节）

- 确保不同线程访问的字段不在同一缓存行

- 示例内存布局：

  ```
  | 64字节缓存行 | Node.item | ...填充... |
  | 64字节缓存行 | Node.match | ...填充... |
  ```

每个线程的 Node 有一个 match 属性用于存储待交换的数据。

#### exchange 方法执行流程

主流程源码（精简版）

```java
public V exchange(V x) throws InterruptedException {
    Object v;
    Node[] a;
    Node q = participant.get();

    // Arena模式（
    if ((a = arena) != null ||
        (q = slotExchange(q, x, false, 0L)) == null)
        return (V)v;
    // ...省略超时处理
}

private final Object slotExchange(Node q, Object x, boolean timed, long nanos) {
    // 核心交换逻辑（
    for (;;) {
        if (slot != null) { // 存在等待节点
            Node node = (Node)slot;
            if (U.compareAndSwapObject(this, SLOT, node, null)) {
                Object v = node.item;
                node.match = x; // 数据交换（
                Thread t = node.parked;
                if (t != null)
                    U.unpark(t); // 唤醒对方线程
                return v;
            }
        } else if (U.compareAndSwapObject(this, SLOT, null, q)) {
            // 挂起当前线程
            return timed ? awaitNanos(q, nanos) : await(q);
        }
    }
}
```

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202504201733983.png)

**关键步骤解释**：

1. **CAS 设置槽位**：`U.compareAndSwapObject(this, SLOT, null, q)`
2. **数据交换**：直接修改对方节点的 `match` 字段
3. **唤醒机制**：通过 `Unsafe.unpark()` 解除线程阻塞

## 最后

作者介绍：大家好呀，我是码哥，《Redis 高手心法》畅销书、公众号「码哥跳动」、InfoQ 签约作者；曾在平安银行担任过架构师，目前在一家做国际旅游的港企 klook 担任技术专家。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/wangzha2.png)