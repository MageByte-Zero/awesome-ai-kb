[《Kafka 高性能 7 大秘诀架构设计》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzMDI1NjcyOQ==&action=getalbum&album_id=3485080895185338375&from_itemidx=1&from_msgid=2247503469#wechat_redirect)系列第 5 弹《kafka高性能之 Page Cache 的应用哲学》，在说 Page Cache 之前，先回顾下上一篇[《Kafka 高性能之 Segment 消息存储机制的奥妙》](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503498&idx=1&sn=9635b58b70c76401e42a2e113dc9a378&chksm=c27f8cbcf50805aa179a504559f23dd28b9e3b50f02eccdb8f3588122d0ee0a7a804e892eee2&scene=178&cur_album_id=3485080895185338375#rd)。码哥补充一些关于稀疏索引的设计哲学，给大家加餐。

## kafka 消息存储设计

Kafka 的消息存储会按照该 Topic 的 Partition 进行保存，即每个Partition都有属于自己的日志，在 Kafka 中被称为分区日志（partition log）。

每条消息在发送前会根据负载均衡策略计算出要发往的目标 Partition 中，broker 收到消息之后把该条消息按照**追加的方式顺序写入**对应Partition的日志文件中，充分了利用磁盘顺序写访问快的特性。如图 1 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406231256012.png)

图 1

从图 2 可以看到磁盘顺序写的性能远高于磁盘随机写，甚至比内存随机写还快。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406231343790.png)

图 2，引自《武哥漫谈 IT》

> Chaya：“具体的存储文件有哪些组成的？”

如果每个 partition 对应一个日志文件，文件可能会变得很大，对于消息的过期清除和检索都是一个大难题，因此 Kafka 会将每个分区的日志文件继续细分成若干个日志文件，这些日志文件也称作日志段文件（segment file），每个日志段文件都会伴随一个索引文件和时间戳索引文件。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/kafka_list/log.png)

### log 文件

.log 后缀文件保存了 Kafka 消息的记录，而且每个 log 文件都有对应的消息记录范围，名字的数字代表了消息记录的初始位移值，并且随着消息数量的增多而增大，因此，每个新创建的分区一定会包含 0 的 log 文件。

### 索引文件

每个 log 文件都会包含两个索引文件，分别是 .index 和 .timeindex，在 Kafka 中它们分别被称为**位移索引文件和时间戳索引文件**，位移索引文件可根据消息的位移值快速地从查询到消息的物理文件位置，时间戳索引文件可根据时间戳查找到对应的位移信息。

### 稀疏索引

> Chaya：“为什么不创建一个哈希索引，从 offset 到物理消息日志文件偏移量的映射关系？”

万万不可，Kafka 作为海量数据处理的中间件，每秒高达几百万的消息写入，这个哈希索引会把把内存撑爆炸。

稀疏索引不会为每个记录都保存索引，而是写入一定的记录之后才会增加一个索引值，具体这个间隔有多大则通过 `log.index.interval.bytes` 参数进行控制，默认大小为 4 KB，意味着 Kafka 至少写入 4KB 消息数据之后，才会在索引文件中增加一个索引项。

哈希稀疏索引把消息划分为多个 block ，只索引每个 block 第一条消息的 offset 即可 。稀疏哈希索引如图 3 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406231537541.png)

图 3

有了稀疏索引，当给定一个 offset 时，Kafka 采用的是二分查找来扫描索引定位不大于 offset 的物理位移 position，再到日志文件找到目标消息。

利用稀疏索引，已经基本解决了高效查询的问题，但是这个过程中仍然有进一步的优化空间，那便是**通过 mmap(memory mapped files) 读写上面提到的稀疏索引文件，进一步提高查询消息的速度**。

就是基于 JDK nio 包下的 MappedByteBuffer 的 map 函数，将磁盘文件映射到内存中。

进程通过调用mmap系统函数，将文件或物理内存的一部分映射到其虚拟地址空间。这个过程中，操作系统会为映射的内存区域分配一个虚拟地址，并将这个地址与文件或物理内存的实际内容关联起来。

一旦内存映射完成，进程就可以通过指针直接访问映射的内存区域。这种访问方式就像访问普通内存一样简单和高效。如图 4。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406231607064.png)

图 4，引自《码农的荒岛求生》

这样，Kafka可以快速定位消息，提升读取性能。同时，顺序写入的方式使得磁盘写操作更加高效，减少了寻道时间和旋转延迟。

## Page Cache

> Chaya：“码哥，使用稀疏索引和 mmap 内存映射技术提高读消息的性能；Topic Partition 加磁盘顺序写持久化消息的设计已经很快了，但是与内存顺序写还是慢了，还有优化空间么？”

