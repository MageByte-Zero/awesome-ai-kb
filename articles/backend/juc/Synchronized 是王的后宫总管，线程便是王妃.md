我是「大王」，是一人之下万人之上的真男人！每次只能有一个王妃获得宠信与我夜夜笙歌。

王妃们会想方设法争宠获得与我共眠的机会，所以我需要一个总管来帮我合理的调度挑选一个「王妃」，他的名字叫 `synchronized`。

假如 `synchronized` 是「王」身边的「大总管」，那么 `Thread` 就像是他后宫的王妃。

今日听「码哥」胡言乱语解开 `synchronized` 大总管如何调度「王妃」获取与王夜夜笙歌的陪伴。

王妃不同状态的变化到底经历了什么？且看 synchronized 大总管又采取了哪些手段更加高效调度一个王妃，宫斗还有 30 秒到达战场！！！

## 故事的开端

「码哥」通过几个故事，通俗易懂的让读者朋友完全掌握 synchronized 的锁优化（偏向锁 -> 轻量级锁 -> 重量级锁）原理以及线程在 6 种状态之间转换的奥秘。

抽象出三个概念：**Thread 对应后宫佳丽「王妃」，synchronized 是后宫大总管负责调度王妃获得与「王」陪伴的机会，「王」则是被王妃们想要竞争的资源。**

当然整个「皇宫」就是 Java JVM 虚拟机。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/juc/20201213112404.png)

## 王妃的 6 种状态

后宫佳丽等级森严，王妃们在这场权贵的游戏中每个人的目的都是为获取「王」宠爱，在这场宫斗剧中自身的状态也随着不同的遭遇有着不同的变化。

「Thread 王妃」的生命周期中一共有 6 种状态。

1. **New（新入宫）**：Thread state for a thread which has not yet started.
2. **Runnable 可运行、就绪**：（身体舒适，准备好了），Java 中的 Runable 状态对应操作系统线程状态中的两种状态，分别是 Running 和 Ready，也就是说，Java 中处于 Runnable 状态的线程有可能正在执行，也有可能没有正在执行，正在等待被分配 CPU 资源。
3. **Blocked 阻塞**（身体欠佳、被打入冷宫、来大姨妈了……）
4. **WAITING（等待）**：（等待传唤）
5. **Timed Waiting（计时等待）**：在门外计时等待
6. **Terminated（终止）**：嗝屁

