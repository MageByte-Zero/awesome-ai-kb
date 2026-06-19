在[《Redis 核心篇：唯快不破的秘密》](https://mp.weixin.qq.com/s/8HN1PqqU57Kdz9ERwDY2cw)中，「码哥」揭秘了 Redis 五大数据类型底层的数据结构、IO 模型、线程模型、渐进式 rehash 掌握了 Redis 快的本质原因。

接着，在[《Redis 日志篇：无畏宕机与快速恢复的杀手锏》](https://mp.weixin.qq.com/s/R-jZnjGNbOOL6zOtVd9omg)中揭晓了当 Redis 发生宕机可以通过重新读取 RDB 快照和执行 AOF 日志实现快速恢复的高可用手段。

**高可用有两个含义：一是数据尽量不丢失，二是服务尽可能提供服务。** AOF 和 RDB 保证了数据持久化尽量不丢失，而主从复制就是增加副本，一份数据保存到多个实例上。即使有一个实例宕机，其他实例依然可以提供服务。

本篇主要带大家全方位吃透 **Redis 高可用技术解决方案之一主从复制架构**。

本篇硬核，建议收藏慢慢品味，我相信读者朋友会有一个质的提升。如有错误还望纠正，谢谢。关注「码哥字节」设置「星标」第一时间接收优质文章，谢谢读者的支持。

**核心知识点**

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/核心知识点.png)

### 开篇寄语

> 问题 = 机会。遇到问题的时候，内心其实是开心的，越大的问题意味着越大的机会。
>
> 任何事情都是有代价的，有得必有失，有失必有得，所以不必计较很多东西，我们只要想清楚自己要做什么，并且想清楚自己愿意为之付出什么代价，然后就放手去做吧！



## 1. 主从复制概述

> 65 哥：有了 RDB 和 AOF 再也不怕宕机丢失数据了，但是 Redis 实例宕机了怎么实现高可用呢？

既然一台宕机了无法提供服务，那多台呢？是不是就可以解决了。Redis 提供了主从模式，通过主从复制，将数据冗余一份复制到其他 Redis 服务器。

前者称为主节点 (master)，后者称为从节点 (slave)；数据的复制是单向的，只能由主节点到从节点。

默认情况下，每台 Redis 服务器都是主节点；且一个主节点可以有多个从节点 (或没有从节点)，但一个从节点只能有一个主节点。

> 65 哥：主从之间的数据如何保证一致性呢？

为了保证副本数据的一致性，主从架构采用了读写分离的方式。

- 读操作：主、从库都可以执行；
- 写操作：主库先执行，之后将写操作同步到从库；

![Redis 读写分离](http://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis主从读写分离.png)

> 65 哥：为何要采用读写分离的方式？

我们可以假设主从库都可以执行写指令，假如对同一份数据分别修改了多次，每次修改发送到不同的主从实例上，就导致是实例的副本数据不一致了。

如果为了保证数据一致，Redis 需要加锁，协调多个实例的修改，Redis 自然不会这么干！

> 65 哥：主从复制还有其他作用么？

1. 故障恢复：当主节点宕机，其他节点依然可以提供服务；
2. 负载均衡：Master 节点提供写服务，Slave 节点提供读服务，分担压力；
3. 高可用基石：是哨兵和 cluster 实施的基础，是高可用的基石。

## 2. 搭建主从复制

**主从复制的开启，完全是在从节点发起的，不需要我们在主节点做任何事情。**

> 65 哥：怎么搭建主从复制架构呀？

可以通过 replicaof（Redis 5.0 之前使用 slaveof）命令形成主库和从库的关系。

在从节点开启主从复制，有 3 种方式：

1. 配置文件

   在从服务器的配置文件中加入 `replicaof <masterip> <masterport>`

2. 启动命令

   redis-server 启动命令后面加入 `--replicaof <masterip> <masterport>`

3. 客户端命令

   启动多个 Redis 实例后，直接通过客户端执行命令：`replicaof <masterip> <masterport>`，则该 Redis 实例成为从节点。

比如假设现在有实例 1（172.16.88.1）、实例 2（172.16.88.2）和实例 3 (172.16.88.3)，在实例 2 和实例 3 上分别执行以下命令，实例 2 和 实例 3 就成为了实例 1 的从库，实例 1 成为 Master。

```shell
replicaof 172.16.88.1 6379
```

## 3. 主从复制原理

主从库模式一旦采用了读写分离，所有数据的写操作只会在主库上进行，不用协调三个实例。

主库有了最新的数据后，会同步给从库，这样，主从库的数据就是一致的。

> 65 哥：主从库同步是如何完成的呢？主库数据是一次性传给从库，还是分批同步？正常运行中又怎么同步呢？要是主从库间的网络断连了，重新连接后数据还能保持一致吗？

65 哥你问题咋这么多，同步分为三种情况：

1. 第一次主从库全量复制；
2. 主从正常运行期间的同步；
3. 主从库间网络断开重连同步。

### 主从库第一次全量复制

> 65 哥：我好晕啊，先从主从库间第一次同步说起吧。

**主从库第一次复制过程大体可以分为 3 个阶段：连接建立阶段（即准备阶段）、主库同步数据到从库阶段、发送同步期间新写命令到从库阶段**；

直接上图，从整体上有一个全局观的感知，后面具体介绍。

![Redis全量同步](http://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis全量复制终极.png)

#### 建立连接

该阶段的主要作用是在主从节点之间建立连接，为数据全量同步做好准备。**从库会和主库建立连接，从库执行 replicaof 并发送 psync 命令并告诉主库即将进行同步，主库确认回复后，主从库间就开始同步了**。

> 65 哥：从库怎么知道主库信息并建立连接的呢？

在从节点的配置文件中的 replicaof 配置项中配置了主节点的 IP 和 port 后，从节点就知道自己要和那个主节点进行连接了。

从节点内部维护了两个字段，masterhost 和 masterport，用于存储主节点的 IP 和 port 信息。

从库执行 `replicaof` 并发送 `psync` 命令，表示要执行数据同步，主库收到命令后根据参数启动复制。命令包含了**主库的 runID** 和 **复制进度 offset** 两个参数。

- **runID**：每个 Redis 实例启动都会自动生成一个 唯一标识 ID，第一次主从复制，还不知道主库 runID，参数设置为 「?」。
- **offset**：第一次复制设置为 -1，表示第一次复制，记录复制进度偏移量。

主库收到 psync 命令后，会用 **FULLRESYNC 响应命令带上两个参数：主库 runID 和主库目前的复制进度 offset，返回给从库**。从库收到响应后，会记录下这两个参数。

**FULLRESYNC 响应表示第一次复制采用的全量复制**，也就是说，主库会把当前所有的数据都复制给从库。

#### 主库同步数据给从库

第二阶段

master 执行 `bgsave`命令生成 RDB 文件，并将文件发送给从库，同时**主库**为每一个 slave 开辟一块 replication buffer 缓冲区记录从生成 RDB 文件开始收到的所有写命令。

从库收到 RDB 文件后保存到磁盘，并清空当前数据库的数据，再加载 RDB 文件数据到内存中。

#### 发送新写命令到从库

第三阶段

从节点加载 RDB 完成后，主节点将 replication buffer 缓冲区的数据发送到从节点，Slave 接收并执行，从节点同步至主节点相同的状态。

> 65 哥：主库将数据同步到从库过程中，可以正常接受请求么？

主库不会被阻塞，Redis 作为唯快不破的男人，怎么会动不动就阻塞呢。

在生成 RDB 文件之后的写操作并没有记录到刚刚的 RDB 文件中，为了保证主从库数据的一致性，所以主库会在内存中使用一个叫 replication buffer 记录 RDB 文件生成后的所有写操作。

> 65 哥：为啥从库收到 RDB 文件后要清空当前数据库？

因为从库在通过 `replcaof`命令开始和主库同步前可能保存了其他数据，防止主从数据之间的影响。

> replication buffer 到底是什么玩意？

一个在 master 端上创建的缓冲区，存放的数据是下面三个时间内所有的 master 数据写操作。

1）master 执行 bgsave 产生 RDB 的期间的写操作；

2）master 发送 rdb 到 slave 网络传输期间的写操作；

3）slave load rdb 文件把数据恢复到内存的期间的写操作。

Redis 和客户端通信也好，和从库通信也好，Redis 都分配一个内存 buffer 进行数据交互，客户端就是一个 client，从库也是一个 client，我们每个 client 连上 Redis 后，Redis 都会分配一个专有 client buffer，所有数据交互都是通过这个 buffer 进行的。

Master 先把数据写到这个 buffer 中，然后再通过网络发送出去，这样就完成了数据交互。

不管是主从在增量同步还是全量同步时，master 会为其分配一个 buffer ，只不过这个 buffer 专门用来**传播写命令**到从库，保证主从数据一致，我们通常把它叫做 replication buffer。

**replication buffer 太小会引发的问题**：

replication buffer 由 client-output-buffer-limit slave 设置，当这个值太小会导致**主从复制连接断开**。

1）当 master-slave 复制连接断开，master 会释放连接相关的数据。replication buffer 中的数据也就丢失了，此时主从之间重新开始复制过程。

2）还有个更严重的问题，**主从复制连接断开，导致主从上出现重新执行 bgsave 和 rdb 重传操作无限循环。**

当主节点数据量较大，或者主从节点之间网络延迟较大时，可能导致该缓冲区的大小超过了限制，此时主节点会断开与从节点之间的连接；

这种情况可能引起全量复制 -> replication buffer 溢出导致连接中断 -> 重连 -> 全量复制 -> replication buffer 缓冲区溢出导致连接中断……的循环。

具体详情：[[top redis headaches for devops – replication buffe](https://redislabs.com/blog/top-redis-headaches-for-devops-replication-buffer)r]
因而推荐把 replication buffer 的 hard/soft limit 设置成 512M。

```shell
config set client-output-buffer-limit "slave 536870912 536870912 0"
```

> 65 哥：主从库复制为何不使用 AOF 呢？相比 RDB 来说，丢失的数据更少。

这个问题问的好，原因如下：

1. RDB 文件是二进制文件，网络传输 RDB 和写入磁盘的 IO 效率都要比 AOF 高。
2. 从库进行数据恢复的时候，RDB 的恢复效率也要高于 AOF。

### 增量复制

> 65 哥：主从库间的网络断了咋办？断开后要重新全量复制么？

在 Redis 2.8 之前，如果主从库在命令传播时出现了网络闪断，那么，从库就会和主库重新进行一次全量复制，开销非常大。

从 Redis 2.8 开始，网络断了之后，主从库会采用增量复制的方式继续同步。

增量复制：**用于网络中断等情况后的复制，只将中断期间主节点执行的写命令发送给从节点，与全量复制相比更加高效**。

**repl_backlog_buffer**

断开重连增量复制的实现奥秘就是 `repl_backlog_buffer` 缓冲区，不管在什么时候 master 都会将写指令操作记录在 `repl_backlog_buffer` 中，因为内存有限， `repl_backlog_buffer` 是一个定长的环形数组，**如果数组内容满了，就会从头开始覆盖前面的内容**。

master 使用 `master_repl_offset`记录自己写到的位置偏移量，slave 则使用 `slave_repl_offset`记录已经读取到的偏移量。

master 收到写操作，偏移量则会增加。从库持续执行同步的写指令后，在 `repl_backlog_buffer` 的已复制的偏移量 slave_repl_offset 也在不断增加。

正常情况下，这两个偏移量基本相等。在网络断连阶段，主库可能会收到新的写操作命令，所以 `master_repl_offset`会大于 `slave_repl_offset`。

![repl_backlog_buffer](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/repl_backlog_buffer.png)

当主从断开重连后，slave 会先发送 psync 命令给 master，同时将自己的 `runID`，`slave_repl_offset`发送给 master。

master 只需要把 `master_repl_offset`与 `slave_repl_offset`之间的命令同步给从库即可。

增量复制执行流程如下图：

![Redis增量复制](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis增量复制.png)

> 65 哥：repl_backlog_buffer 太小的话从库还没读取到就被 Master 的新写操作覆盖了咋办？

我们要想办法避免这个情况，一旦被覆盖就会执行全量复制。我们可以调整 repl_backlog_size 这个参数用于控制缓冲区大小。计算公式：

```scheme
repl_backlog_buffer = second * write_size_per_second
```

1. **second**：从服务器断开重连主服务器所需的平均时间；
2. **write_size_per_second**：master 平均每秒产生的命令数据量大小（写命令和数据大小总和）；

例如，如果主服务器平均每秒产生 1 MB 的写数据，而从服务器断线之后平均要 5 秒才能重新连接上主服务器，那么复制积压缓冲区的大小就不能低于 5 MB。

为了安全起见，可以将复制积压缓冲区的大小设为`2 * second * write_size_per_second`，这样可以保证绝大部分断线情况都能用部分重同步来处理。

### 基于长连接的命令传播

> 65 哥：完成全量同步后，正常运行过程如何同步呢？

当主从库完成了全量复制，它们之间就会一直维护一个网络连接，主库会通过这个连接将后续陆续收到的命令操作再同步给从库，这个过程也称为基于长连接的命令传播，使用长连接的目的就是避免频繁建立连接导致的开销。

在命令传播阶段，除了发送写命令，主从节点还维持着心跳机制：PING 和 REPLCONF ACK。

#### 主->从：PING

每隔指定的时间，**主节点会向从节点发送 PING 命令**，这个 PING 命令的作用，主要是为了让从节点进行超时判断。

#### 从->主：REPLCONF ACK

在命令传播阶段，从服务器默认会以每秒一次的频率，向主服务器发送命令：

```
REPLCONF ACK <replication_offset>
```

其中 replication_offset 是从服务器当前的复制偏移量。发送 REPLCONF ACK 命令对于主从服务器有三个作用：

1. 检测主从服务器的网络连接状态。
2. 辅助实现 min-slaves 选项。
3. 检测命令丢失, 从节点发送了自身的 slave_replication_offset，主节点会用自己的 master_replication_offset 对比，如果从节点数据缺失，主节点会从 `repl_backlog_buffer`缓冲区中找到并推送缺失的数据。**注意，offset 和 repl_backlog_buffer 缓冲区，不仅可以用于部分复制，也可以用于处理命令丢失等情形；区别在于前者是在断线重连后进行的，而后者是在主从节点没有断线的情况下进行的。**

### 如何确定执行全量同步还是部分同步？

在 Redis 2.8 及以后，从节点可以发送 psync 命令请求同步数据，此时根据主从节点当前状态的不同，同步方式可能是**全量复制**或**部分复制**。本文以 Redis 2.8 及之后的版本为例。

关键就是 `psync`的执行：

![增量与全量复制判断](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/增量全量流程图.png)

1. 从节点根据当前状态，发送 `psync`命令给 master：
   - 如果从节点从未执行过 `replicaof` ，则从节点发送 `psync ? -1`，向主节点发送全量复制请求；
   - 如果从节点之前执行过 `replicaof` 则发送 `psync <runID> <offset>`, runID 是上次复制保存的主节点 runID，offset 是上次复制截至时从节点保存的复制偏移量。
2. 主节点根据接受到的`psync`命令和当前服务器状态，决定执行全量复制还是部分复制：
   - runID 与从节点发送的 runID 相同，且从节点发送的 `slave_repl_offset `之后的数据在 `repl_backlog_buffer `缓冲区中都存在，则回复 `CONTINUE`，表示将进行部分复制，从节点等待主节点发送其缺少的数据即可；
   - runID 与从节点发送的 runID 不同，或者从节点发送的 slave_repl_offset 之后的数据已不在主节点的 `repl_backlog_buffer `缓冲区中 (在队列中被挤出了)，则回复从节点 `FULLRESYNC <runid> <offset>`，表示要进行全量复制，其中 runID 表示主节点当前的 runID，offset 表示主节点当前的 offset，从节点保存这两个值，以备使用。

一个从库如果和主库断连时间过长，造成它在主库 `repl_backlog_buffer `的 slave_repl_offset 位置上的数据已经被覆盖掉了，此时从库和主库间将进行全量复制。

**总结下**

每个从库会记录自己的 `slave_repl_offset`，每个从库的复制进度也不一定相同。

在和主库重连进行恢复时，从库会通过 psync 命令把自己记录的 `slave_repl_offset `发给主库，主库会根据从库各自的复制进度，来决定这个从库可以进行增量复制，还是全量复制。

**replication buffer 和 repl_backlog**

1. replication buffer 对应于每个 slave，通过 `config set client-output-buffer-limit slave `设置。
2. `repl_backlog_buffer `是一个环形缓冲区，整个 master 进程中只会存在一个，所有的 slave 公用。repl_backlog 的大小通过 repl-backlog-size 参数设置，默认大小是 1M，其大小可以根据每秒产生的命令、（master 执行 rdb bgsave） +（ master 发送 rdb 到 slave） + （slave load rdb 文件）时间之和来估算积压缓冲区的大小，repl-backlog-size 值不小于这两者的乘积。

总的来说，`replication buffer` 是主从库在进行全量复制时，主库上用于和从库连接的客户端的 buffer，而 r`epl_backlog_buffer` 是为了支持从库增量复制，主库上用于持续保存写操作的一块专用 buffer。

`repl_backlog_buffer `是一块专用 buffer，在 Redis 服务器启动后，开始一直接收写操作命令，这是所有从库共享的。主库和从库会各自记录自己的复制进度，所以，不同的从库在进行恢复时，会把自己的复制进度（`slave_repl_offset`）发给主库，主库就可以和它独立同步。

如图所示：

![repl_backlog与repl_buffer区别](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/repl_backlog与repl_buffer区别.png)

## 4. 主从应用问题

### 4.1 读写分离的问题

**数据过期问题**

> 65 哥：主从复制的场景下，从节点会删除过期数据么？

这个问题问得好，为了主从节点的数据一致性，从节点不会主动删除数据。我们知道 Redis 有两种删除策略：

1. 惰性删除：当客户端查询对应的数据时，Redis 判断该数据是否过期，过期则删除。
2. 定期删除：Redis 通过定时任务删除过期数据。

> 65 哥：那客户端通过从节点读取数据会不会读取到过期数据？

Redis 3.2 开始，通过从节点读取数据时，先判断数据是否已过期。如果过期则不返回客户端，并且删除数据。

### 4.2 单机内存大小限制

如果 Redis 单机内存达到 10GB，一个从节点的同步时间在几分钟的级别；如果从节点较多，恢复的速度会更慢。如果系统的读负载很高，而这段时间从节点无法提供服务，会对系统造成很大的压力。

如果数据量过大，全量复制阶段主节点 fork + 保存 RDB 文件耗时过大，从节点长时间接收不到数据触发超时，主从节点的数据同步同样可能陷入**全量复制->超时导致复制中断->重连->全量复制->超时导致复制中断**……的循环。

此外，主节点单机内存除了绝对量不能太大，其占用主机内存的比例也不应过大：最好只使用 50% - 65% 的内存，留下 30%-45% 的内存用于执行 bgsave 命令和创建复制缓冲区等。

## 总结

1. 主从复制的作用：AOF 和 RDB 二进制文件保证了宕机快速恢复数据，尽可能的防止丢失数据。但是宕机后依然无法提供服务，所以便演化出主从架构、读写分离。
2. 主从复制原理：连接建立阶段、数据同步阶段、命令传播阶段；数据同步阶段又分为 全量复制和部分复制；命令传播阶段主从节点之间有 PING 和 REPLCONF ACK 命令互相进行心跳检测。
3. 主从复制虽然解决或缓解了数据冗余、故障恢复、读负载均衡等问题，但其缺陷仍很明显：**故障恢复无法自动化；写操作无法负载均衡；存储能力受到单机的限制；这些问题的解决，需要哨兵和集群**的帮助，我将在后面的文章中介绍，欢迎关注。

> 65 哥：码哥你的图画的真好看，内容好，跟着你的文章我收获了很多，我要收藏、点赞、在看和分享。让更多的优秀开发者看到共同进步！

谢谢读者支持，另外**读者技术群**也开通了，关注公众号回复「**加群**」，群里 N 多大厂的大佬，跟我一块交流，共同进步！

![码哥字节](https://magebyte.oss-cn-shenzhen.aliyuncs.com/wechat/码哥字节二维码.jpg)

**参考资料：**

[1] redis 设计与实现（黄健宏）

[2] redis replication (http://redis.io/topics/replication)

[3] designing redis replication partial resync (http://antirez.com/news/31)

(4) Redis 核心技术与实战（https://time.geekbang.org/column/intro/329）