小姑娘，你的想法很好，作为快到令人发指的 Kafka，确实想到了一个方式来提高读写写磁盘文件的性能。这就是接下来的主角 Page Cache 。

简而言之：利用操作系统的缓存技术，在读写磁盘日志文件时，操作的是内存，而不是文件，由操作系统决定什么在某个时间将 Page Cache 的数据刷写到磁盘中。如图 5 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/kafka_list/pagecache.png)

图 5

1. Producer 发送消息到 Broker 时，Broker 会使用 `pwrite()` 系统调用写入数据，此时数据都会先写入`page cache`。
2. Consumer 消费消息时，Broker 使用 `sendfile()` 系统调用函数，利用零拷贝技术地将 Page Cache 中的数据传输到 broker 的 Socket buffer，再通过网络传输到 Consumer。
3. leader 与 follower 之间的同步，与上面 consumer 消费数据的过程是同理的。

`page cache`中的数据会随着内核中 flusher 线程的调度以及对 `sync()/fsync() `的调用写回到磁盘，就算进程崩溃，也不用担心数据丢失。

Kafka重度依赖底层操作系统提供的PageCache功能。当上层有写操作时，操作系统只是将数据写入PageCache，同时标记Page属性为Dirty。

当读操作发生时，先从PageCache中查找，如果发生缺页才进行磁盘调度，最终返回需要的数据。如图 6 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406231634022.png)

图 6

对于Produce请求：Server端的I/O线程统一将请求中的数据写入到操作系统的PageCache后立即返回，当消息条数到达一定阈值后，Kafka应用本身或操作系统内核会触发强制刷盘操作（如左侧流程图所示）。

对于Consume请求：主要利用了操作系统的ZeroCopy机制，当Kafka Broker接收到读数据请求时，会向操作系统发送sendfile系统调用，操作系统接收后，首先试图从PageCache中获取数据（如中间流程图所示）；

如果数据不存在，会触发缺页异常中断将数据从磁盘读入到临时缓冲区中（如右侧流程图所示），随后将数据拷贝到网卡缓冲区中等待后续的TCP传输（数据拷贝利用DMA操作减少拷贝次数和上下文切换）。

于是我们得到一个重要结论：**如果Kafka producer的生产速率与consumer的消费速率相差不大，那么就能几乎只靠对broker page cache的读写完成整个生产-消费过程，磁盘访问非常少。**

**实际上PageCache是把尽可能多的空闲内存都当做了磁盘缓存来使用。**

> Chaya：“Kafka为什么不自己管理缓存，而非要用操作系统系统的 Page Cache？”

做大事者，必须要充分利用资源，达到最优解。原因有如下三点。

1. 如果由JVM来管理缓存，JVM的GC线程会频繁扫描Heap空间，带来不必要的开销。如果Heap过大，执行一次Full GC对系统的可用性来说将是极大的挑战。
2. 为了防止内存中的数据随着kafka重启或者崩溃而丢失，因此Kafka将消息数据存储在PageCache中，而在遇到上述状况，Kafka重启后，OS管理的PageCache依然可以继续使用。
3. JVM中一切皆对象，数据的对象存储会带来所谓object overhead，浪费空间。

## 总结

Kafka在运行过程中，会尽可能多的把空闲内存都当做了磁盘缓存来使用。

当上层有写操作时，操作系统只是将数据写入PageCache，同时标记Page属性为Dirty。

当读操作发生时，先从PageCache中查找，如果发生缺页才进行磁盘调度，最终返回需要的数据。

同时如果有其他进程申请内存，会回收抢占一部分PageCache，但也会导致Kafka吞吐量下降会不稳定。这个我们做过相应的测试工作。

Kafka使用PageCache功能同时可以避免在JVM内部缓存数据

Kafka重启后，OS管理的PageCache不会被释放，依然可以继续使用。

Kafka的刷盘机制，会将PageCache中的消息数据刷到磁盘当中，保证数据不会丢失。



**博主简介**

码哥，9 年互联网后端工作经验，目前担任后端架构师。InfoQ 签约作者、51CTO Top 红人，阿里云开发者社区专家博主，擅长 Redis、Spring、Kafka、MySQL 技术和云原生微服务。

**购买权益**

目前活动价 **10** 元，一次买断。购买后除了可以学习知识以外，**重点是提供以下服务**。

1. 享受 1 年内的 VIP 沟通交流服务，对于问题的任何疑问，都可以随时与我沟通进行答疑，直到理解为止。
2. 享受 1 年内未来我的所有付费专栏的 7 折购买优惠。
3. 享受 1 年内一次面试模拟服务（1 小时）。
4. 享受 1 年内一次简历修改优化服务。

加我微信，回复 Kafka，进入 Kafka 专属陪伴群。