![线程状态](http://magebyte.oss-cn-shenzhen.aliyuncs.com/juc/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2.png)

王妃在任意时刻只能是其中的一种状态，通过 getState() 方法获取线程状态。

**New 新入宫**

第一日

「王」微服私访，驾车游玩，途径桃花源。见风景宜人，如人间仙境。停车坐爱枫林晚，霜叶红于二月花。

此时此刻，一女子媚眼含羞合，丹唇逐笑开。风卷葡萄带，日照石榴裙。

「王」拟写一份招入宫的诏书，上面写着`new Thread()` ，「香妃 Thread」的名分便正式成立。

New 表示线程被创建但是还没有启动的状态，犹如「香妃」刚刚入宫，等待她后面的路程将是惊心动魄，荡气回肠。

皇宫（可以理解是 JVM）命令 synchronized 大总管为「香妃 Thread」分配寝宫（也就是分配内存），并初始化其身边的「丫鬟」（初始化成员变量的值）。

**Runnable 可运行、就绪**

「香妃 Thread」安排好衣食住行之后，便准备好陪伴王了。

但是后宫佳丽很多，并不是所有人都能获得陪伴权，「香妃」早已准备好，也在争取可以获得与「王」共舞的机会。

便主动告知 synchronized 大总管，自己琴棋书画样样精通，并塞给它一个红包，希望得到安排。

JVM 安排丫鬟为「香妃 Thread」沐浴更衣，抹上胭脂等待召唤（相当于线程的 start() 方法被调用）。

Java 虚拟机会为其创建方法调用栈可程序计数器，等到调度运行，此刻线程就处于可运行状态。

Java 中的 Runable 状态对应操作系统线程状态中的两种状态，分别是 Running 和 Ready，也就是说，Java 中处于 Runnable 状态的线程有可能正在执行，也有可能没有正在执行，正在等待被分配 CPU 资源。

> 注意：启动线程使用 start() 方法，而不是 run() 方法。
>
> 调用 `start()`方法启动线程，系统会把该 run 方法当成方法执行体处理。
>
> 需要切记的是：调用了线程的 `run()`方法之后，该线程就不在处于新建状态，不要再次调用 `start()`方法，只能对新建状态的线程调用`start()`方法，否则会引发 IllegaIThreadStateExccption 异常。

「香妃 Thread」沐浴更衣之后（被调用`start()` ）便焚香抚琴，可「淑妃 Thread」不跟示弱起舞弄影竞争陪伴权。

「香妃 Thread」毕竟新来的，喜新厌旧的渣男「王」甚是宠爱，「香妃 Thread」获得了今晚的陪伴权，获得 JVM 给予 CPU 分片后，执行 `run()`方法，**该方法核心功能就是造娃…..**

在造娃之前，「香妃 Thread」经历了很多纷争，状态也一直在变化。稍有不慎，可能会进入 `TERMINATED` 状态，直接玩完。

请继续阅读……

**Waiting 等待、Timed Waiting 计时等待、Blocked 阻塞**

「淑妃 Thread」也想获得「王」的宠幸。那晚，她想与「王」造娃，王有要事需要处理，synchronized 大总管使用了 `Object.wait()` 技能卡，「香妃」只能等待王回来……

王处理完要事之后，synchronized 大总管使用 `Object.notify()` 解锁，通知「香妃 Thread」可以一起和「王」么么哒了，此刻「香妃 Thread」竟然来大姨妈触发了 `Thread.join()` 只好去上厕所，让老王稍等片刻。

好不容易「香妃 Thread」处于 Runnable 态，但是被总管施展了 `LockSupport.park()` 技能卡，导致无法进入寝宫，状态由 Runnable 变成了 Waiting 态。

「咔妃 Thread」由于太黑了，直接被 synchronized 大总管拒之门外，从 Runnable 变成 Blocked。

还有其他「妃」她们分别被以下技能卡命中进入，直接进入 `TIMED_WAITING` 状态 ，一直等待有机会才能与王夜夜笙歌:

1. Thread.sleep:
2. Object.wait with timeout
3. Thread.join with timeout
4. LockSupport.parkNanos
5. LockSupport.parkUntil

**第二日**

「咔妃」去韩国整容，变白了，得到了 syncronized 大总管的许可，得到一把叫 monitor 的令牌，由原先的 Blocked 变成了 Runnable ……

另外，有的王妃为了获得陪伴权，或者想掌管后宫。阴谋诡计被识破，被判处 Terminated 刑罚，灭顶之灾，强撸灰飞烟灭！

## synchronized 总管如何提升效率翻牌

王妃们除了使用 LockSupport.unpark() 技能卡等获取陪伴权，还可以通过由钦点大总管 synchronized 的令牌陪伴权。

面对三千佳丽，大总管必须要提高效率，不然将会累死而选不出一个王妃去陪伴老王，这可是要杀头的。

因为在 Java 5 版本之前，synchronized 的筛选方法效率很差，一堆王妃跑进来吵着我行我上，秩序混乱，上一任总管就被杀头了……

到了第 6 任 synchronized ，做了很大改善。运用了**自适应自旋、锁消除、锁粗化、轻量级锁、偏向锁**，效率大大提升。

### 自适应自旋

synchronized 通知王妃们过来排队，「王」有急事需要处理，为了让当前申请陪伴的咖妃“稍等一下”， synchronized 大总管会让王妃自旋，少许的等待是值得的，「王」很快就会处理完成事情。

咖妃只需要每隔一段时间询问大总管「王」是否处理好事情，一旦『王』归来，那么自己就不需要进入阻塞态，获得今日与王为伴。

避免因为要去通知多个王妃来竞争费时费力。

用一句话总结自旋锁的好处，**那就是自旋锁用循环去不停地尝试获取锁，让线程始终处于 Runnable 状态，节省了线程状态切换带来的开销。**

以下是自旋与非自旋获取锁的过程：

![自旋与非自旋](http://magebyte.oss-cn-shenzhen.aliyuncs.com/juc/%E8%87%AA%E6%97%8B%E4%B8%8E%E9%9D%9E%E8%87%AA%E6%97%8B.png)

**AtomicInteger**

在 Java 1.5 版本及以上的并发包中，也就是 java.util.concurrent 的包中，里面的原子类基本都是自旋锁的实现。我们看下 AtomicInteger 类的定义：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

    ......
}
```

各属性的作用：

- unsafe： 获取并操作内存的数据。
- valueOffset： 存储 value 在 AtomicInteger 中的偏移量。
- value： 存储 AtomicInteger 的 int 值，该属性需要借助 volatile 关键字保证其在线程间是可见的。

查看 AtomicInteger 的自增函数 incrementAndGet() 的源码时，自增函数底层调用的是 unsafe.getAndAddInt()。

但是由于 JDK 本身只有 Unsafe.class，只通过 class 文件中的参数名，并不能很好的了解方法的作用，我们通过 OpenJDK 8 来查看 Unsafe 的源码：

```java
// JDK AtomicInteger 自增
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

// OpenJDK 8
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```

通过 do while 实现了自旋，getAndAddInt() 循环获取给定对象 o 中的偏移量处的值 v，然后判断内存值是否等于 v。

如果相等则将内存值设置为 v + delta，否则返回 false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。

整个“比较 + 更新”操作封装在 compareAndSwapInt() 中，在 JNI 里是借助于一个 CPU 指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

synchronized 大总管在 1.6 版本想出了自适应自旋锁来解决长时间自旋的问题，防止一直傻傻等待傻傻的问。会根据最近自旋的成功率、失败率。

如果最近尝试自旋获取某一把锁成功了，那么下一次可能还会继续使用自旋，并且允许自旋更长的时间；

但是如果最近自旋获取某一把锁失败了，那么可能会省略掉自旋的过程，以便减少无用的自旋，提高效率。

### 锁消除

淑妃诡诈，在公元 107 年一个夜黑风高的夜晚，串通后厨小哲子放了无色无味的黯然销魂药，妃子们有气无力。

所以只剩下自己一个人向 synchronized 大总管申请与「王」共眠的机会，不需要繁琐流程，直捣黄龙。**直接面见老王，无需加锁。**

**锁消除即删除不必要的加锁操作**。虚拟机即时编辑器在运行时，对一些“代码上要求同步，但是被检测到不可能存在共享数据竞争”的锁进行消除。

根据代码逃逸技术，如果判断到一段代码中，堆上的数据不会逃逸出当前线程，那么可以认为这段代码是线程安全的，不必要加锁。

```java
public class SynchronizedTest {

    public static void main(String[] args) {
        SynchronizedTest test = new SynchronizedTest();

        for (int i = 0; i < 100000000; i++) {
            test.append("码哥字节", "def");
        }
    }

    public void append(String str1, String str2) {
        StringBuffer sb = new StringBuffer();
        sb.append(str1).append(str2);
    }
}
```

虽然 StringBuffer 的 append 是一个同步方法，但是这段程序中的 StringBuffer 属于一个局部变量，并且不会从该方法中逃逸出去（即 StringBuffer sb 的引用没有传递到该方法外，不可能被其他线程拿到该引用），所以其实这过程是线程安全的，可以将锁消除。

### 锁粗化

「甄嬛 Thread」深得老王宠爱，被偏爱的都有恃无恐。

每次进出 synchronized 大总管的大门都需要验证是否获得 monitor 锁，甄嬛进来后还喜欢出去外面走走过一会有进来看几眼老王又出去，大总管也不用每次都要验证，将限制的范围就加大了，防止反复验证。

**如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体中的，那即使没有出现线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。**

如果虚拟机检测到有一串零碎的操作都是对同一对象的加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部。

```java
public class StringBufferTest {
    StringBuffer stringBuffer = new StringBuffer();

    public void append(){
        stringBuffer.append("关注");
        stringBuffer.append("公众号");
        stringBuffer.append("码哥字节");
    }
}
```

每次调用 stringBuffer.append 方法都需要加锁和解锁，如果虚拟机检测到有一系列连串的对同一个对象加锁和解锁操作，就会将其合并成一次范围更大的加锁和解锁操作，即在第一次 append 方法时进行加锁，最后一次 append 方法结束后进行解锁。

### 偏向锁/轻量级锁/重量级锁

**偏向锁**

老王偏爱「甄嬛」，synchronized 大总管便在一个叫 Mark Word 里柜子存储锁偏向的线程 ID，记录着甄嬛的 ID，不需要执行繁琐的翻牌流程。

只需要判断下申请的王妃 ID 是否跟柜子里记录的 ID 一致。

当一个线程访问同步代码块并获取锁时，会在 Mark Word 里存储锁偏向的线程 ID。

在线程进入和退出同步块时不再通过 CAS 操作来加锁和解锁，而是检测 Mark Word 里是否存储着指向当前线程的偏向锁。

引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次 CAS 原子指令，而偏向锁只需要在置换 ThreadID 的时候依赖一次 CAS 原子指令即可。

偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁。

偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态。

撤销偏向锁后恢复到无锁（标志位为“01”）或轻量级锁（标志位为“00”）的状态。

偏向锁在 JDK 6 及以后的 JVM 里是默认启用的。可以通过 JVM 参数关闭偏向锁：`-XX:-UseBiasedLocking=false`，关闭之后程序默认会进入轻量级锁状态。

**轻量级锁**

是指当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，从而提高性能。

1. 在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的 Mark Word 的拷贝，官方称之为 Displaced Mark Word。这时候线程堆栈与对象头的状态如下图所示。

![轻量级锁](http://magebyte.oss-cn-shenzhen.aliyuncs.com/juc/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81.png)

2. 拷贝 Object 对象头中的 Mark Word 复制到 LockRecord 中。
3. 拷贝成功后，虚拟机将使用 CAS 操作尝试将对象的 Mark Word 更新为指向 Lock Record 的指针，并将 Lock record 里的 owne r 指针指向 object mark word。如果更新成功，则执行步骤 4。
4. 如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象 Mark Word 的锁标志位设置为“00”，即表示此对象处于轻量级锁定状态，这时候线程堆栈与对象头的状态如下图所示。![](http://magebyte.oss-cn-shenzhen.aliyuncs.com/juc/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81%E5%8A%A0%E9%94%81.png)
5. 如果这个更新操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，若当前只有一个等待线程，则可通过自旋稍微等待一下，可能另一个线程很快就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止 CPU 空转，锁标志的状态值变为“10”，Mark Word 中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态。

**重量级锁**

如上轻量级锁的加锁过程步骤（5），轻量级锁所适应的场景是线程近乎交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。Mark Word 的锁标记位更新为 10，Mark Word 指向互斥量（重量级锁）

Synchronized 的重量级锁是通过对象内部的一个叫做监视器锁（monitor）来实现的，监视器锁本质又是依赖于底层的操作系统的 Mutex Lock（互斥锁）来实现的。

而操作系统实现线程之间的切换需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么 Synchronized 效率低的原因。

**锁升级路径**

从无锁到偏向锁，再到轻量级锁，最后到重量级锁。结合前面我们讲过的知识，偏向锁性能最好，避免了 CAS 操作。

而轻量级锁利用自旋和 CAS 避免了重量级锁带来的线程阻塞和唤醒，性能中等。

重量级锁则会把获取不到锁的线程阻塞，性能最差。

![锁升级](http://magebyte.oss-cn-shenzhen.aliyuncs.com/juc/%E9%94%81%E5%8D%87%E7%BA%A7%E8%B7%AF%E5%BE%84.png)

**综上，偏向锁通过对比 Mark Word 解决加锁问题，避免执行 CAS 操作。而轻量级锁是通过用 CAS 操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。**

读者群已开通，加我微信备注加群，加入专属技术群，获取更多成长！