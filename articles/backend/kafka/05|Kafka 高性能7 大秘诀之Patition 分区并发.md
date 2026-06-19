《[**Kafka 高性能提升的 7 大秘诀**](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzkzMDI1NjcyOQ==&scene=1&album_id=3485080895185338375&count=3#wechat_redirect)》系列已经更新篇章如下：

1. [Kafka 高性能 7 大秘诀之零拷贝技术](https://mp.weixin.qq.com/s/nc4Teoqy7NzDNup17jhVPw)
2. [Kafka 高性能 7 大秘诀之 Reactor I/O模型](https://mp.weixin.qq.com/s/wg1-Lz4GrYLf1JYa9KUCxw)

在这篇文章中，我将深入解析 Kafka 高性能 7 大秘诀之 Topic patition 分区并发架构设计。

## kafka 架构

在说 Topic patition 分区并发之前，我们先了解下 kafka 架构设计。一个典型的 Kafka 架构包含以下几个重要组件，如图 1 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202405262242482.png)

图 1

1. **Producer（生产者）**：发送消息的一方，负责发布消息到 Kafka 主题（Topic）。

2. **Consumer（消费者）**：接受消息的一方，订阅主题并处理消息。Kafka 有ConsumerGroup 的概念，每个Consumer 只能消费所分配到的 Partition 的消息，每一个Partition只能被一个ConsumerGroup 中的一个Consumer 所消费，所以同一个ConsumerGroup 中Consumer 的数量如果超过了Partiton 的数量，将会出现有些Consumer 分配不到 partition 消费。

3. **Broker（代理）**：服务代理节点，Kafka 集群中的一台服务器就是一个 broker，可以水平无限扩展，**同一个 Topic 的消息可以分布在多个 broker 中**。

4. **Topic（主题）与 Partition（分区）** ：Kafka 中的消息以 Topic 为单位进行划分，生产者将消息发送到特定的 Topic，而消费者负责订阅 Topic 的消息并进行消费。图中 TopicA 有三个 Partiton（TopicA-par0、TopicA-par1、TopicA-par2）

   为了提升整个集群的吞吐量，Topic 在物理上还可以细分多个Partition，一个 Partition 在磁盘上对应一个文件夹。

5. **Replica（副本）**：副本，是 Kafka 保证数据高可用的方式，Kafka **同一 Partition 的数据可以在多 Broker 上存在多个副本**，通常只有 leader 副本对外提供读写服务，当 leader副本所在 broker 崩溃或发生网络一场，Kafka 会在 Controller 的管理下会重新选择新的 Leader 副本对外提供读写服务。

6. **ZooKeeper**：管理 Kafka 集群的元数据和分布式协调。



## Topic 主题

Topic 是 Kafka 中数据的逻辑分类单元，可以理解成一个队列。Broker 是所有队列部署的机器，Producer 将消息发送到特定的 Topic，而 Consumer 则从特定的 Topic 中消费消息，如图 2 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202405262245581.png)

图 2

也就是说Kafka的消息组织方式实际上是三级结构：主题-分区-消息。主题下的每条消息只会保存在某一个分区中，而不会在多个分区中被保存多份。Topic、Partition、Broker 的关系如图3 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406102059373.png)

图 3



## Partition 

为了提高并行处理能力和扩展性，Kafka 将一个 Topic 分为多个 Partition。每个 Partition 是一个有序的消息队列，消息在 Partition 内部是有序的，但在不同的 Partition 之间没有顺序保证。

Producer 可以并行地将消息发送到不同的 Partition，Consumer 也可以并行地消费不同的 Partition，从而提升整体处理能力，如图 4 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/kafka_list/partition.png)

图 4

因此，可以说，**每增加一个 Paritition 就增加了一个消费并发。Partition的引入不仅提高了系统的可扩展性，还使得数据处理更加灵活。**

每个Partition可以分布在不同的Broker上，实现数据的并行处理。在大数据处理场景下，通过增加Partition数量，Kafka可以横向扩展，增加处理节点，以应对不断增长的数据量和处理需求。

> 码楼：“那是不是 partition 数越多越好呢？”

当然不是。**越多的分区需要打开更多的文件句柄**。在 kafka 的 broker 中，每个分区都会对照着文件系统的一个目录。

在 kafka 的数据日志文件目录中，每个日志数据段都会分配两个文件，一个索引文件和一个数据文件。

因此，随着 partition 的增多，需要的文件句柄数急剧增加，必要时需要调整操作系统允许打开的文件句柄数。

此外，客户端 producer 有个参数 batch.size，默认是 16KB。它会为每个partition 缓存消息，一旦满了就将消息打包批量发出。

**因为这个参数是partition级别的，如果 partition 越多，这部分缓存所需的内存占用也会更多。**

另外，partition 越多，每个 Broker 上分配的partition 也就越多，当一个发生 Broker 宕机，那么恢复时间将很长。

## Partition 分区策略

总结下：**Broker 保存了 Topic，每个 Topic 有多个 partition。消息存储在 Partition 中，每个 Partition 有多个副本，主副本（Leader）负责读写操作，其它副本（Follower）定期从 Leader 同步数据。当 Leader 发生故障时，会从 Follower 中选举新的 Leader。**

> 码楼：“生产者将消息发送到哪个分区是如何实现的？不合理的分配会导致消息集中在某些 Broker 上，岂不是完犊子。”

主要有以下几种分区策略：

1. **轮询策略**：也称Round-robin策略，即顺序分配。
2. **随机策略**：也称Randomness策略。所谓随机就是我们随意地将消息放置到任意一个分区上。
3. **按消息键保序策略**。
4. 基于地理位置分区策略。

###  轮询策略

比如一个 Topic 下有 3个分区，那么第一条消息被发送到分区0，第二条被发送到分区1，第三条被发送到分区2，以此类推。

当生产第4条消息时又会重新开始，即将其分配到分区0，如图 5 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406102114446.png)

图 5

**轮询策略有非常优秀的负载均衡表现，它总是能保证消息最大限度地被平均分配到所有分区上，故默认情况下它是最合理的分区策略，也是我们最常用的分区策略之一。**

### 随机策略

所谓随机就是我们随意地将消息放置到任意一个分区上。如图 6 所示，9 条消息随机分配到不同分区。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406102115575.png)

图 6

随机策略也是力求将数据均匀地打散到各个分区，但从实际表现来看，它要逊于轮询策略，所以**如果追求数据的均匀分布，还是使用轮询策略比较好**。

### 按消息键分配策略

一旦消息被定义了 Key，那么你就可以保证同一个 Key 的所有消息都进入到相同的分区里面，比如订单 ID，那么绑定同一个 订单 ID 的消息都会发布到同一个分区，由于每个分区下的消息处理都是有顺序的，故这个策略被称为按消息键保序策略，如图 7所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406102117110.png)

图 7

### 基于地理位置

这种策略一般只针对那些大规模的 Kafka 集群，特别是跨城市、跨国家甚至是跨大洲的集群。

我们就可以根据 Broker 所在的 IP 地址实现定制化的分区策略。比如下面这段代码：

```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return partitions.stream()
  .filter(p -> isSouth(p.leader().host()))
  .map(PartitionInfo::partition)
  .findAny()
  .get();
```

我们可以从所有分区中找出那些Leader副本在南方的所有分区，然后随机挑选一个进行消息发送。