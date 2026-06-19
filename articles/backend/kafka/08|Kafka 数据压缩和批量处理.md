[《Kafka 高性能架构设计 7 大秘诀》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzMDI1NjcyOQ==&action=getalbum&album_id=3485080895185338375&from_itemidx=2&from_msgid=2247503469#wechat_redirect)专栏第 6 章。

压缩，是一种用时间换空间的 trade-off 思想，用 CPU 的时间去换磁盘或者网络 I/O 传输量，用较小的 CPU 开销来换区更具性价比的磁盘占用和更少的网络 I/O 传输。

Kafka 是一个高吞吐量、可扩展的分布式消息系统，深入掌握 Kafka 的数据压缩和批量数据处理机制，对于优化系统性能和资源使用至关重要。

## 目录

1. [Kafka 高性能秘诀之零拷贝技术](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503458&idx=1&sn=c5cf7a30b96abadaf4497a73f27ec219&chksm=c27f8c54f50805420129980d78b5671fc5d0b7fdcfab259d5249598d88aa0bb2224419311e64&scene=178&cur_album_id=3485080895185338375#rd)
2. [Kafka 高性能 7 大秘诀之 Reactor 网络 I/O模型](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503469&idx=1&sn=fc6dea24311b1b5b6d2bf0f6b58673f3&chksm=c27f8c5bf508054df0fad0f0cc3a4cc6d169e35e13a6e04dd624bdd33d7aba2fea9645c718b5&scene=178&cur_album_id=3485080895185338375#rd)
3. [Kafka 高性能之 Segment 文件结构和磁盘顺序写](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503498&idx=1&sn=9635b58b70c76401e42a2e113dc9a378&chksm=c27f8cbcf50805aa179a504559f23dd28b9e3b50f02eccdb8f3588122d0ee0a7a804e892eee2&scene=178&cur_album_id=3485080895185338375#rd) 
4. [Kafka 高性能之 Patition 分区并发](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503486&idx=1&sn=fd9f7d8f63dad251187f9081edc7cb49&chksm=c27f8c48f508055efbe843d42d1c25d7111090b81d3b377d5999cff7f0a1293a5dd1656bdef9&scene=178&cur_album_id=3485080895185338375#rd)
5. [Kafka 高性能之 PageCache 的妙用](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503509&idx=1&sn=f5d3069534085a2dc5e723571bb2f602&chksm=c27f8ca3f50805b5f38b5ea306dabd769e18530175ee21c61e72f448a3a403d9244f215bc4a2&scene=178&cur_album_id=3485080895185338375#rd)
6. Kafka 高性能之压缩和批量
7. Kafka 高性能之轻量级无锁 offset——待更新
8. Kafka 生产高性能运行的关键配置——待更新

## Kafka 数据压缩机制

数据压缩在 Kafka 中有助于减少磁盘空间的使用和网络带宽的消耗，从而提升整体性能。

**通过减少消息的大小，压缩可以显著降低生产者和消费者之间的数据传输时间。**

> Chaya：Kafka 支持的压缩算法有哪些？

在Kafka 2.1.0版本之前，Kafka支持3种压缩算法：GZIP、Snappy和LZ4。从2.1.0开始，Kafka正式支持Zstandard算法（简写为zstd）。

> Chaya：这么多压缩算法，我如何选择？

一个压缩算法的优劣，有两个重要的指标：压缩比，文件压缩前的大小与压缩后的大小之比，比如源文件占用 1000 M 内存，经过压缩后变成了 200 M，压缩比 = 1000 /200 = 5，压缩比越高越高；另一个指标是压缩/解压缩吞吐量，比如每秒能压缩或者解压缩多少 M 数据，吞吐量越高越好。

如下图是Facebook Zstandard官网提供的一份压缩算法benchmark比较结果：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202407111100486.png)

从图中可以看到，ZSTD 压缩比最高，但是吞吐量中规中矩。LZ4 在吞吐量方面属于王者。

- **GZIP**：压缩比高，但压缩和解压缩速度相对较慢。适用于对传输带宽要求较高的场景。
- **Snappy**：由 Google 开发，压缩和解压缩速度快，但压缩比相对较低。适用于对性能要求较高的场景。
- **LZ4**：在压缩和解压缩速度以及压缩比之间取得良好平衡。适用于对性能和压缩比有综合需求的场景。
- **ZSTD**：由 Facebook 开发，提供高压缩比和较快的压缩解压速度。适用于对高效压缩和快速处理都有需求的场景。

在 Kafka 的性能测试结果中，不同压缩算法的两个指标有以下排序特点。

- 吞吐量方面：LZ4 > Snappy > zstd和GZIP；
- 压缩比方面：zstd > LZ4 > GZIP > Snappy。

### 何时压缩

> Chaya：我觉得可以在生产者和 Broker 端进行压缩，对么？

在生产者端压缩是很自然的想法，**大部分情况下 Broker 收到 Producer 端的消息后是原封不动的保存，并不会进行压缩**。

#### 生产者压缩

Kafka 的数据压缩主要在生产者端进行。具体步骤如下：

1. **生产者配置压缩方式**：在 KafkaProducer 配置中设置 `compression.type` 参数，可以选择 `gzip`、`snappy`、`lz4` 或 `zstd`。
2. **消息压缩**：生产者将消息批量收集到一个 `batch` 中，然后对整个 `batch` 进行压缩。这种批量压缩方式可以获得更高的压缩率。
3. **压缩消息存储**：压缩后的 `batch` 以压缩格式存储在 Kafka 的主题（Topic）分区中。
4. **消费者解压缩**：消费者从 Kafka 主题中获取消息时，首先对接收到的 `batch` 进行解压缩，然后处理其中的每一条消息。

以下是一个配置 KafkaProducer 使用 LZ4 压缩的示例：

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
// 开启 LZ4 压缩
props.put("compression.type", "lz4");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("my-topic", "key", "value"));
producer.close();
```

有两种例外情况可能让 Broker 重新压缩消息。

#### Broker 指定了与 Producer 不同的压缩算法

第一种情况是 Broker 指定了与 Producer 不一样的压缩算法。

Producer 指定了 GZIP 压缩，但是 Broker 指定使用 Snappy 算法压缩。

这个时候，Broker 接受到使用 GZIP 压缩的消息后，只能先解压缩，再使用 Snappy 压缩一遍。

Broker端也有一个参数叫`compression.type`，该参数的默认值是 `producer`，表示 Broker 会使用 Producer 的压缩算法。但是如果你在 Broker 设置了不同的 `compression.type`值，能会发生预料之外的压缩/解压缩操作，通常表现为Broker端CPU使用率飙升。

#### Broker 发生了消息格式转换

另一种情况是Broker 发生了消息格式转换。所谓的消息格式转换主要是为了兼容老版本的消费者程序。

在一个生产环境中，Kafka集群中同时保存多种版本的消息格式非常常见。为了兼容老版本的格式，Broker端会对新版本消息执行向老版本格式的转换。这个过程中会涉及消息的解压缩和重新压缩。

一般情况下这种消息格式转换对性能是有很大影响的，除了这里的压缩之外，它还让Kafka丧失了引以为豪的Zero Copy特性。

关于零拷贝，详见专栏的[《Kafka 高性能 7 大秘诀之零拷贝技术》](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247503458&idx=1&sn=c5cf7a30b96abadaf4497a73f27ec219&chksm=c27f8c54f50805420129980d78b5671fc5d0b7fdcfab259d5249598d88aa0bb2224419311e64&scene=178&cur_album_id=3485080895185338375#rd)篇章。

### 解压缩

有压缩，那必有解压缩。通常情况下，Producer 发送压缩后的消息到 Broker ，原样保存起来。

Consumer 消费这些消息的时候，Broker 原样发给 Consumer，由 Consumer 执行解压缩还原出原本的信息。

> Chaya：Consumer 咋知道用什么压缩算法解压缩？

Kafka会将启用了哪种压缩算法封装进消息集合中，这样当Consumer读取到消息集合时，它自然就知道了这些消息使用的是哪种压缩算法。

总之一句话：**Producer端压缩、Broker端保持、Consumer端解压缩。**

## 批量数据处理

Kafka Producer 向 Broker 发送消息不是一条消息一条消息的发送，将多条消息打包成一个批次发送。

批量数据处理可以显著提高 Kafka 的吞吐量并减少网络开销。

使用过 Kafka 的同学应该知道，Producer 有两个重要的参数：`batch.size`和`linger.ms`。这两个参数就和 Producer 的批量发送有关。

Kafka Producer 的执行流程如下图所示：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/kafka_list/producer.png)

发送消息依次经过以下处理器：

- Serialize：键和值都根据传递的序列化器进行序列化。优秀的序列化方式可以提高网络传输的效率。
- Partition：决定将消息写入主题的哪个分区，默认情况下遵循 murmur2 算法。自定义分区程序也可以传递给生产者，以控制应将消息写入哪个分区。
- Compression：默认情况下，在 Kafka 生产者中不启用压缩。Compression 不仅可以更快地从生产者传输到代理，还可以在复制过程中进行更快的传输。压缩有助于提高吞吐量，降低延迟并提高磁盘利用率。
- Record Accumulator：`Accumulate`顾名思义，就是一个消息累计器。其内部为每个 Partition 维护一个`Deque`双端队列，队列保存将要发送的 Batch**批次数据**，`Accumulate`将数据累计到一定数量，或者在一定过期时间内，便将数据以批次的方式发送出去。记录被累积在主题每个分区的缓冲区中。根据生产者批次大小属性将记录分组。主题中的每个分区都有一个单独的累加器 / 缓冲区。
- Group Send：记录累积器中分区的批次按将它们发送到的代理分组。 批处理中的记录基于 `batch.size` 和 `linger.ms` 属性发送到代理。 记录由生产者根据两个条件发送。 当达到定义的批次大小或达到定义的延迟时间时。
- Send Thread：发送线程，从 Accumulator 的队列取出待发送的 Batch 批次消息发送到 Broker。
- Broker 端处理：Kafka Broker 接收到 `batch` 后，将其存储在对应的主题分区中。
- **消费者端的批量消费**：消费者可以配置一次拉取多条消息的数量，通过 `fetch.min.bytes` 和 `fetch.max.wait.ms` 参数控制批量大小和等待时间。

### 批量发送和批量消费示例

以下是一个配置 KafkaProducer 进行批量处理的示例：

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("batch.size", 16384); // 批量大小，单位字节
props.put("linger.ms", 10); // 最长等待时间，单位毫秒

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
for (int i = 0; i < 100; i++) {
    producer.send(new ProducerRecord<>("my-topic", "key" + i, "value" + i));
}
producer.close();

```

消费者端的批量消费配置示例如下：

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "test");
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "1000");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("fetch.min.bytes", 1024); // 最小批量大小，单位字节
props.put("fetch.max.wait.ms", 500); // 最大等待时间，单位毫秒

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
    }
}

```

**博主简介**

码哥，9 年互联网后端工作经验，后端架构师。InfoQ 签约作者、51CTO Top 红人，阿里云开发者社区专家博主，擅长 Redis、Spring、Kafka、MySQL 技术和云原生微服务技术。























