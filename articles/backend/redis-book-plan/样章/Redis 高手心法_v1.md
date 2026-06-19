# 第 1 章 起势入门

## 1.1 从头说起

天下武功，无坚不摧，唯快不破！我的名字叫 Redis，全称是 Remote Dictionary Server。

> 有人说，组 CP，除了要了解她外，还要给机会让她了解你。

那么，作为开发工程师的你，是否愿意认真阅读此心法抓住机会来了解我，运用到你的系统中提升性能。我遵守 BSD 协议，由意大利人 Salvatore Sanfilippo 使用 C 语言编写的一个基于内存实现的键值型非关系（NoSQL）数据库。

我是一个内存数据结构存储、可作为数据库、缓存、消息队列、流处理引擎，速度快是我的特点。根据官方数据，Redis 的 QPS 可以达到约 100000（每秒请求数）。

我提供了 String（字符串）、Hashes（散列表）、Lists（列表）、Sets（无序集合）、Sorted Sets（可根据范围查询的排序集合）、Bitmap（位图）、HyperLogLog、Geospatial （地理空间）和 Stream（流）等数据结构。

**数据结构的使用技法和实现原理是你核心筑基必经之路，好好修炼。**

除此之外，我还具有主从复制、Lua 脚本、LRU 淘汰机制，事务和不同级别的磁盘持久化功能，并通过 Redis Sentinel（哨兵）和 Redis Cluster（Redis 集群）实现高可用，这部分内容，重中之重，高手必备。

我还支持一些原子操作，支持异步复制实现快速非阻塞同步和自动重连，另外需要注意的是，推荐你在 Lunix /Unix 系统上部署，官方并没有在 Windows 系统上构建安装包。

### 1.1.1 Redis 能干啥

> 程许媛：“Redis 你说了这么多？你能干啥？别王婆卖瓜，自卖自夸。”

**缓存**

这是我被使用的最多的场景，能极大提升应用程序的性能。当单个 MySQL 读写压力比较大，场景是读多写少的时候，把热点数据存储在更快的存储中，也就是 Redis。

读取数据

1. 先从缓存中读取数据是否命中。
2. 缓存未命中，则查询数据库获取数据，并把数据写到 Redis 中，让后续读取相同数据的请求命中缓存，最后把数据返回给调用者。
3. 缓存命中，直接返回。

写数据

至于修改数据，程序员想了很多方法去尽可能保证 Redis 与 MySQL 的数据一致性。

- 先写 MySQL 数据，再删除 Redis 缓存数据。
- 监听 MySQL binlog 日志，修改 Redis 数据。

**排行榜**

使用 MySQL 等关系型数据库，非常麻烦，性能也差，而直接使用 Redis SortedSet 轻松搞定。

**消息队列**

简单消息队列，在一些不需要高可靠，但是数据量大会给 MySQL 带来非常大压力的场景，比如：到货通知、未读消息、邮件发送之列的。程序员可以使用 Lists 或者 Stream 来实现一个队列。

**分布式锁**

Redisson 这个框架，就是使用 Redis 弄出了一套分布式锁解决方案。

**计数器**

Redis 的命令都是原子性的，程序员可以轻松地利用 `INCR`，`DECR`命令来构建计数器系统。

还有很多场景，我会在后面章节详细道来，学完之后，我相信你定能筑基锻体，念头通达，升职加薪。

千古无同局，叶底能否藏花，我们未来印证，愿此心法能让你学有所成，你来，我等着。

### 1.1.2 源代码编译

经过上一篇的 Redis 简介，我相信你一定想继续了解 Redis。本章节会通过源代码编译来安装 Redis 7.0.5，让你在自己的机器上搭建一套可以 Debug 的 Redis 7.0.5 源码环境。

这也是后续原理分析的基础，推荐你部署在 macOS 或者 Linux 上搭建，如果你是 Windows 环境，那就搞一个虚拟机在上面装一个 Linux 系统，再继续搭建 Redis 环境。

> 程许媛：“我的电脑是 mac OS 系统，你就用这个来演示吧。”

#### 获取源代码

有两种方式，第一种是从官网下载 Redis 源码压缩包，如图 1-1 所示。

![图1-1][image-1]

图 1-1

将压缩包解压得到一个文件夹。

第二种方式，通过 git clone 获取源码。

从 Github 上，使用 `git clone https://github.com/redis/redis.git`指令下载，下载完成后你会得到如下文件。

![图 1-2][image-2]

图 1-2

进入 redis 目录，使用 `git checkout` 切换到 7.0.5 这个 tag 。

```shell
gir checkout tags/7.0.5 -b 7.0.5
```

#### 编译 Redis

在编译之前，需要安装一些环境依赖，Redis 是 C 语言编写的，所以还需要 gcc 编译器。

执行 `gcc -v`判断是否安装了编译器。

![图 1-3][image-3]

图 1-3

没有安装的话，使用如下指令安装。

```shell
xcode-select --install
```

一切准备就绪，进入 redis 的源码目录，执行 `make`命令，这个就好比 Java 中的 mvn 命令。

```makefile
make CFLAGS="-g -O0" MALLOC=jemalloc
```
![]()
命令后边的 “-O0” 参数表示告诉编译器不要优化代码，防止你在 Debug 的时候， IDE 里面的 Redis 源码与实际运行的代码对应不上。

`MALLOC=jemalloc` ，指定在 mac OS 系统上 Redis 使用 jemalloc 内存分配器来分配内存，Linux 默认使用该分配器。

需要注意的是内存碎片自动清理功能只在 jemalloc 内存分配器生效。

**如果安装包用于生产环境的 Linux 系统上，那么直接使用指令 `make`命令即可。**

编译成功，将会看到图 1-4 中`Hint: It's a good idea to run 'make test' ;)`，提示我们可以运行单元测试，这一步可以省略。

![图 1-4][image-5]

图 1-4

#### 启动 Redis

编译成功，进入 src 源码目录下，你会看到一个 redis-server 的可执行程序，使用如下指令启动。

```shell
./redis-server ../redis.conf
```

![图 1-5][image-6]

图 1-5

#### 代码调试环境搭建

编译好了，我们还差一个方便阅读和调试源码的工具。为了方便阅读和 debug 源码，极力推荐你使用 CLion 来阅读和调试 Redis 源码，我使用的是 CLion 2021.3 版本。

安装好以后，打开 CLion，点击 open，选择 Redis 源码目录即可。

![图 1-6][image-7]

图 1-6

之后检查下 Run Debug 是否出现这些选项，选择编辑。

![图 1-7][image-8]

图 1-7

选择编辑 redis-server ，指定启动配置文件 `redis.conf`的目录，保存。

![图 1-8][image-9]

图 1-8

在 `server.c` 的`main()` 方法加断点，Debug 启动 redis -server，进行源码 Debug。

![图 1-9][image-10]

图 1-9

大功告成，接下来就可以在 Redis 的知识海洋里呛水了。

### 1.1.3 目录结构

在知识海洋呛水之前，先来了解下 Redis 的目录结构，从 Redis 整体目录结构来对系统有个全局的认识，了解一个系统的主要组成，同时防止陷入细节或者无从下手。

#### deps

这个目录主要包含 Redis 所依赖的第三方代码库。

- Jemalloc，内存分配器，默认情况下选择该内存分配器来代替 Linux 系统的 libc-malloc，**libc-malloc 性能不高，且碎片化严重**。
- hiredis，这是官方 C 语言客户端。
- linenoise 是一种读线替换。它由 Redis 的同一作者开发，但作为一个单独的项目进行管理，并根据需要进行更新。
- lua，顾名思义，就是 lua 相关的功能。
- hdr\_histogram，用于生成每个命令的延迟跟踪直方图。

#### src 目录

这是 Redis 源码的重要组成部分，里面有 `commands` 和 `modules` 两个子目录，其余功能模块的源码都在 src 目录下，这是最重要的目录。

`modules`目录包含了实现`Redis module`的示例代码，`commands`里面都是 json 格式的文件，包含了每个指令的元信息。

#### tests 目录

顾名思义，功能模块测试和单元测试的代码就在这里。

- cluster，Redis Cluster 功能测试。
- sentinel，哨兵集群功能测试。
- unit，单元测试。
- integration，主从复制功能测试。

剩下的 assets、helpers、modules、support 四个目录中是用来支撑测试功能的。

#### utils 目录

辅助性功能的脚本或者代码，比如用于创建 Redis Cluster 的脚本，lru 算法效果展示代码等。

除此之外，Redis 源码目录还有两个重要的文件，redis.conf 和 sentinel.conf，分别用于配置 Redis 实例运行和哨兵配置。

## 1.2 整体架构

在上一篇通过源码编译构建出可调式环境之后，想必你想更深入了解我的整体架构。当你熟悉我的整体架构和每个模块，遇到问题才能直击本源，直捣黄龙，一笑破苍穹。

我的核心模块如图 1-10。

![图1-10][image-11]

图 1-10

- Client 客户端，官方提供了 C 语言开发的客户端，可以发送命令，性能分析和测试等。
- 网络层事件驱动模型，基于 I/O 多路复用，封装了一个短小精悍的高性能 ae 库，全称是 `a simple event-driven programming library`。
  - 在 ae 这个库里面，我通过 `aeApiState` 结构体对 `epoll、select、kqueue、evport`四种 I/O 多路复用的实现进行适配，让上层调用方感知不到在不同操作系统实现 I/O 多路复用的差异。
  - Redis 中的事件可以分两大类：一类是网络连接、读、写事件；另一类是时间事件，比如定时执行 rehash 、RDB 内存快照生成，过期键值对清理操作。
- 命令解析和执行层，负责执行客户端的各种命令，比如 `SET、DEL、GET`等。
- 内存分配和回收，为数据分配内存，提供不同的数据结构保存数据。
- 持久化层，提供了 RDB 内存快照文件 和 AOF 两种持久化策略，实现数据可靠性。
- 高可用模块，提供了副本、哨兵、集群实现高可用。
- 监控与统计，提供了一些监控工具和性能分析工具，比如监控内存使用、基准测试、内存碎片、bigkey 统计、慢指令查询等。

掌握了整体架构和模块后，接下来进入 src 源码目录，使用如下指令执行 `redis-server`可执行程序启动 Redis。

```sh
./redis-server ../redis.conf
```

每个被启动的服务我都会抽象成一个 redisServer，源码定在`server.h` 的`redisServer` 结构体。

这个结构体包含了存储键值对的数据库实例、redis.conf 文件路径、命令列表、加载的 Modules、网络监听、客户端列表、RDB AOF 加载信息、配置信息、RDB 持久化、主从复制、客户端缓存、数据结构压缩、pub/sub、Cluster、哨兵等一些列 Redis 实例运行的必要信息。结构体字段很多，部分核心字段如下。

```c
truct redisServer {
    pid_t pid;  /* 主进程 pid. */
    pthread_t main_thread_id; /* 主线程 id */
    char *configfile;  /*redis.conf 文件绝对路径*/
    redisDb *db; /* 存储键值对数据的 redisDb 实例 */
  	int dbnum;  /* DB 个数 */
    dict *commands; /* 当前实例能处理的命令表，key 是命令名，value 是执行命令的入口 */
    aeEventLoop *el;/* 事件循环处理 */
    int sentinel_mode;  /* true 则表示作为哨兵实例启动 */

  	/* 网络相关 */
    int port;/* TCP 监听端口 */
    list *clients; /* 连接当前实例的客户端列表 */
    list *clients_to_close; /* 待关闭的客户端列表 */

    client *current_client; /* 当前执行命令的客户端*/
};
```

### 1.2.1 数据存储原理

以 redis 7.0 版本为例， server.h 的 `redisDb`结构体为中心抽象存储。其中`redisDb *db`指针非常重要，它指向了一个长度为 dbnum（默认 16）的 redisDb 数组，它是整个存储的核心，我就是用这玩意来存储键值对。

#### redisDb

```c
typedef struct redisDb {
    dict *dict;
    dict *expires;
    dict *blocking_keys;
    dict *ready_keys;
    dict *watched_keys;
    int id;
    long long avg_ttl;
    unsigned long expires_cursor;
    list *defrag_later;
    clusterSlotToKeyMapping *slots_to_keys;
} redisDb;
```

**dict 和 expires**

- dict 和 expires 是最重要的两个属性，底层数据结构是字典，分别用于存储键值对数据和 key 的过期时间。
- expires，底层数据结构是 dict 字典，存储每个 key 的过期时间。

> MySQL：“为什么分开存储？”

好问题，之所以分开存储，是因为过期时间并不是每个 key 都会设置，它不是键值对的固有属性，分开后虽然需要两次查找，但是能节省内存开销。

**blocking\_keys 和 ready\_keys**

底层数据结构是 dict 字典，主要是用于实现 BLPOP 等阻塞命令。

当客户端使用 BLPOP 命令阻塞等待取出列表元素的时候，我会把 key 写到 blocking\_keys 中，value 是被阻塞的客户端。

当下一次收到 PUSH 命令执时，我会先检查 blocking\_keys 中是否存在阻塞等待的 key，如果存在就把 key 放到 ready\_keys 中，在下一次 Redis 事件处理过程中，会遍历 ready\_keys 数据，并从 blocking\_keys 中取出阻塞的客户端响应。

**watched\_keys**

用于实现 watch 命令，存储 watch 命令的 key。

**id**

Redis 数据库的唯一 ID，一个 Redis 服务支持多个数据库，默认 16 个。

**avg\_ttl**

用于统计平均过期时间。

**expires\_cursor**

统计过期事件循环执行的次数。

**defrag\_later**

保存逐一进行碎片整理的 key 列表。

**slots\_to\_keys**

仅用于 Cluster 模式，当使用 Cluster 模式的时候，只能有一个数据库 db 0。slots\_to\_keys 用于记录 cluster 模式下，存储 key 与哈希槽映射关系的数组。

#### dict

**Redis 使用 dict 结构来保存所有的键值对（key-value）数据，这是一个散列表，所以 key 查询时间复杂度是 O(1) 。**

所谓散列表，我们可以类比 Java 中的 `HashMap`，其实就是一个数组，数组的每个元素叫做哈希桶。

```c
struct dict {
    dictType *type;

    dictEntry **ht_table[2];
    unsigned long ht_used[2];

    long rehashidx;

    int16_t pauserehash;
    signed char ht_size_exp[2];
};
```

dict 的结构体里，有 `dictType *type`，`**ht_table[2]`，`long rehashidx` 三个很重要的结构。

- `dictType *type`：一个指向 `dictType` 结构的指针，表示字典的类型。`dictType` 包含了一组函数指针，用于对键值对进行操作，例如哈希函数、复制键、复制值等。
- `ht_table[2]`：两个哈希表的数组。每个哈希表是一个指向 `dictEntry` 指针数组的指针，`dictEntry` 表示字典中的一个键值对。正常情况使用 ht\_table[0] 存储数据，当执行 rehash 的时候，使用 ht\_table[1] 配合完成 。
- `ht_used[2]`：两个哈希表的使用情况，表示当前哈希表已经使用的槽位数量。
- `long rehashidx`：示正在进行 rehash 操作时的索引位置。当 `rehashidx` 的值为 -1 时，表示没有进行 rehash 操作。
- `int16_t pauserehash`： 如果大于 0，表示 rehash 操作被暂停。小于 0 则表示编码错误。
- `signed char ht_size_exp[2]`： 两个哈希表的大小，以 2 的指数形式表示。`ht_size_exp` 数组的每个元素表示对应哈希表的大小指数。

**重点关注 ht\_table 数组，数组每个位置叫做哈希桶，就是这玩意保存了所有键值对，每个哈希桶的类型是 dictEntry。**

> MySQL：“Redis 支持那么多的数据类型，哈希桶咋保存？”

他的玄机就在 dictEntry 中，每个 dict 有两个 ht\_table，用于存储键值对数据和实现渐进式 rehash。

```c
typedef struct dictEntry {
    void *key;
    union {
       // 指向实际 value 的指针
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    // 散列表冲突生成的链表
    struct dictEntry *next;
    void *metadata[];
} dictEntry;
```

- **`void *key;`：** 一个指向键的指针，表示字典中的键。

  **`union {...} v;`：** 一个联合体，包含了字典条目的值，当它的值是 uint64\_t、int64\_t 或 double 类型时，就不再需要额外的存储，这有利于减少内存碎片。（为了节省内存操碎了心）当然，**val 也可以是 void 指针，指向值的指针，以便能存储任何类型的数据。**

  **`struct dictEntry *next;`：** 指向哈希桶中下一个条目的指针，允许多个条目存在于同一个哈希桶中，形成一个链表。`ht_table` 使用链地址法来处理键碰撞： **当多个不同的键拥有相同的哈希值时，哈希表用一个链表将这些键连接起来。**

  **`void *metadata[];`：** 一个灵活数组，用于存储元数据。元数据的大小由 `dictType` 的 `dictEntryMetadataBytes` 函数返回。这个数组允许存储额外的信息，以扩展字典条目的功能。

**哈希桶并没有保存值本身，而是指向具体值的指针，从而实现了哈希桶能存不同数据类型的需求**。

#### 1.2.2 redisObject

`dictEntry` 的 `*val` 指针指向的值实际上是一个 `redisObject` 结构体，这是一个非常重要的结构体。我的 key 是字符串类型，而 value 可以是 String、Lists、Set、Sorted Set、Hashes 等数据类型。键值对的值都被包装成 redisObject 对象， `redisObject` 在 `server.h` 中定义。

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS;
    int refcount;
    void *ptr;
} robj;
```

- `type:4`： 记录了对象的类型，string、set、hash 、Lis、Sorted Set 等，根据该类型才可以确定是哪种数据类型，使用什么样的 API 操作。

- `encoding:4`：编码方式，**表示 ptr 指向的数据类型具体数据结构，即这个对象使用了什么数据结构作为底层实现**保存数据。**同一个对象使用不同编码实现内存占用存在明显差异，内部编码对内存优化非常重要。**例如字符串可以使用 RAW 编码或 INT 编码。
- `lru:LRU_BITS`：LRU 策略下对象最后一次被访问的时间，如果是 LFU 策略，那么低 8 位表示访问频率，高 16 位表示访问时间。
- `refcount` ：表示引用计数，由于 C 语言并不具备内存回收功能，所以 Redis 在自己的对象系统中添加了这个属性，当一个对象的引用计数为 0 时，则表示该对象已经不被任何对象引用，则可以进行垃圾回收了。
- `*ptr` 指针：对象的指针，指向实际存储对象数据的位置。根据对象的类型和编码，`ptr` 可能指向字符串、列表、哈希等具体的数据结构。

如图 1-11 是由 redisDb、dict、dictEntry、redisObejct 关系图：

![图1-11][image-12]

图 1-11

**注意，一开始的时候，我只使用 ht\_table[0] 这个散列表读写数据，ht\_table[1] 指向 NULL，当这个散列表容量不足，触发扩容操作，这时候就会创建一个更大的散列表 ht\_table[1]。**

**接着我会使用渐进式 rehash 的方式和定期迁移的方式将 ht\_table[0] 的数据迁移到 ht\_table[1] 上，全部迁移完成后，再修改下指针，让 ht\_table[0] 指向扩容后的散列表，回收掉原来的散列表，ht\_table[1] 再次指向 NULL。**

### 1.2.2 一条指令的执行过程

> MySQL：“知道了整体架构和各个模块后，一条 Redis 命令是如何执行呢？”

想要了解一个技术的本质，先从整体再到细节，切不可不见森林就去看叶子，所以我先梳理出一个关键执行流程给你，防止陷入细节不能自拔。

![图1-12][image-13]

图 1-12

跟着流程图，我再详细介绍每一步执行详细步骤，本流程图重点展示的是在开启 I/O 多线程模型的情况下的指令执行过程。

先看 `server.c` 中的 `main()` 函数，里面有几个很重要的几个步骤。

- `initServerConfig()`：初始化 Redis 服务的各种配置。
- `initServer()`：初始化服务，分配运行需要的数据结构，设置监听 socket 等。
- `aeMain()`：监听新连接的事件循环，通过事件驱动模块 ae 接收客户端发起的请求。

#### 创建连接

Redis 初始化完成，默认会在 6379 端口监可读事件，通过事件驱动模块 ae 接收客户端发起的请求，这是一个基于 I/O 多路复用的 while 无限循环。

源代码在 `server.c`的 `createSocketAcceptHandler`函数中。

- 初始化的时候，`createSocketAcceptHandler()`函数只监听了可读事件，并将 `acceptTcpHandler()`函数作为处理该事件的回调函数。
- `acceptTcpHandler()` 函数的核心就是一个 while 循环，里面是处理建立连接的请求。
- 执行完 `acceptTcpHandler()` 函数后，一条 Redis 客户端连接就创建完成了。

#### 延迟读取

客户端发来`SET key 码哥字节`命令请求，触发可读事件，实际上就是触发了 `connSocketEventHandler()` 函数，该函数的内部会处理可读事件和可写事件，默认先处理可读事件，再处理可写事件（正常情况下是先有请求可读事件发生，处理好读事件之后，才能响应客户端，这个时候再处理可写事件）。

先看可读事件的处理，实际就是进入 `readQueryFromClient()`函数执行流程，这个流程会判断是否启用 I/O 多线程来选择不同分支处理。

**开启 I/O 多线程模式**

主线程首次调用 `readQueryFromClient()` 函数时会先执行 `postponeClientRead()`将可读事件的 client 放入 `redisServer.clients_pending_read` 列表。

在主线程每次进入 `aeApiPoll()`函数阻塞等待可读可写事件之前，会先调用 `beforeSleep()`函数，这个函数会调用 `handleClientsWithPendingReadsUsingThreads()`函数，他的核心逻辑是把主函数上次调用 `readQueryFromClient()` 函数把 client 写到`redisServer.clients_pending_read` 队列的 client 按照 `Round Robbin` 的方式分配到每个 I/O 线程关联的 `io_threads_list`队列，I/O 线程消费该队列进行请求读取和命令解析。

handleClientsWithPendingReadsUsingThreads 源代码如下，我加了关键注释。

```c
int handleClientsWithPendingReadsUsingThreads(void) {
    // 1. I/O 多线程模型是否启用，未启用的话停止后续执行。
    if (!server.io_threads_active || !server.io_threads_do_reads) return 0;
    int processed = listLength(server.clients_pending_read);
    if (processed == 0) return 0;

    /* 2. 将 server.clients_pending_read 中的 client 分配到不同的 io_threads_list */
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_read,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }

    /* 3. 设置全局变量 io_threads_op 为 IO_THREADS_OP_READ 告诉 I/O 线程此次处理的是
     * 可读事件。
     */
    io_threads_op = IO_THREADS_OP_READ;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        setIOPendingCount(j, count);
    }

    /* 4. 主线程处理 io_threads_list[0] 中的任务，调用 readQueryFromClient 读取数据并解析命令，完成清空 io_threads_list[0] 列表 */
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        readQueryFromClient(c->conn);
    }
    listEmpty(io_threads_list[0]);

    /* 5. 主线程阻塞等待其他 I/O 线程完成数据读取和命令解析。 */
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }

    // 6. 全局变量 io_threads_op 设置成 IO_THREADS_OP_IDLE 表示当前 I/O 线程空闲。
    io_threads_op = IO_THREADS_OP_IDLE;

    /* 7. 主线程从 clients_pending_read 取出 client 并执行已经读取完成并解析好的命令*/
    while(listLength(server.clients_pending_read)) {
        // 省略部分代码

        if (processPendingCommandAndInputBuffer(c) == C_ERR) {
            continue;
        }
        // 省略部分代码
    }

    server.stat_io_reads_processed += processed;

    return processed;
}
```

**未开启 I/O 多线程模型**

主线程自己完成读取命令、解析命令、执行命令、发送结果给客户端全部流程。

#### 命令读取和解析

主线程首次调用 `readQueryFromClient()` 会通过 `postponeClientRead()`把可读事件延迟到 I/O 线程处理，就是把可读事件的 client 放到 `server.clients_pending_read 中`。再按照 `Round Robbin` 的方式把 client 分配到每个 I/O 线程关联的 `io_threads_list`队列

主线程和 I/O 线程从绑定的 `io_treads_list`任务队列中取出任务并处理。主线程和 I/O 线程再次进入 `readQueryFromClient()`流程。

**需要注意的是，这次执行`readQueryFromClient()`前， client 状态已经被设置成 `CLIENT_PENDING_READ`，不会再次加入 `clients_pending_read`队列，而是进入真正的执行流程，读取 Socket 并解析命令。**

主线程在完成 `io_threads_list[0]` client 的读取和解析之后，会阻塞等待全部 I/O 线程完成等到全部命令解析完成，才会真正的执行命令。

#### 命令执行

主线程等所有 I/O 线程完成 Socket 读取和命令解析，接下来到了最重要的一点，执行命令。

回到 `handleClientsWithPendingReadsUsingThreads`函数，在完成了任务分配和命令读取和解析之后，主线程会进入一个 while 循环，从 `server.clients_pending_read` 队列中，每取出一个 client 就调用一次次 `processPendingCommandAndInputBuffer()`函数执行这个 client 已经解析好的命令。

```c
/* 7. 主线程从 clients_pending_read 取出 client 并执行已经读取完成并解析好的命令*/
while(listLength(server.clients_pending_read)) {
    // 省略部分代码

    if (processPendingCommandAndInputBuffer(c) == C_ERR) {
        continue;
    }
    // 省略部分代码
}
```

函数内部会调用 `processCommandAndResetClient()`函数执行实际的命令。

```c
int processPendingCommandAndInputBuffer(client *c) {
    if (c->flags & CLIENT_PENDING_COMMAND) {
        c->flags &= ~CLIENT_PENDING_COMMAND;
        // 主线程执行命令
        if (processCommandAndResetClient(c) == C_ERR) {
            return C_ERR;
        }
    }

    // 省略部分源码
    return C_OK;
}
```

`processCommandAndResetClient` 函数源码如下。

```c
int processCommandAndResetClient(client *c) {
    // 省略部分源码
    if (processCommand(c) == C_OK) {
        commandProcessed(c);
        updateClientMemUsageAndBucket(c);
    }
	// 省略部分源码
    return deadclient ? C_ERR : C_OK;
}
```

重点看 `processCommand(c)`函数，他的核心逻辑是从调用`lookupCommand()`函数从 `server.commands`这个字典中查找命令对应的 `redisCommand`实例。

经过系列检查，不通过就执行 `rejectCommandFormat()` 函数给客户端返回错误信息。

通过的话就调用 `c->cmd->proc()`函数，处理真正的命令，比如 SET 命令， `proc()` 就指向 `setCommand()` 函数。

```c
 /* Exec the command */
if (c->flags & CLIENT_MULTI &&
    c->cmd->proc != execCommand &&
    c->cmd->proc != discardCommand &&
    c->cmd->proc != multiCommand &&
    c->cmd->proc != watchCommand &&
    c->cmd->proc != quitCommand &&
    c->cmd->proc != resetCommand)
{
    // 1. 命令入队
    queueMultiCommand(c, cmd_flags);
    // 2. 入队后响应客户端 "+QUEUED" 字符串
    addReply(c,shared.queued);
} else {
    // 3. 不需要入队，直接执行的命令调用 call() 函数
    call(c,CMD_CALL_FULL);
    c->woff = server.master_repl_offset;
    if (listLength(server.ready_keys))
        handleClientsBlockedOnKeys();
}
```

函数执行流程图如下。

![图1-13][image-14]

图 1-13

回头再看 `processCommandAndResetClient` 函数，发现 `return deadclient ? C_ERR : C_OK;` 根据成功返回 1，错误返回 0。

> MySQL ：“并没有看到将命令产生的返回值写回客户端的代码，你是如何将命令产生的返回值写回客户端的？”

通过流程图你知道，在开启 I/O 多线程模型的时候，我是通过 I/O 线程将命令的返回值写回客户端的。

虽然调用 `redisCommand->proc()`的时候没有返回值，但是我在 proc() 函数里将返回值写到一个专门存放返回值的地方让 I/O 线程去取值并返回客户端。

#### 响应结果给客户端

我用 String 类型的 `GET`命令为例，源码文件是 `t_string.c`。`getCommand` 啥都不干，直接调用 `getGenericCommand`函数。

```c
int getGenericCommand(client *c) {
    robj *o;
	// 1. 从数据库中查询 key 对应的 value 值，并赋值给 o.
    if ((o = lookupKeyReadOrReply(c,c->argv[1],shared.null[c->resp])) == NULL)
        return C_OK;
	// 一些类型校验
    if (checkType(c,o,OBJ_STRING)) {
        return C_ERR;
    }
	// 对返回值进行编码并返回
    addReplyBulk(c,o);
    return C_OK;
}

void getCommand(client *c) {
    getGenericCommand(c);
}
```

重点就在于 `addReplyBulk(c,o)`，执行命令完毕后，进入响应客户端阶段，主线程调用 `addReply`函数把执行结果响应给客户端。

```c
void addReply(client *c, robj *obj) {
    if (prepareClientToWrite(c) != C_OK) return;
    ...
}
```

内部调用 `prepareClientToWrite`把执行结果放到 `clients_pending_write` 可写队列中。

在进入下一次事件循环时， `beforeSleep()` 函数内部会调用 `handleClientsWithPendingWritesUsingThreads` 函数把 `clients_pending_write`队列任务分配给 I/O 线程和主线程。

完成任务分配之后，I/O 线程和主线程会调用 `writeToClient`函数把命令的执行结果发送到客户端，**writeToClient() 函数的核心是一个 while 循环**，内部会不断调用 `_writeToClient()`函数，往底层的 Scoket 连接里面写数据。

详细步骤我在源代码中加了注释。

```c
int handleClientsWithPendingWritesUsingThreads(void) {
    // 1. 没有客户端需要处理，return 0。
    int processed = listLength(server.clients_pending_write);
    if (processed == 0) return 0;

    // 2. 如果没有启用 I/O 线程或者只有少量 client 需要处理，主线程同步处理，不使用 I/O 线程。
    if (server.io_threads_num == 1 || stopThreadedIOIfNeeded()) {
        return handleClientsWithPendingWrites();
    }

    /* 3. 开启了I/O 多线程模式，但是没有激活的话则调用 startThreadedIO 激活 */
    if (!server.io_threads_active) startThreadedIO();

    /* 4. 将 clients_pending_write 队列 client 分配到不同io_threads_list 列表中，主线程负责 io_threads_list[0] 的任务
    */
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_write,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_WRITE;
		// 省略部分代码

        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }

    /* 5. 修改全局变量，标识可写 */
    io_threads_op = IO_THREADS_OP_WRITE;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        setIOPendingCount(j, count);
    }

    /* 6. 主线程调用 writeToClient 把 io_threads_list[0] 的 client 执行结果写回给客户端  */
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        writeToClient(c,0);
    }
    // 7. 清空 io_threads_list[0]
    listEmpty(io_threads_list[0]);

    /* 8. 主线程等待其他 I/O 线程执行完客户端响应 */
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }
	// 9. 全局变量设置为 IO_THREADS_OP_IDLE 表示线程空闲
    io_threads_op = IO_THREADS_OP_IDLE;

    /* 10. 检查 clients_pending_write 中的 client 是否还有数
    	要响应给客户端。有的话就调用 CT_Socket.set_write_handler 函数将 sendReplyToClient() 函数设置为 connection-> write_handler 回调函数
    */
    listRewind(server.clients_pending_write,&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);

        updateClientMemUsageAndBucket(c);

        if (clientHasPendingReplies(c)) {
            installClientWriteHandler(c);
        }
    }
    // 11. 清空 server.clients_pending_write 队列
    listEmpty(server.clients_pending_write);

    server.stat_io_writes_processed += processed;

    return processed;
}
```

1. 查询 `server.clients_pending_write`队列长度，为空则停止后续流程。
2. 如果没有启用 I/O 线程或者只有少量 client 需要处理，主线程同步调用 `handleClientsWithPendingWrites()` 完成全部 client 的写回响应，不使用 I/O 线程。不满足以上条件，则走接下来的步骤。
3. 开启了 I/O 多线程模式，但是没有激活的话则调用 startThreadedIO 激活 I/O 线程。
4. 循环遍历 `clients_pending_write` 队列，使用 Round-Robin 算法将 client 分配到每个 I/O 线程绑定的 `io_threads_list` 列表中。主线程负责 io\_threads\_list[0] 的任务。
5. 修改全局变量 `io_threads_op = IO_THREADS_OP_WRITE`，通知 I/O 线程处理的是可写事件。I/O 线程就会执行 `writeToClient()`函数将 Client 的响应写回客户端。
6. 主线程也会调用 `writeToClient()` 把 io\_threads\_list[0] 的 client 执行结果写回给客户端。
7. 主线程执行 `listEmpty(io_threads_list[0])`清空 io\_threads\_list[0] 列表。
8. 主线程阻塞等待其他 I/O 线程执行完对应的 `io_threads_list` 列表中的 Client 执行结果写回客户端。
9. 主线程将全局变量`io_threads_op` 设置成 `IO_THREADS_OP_IDLE`，表示 I/O 线程空闲。
10. 检查 `clients_pending_write` 中的 client 是否还有数据要响应给客户端。有的话就调用 `CT_Socket.set_write_handler` 函数将 `sendReplyToClient()` 函数设置为\``connection-> write_handler`回调函数。当连接事件变成可写时候，主线程会调用`sendReplyToClient()`函数，内部会调用`writeToClient()` 函数将 client 执行结果数据写回客户端。
11. 调用 `listEmpty(server.clients_pending_write)`清空 `clients_pending_write`队列。

一条 Redis 命令的执行过程到此结束，完结撒花。

# 第 2 章 核心筑基之数据结构与心法

我是 Redis，给开发者提供了 String（字符串）、Hashes（散列表）、Lists（列表）、Sets（无序集合）、Sorted Sets（可根据范围查询的排序集合）、Bitmap（位图）、HyperLogLog、Geospatial （地理空间）和 Stream（流）等数据类型。

接下来我要介绍的是，每种数据类型的使用技巧和使用场景，以及这些数据类型底层数据结构原理。**数据类型的使用技法和以及每种数据类型底层实现原理是你核心筑基必经之路，好好修炼。**筑基稳固，修炼心法，让程序更快还能做到极致节省内存。

## 2.1 String（字符串）

### 2.1.1 是什么

字符串类型的使用最为广泛，比如计数器、缓存、分布式锁、用于存储登录后的用户信息，key = token，value = Java 对象序列化成 JSON 后的字符串。

如下指令。

```
SET user:token:666 {"name": "码哥"，“gender”: “M”,“city”:"shenzhen"}
```

接下来，我先带你深入了解 String 类型，底层数据结构和使用场景。

> MySQL：“你都是用 C 语言开发出来的，C 语言本就有字符串，吓唬谁呢。”

格局能不能打开一点，我并没有直接使用 C 语言的字符串，而是自己搞了一个 SDS 结构体来表示字符串。SDS 的全称是 Simple Dynamic String，中文叫做“简单动态字符串”。

> MySQL：“搞 SDS 的目的是啥？”

字符串使用最为广泛，我要保证能支持**丰富和高性能**的字符串操作函数，**能保存二进制数据**，同时还能**节省内存**占用。实现了你们领导平时经常对你们提出的既要又要还要的目标。先看 **C 语言字符串数组的结构**。比如通过 `char *s = "MageByte"`定义字符串变量。

![图2-1][image-15]

图 2-1

注意，**数组的最后一个字符串是 "\0"，它表示字符串的结束**。

因为 C 语言标准库 `string.h`中的字符串有以下几点不足，所以我才设计了 SDS。

1. C 语言使用 `char*` 字符串数组来实现字符串，在创建字符串的时候就要需要手动检查和分配字符串空间。由于没有 `length`属性记录字符串长度，想要获取一个字符串长度就要从头开始遍历，直到 `\0`为止，作为唯快不破的我来说是不能容忍的。
2. 无法做到“**安全的二进制存储**”：比如图片等二进制数据无法保存。无法存储 `\0`这种特殊字符是因为 `\0` 在 C 语言字符串中表示结尾。
3. 字符串的扩容和缩容：char 数组的长度在创建字符串的时候就确定下来，如果想要追加数据，**要重新申请一块空间**，把追加后的字符串内容**拷贝**进去，再释放旧的空间，十分消耗资源。

### 2.1.2 修炼心法

> MySQL：“说说 SDS 结构体吧，你是如何解决这些问题的。”

为了存储字符串实际内容，我需要有一个 **char 类型数组**来存储，使用一个 int 类型的 **len** 字段用于记录 char 数组使用了多少字节。

除此之外，还要有一个 int 类型 的 alloc 字段记录分配的 char 数组总长度，`alloc - len` 就等于 char 类型的 buf 数组未使用的字节数（Redis 7.0 已经去掉了表示未使用字节数 free 字段）。

![图2-2][image-16]

图 2-2

**SDS 也遵循 C 字符串以空字符“\0”结尾的惯例，保存空字符的大小不计算在 SDS 的 len 属性中。**此外，添加空字符串“\0” 到字符串末尾等操作，都是由 SDS 函数自动完成的。

**O(1) 时间复杂度获取字符串长度**

SDS 中 len 保存了字符串的长度，实现了**O(1) 时间复杂度获取字符串长度。**你注意到了没，SDS 结构有一个 flags 字段，表示的是 SDS 类型。实际上 SDS 一共设计了 5 种类型，分别是`sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64`，区别在于数组的 len 长度和分配空间长度 alloc。

比如 sdshdr8。

```c
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len;
    uint8_t alloc;
    unsigned char flags;
    char buf[];
};
```

len、alloc 字段都是 uint8\_t 这个类型，在 Java 中 int 就是 32 位，而 C 语言里面有不同长度的 int 值，uint8\_t 就是占 8 位的无符号 int 值，能表示的最大值就是 2^8-1，那它的 buf 数组，最大长度就是 2^8 -1。

**节省内存**

之所以这么设计，就是**为了针对不同大小的字符串，使用不同的 SDS 类型保存，从而节省内存占用。**

> MySQL：“SDS 能存储多大的字符串？”

alloc 表示当前 sds 结构允许容纳的最大字符长度， 比如 `uint32_t alloc` 的取值范围是 `0~2^32 = 4294967296`。理论上 char 数组最大长度为 4294967296，一个 char 字符占用一个字节，可以存储 4 G，更不用说 sdshdr64 了。

**这些都是理论值，实际上 Redis 内部会限制最大的字符串长度是 512M。**

**编码格式**

我还对 String 类型的数据采用了三种编码格式来存储，分别是 int、embstr、raw，你可使用 `OBJECT encoding key` 来查值对象所使用的编码类型。

编码选择流程如图 2-3 所示。

![图 2-3][image-17]

图 2-3

- int 编码，8 个字节的长整型，值是数字类型且数字的长度小于 20。
- embstr，小于等于 44 字节的字符串。
- 大于 44 字节的字符串。

> MySQL：“`__attribute__ ((__packed__))`是什么玩意？”

这是我使用了专门的**编译优化手段来节省内存空间**。**作用就是告诉编译器，不要使用字节对齐的方式，而是采用紧凑的方式分配内存。**

默认情况下，编译器会按照 8 字节对齐的方式分配内存，即使这个变量的大小不到 8 字节。

使用了 `__attribute__ ((__packed__))` 定义结构体，编译器会**按照实际占用来分配内存空间。**

**二进制安全**

SDS 不仅可以存储 String 类型数据，还能存储二进制数据。SDS 并不是通过“\0” 来判断字符串结束，用的是 len 标志结束，所以可以直接将二进制数据存储。

**空间预分配**

在需要对 SDS 的空间进行扩容时，不仅仅分配所需的空间，还会分配额外的未使用空间。

**通过预分配策略，减少了执行字符串增长所需的内存重新分配次数，降低由于字符串增加操作的性能损耗。**

**惰性空间释放**

当对 SDS 进行缩短操作时，程序并不会回收多余的内存空间，如果后面需要 append 追加操作，则直接使用 buf 数组 `alloc - len`中未使用的空间。

**通过惰性空间释放策略，避免了减小字符串所需的内存重新分配操作，为未来增长操作提供了优化。**

### 2.1.3 出招实战：分布式 ID 生成器

我相信你会经常遇到要生成唯一 ID 的场景，比如标识每次请求、生成一个订单编号、创建用户需要创建一个用户 ID。

分布式 ID 生成器需要满足以下特性。

1. 有序性之单调递增，想要分而治之、二分法查找就必须实现。另外，MySQL 是你们用的最多的数据库，B+ 树为了维护 ID 的有序性，就会频繁的在索引的中间位置插入而挪动后面节点的位置，甚至导致频繁的页分裂，这对于性能的影响是极大的。
2. 全局唯一性，ID 不唯一就会出现主键冲突。
3. 高性能，生成 ID 是高频操作，如果性能缓慢，系统的整体性能都会受到限制。
4. 高可用，也就是在给定的时间间隔内，一个系统总的可用时间占的比例。
5. 存储空间小，用 MySQL 的 InnoDB B+树来说，普通索引（非聚集索引）会存储主键值，主键越大，每个 Page 页可以存储的数据就越少，访问磁盘 I/O 的次数就会增加。

Redis 集群能保证高可用和高性能，为了节省内存，ID 可以使用数字的形式，并且通过递增的方式来创建新的 ID。防止重启数据丢失，你还需要把 Redis AOF 持久化开启。

> MySQL：“开启 AOF 持久，为了性能设置成 everysec 策略还是有可能丢失一秒的数据，所以你还可以使用一个异步机制将生成的最大 ID 持久化到一个 MySQL。”

好主意，在生成 ID 之后发送一条消息到 MQ 消息队列中，把值持久化到 MySQL 中。

我提供了 `INCR` 指令，它能把 key 中存储的数字加 1 并返回客户端。如果 key 不存在，那么 key 的 value 先被初始化成 0，再执行加 1 操作并返回给客户端。

该指令的值限制在 64 位有符号数字之内。

**设计思路**

1. 假设订单 ID 生成器的 key 是“counter:order”，当应用服务启动的时候先从数据库中查询出最大值 M。执行 `EXISTS counter:order` 判断是否存在 key。

   - Redis 中不存在 key “counter:order”，执行 `SET counter:order M` 将 M 值作写入 Redis。
   - Redis 中存在 key “counter:order”，值为 K，那么就比较 M 和 K 的值，执行 `SET counter:order max(M, N)`将最大值写入 Redis，相等的话就不操作。

2. 应用服务启动完成后，每次需要生成 ID 的时候，应用程序就向 Redis 服务器发送 `INCR counter:order`指令。
3. 应用程序将获取到的 ID 值发送到 MQ 消息队列，消费者监听队列把值更新到 MySQL。

![图 2-4][image-18]

图 2-4

## 2.2 Lists（列表）

### 2.2.1 是什么

作为 Java 开发者的你，看到这个词并不陌生。在 Java 开发中几乎每天都会使用这个数据结构。

Redis 的 List 与 Java 中的 LinkedList 类似，是一种线性的有序结构，可以按照元素被推入列表中的顺序来存储元素，能满足先进先出的需求，这些元素既可以是文字数据，又可以是二进制数据。

你可以把他当做队列、栈来使用。

### 2.2.2 修炼心法

在 C 语言中，并没有现成的链表结构，所以 antirez 为我专门设计了一套实现方式。

关于 List 类型的底层数据结构，可谓英雄辈出，antirez 大佬一直在优化，创造了多种数据结构来保存。

从一开始早期版本使用 **linkedlist（双端列表）**和 **ziplist（压缩列表）**作为 List 的底层实现，到 Redis 3.2 引入了由 linkedlist + ziplist 组成的 **quicklist**，再到 7.0 版本的时候使用 **listpack** 取代 **ziplist**。

> MySQL：“为何弄了这么多数据结构呀？”

antirez 所做的这一切都是为了在内存空间开销与访问性能之间做取舍和平衡，跟着我去吃透每个类型的设计思想和不足，你就明白了。

#### linkedlist（双端列表）

在 Redis 3.2 版本之前，List 的底层数据结构由 linkedlist 或者 ziplist 实现，优先使用 ziplist 存储。

**当列表对象满足以下两个条件的时候，List 将使用 ziplist 存储，否则使用 linkedlist。**

- **List 的每个元素的占用的字节小于 64 字节。**
- **List 的元素数量小于 512 个。**

链表的节点使用 `adlist.h/listNode`结构来表示。

```c
typedef struct listNode {
    // 前驱节点
    struct listNode *prev;
    // 后驱节点
    struct listNode *next;
    // 指向节点的值
    void *value;
} listNode;
```

`listNode` 之间通过 prev 和 next 指针组成双端链表。除此之外，我还提供了 `adlist.h/list` 结构提供了头指针 head、尾指针 tail 以及一些实现多态的特定函数。

```c
typedef struct list {
    // 头指针
    listNode *head;
    // 尾指针
    listNode *tail;
    // 节点值的复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值比对是否相等
    int (*match)(void *ptr, void *key);
    // 链表的节点数量
    unsigned long len;
} list;
```

linkedlist 的结构如图 2-5 所示。

![图 2-5][image-19]

图 2-5

Redis 的链表实现的特性总结如下。

- 双端：链表节点带有 prev 和 next 指针，获取某个节点的前置节点和后继节点的复杂度都是 O(1)。
- 无环：表头节点的 prev 指针和尾节点的 next 指针都指向 NULL，对链表的访问以 NULL 为结束。
- 带表头指针和表尾指针：通过 list 结构的 head 指针和 tail 指针，程序获取链表的头节点和尾节点的复杂度为 O(1)。
- 使用 list 结构的 len 属性来对记录节点数量，获取链表中节点数量的复杂度为 O(1)。

> MySQL：“看起来没啥问题呀，为啥还要 ziplist 呢？”

你知道的，我在追求快和节省内存的方向上无所不及，有两个原因导致了 ziplist 的诞生。

- 普通的 linkedlist 有 prev、next 两个指针，**当存储数据很小的情况下，指针占用的空间会超过数据占用的空间**，这就离谱了，是可忍孰不可忍。
- linkedlist 是链表结构，在内存中不是连续的，遍历的效率低下。

#### ziplist（压缩列表）

为了解决上面两个问题，antirez 创造了 ziplist 压缩列表，是一种内存紧凑的数据结构，占用一块连续的内存空间，提升内存使用率。

**当一个列表只有少量数据的时候，并且每个列表项要么是小整数值，要么就是长度比较短的字符串，那么我就会使用 ziplist 来做 List 的底层实现。**

ziplist 中可以包含多个 entry 节点，每个**节点可以存放整数或者字符串**，结构如图 2-6 所示。

![图 2-6][image-20]

图 2-6

- zlbytes，占用 4 个字节，记录了整个 ziplist 占用的总字节数。
- zltail，占用 4 个字节，指向最后一个 entry 偏移量，用于快速定位最后一个 entry。
- zllen，占用 2 字节，记录 entry 总数。
- entry，列表元素。
- zlend，ziplist 结束标志，占用 1 字节，值等于 255。

因为 ziplist 头尾元数据的大小是固定的，并且在 ziplist 头部 zllen 记录了最后一个元素的位置，所以，当在 ziplist 中查找第一个或最后一个元素的时候，能以 O(1) 时间复杂度找到。

**而查找中间元素时，只能从列表头或者列表尾遍历，时间复杂度就是 O(N)。**

接下来看真正存储数据的 entry 结构长啥样。

![图 2-7][image-21]

图 2-7

正常来说有三部分构成 `<prevlen> <encoding> <entry-data>`。

**prevlen**

记录前一个 entry 占用字节数，能实现逆序遍历就是靠这个字段确定往前移动多少字节拿到上一个 entry 首地址。

这部分会根据上一个 entry 的长度进行变长编码（为了节省内存操碎了心），变长方式如下。

- 前一个 entry 的字节大小小于 254（255 用于 zlend），prevlen 长度为 1 字节，值等于上一个 entry 的长度。
- 前一个 entry 的字节大小大于等于 254，prevlen 占用 5 字节，第一个字节设置为 254 作为一个标识，后面四字节组成一个 32 位的 int 值，用于存放上一个 entry 的字节长度。

**encoding**

简言之用于表示当前 entry 的类型和长度，当前 entry 的长度和值是根据保存的是 int 还是 string 以及数据的长度共同来决定。

前两位用于表示类型，当前两位值为 “11” 则表示 entry 存放的是 int 类型数据，其他表示存储的是 string。

**entry-data**

实际存放数据的区域，需要注意的是，**如果 entry 中存储的是 int 类型，encoding 和 entry-data 会合并到 encoding 中，没有 entry-data 字段。**

此刻结构就变成了 `<prevlen> <encoding>`。

> MySQL：“为什么说 ziplist 省内存？”

1. 与 linkedlist 相比，少了 prev、next 指针。
2. 通过 encoding 字段针对不同编码来细化存储，尽可能做到按需分配，当 entry 存储的是 int 类型时，encoding 和 entry-data 会合并到 encoding ，省掉了 entry-data 字段。
3. 每个 entry-data 占据内存大小不一样，为了解决遍历问题，增加了 prevlen 记录上一个 entry 长度。遍历数据时间复杂度是 O(1)，但是数据量很小的情况下影响不大。

> MySQL：“听起来很完美，为啥还搞什么 quicklist ”

既要又要还要的需求是很难实现的，ziplist 节省了内存，但是也有不足。

- 不能保存过多的元素，否则查询性能会大大降低，O(N) 时间复杂度。
- ziplist 存储空间是连续的，当插入新的 entry 时，内存空间不足就需要重新分配一块连续的内存空间，引发连锁更新的问题。

**连锁更新**

每个 entry 都用 prevlen 记录了上一个 entry 的长度，从当前 entry B 前面插入一个新的 entry A 时，会导致 B 的 prevlen 改变，也会导致 entry B 大小发生变化。entry B 后一个 entry C 的 prevlen 也需要改变。以此类推，就可能造成了连锁更新。

![图 2-8][image-22]

图 2-8

连锁更新会导致 ziplist 的内存空间需要多次重新分配，直接影响 ziplist 的查询性能。于是乎在 Redis 3.2 版本引入了 quicklist。

#### quicklist

quicklist 是综合考虑了时间效率与空间效率引入的新型数据结构。**结合了原先 linkedlist 与 ziplist 各自的优势，本质还是一个链表，只不过链表的每个节点是一个 ziplist。**

数据结构定义在 `quicklist.h` 文件中，链表由 `quicklist` 结构体定义，每个节点由 `quicklistNode` 结构体定义（源码版本为 6.2，7.0 版本使用 listpack 取代了 ziplist）。

quicklist 是一个双向链表，所以每个 quicklistNode 都有前序指针（`*prev`）、后序指针（`*next`）。每个节点是 ziplist，所以还有一个指向 ziplist 的指针 `*zl`。

```c
typedef struct quicklistNode {
    // 前序节点指针
    struct quicklistNode *prev;
    // 后序节点指针
    struct quicklistNode *next;
    // 指向 ziplist 的指针
    unsigned char *zl;
    // ziplist 字节大小
    unsigned int sz;
    // ziplst 元素个数
    unsigned int count : 16;
    // 编码格式，1 = RAW 代表未压缩原生ziplist，2=LZF 压缩存储
    unsigned int encoding : 2;
    // 节点持有的数据类型，默认值 = 2 表示是 ziplist
    unsigned int container : 2;
    // 节点持有的 ziplist 是否经过解压， 1 表示已经解压过，下一次操作需要重新压缩。
    unsigned int recompress : 1;
    // ziplist 数据是否可压缩，太小数据不需要压缩
    unsigned int attempted_compress : 1;
    // 预留字段
    unsigned int extra : 10;
} quicklistNode;
```

quicklist 作为链表，定义了 头、尾指针，用于快速定位表表头和链表尾。

```c
typedef struct quicklist {
    // 链表头指针
    quicklistNode *head;
    // 链表尾指针
    quicklistNode *tail;
    // 所有 ziplist 的总 entry 个数
    unsigned long count;
    // quicklistNode 个数
    unsigned long len;
    int fill : QL_FILL_BITS;
    unsigned int compress : QL_COMP_BITS;
    unsigned int bookmark_count: QL_BM_BITS;
    // 柔性数组，给节点添加标签，通过名称定位节点，实现随机访问的效果
    quicklistBookmark bookmarks[];
} quicklist;
```

结合 `quicklist 和 quicklistNode`定义，quicklist 链表结构如下图所示。

![图 2-9][image-23]

图 2-9

从结构上看，quicklist 就是 ziplist 的升级版，优化的关键点在于控制好每个 ziplist 的大小或者元素个数。

- quicklistNode 的 ziplist 越小，可能会造成更多的内存碎片，极端情况下是每个 ziplist 只有一个 entry，退化成了 linkedlist。
- quicklistNode 的 ziplist 过大，极端情况下一个 quicklist 只有一个 ziplist，退化成了 ziplist。连锁更新的性能问题就会暴露无遗。

合理配置很重要，Redis 提供了 `list-max-ziplist-size -2`，

当 `list-max-ziplist-size` **为负数时表示限制每个 quicklistNode 的 ziplist 的内存大小**，超过这个大小就会使用 linkedlist 存储数据，每个值有以下含义：

- -5：每个 quicklist 节点上的 ziplist 大小最大 64 kb \<--- 正常环境不推荐
- -4：每个 quicklist 节点上的 ziplist 大小最大 32 kb \<--- 不推荐
- -3：每个 quicklist 节点上的 ziplist 大小最大 16 kb \<--- 可能不推荐
- -2：每个 quicklist 节点上的 ziplist 大小最大 8 kb \<--- 不错
- -1：每个 quicklist 节点上的 ziplist 大小最大 4kb \<--- 不错

默认值为 -2，也是官方最推荐的值，当然你可以根据自己的实际情况进行修改。

> MySQL：“搞了半天还是没能解决连锁更新的问题嘛”

别急，饭要一口口吃，路要一步步走，步子迈大了容易扯着蛋。

ziplist 是紧凑型数据结构，可以有效利用内存。但是每个 entry 都用 `prevlen` 保留了上一个 entry 的长度，所以在插入或者更新时可能会出现连锁更新影响效率。

于是 antirez 又设计出了“链表 + ziplist” 组成的 quicklist 来避免单个 ziplist 过大，降低连锁更新的影响范围。

可毕竟还是使用了 ziplist，本质上无法避免连锁更新的问题，于是乎在 5.0 版本设计出另一个内存紧凑型数据结构 listpack，于 7.0 版本替换掉 ziplist。

#### listpack

出现 listpack 的原因是因为用户上报了一个 Redis 崩溃的问题，但是 antirez 并没有找到崩溃的明确原因，猜测可能是 ziplist 结构导致的连锁更新导致的，于是就想设计一种简单、高效的数据结构来替换 ziplist 这个数据结构。

> MySQL：“listpack 是啥？”

**listpack 也是一种紧凑型数据结构，用一块连续的内存空间来保存数据，并且使用多种编码方式来表示不同长度的数据来节省内存空间。**

源码文件 `listpack.h`对 listpack 的解释：A lists of strings serialization format，意思是一种字符串列表的序列化格式，可以把字符串列表进行序列化存储，可以存储字符串或者整形数字。

先看 listpack 的整体结构。

![图 2-10][image-24]

图 2-10

一共四部分组成，tot-bytes、num-elements、elements、listpack-end-byte。

- tot-bytes，也就是 total bytes，占用 4 字节，记录 listpack 占用的总字节数。
- num-elements，占用 2 字节，记录 listpack elements 元素个数。
- elements，listpack 元素，保存数据的部分。
- listpack-end-byte，结束标志，占用 1 字节，值固定为 255。

> MySQL：“好家伙，这跟 ziplist 有啥区别？别以为换了个名字，换个马甲我就不认识了”

听我说完！确实有点像，listpack 也是由元数据和数据自身组成。最大的区别是 elements 部分，为了解决 ziplist 连锁更新的问题，element **不再像 ziplist 的 entry 保存前一项的长度**。

![图 2-11][image-25]

图 2-11

- encoding-type，元素的编码类型，会不同长度的整数和字符串编码。
- element-data，实际存放的数据。
- element-tot-len，encoding-type + element-data 的总长度，不包含自己的长度。

**每个 element 只记录自己的长度，不像 ziplist 的 entry，记录上一项的长度。当修改或者新增元素的时候，不会影响后续 element 的长度变化，解决了连锁更新的问题。**

从 **linkedlist**、 **ziplist** 到“链表 + ziplist” 构成的 **quicklist**，再到 **listpack** 结构。可以看到，设计的初衷都是能够高效的使用内存，同时避免性能下降。

### 2.2.3 出招实战：消息队列

学完了 List 的底层数据结构，终于到我（Redis）大显身手上才艺搞实战的环节了。

分布式系统中必备的一个中间件就是消息队列，通过消息队列你能对服务间进行异步解耦、流量消峰、实现最终一致性。

目前市面上已经有 `RabbitMQ、RochetMQ、ActiveMQ、Kafka`等，有人会问：“Redis 适合做消息队列么？”

在回答这个问题之前，你先从本质思考。

- 消息队列提供了什么特性？
- Redis 如何实现消息队列？是否满足存取需求？

我将结合消息队列的特点，分析使用 Redis 的 List 作为消息队列的实现原理，并分享如何把 SpringBoot 与 Redission 整合来操作 Redis 运用到项目中。

学会这招，今年的优秀员工就是你的了。

#### 什么是消息队列

消息队列是一种异步的服务间通信方式，适用于分布式和微服务架构。消息在被处理和删除之前一直存储在队列上。

每条消息仅可被一位用户处理一次。消息队列可被用于分离重量级处理、缓冲或批处理工作以及缓解高峰期工作负载。

![图2-12][image-26]

图 2-12

- Producer：消息生产者，负责产生和发送消息到 Broker；
- Broker：消息处理中心。负责消息存储、确认、重试等，一般其中会包含多个 queue；
- Consumer：消息消费者，负责从 Broker 中获取消息，并进行相应处理；

> MySQL：“消息队列的使用场景有哪些呢？“

消息队列在实际应用中包括如下四个场景。

- 应用耦合：发送方、接收方系统之间不需要了解双方，只需要认识消息。多应用间通过消息队列对同一消息进行处理，避免调用接口失败导致整个过程失败。
- 异步处理：多应用对消息队列中同一消息进行处理，应用间并发处理消息，相比串行处理，减少处理时间。
- 限流削峰：广泛应用于秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况。
- 消息驱动的系统：系统分为消息队列、消息生产者、消息消费者，生产者负责产生消息，消费者(可能有多个)负责对消息进行处理。

#### 消息队列满足哪些特性

**消息有序性**

消息是异步处理的，但是消费者需要按照生产者发送消息的顺序来消费，避免出现后发送的消息被先处理的情况。

**重复消息处理**

生产者可能因为网络问题出现消息重传导致消费者可能会收到多条重复消息。

同样的消息重复多次的话可能会造成一业务逻辑多次执行，需要确保如何避免重复消费问题。

**可靠性**

一次保证消息的传递。如果发送消息时接收者不可用，消息队列会保留消息，直到成功地传递它。

当消费者重启后，可以继续读取消息进行处理，防止消息遗漏。

**LPUSH**

生产者使用 `LPUSH key element[element...]` 将消息插入到队列的头部，如果 key 不存在则会创建一个空的队列再插入消息。

如下，生产者向队列 queue 先后插入了 “Java”、“码哥字节”、“Go”，返回值表示消息插入队列后的个数。

```bash
> LPUSH queue Java 码哥字节 Go
(integer) 3
```

> MySQL：“如果生产者消息发送很快，消费者处理不过来，会导致消息积压，占用过多的 Redis 内存。”

确实，List 并没有提供类似于 Kafka 的 ConsumeGroup ，会使用多个消费者策划给你续组成一个消费组来分担处理队列消息。不过在 Redis 5.0 之后，提供了 Streams 数据类型，后面我会介绍到。

**RPOP**

消费者使用 `RPOP key` 依次读取队列的消息，先进先出，所以 “Java”会先读取消费：

```bash
> RPOP queue
"Java"
> RPOP queue
"码哥字节"
> RPOP queue
"Go"
```

![图2-13][image-27]

图 2-13

#### 实时消费问题

> 谢霸戈：“这么简单就实现了？”

别高兴的太早，`LPUSH、RPOP` 存在一个性能风险，生产者向队列插入数据的时候，List 并不会主动通知消费者及时消费。

> 谢霸戈：“那我写一个 `while(true)` 不停地调用 `RPOP` 指令，当有新消息就消费“

程序需要不断轮询并判断是否为空再执行消费逻辑，这就会导致即使没有新消息写入队列，消费者也在不停地调用 `RPOP` 命令占用 `CPU` 资源。

> 谢霸戈：“如何避免循环调用导致的 CPU 性能损耗呢？”

请叫我贴心哥 Redis，我提供了 `BLPOP、BRPOP` 阻塞读取的命令，**消费者在读取队列没有数据的时候自动阻塞，直到有新的消息写入队列，才会继续读取新消息执行业务逻辑。**

```
BRPOP queue 0
```

参数 0 表示阻塞等待时间无止期，哪怕是烟花易冷人事易分，雨纷纷旧故里草木深，斑驳的城门盘踞着老树根，石板上回荡的是再等，一直等到“心上人”来。

#### 重复消费解决方案

- 消息队列为自动每一条消息生成一个全局 ID；
- 生产者为每一条消息创建一个全局 ID，消费者把处理过的消息 ID 记录下来判断是否重复。

其实这就是幂等，对于同一条消息，消费者收到后处理一次的结果和多次的结果是一致的。

#### 消息可靠性解决方案

> 谢霸戈：“消费者读取消息，处理过程中宕机了就会导致消息没有处理完成，可是数据已经不在队列中了咋办？”

本质就是消费者在处理消息的时候崩溃了，无法再读取消息，缺乏一个消息确认可靠机制。

我提供了 `BRPOPLPUSH source destination timeout`指令，含义是阻塞的方式从 `source` 队列读取消息的同时把这条消息复制到另一个 `destination` 队列中（备份），并且是原子操作。

不过这个指令在 6.2 版本被 `BLMOVE` 取代。接下来，上才艺！生产者使用 `LPUSH` 把消息依次从存入 `order:pay` 队列队头（左端）。

```bash
LPUSH order:pay "谢霸戈"
LPUSH order:pay "肖材吉"
```

消费者消费消息的时候在 `while`循环使用`BLMOVE` 以阻塞的方式从队列 `order:pay` 队尾（右端）弹出消息“谢霸戈”，同时把该消息复制到队列 `order:pay:back` 队头（左端），该操作是原子性的，最后一个参数 timeout = 0 表示持续等待。

```bash
BLMOVE order:pay order:pay:back RIGHT LEFT 0
```

如果消费消息“谢霸戈”成功，那就使用 `LREM` 把队列 `order:pay:back` 的“谢霸戈”消息删除，从而实现 ACK 确认机制。

```bash
LREM order:pay:back 0 "谢霸戈"
```

倒数第二个参数 count 的含义如下。

- count \> 0，从表头（左端）向表尾（右端），依次删除 count 个 value。
- count \< 0，从表尾（右端）向表头（左端），依次删除 count 绝对值个 value。
- count = 0，删除所有的 value。

消费异常的话，应用程序使用 `BRPOP order:pay:back` 从备份队列再次读取消息处理即可。

![图2-14][image-28]

图 2-14

## 2.3 Sets（无序集合）

Sets 无序集合，他的功能就好像你熟悉的 Java 中的 HashSet 一样。集合是通过散列表实现的，所以添加、删除、查找元素的时间复杂度是 O(1)。

### 2.3.1 是什么

Sets 是 String 类型的无序集合，集合中的元素是唯一的，集合中**不会出现重复的数据**。

Java 的 HashSet 底层是用 HashMap 实现，Sets 的底层数据结构也是用 Hashtable（散列表）实现，散列表的 key 存的是 Sets 集合元素的 value，散列表的 value 则指向 NULL。。

不同的是，当元素内容都是 64 位以内的十进制整数的时候，并且元素个数不超过 `set-max-intset-entries` 配置的值（默认 512）的时候，会使用更加省内存的 intset（整形数组）来存储。

![图2-15][image-29]

图 2-15

**使用场景**

当你需要存储多个元素，并且要求不能出现重复数据，无需考虑元素的有序时，就可以使用 Sets 来存储，这样能利用我对单个元素操作 O(1) 时间复杂度带来的性能优势。

并且 Sets 还支持在集合之间做交集、并集、差集操作，比如当你遇到如下场景，需要统计多个集合元素的聚合结果。

- 统计多个元素的共有数据（交集）。
- 统计两个集合其中的一个独有元素（差集统计）。
- 统计多个集合的所有元素（并集统计）。

常见的使用场景。

1. 社交软件中共同关注，通过交集实现。
2. 每日新增关注数，只需要对近两天的总注册用户量集合取差集即可。
3. 打标签：比如微信收藏功能，你可以为自己收藏的每一篇文章打标签，这样你可以快速的找到被添加了某个标签的所有文章。

### 2.3.2 修炼心法

关于散列表结构我会在专门的章节介绍，先看 intset 结构，结构体定义在源码 `intset.h`中。

```c
typedef struct intset {
    uint32_t encoding;
    uint32_t length;
    int8_t contents[];
} intset;
```

- length，记录整数集合存储的元素个数，其实就是 contents 数组的长度。
- contents，真正存储整数集合的数组，是一块连续内存区域。每个元素都是数组的一个数组元素，数组中的元素会按照值的大小从小到大有序排列存储，并且不会有重复元素。
- encoding，编码格式，决定数组类型，一共有三种不同的值。
  - INTSET\_ENC\_INT16，表示 contents 数组的存储元素是 int16\_t 类型，每 2 字节表示一个整数元素。
  - INTSET\_ENC\_INT32，表示 contents 数组的存储元素是 int32\_t 类型，每 4 字节表示一个元素。
  - INTSET\_ENC\_INT64，表示 contents 数组的存储元素是 int64\_t 类型，每 8 字节表示一个元素。

![图2-16][image-30]

图 2-16

> MySQL：“如果在一个 int16\_t 类型的整数集合中插入一个 int64\_t 类型的值会怎样？”

这个问题问得好，下次可以继续保持。

这种情况会触发整数集合升级，也就是集合的所有元素都会转换成 int64\_t 类型，步骤如下。

1. 根据新元素的类型，以及集合元素的数量，包括新添加的元素在内，计算新的空间大小，对底层数组空间扩容，进行空间重新分配。
2. 将数组原有的元素都转换成新元素类型，把转换后的元素按照从大到小的顺序放到正确的位置上，**需要保证数组元素的有序性**。
3. 修改 encoding 的值，length + 1。

所以每次向整形数组集合添加新元素都可能会引起升级，升级又会对原始数据进行类型转换，时间复杂度是 O(N)。

> MySQL：“如果删除刚刚添加的 int64\_t 类型元素，会执行降级操作么?”

整形数组不支持降级操作。

> MySQL：“Sets 是无序集合，为何存储整形数字的场景下 contents 数组元素需要有序？”

**为了查询元素速度，数组有序我就能使用二分法来提高查询效率**。`insetFind()` 函数返回值等于 0 表示集合中没有目标数据，反之 1 存在目标数据。方法的内部会调用 `intsetSearch()` 函数使用二分法来实现。

```c
static uint8_t intsetSearch(intset *is, int64_t value, uint32_t *pos) {
    int min = 0, max = intrev32ifbe(is->length)-1, mid = -1;
    int64_t cur = -1;
    // 省略一些检查代码

    while(max >= min) {
        mid = ((unsigned int)min + (unsigned int)max) >> 1;
        cur = _intsetGet(is,mid);
        if (value > cur) {
            min = mid+1;
        } else if (value < cur) {
            max = mid-1;
        } else {
            break;
        }
    }
	// 修改 pos 指针
    if (value == cur) {
        if (pos) *pos = mid;
        return 1;
    } else {
        if (pos) *pos = min;
        return 0;
    }
}

```

pos 指针的作用有两个，如果查找到目标值， pos 记录目标值的位置；查找不到目标值，pos 记录的就是这个目标值插入到 intset 的位置。

### 2.3.3 出招实战：共同好友

三国天下有限公司开发了一个名叫“三国恋”的社交 APP，想要实现共同好友功能，这个场景就能使用集合交集来实现。为每个用户创建一个 Sets 集合，账号名作为集合的 key，集合 value 存储该账号的好友。

如下指令构建刘备和曹操的好友集合。

```bash
SADD user:刘备 赵子龙 张飞 关羽 貂蝉
SADD user:曹操 貂蝉 夏侯惇 典韦 张辽
```

想要知道两个人的共同好友，也就是两个集合的交集，只需要使用 `SINTERSTORE`指令。

```bash
SINTERSTORE user:曹刘好友 user:刘备 user:曹操
```

命令执行后，刘备与曹操两个集合的交集数据就存储到了“user:曹刘好友”集合中。使用 `SMEMBERS` 查看曹操与刘备的共同好友。

```bash
redis> SMEMBERS user:曹刘好友
1) "貂蝉"
```

好家伙，他们都喜欢貂蝉，你喜不喜欢呢？

![图2-17][image-31]

图 2-17

## 2.4 Hash（散列表）

### 2.4.1 是什么

Redis Hash（散列表）是一种 field-value pairs（键值对）集合类型，类似于 Python 中的字典、Java 中的 HashMap。一个 field 对应一个 value，你可以通过 field 在 O(1) 时间复杂度查 field 找关联的 field，也可以通过 field 来更新或者删除这个键值对。

Redis 的散列表 dict 由**数组 + 链表**构成，数组的每个元素占用的槽位叫做**哈希桶**，当出现散列冲突的时候就会在这个桶下挂一个链表，用“**拉链法”解决散列冲突的问题**。

简单地说就是将一个 key 经过散列计算均匀的映射到散列表上。

![图 2-18][image-32]

图 2-18

### 2.4.2 修炼心法

Hashes 数据类型底层存储数据结构实际上有两种。

1. dict 结构。
2. 在 7.0 版本之前使用 ziplist，之后被 listpack 代替。

通常情况下使用 dict 数据结构存储数据，每个 `field-value pairs` 构成一个 dictEntry 节点来保存。

只有同时满足以下两个条件的时候，才会使用 listpack（7.0 版本之前使用 ziplist）数据结构来代替 dict 存储， **把 key-value 键值对按照 field 在前 value 在后，紧密相连的方式放到一次把每个键值对放到列表的表尾**。

- 每个键值对中的 field 和 value 的字符串字节大小都小于`hash-max-listpack-value` 配置的值（默认 64）。
- field-value pairs 键值对数量小于 `hash-max-listpack-entries`配置的值（默认 512）。

每次向散列表写数据的时候，都会调用 `t_hash.c` 中的`hashTypeConvertListpack()`函数来判断是否需要转换底层数据结构。

当插入和修改的数据不满足以上两个条件时，就把散列表底层存储结构转换成 `dict`结构。需要注意的是，**不能由 dict 退化成 listpack**。

虽然使用了 listpack 就无法实现 O(1) 时间复杂度操作数据，但是**使用 listpack 能大大减少内存占用，而且数据量比较小，性能并不是有太大差异。**

**为了对上层屏蔽散列表底层使用了不同数据结构存储，所以抽象了一个 hashTypeIterator 迭代器来实现散列表的查询。**

Hashes 数据类型使用 listpack 作为存储数据时的情况，如图 2-19 所示。

![图 2-19][image-33]

图 2-19

listpack 数据结构在之前的已经介绍过， 接下来带你揭秘 dict 到底长啥样。

**Redis 数据库就是一个全局散列表**。正常情况下，我只会使用 `ht_table[0]`散列表，图 2-20 是一个没有进行 rehash 状态下的字典。

![图 2-20][image-34]

图 2-20

dict 字典在源代码 `dict.h`中使用 dict 结构体表示。

```c
struct dict {
    dictType *type;
		// 真正存储数据的地方，分别存放两个指针
    dictEntry **ht_table[2];
    unsigned long ht_used[2];

    long rehashidx;

    int16_t pauserehash;
    signed char ht_size_exp[2];
};
```

- `dictType *type`，存放函数的结构体，定义了一些函数指针，可以通过设置自定义函数，实现 dict 的 key 和 value 存放任何类型的数据。
- 重点看 `dictEntry **ht_table[2]`，存放了两个 dictEntry 的二级指针，指针分别指向了一个 dictEntry 指针的数组。
- `ht_used[2]`，记录每个散列表使用了多少槽位（比如数组长度 32，使用了 12）。
- `rehashidx`，用于标记是否正在执行 rehash 操作，-1 表示没有进行 rehash。如果正在执行 rehash，那么其值表示当前 rehash 操作执行的 ht\_table[0] 散列表 dictEntry 数组的索引。
- `pauserehash` 表示 rehash 的状态，大于 0 时表示 rehash 暂停了，小于 0 表示出错了。

继续看 **dictEntry**，数组中每个元素都是 dictEntry 类型，就是这玩意存放了键值对，表示字典的一个节点。

```c
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

- `*key`指针指向键值对中的键，实际上指向一个 SDS 实例。
- `v`是一个 union 联合体，表示键值对中的值，同一时刻只有一个字段有值，用联合体的目是节省内存。
  - `*val` 如果值是非数字类型，那就使用这个指针存储。
  - `uint64_t u64`，值是无符号整数的时候使用这个字段存储。
  - `int64_t s64`，值是有符号整数时，使用该字段存储。
  - `double d`，值是浮点数是，使用该字段存储。
- `*next`指向下一个节点指针，当散列表数据增加，可能会出现不同的 key 得到的哈希值相等，也就是说多个 key 对应在一个哈希桶里面，这就是哈希冲突。Redis 使用拉链法，也就是用链表将数据串起来。

> MySQL：“为啥 ht\_table[2] 存放了两个指向散列表的指针？用一个散列表不就够了么。”

默认使用 `ht_table [0]` 进行读写数据，当散列表的数据越来越多的时候，哈希冲突严重会出现哈希桶的链表比较长，导致查询性能下降。

我为了唯快不破想了一个法子，当散列表保存的键值对太多或者太少的时候，需要通过 rehash（重新散列）对散列表进行扩容或者缩容。

**扩容和缩容**

1. 为了高性能，减少哈希冲突，我会创建一个大小等于 `ht_used[0] * 2`的散列表 `ht_table[1]`，也就是每次扩容时根据散列表 `ht_table [0]`已使用空间扩大一倍创建一个新散列表`ht_table [1]`。反之，如果是缩容操作，就根据`ht_table [0]`已使用空间缩小一倍创建一个新的散列表。

2. 重新计算键值对的哈希值，得到这个键值对在新散列表 `ht_table [1]`的桶位置，将键值对迁移到新的散列表上。
3. 所有键值对迁移完成后，修改指针，释放空间。具体是把 `ht_table[0]`指针指向扩容后的散列表，回收原来小的散列表内存空间，`ht_table[1]`指针指向`NULL`，为下次扩容或者缩容做准备。

> MySQL：“什么时候会触发扩容？”

1. 当前没有执行 `BGSAVE`或者 `BGREWRITEAOF`命令，同时负载因子大于等于 1。也就是当前没有 RDB 子进程和 AOF 重写子进程在工作，毕竟这俩操作还是比较容易对性能造成影响的，就不扩容火上浇油了。
2. 正在执行 `BGSAVE`或者 `BGREWRITEAOF`命令，负载因子大于等于 5。（这时候哈希冲突太严重了，再不触发扩容，查询效率太慢了）。

`负载因子 = 散列表存储 dictEntry 节点数量 / 散列表桶个数`。完美情况下，每个哈希桶存储一个 dictEntry 节点，这时候负载因子 = 1。

> MySQL：“需要迁移数据量很大，rehash 操作岂不是会长时间阻塞主线程？”

**为了防止阻塞主线程造成性能问题，我并不是一次性把全部的 key 迁移，而是分多次，将迁移操作分散到每次请求中，避免集中式 rehash 造成长时间阻塞，这个方式叫渐进式 rehash**。

在执行渐进式 rehash 期间，dict 会同时使用 `ht_table[0]` 和 `ht_table[1]`两个散列表，rehash 具体步骤如下。

1. 将 `rehashidx`设置成 0，表示 rehash 开始执行。
2. 在 rehash 期间，服务端每次处理客户端对 dict 散列表执行添加、查找、删除或者更新操作时，除了执行指定操作以外，还会检查当前 dict 是否处于 rehash 状态，是的话就把散列表`ht_table[0]`上索引位置为 `rehashidx` 的桶的链表的所有键值对 rehash 到散列表 `ht_table[1]`上，这个哈希桶的数据迁移完成，就把 `rehashidx` 的值加 1，表示下一次要迁移的桶所在位置。
3. 当所有的键值对迁移完成后，将 `rehashidx`设置成 -1，表示 rehash 操作已完成。

> MySQL：“rehash 过程中，字典的删除、查找、更新和添加操作，要从两个 ht\_table 都搞一遍么？”

删除、修改和查找可能会在两个散列表进行，第一个散列表没找到就到第二个散列表进行查找。但是增加操作只会在新的散列表上进行。

> MySQL：“如果请求比较少，岂不是会很长时间都要使用两个散列表。”

好问题，在 Redis Server 初始化时，会注册一个时间事件，定时执行 `serverCron` 函数，其中包含 rehash 操作用于辅助迁移，避免这个问题。

`serverCron` 函数主要处理如下工作。

- 过期 key 删除。
- 监控服务运行状态。
- 更新统计数据。
- 渐进式 rehash。
- 触发 BGSAVE / AOF rewrite 以及停止子进程。
- 处理客户端超时。
- ......

### 2.4.3 出招实战：购物车

分析下在线购物 APP 的购物车功能。

- 用户可以添加商品到购物车。
- 浏览购物车的所有商品。
- 更新某个商品的数量（增加或者减少）以及查看商品信息（价格、图片，描述等）。
- 删除商品。
- 清空购物车。

![图 2-21][image-35]

图 2-21

在这里仅仅讨论购物车的模型设计，不涉及购物车与数据库的同步、购物车与订单的关系等问题。

为每个用户创建一个 Hashes 集合存存储购物车信息，`key = shoppingCart:用户ID`，value 就是购物车信息。

![图 2-22][image-36]

图 2-22

**添加商品**

商品的编码作为 field，购买数量作为 value，如果要添加商品的话就向集合中新增 `field-value pairs` 。

**假设用户的 ID 是 660，鼠标的商品编码为 `SUPPLY`，耳机的商品编码为 `WF-1000XM4`，添加两个商品到购物车执行如下指令实现。**

```bash
HMSET shoppingCart:660 SUPPLY 1 WF-1000XM4 1
```

**修改商品数量**

多买一个 WF-1000XM4 降噪耳机，买俩！在职场中耳机一戴，谁都不爱，必须买。

```bash
HINCRBY shoppingCart:660 WF-1000XM4 1
```

指令的含义是对 key 为 `shoppingCart:660`的 Hashes 集合中 `field` 是 WF-1000XM4 的 value 与给定值 `1`做相加操作。如果想减少商品数量呢，那就把参数改成 `-1`即可。

**查看商品总量**

查看购物车商品总数量，只要知道 Hashes 中有多少个 `field-value pairs`即可。

```bash
HLEN shoppingCart:660
```

**全选**

获取购物车**所有商品的商品编码和数量**。

```bash
> HGETALL shoppingCart:660
SUPPLY
1
WF-1000XM4
2
```

每个 field 后面紧跟着 value。

**删除商品**

删除购物车 `shoppingCart:660` 中编号为 `SUPPLY`的鼠标，也就是删除 Hashes 集合中 field = SUPPLY 的键值对。

```bash
HDEL shoppingCart:660 SUPPLY
```

**清空购物车**

就是把整个 Hashes 集合删除。

```bash
DEL shoppingCart:660
```

**查询商品明细**

> MySQL：“当前设计并没有提升购物车查询性能呀，还要使用商品编号去数据库查询商品明细信息（价格、图片地址、文字描述等）。”

问得好，商品明细信息是不会随着用户购物车而变化的，本着分离变与不变原则，你可以**开辟一个独立的 Hashes 集合专门保存商品明细信息，让查询商品明细的请求先从这个集合获取数据，获取不到再查询数据库**，并把从数据库查到的数据写到这个集合中。field 保存商品编码，value 存储这个商品明细信息 json 字符串。

![图2-23][image-37]

图 2-23

如下指令，创建一个名为 `goods:info`的 Hashes 集合，并把商品编号为 `WF-1000XM4`的降噪耳机图片地址、价格、描述信息序列化成 JSON 字符串保存到 value 中。

```bash
HSETNX "goods:info" "WF-1000XM4" "{\"price\":1899,\"url\":\"https://ww.xxx.com/ughgg\",\"description\":\"真无线蓝牙降噪耳机\"}"
```

`HSETNX` 命令的作用是只有当`field` 不存在的时候才设置 value，否则啥也不干。

这么做的目的是一个商品有可能被多个用户添加到购物车，但是你没必要重复写数据。

**查询流程**

1. 先从集合 `shoppingCart:660`中查到商品的编码和价格。
2. 再根据商品编码去集合 `goods:info`中查找商品明细信息，查询不到则从数据库查询，并把查到的数据通过 `HSETNX` 命令写到集合中，以便下次查询可以从 Redis 获取，提高性能。

## 2.5 Sorted Sets（有序集合）

### 2.5.1 是什么

Sorted Sets 与 Sets 类似，是一种集合类型，集合中**不会出现重复的数据（member）**。区别在于 Sorted Sets 元素由两部分组成，分别是 member 和 score。member 会关联一个 double 类型的分数（score），sorted sets 默认会根据这个 score 对 member 进行从小到大的排序，如果 member 关联的分数 score 相同，则按照字符串的字典顺序排序。

![2-24][image-38]

图 2-24

常见的使用场景：

- 排行榜，比如维护大型在线游戏中根据分数排名的 Top 10 有序列表。
- 速率限流器，根据排序集合构建滑动窗口速率限制器。
- 延迟队列，score 存储过期时间，从小到大排序，最靠前的就是最先到期的数据。

### 2.5.2 修炼心法

Sorted Sets 底层有两种方式来存储数据。

- 在 7.0 版本之前是 ziplist，之后被 listpack 代替，使用条件是集合元素个数小于等于 `zset-max-listpack-entries` 配置值（默认 128），且 member 占用字节大小小于 `zset-max-listpack-value` 配置值（默认 64）时使用 listpack 存储，member 和 score 紧凑排列作为 listpack 的一个元素进行存储。
- 不满足上述条件，使用 skiplist + dict（散列表） 组合方式存储，数据会插入 skiplist 的同时也会向 dict（散列表）中插入数据 ，是一种用空间换时间的思路。散列表的 key 存放的是元素的 member，value 存储的是 member 关联的 score。

> MySQL：“也就是说 listpack 适用于元素个数不多且元素内容不大的场景。”

对，使用 listpack 存储的目的就是节省内存。

Sorted Sets 能支持高效的范围查询，正是因为采用了 skiplist 跳表，比如 `ZRANGE` 命令时**时间复杂度为 `O(log(n)) + m`， n 是 member 个数，m 是返回结果数。需要注意的是，你应该避免命令会返回大量结果。**

而使用 dict 的原因是实现 O(1) 时间复杂度查询单个元素。比如 `ZSCORE key member` 指令。

总结来说，Sorted Sets 在插入或者更新的时候，会同时往 skiplist 和 散列表中插入或者更新对应的数据。保证 skiplist 和散列表的数据一致。

> MySQL：“这个方式很巧妙呀，skiplist 用来根据 score 进行范范围查询或者单个查询，dict 散列表则用于实现 O(1) 时间复杂度查根据数据查询对应 score，满足高效范围查询和单元素查询。“

sorted sets 实现源码主要在以下两个文件中。

- 结构定义在 `server.h`。
- 功能实现在 `t_zset.c`。

先看 skiplist（跳表） + dict（散列表）数据结构如何存储数据。

#### skiplist + dict

> MySQL：“说说什么是跳表吧”

实质就是**一种可以进行二分查找的有序链表**。跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。

不仅能提高搜索性能，还可以提高插入和删除操作的性能。它在性能上和红黑树、AVL 树不相上下，但是跳表的原理和实现比红黑树简单。

回顾链表，它的痛点就是查询很慢，O(n) 时间复杂度，作为唯快不破的 Redis 是不能忍的。

![图2-25][image-39]

图 2-25

如果在有序链表的每相邻两个节点增加一个“跳跃”指向下下个节点的指针，那么查找的时间复杂度就可以降低为原来的一半，如下图所示。

![图 2-26][image-40]

图 2-26

这样 level 0 和 level 1 分别形成两个链表，level 1 层的链表节点个数只有 2 个（6、26）。

**跳表节点查找**

查找数据总是从最高层开始比较，如果节点保存的值比待查数据小，跳表就继续访问该层的下一个节点；

如果碰到比待查数据值大的节点时，那就跳到当前节点的下一层的链表继续查找。

比如现在想查找 17，查找的路径如下图红色指向的方向进行。

![][image-41]

图 2-27

- 从 level 1 开始，17 与 6 比较，值大于节点，继续与下一个节点比较。
- 与 26 比较，17 \< 26，回到原节点，跳到当前节点的 level 0 层链表，与下一个节点比较，找到目标 17。

skiplist 正是受这种多层链表的想法启发设计出来的。按照上面的生成链表方式，每次往上增加一层链表的节点个数是下面一层的一半，这样的查找过程就类似于一个二分查找，时间复杂度为 O(log n)。

但是，这种方式在插入数据的时候有很大的问题，每次新增一个节点，就会打乱相邻的两层链表节点个数 2:1 的关系，如果要维持这个关系，就需要对链表调整，事件复杂度是 O(n)。

为了避免这个问题，**它不要求上下相邻的两层链表节点个数有严格的比例关系，而是为每个节点随机出一个层数，这样插入节点只需要修改前后指针**。

如下图是一个有 4 层链表的 skiplist，假设我们要查找 26，下图给出了查找经历过的路径。

![][image-42]

图 2-28

对经典跳表有个直观的映像后，来看看 Redis 中 skiplist 的实现细节，Sorted Sets 数据结构定义如下。

```c
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```

`zset` 结构体中有两个变量，分别是散列表 dict 和跳表 zskiplist。dict 在前文已经讲过， 重点看 `zskiplist` 。

```c
typedef struct zskiplist {
    // 头、尾指针便于双向遍历
    struct zskiplistNode *header, *tail;
    // 当前跳表包含元素个数
    unsigned long length;
    // 表内节点的最大层级数
    int level;
} zskiplist;
```

- `zskiplistNode *header, *tail`，两个头、尾指针，用于实现双向遍历。
- `length`，链表包含的节点总数。需要注意的是，新创建的 `zskiplist` 会生成一个空的头指针，它不包含在 length 计数中。
- `level`，表示 `skiplist` 中，所有节点层数的最大值。

接着继续看 skiplist 中每个节点的定义 `zskiplistNode` 结构体。

```c
typedef struct zskiplistNode {
    sds ele;
    double score;

    struct zskiplistNode *backward;

    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];

} zskiplistNode;
```

- Sorted Set 既要保存元素，又要保存元素的权重。所以对应了 sds 类型的 ele 存储实际内容， double 类型 score 用于保存权重。
- `*backward`，后退指针，指向该节点的上一个节点，便于从尾节点实现倒序查找。注意，每个节点只有一个后向指针，只有 level 0 层链表是一个双向链表。
- `level[]`，是一个 `zskiplistLevel` 结构体类型的柔性数组。跳表是一个多层的有序链表，每一层的节点也是由指针链接起来的，所以数组中每个元素代表着 skiplist 的一层。
  - `*forward`，该层的前进指针。
  - `span`，跨度，用来记录节点在该层的 `*forward` 指针到指针指向的下一个节点之间跨越了 level0 层的节点数。span 用于计算元素排名(rank)，例如查找 ele = 肖菜鸡、score = 17 的排名，只需要把查找路径经过的节点的 span 相加即可，如下图的红色路径的 span 累加，`rank = (2 + 2) - 1 = 3`（减 1 是因为 rank 从 0 开始）。如果要计算从大到小的排名，只需要用 skiplist 长度减去查找路径上的 span 累加值，即 `4 - (2 + 2) = 0`。

下图展示了 Redis 中一个 skiplsit 的可能结构。

![][image-43]

图 2-29

#### listpack

> MySQL：“根据 `zset` 结构体定义可知，分别使用了 dict、zskiplist 两种数据结构，listpack 影子都见不着呀。“

这个问题问得好，使用 listpack 存储的细节在源码文件`t_zset.c` 中的`zaddGenericCommand`函数中体现，部分代码如下，内部会判断是否使用 listpack 来存储。·

```c
void zaddGenericCommand(client *c, int flags) {
    // 省略部分代码

    // key 不存在则创建 sorted set
    zobj = lookupKeyWrite(c->db,key);
    if (checkType(c,zobj,OBJ_ZSET)) goto cleanup;
    if (zobj == NULL) {
        if (xx) goto reply_to_client;
   			// 当 zset_max_listpack_entries == 0 或者
        // 元素字节大小大于 zset_max_listpack_value 配置
        // 则使用 skiplist + dict 存储，否则使用 listpack。
        if (server.zset_max_listpack_entries == 0 ||
            server.zset_max_listpack_value < sdslen(c->argv[scoreidx+1]->ptr))
        {
            zobj = createZsetObject();
        } else {
            zobj = createZsetListpackObject();
        }
        dbAdd(c->db,key,zobj);
    }
   // 省略部分代码
}
```

我们知道，listpack 是一块由多个数据项组成的连续内存。而 sorted set 每一项元素是由 member 和 score 两部分组成。

采用 listpack 存储插入一个（member、score）数据对的时候，每个 member/score 数据对紧凑排列存储。

listpack 最大的优势就是节省内存，查找元素的话只能按顺序查找，时间复杂度是 O(n)。正是如此，在少量数据的情况下，才能做到既能节省内存，又不会影响性能。每一步查找前进两个数据项，也就是跨越一个 member/score 数据对。

![][image-44]

图 2-30

### 2.5.3 出招实战：排行榜

很多地方都会用到排行榜功能，比如微博热榜、知乎热榜、电影排行榜、游戏战力排行等。

以游戏排行榜为例，我教你使用 Sorted Set 实现一个实时游戏高分排行榜。

**玩家的得分越高，排行越靠前，如果分数相同则先达到该分数的玩家排在前面**，游戏排行榜的提供的功能如下。

- 按照分数从大到小排名，查询前 N 位玩家信息。
- 新注册玩家，需要把新玩家信息添加到排行榜中。
- 能查看某个玩家的排名和分数。

Sorted Set 每个元素有两部分组成（member + score），可利用 score 进行排序，正好满足我们的场景。**用 score 保存玩家的游戏得分，member 保存玩家 ID**。

> 王架构：“分数相同，先达到该分数的排在前面，也就是说，游戏分数相同的情况下，时间戳越小，排名越靠前，咋实现？”

这个问题问得好，既然时间也会影响排名，那就把时间戳考虑到 score 中。

> 王架构：“有问题，分数越大，排名越靠前；而时间戳越小，排名越靠前。两个规则相反的，怎么结合在一起。”

好问题，这时候你可以指定一个非常大的时间作为基准时间，比如这个时间就是你当年信誓旦旦的对那个女孩说：“如果非要在我们的爱上加一个期限，我是希望……一万年”，也就是 2023 + 10000 年。

执行`时间排序值 =（基准时间 - 玩家达到分数时间）/ 基准时间`公式计算，得到的结果值一定小于 1，正好可作为 score 小数部分。越早达到，这个值就越大，满足排序。

最后**score = 玩家游戏分 + ((基准时间 - 玩家获得某分数时间) / 基准时间)**，就实现了**分数相同，先达到该分数的排在前面**的功能。

代码逻辑如下所示。

```java
private double calcScore(int playerScore, long playerScoreTime) {
  return playerScore + (BASE_TIME - playerScoreTime) * 1.0 / BASE_TIME;
}
```

- playerScore，玩家游戏分。
- playerScoreTime，玩家获得分数的时间秒数。
- BASE\_TIME，基准时间的时间秒数。

想要获取真正玩家游戏分数的时候，取整数位即可。接下来我来演示一下如何使用 zset 的指令实现排行榜。

假设 BASE\_TIME 为 12023 年 1 月 1 日 0 时 0 分 0 秒时间戳秒数 = 317242022400。

#### 更新排行榜

使用指令 `ZADD key score member [score member...]` 用于新增或者更新玩家排行榜。如下指令表示新增了 4 个玩家信息到排行榜。`leaderboard:339` 作为 key，表示区服 339 战力排行榜，玩家 2 和玩家 3 的战力都是 500 分，玩家 3 比玩家 2 先到达 500 战力。

```bash
redis> ZADD leaderboard:339 2500.994707057989 player:1
(integer) 1
redis> ZADD leaderboard:339 500.99470705798905 player:2
(integer) 1
redis> ZADD leaderboard:339 500.9947097814618 player:3
(integer) 1
redis> ZADD leaderboard:339 987770.994707058 player:4
(integer) 1
```

假设某天玩家 4 的女朋友不在家，他就天天玩游戏，战力提升到 1987770。执行如下指令，player:4 的 score 机会更新为 1987770.994707055。

```
ZADD leaderboard:339 1987770.994707055 player:4
```

#### 获取 Top 3 玩家排行信息

`ZRANGE` 命令可以按照排名、score、字典排序进行范围查询。语法使用规则

```bash
ZRANGE key start stop [BYSCORE | BYLEX] [REV] [LIMIT offset count] [WITHSCORES]
```

默认排序是按照 score 由低到高，分数相同则根据 member 字典排序。

- `REV`，可选参数，按照 score 由高到低逆序排序。
- `LIMIT offset count` 可选参数，类似于 MySQL 的使用，需要注意的是， count 为负数则返回所有符合数据。
- `WITHSCORES` 可选参数，返回 score 和 member，返回的格式是 `member 1,score 1,…memberN,scoreN`。

你可以使用 `REV` 来实现逆序，`WITHSCORES`返回 `member` 和 `score`。如下指令的一是是从 key 为 `leaderboard:339` 的 Sorted Set 中按照 score 逆序排序获取 3 个元素。

```bash
> ZRANGE leaderboard:339 0 2 REV WITHSCORES
player:4
1987770.9947070549
player:1
2500.9947070579892
player:3
500.99470978146178
```

#### 获取指定玩家排名

我提供了 `ZREVRANK`指令，用于返回指定 member 的排名，需要注意的是，排名从 0 开始。如下指令查找 player:4 的排名，0 表示第一。

```bash
> ZREVRANK leaderboard:339 player:4
0
```

## 2.6 Streams （流）

我在 2.1.2 章节说过使用 List 实现消息队列有很多局限性。

- 没有 ACK 机制。
- 没有类似 Kafka 的 ConsumerGroup 消费组概念。
- 消息堆积。
- List 是线性结构，查询指定数据需要遍历整个列表。

### 2.6.1 是什么

Stream 是 Redis 5.0 版本专门为消息队列设计的数据类型，**借鉴了 Kafka 的 Consume Group 设计思路，提供了消费组概念**。

**同时提供了消息的持久化和主从复制机制，客户端可以访问任何时刻的数据，并且能记住每一个客户端的访问位置，从而保证消息不丢失。**

以下几个是 Stream 类型的主要特性。

- 使用 **Radix Tree 和 listpack** 结构来存储消息。
- 消息 ID 序列化生成。
- 借鉴 Kafka Consume Group 的概念，多个消费者划分到不同的 Consume Group 中，消费同一个 Streams，同一个 Consume Group 的多个消费者可以一起并行但不重复消费，提升消费能力。
- 支持多播（多对多），阻塞和非阻塞读取。
- ACK 确认机制，保证了消息至少被消费一次。
- 可设置消息保存上限阈值，我会把历史消息丢弃，防止内存占用过大。

需要注意的是，Redis Stream 是一种超轻量级的 MQ，并没有完全实现消息队列的所有设计要点，所以它的使用场景需要考虑业务的数据量和对性能、可靠性的需求。

**适合系统消息量不大，容忍数据丢失，使用 Redis Stream 作为消息队列就能享受高性能快速读写消息的优势。**

### 2.6.2 修炼心法

每个 Stream 都有一个唯一的名称，作为 Stream 在 Redis 的 key，在首次使用 `xadd` 指令添加消息的时候会自动创建。

**可以看到 Stream 在一个 Redix Tree 树上，树上存储的是消息 ID，每个消息 ID 对应的消息通过一个指针指向 listpack。**

**Stream 流就像是一个仅追加内容的消息链表，把消息一个个串起来，每个消息都有一个唯一的 ID 和消息内容，消息内容则由多个 field/value 键值对组成**。底层使用 Radix Tree 和 listpack 数据结构存储数据。

为了便于理解，我画了一张图，并对 Radix Tree 的存储数据做了下变形，使用列表来体现 Stream 中消息的逻辑有序性。

![图 2-31][image-45]

图 2-31

这张图涉及很多概念，但是你不要慌。我一步步拆开说，最后你再回头看就懂了。

先带你屡下全局思路。

- `Consumer Group`：消费组，每个消费组可以有一个或者多个消费者，消费者之间是竞争关系。不同消费组的消费者之间无任何关系。
- `*pel`，全称是 Pending Entries List，记录了当前被客户端读取但是还没有 ack（Acknowledge character 确认字符）的消息。如果客户端没有 ack，这个变量的消息 ID 会越来越多。这是一个核心数据结构，用来确保客户端至少消费消息一次。

#### Stream 结构

Streams 结构的源码定义在 `stream.h` 源码中的 `stream` 结构体中。

```c
typedef struct stream {
    rax *rax;
    uint64_t length;
    streamID last_id;
    streamID first_id;
    streamID max_deleted_entry_id;
    uint64_t entries_added;
    rax *cgroups;
} stream;

typedef struct streamID {
    uint64_t ms;
    uint64_t seq;
} streamID;
```

- `*rax`，是一个 `rax` 的指针，指向一个 Radix Tree，key 存储消息 ID，v**alue 实际上指向一个 listpack 数据结构，存储了多条消息**，每条消息的 ID 都大于等于 这个 key 的消息 ID。
- `length`，该 Stream 的消息条数。
- `streamID`结构体，消息 ID 抽象，一共占 128 位，内部维护了毫秒时间戳（字段 ms）；一个毫秒内的自增序号（字段 seq），**用于区分同一毫秒内插入多条消息**。
- `last_id`，当前 Stream 最后一条消息的 ID。
- `first_id`，当前 Stream 第一条消息的 ID。
- `max_deleted_entry_id`，当前 Stream 被删除的最大的消息 ID。
- `entries_added`，总共有多少条消息添加到 Stream 中，`entries_added = 已删除消息条数 + 未删除消息条数`。
- `*cgroups`，rax 指针，也指向一个 Radix Tree ，**记录当前 Stream 的所有 Consume Group**，每个 Consume Group 的名称都是唯一标识，作为 Radix Tree 的 key，Consumer Group 实例作为 value。

#### Consumer Group

Consumer Group 由 `streamCG` 结构体定义，每个 Stream 可以有多个 Consumer Group，一个消费组可以有多个消费者同时对组内消息进行消费。

```c
/* Consumer group. */
typedef struct streamCG {
    streamID last_id;
    long long entries_read;
    rax *pel;
    rax *consumers;
} streamCG;
```

- `last_id`，表示该消费组的消费者已经读取但还未 ACK 的最后一条消息 ID。
- `*pel`，是 `pending entries list` 简写，指向一个 Radix Tree 的指针，**保存着 Consumer group 中所有消费者读取但还未 ACK 确认的消息**，就是这玩意实现了 ACK 机制。该树的 key 是消息 ID，value 关联一个 `streamNACK` 实例。
- `*consumers`， Radix Tree 指针，表示消费组中的所有消费者，key 是消费者名称，value 指向一个 `streamConsumer` 实例。

##### streamNACK

`streamCG -> *pel` 对应的 value 是一个 streamNACK 实例，用于抽象消费者已经读取，但是未 ACK 的消息 ID 相关信息。

```c++
/* Pending (yet not acknowledged) message in a consumer group. */
typedef struct streamNACK {
    mstime_t delivery_time;
    uint64_t delivery_count;
    streamConsumer *consumer;
} streamNACK;
```

- `delivery_time`，该消息最后一次推送给 Consumer 的时间戳。
- `delivery_count`，消息被推送次数。
- `*consumer`，消息推送的 Consumer 客户端。

##### streamConsumer

Consumer Group 中对 Consumer 的抽象。

```c++
/* A specific consumer in a consumer group.  */
typedef struct streamConsumer {
    mstime_t seen_time;
    sds name;
    rax *pel;
} streamConsumer;
```

- `seen_time`，消费者最近一次被激活的时间戳。
- `name`，消费者名称。
- `*pel`， Radix Tree 指针，对于同一个消息而言，\``streamCG -> pel`与`streamConsumer -> pel`的`streamNACK` 实例是同一个。

最后来一张图，便于你理解。

![图 2-32][image-46]

图 2-32

> 肖材积：“Redis 你好，Stream 如何结合 Radix Tree 和 listpack 结构来存储消息？为什么不使用散列表来存储，消息 ID 作为散列表的 key，散列表的 value 存储消息键值对内容。’”

在回答之前，先插入几条消息到 Stream，让你对 Stream 消息的存储格式有个大体认知。

该命令的语法如下。

```bash
XADD key id field value [field value ...]
```

Stream 中的**每个消息可以包含不同数量的多个键值对**，写入消息成功后，我会把消息的 ID 返回给客户端。

执行如下指令把用户购买书籍的下单消息存放到 `hotlist:books`队列，消息内容主要由 payerID、amount 和 orderID。

```bash
> XADD hotlist:books * payerID 1 amount 69.00 orderID 9
1679218539571-0
> XADD hotlist:books * payerID 1 amount 36.00 orderID 15
1679218572182-0
> XADD hotlist:books * payerID 2 amount 99.00 orderID 88
1679218588426-0
> XADD hotlist:books * payerID 3 amount 68.00 orderID 80
1679218604492-0
```

`hotlist:books` 是 Stream 的名称，后面的 “\*” 表示让 Redis 为插入的消息自动生成一个唯一 ID，你也可以自定义。

消息 ID 由两部分组成。

- 当前毫秒内的时间戳；
- 顺序编号。从 0 为起始值，用于区分同一时间内产生的多个命令。

> 肖材积：“如何理解 Stream 是一种只执行追加操作（append only）的数据结构？”

通过将元素 ID 与时间进行关联，并强制要求新元素的 ID 必须大于旧元素的 ID, **Redis 从逻辑上将 Stream 变成了一种只执行追加操作（append only）的数据结构**。

用户可以确信，新的消息和事件只会出现在已有消息和事件之后，就像现实世界里新事件总是发生在已有事件之后一样，一切都是有序进行的。

> 肖材积：“插入的消息 ID 大部分相同，比如这四条消息的 ID 都是 1679218 前缀。另外，每条消息键值对的键通常都是一样的，比如这四条消息的键都是 payerID、amount 和 orderID。使用散列表存储的话会很多冗余数据，你这么抠门，所以不使用散列表对不对？”

没毛病，小老弟很聪明。为了节省内存，我使用了 Radix Tree 和 listpack。**Radix Tree 的 key 存储消息 ID，value 使用 listpack 数据结构存储多个消息， listapck 中的消息 ID 都大于等于 key 存储的消息 ID。**

我在前面已经讲过 listpack，这是一个紧凑型列表，非常节省内存。而 Radix Tree 数据结构的最大特点是适合保存具有相同前缀的数据，从而达到节省内存。

到底 Radix Tree 是怎样的数据结构，继续往下看。

#### Radix Tree

Radix Tree，也被称为 Radix Trie，或者 Compact Prefix Tree)，用于高效地存储和查找字符串集合。**它将字符串按照前缀拆分成一个个字符，并将每个字符作为一个节点存储在树中**。

当插入一个键值对时，Redis 会将键按照字符拆分成一个个字符，并根据字符在 Radix tree 中的位置找到合适的节点，如果该节点不存在，则创建新节点并添加到 Radix tree 中。

当所有字符都添加完毕后，将值对象指针保存到最后一个节点中。当查询一个键时，Redis 按照字符顺序遍历 Radix tree，如果发现某个字符不存在于树中，则键不存在；否则，如果最后一个节点表示一个完整的键，则返回对应的值对象。

如下图展示一个简单的前缀树，将根节点到叶子节点的路径对应字符拼接起来，就得到了两个 key（“他说碉堡了”、“他说碉炸了”）。

![图 2-33][image-47]

图 2-33

你应该发现了，这两个 key 拥有公共前缀（他说碉），前缀树实现了共享使用，这样就可以避免相同字符串重复存储。如果采用散列表的保存方式，那个 key 的相同前缀就会被多次存储，导致内存浪费。

**Radix Tree 改进**

每个节点只保存一个字符，一是会浪费内存空间，二是在进行查询时，还需要逐一匹配每个节点表示的字符，对查询性能也会造成影响。

所以，Redis 并没有直接使用标准前缀树，而是做了一次变种——Compact Prefix Tree（压缩前缀树）。通俗来说，**当多个 key 具有相同的前缀时，那就将相同前缀的字符串合并在一个共享节点中，从而减少存储空间**。

如下几个 key（test、toaster、toasting、slow、slowly）在 Radix Tree 上的布局。

![][image-48]

图 2-34

由于 Compact Prefix Tree 可以共享相同前缀的节点，所以在存储一组具有相同前缀的键时，Redis 的 Radix tree 比其他数据结构（如哈希表）具有更低的空间消耗和更快的查询速度。

Radix Tree 节点的数据结构由 `rax.h`文件中的 `raxNode` 定义。

```c
typedef struct raxNode {
    uint32_t iskey:1;
    uint32_t isnull:1;
    uint32_t iscompr:1;
    uint32_t size:29;
    unsigned char data[];
} raxNode;
```

- iskey：从 Radix Tree 根节点到当前节点组成的字符串是否是一个完整的 key。是的话 iskey 的值为 1。
- isnull：当前节点是否为空节点，如果当前节点是空节点的话，就不需要为该节点分配指向 value 的指针内存。
- iscompr，是否为压缩节点。
- size，当前节点的大小，具体指会根据节点类型而改变。如果是压缩节点，该值表示压缩数据的长度；如果是非压缩节点，该值表示节点的子节点个数。
- data[]，实际存储的数据，根据节点类型不同而有所不同。
  - 压缩节点，data 数据包括子节点对应的字符、指向子节点的指针，节点为最终 key 对应的 value 指针。
  - 压缩节点，data 数据包含子节点对应的合并字符串、指向子节点的指针，以及节点为最终 key 的 value 指针。
  - **value 指针指向一个 listpack 实例，里面保存了消息实际内容**

Radix Tree 最大的特点就是适合保存具有相同前缀的数据，实现节省内存的目标，以及支持范围查找。而这个就是 Stream 采用 Radix Tree 作为底层数据结构的原因。

### 2.6.3 出招实战：消息队列

废话少说，实战。

> 周五快下班，靠卷团队获得业绩的李易卷领导为了达到 KPI，获得更高的股票和年终奖，于是就对开发负责人张无剑说：“明天我建议还是赶一赶进度，隔壁老王明天周六也过来加班，要不下周风险起来了比较麻烦，你动员下团队冲一冲！”。

张无剑就想着过度加班，长时间处于高度紧张和压力的状态下，焦虑、烦躁和易怒等情绪变化，肝气不舒、脾胃失调以及肾精亏损。

为了缓解压力，调理身体。于是打开美团 APP， 点击下单，请组内成员一起大保健，英文名叫 Double Joy，意为双倍快乐，爽一把。

**异步并行处理**

下单过程中，除了生成订单核心业务流程以外，还涉及赠送活动积分、优惠券发放、发送下单成功通知等一系列业务功能。假设每个业务节点耗时 100 ms，串行的方式调用则需要 400 ms，并行处理只需要 200 ms。消息队列在里面可以起到异步并行处理，从而减少请求响应时间，提高系统吞吐量。

![2-35][image-49]

图 2-35

**应用解耦**

此外，通过消息队列，还实现了应用解耦，使得每个应用系统不必受其他系统影响。

比如用户下单后，订单系统需要通知积分系统，订单系统将消息写入消息队列，积分系统订阅消息进行积分操作即可，实现系统之间解耦。

#### 普通消费队列

##### XADD 插入消息

`XADD` 命令主打的就是有序将消息插入到到末尾，并自动生成全局唯一 ID。

张无剑的下单请求到了订单中心生成订单后，可通过 XADD 将订单创建完成的消息发送到消息队列，让其他服务监听消息异步执行。

Streams 的每个元素由键值对的形式组成，不同元素可以包含不同数量的键值对，`XADD` 的语法如下。

```bash
XADD streamName id field value [field value ...]
```

比如订单系统执行如下命令，就是向名称为 `order:doubleJoy`的消息队列插入一条消息，消息的内容表示张无剑的订单 ID 是 1，technician （技师）编号是 68，消费金额是 598。消息的内容由三个键值对组成，分别是 `orderID -> 1、technician -> 68 amount -> 598`。（ps：重复执行三次指令向队尾插入消息， orderID 依次 + 1 递增，以便后边举例验证，命令不再重复。）

```bash
XADD order:doubleJoy * orderID 1 technician 68 amount 598
"1685782062437-0"
```

队列名称后面的 `*` ，表示让 Redis 为插入的消息自动生成唯一 ID。当然，你也可以不用 `*`，在名称后边设定一个自定义的 ID，只要保证这个 ID 全局唯一即可。

消息 ID 由两部分组成。

- 时间戳，消息插入时，以毫秒为单位计算的当前服务器时间；
- 顺序编号，从 0 为起始值，用于区分同一时间内产生的多个命令。

将元素 ID 与时间进行关联，并强制要求新元素的 ID 必须大于旧元素的 ID, Redis 从逻辑上将流变成了一种只执行追加操作（append only）的数据结构。

这种特性对于使用流实现消息队列和事件系统的用户来说是非常重要的。

新的消息和事件只会出现在已有消息和事件之后，就像现实世界里新事件总是发生在已有事件之后一样，一切都是有序进行的。

使用 `XLEM` 指令可以查看当前 Stream 由多少条消息。

```bash
XLEN order:doubleJoy
(integer) 1
```

##### XREAD 读取消息

> 张无剑：“积分系统如何读取队列的消息进行消费呢？”

如下指令的含义是：客户端用阻塞等待读取的方式从队头读取 1 个消息。

```bash
XREAD COUNT 1 BLOCK 0 STREAMS order:doubleJoy 0-0
1) 1) "order:doubleJoy"
   2) 1) 1) "1685785480628-0"
         2) 1) "orderID"
            2) "1"
            3) "technician"
            4) "68"
            5) "amount"
            6) "598"
```

该指令可以同时对多个流进行读取。

```bash
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]
```

- COUNT：表示从每个流中最多读取的元素个数。
- BLOCK：阻塞读取，当消息队列没有新消息插入的时候，则阻塞等待， 0 表示无限等待，单位是毫秒。
- ID：消息 ID，**在读取消息的时候可以指定 ID，并从这个 ID 的下一条消息开始读取。**
  - 0-0 则表示从第一个元素开始读取；
  - “\$” 符号表示读取最新插入的消息。

**如果想使用 XREAD 进行顺序消费，每次读取后要记住返回的消息 ID，下次调用 XREAD 就将上一次返回的消息 ID 作为参数传递到下一次调用就可以继续消费后续的消息了。**

比如执行下面指令，从 ID 为 1685785480628-0 的消息开始，读取下一条消息。

```bash
XREAD COUNT 1 BLOCK 0 STREAMS order:doubleJoy 1685785480628-0
1) 1) "order:doubleJoy"
   2) 1) 1) "1685785502168-0"
         2) 1) "orderID"
            2) "2"
            3) "technician"
            4) "79"
            5) "amount"
            6) "598"
```

> “张无剑”：“客户端阻塞等待的方式读取最新插入的消息怎么实现？”。

如下指令，命令最后的“\$”符号表示从尾部读取最新插入的消息，`BLOCK 0` 表示阻塞等待。需要注意的是，**如果没有新消息插入，会一直阻塞等待**。

```bash
XREAD BLOCK 0 STREAMS order:doubleJoy $
```

毫无疑问，这里不会返回任何消息，除非现在有新消息插入。

> 张无剑：“这么容易就实现消息队列了么？说好的 ACK 机制呢？”

我们可以在不定义 Consumer Group 的情况下进行通过 `XREAD` 单独消费，将 Streams 当成普通队列使用。

这里只是开胃菜，通过 `XREAD` 读取的数据其实并没有被删除，当重新执行 `XREAD BLOCK 0 STREAMS order:doubleJoy 0-0` 指令的时候又会重新读取所有数据。

```bash
XREAD BLOCK 0 STREAMS order:doubleJoy 0-0
1) 1) "order:doubleJoy"
   2) 1) 1) "1685785480628-0"
         2) 1) "orderID"
            2) "1"
            3) "technician"
            4) "68"
            5) "amount"
            6) "598"
      2) 1) "1685785502168-0"
         2) 1) "orderID"
            2) "2"
            3) "technician"
            4) "79"
            5) "amount"
            6) "598"
      3) 1) "1685785517633-0"
         2) 1) "orderID"
            2) "3"
            3) "technician"
            4) "88"
            5) "amount"
            6) "598"
```

**一个 Stream 可以有多个客户端（消费者）等待数据。默认情况下，每个新消息都将发送到 Stream 中等待数据的每个消费者。**

**所有的消息都无限期地存储在 Stream 中（除非用户明确要求删除消息），不同的消费者会通过收到的最后一条消息的 ID 来确定下一条消息。**

接下来，我们来一个真正的消息队列。

#### Consumer Group

Redis Stream 的 ConsumerGroup（消费者组）允许用户将一个流从逻辑上划分为多个不同的流，并让 ConsumerGroup 的消费者去处理。

**支持多播的可持久化的消息队列**， 借鉴了 Kafka 的设计。Stream 高可用是建立主从复制基础上的，和其他数据结构一样，Stream 也会被异步复制到副本并持久化到 AOF 和 RDB 文件中。

也就是说在 Sentinel 和 Cluster 集群环境下 Stream 是可以支持高可用的。

![2-36][image-50]

图 2-36

- **每个消费组的状态是独立的，互不影响，同一份的 Stream 消息会被所有的消费组消费；**
- **一个消费组可以有多个消费者组成，消费者之间是竞争关系，任意一个消费者读取了消息都会使 last\_deliverd\_id 往前移动；**
- **每个消费者有一个 pel 变量，用于记录当前消费者读取了但是还没 ack 的消息。它用来保证消息至少被客户端消费了一次。**

消费者组的相关命令：

1. `XGROUP` 用于创建、销毁和管理消费者组。
2. `XREADGROUP` 用于通过消费者组从 Stream 中读取消息。
3. `XACK` 允许消费者将待处理消息标记为已正确处理，可以移除。
4. `XPENDING` ，显示已读取，但未 ACK 的消息的相关信息。
5. `XINFO` ，查看流和消费者组的相关信息；

##### XGROUP CREATE 创建消费者组

`XGROUP CREATE`指令语法如下，`<>` 标记是必备参数，`[]` 标记的参数是可选的。

```bash
XGROUP CREATE $streamName $groupName <id | $> [MKSTREAM][ENTRIESREAD]
```

- `streamName`，指定队列的名称；
- `groupName`，指定消费组的名称；
- `<id | $>`，指定消费组在 Stream 的 ID，它决定了消费者组从哪个 ID 之后开始读取消息，`0-0` 从队头一条开始读取， `$` 表示从现在开始，从队尾读取新插入的消息，你也可以自定义 ID。
- `MKSTREAM`，默认情况下，`XGROUP CREATE` 命令在 Stream 不存在时返回错误。使用可选`MKSTREAM`子命令作为最后一个参数来自动创建 Stream。

如下指令，为消息队列名为 `order:doubleJoy` 创建 `pointsGroup`和 `couponGroup` 两个消费组分别代表“积分服务消费组”和“优惠券服务消费组”。

```sh
XGROUP CREATE order:doubleJoy pointsGroup 0-0 MKSTREAM
XGROUP CREATE order:doubleJoy couponGroup 0-0 MKSTREAM
```

##### XREADGROUP 读取消息

我是 Redis，作为贴心哥，为开发者提供了 `XREADGROUP`指令来实现消费组的组内消费者消费消息。

比如，积分服务的 `pointsGroup` 消费者组的消费者 `consumer1` 从名称为 `order:doubleJoy` 的 Stream 的队头阻塞读取一条消息的指令如下所示。

```bash
XREADGROUP GROUP pointsGroup consumer1 COUNT 1 BLOCK 0 STREAMS order:doubleJoy >
1) 1) "order:doubleJoy"
   2) 1) 1) "1685785480628-0"
         2) 1) "orderID"
            2) "1"
            3) "technician"
            4) "68"
            5) "amount"
            6) "598"
```

语法如下。

```bash
XREADGROUP GROUP $groupName $consumerName [COUNT count] [BLOCK milliseconds]
  [NOACK] STREAMS streamName [streamName ...] id [id ...]
```

该命令与 `XREAD` 大同小异，区别在于新增 `GROUP groupName consumerName` 选项。这两个参数分别用于指定 Stream 的消费者组以及该消费组中负责处理消息的消费者。

- groupName，消费组名称。
- consumerName， 消费者组的消费者名称。

- `NOACK`，如果你不需要可靠性，可以接受偶尔的消息丢失的情况，`NOACK` 子命令不会将消息添加到 PEL，相当于读取消息的时候就执行 ACK。
- `BLOCK`，阻塞读取，单位是毫秒。为 0 表示无限阻塞等待。

使用 `XREADGROUP` 时，要在 `STREAMS` 选项中指定 ID，有两种设置方式。

- `>` ，通常使用这个设置，消费者只接收自上次读取后产生的新消息，其实就是 从 Consumer Group 的 last\_id 开始一个个读取到的消息，都是未分配给其他 Consumer 的消息 。也就是**说只获取比上次读取的消息 ID 更大的消息。**
- 0 或者其他有效 ID，仅返回所有或 ID 大于指定 ID 的**未 ACK 的历史消息，不包含新消息**。

需要注意的是**，`XREADGROUP`实际是一个写命令，看起来是从流中读取数据，但作为读取的副作用，会修改消费者组的 last\_delivered\_id ，所以它只能在 master 实例上调用。**

> 张无剑：“当消息传递给消费者组的消费者时会执行哪些步骤？”

**Streams 内部有一个队列 PEL 保存每个消费者读取但是还没有执行 ACK 的消息**。

1. 如果该消息从未被任何消费者读取过，消费者会创建一个 PEL（Pending Entries List），并把 ID 保存在 PEL 中标记为待处理。
2. 消费者接收消息，处理业务逻辑。
3. 处理完成消息后，消费者可以选择执行 `XACK` 确认或者拒绝该消息。
   1. 如果消费者执行 `XACK`确认消息，则表示处理成功，从 PEL 中移除该 ID。
   2. 如果消费者拒绝消息，表示处理失败或者错误。消息依然保存在 `PEL`中，可以由同一或其他消费者重新处理。

**如果消息队列中的消息被消费者组的一个消费者消费了，这条消息就不会再被这个消费者组的其他消费者读取到。**

比如 `consumer2` 执行读取操作，读取到的是 orderID = 2 的消息，因为 `consumer1` 已经把 orderID = 1 的消息消费过了。

```bash
XREADGROUP GROUP pointsGroup consumer2 COUNT 1 BLOCK 0 STREAMS order:doubleJoy >
1) 1) "order:doubleJoy"
   2) 1) 1) "1685785502168-0"
         2) 1) "orderID"
            2) "2"
            3) "technician"
            4) "79"
            5) "amount"
            6) "598"
```

**Consumer Group 的一个作用就是可以让组内的多个消费者分担读取消息，从而实现负载均衡。**比如一个消费组有三个消费者 C1、C2、C3 和一个包含消息 1、2、3、4、5、6、7 的流。

![][image-51]

图 2-37

##### XACK 确认消息

当消费者接收消息的时候，如果消息需要 ACK 确认，Streams 会为每条消息创建对应的 `streamNACK`实例记录到 Consumer Group 和 Consumer 的 PEL（Pending Entries List）队列中。

消费者消费成功，需要使用 `XACK` 命令对消息进行确认才会从 PEL 队列清除。如果 `XREADGROUP` 命令携带 `NOACK`子命令，则消息无需确认，也就意味着不会进入 PEL 队列。

**一旦消费者成功处理了一条消息，就应该调用 XACK，这样这条消息就不会被再次读取，同时这条消息的 PEL 记录也被清除，从 Redis 服务器释放内存。**

如下指令，表示将名称为 `order:doubleJoy`的 Stream 流的 `pointsGroup`消费者组的消息 ID 为 `1685785480628-0`进行确认。

```bash
XACK order:doubleJoy pointsGroup 1685785480628-0
```

`XACK` 语法如下。

```bash
XACK $streamName $groupName id [id ...]
```

##### XPENDING 查看已读未确认消息

> 张无剑：“消费过程中，消费者 A 读取了消息，还没执行业务逻辑就崩溃了，如何实现消息至少能消费一次？”

问得好，除了使用 `XREADGROUP GROUP pointsGroup consumer2 COUNT 1 BLOCK 0 STREAMS order:doubleJoy >` 正常读取新消息，你还可以再执行一条新指令：指定实际的 ID 值或者 `0`来替换 `>` 这个参数，意思是让当前消费者读取分配给自己但是还没执行 ACK 的历史消息，保证了 At Least Once 的语义。

> 张无剑：“然而，现实中可能消费者永久失败，无法恢复。这个消费者对应的 PEL 未执行 ACK 的消息该如何处理？”。

为了保证消费者在消费的时候发生故障或者宕机重启后，未 ACK 的消息依然可以被其他消费者消费。

Streams 提供了 `XPENDING` 指令，这是一个专用于查询 Consumer Group 中未执行 ACK 的消息相关信息。比如查看 `order:doubleJoy` 队列中，消费者组`pointsGroup`每个消费者未 ACK 的消息信息。

```bash
XPENDING order:doubleJoy pointsGroup
1) (integer) 2
2) "1685785480628-0"
3) "1685785502168-0"
4) 1) 1) "consumer1"
      2) "1"
   2) 1) "consumer2"
      2) "1"
```

- `1)`已读取未确认消息条数。
- `2) ~ 3)`消费者组 `pointsGroup` 中所有消费者已读取的消息最小和最大 ID。
- `4)`当前消费组的消费者信息，可以看到一共有两个消费者（`consumer1`、`consumer2`）。每个消费者已读取但是未 ACK 的消息条数。

> 张无剑：“上边的信息点笼统，我想知道更多细节，怎么办？”

你的问题真多，`XPENDING`指令可以提供更多的的参数来获取更多的信息，完整的指令语法如下。

```bash
XPENDING <key> <groupname> [IDLE <min-idle-time>] <start-id> <end-id> <count> [<consumer-name>]]
```

你这么聪明，有的参数一看就知道是啥意思了，我就不啰嗦了。重点介绍几个特别的。

- `[IDLE <min-idle-time>]`：可选参数，可以对 `PEL` 队列进行筛选，只返回指定时间内处于空闲状态的消费者组的待处理消息。`<min-idle-time>` 是一个整数，单位为毫秒。比如你想要获取在最近 5 秒内没有接收新消息的消费者组 `mygroup` 的待处理消息，可以这样使用 `XPENDING` 命令 `XPENDING mystream mygroup IDLE 5000`。时间越长，说明这个消费者组一直在摸鱼。
- `<start-id>`：指定的起始消息 ID。命令将返回在此 ID 之后但在 `<end-id>` 之前的消息，可以用 `-` 表示从最早一条消息开始获取消息。
- `<end-id>`：指定的结束消息 ID。命令将返回在 `<start-id>` 之后但在此 ID 之前的消息，设置成 `+` 等同指定了当前 Stream 流中当前最新消息的 ID 作为 `end-id`。
- `<count>`：表示要返回的消息数量。
- `[<consumer-name>]`：可选参数，用于指定只返回属于特定消费者的待处理消息。

这么多参数，你想要的都满足了。

```bash
XPENDING order:doubleJoy pointsGroup IDLE 5000 - + 10
1) 1) "1685785480628-0"
   2) "consumer1"
   3) (integer) 1113535381
   4) (integer) 1
2) 1) "1685785502168-0"
   2) "consumer2"
   3) (integer) 1112153314
   4) (integer) 1
127.0.0.1:6379>
```

每个待处理消息响应体对应一个子数组，每个子数组包含以下信息。

1. 消息唯一 ID。
2. 消费者名称，获取该消息但是还未执行 ACK 的消费者名称，我把它称作为消息的当前所有者。
3. 消费者的空闲时间，通俗的说就是“上班摸鱼时间”，以毫秒为时间单位，从上次消息传递给该消费者到现在经过的毫秒数。
4. 该消息已传递点给消费者的次数。

第二个待处理消息的信息与第一个类似，以此类推。如果你想查看 `consumer1` 消费者的信息，在末尾新增一个参数表示消费者即可。

```bash
XPENDING order:doubleJoy pointsGroup IDLE 5000 - + 10 consumer1
1) 1) "1685785480628-0"
   2) "consumer1"
   3) (integer) 1186071318
   4) (integer) 1
```

> 张无剑：“时钟回拨会导致消息 ID 重复么？”

根据上文，我们已经知道消息 ID 由两部分组成，`时间戳-序号`。时间戳是毫秒级单位，序号是在这个毫秒内的消息序号。

**每个 Stream 类型数据都维护了一个 `latest_generated_id`属性，记录最后一个消息 ID。如果发现时间戳倒退（小于 latest\_generated\_id 所记录的 ID），则采用时间戳不变，序号递增的方式来生成新消息 ID，从而保证 ID 单调递增。**

## 2.7 Geospatial （地理空间）

Redis 老兄，产品经理跟我说，他有一个 idea，想为广大少男少女提供一个连接彼此的机会。

所谓花有重开日，人无再少年，为了让处于这美好年龄的少男少女，能在以每一个十二时辰里邂逅那个 ta”。想开发一款 APP，用户登录登录后，基于地理位置发现附近的那个 Ta 链接彼此。

我该如何实现“附近的人”？我也希望通过这个 APP 邂逅女神……

记忆中，一个下班的夜晚，她从人群中轻盈的移动着，那高挑苗条的身材像漂浮在空间中的一个飘逸的音符。她的眼睛充满清澈的阳光和活力，她的双眸中印着银河系的星光。

### 2.7.1 是什么？

在邂逅女神之前，先了解下什么是面向 LBS 应用。经纬度是经度与纬度的合称组成一个**坐标系统**。又称为地理坐标系统，它是一种利用**三度空间的球面**来定义地球上的空间的球面坐标系统，能够标示地球上的**任何一个位置**（小数点后 7 位，精度可以到 1 厘米）。

经度的范围在 (-180, 180]，纬度的范围 在(-90, 90]，纬度正负以赤道为界，北正南负，经度正负以本初子午线 (英国格林尼治天文台) 为界，东正西负。

`附近的人` 也就是常说的 `LBS` (Location Based Services，基于位置服务)，它围绕用户当前地理位置数据而展开的服务，为用户提供精准的邂逅服务。

`附近的人`核心思想如下。

1. 以 “我” 为中心，搜索附近的 Ta。
2. 以 “我” 当前的地理位置为准，计算出别人和 “我” 之间的距离。
3. 按 “我” 与别人距离的远近排序，筛选出离我最近的用户。

#### MySQL 实现

以登录用户为中心，给定一个 1000 米作为半径画圆，那么圆形区域内的用户就是我们想要邂逅的“附近的人“。

> Redis 老哥，我想到可以把经纬度存储到 MySQL 中。

```sql
CREATE TABLE `nearby_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL COMMENT '名称',
  `longitude` double DEFAULT NULL COMMENT '经度',
  `latitude` double DEFAULT NULL COMMENT '纬度',
  `create_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

> 总不能把数据库所有的“女神”的经纬度数据查询出来与自己的经纬度数据计算两者之间的距离，再根据距离排序，这个时间复杂度也太高，计算量太大了。

你可以分别把男女坐标数据存放到不同的表，来减少表数据。并且通过一个矩形区域来过滤有限“女神”坐标记录，这时候再对矩形区域内的数据进行距离计算再排序，这样计算量明显降低。

> 如何划分这个矩形区域呢？

在圆形外套上一个正方形，矩形的区域可根据用户经、纬度的最大、最小值（经、纬度 + 距离 R），作为查询语句的筛选条件过滤数据，就很容易将正方形内的「女神」信息搜索出来。

为了满足高性能的矩形区域算法，数据表需要在经纬度坐标加上复合索引 `(longitude, latitude)`，这样可以最大优化查询性能。

![][image-52]

图 2-38

> 多出来的阴影部分区域咋办？这些不是目标数据。

多出来的这部分区域内的用户，到圆点的距离一定比圆的半径 R 要大，那么我们就计算用户中心点与正方形内所有用户数据的距离，**筛选出所有距离小于等于半径的用户**，即符合要求的`附近的人`。

再说了，你不是想邂逅更多的女神么，不需要那么精确。

**实战**

根据经纬度和距离来计算出外接矩形、根据两个用户的经纬度计算两点之间的距离使用了一个第三方类库。

```xml
<dependency>
     <groupId>com.spatial4j</groupId>
     <artifactId>spatial4j</artifactId>
     <version>0.5</version>
</dependency>
```

执行步骤如下。

1. 根据用户的经纬度、搜索距离获取外接正方形。
2. 执行 SQL，查询出经纬度在正方形范围内的数据。
3. 剔除距离超过指定距离的多余用户数据（不需要很精确，想要更多配对的话，不用执行）。

```java
/**
 * 获取附近 x 米的人
 *
 * @param distance 搜索距离范围 单位km
 * @param userLng  当前用户的经度
 * @param userLat  当前用户的纬度
 */
public String nearBySearch(double distance, double userLng, double userLat) {
  //1.获取外接正方形
  Rectangle rectangle = getRectangle(distance, userLng, userLat);
  //2.获取位置在正方形内的所有用户
  List<User> users = userMapper.selectUser(rectangle.getMinX(), rectangle.getMaxX(), rectangle.getMinY(), rectangle.getMaxY());
  //3.剔除半径超过指定距离的多余用户
  users = users.stream()
    .filter(a -> getDistance(a.getLongitude(), a.getLatitude(), userLng, userLat) <= distance)
    .collect(Collectors.toList());
  return JSON.toJSONString(users);
}

// 获取外接矩形
private Rectangle getRectangle(double distance, double userLng, double userLat) {
  return spatialContext.getDistCalc()
    .calcBoxByDistFromPt(spatialContext.makePoint(userLng, userLat),
                         distance * DistanceUtils.KM_TO_DEG, spatialContext, null);
}

     /***
     * 球面中，两点间的距离
     * @param longitude 经度1
     * @param latitude  纬度1
     * @param userLng   经度2
     * @param userLat   纬度2
     * @return 返回距离，单位km
     */
    private double getDistance(Double longitude, Double latitude, double userLng, double userLat) {
        return spatialContext.calcDistance(spatialContext.makePoint(userLng, userLat),
                spatialContext.makePoint(longitude, latitude)) * DistanceUtils.DEG_TO_KM;
    }
```

getDistance 方法可以获取两点之间的距离，所以用户间距离的排序可以在业务代码中实现，SQL 语句也非常的简单。

```sql
SELECT * FROM nearby_user
WHERE
(longitude BETWEEN #{minlng} AND #{maxlng})
AND (latitude BETWEEN #{minlat} AND #{maxlat})
```

**但是数据库查询性能毕竟有限，如果「附近的人」查询请求非常多，在高并发场合，这可能并不是一个很好的方案。**

#### 尝试 Redis Hash 未果

我们一起分析下 LBS 数据的特点。

1. 每个“女神”都有一个 ID 编号，每个 ID 对应着经纬度信息。
2. “靓仔”登陆 `app` 查找附近的人的时候，`app`根据“靓仔”的经纬度获取指定范围内的“女神信息”。
3. 获取到位置符合的“女神 ID” 列表后，再根据 ID 从数据库查询“女神”列表返回用户。

数据特点就是一个女神（用户）对应着一组经纬度，让我想到了 Redis 的 Hash 结构，也就是一个 key（女神 ID） 对应着 value（经纬度）。

![][image-53]

图 2-39

`Hash`看起来好像可以实现，但是 LBS 应用除了记录经纬度以外，还需要对 Hash 集合中的数据进行范围查询，根据经纬度换算成距离排序。**而 Hash 集合的数据是无序的，显然不可取**。

#### Sorted Set 初见端倪

> Sorted Set 类型是否合适呢？因为它可以排序。

Sorted Sets 集合中，每个元素由两部分组成，分别是 member 和 score。可以根据权重分数对 member 排序，这样看起来就满足我的需求了。比如，member 存储 “女神 ID”，score 是该女神的经纬度信息。

![][image-54]

图 2-40

> 还有一个问题，Sorted Set 元素的权重值是一个浮点数，经纬度是经度、纬度两个值，咋办呢？如何将经纬度转换成一个浮点数呢？

思路对了，为了实现对经纬度比较，Redis 采用业界广泛使用的 GeoHash 编码，分别对经度和纬度编码，最后再把经纬度各自的编码组合成一个最终编码。

这样就实现了将经纬度转换成一个值，而 **Redis 的 GEO 类型的底层数据结构用的就是 `Sorted Set`来实现**。

### 2.7.2 修炼心法

#### GEOHash 编码

Geohash 算法就是将经纬度编码，将二维变一维，给地址位置分区的一种算法，核心思想是区间二分：将地球编码看成一个二维平面，然后将这个平面递归均分为更小的子块。

一共可以分为三步。

1. 将经纬度变成一个 N 位二进制。
2. 将经纬度的二进制合并。
3. 按照 Base32 进行编码。

##### 经纬度编码

GeoHash 编码会把一个经度值编码成一个 N 位的二进制值，比如对经度范围[-180,180] 做 N 次的二分区操作，其中 N 可以自定义。

在进行第一次二分区时，经度范围[-180,180]会被分成两个子区间：[-180,0) 和[0,180]（我称之为左、右分区）。

此时，我们可以查看一下要编码的经度值落在了左分区还是右分区。如**果是落在左分区，我们就用 0 表示；如果落在右分区，就用 1 表示**。这样一来，**每做完一次二分区，我们就可以得到 1 位编码值（不是 0 就是 1）。**

再对经度值所属的分区再做一次二分区，同时再次查看经度值**落在了二分区后的左分区还是右分区，按照刚才的规则再做 1 位编码**。当做完 N 次的二分区后，经度值就可以用一个 N bit 的数来表示了。

**所有的地图元素坐标都将放置于唯一的方格中。分区次数越多方格越小，坐标越精确。然后对这些方格进行整数编码，越是靠近的方格编码越是接近。**

编码之后，每个地图元素的坐标都将变成一个整数，通过这个整数可以还原出元素的坐标，整数越长，还原出来的坐标值的损失程度就越小。对于「附近的人」这个功能而言，损失的一点精确度可以忽略不计。

比如对经度值等于 `169.99` 进行 4 位编码（N = 4，做 4 次分区），把经度区间[-180,180]分成了左分区[-180,0) 和右分区[0,180]。

1. 169.99 属于右分区，使用 `1` 表示第一次分区编码；
2. 再将 169.99 经过第一次划分所属的 [0, 180] 区间继续分成 [0, 90) 和 [90, 180]，169.99 依然在右区间，编码 ‘1’。
3. 将[90, 180] 分为[90, 135) 和 [135, 180]，这次落在左分区，编码 ‘0’。

而纬度的编码思路跟经度也是一样的，不再赘述。

##### 合并经纬度编码

假如计算的经纬度编码分别是 `11011` 和 `00101`，合并编码第 0 位从经度编码的第 0 位的值决定，合并编码的第 1 位从纬度编码第 0 位值 0 决定，以此类推。

其实质其实是二分法。不断地将地球的经度、纬度范围，进行**二分**，输出 1/0 比特，**偶数位放经度，奇数位放纬度**，把 2 串编码组合生成新串形成一串二进制码（二分的次数越多，输出的 bit 串越长），然后将这一串二进制码，按照 5bit 一组进行**base32 编码**，得到最终结果。

![][image-55]

图 2-41

就这样，经纬度（35.679，114.020）就可以使用 `1010011011` 表示，而这个值就可以作为 `SortedSet` 的权重值实现排序。

每个地理位置的坐标由 `geo.h` 的 `geoPoint` 结构体定义，所有的坐标信息存放在 geoArray 数组中。

```c
typedef struct geoPoint {
    double longitude;
    double latitude;
    double dist;
    double score;
    char *member;
} geoPoint;

typedef struct geoArray {
    struct geoPoint *array;
    size_t buckets;
    size_t used;
} geoArray;
```

#### 添加地理位置执行原理

添加地理位置信息到 `Geospatial` 的核心源代码在 `geo.c` 文件的`geoaddCommand` 中，主要步骤如下，我删除部分代码方便理解。

```c
void geoaddCommand(client *c) {
    int xx = 0, nx = 0, longidx = 2;
    int i;

  // 1. 解析可选命令可选参数。
    while (longidx < c->argc) {
        char *opt = c->argv[longidx]->ptr;
        if (!strcasecmp(opt,"nx")) nx = 1;
        else if (!strcasecmp(opt,"xx")) xx = 1;
        else if (!strcasecmp(opt,"ch")) { /* Handle in zaddCommand. */ }
        else break;
        longidx++;
    }

   // 省略部分代码...
  ......

    /* 2. 创建一个参数数组，用于构建调用 ZADD 命令所需的参数和命令 */
    int elements = (c->argc - longidx) / 3;
    int argc = longidx+elements*2; /* ZADD key [CH] [NX|XX] score ele ... */
    robj **argv = zcalloc(argc*sizeof(robj*));
    argv[0] = createRawStringObject("zadd",4);
    for (i = 1; i < longidx; i++) {
        argv[i] = c->argv[i];
        incrRefCount(argv[i]);
    }

 // 省略部分代码
	......

  // 3. 将经纬度转换成 GeoHah 编码作为 zset 的 score 部分
        GeoHashBits hash;
        geohashEncodeWGS84(xy[0], xy[1], GEO_STEP_MAX, &hash);
        GeoHashFix52Bits bits = geohashAlign52Bits(hash);
        robj *score = createObject(OBJ_STRING, sdsfromlonglong(bits));
        robj *val = c->argv[longidx + i * 3 + 2];
        argv[longidx+i*2] = score;
        argv[longidx+1+i*2] = val;
        incrRefCount(val);
    }

		// 4. 使用 replaceClientCommandVector 替换客户端的命令参数向量，然后调用 zaddCommand 来实际执行 ZADD 命令，将位置成员添加到有序集合中
    replaceClientCommandVector(c,argc,argv);
    zaddCommand(c);
}
```

1. 解析命令可选参数。比如 NX、XX、CH 等选项。
2. 创建一个参数数组，用于构建调用 ZADD 命令所需的参数和命令，指令 `ZADD key [CH] [NX|XX] score ele ...` 的命令。
3. 将一组经度和纬度坐标转换为 GeoHash 编码，并将编码后的结果和相关的值作为有序集合的 score 和 member，构建成一个 Redis 命令的参数数组 `argv`。这样，就可以通过执行相应的 sorted Set 的相关命令将这些坐标和值添加到有序集合中。这部分是核心，我逐步解释。
   1. `GeoHashBits hash;`: 这是一个结构体变量，用于存储 GeoHash 编码的结果。
   2. `geohashEncodeWGS84(xy[0], xy[1], GEO_STEP_MAX, &hash);`: 这是一个函数调用，用于将 WGS84 坐标（经度和纬度）编码为 GeoHash。`xy[0]` 是经度，`xy[1]` 是纬度，`GEO_STEP_MAX` 可能是指定编码的精度。函数执行后，编码结果将存储在 `hash` 变量中。
   3. `GeoHashFix52Bits bits = geohashAlign52Bits(hash);`: 这是另一个函数调用，用于将 GeoHash 编码对齐为 52 位。GeoHash 实际上是一个可变长度的编码，在我的（Redis）世界里，通常会使用 52 位的固定长度编码。函数执行后，固定长度的编码结果将存储在 `bits` 变量中。
   4. `robj *score = createObject(OBJ_STRING, sdsfromlonglong(bits));`: 这行代码创建了一个 Redis 对象 `score`，其类型是 "string"，并将 GeoHash 编码结果 `bits` 转换为字符串格式。GeoHash 编码通常是二进制数据，为了在 Redis 中使用，需要将其转换为字符串类型的对象。
   5. `robj *val = c->argv[longidx + i * 3 + 2];`: 这行代码从输入参数 `c->argv` 中获取与当前元素相关的值（value）。在这段代码的上下文中，`c->argv` 是一个包含 Redis 命令的参数的数组。
   6. `argv[longidx+i*2] = score;` 和 `argv[longidx+1+i*2] = val;`: 这两行代码将前面创建的 `score` 对象和获取的值对象 `val` 分别放入 Redis 命令的参数数组 `argv` 中。在这段代码的上下文中，这个过程是为了构建一个可以传递给 Redis 命令的参数数组。
   7. `incrRefCount(val);`: 这行代码增加了值对象 `val` 的引用计数。在 Redis 的对象引用计数机制中，当一个对象被引用时，需要增加其引用计数。在这个上下文中，是为了确保在参数数组 `argv` 使用这个对象时，它的引用计数正确。
4. 使用 `replaceClientCommandVector` 替换客户端的命令参数，然后调用 `zaddCommand` 来实际执行 ZADD 命令，将位置成员添加到有序集合中。

#### 地理位置信息查询

添加地理位置信息到 `Geospatial` 的核心源代码在 `geo.c` 文件的`geoaddCommand` 中，主要步骤如下，作为贴心哥，我删除部分代码方便你理解。

```c
void georadiusGeneric(client *c, int srcKeyIndex, int flags) {
   // 省略部分源码
  ......
   // 1. 从命令参数中解析出相关的选项和参数，包括搜索的中心点坐标、搜索半径或区域尺寸、排序方式、返回结果的数量等。一大坨，大家知道这里主要就是干这个事情就好，不用在意这个细节。


    int withdist = 0, withhash = 0, withcoords = 0;
    int frommember = 0, fromloc = 0, byradius = 0, bybox = 0;
    int sort = SORT_NONE;
    int any = 0;
    if (c->argc > base_args) {
        int remaining = c->argc - base_args;
        for (int i = 0; i < remaining; i++) {
            char *arg = c->argv[base_args + i]->ptr;
            if (!strcasecmp(arg, "withdist")) {
                withdist = 1;
            } else if (!strcasecmp(arg, "withhash")) {
                withhash = 1;
            } else if (!strcasecmp(arg, "withcoord")) {
                withcoords = 1;
            } else if (!strcasecmp(arg, "any")) {
                any = 1;
            } else if (!strcasecmp(arg, "asc")) {
                sort = SORT_ASC;
            } else if (!strcasecmp(arg, "desc")) {
                sort = SORT_DESC;
            } else if (!strcasecmp(arg, "count") && (i+1) < remaining) {
                if (getLongLongFromObjectOrReply(c, c->argv[base_args+i+1],
                                                 &count, NULL) != C_OK) return;
                if (count <= 0) {
                    addReplyError(c,"COUNT must be > 0");
                    return;
                }
                i++;
            } else if (!strcasecmp(arg, "store") &&
                       (i+1) < remaining &&
                       !(flags & RADIUS_NOSTORE) &&

             // 省略一些 代码
        }
    }

   // 省略部分代码
  ......


    // 2.根据给定的搜索条件，计算出所有邻近的 geohash 区域，这些区域可能包含符合搜索条件的地理位置点。
    GeoHashRadius georadius = geohashCalculateAreasByShapeWGS84(&shape);

    // 3. 遍历指定的有序集合（zset），根据计算得到的 geohash 区域信息，找出所有符合条件的地理位置点，并将它们存储在一个 geoArray 结构中。
    geoArray *ga = geoArrayCreate();
    membersOfAllNeighbors(zobj, &georadius, &shape, ga, any ? count : 0);

    // 4. 如果没有指定存储目标键（storekey），则将搜索结果作为回复返回给客户端。否则，将搜索结果存储在一个新的有序集合中，存储键为 storekey，并返回结果数量。
    // 省略部分代码
		......
  // 4.1 如果没有指定存储目标键（storekey），则将搜索结果作为回复返回给客户端。
    if (storekey == NULL) {
       // 省略部分代码
      ......

        addReplyArrayLen(c, returned_items);


       // 省略部分代码
      ......
    } else {

      省略部分代码....
        // 4.2 否则，将搜索结果存储在一个新的有序集合中，存储键为 storekey，并返回结果数量。


        for (i = 0; i < returned_items; i++) {
            // 省略部分代码
          ......
            znode = zslInsert(zs->zsl,score,gp->member);
            serverAssert(dictAdd(zs->dict,gp->member,&znode->score) == DICT_OK);
            gp->member = NULL;
        }

        if (returned_items) {
          // 判断是否需要使用 listpack 数据结构来存储
            zsetConvertToListpackIfNeeded(zobj,maxelelen,totelelen);
            setKey(c,c->db,storekey,zobj,0);
            decrRefCount(zobj);

        } else if (dbDelete(c->db,storekey)) {
            // 省略部分代码......
        }
        addReplyLongLong(c, returned_items);
    }
    geoArrayFree(ga);
}

```

下面是代码逻辑的简要解释：

1. 首先，从命令参数中解析出相关的选项和参数，包括搜索的中心点坐标、搜索半径或区域尺寸、排序方式、返回结果的数量等。
2. 然后，根据给定的搜索条件，计算出所有邻近的 geohash 区域，这些区域可能包含符合搜索条件的地理位置点。
3. 接着，遍历指定的有序集合（zset），根据计算得到的 geohash 区域信息，找出所有符合条件的地理位置点，并将它们存储在一个 geoArray 结构中。
4. 如果没有指定存储目标键（storekey），则将搜索结果作为回复返回给客户端。否则，将搜索结果存储在一个新的有序集合中，存储键为 storekey，并返回结果数量。

### 2.7.3 出招实战：附近的人

Redis Geospatial 数据类型采用了 GeoHash 编码算法，把经纬度经过 GeoHash 编码的合并值作为 Sorted Set 元素的 score 权重。

你可以使用 Geospatial 提供的两个指令来是实现附近的人这个功能。

- `GEOADD`，把地理位置信息（longitude, latitude, name）添加到集合中，注意，经度位于纬度之前。
- `GEOSEARCH`，搜索位于给定形状指定的区域内的地理位置数据，除了支持在圆形区域搜索，还支持矩形区域内搜索。该命令从 6.2.0 开始提供，用于代替已弃用的 `GEORADIUS`和 `GEORADIUSBYMEMBER` 命令。

#### GEOADD

比如一次记录多个用户（“苍无井空”、“波节野衣”）的地理位置信息到集合中。

```shell
GEOADD girl:localtion 13.361389 38.115556 "苍无井空" 15.087269 37.502669 "波节野衣" 15.087269 37.502669 "码哥"
```

解释下语法，掌握真本事。

```shell
GEOADD key [NX | XX] [CH] longitude latitude member [longitude
  latitude member ...]
```

- `key`，没啥好说的，例子中 “girl:localtion” 就是 key。
- `NX`，不更新已存在的元素，只添加新元素。
- `XX`，只更新已经存在的元素，不添加新元素。
- `CH`，表示返回值为修改的元素总数，包括添加的新元素和更新坐标的已存在元素。
- `longitude`，经度。
- `latitude`，维度。
- `member`，元素内容，与地理位置关联的数据，可以是任何字符串。

`GEOADD` 指令的源代码在 `geo.c 的 geoaddCommand`方法中。

#### 移除地理位置

> 如何删除下线的用户经纬度信息呢？

这个问题问得好，`Geospatial` 类型是基于 `Sorted Set` 实现的，可以借用 `ZREM` 命令实现对地理位置信息的删除。比如删除 “苍无井空” 的地理位置信息。

```shell
ZREM girl:localtion "苍无井空"
```

#### GEOSEARCH

> 现在，我登录了 APP，如何根据我的经纬度信息查找附近指定范围内的女神呢？

别着急，`Geospatial` 提供了 `GEOSEARCH` 用于搜索指定地理位置的数据。假设自己的经纬度是（15.087269 37.502669），需要获取附近 10 km 的“女神”数据，由近到远排序。

```shell
GEOSEARCH girl:localtion FROMLONLAT 15.087269 37.502669 BYRADIUS 10 KM ASC WITHCOORD WITHDIST
1) 1) "波节野衣"
   2) "0.0002"
   3) 1) "15.08726745843887329"
      2) "37.50266842333162032"
```

指令还有点复杂，发你会的响应信息也有些复杂，我来分别解释下，不要慌。

```shell
GEOSEARCH key
<FROMMEMBER member | FROMLONLAT longitude latitude>
<BYRADIUS radius <M | KM | FT | MI> | BYBOX width height <M | KM | FT | MI>>
[ASC | DESC]
[COUNT count [ANY]]
[WITHCOORD] [WITHDIST] [WITHHASH]
```

为了满足各种骚操作查询，Geospatial 数据类型对查询 `GEOSEARCH` 指令做了很多各种参数来满足需求。

比如，支持使用 **member 或者经纬度**来作为中心点执行范围搜索，这是一个必填参数。

- `FROMMEMBER member`，使用 member 作为中心点，比如当前当前登录用户 “码哥”。
- `FROMLONLAT longitude latitude`，根据给定的经纬度作为中心来搜索。

除此之外，还支持**不同的形状**作为搜索区域，这是一个必填参数。

- `BYRADIUS`，允许用户在给定的半径范围内（radius）搜索，单位可以是 `M | KM | FT | MI`。
- `BYBOX`，允许用户在一个由高度（height）和宽度（width）确定的轴对齐的矩形内搜索，单位可以是 `M | KM | FT | MI`。

搜索的数据还可以排序返回，这也是一个可选参数。

- `ASC` 按照当前用户经纬度作为中心点，数据按照由近到远排序。
- `DESC`，反之，按照由远到近排序。

`COUNT` 选项表示指定返回的数据数量，防止附近“女神”太多，节省带宽资源。如果觉得自己需要更多女神列表，那么可以无限制，但是需要注意身体，多吃鸡蛋补一补。

除此之外，你还可以控制命令返回的格式信息，这是一个可选参数。如果，没有设置该参数，指令只会返回一个数组，数组的元素是 `member` 比如返回 [“波节野衣”, “苍无井空”, “码哥”]。

- `WITHDIST`：返回匹配数据项与指定中心点的距离，距离的单位与半径或高度和宽度参数与指定的中心点单位相同。
- `WITHCOORD`：返回匹配数据的经度和纬度。
- `WITHHASH`：返回匹配数据的原始 geohash 编码，以 52 位无符号整数的形式表示，一般用户对这个没兴趣。

在 Redis 源码中，定义了一个结构体 `GeoShape`，用于表示地理空间搜索的形状和相关参数。

```c
typedef struct {
    int type;
    double xy[2];
    double conversion;
    double bounds[4];
    union {
        double radius;
        struct {
            double height;
            double width;
        } r;
    } t;
} GeoShape;
```

1. `int type;`: 表示搜索类型，可以是圆形搜索或矩形搜索。通常使用预定义的常量或枚举来标识搜索类型，例如 `CIRCULAR_TYPE` 和 `RECTANGLE_TYPE`。

2. `double xy[2];`: 表示搜索中心点的经纬度坐标。`xy[0]` 存储经度（Longitude），`xy[1]` 存储纬度（Latitude）。

3. `double conversion;`: 表示搜索半径或矩形的高度和宽度的单位转换因子。通常以公里为单位，因此 `conversion` 可能为 1000，将距离转换为米（即 1 km = 1000 m）。

4. `double bounds[4];`: 表示搜索区域的边界框（bounding box）。`bounds[0]` 和 `bounds[1]` 分别表示最小经度和最小纬度，`bounds[2]` 和 `bounds[3]` 分别表示最大经度和最大纬度。这个边界框用于限制搜索结果在指定范围内。

5. `union { ... } t;`: 这是一个联合体（union），用于根据搜索类型存储不同的参数。

   a. 如果 `type` 是圆形搜索 (`CIRCULAR_TYPE`)，则使用 `radius` 存储搜索半径。这个圆形搜索将在以 `xy` 为中心，半径为 `radius` 的圆内进行搜索。

   b. 如果 `type` 是矩形搜索 (`RECTANGLE_TYPE`)，则使用 `r` 结构体存储搜索矩形的高度和宽度。这个矩形搜索将在以 `xy` 为中心，高度为 `r.height`，宽度为 `r.width` 的矩形内进行搜索。

通过这个 `GeoShape` 结构体，可以灵活地表示不同形状和尺寸的地理空间搜索，并根据实际需求使用圆形或矩形搜索来查找地理位置的数据。**Geospatial 本身并没有设计新的底层数据结构，而是直接使用了 Sorted Set 类型。**

**使用 GeoHash 编码方法把从经纬度转换成 Sorted Set 中元素权重分数，这其中的两个关键机制就是对二维地图做区间划分，以及对区间进行编码。**

一组经纬度落在某个区间后，就用区间的编码值来表示，并把编码值作为 `Sorted Set` 元素的权重分数。

在一个地图应用中，车的数据、餐馆的数据、人的数据可能会有百万千万条，如果使用 Redis 的 `Geospatial` 数据结构来保存，将会出现 Bigkey。

在 Redis 的集群环境中，集合可能会从一个节点迁移到另一个节点，如果单个 key 的 value 过大，会对集群的迁移工作造成较大的影响，**在集群环境中单个 key 对应的数据大小不宜超过 5M**，否则会导致集群迁移出现卡顿现象，影响线上服务的正常运行。

所以，建议 `Geospatial` 的数据使用单独的 Redis 集群实例部署。

**如果数据量过亿甚至更大，就需要对 `Geospatial` 数据进行拆分，按国家拆分、按省拆分，按市拆分，在人口特大城市甚至可以按区拆分。**这样就可以显著降低单个 `Sorted Set` 集合的大小。

## 2.8 Bitmaps（位图）

在移动应用的业务场景中，我们需要保存这样的信息：一个 key 关联了一个数据集合。

常见的场景如下：

- 用户在线状态统计：可以使用位图来记录用户的在线状态，其中每个位表示一个用户的在线状态（在线为 1，离线为 0）。这样可以高效地统计在线用户数量和在线用户的分布情况。
- 用户签到记录：位图可以用于记录用户的签到情况，其中每个位表示一个日期（已签到为 1，未签到为 0）。这样可以轻松统计用户的连续签到天数、活跃用户数等信息。
- 页面点击量统计：位图可以用于统计网站的页面点击量，其中每个位表示一个页面的点击情况（点击为 1，未点击为 0）。这样可以快速获取每个页面的点击量以及总点击量。
- 在线商品库存管理：位图可以用于管理在线商品的库存情况，其中每个位表示一个商品的库存状态（有库存为 1，无库存为 0）。这样可以高效地查询商品的库存情况。
- 布隆过滤器（Bloom Filter）：位图可以用于实现布隆过滤器，用于快速判断一个元素是否可能存在于集合中。布隆过滤器适用于判定某个元素可能不在集合中，但不能确定元素一定存在于集合中的情况。
- 用户权限管理：位图可以用于管理用户的权限信息，其中每个位表示一个权限（拥有权限为 1，无权限为 0）。这样可以高效地判断用户是否具有某项权限。
- 基于时间的事件触发：位图可以用于基于时间的事件触发，其中每个位表示一个时间点，当某个时间点的位被设置为 1 时，触发相应的事件。

通常情况下，我们面临的用户数量以及访问量都是巨大的，比如百万、千万级别的用户数量，或者千万级别、甚至亿级别的访问信息。所以，我们必须要选择能够非常高效地统计大量数据（例如亿级）的集合类型。

### 2.8.1 是什么

Redis Bitmap（位图）是 Redis 提供的一种特殊的数据结构，用于处理位级别的数据。

实际上是在 String 类型上定义的面向 bit 位的操作，将位图存储在字符串中，每个字符代表 8 位二进制，是一个由二进制位（bit）组成的数组，其中的每一位只能是 0 或 1。String 数据类型最大容量是 512MB，所以一个 Bitmap 最多可设置 2^32 个不同位。

Bitmap 解决的是二值状态统计场景问题。也就是集合中的元素的值只有 0 和 1 两种，在签到打卡和用户是否登陆的场景中，只需记录`签到(1)`或 `未签到(0)`，`已登录(1)`或`未登陆(0)`。

假如我们在判断用户是否登陆的场景中使用 Redis 的 String 类型实现（**key -\> userId，value -\> 0 表示下线，1 - 登陆**），假如存储 100 万个用户的登陆状态，如果以字符串的形式存储，就需要存储 100 万个字符串了，内存开销太大。

> 为何 String 类型内存开销大呢？

String 类型除了记录实际数据以外，还需要额外的 len（数组已使用长度）、alloc（buf 数组的分配空间总长度）等信息。

![图2-42][image-56]

图 2-42

- **len**：占 4 个字节，表示 buf 的已用长度。
- **alloc**：占 4 个字节，表示 buf 实际分配的长度，通常 \> len。
- **buf**：字节数组，保存实际的数据，Redis 自动在数组最后加上一个 “\0”，额外占用一个字节的开销。

所以，在 SDS 中除了 buf 保存实际的数据， len 与 alloc 就是额外的开销。

另外，还有一个 **RedisObject 结构的开销**，因为 Redis 的数据类型有很多，而且，不同数据类型都有些相同的元数据要记录（比如最后一次访问的时间、被引用的次数等）。

所以，Redis 会用一个 `RedisObject` 结构体来统一记录这些元数据，ptr 指针指向实际数据。

![][image-57]

图 2-43

对于二值状态场景，我们就可以利用 Bitmap 来实现。比如登陆状态我们用一个 bit 位表示，一亿个用户也只占用一亿 个 bit 位内存 ≈ （100000000 / 8/ 1024/1024）12 MB。

```
大概的空间占用计算公式是：($offset/8/1024/1024) MB
```

### 2.8.2 修炼心法

Bitmap 的底层数据结构用的是 String 类型的 SDS 数据结构来保存位数组，Redis 把每个字节数组的 8 个 bit 位利用起来，每个 bit 位 表示一个元素的二值状态（不是 0 就是 1）。

可以将 Bitmap 看成是一个 bit 为单位的数组，数组的每个单元只能存储 0 或者 1，数组的每个 bit 位下标在 Bitmap 中叫做 offset 偏移量。

为了直观展示，我们可以理解成 buf 数组的每个槽位中的字节用一行表示，每一行有 8 个 bit 位，8 个格子分别表示这个字节中的 8 个 bit 位，如下图所示：

![2-44][image-58]

图 2-44

**8 个 bit 组成一个 Byte，所以 Bitmap 会极大地节省存储空间。** 这就是 Bitmap 的优势。

#### 设置 Bitmap offset value 原理

Bitmap 提供了 `SETBIT` 指令设置或者清空 bitmap 集合的 offset 处的 bit 为 value（只能是 0 或者 1）。

命令的实现在源代码 `bitops.c` 文件的 `setbitCommand` 方法实中，我省略了一些代码便于理解。

```c
/* SETBIT key offset bitvalue */
void setbitCommand(client *c) {
    // 省略部分代码
  ....

		// 1. 命令参数中解析出偏移量 bitoffset，表示要设置的位在位图中的位置。
    if (getBitOffsetFromArgument(c,c->argv[2],&bitoffset,0,0) != C_OK)
        return;
		// 2. 从命令参数中解析出位的值 `on`，这个值只能是 0 或 1，表示要设置的位值。
    if (getLongFromObjectOrReply(c,c->argv[3],&on,err) != C_OK)
        return;

    /* 3. 如果 on 不是 0 或 1，即不是有效的位值，将返回错误回复并结束 */
    if (on & ~1) {
        addReplyError(c,err);
        return;
    }
    // 4. 查找并返回键 c->argv[1] 对应的字符串对象 o。字符串对象在 Redis 中用于存储位图。
    int dirty;
    if ((o = lookupStringForBitCommand(c,bitoffset,&dirty)) == NULL) return;

    /* 5. 计算偏移量 bitoffset 对应的字节索引 byte 和位索引 bit。由于 Redis 中的位图是按字节存储的，所以需要计算偏移量对应的字节位置和位位置。 */
    byte = bitoffset >> 3;
    byteval = ((uint8_t*)o->ptr)[byte];
    bit = 7 - (bitoffset & 0x7);
  	// 6. 获取字节中指定位的当前值 bitval。
    bitval = byteval & (1 << bit);


    /* 7. 比较当前位值 bitval 与要设置的位值 on 是否相同。如果位值有变化，或者该位是新创建的，或者位图长度发生变化，那么进行位值更新。 */
    if (dirty || (!!bitval != on)) {
        /* Update byte with new bit value. */
        byteval &= ~(1 << bit);
        byteval |= ((on & 0x1) << bit);
        ((uint8_t*)o->ptr)[byte] = byteval;
        signalModifiedKey(c,c->db,c->argv[1]);
        // 省略部分代码
      ...
        server.dirty++;
    }

    /* 8. 返回设置前的位值 bitval 作为回复 */
    addReply(c, bitval ? shared.cone : shared.czero);
}
```

1. `getBitOffsetFromArgument` 函数用于从命令参数中解析出偏移量 `bitoffset`，表示要设置的位在位图中的位置。
2. `getLongFromObjectOrReply` 函数用于从命令参数中解析出位的值 `on`，这个值只能是 0 或 1，表示要设置的位值。
3. 如果 `on` 不是 0 或 1，即不是有效的位值，将返回错误回复并结束。
4. `lookupStringForBitCommand` 函数用于查找并返回键 `c->argv[1]` 对应的字符串对象 `o`。字符串对象在 Redis 中用于存储位图。
5. 计算偏移量 `bitoffset` 对应的字节索引 `byte` 和位索引 `bit`。由于 Redis 中的位图是按字节存储的，所以需要计算偏移量对应的字节位置和位位置。
6. 获取字节中指定位的当前值 `bitval`。
7. 比较当前位值 `bitval` 与要设置的位值 `on` 是否相同。如果位值有变化，或者该位是新创建的，或者位图长度发生变化，那么进行位值更新。
8. 最后，返回设置前的位值 `bitval` 作为回复。

#### 获取 offset value 原理

获取 key 关联的 Bitmap 在 offset 处的 bit 位的值，当 key 不存在时，返回 0。源代码 `bitops.c`的 `getbitCommand` 方法实现了 `GETBIT` 命令。

```c
/* GETBIT key offset */
void getbitCommand(client *c) {
    省略部分代码...
		// 1. 从命令参数中解析出偏移量 `bitoffset`，表示要获取的位在位图中的位置
    if (getBitOffsetFromArgument(c,c->argv[2],&bitoffset,0,0) != C_OK)
        return;
		// 2. 查找并返回键 c->argv[1] 对应的字符串对象 o。如果键不存在，或者类型不是字符串对象，将返回零值的回复。
    if ((o = lookupKeyReadOrReply(c,c->argv[1],shared.czero)) == NULL ||
        checkType(c,o,OBJ_STRING)) return;

  // 3. 计算偏移量 bitoffset 对应的字节索引 byte 和位索引 bit。由于 Redis 中的位图是按字节存储的，所以需要计算偏移量对应的 byte 位置和 bit 位置。
    byte = bitoffset >> 3;
    bit = 7 - (bitoffset & 0x7);

  	// 4. 根据字符串对象类型，从字节中获取指定位的值 bitval。
    if (sdsEncodedObject(o)) {
        if (byte < sdslen(o->ptr))
            bitval = ((uint8_t*)o->ptr)[byte] & (1 << bit);
    } else {
        if (byte < (size_t)ll2string(llbuf,sizeof(llbuf),(long)o->ptr))
            bitval = llbuf[byte] & (1 << bit);
    }
		// 5. 将获取到的位值 bitval 作为回复发送给客户端
    addReply(c, bitval ? shared.cone : shared.czero);
}
```

1. `getBitOffsetFromArgument` 函数用于从命令参数中解析出偏移量 `bitoffset`，表示要获取的位在位图中的位置。
2. 使用 `lookupKeyReadOrReply` 函数查找并返回键 `c->argv[1]` 对应的字符串对象 `o`。如果键不存在，或者类型不是字符串对象，将返回零值的回复。
3. 计算偏移量 `bitoffset` 对应的字节索引 `byte` 和位索引 `bit`。由于 Redis 中的位图是按字节存储的，所以需要计算偏移量对应的字节位置和位位置。
4. 根据字符串对象类型，从字节中获取指定位的值 `bitval`。
5. 将获取到的位值 `bitval` 作为回复发送给客户端。

### 2.8.3 出招实战：登录判断、签到统计系统

#### 用户登录判断

Bitmap 提供了 `GETBIT、SETBIT` 操作，通过一个偏移值 offset 对 bit 数组的 offset 位置的 bit 位进行读写操作，需要注意的是 offset 从 0 开始。

可以使用 `key = login_status` 关联一个 Bitmap 集合，表示存储用户登陆状态数据， 将用户 ID 作为 offset，在线就设置为 1，下线设置 0。通过 `GETBIT`判断对应的用户是否在线。 50000 万用户只需要 6 MB 的空间。

**SETBIT 命令**

```shell
SETBIT <key> <offset> <value>
```

设置或者清空指定的 key 关联的 bitmap 在 offset 处的 bit 值（只能是 0 或者 1）。

**GETBIT 命令**

```shell
GETBIT <key> <offset>
```

获取 key 关联的 Bitmap 在 offset 处的 bit 位的值，当 key 不存在时，返回 0。举个例子，假如你要判断 ID = 10086 的用户的登陆情况。

第一步，用户登录时，执行以下指令，表示用户已登录。

```
SETBIT login_status 10086 1
```

第二步，检查该用户是否登陆，返回值 1 表示已登录。

```
GETBIT login_status 10086
```

第三步，登出，将 offset 对应的 value 设置成 0。

```
SETBIT login_status 10086 0
```

#### 用户每月签到情况

在签到统计中，每个用户每天的签到用 1 个 bit 位表示，一个月最多只有 31 天，占用 31 个 bit 位。

考虑到每月要重置连续签到次数，设计思路就是为每个登录用户的每个月创建一个 Bitmap 集合，到期后删除节省内存。`key = uid:sign:{userId}:{yyyyMM}`。**月份的日期值 - 1** 作为 offset（因为 Bitmap offset 从 0 开始，所以 `offset = 日期 - 1`），签到就把这个 offsset 的 bit 位设置成 1。

第一步，执行下面指令表示记录用户在 2023 年 7 月 1 号和 2023 年 7 月 29 号打卡签到。

```
SETBIT uid:sign:89757:202307 0 1
SETBIT uid:sign:89757:202307 28 1
```

执行以上两个指令后，Bitmap 数据就是 `10000000000000000000000000001000（一共 32 位）`。**需要注意的是，虽然你在 offset = 28 的位置设置 bit = 1，实际上 bitmap 占用了 32 个 bit，Bitmap 占用大小是 Byte 的整数倍。**

第二步，判断编号 89757 用户在 2023 年 7 月 29 号是否打卡签到。

```
GETBIT uid:sign:89757:202307 28
```

第三步，统计该用户在 7 月份的签到次数，使用 `BITCOUNT` 指令。该指令用于统计给定的 bit 数组中，值等于 1 的 bit 位的数量。

```
BITCOUNT uid:sign:89757:202307
```

这样我们就可以实现用户每个月的打卡情况了，是不是很赞。

> 如何统计每个月首次签到日期呢？

Bitmap 提供了 `BITPOS key bit [start [end [BYTE | BIT]]]` 指令，返回数据表示 Bitmap 中第一个值为 1 或者 0 的位置。

需要注意的是，该命令会遍历整个 Bitmap，你可以通过可选的`start` 参数和 `end` 参数指定要检测的范围。所以我们可以通过执行以下指令来获取 userID = 89757 在 2023 年 7 月份**首次打卡**日期。

```shell
>BITPOS uid:sign:89757:202307 1
(integer) 0
```

**需要注意的是，我们需要将返回的 value + 1 来作为首次签到日期，因为 offset 从 0 开始。**所以首次签到的日期是 2023 年 07 月 1 号。

使用 Bitmap 之后会超级节省内存，我来给你做个简单计算。

- 1 个用户连续签到一个月产生 31 bit，大约 4 byte（每个月都按 31 天 算，别说我欺负关系型数据库）。
- 一个用户签到一年需要 48 byte。
- 1000 万签到一年会产色会给你 48000 万 byte 数据（48000W byte ÷ 1024 ÷ 1024 ≈ 457.76MB）。

关系型数据库 1000 万签到用户签到一年大约会产生 68.66 TB 数据，而 Bitmap 只需要 457.76MB 数据。

> 谢霸哥：“之前利于上线了同城约会 APP，运营人员提出月份内连续签到越长，发放奖励更多，异常火爆，老板赢麻了。
> 
> 但是问题来了，如何统计一个用户每月签到详情呢（每月签到情况、连续签到天数）？“

Bitmap 提供了 `BITFIELD` 指令，这个指令可以在一次调用对多个 bit 位范围进行操作。语法比较复杂， 我解释下。

```shell
BITFIELD key
  [GET type offset | [OVERFLOW <WRAP | SAT | FAIL>]
    <SET type offset value | INCRBY type offset increment>
    [GET type offset | [OVERFLOW <WRAP | SAT | FAIL>]
      <SET type offset value | INCRBY type offset increment>
    ...]

```

- `key`: 要操作的 Redis 字符串键，存储 Bitmap 数据。

在 `BITFIELD` 命令中，你可以通过连续的子命令链来对同一个字符串键进行多个位操作。每个子命令由一个或多个参数组成，指定要执行的操作、位字段的类型、偏移量、值等。

以下是每个子命令的详细说明：

- `GET type offset`: 从 Bitmap 中获取位字段（bit field）的值。`type` 表示位字段的数据类型，`offset` 表示位偏移量（从 0 开始）。你可以指定多个 `GET` 操作。
- `SET type offset value`: 设置位字段的值。`type` 表示位字段的数据类型，`offset` 表示位偏移量，`value` 表示要设置的值。你可以指定多个 `SET` 操作。
- `INCRBY type offset increment`: 将指定位字段的值递增指定的增量。`type` 表示位字段的数据类型，`offset` 表示位偏移量，`increment` 表示递增的数量。你可以指定多个 `INCRBY` 操作。
- `OVERFLOW <WRAP | SAT | FAIL>`: 当位操作导致溢出时的处理方式。你可以选择 `WRAP`（环绕）、`SAT`（饱和）或 `FAIL`（失败）。这个参数对应于整个子命令链中的溢出处理方式。

**数据类型（type）：**

- `u<N>`：无符号整数类型，其中 `<N>` 是位数。例如，`u8` 表示 8 位无符号整数。
- `i<N>`：有符号整数类型，其中 `<N>` 是位数。例如，`i16` 表示 16 位有符号整数。
- `N`：使用默认类型，可以是 `u` 或 `i`。

如下指令，表示获取 2023 年 7 月从 offset 位置为 0 开始，31 天签到情况，返回值是无符号十进制。

```shell
> BITFIELD uid:sign:89757:202307 GET u31 0
1073741828
```

需要注意的是，一个月最长 31 天，最长需要 31 bit ，29 号签到的时候，实际上 Bitmap 会占用 32 bit，因为底层是 SDS 数据结构，每个 Byte 由 8 bit 组成，其他位置会自动补 0。

> 谢霸哥：“命令返回的是一个 10 进制数，我哪知道到底有哪些 bit 位是 0 还是 1 呀？”

你只需要把这个 10 进制数字和 1 做与运算就可以了，

## 2.9 HyperlogLogs（ 基数统计）

在移动互联网的业务场景中，**数据量很大**，系统需要保存这样的信息：一个 key 关联了一个数据集合，同时对这个数据集合做统计做一个报表给运营人员看。

比如。

- 统计一个 `APP` 的日活、月活数。
- 统计一个页面的每天被多少个不同账户访问量（Unique Visitor，UV）。
- 统计用户每天搜索不同词条的个数。
- 统计注册 IP 数。

通常情况下，系统面临的用户数量以及访问量都是巨大的，比如**百万、千万级别的用户数量，或者千万级别、甚至亿级别**的访问信息，咋办呢？

> Redis：“这些就是典型的基数统计应用场景，基数统计：统计一个集合中不重复元素，这被称为基数。”

### 2.9.1 是什么

HyperLogLog 是一种概率数据结构，用于估计集合的基数。每个 `HyperLogLog` 最多只需要花费 12KB 内存，在标准误差 `0.81%`的前提下，就可以计算 2 的 64 次方个元素的基数。

`HyperLogLog` 的优点在于**它所需的内存并不会因为集合的大小而改变，无论集合包含的元素有多少个，HyperLogLog 进行计算所需的内存总是固定的，并且是非常少的**。

主要特点如下。

- 高效的内存使用：HyperLogLog 的内存消耗是固定的，与集合中的元素数量无关。这使得它特别适用于处理大规模数据集，因为它不需要存储每个不同的元素，只需要存储估计基数所需的信息。
- 概率估计：HyperLogLog 提供的结果是概率性的，而不是精确的基数计数。它通过哈希函数将输入元素映射到位图中的某些位置，并基于位图的统计信息来估计基数。由于这是一种概率性方法，因此可能存在一定的误差，但通常在实际应用中，这个误差是可接受的。
- 高速计算：HyperLogLog 可以在常量时间内计算估计的基数，无论集合的大小如何。这意味着它的性能非常好，不会受到集合大小的影响。

### 2.9.2 修炼心法

**基本原理**

HyperLogLog 是一种概率数据结构，它使用概率算法来统计集合的近似基数。而它算法的最本源则是伯努利过程。

伯努利过程就是一个抛硬币实验的过程。抛一枚正常硬币，落地可能是正面，也可能是反面，二者的概率都是 `1/2` 。

伯努利过程就是一直抛硬币，直到落地时出现正面位置，并记录下抛掷次数`k`。

比如说，抛一次硬币就出现正面了，此时 `k` 为 `1`; 第一次抛硬币是反面，则继续抛，直到第三次才出现正面，此时 `k` 为 3。

对于 `n` 次伯努利过程，我们会得到 n 个出现正面的投掷次数值 `k1, k2 ... kn`, 其中这里的最大值是 `k_max`。

根据一顿数学推导，我们可以得出一个结论： **2^{k\_ max} 来作为 n 的估计值。**

也就是说你可以根据最大投掷次数近似的推算出进行了几次伯努利过程。

所以 HyperLogLog 的基本思想是利用集合中数字的比特串第一个 1 出现位置的最大值来预估整体基数，但是这种预估方法存在较大误差，为了改善误差情况，HyperLogLog 中引入分桶平均的概念，计算 m 个桶的调和平均值。

Redis 内部使用字符串位图来存储 HyperLogLog 所有桶的计数值，一共分了 2^14 个桶，也就是 16384 个桶。每个桶中是一个 6 bit 的数组。

这段代码描述了 Redis HyperLogLog 数据结构的头部定义（hyperLogLog.c 中的 hllhdr 结构体）。以下是关于这个数据结构的各个字段的解释。

```c
struct hllhdr {
    char magic[4];
    uint8_t encoding;
    uint8_t notused[3];
    uint8_t card[8];
    uint8_t registers[];
};
```

1. **magic[4]**：这个字段是一个 4 字节的字符数组，用来表示数据结构的标识符。在 HyperLogLog 中，它的值始终为"HYLL"，用来标识这是一个 HyperLogLog 数据结构。
2. **encoding**：这是一个 1 字节的字段，用来表示 HyperLogLog 的编码方式。它可以取两个值之一：
   3. `HLL_DENSE`：表示使用稠密表示方式。
   4. `HLL_SPARSE`：表示使用稀疏表示方式。
3. **notused[3]**：这是一个 3 字节的字段，目前保留用于未来的扩展，要求这些字节的值必须为零。
4. **card[8]**：这是一个 8 字节的字段，用来存储缓存的基数（基数估计的值）。
5. **egisters[]**：这个字段是一个可变长度的字节数组，用来存储 HyperLogLog 的数据。

![4-45][image-59]

图 2-45

Redis 对 `HyperLogLog` 的存储进行了优化，在计数比较小的时候，存储空间采用系数矩阵，占用空间很小。

只有在计数很大，稀疏矩阵占用的空间超过了阈值才会转变成稠密矩阵，占用 12KB 空间。

### 2.9.3 出招实战：网页访问量统计

在移动互联网的业务场景中，**数据量很大**，系统需要保存这样的信息：一个 key 关联了一个数据集合，同时对这个数据集合做统计做一个报表给运营人员看。

比如。

- 统计一个 `APP` 的日活、月活数。
- 统计一个页面的每天被多少个不同账户访问量（Unique Visitor，UV）。
- 统计用户每天搜索不同词条的个数。
- 统计注册 IP 数。

通常情况下，系统面临的用户数量以及访问量都是巨大的，比如**百万、千万级别的用户数量，或者千万级别、甚至亿级别**的访问信息，咋办呢？

#### 使用 Set 实现

**一个用户一天内多次访问一个网站只能算作一次**，所以很容易就想到通过 **Redis 的 Set 集合**来实现。

比如微信昵称叫 “Chaya” 的小姐姐访问【爱一个人总是要掉眼泪的风险】这篇文章时，我把这个微信昵称 “Chaya” 存到 Set 集合中。

```bash
SADD 爱一个人总是要掉眼泪的风险:uv 码哥 Chaya 赵小因 Chaya
(integer) 3
```

“Chaya” 多次访问这篇文章， Set 的去重特性保证集合中只有一个记录。接着，通过 `SCARD` 命令，统计页面 UV。指令返回这个集合的元素个数（也就是微信昵称个数）。

```bash
SCARD 爱一个人总是要掉眼泪的风险:uv
(integer) 3
```

#### 使用 HyperLogLog 实现

> Chaya：“Set 集合虽好，如果文章非常火爆达到千万级别，一个 Set 集合就保存了千万个用户的 ID，页面多了消耗的内存也太大了。”

不要怕，只要思想不滑坡，办法总比困难多。这些就是典型的基数统计应用场景，**基数统计：统计一个集合中不重复元素的个数。**

`HyperLogLog` 的优点在于**它所需的内存并不会因为集合的大小而改变，无论集合包含的元素有多少个，HyperLogLog 进行计算所需的内存总是固定的，并且是非常少的**。

每个 `HyperLogLog` 最多只需要花费 12KB 内存，在标准误差 `0.81%`的前提下，就可以计算 2 的 64 次方个元素的基数。

HyperLogLog 使用太简单了。`PFADD、PFCOUNT、PFMERGE`三个指令打天下。

#### PFADD

每访问一次页面，调用 `PFADD` 指令 将这个用户 ID 添加到 HyperLogLog 中。如下 一共有三个用户访问了这页面，其中 Chaya 访问了两次，但只算一次。

```bash
PFADD 爱一个人总是要掉眼泪的风险:uv 码哥 Chaya 赵小因 Chaya
```

如果执行命令后 HyperLogLog 估计的近似基数发生变化，`PFADD`则返回 1，否则返回 0。如果指定的键不存在，该命令会自动创建一个空的 HyperLogLog 结构。

`pfadd` 命令并不会一次性分配 12k 内存，而是随着基数的增加而逐渐增加内存分配；

#### PFCOUNT

接下来，通过 `PFCOUNT` 指令获取文章【爱一个人总是要掉眼泪的风险】的 UV 值，可以看到返回值是 3 ，符合预期。

```bash
> PFCOUNT 爱一个人总是要掉眼泪的风险:uv
3
```

#### PFMERGE 合并统计

> Chaya：“还有一个变态需求，对文章进行标签分类，运营说要把都是情感文章标签的几个页面数据合并统计。”

其中页面的 UV 访问量也需要合并，那这个时候 `PFMERGE` 就可以派上用场了，也就是**同样的用户访问这两个页面则只算做一次**。

如下指令，把`爱一个人总是要掉眼泪的风险:uv`和`爱情是幸福和不委屈:uv` 两个 HyperLogLog 集合数据合并到`情感分类文章:uv`这个集合中。

```bash
PFADD 爱情是幸福和不委屈:uv Chaya 赵小因 幸运草
# 合并两个页面 UV
PFMERGE 情感分类文章:uv 爱一个人总是要掉眼泪的风险:uv 爱情是幸福和不委屈:uv
```

接着，执行 `PFCOUNT 情感分类文章:uv` 统计合并后的数据。

```bash
> PFCOUNT 情感分类文章:uv
4
```

**将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。**

## 2.10 Bloom Filter（布隆过滤器）

> MySQL：“Redis 老哥，我遇到难题了。程序员开发了一个“明日头条” APP 阅读新闻资讯或视频，需要实现每次推荐给该用户的内容不重复，过滤历史看过的内容。
> 
> 系统并发量特别大，我快扛不住了，怎么破？ ”

把每个用户浏览过的历史记录存储在 MySQL 中，去重就需要频繁地对数据库进行 exists 查询，当系统并发量很高时，数据库很难扛住压力。

> MySQL：“可以使用 Redis 缓存么，把浏览数据存储在 Redis 中。”

万万不可，这么多的历史记录那要浪费多大的内存空间，这个时候你可以使用布隆过滤器去解决这种**去重问题**。又快又省内存，互联网开发必备杀招！

当你**遇到数据量大，又需要去重的时候就可以考虑布隆过滤器**，如下场景：

- 解决 Redis 缓存穿透问题。
- 邮件过滤，使用布隆过滤器实现邮件黑名单过滤。
- 爬虫爬过的网站过滤，爬过的网站不再爬取。
- 推荐过的新闻不再推荐。

### 2.10.1 是什么

布隆过滤器 (Bloom Filter)是由 Burton Howard Bloom 于 1970 年提出，**它是一种 space efficient 的概率型数据结构，用于判断一个元素是否在集合中**。

是一种空间效率高、时间复杂度低的数据结构，用于检查一个元素是否存在于一个集合中。它通常用于快速判断某个元素是否可能存在于一个大型数据集中，而无需实际存储整个数据集。

**布隆过滤器客户以保证某个数据不存在时，那么这个数据一定不存在；当给出的响应是存在，这个数可能不存在。**

哈希表也能用于判断元素是否在集合中，但是布隆过滤器只需要哈希表的 1/8 或 1/4 的空间复杂度就能完成同样的问题。

**布隆过滤器可以插入元素，但不可以删除已有元素。**

### 2.10.2 修炼心法

Redis 的 Bloom Filter 实现基于一个位数组（bit array）和一组不同的哈希函数。

1. 首先分配一块内存空间做 bit 数组，这个位数组的长度是固定的，通常由用户指定，决定了 Bloom Filter 的容量。每个位都初始为 0。

2. 添加元素时，采用 k 个相互独立的 Hash 函数对这个数据计算，这些哈希函数应该是独立的，均匀分布的，以减小冲突的可能性，然后将元素 Hash 映射的 K 个位置全部设置为 1。

3. 检测 key 是否存在，仍然用这 k 个 Hash 函数计算出 k 个位置，如果位置全部为 1，则表明 key 可能存在，否则不存在。

![2-46][image-60]

图 2-46

**哈希函数会出现碰撞，所以布隆过滤器会存在误判。**这里的误判率是指，BloomFilter 判断某个 key 存在，但它实际不存在的概率。

这个误判率主要受位数组的大小和哈希函数的数量影响。较大的位数组和更多的哈希函数可以降低误判率，但也会增加存储开销和计算开销。

> MySQL：“为什么不允许删除元素呢？”

由于 Bloom Filter 的设计目的是快速检查元素的可能性，而不是支持元素的删除操作，删除意味着需要将对应的 k 个 bits 位置设置为 0，其中有可能是其他元素对应的位。

### 2.10.3 出招实战：缓存穿透预防

Redis Bloom Filter 不是我的的标准功能，而是通过拓展实现的，Redis 4.0 的时候官方提供了插件机制，布隆过滤器正式登场。

你可以下载官方提供的已经编译好的可拓展模块，或者自己去 github 下载源码，自己编译。

接下来我以下载源码编译的方式来说明如何集成 Redis Bloom Filter。

#### 下载源码

从 github 下载，目前的 release 版本是 v2.2.14。

#### 解压编译

解压

```bash
tar -zxvf RedisBloom-2.2.14.tar
```

编译插件

```bash
cd RedisBloom-2.6.3
make
```

编异成功，会看到 `redisbloom.so` 文件。

#### 安装集成

需改 redis.conf 文件，新增 `loadmodule`配置，并重启 Redis。

```bash
loadmodule /opt/app/RedisBloom-2.2.14/redisbloom.so
```

**如果是集群，则每个实例的配置文件都需要加入配置。**

指定配置文件并启动 Redis：

`redis-server /opt/app/redis-6.2.6/redis.conf`。加载成功的页面如下。

![2-47][image-61]

图 2-47

#### 缓存穿透预防

你可以用布隆过滤器来解决缓存穿透问题，缓存穿透：意味着有特殊请求在查询一个不存在的数据，**即数据不存在 Redis 也不存在于数据库。**

当用户购买商品创建订单的时候，你就往 mq 发送消息，把订单 ID 添加到布隆过滤器。

![订单同步到布隆过滤器][image-62]

图 2-48

#### 创建过滤器

在添加到布隆过滤器之前，我们通过`BF.RESERVE`命令手动创建一个名字为 `orders` error\_rate = 0.1 ，初始容量为 10000000 的布隆过滤器。

```bash
BF.RESERVE key error_rate capacity [EXPANSION expansion]
  [NONSCALING]
```

- key：filter 的名字；
- error\_rate：期望的错误率，默认 0.1，值越低，需要的空间越大；
- capacity：初始容量，默认 100，当实际元素的数量超过这个初始化容量时，误判率上升。
- EXPANSION：可选参数，当添加到布隆过滤器中的数据达到初始容量后，布隆过滤器会自动创建一个子过滤器，子过滤器的大小是上一个过滤器大小乘以 expansion；expansion 的默认值是 2，也就是说布隆过滤器扩容默认是 2 倍扩容；
- NONSCALING：可选参数，设置此项后，当添加到布隆过滤器中的数据达到初始容量后，不会扩容过滤器，并且会抛出异常（(error) ERR non scaling filter is full）
  说明：BloomFilter 的扩容是通过增加 BloomFilter 的层数来完成的。每增加一层，在查询的时候就可能会遍历多层 BloomFilter 来完成，每一层的容量都是上一层的两倍（默认）。

如果不使用`BF.RESERVE`命令创建，而是使用 Redis 自动创建的布隆过滤器，**默认的 `error_rate` 是 `0.01`，`capacity`是 100。**

隆过滤器的 error\_rate 越小，需要的存储空间就越大，对于不需要过于精确的场景，error\_rate 设置稍大一点也可以。

布隆过滤器的 capacity 设置的过大，会浪费存储空间，设置的过小，就会影响准确率，所以在使用之前一定要尽可能地精确估计好元素数量，还需要加上一定的冗余空间以避免实际元素可能会意外高出设置值很多。

#### 添加数据到过滤器

```bash
# BF.ADD {key} {item}
BF.ADD orders 10086
(integer) 1
```

使用 `BF.ADD`向名称为 `orders` 的布隆过滤器添加 10086 这个元素。

如果是多个元素同时添加，则使用 `BF.MADD key {item ...}`.

```bash
BF.MADD orders 10087 10089
1) (integer) 1
2) (integer) 1
```

#### 判断元素是否存在

```bash
# BF.EXISTS {key} {item}
BF.EXISTS orders 10086
(integer) 1
```

`BF.EXISTS` 判断一个元素是否存在于`BloomFilter`，返回值 = 1 表示存在，0 表示不存在。

如果需要批量检查多个元素是否存在于布隆过滤器则使用 `BF.MEXISTS`，返回值是一个数组。

```bash
# BF.MEXISTS {key} {item}
BF.MEXISTS orders 100 10089
1) (integer) 0
2) (integer) 1
```

只需要通过`BF.RESERVE、BF.ADD、BF.EXISTS`三个指令就能实现避免缓存穿透问题。

> Chaya：“如何查看创建的布隆过滤器信息呢？”

这个问题问得好，好奇心还是要有的。用 `BF.INFO key`查看。

```bash
BF.INFO orders
 1) Capacity
 2) (integer) 10000000
 3) Size
 4) (integer) 7794184
 5) Number of filters
 6) (integer) 1
 7) Number of items inserted
 8) (integer) 3
 9) Expansion rate
10) (integer) 2
```

- Capacity：预设容量。
- Size：实际占用情况，但如何计算待进一步确认。
- Number of filters：过滤器层数。
- Number of items inserted：已经实际插入的元素数量。
- Expansion rate：子过滤器扩容系数（默认 2）。

## 2.11 Redis 为什么这么快

我叫 Redis，如今已经成了软件系统必备中间件之一，经常出现在面试官的青睐对象。这一章节，主要从面试角度，把上一章的高频知识点提炼，带你融会贯通梳理一遍上一节的知识点。

**学习一个技术，通常只接触了零散的技术点，没有在脑海里建立一个完整的知识框架和架构体系，没有系统观。这样会很吃力，而且会出现一看好像自己会，过后就忘记，一脸懵逼。**一起搭建一套完整的知识框架，学会全局观去整理整个知识体系。

65 哥前段时间去面试 996 大厂，被问到 “Redis 为什么快？”

> 65 哥：“额，因为它是基于内存实现和单线程模型。”

面试官：还有呢？

> 65 哥：“没了呀。”

很多人仅仅只是知道基于内存实现，其他核心的原因模凌两可。今日，一起探索我（Redis） 真正快的原因。

为了让我（Redis）的性能一骑绝尘，Redis 的创始人 antirez 从各方面都进行了优化。下次小伙伴们面试的时候，面试官问 Redis 性能为什么如此高，可不能傻傻的只说单线程和内存存储了。

根据官方数据，Redis 的 QPS 可以达到约 100000（每秒请求数），有兴趣的可以参考官方的基准程序测试《How fast is Redis？》。

![基准测试][image-63]

图 2-49

1. 基于内存实现。
2. 使用 I/O 多路复用模型。
3. 单线程模型。
4. 高效的底层数据结构支撑。
5. 全局散列表。

### 2.11.1 基于内存实现

> 65 哥：“这个我知道，Redis 是基于内存的数据库，跟磁盘数据库相比，完全吊打磁盘的速度，就像段誉的凌波微步。对于磁盘数据库来说，首先要将数据通过 IO 操作读取到内存里。“

没错，不论读写操作都是在内存上完成的，来分别对比下内存操作与磁盘操作的差异。

**磁盘操作调用栈**

![2-50][image-64]

图 2-50

**内存操作**

内存直接由 CPU 控制，也就是 CPU 内部集成的内存控制器，所以说内存是直接与 CPU 对接，享受与 CPU 通信的最优带宽。

**Redis 将数据存储在内存中，读写操作不会因为磁盘的 IO 速度限制，所以速度飞一般的感觉！**

最后以一张图量化系统的各种延时时间（部分数据引用 Brendan Gregg）。

![图 2-50][image-65]

图 2-51

### 2.11.2 I/O 多路复用模型

Redis 采用 I/O 多路复用技术，并发处理连接。采用了 epoll + 自己实现的简单的事件框架。epoll 中的读、写、关闭、连接都转化成了事件，然后利用 epoll 的多路复用特性，绝不在 IO 上浪费一点时间。

> 65 哥：“那什么是 I/O 多路复用呢？”

在解释 IO 多虑复用之前我们先了解下基本 IO 操作会经历什么。

**基本 IO 模型**

一个基本的网络 IO 模型，当处理 get 请求，会经历以下过程：

1. 和客户端建立建立 `accept`;
2. 从 socket 种读取请求 `recv`;
3. 解析客户端发送的请求 `parse`;
4. 执行 `get` 指令；
5. 响应客户端数据，也就是 向 socket 写回数据。

其中，bind/listen、accept、recv、parse 和 send 属于网络 IO 处理，而 get 属于键值数据操作。既然 Redis 是单线程，那么，最基本的一种实现是在一个线程中依次执行上面说的这些操作。

关键点就是 **accept 和 recv 会出现阻塞**，当 Redis 监听到一个客户端有连接请求，但一直未能成功建立起连接时，会阻塞在 accept() 函数这里，导致其他客户端无法和 Redis 建立连接。

类似的，当 Redis 通过 recv() 从一个客户端读取数据时，如果数据一直没有到达，Redis 也会一直阻塞在 recv()。

![2-52][image-66]

图 2-52

阻塞的原因由于使用传统阻塞 IO ，也就是在执行 read、accept 、recv 等网络操作会一直阻塞等待。

![2-53][image-67]

图 2-53

**I/O 多路复用**

**多路**指的是多个 socket 连接，**复用**指的是复用一个线程。多路复用主要有三种技术：select，poll，epoll。epoll 是最新的也是目前最好的多路复用技术。

**它的基本原理是，内核不是监视应用程序本身的连接，而是监视应用程序的文件描述符。**

当客户端运行时，它将生成具有不同事件类型的套接字。在服务器端，I / O 多路复用程序（I / O 多路复用模块）会将消息放入队列（也就是 下图的 I/O 多路复用程序的 socket 队列），然后通过文件事件分派器将其转发到不同的事件处理器。

简单来说：单线程情况下，内核会一直监听 socket 上的连接请求或者数据请求，一旦有请求到达就交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

select/epoll 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的事件处理器。所以 Redis 一直在处理事件，提升 Redis 的响应性能。

![2-54][image-68]

图 2-54

**Redis 线程不会阻塞在某一个特定的监听或已连接套接字上，也就是说，不会阻塞在某一个特定的客户端请求处理上。**正因为此，Redis 可以同时和多个客户端连接并处理请求，从而提升并发能力。

### 2.11.3 单线程模型

> 65 哥：“为什么 Redis 是单线程的而不用多线程并行执行充分利用 CPU 呢？”

**单线程指的是 Redis 的网络 I/O 以及键值对指令读写是由一个线程来执行的。** 对于 Redis 的持久化、集群数据同步、异步删除等都是其他线程执行。

不过在 6.0 及之后的版本，**Redis 多线程模型**开始支持，**需要注意的是，Redis 多 IO 线程模型只用来处理网络读写请求，对于 Redis 的读写命令，依然是单线程处理**。

至于为啥用单线程，我们先了解多线程有什么缺点。

##### 多线程的弊端

使用多线程，通常可以增加系统吞吐量，充分利用 CPU 资源。

但是，使用多线程后，没有良好的系统设计，可能会出现如下图所示的场景，增加了线程数量，前期吞吐量会增加，再进一步新增线程的时候，系统吞吐量几乎不再新增，甚至会下降！

在运行每个任务之前，CPU 需要知道任务在何处加载并开始运行。也就是说，系统需要帮助它预先设置 CPU 寄存器和程序计数器，这称为 CPU 上下文。

这些保存的上下文存储在系统内核中，并在重新计划任务时再次加载。这样，任务的原始状态将不会受到影响，并且该任务将看起来正在连续运行。

**切换上下文时，我们需要完成一系列工作，这是非常消耗资源的操作。**

另外，当多线程并行修改共享数据的时候，为了保证数据正确，需要加锁机制就会带来额外的性能开销，面临的共享资源的并发访问控制问题。

引入多线程开发，就需要使用同步原语来保护共享资源的并发读写，增加代码复杂度和调试难度。

##### 单线程高性能的原因

1. 不会因为线程创建导致的性能消耗；
2. 避免上下文切换引起的 CPU 消耗，没有多线程切换的开销；
3. 避免了线程之间的竞争问题，比如添加锁、释放锁、死锁等，不需要考虑各种锁问题。
4. 代码更清晰，处理逻辑简单。

> Redis：“你可能会问，单线程是否可以充分利用 CPU 资源呢？”

官方答复。

- 使用 Redis 时，几乎不存在 CPU 成为瓶颈的情况， Redis 主要受限于内存和网络。

- 在一个普通的 Linux 系统上，Redis 通过使用`pipelining` 每秒可以处理 100 万个请求，所以如果应用程序主要使用 O(N) 或 O(log(N)) 的命令，它几乎不会占用太多 CPU。

- 使用了单线程后，可维护性高。多线程模型虽然在某些方面表现优异，但是它却引入了程序执行顺序的不确定性，带来了并发读写的一系列问题，增加了系统复杂度、同时可能存在线程切换、甚至加锁解锁、死锁造成的性能损耗。

antirez 大佬给我设计了 AE 事件模型以及 I/O 多路复用等技术，处理性能非常高，因此没有必要使用多线程。

**单线程机制让 Redis 内部实现的复杂度大大降低，Hash 的惰性 Rehash、Lpush 等等线程不安全的命令都可以无锁进行**。

### 2.11.4 高效的数据结构

> 65 哥：“学习 MySQL 的时候我知道为了提高检索速度使用了 B+ Tree 数据结构，所以 Redis 速度快应该也跟数据结构有关。”

回答正确，这里所说的数据结构并不是 Redis 提供给我们使用的 5 种数据类型 String、List、Hash、Set、SortedSet。

在 Redis 中，常用的 5 种数据类型和应用场景如下：

- **String：** 缓存、计数器、分布式锁等。
- **List：** 链表、队列、微博关注人时间轴列表等。
- **Hash：** 用户信息、Hash 表等。
- **Set：** 去重、赞、踩、共同好友等。
- **SortedSet：** 访问量排行榜、点击量排行榜等。

上面的应该叫做 Redis 支持的数据类型，也就是数据的保存形式。针对这 5 种数据类型，每种数据类型底层不止用了一种数据局结构，目的都是为了在性能和内存之间做平衡。

![2-55][image-69]

图 2-55

### 2.11.5 全局散列表

Redis 整体就是一个散列表来保存所有的键值对，无论数据类型是 5 种的任意一种。散列表，本质就是一个数组，每个元素被叫做哈希桶，不管什么数据类型，每个桶里面的 entry 保存着实际具体值的指针。

![2-56][image-70]

图 2-56

整个数据库就是一个**全局哈希表**，而哈希表的时间复杂度是 O(1)，只需要计算每个键的哈希值，便知道对应的哈希桶位置，定位桶里面的 entry 找到对应数据，这个也是 Redis 快的原因之一。

# 第 3 章 不死之身高可用

## 3.1 宕机恢复，不丢数据稳如山

我是一个基于内存的数据库，名字叫 `Redis`。我对数据读写操作的速度快到令人发指，很多程序员把我当做缓存使用系统，用于提高系统读取响应性能。

然而，快是需要付出代价的：内存无法持久化，一旦断电或者宕机，我保存在内存中的数据将全部丢失。

此时此刻，MySQL 失去了我这道高性能缓存大佬支撑，大量流量会打到 `MySQL`， 可能带来更严重的问题。

MySQL：“你赶紧重启从我这里获取数据加载到内存里呀。”

Redis：“不行呀，如果是大量数据需要恢复，会给你造成更大的压力。“

MySQL：“那怎么办？”

Redis：“别怕，我有两大杀手锏，实现了数据持久化，做到宕机快速恢复，不丢数据稳如狗，避免从数据库中慢慢恢复数据，他们分别是 RDB 快照和 AOF（Append Only File）。“

MySQL 说道：“别墨迹，赶紧开搞吧，我快扛不住了。”

### 3.1.1 RDB 快照

数据存储在内存中，我把内存中的数据写到磁盘上就实现了持久化，当重启的时候就把保存在磁盘的快照数据加载快速恢复到内存中，这样就能实现重启后正常提供服务。

MySQL：“我有一个建议，每次执行写指令操作内存的同时写到磁盘。”

“你的建议很好，下次不要再建议了。

这个方案有个致命问题：每次写指令不仅写内存还写磁盘，磁盘的性能相对内存而言太慢，会导致我的性能大大降低，让我快不起来了。”

> MySQL：那你如何规避这个问题呢？

程序员通常把我当做缓存使用，一致性要求没那么高，我不需要把你所有的数据都保存下来，数据持久化使用了**RDB 内存快照**的方式来实现宕机快速恢复。

我在快速执行大量写指令过程中，内存数据会一直变化。持久化的数据并不需要时刻与内存中数据一致，RDB 内存快照，指的就是 Redis 内存中的某一刻的数据。

好比时间定格在某一刻，当我们拍照时，把某一刻的瞬间画面定格记录下来。

我跟这个类似，就是把某一刻的数据以文件的形式“拍”下来，写到磁盘上。这个文件叫做 RDB 文件，是 Redis Database 的缩写。

我只需要定时执行 RDB 内存快照，就不必每次执行写指令都写磁盘，既实现了快，还实现了持久化。

![RDB内存快照][image-71]

图 3-1

当在进行宕机后重启数据恢复时，直接将磁盘的 RDB 文件读入内存即可。

#### 1. RDB 生成策略

> MySQL：”什么时候触发生成 RDB 快照呢？“

我用的是单线程模型执行读写指令，所以需要尽可能避免阻塞 RDB 文件生成阻塞主线程，先看看有哪些情况会触发 RDB 快照持久化操作。

有两种情况会触发 RDB 持久化。

- 手动触发：执行 `save` 或`bgsave`命令。
- 自动触发：一共有四种情况会自动触发执行 `bgsave`命令生成 RDB 文件，后文细说。

##### 手动触发

我提供了两个指令用于手动生成 RDB 文件。

- save：主线程执行，会阻塞。
- bgsave：调用 glibc 的函数`fork`产生一个子进程用于写入临时 RDB 文件，快照持久化完全交给子进程来处理，完成后自动结束，**父进程可以继续处理客户端请求**，阻塞只发生在 `fork` 阶段，时间很短，生成 RDB 文件的默认配置使用的就是该指令。当子进程写完新的 RDB 文件后，它会替换旧的 RDB 文件。

##### 自动触发

程序员总不能半夜起来手动执行命令生成 RDB，为了让他们安稳的过夜生活，我会在以下 4 种情况自动触发执行 `bgsave`生成 RDB 文件：

- redis.conf 中配置`save m n` ，在 m 秒内至少有 n 个 key 更改，自动触发 `bgsave` 生成 RDB 文件。
- 主从复制，从节点需要从主节点进行全量复制时会触发 `bgsave` 操作，把生成的 RDB 文件发送给从节点。
- 执行 `debug reload`命令重新加载 Redis 会触发 `bgsave` 执行。
- 默认情况下执行 `shutsown` 命令，如果没有开启 AOF 持久化，我也会触发 `bgsave`操作。

如果配置成`save ""`，则表示关闭 RDB 快照功能。聪明的程序员可根据实际请求压力调整快照周期执行策略。

##### 其他配置

我还提供了其他用于控制生成 RDB 文件的配置。

```sh
# 文件名称
dbfilename dump.rdb
# 文件保存路径
dir /opt/app/redis/data/
# 如果持久化出错，主进程是否停止写入
stop-writes-on-bgsave-error yes
# 是否压缩
rdbcompression yes
# 导入时是否检查
rdbchecksum yes

```

**stop-writes-on-bgsave-error**

上边我提到在执行快照生成的过程中，主线程依然可以接收客户端的写指令，是在快照操作正常情况下。

如果生成快照期间出现异常，比如操作系统权限不够，磁盘已满。该配置配置成 yes，我就会禁止执行写操作。

反之，出现快照错误也允许执行写操作。

**rdbcompression**

启用 LZF 压缩算法对字符串类型的数据进行压缩生成 RDB 快照文件，则设置成 `yes`。

**rdbchecksum**

从 Redis 5.0 开始，在 RDB 的末尾会有一个 64 位的 CRC 校验码，用于验证整个 RDB 文件的完整性。**这个功能大概会损失 10% 左右的性能，但是能获得更高的数据可靠性，追求极致性能的程序员可将这个配置成 `no`。**

#### 2. 写时复制

> MySQL：“实际生产环境中，程序员通常给你配置 6GB 的内存，将这么大的内存数据生成 RDB 快照文件落到磁盘的过程会持续比较长的时间。
> 
> 你如何做到继续处理写指令请求，又保证 RDB 与内存中的数据的一致性呢？”

作为唯快不破的 NoSQL 数据库扛把子，我在对内存数据做快照的时候，并不会暂停写操作（读操作不会造成数据的不一致）。

我使用了操作系统的多进程**写时复制技术 COW(Copy On Write)** 来实现快照持久化。

在持久化时我会调用操作系统 `glibc` 函数`fork`产生一个子进程，**快照持久化完全交给子进程来处理，主进程继续处理客户端请求。**

子进程刚刚产生时，它和父进程共享内存里面的代码段和数据段，你可以将父子进程想像成一个连体婴儿，共享身体。

这是 Linux 操作系统的机制，为了节约内存资源，所以尽可能让它们共享起来。在进程分离的一瞬间，内存的增长几乎没有明显变化。

`bgsave` 子进程可以共享主线程的所有内存数据，所以能读取主线程的数据并写入 RDB 文件。

如果主线程对这些数据是读操作，那么主线程和 `bgsave`子进程互不影响。

当主线程要修改某个键值对时，这个数据会把发生变化的数据复制一份，生成副本。

接着，`bgsave` 子进程会把这个副本数据写到 RDB 文件，从而保证了数据一致性。

![写时复制技术保证快照期间数据客修改][image-72]

图 3-2

> MySQL：“在执行快照期间，你崩溃了怎么办？”

数据没有全部写到磁盘中，这次快照操作就不算成功，崩溃恢复的时候只能将上次一完整的 RDB 快照文件作为恢复文件。

> MySQL：“那我建议你每秒执行一次快照，这样宕机最多丢失一秒的数据。”

你的建议很好，下次真的不要再建议了。

这个方法是错误的，bgsave 执行时不阻塞主线程，但是，**如果频繁地执行全量快照，也会带来两方面的开销**：

- 频繁生成 RDB 文件写入磁盘，磁盘压力过大。会出现上一个 RDB 还未执行完，下一个又开始生成的情况，陷入死循环。
- `bgsave`子进程通过主线程 `fork`出来的，虽然创建后不会阻塞主线程，但是 `fork` 本身会阻塞主线程。**内存越大，阻塞时间越长，导致频繁阻塞主线程。**

#### 3. 优缺点

- 优点
  - RDB 采用二进制 + 数据压缩的方式写磁盘，文件体积远小于内存大小，适用于备份和全量复制。
  - RDB 加载恢复数据速度远远快于 AOF 文件。
- 缺点
  - 实时性不够，无法做到秒级持久化。
  - 调用 `bgsave`需要 fork 子进程，子进程属于重量级操作，频繁操作执行成本高。

针对 RDB 不适合实时持久化等问题，我提供 AOF 持久化方式来破解。

### 3.1.2 AOF

AOF （Append Only File）持久化记录的是服务器接收的每个写操作，在服务器启动执行重放还原数据集。

AOF 采用的是写后日志模式，即**先写内存，后写日志**。

![AOF写后日志][image-73]

​ 图 3-3

还有一个叫做**写前日志（Write Ahead Log）相反方式：** 在实际写数据之前，将修改的数据写到日志文件中，故障恢复得以保证。

例如 MySQL Innodb 存储引擎 中的 redo log（重做日志）便是记录修改的数据日志，在实际修改数据前先记录修改日志，再执行修改数据。

在默认情况，我并不会开启 AOF，程序员可以通过配置 `redis.conf` 文件来开启 AOF 持久化。

```sh
# yes 开启AOF持久化，默认是 no
appendonly yes

# AOF持久化的文件名，默认是appendonly.aof
appendfilename "appendonly.aof"

# AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的
dir ./

```

#### 1.日志格式

当我接收到 `set key MageByte` 命令将数据写到内存后， 会按照如下格式写入 AOF 文件。

- `*3`：表示当前指令分为三个部分，每个部分都是 `$ + 数字`开头，紧跟后面是该部分具体的`指令、键、值`。
- `数字`：表示这部分的命令、键、值多占用的字节大小。比如 `$3`表示这部分包含 3 个字节，也就是 `SET` 指令。

![AOF 日志格式][image-74]

​ 图 3-4

#### 2.写回策略

为了提高文件的写入效率，当系统调用 `write` 函数，把数据写入到文件的时候，操作系统通常会将待写入的数据暂存在一个内存缓冲区里面，等到缓冲区的空间被填满或者超过了指定的时限之后，才真正地将缓冲区中的数据写入到磁盘里面。

这种做法虽然提高了效率，但也为写入数据带来了安全问题，因为如果计算机发生停机，那么保存在内存缓冲区里面的写入数据将会丢失。

**为此，系统提供了`fsync`和`fdatasync`两个同步函数，它们可以强制让操作系统立即将缓冲区中的数据写入到硬盘里面，从而确保写入数据的安全性。**

Redis 提供的 AOF 配置项`appendfsync`写回策略直接决定 AOF 持久化功能的效率和安全性。

- **always**：同步写回，写指令执行完毕立马将 `aof_buf`缓冲区中的内容刷写到 AOF 文件。
- **everysec**：每秒写回，写指令执行完，日志只会写到 AOF 文件缓冲区，每隔一秒就把缓冲区内容同步到磁盘。
- **no：** 操作系统控制，写执行执行完毕，把日志写到 AOF 文件内存缓冲区，由操作系统决定何时刷写到磁盘。

没有两全其美的策略，我们需要在性能和可靠性上做一个取舍。

`always`同步写回可以做到数据不丢失，但是每个写指令都需要写入磁盘，性能最差。

`everysec`每秒写回，避免了同步写回的性能开销，发生宕机可能有一秒位写入磁盘的数据丢失，在性能和可靠性之间做了折中。

`no`操作系统控制，执行写指令后就写入 AOF 文件缓冲就可以执行后续的写指令，性能最好，但是有可能丢失很多的数据。

#### 3. AOF 重写瘦身

> MySQL：“随着写入操作的执行，AOF 日志过大怎么办？文件越大，数据恢复恢复就越慢。”

为了解决 AOF 文件体积膨胀的问题，创造我的 antirez 老哥设计了一个杀手锏——AOF 重写机制，对文件进行瘦身。

例如，使用 `INCR counter` 实现一个自增计数器，初始值 1，递增 1000 次的最终目标是 1000，在 AOF 中保存着 1000 次指令。

在重写的时候并不需要其中的 999 个写操作，重写机制有**多变一**功能，将旧日志中的多条指令，重写后就变成了一条指令。

**其原理就是开辟一个子进程将内存中的数据转换成一系列 Redis 的写操作指令，写到一个新的 AOF 日志文件中。再将操作期间发生的增量 AOF 日志追加到这个新的 AOF 日志文件中，追加完毕后就立即替代旧的 AOF 日志文件了，瘦身工作就完成了。**

![AOF重写机制(纠错：3条变一条)][image-75]

​ 图 3-5

我提供了 `bgrewriteaof`指令用于对 AOF 日志进行瘦身。程序员不可能随时随地使用该指令去重写文件，这样的话都没有时间谈恋爱。

所以，我还提供了以下两个配置，实现自动重写策略，解放程序员的双手。

```sh
# 触发重写 AOF 配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

- `auto-aof-rewrite-percentage`：如果当前 AOF 文件的大小超过了上次重写后的 AOF 文件大小的百分比后，则开始重写 AOF。比如例子中设置为 100，当 AOF 文件的大小超过上次 AOF 文件重写后的 1 倍，就执行重写。
- `auto-aof-rewrite-min-size`：表示触发 AOF 文件重写的最小值。如果 AOF 文件大小低于这个值，则不触发重写操作。

**注意的是，程序员手动执行 `bgrewriteaof` 命令并不受这两个条件限制。**

> MySQL：“AOF 重写会阻塞主线程么？”

AOF 重写是通过主线程 fork 出后台 bgrewriteaof 子进程执行，会把主线程的内存拷贝一份给 bgrewriteaof 子进程，子进程就能在不影响主线程的情况下，将内存中的数据生成写操作记录到重写日志。

**因此，在 AOF 重写时，阻塞主线程只发生在主线程 fork 子进程那一刻 。**

##### 重写过程

> MySQL：“在重写日志时，有新数据写入内存怎么办？”

总的来说，重写过程一共出现 **两个日志和一次拷内存数据拷贝。分别是旧的 AOF 日志和新的 AOF 重写日志， Redis 数据拷贝**。

Redis 会将重写过程中的接收到的写操作同时记录到**旧的 AOF 缓冲区和 AOF 重写缓冲区**。

这样新的重写日志也保存最新的操作。等到拷贝数据的所有操作记录重写完成后，重写缓冲区记录的最新操作也会写到新的 AOF 文件中。

每次 AOF 重写时，Redis 会先执行一个内存拷贝，让 bgrewriteaof 子进程拥有此时的 Redis 内存快照，子进程遍历 Redis 中的全部键值对，生成重写记录。

使用两个日志保证在重写过程中，新写入的数据不会丢失，并且保持数据一致性。

![AOF 重写过程][image-76]

​ 图 3-6

> MySQL：”为什么 AOF 重写不复用原 AOF 日志？“

这个问题问得好，有以下两个原因：

1. 一个原因是父子进程写同一个文件必然会产生竞争问题，控制竞争就意味着会影响父进程的性能。
2. 如果 AOF 重写过程中失败了，那么原本的 AOF 文件相当于被污染了，无法做恢复使用。所以 Redis AOF 重写一个新文件，重写失败的话，直接删除这个文件就好了，不会对原先的 AOF 文件产生影响。等重写完成之后，直接替换旧文件即可。

##### Multi-Part AOF 机制

> MySQL：“在 AOF Rewrite 过程中，主进程除了把写指令写到 AOF 缓冲区以外，还要写到 AOF 重写缓冲区中。一份数据要写两个缓冲区，还要写到两个 AOF 文件，产生两次磁盘 I/O ，太浪费了。”

上述的 AOF Rewrite 操作是 Redis 7.0 之前的逻辑，俗话说的好，只要思想不滑坡，办法总比困难多。为了解决性能问题，7.0 版本之后引入了 Multi-Part AOF 机制。

除了这个问题以外，其实还有以下几点性能问题。

1. 开辟 AOF Rewrite 缓冲区，存放 AOF 重写期间的所有日志，在写指令密集的场景下， AOF Rewrite 缓冲区会占据大量的内存。
1. AOF Rewrite 结束后，由主进程把 Rewrite 缓冲区的数据写入磁盘，缓冲区过大会阻塞命令执行，造成 Redis 耗时尖刺。
1. AOF Rewrite 需要主子进程进行复杂的通信，实现逻辑复杂。

Multi-Part AOF 机制就是把原来单个 AOF 文件拆分成多个，并分三种不同类型，不同类型的 AOF 文件由不同的职责。

- Base AOF 文件：子进程执行 AOF Rewrite 时生成的文件，有且只有一个。
- Incr AOF 文件：增量 AOF 文件，在 AOF Rewrite 开始的时候主进程创建，用于保存在 AOF 重写期间收到的写操作，可能存在多个这样的文件。
- History AOF 文件：历史版本的 Base AOF 文件和 Incr AOF 文件，AOF Rewrite 执行完成，原先的 Base AOF 文件和 Incr AOF 文件则被标记成 History 类型。Redis 会自动删除这些类型文件。

当进行 AOF Rewrite 时，Redis 主进程会新建一个 Incr AOF 类型的 AOF 文件，用于保存整个 AOF Rewrite 期间的 AOF 日志，旧的 Incr AOF 文件不再写入。

接着，主进程 fork 出一个子进程，用于执行 AOF Rewrite 操作。子进程会生成一个新的 Base AOF 文件，执行一个内存拷贝，拥有此时的 Redis 内存快照，遍历 Redis 中的全部键值对，生成重写记录，写入到 Base AOF 文件。

新生成的 Base AOF 文件与新建的 Incr AOF 文件结合在一起，就包含了当前 Redis 的所有数据，AOF Rewrite 结束后，主进程会使用一个 manifest 文件，来维护这些 AOF 文件的信息。

其实就是把新生成的 Base AOF 文件与新建的 Incr AOF 文件信息记录，同时把之前的 Base AOF 和 Incr 文件标记成 History。

你会发现，整个 AOF Rewrite 过程中，不再重复写 AOF 文件，也没有使用 AOF Rewrite 缓冲区暂存日志。

![3-7][image-77]

图 3-7

#### 4. AOF 优缺点

**优点**

- 持久化实时性高：在使用 fsync 每秒持久化的写入性能依然很棒。
- 是一种追加日志，不会出现磁盘寻道问题。也不会在断电时出现损坏问题。即使由于某种原因（磁盘已满或其他原因）日志以写一半的命令结束，redis-check-aof 工具也能够轻松修复它。
- 易于理解和解析的格式依次包含所有操作的日志。
- 写操作执行成功才记录日志，避免了指令语法检查开销，同时，不会阻塞当前「写」指令。

**缺点**

- 由于 AOF 记录的是一个个指令内容，故障恢复的时候需要执行每一个指令，如果日志文件太大，整个恢复过程就会非常缓慢。
- 另外文件系统对文件大小也有限制，不能保存过大文件，文件变大，追加效率也会变低。
- 指令执行完成，写日志之前宕机了，会丢失数据。
- **AOF 避免了当前命令的阻塞，但是可能会给下一个命令带来阻塞的风险**。AOF 日志是主线程执行，将日志写入磁盘过程中，如果磁盘压力大就会导致写磁盘很慢，导致后续的写指令阻塞。

> MySQL：“两种持久化方式都有优缺点，可不可以结合下做到更好呢？”

重启 Redis 时，我们很少使用 RDB 来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是重放 AOF 日志性能相对 RDB 来说要慢很多，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。

antirez 在 4.0 版本中给我提供了一个**混合使用 AOF 日志和 RDB 内存快照**的方法。简单来说，**RDB 内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有写操作。**

如此一来，快照就不需要频繁执行，避免了 fork 对主线程的性能影响，AOF 不再是全量日志，而是生成 RDB 快照时间的增量 AOF 日志，这个日志就会很小，都不需要重写了。

等到，第二次做 RDB 全量快照，就可以清空旧的 AOF 日志，恢复数据的时候就不需要使用 AOF 日志了。

> MySQL：“RDB 和 AOF 持久化搞定了，如何从这些持久化文件中恢复数据呢？”

如果一台服务器上既有 RDB 文件，又有 AOF 文件，当我重新启动的时候，将优先选择 AOF 文件来恢复数据，因为它能保证数据更完整。

如果 AOF 文件不存在，则去加载 RDB 文件。恢复流程图 3-7 所示。

![持久化文件恢复数据][image-78]

​ 图 3-7

## 3.2 主从复制架构

当我发生宕机时，可以通过重新读取 RDB 内存快照文件和执行 AOF 日志实现快速恢复的高可用手段。

**高可用有两个含义：一是数据尽量不丢失，二是服务尽可能提供服务。** AOF 和 RDB 内存快照保证了数据持久化尽量不丢失，而主从复制就是增加副本，一份数据保存到多个实例上，即使有一个实例宕机，其他实例依然可以提供服务。

本章节主要带你全方位吃透 **Redis 高可用技术解决方案之一主从复制架构**。

李老师：“有了 RDB 内存快照和 AOF 再也不怕宕机丢失数据了，但是 Redis 实例宕机了办？如何实现高可用呢？“

李老师愣了一会儿，又赶紧补充道：“依然记得那晚我和我的恋人 Chaya 鸳语轻传，香风急促，朱唇紧贴。香肌如雪，罗裳慢解春光泄。含香玉体说温存，多少风和月。今宵鱼水和谐，抖颤颤，春潮难歇。千声呢喃，百声喘吁，数番愉悦。”

可是这时候 Redis 忽然宕机了，无法对外提供服务，电话连环 call，岂不是折煞人也。

Redis：“你还念上诗歌了，莫怕，为了你们的幸福。我提供了主从模式，通过主从复制，将数据冗余一份复制到其他 Redis 服务器，实现高可用。你们放心的说温存，说风月。”

既然一台宕机了无法提供服务，那多台呢？是不是就可以解决了。

前者称为 mater (master)，后者称为 slave (slave)，数据的复制是单向的，只能由 mater 到 slave。

默认情况下，每台 Redis 服务器都是 mater；且一个 mater 可以有多个 slave (或没有 slave)，但一个 slave 只能有一个 mater。

Chaya：“主 slave 之间的数据如何保证一致性呢？”

- 读操作：主、slave 都可以执行。
- 写操作：mater 先执行，之后将写操作同步到 slave。

为了保证副本数据的一致性，主从架构采用了读写分离的方式，mater 在执行修改操作的时候，会把相应的写命令近乎实时地同步给 slave，slave 回放这些命令，就可以保证自己的主从数据保持一致。

![3-1][image-79]

图 3-8

> Chaya：“主 slave 都可以执行写指令不是更好么？”

我们可以假设主 slave 都可以执行写指令，假如对同一份数据分别修改了多次，每次修改请求发送到不同的主从实例上，就导致实例的副本数据不一致。

如果为了保证数据一致，Redis 需要加锁，协调多个实例的修改，Redis 自然不会这么干！

> Chaya：“主从复制还有其他作用么？”

1. 故障恢复：当 mater 宕机，其他节点依然可以提供服务。
2. 负载均衡：Master 节点提供写服务，Slave 节点提供读服务，分担压力。
3. 高可用基石：是哨兵和 Cluster 实施的基础，是高可用的基石。

### 3.2.1 主从数据同步原理

> Chaya：“主 slave 同步是如何完成的呢？mater 数据是一次性传给 slave，还是分批同步？主从正常运行期间中又怎么同步呢？要是主 slave 间的网络断连了，重新连接后数据还能保持一致吗？”

你咋问题这么多，不要急。我知道你想安心的与爱人相会，不受 Redis 宕机导致服务报警的干扰。主从数据同步分为四种情况：

1. 第一次主 slave 全量同步。
2. 主 slave 正常运行期间的数据同步。
3. 主 slave 网络断开重连同步。
4. 无盘复制。

在介绍实现原理之前，先看下如何配置主从复制，每个配置的具体解释，未来章节会解释。

```bash
# 建立主从关系命令，设置该节点为其它节点的 slave。
replicaof <masterip> <masterport>
# slave 只读。
replica-read-only yes

# 积压缓冲区大小。缓冲区在 master 与 slave 断线重连后，如果是增量复制，master 就从缓冲区里取出数据复制给 slave。
repl-backlog-size 128mb
```

#### 1. 全量同步

> Chaya：“先从主从实例第一次同步说起吧。”

**主从库第一次复制过程大体可以分为 3 个阶段：连接建立阶段（即准备阶段）、mater 同步数据到 slave 阶段、发送同步期间接受到的新写命令到 slave 阶段**。

直接上图，从整体上有一个全局观的感知，后面具体介绍。

![Redis全量同步][image-80]

图 3-9

##### 连接建立

**第一阶段**

该阶段的主要作用是在主 slave 之间建立连接，相互认识下，为数据全量同步做好准备。

建立信任，才能开始同步，就好像 Chaya 你跟你的爱人建立信任才会牵手接吻说风月。

**slave 会和 mater 建立连接，slave 启动后根据配置发送 psync 命令并告诉 mater 即将进行同步，主库确认回复后，主从库间就进入下一阶段开始同步了**。

> Chaya：“slave 怎么知道 mater 信息并建立连接的呢？”

在 slave 的配置文件中的 replicaof 配置项中配置了 mater 的 IP 和 port 后，slave 就知道自己要和那个 mater 进行连接了。

slave 内部维护了两个字段，masterhost 和 masterport，用于存储 mater 的 IP 和 port 信息。

从库发送的 `psync` 命令包含了**主库的 replid** 和 **复制进度 offset** 两个参数。

```bash
PSYNC <runID> <offset>
```

- **runID**：每个 Redis 实例启动都会自动生成一个唯一标识 ID，第一次主从复制，slave 还不知道主库 runID，所以 runID 会设置成 “?”。
- **repl\_offset**：记录当前复制进度偏移量，slave 记录的偏移量与 master 记录的偏移量之间的数据差，就是需要复制的增量数据。第一次主从复制，repl\_offset 设置成 -1 表示全量复制。

master 收到 slave 的 PSYNC 命令后，一看是 “?” 和 “-1”，表示第一次要进行全量复制，并向 slave 回复 `+FULLRESYNC <runID> <repl_offset>`。

##### 发送 RDB 文件给 slave

**第二阶段**

这个阶犹如 Chaya 你跟你的爱人建立了信任和掌握双方信息之后，就会进入热恋，你们之间会通过赠送礼物、接吻、贴贴把爱意传输到对方。

Redis master 执行 `bgsave` 命令生成 RDB 内存快照文件，这个就是 master 对 slave 的 “爱意”，并将文件发送给 slave。

slave 收到 RDB 内存快照文件互保存到磁盘，并清空当前数据库的数据，再加载 RDB 文件数据到内存中，同时会把 master 的 runID 和 master\_offset 记录下来。

> Chaya：“我的网络是万兆专线，能不能直接传输，不使用磁盘作为中间存储了？”

你真 6，确实可以。通常全量同步需要在磁盘上创建 RDB 快照文件，把文件传输到 slave 之后保存到磁盘再重新加载到内存。

如果磁盘比较慢，对于 master 服务器来说可能是一个压力很大的操作，Redis 2.8.18 版本是第一个支持无盘复制的版本。如果网络快到飞起，那确实可以开启无盘复制的方式，master 直接通过网络将 RDB 数据发送到副本，而不将其作为中间步骤写入磁盘。

无磁盘复制不仅提高了性能，还通过在完全重新同步期间消除了写入和读取 RDB 文件到磁盘的需求，简化了复制过程。

##### 发送同步期间接受到的新写命令到 slave

**第三阶段**

master 为每一个 slave 开辟一块 `replication buffer` 缓冲区。

> Chaya：“这个缓冲区有啥用？”

RDB 快照文件只是某一时刻的内存快照，之后 master 接收到的写指令，没有传输到 slave，所以 master 为每一个 slave 开辟一块 `replication buffer` 缓冲区记录复制积压缓冲区记录从生成 RDB 文件开始收到的所有写命令。

slave 加载 RDB 完成后，master 把 `replication buffer` 缓冲区的数据发送到 slave，slave 接收并执行，主从数据就保持一致。

我会为每一个连接到 master 的 slave 开辟一个`replication buffer` 缓冲区，因为每个 slave 开始同步的时刻可能不一样，所以要分别设置一个缓冲区。

**只要 slave 和 master 建立好连接，对应的缓冲区就会创建，断开连接，这个缓冲区就会释放。**

一个在 master 端上创建的缓冲区，存放的数据是下面三个时间内所有的 master 数据写操作。

1）master 执行 bgsave 产生 RDB 的期间的写操作；

2）master 发送 rdb 到 slave 网络传输期间的写操作；

3）slave load rdb 文件把数据恢复到内存的期间的写操作。

Redis 和客户端通信也好，和从库通信也好，Redis 都分配一个内存 buffer 进行数据交互，客户端就是一个 client，从库也是一个 client，我们每个 client 连上 Redis 后，Redis 都会分配一个专有 client buffer，所有数据交互都是通过这个 buffer 进行的。

Master 先把数据写到这个 buffer 中，然后再通过网络发送出去，这样就完成了数据交互。

不管是主从在增量同步还是全量同步时，master 会为其分配一个 buffer ，只不过这个 buffer 专门用来**传播写命令**到从库，保证主从数据一致，我们通常把它叫做 `replication buffer`。

#### 2. 增量同步

> Chaya：“主 slave 的网络断开重连了咋办？要重新进行全量复制么？”

其实在上面整个过程完成之后，全量复制就完成了，只要连接不中断，就会持续进行基于长连接的命令传播复制。

在 Redis 2.8 之前，如果主从复制在命令传播时出现了网络闪断，那么，slave 就会和 mater 重新进行一次全量复制，开销非常大。

Redis：“Chaya 你跟你的爱人偶尔吵架闹小矛盾，只是一时断开联系，不可能完全忘了对方说，分手吧。气头过了之后依然还会接吻表达爱意对吧。“

从 Redis 2.8 开始，我也做了优化，网络断了重连之后，slave 会尝试采用增量复制的方式继续同步。

增量复制：**用于网络中断等情况后的复制，只将中断期间 mater 执行的写命令发送给 slave，与全量复制相比更加高效**。

**repl\_backlog\_buffer**

断开重连增量复制的实现奥秘就是 `repl_backlog_buffer` 缓冲区，是一个定长的环形数组，如果数组内容满了，就会从头开始覆盖前面的内容。不管在什么时候 master 都会将写指令操作记录在 repl\_backlog\_buffer 中，它记录 Master 接收新的写请求数据的偏移量和新写命令，这样 Slave 再重新连接之后，就可以从这里获取未同步的命令发送给 Slave 了。

master 使用 `master_repl_offset`记录自己写到的位置偏移量，slave 则使用 `slave_repl_offset`记录已经读取到的偏移量。

master 收到写操作，偏移量则会增加。slave 持续执行同步的写指令后， `repl_backlog_buffer` 的已复制的偏移量 slave\_repl\_offset 也在不断增加。

正常情况下，这两个偏移量基本相等，在网络断连阶段，mater 可能会收到新的写操作命令，所以 `master_repl_offset`会大于 `slave_repl_offset`。

![图 3-3][image-81]

图 3-10

当主从断开重连后，slave 会先发送 psync 命令给 master，同时将自己之前保存的 master `runID`，`slave_repl_offset`发送给 master。

master 只需要把 `master_repl_offset`与 `slave_repl_offset`之间的差异命令同步给从库即可。

增量复制执行流程如下图：

![图 3-4][image-82]

图 3-11

需要注意的是，**只要进行主从复制，master 接收到的写操作在写入 replication buffer 的同时，也会写入到 repl\_backlog 的缓冲区内。**

- replication buffer，是主 slave 建立连接后创建的，主从断开后，master 就会删除该 buffer，主从之间复制命令的传输都会经过这个 buffer，每个 slave 节点独有自己的专属一个。
- repl\_backlog， 是一个环形缓冲区，整个 master 进程中只会存在一个，所有的 slave 公用。repl\_backlog 的大小通过 repl-backlog-size 参数设置，默认大小是 1M，其大小可以根据每秒产生的命令、（master 执行 rdb bgsave） +（ master 发送 rdb 到 slave） + （slave load rdb 文件）时间之和来估算积压缓冲区的大小，repl-backlog-size 值不小于这两者的乘积。

总的来说，`replication buffer` 是主从库在进行全量复制时，主库上用于和从库连接的客户端的 buffer，master 会为每一个 client 开辟一块，命令都通过这个来传输。

而 `repl_backlog_buffer` 是为了支持从库增量复制，主库上用于持续保存写操作的一块专用 buffer。

**`repl_backlog_buffer`在 Redis 服务器启动后，开始一直接收写操作命令，这是所有从库共享的。**master 和 slave 会各自记录自己的复制进度，所以，不同的 slave 在进行恢复时，会把自己的复制进度（`slave_repl_offset`）发给 master，master 就可以和它独立同步。

![图 3-5][image-83]

图 3-12

> Chaya：“repl\_backlog\_buffer 太小的话从库还没读取到就被 Master 的新写操作覆盖了咋办？”

一旦被覆盖就会执行全量复制。我们可以调整 repl\_backlog\_size 这个参数用于控制缓冲区大小，计算公式如下。

```bash
repl_backlog_buffer = second * write_size_per_second
```

1. **second**：从服务器断开重连主服务器所需的平均时间；
2. **write\_size\_per\_second**：master 平均每秒产生的命令数据量大小（写命令和数据大小总和）；

例如，如果 mater 服务器平均每秒产生 1 MB 的写数据，而 slave 断线之后平均要 5 秒才能重新连接上主服务器，那么复制积压缓冲区的大小就不能低于 5 MB。

为了安全起见，可以将复制积压缓冲区的大小设为`2 * second * write_size_per_second`，这样可以保证绝大部分断线情况都能用部分重同步来处理。

#### 3. 正常运行期间的同步

> Chaya：“完成全量同步后，正常运行期间主从如何同步呢？”

当主从完成了全量复制，它们之间就会一直维护一个网络连接，master 会通过这个连接将后续陆续收到的命令操作再传播给 slave，这个过程也称为基于长连接的命令传播，使用长连接的目的就是避免频繁建立连接导致的开销。

#### 4. 缓冲区演化

> Chaya：“我发现个问题， 不管是全量复制还是增量复制，当写请求到达 master 时，指令会分别写入所有 slave 的 replication buffer 以及 repl\_backlog\_buffer。重复保存，太浪费内存了。”

确实，master 为每个 slave 开辟的 replication buffer，他们存储的内容却是一样的。此外，repl\_backlog 的内容也与 slave 的 replication buffer 有重复。

![][image-84]

图 3-13

所以，在 7.0 版本中，我对上述问题进行了相关优化，采用了**共享缓冲区**的设计。

既然存储内容是一样，所以本着勤俭持家原则，最直观的想法就是主从复制在命令传播时，**将这些写命令放在一个全局的复制缓冲区中，多个 slave 共享这份数据，不同 slave 引用缓冲区的不同内容，这就是共享缓冲区的核心思想。**

共享缓冲区方案是将 ReplicationBuffer 数据切割成多个 16KB 的数据块（**replBufBlock**），并使用 redisServer.repl\_buffer\_blocks 链表将他们维护起来。

新的写指令会写到队尾的 replBufBlock 中，满了后就创建新的，缓冲区改良后，如下图所示。

![3-14][image-85]

图 3-14

replBufBlock 定义如下源码所示。

```c
typedef struct replBufBlock {
    int refcount;
    long long id;
    long long repl_offset;
    size_t size, used;
    char buf[];
} replBufBlock;
```

- `refcount`，当前 replBufBlock 被引用次数，当降为 0 的时候表示可以回收。
- `id`，block 的唯一标识，单调递增。
- `repl_offset`，记录 buf 第一个字节对应的 offset，表示从该块的哪个位置开始向副本发送数据。
- `size` 和 `used`：这些字段描述缓冲块的总大小和已使用的空间量。
- `buf[]`：这是缓冲块内部的实际数据存储，用于保存要发送到副本的复制数据。

master 向 slave 传播命令的时候，master 可直接从 redisServer.repl\_buffer\_blocks 链表定位到需要传输给 slave 的 replBufBlock，接着让 slave 的 `client->ref_repl_buf_node` 指针指向这个 replBufBlock 实例，将这块命令传播给 slave，避免为每个 slave 创建一块缓冲区，存储重复的内容。

除此之外，repl\_backlog\_buffer 复用了 redisServer.repl\_buffer\_blocks 链表的数据，epl\_backlog\_buffer 中有一个 blocks\_index 字段中维护了一个 rax 树，它的 Key 是 replBufBlock 的起始 repl\_offset，Value 指向相应的 replBufBlock 实例。

![3-15][image-86]

图 3-15

#### 5.如何确定执行全量同步还是部分同步？

在 Redis 2.8 及以后，slave 可以发送 psync 命令请求同步数据，此时根据主 slave 当前状态的不同，同步方式可能是**全量复制**或**部分复制**。本文以 Redis 2.8 及之后的版本为例。

![图 3-6][image-87]

图 3-16

1. slave 根据当前状态，发送 `psync`命令给 master：
   2. 如果 slave 从未执行过 `replicaof` ，则 slave 发送 `psync ? -1`，向 mater 发送全量复制请求；
   3. 如果 slave 之前执行过 `replicaof` 则发送 `psync <runID> <offset>`, runID 是上次复制保存的 mater runID，offset 是上次复制截至时 slave 保存的复制偏移量。
2. mater 根据接受到的`psync`命令和当前服务器状态，决定执行全量复制还是部分复制：
   5. master runID 与 slave 发送的 runID 相同，且 slave 发送的 `slave_repl_offset`之后的数据在 `repl_backlog_buffer`缓冲区中都存在，则回复 `CONTINUE`，表示将进行部分复制，slave 等待 mater 发送其缺少的数据即可；
   6. mater runID 与 slave 发送的 runID 不同，或者 slave 发送的 slave\_repl\_offset 之后的数据已不在 mater 的 `repl_backlog_buffer`缓冲区中 (在队列中被挤出了)，则回复 slave `FULLRESYNC <runid> <offset>`，表示要进行全量复制，其中 runID 表示 mater 当前的 runID，offset 表示 mater 当前的 offset，slave 保存这两个值，以备使用。

一个 slave 节点如果和 master 断连时间过长，导致 master `repl_backlog_buffer` 的 slave\_repl\_offset 位置上的数据已经被覆盖掉了，此时 master 和 slave 将进行全量复制。

### 3.2.2 主从同步的缺点

#### 1. 主从复制无限循环

**replication buffer 太小会引发的问题**：

replication buffer 由 client-output-buffer-limit slave 设置，当这个值太小会导致**主从复制连接断开**。

1）当 master-slave 复制连接断开，master 会释放连接相关的数据。replication buffer 中的数据也就丢失了，此时主从之间重新开始复制过程。

2）还有个更严重的问题，**主从复制连接断开，导致主从上出现重新执行 bgsave 和 rdb 重传操作无限循环。**

当 mater 数据量较大，或者主 slave 之间网络延迟较大时，可能导致该缓冲区的大小超过了限制，此时 mater 会断开与 slave 之间的连接。

这种情况可能引起全量复制 -\> replication buffer 溢出导致连接中断 -\> 重连 -\> 全量复制 -\> replication buffer 缓冲区溢出导致连接中断……的循环。

#### 2. 内存过大

如果 Redis 单机内存达到 10GB，一个 slave 的同步时间在几分钟的级别；如果 slave 较多，恢复的速度会更慢。如果系统的读负载很高，而这段时间 slave 无法提供服务，会对系统造成很大的压力。

如果数据量过大，全量复制阶段 mater fork + 保存 RDB 文件耗时过大，slave 长时间接收不到数据触发超时，主 slave 的数据同步同样可能陷入**全量复制-\>超时导致复制中断-\>重连-\>全量复制-\>超时导致复制中断**……的循环。

此外，mater 单机内存除了绝对量不能太大，其占用主机内存的比例也不应过大：最好只使用 50% - 65% 的内存，留下 30%-45% 的内存用于执行 bgsave 命令和创建复制缓冲区等。

#### 3. 主从不一致问题

> Chaya：“主从模式下，slave 可以执行客户端写请求么？”

Redis：“我的建议是将 slave 配置成 read-only 模式，也就是只读。不然就有可能导致主从数据不一致。”

此外，一般还会给 slave 添加 `eplica-ignore-maxmemory no` 配置，不让 slave 执行内存淘汰的操作，而是由 mater 来决定是否进行淘汰，并发送 DEL 指令给 slave。

## 3.3 哨兵集群

我叫 Redis，通过之前的学习，你已知道 Redis 主从复制是高可用的基石，某个 Slave 宕机依然可以将请求发送给 Mater 或者其他 Slave，但是 Master 宕机，只能响应读操作，写请求无法再执行。

所以主从复制架构面临一个严峻问题，**master 挂了，无法执行写操作，无法自动选择一个 Slave 切换为 Master**，也就是无法**故障自动切换**。

> 李老师：“还记得那晚与我女友 Chaya 约会，眼前是橡树的绿叶，白色的竹篱笆。好想告诉我的她，这里像幅画。一起手牵手么么哒……（此处省略 10000 字）。
> 
> Redis 忽然宕机，我总不能推开 Chaya，停止甜蜜，然后打开电脑手工进行主从切换，再通知其他程序员把地址重新改成新 Master 信息上线？”。

Redis：“如此一折腾恐怕李老师已被 Chaya 切换成前男友了，心里的雨倾盆的下，万万使不得。所以必须有一个高可用的方案，为此，我提供一个高可用方案——**哨兵（Sentinel）**“。

> 吃瓜群众：“Redis 大佬，虽然我没有女朋友，但是，未雨绸缪我要掌握这个哨兵模式，防止当我与女朋友约会被打扰，你快说说什么事哨兵以及哨兵的实现原理吧。”

先来看看哨兵是什么？搭建哨兵集群的方法我就不细说了，假设三个哨兵组成一个哨兵集群，三个数据节点构成一个一主两从的 Redis 主从架构。

![3-17][image-88]

图 3-17

Redis 哨兵集群高可用方法，有三种角色，分别是 `master`，`slave`，`sentinel`。

- setinel 节点之间互相通信，组成一个集群视线哨兵高可用，选举出一个 leader 执行故障转移。
- master 与 slave 之间通信，组成主从复制架构。
- sentinel 与 master/ slave 通信，是为了对该主从复制架构进行管理：**监视（Monitoring）**、**通知（Notification）**、**自动故障切换（Automatic Failover）**、**配置提供者（Configuration Provider）**。

```bash
# sentinel.conf
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2
```

sentinel 监控的 master 的名字叫做 mymaster，master 的 ip 是 127.0.0.1，端口是 6379。

`quorum` 的作用，这是关键参数。

- 它指定了在标记 master 故障并尝试执行故障切换时需要达成一致意见的 Sentinel 进程数量。大白话就是需要多少个 Sentinel 进程认为 master 宕机，真正标记 master 宕机障才能启动故障切换过程。

- 多个 sentinel ，需要选出一个 leader 来执行实际的故障自动转移，某个哨兵超过 quorum 的投票，那么就选举这个哨兵为 leader，负责自动故障转移。quorum 的值一般是 sentinel 个数一半以上 (n/2 + 1) 比较合理。

sentinel 只要配置 redis master 信息即可与三个角色建立联系。

> Chaya：“为啥哨兵只需要配置 master 信息就可以与三个角色建立联系？”

- sentinel 可以通过 master 获取 slave 的的信息，并与 slave 建立连接。因为，master 与 slave 是主从关系，通过 `info`命令 就可以 通过 master 获取 slave 的 ip 和 port 、runid 等信息。
- 通过上面的步骤，sentinel 与 master 和所有的 slave 建立链接， sentinel 之间的互相感知则是利用了 redis pub/sub 发布订阅机制实现。每个 sentinel 通过发布订阅 master 的 `__sentinel__:hello` 频道进行发布和接收信息，来感知每个 sentinel 的存在并建立连接。

### 3.3.1 哨兵的任务

哨兵是 Redis 的一种运行模式，它专注于**对 Redis 实例（主节点、从节点）运行状态的监控，并能够在主节点发生故障时通过一系列的机制实现选主及主从切换，实现自动故障转移，确保整个 Redis 系统的可用性**。

李老师可以安心的与你的爱人 Chaya 在欢乐港湾约会，靠在她的身后双手贴合玩夹娃娃机，尽情享受甜蜜，哪怕是吵架都那么醉人，不再需要担心 Redis 集群忽然宕机带来的烦恼。

我们先从全局观看哨兵，简要的了解整个运作流程，接着再针对每一个任务详细分析，Redis 哨兵主要职责如下。

- **监控（Monitoring）**：Redis Sentinel 不断检查 master 和 slave 实例是否按预期工作。它监视实例的健康状态，包括 master 和所有 slave。
- **自动故障切换（Automatic Failover）**：如果 master 出现故障或不按预期工作，Redis Sentinel 启动自动故障切换流程。在此过程中，一个 slave 会被晋升为新的 master。
- **通知（Notification）**：让 slave 执行 replicaof 与新的 master 同步数据；并且通知客户端与新 master 建立连接。
- **配置提供者（Configuration Provider）**：Redis Sentinel 充当了客户端服务发现的权威来源。客户端连接到 Sentinels 以获取与当前 Redis master 的地址，以确保客户端能够连接到正确的实例。

![3-18][image-89]

图 3-18

哨兵也是一个 Redis 进程，只是不对外提供读写服务，通常哨兵要配置成单数，为啥呢？且听我慢慢分析。

#### 1. 监控（Monitoring）

> Redis：“ 八卦下李老师用什么方式来了解你的爱人 Chaya 每天的喜怒哀乐呢？”

李老师：“这很简单，每天微信、打电话或者视频跟她说情话沟通了解她的呀，若是哪天不接电话，或者微信发送消息出现红色感叹号，说明她生气把我拉黑了。”

sentinel 与各个角色节点建立链接以后，则是通过 `PING` 、`INFO`、`PUBLISH / SUBSCRIBE`指令来监控所有实例的健康状态，当然不会说情话。

sentinel 默认会以**每秒一次**的频率向所有的 master、slave、sentinel 节点发送 `PING` 命令，这个其实是一个心跳检测，用于探测实例是否存活。

- `PING`，所有节点之间通过发送 `PING`指令作为心跳，确认对方是否在线，默认每秒发送一次。
- `INFO`，sentinel 向 master、slave 发送该命令，用于获取 slave 节点的详细信息。
- `PUBLISH / SUBSCRIBE`，sentinel 会订阅 master 节点和 slave 节点的 `__sentinel__:hello` 频道，并通过该频道发布自己的信息，这样其他 sentinel 之间就客户以建立联系。

如果一个 master 实例距离最后一次有效回复 `PING` 命令的时间超过 `down-after-milliseconds` 选项所指定的值，这个 master 就会被 sentinel 标记为“主观下线”。

如果 slave 没有在指定时间内响应 sentinel 的 `PING`命令，直接标记为 “主观下线”即可。

**只有当大于等于法定个数（quorum）的 sentinel 节点认为该 master 主观下线，那么才能将该 master 改为客观下线。接着才会开启故障自动故障切换流程。**

`PING`命令的回复有两种情况。

1. 有效回复：返回 `+PONG`、`-LOADING`、-`MASTERDOWN` 任何一种。
2. 无效回复：有效回复之外的回复，或者不在指定时间内返回任何回复。

> Chaya：“主观下线和客观下线的作用是什么？”

主要是为了避免出现 sentinel 误判 master 的运行情况，一旦出现误判，就会出现 master 实际没有下线，可是哨兵误以为已经下线的情况，接着就会启动主从故障切换流程，之后的选主和通知操作都会消耗大量资源。

误判一般会发生在集群网络压力较大、网络拥塞或者是 master 本身压力较大的情况下。

**既然一台 sentinel 容易误判，那就多个 sentinel 一起投票判断。哨兵机制也是类似的，采用多实例组成的集群模式进行部署**，这就是**哨兵集群**。引入多个哨兵实例一起来判断，就可以避免单个哨兵因为自身网络状况不好，而误判主库下线的情况。

同时，多个哨兵的网络同时不稳定的概率较小，由它们一起做决策，误判率也能降低。

##### 主观下线

所谓主观下线（Subjectively Down， 简称 SDOWN），表示一个 sentinel 认为一个 redis 实例已经不可用或者已下线，有可能是网络不通、心跳超时、连接失败等原因。

比如 master 或者 slave 在 down-after-milliseconds 指定的毫秒数之内， 没有向 sentinel 发送的 PING 命令的回复， 或者返回一个错误， 那么 sentinel 将这个服务器标记为主观下线（SDOWN ）。

需要注意的是，Redis sentinel 的主要目标是确保 master 的高可用性，而不是 slave 的高可用性。因此，主观下线和客观下线的主要关注点通常是 master。slave 通常不会被单独标记为客观下线，因为它们不承担 master 的关键角色，它们的主要责任是复制数据。

##### 客观下线

**判断 master 是否下线不能只有一个哨兵说了算，只有过半的哨兵判断 master 主观下线，才能将 master 标记为客观下线**。

![3-19][image-90]

图 3-19

之前提过的`sentinel monitor <master-name> <ip> <redis-port> <quorum>` 配置，quorum 这个参数是判断客观下线的一个依据，意思是至少有 quorum 个 sentinel 主观的判定这个 master 是主观下线，才会对这个 master 标记为客观下线。

只有 master 被判定为**客观下线**，才会进一步触发哨兵开始主从切换流程。

#### 2. 自动故障切换（Automatic Failover）

> Chaya：“一旦判断 master 客观下线，那就要 slave 中选一个作为新的 master 了吧？“。

哨兵的第二个任务，选择一个 slave 提升为新的 master，并对外提供服务。之后其他 slave 节点会与新的 master 进行主从复制，这个过程就叫做“**自动故障转移**”。

> 吃瓜群众：“如何从众多 slave 中选出一个做 master 呢？”

Chaya：“我觉得筛选过程就像谈恋爱找对象，每个人心中都会有标尺，会通过直觉、习惯以及标准从所有的追求者中选择一个最优的适合自己的。就如李老师会有自己擅长的领域及发光点，也是会深懂大众的心声。"

李老师：“类似的，Redis 有自己的筛选规则，按照一定的**筛选条件 + 打分策略**，选出一个最强王者担任 master。“

![图 3-20][image-91]

图 3-20

##### 筛选条件

> Chaya：“有哪些筛选条件？”

1. 下线或网络断连的 slave 直接丢弃。
2. 网络无异常：slave 最后一次响应 `PING` 命令的时间不能超过 5 倍 `PING` 周期；slave INFO（每 10s 发送一次 `INFO` 命令） 信息更新时间不能超过 3 倍 INFO 刷新周期。
3. 评估过往的网络状态：slave 与 master 断开连接，断连时间不能超过 （现在 - master 被标记为下线的时间） + （master 的 down-after-milliseconds \* 10）。

总之，下线或者网络经常断开的 slave 不能要。你想呀，即使变成新 master，可是很快网络出了故障，又得重新选择新 master，这不闹着玩么，得排除掉！

##### 打分

过滤掉不合适的 slave 之后，使用快速排序从 slave 列表进行打分，按照以下排序找出最强王者。

1. slave 优先级，通过 `replica-priority 100` 配置项，给不同的 slave 设置不同优先级，默认是 100，值越低，优先级越高，设置成特殊值 0 表示不会晋升为 master。
2. 更大复制偏移量（processed replication offset），已复制的数据量越多越好，`slave_repl_offset`与 `master_repl_offset` 差值越小。
3. slave runID，在优先级和复制进度都相同的情况下，runID 最小的 slave 得分最高，会被选为新主库。

sentinel 给筛选出来的最强王者 slave 发送 `slave no one` 命令，使得该 slave 成为 master 角色，sentinel 并不关心命令返回的结果，它会发送 `info` 命令给 slave，并根据命令的回复内容，确认 slave 是否成功转换为 master。

> Chaya：“旧 master 重新恢复正常的话要怎么处理？”

既然已经错过，相逢也只能过客。原 master 恢复正常，重新连接 sentinel，这时候集群已经有新的 master，所以旧 master，被 sentinel 降级为 slave。

#### 3. 通知（Notification）

新 master 出现后，哨兵还有一件重要的事情要做，将新 master 的连接信息发送给其他 slave ，通知 slave 执行 replacaof 命令和新 master 建立连接进行主从复制。

接着，sentinel 会定时给 slave 发 `INFO` 命令，从 `INFO`命令的回复内容来确认 slave 是否与新 master 成功建立连接。检测到所有 slave 全部与新 master 建立连接，自动故障转移就完成了，

如果还有剩余 slave 没有连上新的 master，sentinel 还会再做一次努力，对这些 slave 再次发送 `slave` 命令。

除此之外，sentinel 还需要把新 master 的信息通知所有客户端，客户端才能正确的读写请求发到新的 master 上。

#### 4. 配置提供者（Configuration Provider）

Redis 客户端只需要跟 sentinel 打交道，就可以无感知的连接到新 master，最重要的原因是哨兵提供了一些 API 来检查主从节点的运行状况、并且可以特定通知以及在运行时更改 sentinel 配置。

默认情况下，Sentinel 使用 TCP 端口 26379 运行（请注意，6379 是普通的 Redis 端口）。Sentinel 使用 Redis 协议接受命令，你可以使用`redis-cli`或任何其他未经修改的 Redis 客户端来与 Sentinel 进行通信。

### 3.3.2 哨兵集群原理

只有一个 sentinel 的话，会存在单点故障问题。Redis Sentinel 是一个分布式系统，有多个 Sentinel 一起协作组成集群实现高可用。

1. 当多个 sentinel 达成一致认为某个 master 不可用，才执行故障转移，降低了误报的概率。
2. 不需要所有 sentinel 都可用，sentinel 集群依然可以正常工作。

> Chaya：“sentinel 是如何感知到其他 sentinel 节点呢？又如何知道 slave 节点的信息并监控呢？master 不可用时，到底哪个 sentinel 来执行故障自动切换呢？”

李老师：“小朋友，你是否有很多问号。为什么别人在那看漫画，我却在学画画对着钢琴说话……”。

Chaya：“……”。

#### 1. Pub/Sub 发布订阅

##### sentinel 互相发现

sentinel 之间互相感知发现，归功于 Redis 的 Pub/Sub 发布订阅机制。当 sentinel 与 master 建立连接后，使用 Pub/Sub 发布订阅机制在特殊的频道发布自己的信息，比如 IP 和端口。同时还会订阅该频道获取其他 sentinel 发布的消息。

master 有一个 `__sentinel__:hello` 的专用通道，用于 sentinel 之间发布和订阅消息。

**这就好比是 `__sentinel__:hello` 微信群，哨兵利用 master 建立的微信群发布自己的消息，同时关注其他哨兵发布的消息**。

![3-21][image-92]

图 3-21

##### sentinel 如何知道 slave 并监控

Sentinel 之间建立连接形成集群还不够，还需要跟所有 slave 建立连接，不然没法监控他们。除此之外，如果发生了主从切换也得通知 slave 重新跟新 master 建立连接执行数据同步。

关键也是利用 master 来实现，sentinel 向 master 发送 `INFO` 命令， master 自然是知道自己所有的 salve。所以 master 接收到命令后，便将 slave 列表告诉 sentinel。

Sentinel 根据 master 响应的 slave 名单信息与每一个 salve 建立连接，并且根据这个连接持续监控 slave，剩下的哨兵也同理基于此实现监控。

![3-22][image-93]

图 3-22

#### 2. 选择 sentinel 执行主从切换

> Chaya：“master 不可用后，如何选择一个 sentinel 来执行自动故障切换呢？”

简单地说，sentinel 标记 master “客观下线”后，则通过选举投票方式获得 leader 角色执行主从切换。

**任何一个 sentine 判断 master “主观下线”后，就会向其它 sentine 发送 `is-master-down-by-addr` 命令，当其他 sentinel 收到命令后则根据自己与 master 之间的连接状况分别响应 `Y` 或者 `N` ，`Y` 表示赞成票， `N` 就是反对。**

如果某个 sentinel 获得了大多数 sentinel 的“赞成票”之后，就标记 master 为 “客观下线”。

比如一共 3 个 sentinel 组成集群，那么 quorum 就可以配置成 2，当一个哨兵获得了 2 张赞成票，就可以标记 master “客观下线”，当然这个票包含自己的那一票。

**获得多数赞成票的 sentinel 向其他 sentine 发送 `SENTINEL is-master-down-by-addr <masterip> <masterport> <sentinel.current_epoch> *` 命令，申明自己想要执行主从切换并开始拉票**。其他 sentinel 则进行投票，投票过程就叫做 “**Leader 选举**”。sentinel 选举的过程，借鉴了分布式系统中的 Raft 协议。

> Chaya：“我发现判断 master 是否客观下线和拉票选举 leader 是同样的指令。”

没错， `is-master-down-by-addr` 命令有两个作用：一是询问其他哨兵是否认为某个主节点已经主观下线；二是开始故障迁移时，当前哨兵向其他哨兵实例进行"拉票"，让其选自己为领导节点。

想要成为 “Leader” 没那么简单，得有两把刷子。需要满足以下条件：

1. 获得其他 sentine 过半的投票；
2. 投票的数量还要**大于等于** quorum 的值。

如果 sentine 集群有 2 个实例，此时，一个 sentinel 要想成为 Leader，必须获得 2 票，而不是 1 票。所以，如果有个哨兵挂掉了，那么，此时的集群是无法进行主从库切换的。因此，通常我们至少会配置 3 个哨兵实例。

![图 3-23][image-94]

图 3-23

#### 3. Pub/Sub 实现通知

在 Redis 中， pub/sub 机制发布不同事件，让客户端订阅消息。sentinel 提供的消息订阅频道有很多，不同频道包含了主从库切换过程中的不同关键事件。

##### master 相关

- +sdown：节点处于 “主观下线”状态；
- -sdown：节点不再属于“主观下线”状态；
- +odown：节点进入“客观下线”状态；
- -odown：节点退出“客观下线”状态；
- +switch-master：master 地址发生了变化。

##### slave 相关

- +slave-reconf-sent：leader sentinel 发送 `REPLICAOF`命令命令重新配置从库；
- +slave-reconf-inprog：slave 配置了新 master，但是尚未进行同步；
- +slave-reconf-done：slave 配置了新 master，并与新 master 完成了数据同步；

Redis 的 pub/sub 发布订阅机制尤其重要，有了 pub/sub 机制，哨兵和哨兵之间、哨兵和从库之间、哨兵和客户端之间就都能建立起连接了，各种事件的发布也是通过这个机制实现。

## 3.4 Redis Cluster 集群

> Chaya：“感谢 Redis 大佬，自从用上了 sentinel 集群实现自动故障后，男朋友终于可以开心的跟我约会，不怕 Redis 忽然宕机了。
> 
> 可是他最近遇到一个糟心的问题，Redis 数据库需要保存 800 万个键值对，占用 20 GB 的内存。
> 
> 于是使用了一台 32G 的内存主机部署，但是 Redis 响应有时候非常慢，使用 INFO 命令查看 latest\_fork\_usec 指标（最近一次 fork 耗时），发现特别高“

Redis：“latest\_fork\_usec 指标高的原因，是 RDB 持久化机制导致的，Fork 子进程完成 RDB 持久化操作，fork 执行的耗时与 Redis 数据量成正相关。Fork 执行的时候会阻塞主线程，由于数据量过大导致阻塞主线程过长，所以出现了 Redis 响应慢的表象。”

> Chaya：“除此之外，随着业务规模的拓展，数据量越来越大，**主从架构升级单个实例硬件难以拓展**。高并发情况下，即使设置了读写分离，但是写请求压力都集中在单个 master 上，快扛不住了。”

除了使用大内存主机的方式，我们还可以使用切片集群。俗话说众人拾材火焰高，一台机器无法保存所有数据，那就多台分担。

**使用 Redis Cluster 集群，主要解决了大数据量存储导致的各种慢问题，同时也便于横向拓展。**

两种方案对应着 Redis 数据增多的两种拓展方案：**垂直扩展（scale up）、水平扩展（scale out）。**

1. 垂直拓展：升级单个 Redis 的硬件配置，比如增加内存容量、磁盘容量、使用更强大的 CPU。
2. 水平拓展：横向增加 Redis 实例个数，每个节点负责一部分数据。

比如需要一个内存 24 GB 磁盘 150 GB 的服务器资源，有以下两种方案。

![3-24][image-95]

图 3-24

**在面向百万、千万级别的用户规模时，横向扩展的 Redis 切片集群会是一个非常好的选择。**

> Chaya：“这两种方案都有什么优缺点呢？”

- 垂直拓展部署简单，但是当数据量大，使用 RDB 实现持久化会造成阻塞导致响应慢。另外受限于硬件和成本，拓展内存的成本太大，比如拓展到 1TB 内存。
- 水平拓展便于拓展，不需要担心单个实例的硬件和成本的限制。但是，切片集群会涉及多个实例的分布式管理问题，**需要解决如何将数据合理分布到不同实例，同时还要让客户端能正确访问到实例上的数据**。

### 3.4.1 Redis Cluster 是什么

Redis Cluster 在 Redis 3.0 及以上版本提供，是一种分布式数据库方案，通过分片（sharding）来进行数据管理（分治思想的一种实践），并提供复制和故障转移功能。

Redis Cluster 并没有使用一致性哈希算法，而是将数据划分为 16384 的 slots ，每个节点负责一部分 slots，slot 的信息存储在每个节点中。

它是去中心化的，如图 3-25 所示，该集群有三个 Redis mater 节点组成（省略每个 master 对应的的 slave 节点），每个节点负责整个集群的一部分数据，每个节点负责的数据多少可以不一样。

![图 3-25][image-96]

图 3-25

三个节点相互连接组成一个对等的集群，它们之间通过 `Gossip`协议相互交互集群信息，最后每个节点都保存着其他节点的 slots 分配情况。

#### 1. Redis Cluster 目标

Redis Cluster 是 Redis 的分布式实现，在设计中按重要性顺序具有以下目标。

1. 高性能和水平可伸缩性：为实现高性能，去掉客户端代理，主从之间使用异步复制，并避免在值上执行合并操作。此外水平拓展最高可高达 1000 个节点。
2. 写入安全性：系统尝试(采用 best-effort 方式)保留所有连接到 master 节点的 client 发起的写操作。
3. 高可用性：可以在大多数 master 可用的情况下下依然对外提供服务，每个 master 必须至少有一个 slave 。此外，通过 replicas migration 技术，会从拥有拥有多个 slave 的 master 的冗余 slave 漂移出去给没有 slave 的 master 来确保可用性。

> Chaya：“Redis Cluster 避免合并操作的原因是什么？”

在 Redis，值通常非常大，通过会看到包含数百万个元素的 List 和 sorted set，另外 redis 的数据类型也比较复杂。**如果需要传输并且合并不同节点上的数据，这可能会是一个性能瓶颈**。

#### 2. Cluster 集群安装

一个 Redis 集群通常由多个节点（node）组成，在刚开始的时候，每个节点都是相互独立的，它们都处于一个只包含自己的集群当中，要组建一个真正可工作的集群，我们必须将各个独立的节点连接起来，构成一个包含多个节点的集群。

连接各个节点的工作可以通过 `CLUSTER MEET` 命令完成：`CLUSTER MEET <ip> <port>` 。

向一个节点 node 发送 `CLUSTER MEET` 命令，可以让 node 节点与 ip 和 port 所指定的节点进行握手（handshake），当握手成功时，node 节点就会将 ip 和 port 所指定的节点添加到 node 节点当前所在的集群中。

具体安装过程，详见官方文档。我在这里简单说下大致流程让你心里有个概念。

![图 3-26][image-97]

图 3-26

### 3.4.2 Redis Cluster 原理

> Chaya：“说的这么强大，到底是如何实现的？”

Redis Cluster 没有使用一致性 Hash，而是引入了**哈希槽（HASH\_SLOT）**概念，一共有 16384 个槽（slot），集群 mater 节点最大上限是 16384（官方建议最大节点数为 1000 个），数据库的每个 key 会映射到这 16384 个槽中的其中一个，每个节点可以处理 1 个或者最多 16384 个槽。

将键映射到哈希槽的基本算法 `HASH_SLOT = CRC16(key) mod 16384`。Key 与哈希槽映射过程可以分为两大步骤。

1. 对键值对的 key 使用 CRC16 算法，计算出一个 16 bit 的值。
2. 将 16 bit 的值对 16384 执行取模，得到 0 ～ 16383 的数表示 key 对应的哈希槽。

Redis Cluster 还允许用户强制某个 key 挂在特定槽位上，只需要将特定的 key 放置在大括号“{}”内，确保它们被哈希到相同的槽位，这种方式叫做 hash tag。

hash tag 规则如下，用于保证满足以下规则的 key 保存在同一个 槽位上。

比如，两个键"{user1000}.following"和"{user1000}.followers"将哈希到相同的哈希槽，因为只有"user1000"子字符串会被用来计算哈希槽。

> Chaya：“hash tag 强行把多个 key 分配到相同的槽位上，有啥用呀？”

问得好，这是 **Redis Cluster 中实现 multi-key 操作的基础**，比如执行复杂的多键操作的命令（如集合并集和交集）在涉及到操作的 key 可能会分配到不同节点，通过 hash tag 可以实现都散列到相同的槽位，从而实现 multi-key。

在 Redis Cluster 中，仅支持一个数据库，即数据库 0；因此，不允许使用 SELECT 命令切换数据库。

在 Redis Cluster 中，有四个非常核心的数据结构，`clusterNode`、 `clusterState` `clusterSlotToKeyMapping`、`clusterLink`。

#### clusterNode

Redis Cluster 使用 clusterNode 来抽象一个节点，有几个重要的字段展开说下。

```c
typedef struct clusterNode {

  // 省略部分字段
    char name[CLUSTER_NAMELEN];
    unsigned char slots[CLUSTER_SLOTS/8];
    int numslots;
    int numslaves;
    struct clusterNode **slaves;
    struct clusterNode *slaveof;
  // 省略一些字段
} clusterNode;
```

- name，长度为 40 的字符串，存储该节点的名称。
- slots、numslots：如果这个节点是 master 的话，slots 是一个 char 类型数组，记录该节点维护的 slot 信息，使用 bitmap 的方式表示。Redis Cluster 会把数据映射到 16384 个 slot 存储，因此，这个数组的长度为 16384 / 8 = 2048；numslots 记录当前节点维护的 slot 个数。
- `**slaves`、`numslaves`，当前节点是一个 master 节点，slaves 是一个 clusterNode 二级指针，指向一个 clusterNode 数组，数组每个 clusterNode 指针指向一个 slave 节点。numslaves 表示当前 master 节点的 slave 个数。
- slaveof，如果当前节点是 slave，则指向 master 的 clusterNode 实例。

#### clusterState

> 谢霸哥：“clusterNode 抽象了每个节点的信息，节点运行状态由什么来表示呢？”

Redis Cluster 中每个节点都维护了一个 clusterState 实例，表示节点状态。

在 Redis 源代码的`server.h`文件中，`redisServer`结构体包含一个名为`cluster`的字段，它是一个指向`clusterState`类型的指针。

这指针用于感知 Redis Cluster 中每个节点的状态。通过`cluster`指针，Redis 服务器可以管理和监视整个 Redis 集群，以便进行故障检测、状态管理和集群维护等操作。这是 Redis Cluster 的关键组件之一，用于实现集群功能。

```c
struct redisServer {
  // 省略其他字段
	struct clusterState *cluster;
};
```

来展开分析下 clusterState 结构体核心字段。

```c
typedef struct clusterState {
    clusterNode *myself;
    uint64_t currentEpoch;
    int state;
    int size;
    dict *nodes;
    clusterNode *slots[CLUSTER_SLOTS];

  // 省略部分源码
} clusterState;
```

- myself，指向当前节点的指针。
- currentEpoch 字段：记录了当前节点看到的最新 Cluster 纪元，可以认为这是一个时钟逻辑，主要用于故障自动转移过程中的选举投票环节。
- state，当前 Redis Cluster 的状态，CLUSTER\_OK 表示集群在线，CLUSTER\_FAIL 表示集群下线。
- size ，当前 Redis Cluster 中有效的 master 节点个数。master 至少负责至少一个 slot 时，才会有读写请求发送到该 master 节点，这才是一个有效 master 节点。
- nodes，这是一个 dict 指针，key 是 节点名称，value 是表示每个节点的 clusterNode 实例。
- slots，这是一个 clusterNode 数组，数组长度为 16384，用于记录每个 slot 被哪些节点负责。

#### clusterSlotToKeyMapping

在 `server.h`文件的 `redisDb` 结构体中有一个 `clusterSlotToKeyMapping *slots_to_keys` 指针，指向一个长度为 16384 的 slotToKeys 类型数组。数组下标是 slot 编号，数组每个元素 slotToKeys 是一个结构体，slotToKeys 把位于这个 slot 的 键值对串联成一个双端链表。这里并没有复制一份键值对数据，而是指向 redisDb 的 dictEntry 实例。

**server.h 的 redisDb 结构体**

省略部分源码，具体每个字段类型在之前的篇章已经详细说过，这里不再赘述。

```c
typedef struct redisDb {
 		// 省略部分源码
    dict *dict;
    dict *expires;
    clusterSlotToKeyMapping *slots_to_keys;
} redisDb;
```

**cluster.h 中的 clusterSlotToKeyMapping 与 slotToKeys 结构体**

```c
typedef struct slotToKeys {
    // 该 slot 中键值对数量
    uint64_t count;
    // 指向 dictEntry 链表头结点
    dictEntry *head;
} slotToKeys;

struct clusterSlotToKeyMapping {
    // 长度16384 的 slotToKeys 类型数组
    slotToKeys by_slot[CLUSTER_SLOTS];
};
```

有此可知，通过 clusterSlotToKeyMapping 可定位每个 slot 负责的 key 集合。

#### clusterLink

> Chaya：“Redis Cluster 节点点的通信连接信息，包括连接的创建时间、连接对象、发送和接收缓冲区等用什么表示呢？”

定义在源码 `cluster.h` 的 `clusterLink` 结构体，用于抽象节点间的通信信息。

```c
typedef struct clusterLink {
    mstime_t ctime;
    connection *conn;
    sds sndbuf;
    char *rcvbuf;
    size_t rcvbuf_len;
    size_t rcvbuf_alloc;
    struct clusterNode *node;
    int inbound;
} clusterLink;
```

- ctime，连接创建时间。
- conn，两个节点的网络连接对象。
- sndbuf，用于发送数据包的缓冲区。
- rcvbuf，用于接收数据包的缓冲区。
- node，clusterNode 指针，指向的 clusterNode 实例表示连接的对端节点。

#### clusterMsg

> Chaya：“集群中每个节点的信息、节点状态、每个节点负责的 slots 主从复制进度等是如何传播的？”

定义在源码 `cluster.h` 的 `clusterMsg` 结构体，用于抽象集群中传播信息的载体抽象。

```c
typedef struct {
    char sig[4]; //消息签名
    uint32_t totlen; //消息长度
    uint16_t ver;    // 版本号
    uint16_t port;      /* 当前节点 port */
    uint16_t type;      // 消息类型
    uint16_t count;     /* 携带的节点信息条数 */
    uint64_t currentEpoch;  /* 当前纪元. */
    uint64_t configEpoch;   /* 配置纪元 */
    uint64_t offset;    /* 主从复制 offset 进度*/
    char sender[CLUSTER_NAMELEN]; /* 发送消息的节点名称 */
    unsigned char myslots[CLUSTER_SLOTS/8]; // 当前节点负责的 slots
    char slaveof[CLUSTER_NAMELEN]; // 当前节点的 master 节点名称
    char myip[NET_IP_STR_LEN];    /* 发送消息的节点 IP */
    uint16_t extensions; /*拓展字段*/
    char notused1[30];   /* 预留的 30 字节，未来使用 */
    uint16_t pport;      /*发送方 TCP 明文端口（如果基本端口是 TLS） */
    uint16_t cport;      /* Sender TCP cluster bus port */
    uint16_t flags;      /* 当前节点状态 */
    unsigned char state; /* Cluster state from the POV of the sender */
    unsigned char mflags[3]; /* Message flags: CLUSTERMSG_FLAG[012]_... */
    union clusterMsgData data; // 具体消息内容
} clusterMsg;
```

重点关注 data 字段 ，clusterMsgData 是一个 union 联合体，同一时刻只能使用其中一个成员的值，成员变量 ping、fail、publish、update、module 结构体的任意一个，其中 `PING`、`MEET`和 `PONG`都共用 ping 结构体。

```c
union clusterMsgData {

    struct {
        /* clusterMsgDataGossip 数组，每个 元素就包含一个节点的信息 */
        clusterMsgDataGossip gossip[1];
    } ping;

    /* FAIL */
    struct {
        clusterMsgDataFail about;
    } fail;

    /* PUBLISH */
    struct {
        clusterMsgDataPublish msg;
    } publish;

    /* UPDATE */
    struct {
        clusterMsgDataUpdate nodecfg;
    } update;

    /* MODULE */
    struct {
        clusterMsgModule msg;
    } module;
};
```

#### 哈希槽与 Redis 实例映射

> Chaya：“hash slot 与 Redis 实例之间是如何映射关联上的？”

你可以使用 `cluster create` 创建 Redis Cluster，Redis 会自动将 16384 哈希槽平均分布在集群 master 实例上，比如 N 个节点，每个节点上的哈希槽数 = 16384 / N 个。

除此之外，你还可以使用 `CLUSTER MEET` 命令将 7000、7001、7002 三个节点连在一个集群，但是集群目前依然处于下线状态，因为三个实例都没有处理任何哈希槽。

可以使用 `cluster addslots` 命令，指定每个实例负责的哈希槽范围。

> Chaya：“为什么要手动指定呢？”

能者多劳嘛，加入集群中的 Redis 实例配置不一样，如果承担一样的压力，对于垃圾机器来说就太难了，让牛逼的机器多承担一点压力。

三个 master 实例的集群，通过下面的指令为每个实例分配哈希槽：`实例 1`负责 0 ～ 5460 哈希槽，`实例 2` 负责 5461\~10922 哈希槽，`实例 3` 负责 10923 ～ 16383 哈希槽。

```bash
redis-cli -h 172.16.19.1 –p 6379 cluster addslots 0,5460
redis-cli -h 172.16.19.2 –p 6379 cluster addslots 5461,10922
redis-cli -h 172.16.19.3 –p 6379 cluster addslots 10923,16383
```

假设键值对、哈希槽、 Redis 实例之间的映射关系如下图所示。

![图3-27][image-98]

图 3-27

Redis 键值对的 key “码哥字节”、“牛逼”，经过 CRC16 计算后再对哈希槽总个数 16394 取模，模数结果分别映射到实例 1 与实例 2 上。切记，**当 16384 个槽都分配完全，Redis 集群才能正常工作**。

#### 复制与自动故障转移

> Chaya：“Redis Cluster 如何实现高可用呢？”

每个 master 会处理自己负责的 slots 读写请求，必须至少有一个 slave 通过主从复制架构同步 master 节点数据，当 master 故障，slave 晋升为新 master 继续处理请求，当下线的旧 master 重新上线，则作为 slave 角色与新 master 建立主从关系。

主从之间并没有读写分离，slave 只用作 master 宕机的高可用备份。我还提供了一个参数`cluster-require-full-coverage`默认值是 `yes`，如果部分 key 对应的哈希槽映射的实例故障，redis cluster 将停止执行写请求。如果设置成 `no`，集群依然继续响应读请求。

比如 7000 主节点宕机，作为 slave 的 7003 成为 Master 节点继续提供服务。当下线的节点 7000 重新上线，它将成为当前 70003 的从节点。

> Chaya：“Redis Cluster 如何实现自动故障转移呢？”

简而概之的说，Redis Cluster 会经历以下三个步骤实现自动故障转移实现高可用。

1. **故障检测**：集群中每个节点都会定期通过 `Gossip` 协议向其他节点发送 `PING` 消息，检测各个节点的状态（**在线状态**、**疑似下线状态 PFAIL**、**已下线状态 FAIL**）。并通过 `Gossip` 协议来广播自己的状态以及自己对整个集群认知的改变。
2. **master 选举**：使用从当前故障 master 的所有 slave 选举一个提升为 master。
3. **故障转移**：取消与旧 master 的主从复制关系，将旧 master 负责的槽位信息指派到当前 master，更新 Cluster 状态并写入数据文件，通过 `gossip` 协议向集群广播发送 `CLUSTERMSG_TYPE_PONG`消息，把最新的信息传播给其他节点，其他节点收到该消息后更新自身的状态信息或与新 master 建立主从复制关系。

##### 故障检测

> Chaya：“什么是 Gossip 协议，在 Redis Cluster 中作用是什么？”

Gossip 协议是一种去中心化的分布式协议，用于节点之间的信息传递、状态同步和故障检测。它基于节点之间相互随机交流信息，以便全局传播信息并保持一致性。

Gossip 算法，又被称为反熵（Anti-Entropy），源自物理学中的熵的概念，熵代表了一种无序和混乱的状态。而反熵则是指在混乱无序中寻求一致性的过程。

这个名称充分体现了 Gossip 算法的核心特点：**在一个有界网络中，每个节点都随机地与其他节点进行通信。经过一系列随机、无序的信息传递后，最终所有节点的状态都会趋于一致。**

> Chaya：“李老师，说人话。”

新冠病毒传播知道吧，Gossip 算法就好像病毒一样传播，当你出去耍的时候，随机遇到的人，不戴口罩就互相传播，一传十十传百。

Redis Cluster 彼此之间状态同步信息交换靠的就是 Gossip 协议。

通过 Gossip 协议进行通信，节点之间不断交换信息，交换的信息包括节点出现故障、新节点加入、主从节点变更， slots 信息变更等。常用的 Gossip 消息分为 4 种，分别是：ping、pong、meet、fail。

- `meet` 消息：通知新节点加入。消息发送者通知接受者加入当前集群。
- `ping`消息：每个节点每秒向其他节点发送 ping 消息，用于检测节点在线和交换刺激状态信息。
- `pong`消息：节点接受到 `ping` 消息后，作为响应消息回复发送方确认正常，同时 pong 还包含了自身的状态数据，想集群广播 pong 消息来通知集群自身状态进行更新。
- `fail`消息：节点 ping 不通谋节点后，则向集群所有节点广播该节点挂掉的消息。

**基于 Gossip 协议的故障检测**

Redis Cluster 中的节点会按照每秒一次的频率随机向其他节点发送 `PING` 数据包，对端节点收到 `PING` 消息后会回复应 `PONG` 数据包。Redis Cluster 节点就是通过某个节点能否及时回复 `PONG`包来判断节点是否下线和互相交换节点信息。

下线状态分两种情况，**疑似下线（PFAIL）和下线（FAIL）**，就类似于 sentinel 中的主观下线和客观下线，目的还是为了防止误判。

比如节点 A 没有在`server.cluster_node_timeout`时间内收到节点 B 对 `PING` 包的回复，就会将节点 B 标记为 PFAIL。

一个节点认为某个 下线，并不代表真的下线，只有**当大多数负责处理 slots 的节点都判定为 PFAIL，才能标记为 FAIL 状态**。

一旦 slave 标记某个节点为 FAIL 状态，就会通过 Gossip 协议向整个集群广播 `fail` 消息，进入选主流程，接着执行故障转移。

##### slave 选举与晋升流程

> 肖菜鸡：“新 master 是如何选举出来的？”

**纪元（epoch）**

集群的配置纪元 +1，是一个自曾计数器，初始值 0 ，每次执行故障转移都会 +1。它的作用在于当集群的状态发生改变，某个节点为了执行一些动作需要寻求其他节点的同意时，就会增加 epoch 的值，提供增量版本管控制。

并在节点互相交换信息出现冲突时帮助解决冲突，用来确定哪个状态是最新的。

**广播拉票消息**

检测到 master FAIL 的 slave A 节点利用 Gossip 协议广播`CLUSTERMSG_TYPE_FAILOVER_AUTH_REQUEST`消息，要求所有收到这条消息并且具有投票权的 master 节点进行投票。只有负责 slots 的 节点才会有投票资格。

如果该 master 没有投票给其他 slave 节点，那么该 master 将对 slave A 返回一条 `CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK`消息，表示支持 slave A 成为新的主节点。

**投票判决**

参与选举的 slave 节点都会接收`CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK`消息，如果 slave 收到的支持票数 \>= (N/2) + 1 支持，那么这个 slave 就被选举为新 master。

如果在一个纪元里面没有 slave 能收集到足够多的支持票，那么集群进入一个新的纪元，并再次进行选举，直到选出新的 master 为止。

跟哨兵类似，两者都是基于 Raft 算法来实现的，流程如图所示。

![3-28][image-99]

图 3-28

**纪元（epoch）**

##### 故障转移

投票选举出新 master 之后，就开始对下线的 master 执行故障转移。

1. 新 master 会撤销已下线的旧 master 负责的 slots 指派，并把这些 slots 指派给自己负责，断开之前的主从复制连接。
2. 新的 master 向集群中广播一条 `PONG` 消息，通知集群中的其他节点自己已经从 slave 节点变成 master 节点，并且接管了已经下线的 master 负责的 slots。
3. 新 master 开始接收处理槽有关的命令请求，故障转移完成。

#### 客户端如何定位数据

> 李老师：“Chaya，我考下你。Redis Cluster 并没有采用一致性哈希算法把 key/value 分配不到不同节点上。而是分为 16384 个 slots，集群中每个节点负责一部分 slots。对 key 使用 CRC16 算法，计算出一个 16 bit 的值。将 16 bit 的值对 16384 执行取模，得到 0 ～ 16383 的数表示 key 对应的 slots，从而定位到节点。
> 
> 如果用一个散列表直接把键值对与节点的映射关系记录下来（比如字典的 key 保存的是键值对的 key，value 存储实例 runID），这样的话不需要使用 CRC16 算法，直接查散列表就可以，Redis 为啥不这么做呢？

Chaya：“使用一个散列表记录的话，假如键值对和节点之间的关系改变（重新分片、节点增减），需要修改散列表。如果是单线程操作，所有操作都要串行处理，性能太慢了，对于唯快不破的 Redis 来说是不会答应的。

而采用多线程的话，就涉及到加锁，另外，**如果键值对数据量非常大，保存键值对与节点关系的表数据所需要的存储空间也会很大。**

而哈希槽计算，虽然也要记录哈希槽与实例时间的关系，但是哈希槽的数量少得多，只有 16384 个，开销很小。”

> Chaya：“李老师，Redis 客户端是如何知道请求的数据分布在哪个节点呢？”

还记得前面说的 `clusterMsg` 结构体么？Redis 会构建一条 clusterMsg 消息，该消息就包含了 slots 信息，并通过 Gossip 协议发送给器群其他节点，实现了 slots 分配信息的扩散。

如此一来，集群中每个节点都知道了所有 slots 与节点之间的映射关系。所以客户端连接到任何一个 Redis 实例，节点就把 slots 与节点映射信息响应给客户端缓存在本地。

当客户端发起请求时，**客户端会先对 key 执行 CRC16 计算得到对应的 slot，再去本地缓存查找 slot 映射的 Redis 节点，将请求发送到目标节点。**

![图 3-29][image-100]

图 3-29

#### 请求重定向

> Chaya：“新增节点或者重新分配 slots 导致 slots 与节点之间的映射关系改变了，客户端如何知道把请求发到哪里？”

这个问题问的好，集群中的实例通过 Gossip 协议互相传递消息，以此获取每个节点的 slots 分配信息。可是，客户端无法知道集群的变更。

于是，Redis Cluster 提供了**请求重定向机制**解决：**客户端将请求发送到某个节点上，这个节点没有相应的数据，该 Redis 节点会告诉客户端将请求发送到其他的节点**。

在 Cluster 模式下，客户端发起请求到指令处理会发生如下过程。

1. 客户端通过 `CRC16(key) / 16384` 计算出 slot，通过本地缓存的 slots 与节点之间的映射得到负责该节点的指针，并把请求发送到节点上。
2. 节点判断该 slot 是否由自己负责，key 在 slot 中，则向客户端返回 key 对应的结果。
3. 若 key 的 slot 不是该节点负责，则向客户端返回 `MOVED`错误，告知客户端重定向到目标节点。
4. 若 key 对应的 slot 正在迁移（还未把 slot 全部的 key 迁移完成），向客户端返回 `ASK` 错误重定向到迁移的目标节点上。

##### MOVED 重定向

当 slots 重新分配实现负载均衡或者 slot 的数据已经迁移到其他节点，而客户端将一个键值对的操作请求发送到某个节点，而 这个 key 对应的 slot 并非自己负责的时候，该节点会响应一个 MOVED 错误指引客户端重定向负责该 slot 的节点。

```bash
GET 公众号:码哥字节
-MOVED 16330 172.17.18.2:6379
```

该响应的含义是客户端请求的键值对所在的 slot 16300 已经迁移到了 172.17.18.2 这个节点上，端口是 6379。

同时，**客户端还会更新本地缓存，将该 slot 与 Redis 实例对应关系更新正确**。

![3-30][image-101]

图 3-30

##### ASK 重定向

> 程旭源：“如果某个 slot 的数据比较多，部分迁移到新实例，还有一部分没迁移过去怎么办？”

这时候不能直接使用 `MOVED` 重定向，因为 `MOVED` 表示 slot 已经由另一个节点提供服务，而此刻是可能还在当前节点，也可能迁移过去了。

节点收到客户端请求如果能根据 key -\> slot -\> node 映射关系定位到的节点存在该 key，则直接执行命令，否则就向客户端响应 `ASK` 错误。

比如需要访问的 key 所在 slot 正在从节点 1 迁移到节点 2，节点 1 会返回客户端一条 ASK 报错信息，表示**客户端请求的 key 所在的 slot 正在迁移到节点 2 上，你先给实例 2 发送一个 ASKING 命令，问下节点 2 是否可以处理，接着再向节点 2 发送操作命令**。

比如客户端请求定位到 key = “公众号:码哥字节” 的 slot 是 16330 由实例 172.17.18.1 负责，节点 1 如果找得到就直接执行命令，否则响应 ASK 错误信息，指引客户端转向正在迁移的目标节点 172.17.18.2，端口是 6379。

```bash
GET 公众号:码哥字节
-ASK 16330 172.17.18.2:6379
```

![3-31][image-102]

图 3-31

注意：**ASK 错误指令并不会更新客户端缓存的 slot 分配信息**。

所以客户端再次请求 slot 16330 的数据，还是会先给 `172.17.18.1` 实例发送请求，只不过节点会响应 ASK 命令让客户端给新节点发送一次请求。

`MOVED`指令则**更新客户端本地缓存，让后续指令都发往新实例。**

#### 集群大小受限原因

> Chaya：“有了 Redis Cluster，再也不怕大数据量了，我可以无限水平拓展么？”

答案是否定的，**Redis 官方给的 Redis Cluster 的规模上限是 1000 个实例**。

到底是什么限制了集群规模呢？

1. **协调和通信开销：** 随着节点数量的增加，集群需要进行更多的协调和通信。每个节点都要与其他节点保持连接，共享集群状态和信息。当节点规模增加时，这种通信和协调开销也随之增加，可能导致网络和性能方面的负担。
2. **集群状态复杂性：** Redis Cluster 需要维护关于 slot 分配、节点状态等的集群状态信息。随着节点数量的增加，这种状态的复杂性也会增加。更多的节点可能使集群管理变得更加复杂，增加维护和监控的难度。
3. **故障转移和恢复：** 在节点故障时，Redis Cluster 需要进行故障转移和数据恢复。增加节点数量会增加故障转移和恢复的开销。在较大的集群中，恢复可能需要更长的时间，并可能对集群整体性能产生影响。
4. **一致性和可用性：** 随着节点数量的增加，维护一致性和提高可用性可能变得更加复杂。更多的节点可能导致更复杂的网络拓扑，增加了一致性和可用性的管理难度。

比如关键的原因在于节点间的通信开销，Cluster 集群中的每个实例都保存所有 slots 与实例对应关系信息（slot 映射到节点的表），以及自身的状态信息。

在集群之间每个节点通过 `Gossip`协议传播节点的数据，`Gossip` 协议工作原理大概如下。

1. 从集群中随机选择一些节点按照一定的频率发送 `PING` 消息，用于检测实例状态以及交换彼此的信息。 `PING` 消息中封装了发送者自身的状态信息、部分其他实例的状态信息、slot 与节点映射表信息。
2. 节点接收到 `PING` 消息后，响应 `PONG` 消息，消息包含的信息跟 `PING` 消息一样。

集群之间通过 `Gossip`协议可以在一段时间之后每个实例都能获取其他所有实例的状态信息。

所以在有新节点加入，节点故障，Slot 映射变更都可以通过 `PING`，`PONG` 的消息传播完成集群状态在每个实例的传播同步。

##### Gossip 消息

`PING`、`MEET`和 `PONG`三种消息都共用 ping 字段，是 `clusterMsgDataGossip` 结构体。

```c
typedef struct {
    char nodename[CLUSTER_NAMELEN];// 字符数组，节点名称，40 字节
    uint32_t ping_sent;// 4 字节，发送 ping 消息次数
    uint32_t pong_received; //4 字节，接收到的 pong 消息次数
    char ip[NET_IP_STR_LEN]; // 46 字节
    uint16_t port; // 2 字节
    uint16_t cport; // 2 字节
    uint16_t flags; // 2 字节
    uint16_t pport; //  使用 TLS 才会使用，2 字节
    uint16_t notused1;// 2 字节
} clusterMsgDataGossip;
```

每个节点发送一个 `Gossip`消息，就需要发送 104 字节。如果集群是 1000 个节点，那么每个接地啊发送一个 `PING` 消息则会占用 大约 10KB。

除此之外，节点在传播 slot 映射表的时候，每个消息还包含了 一个长度为 16384 bit 的 `Bitmap`。每一位对应一个 slot，如果值 = 1 则表示这个 slot 属于当前节点，这个 Bitmap 占用 2KB，所以一个 `PING` 消息大约 12KB。

`PONG`与`PING` 消息一样，一发一回两个消息加起来就是 24 KB。集群规模的增加，心跳消息越来越多就会占据集群的网络通信带宽，降低了集群吞吐量。

### 3.4.3 Cluster 集群配置注意事项

Redis Cluster 集群的部分配置，使用集群方式的你必须重视和知晓。别嘴上原理说的头头是道，如何实现真正的高可用却一头雾水。

#### 降低节点间的通信开销

Redis Cluster 的实例每 100 ms 就会扫描本地实例列表，当发现有实例最近一次收到 `PONG` 消息的时间 \> `cluster-node-timeout / 2`。那么就立刻给这个实例发送 `PING` 消息，更新这个节点的集群状态信息。

当集群规模变大，就会进一步导致实例间网络通信延迟怎加。可能会引起更多的 PING 消息频繁发送。

- 每个实例每秒发送一条 `PING`消息，降低这个频率可能会导致集群每个实例的状态信息无法及时传播。
- 每 100 ms 检测实例 `PONG`消息接收是否超过 `cluster-node-timeout / 2`，这个是 Redis 实例默认的周期性检测任务频率，我们不会轻易修改。

只能修改 `cluster-node-timeout`的值：集群中判断实例是否故障的心跳时间，默认 15S。

所以，**为了避免过多的心跳消息占用集群宽带，将 `cluster-node-timeout`调成 20 秒或者 30 秒，这样 `PONG` 消息接收超时的情况就会缓解。**

但是，也不能设置的太大。都则就会导致实例发生故障了，却要等待 `cluster-node-timeout`时长才能检测出这个故障，影响集群正常服务。

#### cluster-migration-barrier

没有 slave 节点的 master 节点称为孤儿 master 节点，这个配置就是用于防止出现裸奔的 master。

当某个 master 的 slave 节点宕机后，集群会从其他 master 中选出一个富余的 slave 节点迁移过来，确保每个 master 节点至少有一个 slave 节点，防止当孤立 master 节点宕机时，没有 slave 节点可以升为 master 导致集群不可用。默认配置为 `cluster-migration-barrier 1`，是一个迁移临界值。

# 第 4 章 结丹飞升高级技能进阶

## 4.1 Redis 事务修炼手册

> 吴颜组面试官：“数据库事务的 ACID 了解么？”

程旭源内心独白：“小意思，不就是 ACID 嘛，转眼一想，我面试的可是技术专家，不会这么简单的问题吧？”

程旭源：“balabala……极其自信且从容淡定的说了一通。”

> 吴颜组面试官：“Redis 的事务机制了解么？它的事务实现了 ACID 么？”

程旭源：“挠头，这个……我知道 lua 脚本可以实现事务，Redis 可以保证脚本内的命令一次性、按顺序地执行，但是不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完。”

> 吴颜组面试官：“好的，回去等通知吧。”

程旭源：“码哥，我学了你的 Redis 高手心法在寒潮中依然斩获了许多 offer，没想到今天败在了 Redis 如何实现事务这个问题上。心有不甘呀。”

别着急，一步步分析，把 Redis 事务修炼到家。

1. 事务的 ACID 是什么？
2. Redis 如何实现事务？
3. Redis 的事务满足 ACID 么？

### 4.1.1 什么是事务的 ACID？

事务（Transaction）是由**一系列**对系统中数据进行访问或更新的操作所组成的一个程序执行逻辑单元（Unit）。**这些操作要么都执行，要么都不执行**。

例如，傻狗哥发工资了，要上交给 Chaya。这个过程涉及到银行转账工作：从傻狗哥的源账号**扣款**给 Chaya 的目标账号**增款**，这两个操作必须要么全部执行，要的都不执行，否则会出现该笔金额平白消失或多出的情况，所以，要当成一个事务来处理。

事务在执行时，为了保持数据库的一致性，在事务处理之前和之后，都遵循某些属性，也就是大家耳熟能详的 ACID 属性。

- **原子性（Atomicity）**：一个事务的多个操作必须完成，或者都不完成。事务执行过程中发生错误，就回滚到事务开始前的状态，就好像没有被执行过一样。

- **一致性（Consistency）**：事务执行结束后，数据库的完整性约束没有被破坏。数据库的完整性约束包括但不限于。

  - 实体完整性（如行的主键存在且唯一）。
  - 列完整性（如字段的类型、大小、长度要符合要求）。
  - 外键约束。
  - 用户自定义完整性（如转账前后，两个账户余额的和应该不变）。

  还是上面的例子，如果傻狗哥和 Chaya 两个账户加起来的金额一共 999 元，不管他们两人之间如何转账，事务结束后两个账户的钱相加必须是 999 元。

  你会发现在转账过程中，未保证原子性的话，那么结果数据如上述的完整性约束也无法得到保障，不满足一致性。也就是说事务的一致性和原子性是密切相关的。**一致性既是事务的属性，也是事务的目的。**

- **隔离性（Isolation）**：**事务内部的操作与其他事务是隔离的，并发执行的各个事务之间不能互相干扰**。讲究的是不同事务之间的相互影响，严格的隔离性对应隔离级别是可串行化（Serializable）。

- **持久性（Durability）**：事务一旦提交，所有的修改将永久的保存到数据库中，即使系统崩溃重启后数据也不会丢失。

### 4.1.2 Redis 如何实现事务？

> 程旭源：“码哥，捋明白事务的 ACID 之后，Redis 是如何实现事务呢？”

总的来说，`MULTI`、`EXEC`、`DISCARD` 和 `WATCH` 命令是 Redis 实现事务的的基础。Redis 事务的执行过程包含以下步骤。

1. `MULTI` 命令开启一个事务。
2. 将命令逐个当如等待执行的事务队列里面，命令并不会马上执行。
3. `EXEC` 执行事务中的所有命令或 `DISCARD`丢弃第二步的指令。

> 程旭源：“talk is cheap show me the code.”

#### 开启事务

客户端使用 `MULTI`命令开启一个事务，客户端进入事务态。

```bash
> MULTI
OK
```

#### 命令入队

当客户端处于**非事务状态时**，这个客户端发送的命令会被立即执行。`MULTI` 该命令可以将执行该命令的客户端从非事务状态切换至事务状态。

客户端进入状态，由 `server.h` 中的 `multiState` 结构体用于表示整个事务的状态，包括事务队列、命令数量、命令标志等信息。

```c
typedef struct multiState {
    multiCmd *commands;     /* 存储事务中的所有命令的队列*/
    int count;              /* 事务中包含命令个数*/
    int cmd_flags;
    int cmd_inv_flags;
    size_t argv_len_sums;    /* 所有命令参数使用的内存总量。 */
    int alloc_count;         /* 记录分配的命令结构体 `multiCmd` 数量*/
} multiState;
```

进入**事务状态**后，你只需要把一些列需要执行的命令暂存到队列中，它们并不会立刻执行，除非发送的命令为 `EXEC` 、 `DISCARD` 、 `WATCH` 、 `MULTI` 四个命令的其中一个，服务器才会立即执行这个命令。

```bash
> SET "姓名" "Chaya"
QUEUED
> SET "身高" "158"
QUEUED
> SET "性格画像" "情商高，凶巴巴,有时候很气人又体贴"
QUEUED
```

你可以看到每个指令执行后的返回结果都是 `QUEUED`，表示操作都被暂存到了命令队列，但没有实际执行。

`multiCmd` 结构体用于表示一个事务中的单个命令。

```c
typedef struct multiCmd {
    robj **argv; //指向 Redis 对象的指针数组，表示事务命令的参数。
    int argv_len;  // 参数数组 argv 的长度。
    int argc; // 事务命令的参数个数。
    struct redisCommand *cmd;//指向 Redis 命令结构体的指针，表示要执行的命令。
} multiCmd;
```

#### 执行事务或丢弃命令

##### 执行事务

只需要执行 `EXEC` 命令，就表示提交事务，执行第二步中发送的具体命令。

```bash
> EXEC
OK
OK
OK
```

可以看到，`EXEC` 返回一个应答数组，每个元素是事务中单个指令的应答，执行 `GET` 命令验证命令，获得一个 “情商高，凶巴巴,有时候很气人又体贴”的小姐姐。

```bash
> GET "姓名"
Chaya
> GET "身高"
158
> GET "性格画像"
情商高，凶巴巴,有时候很气人又体贴
```

##### 丢弃命令放弃事务

通过 `DISCARD` 丢弃第二步中保存在队列中的命令。

```bash
# 初始化订单数
> SET "order:mobile" 100
OK
# 开启事务
> MULTI
OK
# 订单 - 1
> DECR "order:mobile"
QUEUED
# 丢弃丢列命令
> DISCARD
OK
# 数据没有被修改
> GET "order:mobile"
"100"
```

### 4.1.3 Redis 事务满足 ACID 么？

Redis 事务做了三个重要保证。

1. 事务支持一次执行多个命令，所有命令都会被序列化并按照顺序串行化执行。
2. 执行 `EXEC` 命令，进入事务执行过程中，其他客户端发送的请求不会插入到该事务执行命令序列中。
3. `EXEC` 命令会触发事务中所有指令的执行，如果在调用 `EXEC` 命令之前，客户端与服务器失去连接，不会执行任何操作。如果在执行过程中，任意命令执行失败，其余命令依然会执行。

先说结论，**Redis 事务可以理解具备原子性，但不支持回滚。具备一致性和通过 WATCH 保证隔离性，但是无法保证持久性。**

#### 1. 原子性 (Atomicity)

> 程旭源：“码哥，事务执行过程中出错，能保证原子性么？”

在 Redis 事务期间，可能遇到以下两种错误。

- **语法错误：**执行 `EXEC` 命令前，入队的命令语法是错误的（参数数量错误、命令名称错误等），或者内存不足（Redis 实例使用 `maxmemory`指令配置内存限制）。
- **命令报错**：`EXEC` 执行后，某些命令可能会出错。比如，命令和操作的数据类型不匹配（对 `String` 类型的 key 执行 `List` 操作）。
- **Redis 忽然宕机**：在执行 `EXEC` 后，忽然断电宕机，只有部分事务操作记录到 AOF 日志的场景。

接下来分别分析在这三种场景下事务是否保证原子性。

##### 语法错误（EXEC 执行前）

从 Redis 2.6.5 开始，Redis 会在命令入队的时候检测指令语法错误，你可以继续提交命令到队列。但是在执行 `EXEC`命令后，**Redis 将拒绝执行所有的命令，并返回一个事务失败给客户端，保证原子性。**

```bash
#开启事务
> MULTI
OK
#发送事务中的第一个操作，但是Redis不支持该命令，返回报错信息
127.0.0.1:6379> PUT order 6
(error) ERR unknown command `PUT`, with args beginning with: `order`, `6`,
#发送事务中的第二个操作，这个操作是正确的命令，Redis把该命令入队
> SET "码哥字节" "拥抱技术和对象，面向人民币编程"
QUEUED
#实际执行事务，命令有错误，Redis拒绝执行
> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
```

注意，在 Redis 2.6.5 之前，客户端需要自己检查命令入队的返回值来选择丢弃还是继续。如果命令回复 `queued` 则表示语法正确，否则返回错误。如果命令入队错误，客户端需要手动执行 `DISCARD` 丢弃事务，否则继续执行 `EXEC` 的话将执行队列中的所有命令。

都 2023 了，大清早就亡了。应该没公司还用 Redis 2.6.5 之前的版本了吧？

##### 命令运行时报错（EXEC 执行后）

**命令和操作的数据类型不匹配， Redis 实例没有检查出错误，可以正常入队，在 `EXEC` 之后运行指令报错，但是队列中其他正确指令也会被执行。**

如下，我使用 List 的命令来操作 string 类型的数据，指令能正常入队，但运行时错误。

```bash
> MULTI
OK
> SET chaya 爱上李老师，开心又快乐
QUEUED
> LPUSH chaya 跪着唱征服
QUEUED
> EXEC
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
> GET chaya
爱上李老师，开心又快乐
```

chaya 爱上李老师，开心又快乐，但是没有跪着唱征服，想让 Chaya 跪着唱征服，现实中不存在。Redis 都报错了（`(error) WRONGTYPE Operation against a key holding the wrong kind of value`）。

**敲黑板了：Redis 虽然会对错误指令报错，但是事务依然会把正确的命令执行完，错误的指令本身是没意义的，我们可以理解保证原子性！**

> 程旭源：“码哥，Redis 支持回滚么？”

**Redis 在事务失败时不进行回滚，而是继续执行余下的正确命令**因为支持回滚会对 Redis 的简单性和性能产生重大影响。

此外，**Redis 的命令只会因为错误的语法或者命令用在了错误类型的键上面，也就是说事务中失败的指令是由于编程错误造成，这些错误应该在开发过程中发现，而不应该出现在生产中。**

##### EXEC 执行时，宕机

断电 Redis 忽然宕机，开启 AOF，只有部分事务的指令记录到 AOF 中，我们可以使用 redis-check-aof 工具检查 AOF 日志文件，这个工具可以**把未完成的事务操作从 AOF 文件中去除，相当于事务不会被执行，从而保证了原子性**。

##### 简单总结

- **EXEC 执行前，命令语法错误，入队时就报错，Redis 将拒绝执行所有的命令，并返回一个事务失败给客户端，保证原子性。**
- **EXEC 执行后，命令和操作类型不匹配，实际执行时报错，不会因为失败指令终止事务，正确指令会被继续执行，不能保证原子性。**
- **`EXEC` 命令执行时实例故障，开启了 AOF 日志，可以保证原子性。**

综上诉述，Redis 事务在**特定的条件下才具备原子性**。

#### 2. 一致性(Consistency)

分三种异常场景来讨论。

- **入队错误**：执行 `EXEC`之前，客户端发送的操作指令错误，事务终止执行，**可以保证一致性**。
- **执行错误**：`EXEC` 执行之后，命令和操作的数据类型不匹配，错误的命令会报错，正确的指令会继续执行，从这个角度看，可以保证一致性。
- **服务宕机**：执行事务过程中，Redis 服务宕机，要根据持久化的模式来分别讨论。
  - 无持久化的内存模式：没有开启 RDB 或者 AOF。实例故障重启，数据库包邮保存任何数据，可以**保证一致性。**
  - 开启 RDB 内存快照，可以根据 RDB 内存快照文件恢复数据，从而保证数据库还原到一个一致状态，如果找不到找不到 RDB 内存快照文件，空白数据库**满足一致性。**
  - 开启 AOF 持久化模式，事务的操作还没有被记录到 AOF 中就宕机，当重启使用 AOF 日志恢复数据的时候数据满足一致性；如果只有部分操作记录到 AOF 中，我们可以使用 redis-check-aof 清除事务中已经完成的操作，数据库恢复后也**满足一致性**。

**综上所述，Redis 事务满足一致性。**

#### 3. 隔离性(Isolation)

需要明确的一点：**Redis 事务没有事务隔离级别的概念。Redis 的隔离性指的是并发场景下，事务之间是否可以做到互不干扰。**

可以将事务的执行分为在 `EXEC` 执行前和 `EXEC` 命令之后后两个阶段分开讨论，先说结论。

1. 并发操作在 `EXEC` 命令前执行，隔离性通过 `WATCH` 机制保证；
2. 并发操作在 `EXEC` 命令之后，隔离性可以保证。

##### `EXEC` 执行前

在 `EXEC`执行前，事务的所有命令暂存在队列中。此刻，如果出现并发操作，其他客户端的的指令对事务中同样的 key 做修改，是否满足隔离性需要根据事务是否使用 `WATCH` 机制来判断。

**使用 WATCH 机制**

WATCH 机制的作用是：在事务执行前，监控一个或多个键的值变化情况，当调用 EXEC 命令执行时，WATCH 机制会先检查监控的键是否被其它客户端修改。**如果修改了，就自动放弃事务执行，避免事务的隔离性被破坏。**

客户端 A，初始化 key = order:books 的值为 100，接着使用 `WATCH` 机制监听该 key 的变化。

```bash
> SET order:books 100
OK
> WATCH order:books
OK
> MULTI
OK
(TX)> DECR order:books
QUEUED
(TX)> EXEC
(nil)
```

接着在客户端 A 执行 `EXEC` 之前，客户端 B 执行如下指令修改数据。

```bash
> DECR order:books
(integer) 99
```

客户端 A 执行 `EXEC` ，发现事务中的指令并没有被执行，事务取消，符合隔离性。

```bash
(TX)> EXEC
(nil)
> GET order:books
"99"
```

综上，**如果存在竞争条件，并且在调用 `WATCH` 和调用 `EXEC` 之间的时间内，另一个客户端修改 `order:books` 的结果，则终止事务执行。**

![4-1][image-103]

图 4-1

**未使用 WATCH 机制**

如果没有 WATCH 机制， 在 `EXEC` 命令执行前存在并发操作对同样的 key 做写操作，Redis 事务不能保证隔离性。代码不再演示，你只需要把上文 `WATCH` 代码去掉即可。

![4-2][image-104]

图 4-2

##### `EXEC` 执行后

因为 Redis 是用单线程执行命令，`EXEC` 命令执行后，Redis 会保证先把事务队列中的所有命令执行完再执行其他指令。这样就可以保证事务的隔离性。

![][image-105]

图 4-3

#### 4. 持久性(Durability)

如果 Redis 没有开启 RDB 内存快照或 AOF 日志持久化，事务肯定不能保证持久性。

如果开启 RDB 内存快照持久化，一个事务执行完成后，下一个 RDB 快照还未执行前，突然发生宕机，数据就会丢失，无法保证持久性。

如果开启 AOF 日志，AOF 模式的三种配置选项 no 、everysec、always(AOF 使用的是先写内存数据再写日志的方式，如果刚执行完指令，还没记录日志到磁盘就宕机，就有可能丢失这个指令操作) 都会存在数据丢失的情况 。

综上， **Redis 事务的持久性无法保证。**

## 4.2 Redis 内存管理（淘汰策略与过期删除策略）

我是 Redis， 是一个基于内存操作的数据库，所有的数据保存在内存中，所以读写速度非常快。然，内存容量远远比不上磁盘存储空间，若不及时清理无用或者价值不大的数据，就会导致内存不足。

我就从内存淘汰策略和过期删除策略来聊一聊我的的内存管理心法。此等武功在手，天下我有。

> 谢霸哥：“Redis 的内存是否可以设置资源限制？当内存不足的时候会发生什么？设置过期时间的 key 达到过期时间时，是怎么从内存中删除呢？”

先说结论，分别回答三个问题。

1. 你可以通过 `redis.conf`配置`maxmemory 4gb`限制我的最大内存使用量。需要注意的是，如果 `maxmemory` 为 0 ，在 `64` 位操作系统上则没有限制，而 `32` 位操作系统则有 `3GB` 的隐式限制。
2. 当 Redis 主库内存超过限制，命令处理会**触发数据淘汰机制淘汰数据**，该机制由`maxmemory-policy`配置的策略来控制， 直到内存使用量小于限制阈值或者拒绝服务。
3. **并不会立马删除**，Redis 有两种删除过期数据的策略。
   1. 定时任务选取部分数据删除。
   2. 惰性删除：当有客户端的请求该 `key` 的时候，检查下 `key` 是否过期，如果过期，则删除。

### 4.2.1 淘汰策略概述

淘汰的目标数据可分为两种。

1. 数据库素有键值对数据。
2. 数据库中被设置了过期时间的键值对数据。

针对这两种目标数据，一共有八种淘汰策略。

- **noeviction**：默认策略，不淘汰任何数据。
- **allkeys-lru**：使用近似 LRU 算法淘汰长时间没有使用的 key。
- **allkeys-lfu**：使用近似 LFU 算法，保留常用的键，删除整个数据库最不常用的键。
- **volatile-lru**：使用近似 LRU 算法删除设置了过期时间，最近最少使用的键。
- **volatile-lfu**：使用近似 LFU 算法删除设置了过期时间， 使用频率最低的键。
- **allkeys-random**：对所有 key 随机删除键，为添加的新数据腾出空间。
- **volatile-random**：随机删除`expire`字段设置为 的键`true`。
- **volatile-ttl**：删除最接近过期时间的键，越早过期越线被淘汰。

**需要注意的是，LRU、LFU、volatile-ttl 都是近似随机算法来采样数据进行淘汰。**

> 谢霸哥：“这么多内存淘汰策略，我怎么记得住呀？”

莫慌，我们可以用两个维度、四个算法来考虑。

两个维度。

- 对 key 设置了过期时间。
- 对所有键。

四个算法。

- LRU 最近最少使用。
- LFU 使用频率最低。
- random 随机。
- ttl 存活时间。

#### 淘汰策略如何选择？

##### allkeys-random 使用场景

假如数据没有明显的冷热分别，所有的数据分布查询比较均衡，这些数据都会被随机查询，那就使用 **allkeys-random** 策略，让其随机选择淘汰数据。

##### volatile-lru、allkeys-lru 使用场景

使用 `volatile-lru` 策略，业务场景有一些数据不能删除，比如置顶新闻、视频，这时候我们为这些数据不设置过期时间，数据就不会被删除，该策略就会去根据 LRU 算法去淘汰那些设置了过期时间且最近最少被访问的数据。

**将需要持数据不能删除的和全都可以淘汰数据的业务系统分别使用不同的 Redis 实例集群是更好的方案。**

##### volatile-lfu、allkeys-lfu 使用场景

业务场景明显存在访问频率差异明显，且我们可以删除低频数据的场景。

#### 1. 数据淘汰过程概述

淘汰执行过程如下图所示。

![图 4-4][image-106]

图片 4-4

- 客户端发送新命令到服务端。

- 服务端收到客户端命令，Redis 检查内存使用情况，如果大于 `maxmemory` 限制，则根据`maxmemory-policy`配置的策略来淘汰数据。
- 执行新命令。

Redis 处理新命令并执行淘汰策略的源码位于 `server.c`的 `processCommand`方法中。重点关注 `int out_of_memory = (performEvictions() == EVICT_FAIL);` 该方法源码位于 `evict.c`中，主要功能是检查内存使用是否超过 `maxmemory`限制，如果超过则尝试通过淘汰策略释放内存。如果淘汰了一部分数据，占用内存依然超过限制，就会启动 `aeTimeProc` 定时任务继续调用 `performEvictions()`淘汰数据，直到小于内存限制或无法淘汰为止。

截取部分源码如下，源码基于 Redis 7.0 版本。

```c
int processCommand(client *c) {
    // 省略部分代码

    if (server.maxmemory && !isInsideYieldingLongCommand()) {
       // 内存超过限制，调用 performEvictions() 检测内存使用情况，进行淘汰策略清理数据
        int out_of_memory = (performEvictions() == EVICT_FAIL);

        trackingHandlePendingKeyInvalidations();

        if (server.current_client == NULL) return C_ERR;
        int reject_cmd_on_oom = is_denyoom_command;
        /* 如果客户端处于 MULTI/EXEC 上下文，入队操作可能会消耗无限制的内存，因此我们希望阻止这种情况发生。然而，我们绝不希望拒绝 DISCARD 操作，毕竟这个会取消事务执行，可以降低内存*/
        if (c->flags & CLIENT_MULTI &&
            c->cmd->proc != execCommand &&
            c->cmd->proc != discardCommand &&
            c->cmd->proc != quitCommand &&
            c->cmd->proc != resetCommand) {
            reject_cmd_on_oom = 1;
        }

      // 内存溢出，调用 rejectCommand 拒绝命令执行
        if (out_of_memory && reject_cmd_on_oom) {
            rejectCommand(c, shared.oomerr);
            return C_OK;
        }
        server.pre_command_oom_state = out_of_memory;
    }
}
```

#### 2. 淘汰策略原理

下面从简单到复杂，分别说说每种策略实现原理。淘汰策略源码位于 `evict.c`的 `performEvictions`方法中。

##### 不淘汰数据（noeviction）

默认策略，不淘汰任何数据。当内存达到设置的最大值时，所有需要申请内存的操作都会返回 oomerr 错误，服务支持读操作，少数写命令可以执行，比如 `DEL`、`unlink`这些可以降低内存使用的写指令。

- 32 位操作系统没有设置 `maxmemory` ，系统默认最大值是 3G。
- 64 位操作系统没有设置 `maxmemory` ，是你没有限制的。Linux 系统通过虚拟内存管理物理内存，进程可以使用超过物理内存大小的值，只不过这时候物理内存与磁盘会频繁的进行 swap，严重影响性能。

操作系统是 32 位，设置默认淘汰策略和最大内存限制的源码位于 `server.c` 的 `initServer` 方法。

```c
void initServer(void) {
    // 省略其他源码...
    if (server.arch_bits == 32 && server.maxmemory == 0) {
        serverLog(LL_WARNING,"Warning: 32 bit instance detected but no memory limit set. Setting 3 GB maxmemory limit with 'noeviction' policy now.");
      // 限制最大内存
        server.maxmemory = 3072LL*(1024*1024); /* 3 GB */
      // 设置淘汰策略为 noeviction
        server.maxmemory_policy = MAXMEMORY_NO_EVICTION;
      // 省略其他源码...
    }
}
```

##### 随机淘汰策略

`allkeys-random`、`volatile-random`两种都属于随机淘汰策略，使用 next\_db 变量逐渐访问所有数据库，每个数据库都有机会进行淘汰，而不是只在一个数据库中进行，并**随机选择一个键名 `bestkey` 和数据库索引 `bestdbid` 则退出本次循环**。之后则执行删除选出来的 key。

```c
int performEvictions(void) {
  		// 省略其他代码......
  while (mem_freed < (long long)mem_tofree) {
    // 省略其他代码......

        else if (server.maxmemory_policy == MAXMEMORY_ALLKEYS_RANDOM ||
                 server.maxmemory_policy == MAXMEMORY_VOLATILE_RANDOM)
        {

            for (i = 0; i < server.dbnum; i++) {
                j = (++next_db) % server.dbnum;
                db = server.db+j;
                dict = (server.maxmemory_policy == MAXMEMORY_ALLKEYS_RANDOM) ?
                        db->dict : db->expires;
                if (dictSize(dict) != 0) {
                  // 随机选择一个键
                    de = dictGetRandomKey(dict);
                   // 键名称
                    bestkey = dictGetKey(de);
                  // 数据库索引
                    bestdbid = j;
                    break;
                }
            }
        }
    // 省略其他代码......
  }
  		// 省略其他代码......
}
```

##### 采样淘汰

`volatile-ttl`、`volatile-lru`、`volatile-lfu`、`allkeys-lru`、`allkeys-lfu`淘汰策略可以根据到期时间、LRU、LFU 进行淘汰数据，严格意义上来说，需要一些数据结构来维护才能准确的筛选出这些目标数据，而 Redis 通过采样的方法，近似的数据淘汰策略，并非严格意义上的 LRU 或者 LFU 算法。

比如 LRU 算法，最近最少使用算法，我们把所有的数据用一个链表维护。

- **MRU**：表示链表的表头，代表着最近最常被访问的数据。
- **LRU**：表示链表的表尾，代表最近最不常使用的数据。

![图 4-5][image-107]

图 4-5

可以发现，**LRU 更新和插入新数据都发生在链表首，删除数据都发生在链表尾**，被访问的数据会被移动到 MRU 端。

如果 Redis 使用该 LRU 算法管理所有的缓存数据，会造成大量额外的空间消耗。除此之外，**大量的节点被访问就会带来频繁的链表节点移动操作，从而降低了 Redis 性能。**

所以 Redis 对该算法做了简化，Redis LRU 算法并不是真正的 LRU，通过**对少量的 key 采样**避免内存消耗。

主要经历以下几个步骤来选出需要淘汰的 key 并进行淘汰。

1. 根据 `maxmemory-samples` 配置初始化一个样本池，默认配置是 5。
2. 遍历数据库随机采集 `maxmemory-samples 5` 个样本，按照 idle 值从大小小排序放进样本池中（对 key 计算出一个 idle 值，用于确定插入样本池的位置）。
3. 遍历样本池，从里面选择优先级最高的键来淘汰。
4. 当成功选择到了要淘汰的键后，从样本池中移除该 key。
5. 通过 `bestkey` 变量返回选定的键，执行淘汰逻辑。

很明显，样本越大，越接近于真实 LRU 算法。Redis 作者根据实践经验，`maxmemory_samples` 默认每次采样 5 个已经比较高效了，10 个就非常接近 LRU 算法效果。

扫描数据库，调用 `evictionPoolPopulate`方法随机采样多个数据放到样本池中， 从样本池中取出淘汰键 `bestkey` 进行淘汰部分源代码如下所示。

```c
 int performEvictions(void) {
    ...

     while (mem_freed < (long long)mem_tofree) {
       if (server.maxmemory_policy & (MAXMEMORY_FLAG_LRU|MAXMEMORY_FLAG_LFU) ||
            server.maxmemory_policy == MAXMEMORY_VOLATILE_TTL)
        {
            struct evictionPoolEntry *pool = EvictionPoolLRU;

            while (bestkey == NULL) {
                unsigned long total_keys = 0, keys;

                /*遍历所有数据库*/
                for (i = 0; i < server.dbnum; i++) {
                    db = server.db+i;
                  // 从设置过期时间的 key 中扫描，还是所有 key
                    dict = (server.maxmemory_policy & MAXMEMORY_FLAG_ALLKEYS) ?
                            db->dict : db->expires;

                    if ((keys = dictSize(dict)) != 0) {
                       // 随机采样多个数据并放到样本池中
                        evictionPoolPopulate(i, dict, db->dict, pool);
                        total_keys += keys;
                    }
                }
                if (!total_keys) break; /* No keys to evict. */

                /* 数组从右到左遍历，查找键进行数据淘汰 */
                for (k = EVPOOL_SIZE-1; k >= 0; k--) {
                    if (pool[k].key == NULL) continue;
                    bestdbid = pool[k].dbid;

                    if (server.maxmemory_policy & MAXMEMORY_FLAG_ALLKEYS) {
                        de = dictFind(server.db[bestdbid].dict,
                            pool[k].key);
                    } else {
                        de = dictFind(server.db[bestdbid].expires,
                            pool[k].key);
                    }

                    /* 从样本池中移除选中的 key */
                    if (pool[k].key != pool[k].cached)
                        sdsfree(pool[k].key);
                    pool[k].key = NULL;
                    pool[k].idle = 0;

                    /* key 存在，删除并跳出循环 */
                    if (de) {
                        bestkey = dictGetKey(de);
                        break;
                    } else {
                        /* Ghost... Iterate again. */
                    }
                }
            }
        }
     }

 }
```

采样核心流程：**遍历数据库，每个数据库随机采集 `maxmemory_samples` 个样本放入样本池中，样本池的 key 会按照 idle 的值从低到高排序（数组从左到右存储），淘汰的时候会每次选择 idle 值最高的那个数据。所以采集样本的时候要计算 idle 值来确定插入到样本池的位置。**

所以，经过经过多次循环采样，池子中一定存储着 idle 最大的，优先级最高被淘汰的数据。采样流程和源代码如下。

![图 4-6][image-108]

图 4-6

随机采样并计算 idle 值，确定插入到样本池中中的核心源码如下。

```c
void evictionPoolPopulate(int dbid, dict *sampledict, dict *keydict, struct evictionPoolEntry *pool) {
    int j, k, count;
    dictEntry *samples[server.maxmemory_samples];
		// 随机采样多个 key 样本池中，
    count = dictGetSomeKeys(sampledict,samples,server.maxmemory_samples);
    // 分别计算每个 key 的 idle 值，用于确定插入到样本池的位置。
    for (j = 0; j < count; j++) {
       ...

        /* 计算评分 idle，表示淘汰优先级，分值越高意味着优先淘汰*/
        if (server.maxmemory_policy & MAXMEMORY_FLAG_LRU) {
           // 近似 LRU 算法，淘汰最长时间没有使用的数据。
            idle = estimateObjectIdleTime(o);
        } else if (server.maxmemory_policy & MAXMEMORY_FLAG_LFU) {
            /* 淘汰使用频率小的数据*/
            idle = 255-LFUDecrAndReturn(o);
        } else if (server.maxmemory_policy == MAXMEMORY_VOLATILE_TTL) {
            /* 淘汰接近过期时间的数据 */
            idle = ULLONG_MAX - (long)dictGetVal(de);
        } else {
            serverPanic("Unknown eviction policy in evictionPoolPopulate()");
        }

        /* 将采集到 key 放到 pool 数组中，在 pool 数组中需要找到合适位置，pool[k].key == NULL 或者 idle < pool[k].idle */
        k = 0;
        while (k < EVPOOL_SIZE &&
               pool[k].key &&
               pool[k].idle < idle) k++;
        if (k == 0 && pool[EVPOOL_SIZE-1].key != NULL) {
            /* pool 满了，本次采样找不到合适位置插入 */
            continue;
        } else if (k < EVPOOL_SIZE && pool[k].key == NULL) {
            /* 找到平合适位置插入，不需要移动数组其他元素 */
        } else {
            /* 还有闲置空间，但是需要移动数据插入到指定位置维护顺序。  */
            if (pool[EVPOOL_SIZE-1].key == NULL) {

                sds cached = pool[EVPOOL_SIZE-1].cached;
                memmove(pool+k+1,pool+k,
                    sizeof(pool[0])*(EVPOOL_SIZE-k-1));
                pool[k].cached = cached;
            } else {
                // pool 数组没有空间，删除 idle 最小的元素，因为 idle 值越大，淘汰优先级越高。
                k--;

                sds cached = pool[0].cached;
                if (pool[0].key != pool[0].cached) sdsfree(pool[0].key);
                memmove(pool,pool+1,sizeof(pool[0])*k);
                pool[k].cached = cached;
            }
        }

        /* 尝试重复使用淘汰池中预分配的内存空间。因为内存的分配和销毁开销大*/
        int klen = sdslen(key);
        if (klen > EVPOOL_CACHED_SDS_SIZE) {
            pool[k].key = sdsdup(key);
        } else {
            memcpy(pool[k].cached,key,klen+1);
            sdssetlen(pool[k].cached,klen);
            pool[k].key = pool[k].cached;
        }
        pool[k].idle = idle;
        pool[k].dbid = dbid;
    }
}
```

### 4.2.2 过期删除策略

使用 `EXPIRE key seconds [ NX | XX | GT | LT]` 指令可以为 key 设置过期时间，如果没有设置过期时间， key 将一直存在，除非我们明确将其删除，比如执行 `DEL` 指令。

从 Redis 版本 7.0.0 开始，`EXPIRE` 添加了选项：`NX`、`XX`和`GT`、`LT` 选项。

- NX：当 key 没有过期时才设置过期时间。
- XX：只有 key 已过期的时候才设置过期时间。
- GT：仅当**新设置的到期时间**大于当前到期时间时才设置。
- LT：仅在新过期时间小于当前到期时间才设置。

> Chaya：“码哥，前面介绍的是在 Redis 内存占满的情况下淘汰数据策略。若是在内存没被占满，Redis 如何把过期的键值对数据删除优化内存占用呢？”

Redis 中不会立马把过期的 key 从内存中删除，为了保证系统高性能会同时按照以下两种策略进行删除。

- 惰性删除：当 key 被访问时，检查 key 的过期时间，若已经过期则删除。
- 定期删除：每隔一段时间随机检查设置了过期的 key 并删除已过期的数据。

#### 1. 惰性删除

惰性删除很简单，就是当有客户端的请求查询该 `key` 的时候，检查下 `key` 是否过期，如果过期，则删除该 `key`。

比如当 Redis 收到客户端的`GET movie:小泽#玛……利亚.rmvb` 请求，就会先检查 `key = movie:小泽#玛……利亚.rmvb` 是否已经过期，如果过期那就删除。**删除过期数据的主动权交给了每次访问请求。**

该实现是通过 `expireIfNeeded`函数实现，源码路径：`src/db.c`。

```c
int expireIfNeeded(redisDb *db, robj *key, int flags) {
  // 检查 key 是否已过期，未过期 return 0 返回。
    if (!keyIsExpired(db,key)) return 0;

   // 如果当前请求的是 master 主节点，不执行删除，由 master 发送 DEL 执行来删除过期键
    if (server.masterhost != NULL) {
        if (server.current_client == server.master) return 0;
        if (!(flags & EXPIRE_FORCE_DELETE_EXPIRED)) return 1;
    }

    if (flags & EXPIRE_AVOID_DELETE_EXPIRED)
        return 1;


    if (checkClientPauseTimeoutAndReturnIfPaused()) return 1;

    // 删除 key，并传播过期删除操作。
    deleteExpiredKeyAndPropagate(db,key);
    return 1;
}
```

#### 2. 定期删除

> Chaya：“如果某个 key 一直没有被访问，岂不是一直保存在内存中？”

这个问题问得好。仅仅靠客户端访问来判断 key 是否过期才执行删除肯定不够，因为有的 key 过期了，但是还一直没被访问，不能让这些数据“占着茅坑不拉屎”。

靠的就是**定期删除**，也就是 Redis 默认每 1 秒运行 10 次（每 100 ms 执行一次），每次随机抽取 20 个设置了过期时间的 key，检查是否过期，如果发现过期了就直接删除。

步骤如下。

![图 4-7][image-109]

图 4-7

1. 从所有设置了过期时间的 key 集合中随机**选择 20（ACTIVE\_EXPIRE\_CYCLE\_KEYS\_PER\_LOOP） 个 key**。
2. 删除步骤 1 发现的所有过期 key 数据；
3. 步骤 2 结束，过期的 key 占比超过 25%，则重复执行步骤 1。

> Chaya：“码哥，为啥不检查所有设置过期时间的 key？”

你想呀，假设 Redis 里存放了 100 w 个 key，都设置了过期时间，每隔 100 毫秒就检查 100 w 个 key，CPU 全浪费在检查过期 key 上了，Redis 也就废了。

注意了：**不管是定时删除，还是惰性删除。当数据删除后**，`master` **会生成删除的指令记录到** `AOF` 和 `slave` **节点**。

删除的源码 `expire.c` 的 `activeExpireCycle` 函数实现，代码好长，不贴了。

> Chaya：“如果过期的数据太多，定时删除无法完全删除（每次删除完过期的 key 占比还是超过 25%），会怎样？
> 
> 会不会导致 Redis 内存耗尽，怎么破？”

这个问题问得好，答案是走前面说的**内存淘汰机制**，完美不闭环。

今天就到这里，说太多的话，大家容易在知识的海量里呛死，保命要紧，Redis 内存淘汰策略和过期删除策略原理到此结束。

#### 3. 过期与 RDB 持久化

key 过期信息是用 **Unix 绝对时间戳**表示的。

**为了让过期操作正常运行，机器之间的时间必须保证稳定同步，否则就会出现过期时间不准的情况。**

比如两台时钟严重不同步的机器发生 RDB 传输， slave 的时间比实际大 2000 秒，假如在 master 的一个 key 设置生存时间是 1000 秒，当 Slave 加载 RDB 的时候就会认为该 key 过期（因为 slave 机器时间设置比实际大 2000 s），并不会等待 1000 s 才过期。

![图 4-8][image-110]

图 4-8

#### 4. AOF 和 RDB 是否存储已过期 key

当 Redis 中的 key 已过期但是未被删除时，Redis 并不会把过期的 key 持久话到 RDB 内存快照文件或 AOF 文件中。

为了确保一致性，在 AOF 文件中，当 key 过期时，会改写成 `DEL`命令存储。slave 节点也不会主动删除过期 key，除晋升为 master 节点或者收到 master 节点发送过来的 `DEL` 同步执行令。

## 4.3 Redis 事件驱动：文件和时间的协奏曲

在这篇文章中，我们将聚焦 Redis 的事件驱动机制，研究文件事件和时间事件如何构建出协奏曲般的高性能处理引擎。

Redis 服务器是一个事件驱动程序，服务需要处理两类事件：文件事件（file event）和时间事件（time event），也就是关注网络 I/O，以及周期定时任务。

### 4.3.1 Redis server 启动入口

Redis 使用 C 语言开发，软件启动通常放在 main 函数中。main 方法源码定义在 `server.c`中，我省略一些代码，只保留一些关键步骤，看起来清爽多了，想研究全部细节 debug 的话就打开第一章编译一个源码阅读调试环境。

我抽取了关键步骤，让你从全局视角来看 Redis 是如何启动并处理命令请求的。

```c
int main(int argc, char **argv) {

   // 省略部分代码...

    /* 阶段一：初始化服务器，比如 Redis 的配置、时区,只保留部分让你知道这么一大串东西都是初始化工作，贴太多也不想看 */

    setlocale(LC_COLLATE,"");
    init_genrand64(((long long) tv.tv_sec * 1000000 + tv.tv_usec) ^ getpid());
    crc64_init();
    umask(server.umask = umask(0777));
    ACLInit();
    moduleInitModulesSystem();
    tlsInit();

     // ... 再次省略一大段初始化代码

    // 如果是哨兵模式则加载哨兵配置并初始化哨兵
    if (server.sentinel_mode) {
        initSentinelConfig();
        initSentinel();
    }

    /*阶段二：加载配置文件*/
    // 检查是否在运行 redis-check-rdb 或 redis-check-aof，是的话就执行对应的检查逻辑
    if (strstr(exec_name,"redis-check-rdb") != NULL)
        redis_check_rdb_main(argc,argv,NULL);
    else if (strstr(exec_name,"redis-check-aof") != NULL)
        redis_check_aof_main(argc,argv);
        // 省略一大串代码......
        while(j < argc) {

          // 省略一大串代码......

        // 加载所有配置文件：来自磁盘、命令输入等
        loadServerConfig(server.configfile, config_from_stdin, options);
        if (server.sentinel_mode) loadSentinelConfigFromQueue();
        sdsfree(options);
    }

    /* 阶段三：初始化服务器、处理模块、加载 AOF 文件、从磁盘加载 RDB 数据等等。*/
    initServer();

   ......
    moduleInitModulesSystemLast();
    moduleLoadFromQueue();

    aofLoadManifestFromDisk();
    loadDataFromDisk();
    ......

		/* 阶段四：事件驱动框架，重点来了*/
    aeMain(server.el);
    aeDeleteEventLoop(server.el);
    return 0;
}
```

- 阶段一，初始化服务器：设置一些初始化变量，比如内存分配错误处理函数；初始化随机种子、时钟、CRC64 计算。
- 阶段二，加载配置文件：，检查是否在运行 redis-check-rdb 或 redis-check-aof，是的话就执行对应的检查逻辑。
- 阶段三，初始化服务器：完成解析和各种初始化配置后，调用 `initServer` 运行时的各种资源做初始化；此外，还会初始化 Modules、加载 AOF 文件、从磁盘加载 RDB 数据等。
- 阶段四，事件驱动框架，也就是今天要说的主角。就是这玩意能处理高并发客户端请求。**该框架启动后，只要电线没有被挖断，地球没爆炸，就一直循环执行，每次循环都会处理一批网络连接、读写事件和定时任务**。

#### 事件驱动框架 aeMain

事件框架处理一下两类事情。

- 文件事件：处理 Redis 服务器与客户端之间的网络 I/O（客户端连接请求、客读取客户端内容、将命令执行后的结果写回客户端）。
- 时间事件：过期 key 定期删除、 AOF 定时刷写到磁盘；周期性的后台任务，比如 AOF 文件重写，定时 RDB 内存快照文件生成等。

`aeMain` 函数定义在 `ae.c` 源码中。

```c
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        aeProcessEvents(eventLoop, AE_ALL_EVENTS|
                                   AE_CALL_BEFORE_SLEEP|
                                   AE_CALL_AFTER_SLEEP);
    }
}
```

好家伙，映入眼帘就是一个 while 主循环开头，不停止监听 `aeEventLoop` 。aeEventLoop 这个玩意，你可以理解成里面包含了需要处理的文件事件或者事件事件相关的数据。结构体定义在 `ae.h` 源码文件中。

```c
typedef struct aeEventLoop {
    int maxfd;// 注册的文件描述符最大值
    int setsize; /* 监听的文件描述符数量*/
    long long timeEventNextId;// 下一个时间事件的唯一标识符。每次创建时间事件时，此标识符会递增。
    aeFileEvent *events; /* 已注册的文件事件数组。每个元素是一个 aeFileEvent 结构体，表示一个文件事件。*/
    aeFiredEvent *fired; /* 触发的事件数组 */
    aeTimeEvent *timeEventHead; // 时间事件链表的表头
    int stop;
    void *apidata;
    aeBeforeSleepProc *beforesleep; // 在事件循环进入休眠之前调用的回调函数。
    aeBeforeSleepProc *aftersleep;// 在事件循环唤醒之后调用的回调函数。
    int flags;
} aeEventLoop;
```

循环体里面就一个 `aeProcessEvents` 函数，用来处理文件事件或者时间事件的。为了防止过度陷入细节，我删减了部分代码，只保留主干，让你能愉快的在知识的海洋徜徉。进入 `aeProcessEvents` 函数看看干了啥事。

```c
int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    int processed = 0, numevents;

    /* 1. 检查是否有时间事件和文件事件需要处理，没有的话直接返回，去蹦迪 */
    if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;

        .... 省略一些代码

        /* 2. I/O 多路复用，从内核取出就绪的 I/O 事件（可读事件、可写事件） */
        numevents = aeApiPoll(eventLoop, tvp);

       ...... 省略一些代码

    // 文件事件和事件事件至少有一种需要处理，进入分支
    if (eventLoop->maxfd != -1 ||
        ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {

       ...... 省略一些代码
        /* 2. 遍历就绪的 I/O 事件，依次处理*/
        for (j = 0; j < numevents; j++) {
            int fd = eventLoop->fired[j].fd;
            aeFileEvent *fe = &eventLoop->events[fd];
            int mask = eventLoop->fired[j].mask;
            int fired = 0;
            int invert = fe->mask & AE_BARRIER;

            // 2.1 将可读事件调度给 rfileProc 处理
            if (!invert && fe->mask & mask & AE_READABLE) {
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                fired++;
                fe = &eventLoop->events[fd]; */
            }

            /* 2.2 将可写事件分派给 wfileProc 处理 */
            if (fe->mask & mask & AE_WRITABLE) {
                if (!fired || fe->wfileProc != fe->rfileProc) {
                    fe->wfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            /* 2.3 通常是先执行可读事件，然后执行可写事件，当设置 invert 标志则反转顺序，先写后读*/
            if (invert) {
                fe = &eventLoop->events[fd];
                if ((fe->mask & mask & AE_READABLE) &&
                    (!fired || fe->wfileProc != fe->rfileProc))
                {
                    fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            processed++;
        }
    }
    /* 3. 检查是否需要处理时间事件，调用 processTimeEvents 函数处理 */
    if (flags & AE_TIME_EVENTS)
        processed += processTimeEvents(eventLoop);

   // 返回处理的事件数量
    return processed;
}
```

总的来说就是 `aeMain`函一直循环数调用 `aeProcessEvents` 来进行文件事件和时间事件的分发和执行，`aeEventLoop` 用于保存事件的相关信息。

进入 `aeProcessEvents` 函数调用 `aeApiPoll` I/O 多路复用，从内核检查就绪的 I/O 事件（连接事件、读事件、写事件），将可读事件调度给 `rfileProc` 处理，将可写事件分派给 `wfileProc` 处理。在这还有一个反转顺序的分支，将读写顺序倒置，具体场景按下不表。

第三步，检查是否需要处理时间事件，调用 `processTimeEvents` 函数处理。最后来一张图，让你提升又醒脑。

![图 4-9][image-111]

图 4-9

### 4.3.2 文件事件

Redis 基于 `Reactor` 模式开发了一个 I/O 事件处理器，也就是文件事件处理器。该处理器基于操作系统提供的 I/O 多路复用技术实现了一个非常简洁且高性能的时间驱动处理器。

主要有四个部分组成，socket 套接字、I/O 多路复用、文件事件分配器、事件处理器。

![图 4-10][image-112]

图 4-10

文件事件用于抽象 socket，当每个 socket 达到就绪状态时候（ `accept`、`read`、`write`和 `close` 等），就会创建一个对应的文件事件。

I/O 多路会监听多个 socket，并把生成的文件事件传递给文件事件分配器，事件分配器就会调用每个事件管来你的处理器来处理事件。

### 4.3.3 时间事件

Redis 时间事件分为定时事件和周期性事件，时间事件由源码`ae.h` 的 `aeTimeEvent` 结构体抽象。

```c
typedef struct aeTimeEvent {
    long long id; // 全局唯一 ID
    // 秒精确的UNIX时间戳，记录时间事件到达的时间
    monotime when;
   // 时间事件触发时调用的处理函数（回调函数）
    aeTimeProc *timeProc;
    aeEventFinalizerProc *finalizerProc;
   // 传递给时间事件处理函数的数据
    void *clientData;
    // 时间事件形成一个双向链表，这两个指针用于连接链表中的前一个和后一个事件
    struct aeTimeEvent *prev;
    struct aeTimeEvent *next;
    int refcount;
} aeTimeEvent;
```

`ae.c` 中的 `processTimeEvents` 函数是处理时间事件的入口， 会把所有的时间事件放在一个无序双向链表中，每当事件执行器运行的时候就遍历整个链表，找到时间到达的事件并调用事件关联的 `aeTimeProc` 回调函数。

```c
static int processTimeEvents(aeEventLoop *eventLoop) {

    ...
    // 遍历时间事件链表
    while(te) {
        long long id;

        /* 移除已标记为删除的时间事件. */
        if (te->id == AE_DELETED_EVENT_ID) {
            aeTimeEvent *next = te->next;
            /* 如果存在对该定时器事件的引用，不释放该 aeTimeEvent */
            if (te->refcount) {
                te = next;
                continue;
            }
            if (te->prev)
                te->prev->next = te->next;
            else
                eventLoop->timeEventHead = te->next;
            if (te->next)
                te->next->prev = te->prev;
            if (te->finalizerProc) {
                te->finalizerProc(eventLoop, te->clientData);
                now = getMonotonicUs();
            }
            // 释放被删除的时间事件内存
            zfree(te);
            te = next;
            continue;
        }

        /* 确保不处理在这次迭代中由时间事件创建的时间事件。 */
        if (te->id > maxId) {
            te = te->next;
            continue;
        }

        // 处理已到期的时间事件
        if (te->when <= now) {
            int retval;

            id = te->id;
            te->refcount++;
            retval = te->timeProc(eventLoop, id, te->clientData);
            te->refcount--;
            processed++;
            now = getMonotonicUs();
           // 时间事件的处理函数返回值不是 AE_NOMORE，则更新下一次到期时间并重新加入时间事件链表。
            if (retval != AE_NOMORE) {
              // 更新下一次到期时间的时间间隔，retval 是时间回调函数的返回值
                te->when = now + retval * 1000;
            } else {
               // 否则标记为删除
                te->id = AE_DELETED_EVENT_ID;
            }
        }
        te = te->next;
    }
    return processed;
}
```

这个函数的作用是处理 Redis 事件循环中的时间事件，其中包括删除过期的事件、调用事件处理函数等。具体步骤如下：

1. 遍历时间事件链表。
2. 如果发现时间事件被标记为删除，将其从链表中移除，并调用相应的清理函数。
3. 如果时间事件的处理函数返回值不是 `AE_NOMORE` 也就是 -1，则修改下一次到期的时间间隔重新将其加入链表。从而达到时钟定期执行的效果。
4. 返回处理的时间事件数量。

> Chaya：“服务器的第一个时间事件如何创建出来的？”

这个问题问得好，回头看前面说的 Redis server 启动流程中的步骤三`initServer()`初始化服务器资源的时候，会调用 `aeCreateTimeEvent` 方法创建时间事件。

```c
void initServer(void) {
  ....
    if (aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL) == AE_ERR) {
        serverPanic("Can't create event loop timers.");
        exit(1);
    }
  ....
}
```

重点关注该时间事件设置的处理函数 `serverCron`，定时器核心逻辑都在里面了，比如生成 AOF、BGSAVE 创建 RDB 内存快照、AOF Rewrite、异步回收关闭的链接、对设置了过期时间的键值对进行删除等。

该函数的返回值 `retval = 1000/server.hz` ，只要不等于 -1，那么就把返回值作为下一次事件到期的时间间隔，重新将事件加入链表，达到时钟定期执行的效果。函数定义在 `server.c` 中，代码很多我就不贴出来了。

总的来说，`initServer` 调用 `aeCreateTimeEvent` 创建第一个时间事件，`serverCron` 作为该事件的回调函数，这个函数是定时器的核心逻辑，函数返回值不等于 -1 就继续把该事件重新加入时间事件链表，只要地球不爆炸，就这样无线循环下去，达到时钟定期执行的效果，闭环了家人们！

![图 4-11][image-113]

图 4-11

## 4.4 Redis Pub/Sub 深度解析：频道与模式的技术内幕

“Q 哥，如果漂亮小姐姐 Chaya 做了你的女朋友，你会通过什么方式将这个消息告诉你身边的好友？“

> Q 哥：“那不得拍女朋友的美照加亲密照弄一个九宫格图文消息在朋友圈发布大肆宣传，暴击单身狗。”

我们可以把这种 Q 哥通过朋友圈发布消息，关注 65 哥的好友能收到通知的场景叫做“发布/订阅机制”。今天我们深入了解下“Redis 发布/订阅机制”的原理与实战。

Redis 通过 `SUBSCRIBE`，`UNSUBSCRIBE`和`PUBLISH`实现发布订阅消息传递模式。Redis 提供了两种模式实现该机制，分别是“发布/订阅到频道”和“发布\订阅到模式”。

### 4.4.1 发布订阅简介

Redis 发布订阅（Pus/Sub）是一种消息通信模式：发送者通过 `PUBLISH`发布消息，订阅者通过 `SUBSCRIBE` 订阅或通过`UNSUBSCRIBE` 取消订阅。

发布到订阅模式主要包含三个部分组成。

- 发布者（Publisher），发送消息到频道中，每次只能往一个频道发送一条消息。
- 订阅者（Subscriber），可以同时订阅多个频道。
- 频道（Channel），将发布者发布的消息转发给**当前**订阅此频道的订阅者。

发布者和订阅者归属于客户端，Channel 属于 Redis 服务端，发布者将消息发布到频道，订阅这个频道的订阅者则收到消息。

如下图所示，三个码哥的粉丝作为“订阅者”订阅“ChannelA”频道来表示“码哥字节”公众号。

![图 4-12][image-114]

图 4-12

码哥，写好了一篇技术文章则通过 “ChannelA” 发布消息，消息的订阅者就会收到“关注码哥字节，提升技术”的消息。

![图4-13][image-115]

图 4-13

### 4.4.2 Pub/Sub 实战

废话不多说，知道基本概念以后，学习一个技术的第一步是把它跑起来，接着才是探索原理。从而达到“知其然，知其所以然”的境界 。

一共有两种模式实现“发布\订阅”。

- 使用频道（Channel）的发布订阅。
- 使用模式（Pattern）的发布订阅。

**需要注意的是，发布订阅机制与 db 空间无关，比如在 db 10 发布， db0 的订阅者也会收到消息。**

#### 1. 通过频道（Channel）实现

三步走，家人们。

1. 订阅者订阅感兴趣的频道。
2. 发布者向特定频道发布消息。
3. 所有订阅该**频道**的订阅者收到消息。

**订阅者订阅频道**

使用 `SUBSCRIBE channel [channel ...]`命令订阅一个或者多个频道，O(n) 时间复杂度，n = 订阅的 Channel 数量。

```bash
SUBSCRIBE develop
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 消息类型
2) "develop" // 频道
3) (integer) 1 // 消息内容
```

执行该指令后，客户端就会进入订阅状态，进入该状态的客户端只能使用`subscribe`、`unsubscribe`、`psubscribe`和`punsubscribe`这四个属于"发布/订阅" 的指令。

客户端“肖菜鸡”订阅了 `develop`频道接受消息，进入订阅状态后的客户端可能会收到三种类型的回复，每种消息类型响应体都包含三个值。

第一个值是消息类型，需要注意的是：**随着消息类型的不同，对应的第二与第三部分的值表示的含义也不同。**

1. **subscribe**：订阅成功的反馈消息，第二个值是订阅成功的频道名称，第三个是当前客户端订阅的频道数量。
2. **message**：客户端接收到消息，第二个值表示产生消息的频道名称，第三个值是消息的内容。
3. **unsubscribe**：表示成功取消订阅某个频道。第二个值是对应的频道名称，第三个值是当前客户端订阅的频道数量，当此值为 0 时客户端会退出订阅状态，之后就可以执行其他非"发布/订阅"模式的命令了。

**发布者发布消息**

发布者使用 `PUBLISH channel message` 命令向指定 develop 频道发布消息，指令的响应体 “3” 表示监听该频道的客户端数量。

```bash
PUBLISH develop 'do job'
(integer) 3
```

需要注意的是，发布的**消息并不会持久化，一旦消息被 Redis 服务器发布出去，如果订阅者无法处理消息（错误或者网络断开）则消息会永远丢失。消息发布之后有新的订阅者订阅该频道的话，只能接收后续发布到该频道的消息。**好一个“不问过往，只争当下。”

**订阅者接受消息**

当有消息需要发布给订阅者时，Redis 服务端会调用 `publishMessage`函数遍历订阅者集合，将消息发送给关注了 develop 频道的订阅者。

```bash
// 订阅 develop 频道
SUBSCRIBE develop
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 订阅频道成功
2) "develop" // 频道
3) (integer) 1
// 当发布者发布消息，订阅者读取到的消息如下
1) "message" // 接受到消息
2) "develop" // 频道名称
3) "do job" // 消息内容
```

**退订频道**

订阅的反向操作，客户端使用 `UNSUBSCRIBE` 命令可以退订指定的频道。

```bash
> UNSUBSCRIBE mychannel
1) "unsubscribe"
2) "develop"
3) (integer) 0
```

#### 2. 通过模式（Pattern）实现

接下来看另一种实现发布订阅的方式，当客户端的“匹配模式“与这个频道匹配上的话，那么发布者向频道发布消息时，该消息还会发布到通过“模式”与这个频道匹配的客户端上。

比如，Chaya 和 Maggi 两个小姐姐分别在`smile.girls.Tina`、`smile.girls.maggi` 两个频道播音发布动态，她们**微笑时好美**，声音甜美又治愈。她们都有许多的粉丝，粉丝们只需要使用前面学的 `SUBSCRIBE` 订阅指定频道即可听到她们甜美的声音。

这时候，还有许多单身狗想把所有“微笑时好美”的的小姐姐都关注，那么可以使用 `smile.girl.*` 来匹配，也叫做**模式**匹配。

![图 4-14][image-116]

图 4-14

现在 Tina 通过 `smile.girls.Tina`频道发布动态，除了订阅了 `smile.girls.Tina` 这个频道的粉丝收到消息以外，这 个消息还会发送到匹配 `smile.girl.*` 模式的频道。

这些粉丝比较贪心，所有“微笑时好美的 girls”都关注了，LSP\~\~，码哥可不是这样的人。

![图 4-15][image-117]

图 4-15

**订阅模式**

订阅模式的指令是`PSUBSCRIBE`，如下表示 LSP 订阅`smile.girl.*`模式。

```bash
PSUBSCRIBE smile.girls.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe" // 消息类型
2) "smile.girls.*"// 模式
3) (integer) 1 //订阅数
```

取消订阅模式的指令是`PUNSUBSCRIBE smile.girl.*`。

喜欢 Tina 的粉丝使用 `SUBSCRIBE` 订阅 `smile.girls.Tina`频道。

```bash
SUBSCRIBE smile.girls.Tina
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "smile.girls.Tina"
3) (integer) 1
```

喜欢 maggi 的粉丝订阅 `smile.girls.maggi`频道。

```bash
SUBSCRIBE smile.girls.maggi
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "smile.girls.maggi"
3) (integer) 1
```

**发布消息**

Tina 使用 `PUBLISH smile.girls.Tina "love u"`命令发布消息，**关注 `smile.girls.Tina`频道**的粉丝和与该频道匹配的 `smile.girls.*`模式的 LSP 都会收到消息。

**接受消息**

如下是匹配 `smile.girls.*`模式的 LSP 收到的消息。

```bash
> PSUBSCRIBE smile.girls.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "smile.girls.*"
3) (integer) 1
//进入订阅状态，接收到消息
1) "pmessage" 消息类型
2) "smile.girls.*"
3) "smile.girls.Tina"
4) "love u" // 消息内容
```

而订阅 `smile.girls.Tina`频道的粉丝收到的消息如下。

```bash
> SUBSCRIBE smile.girls.Tina
Reading messages... (press Ctrl-C to quit)
// 订阅成功
1) "subscribe"
2) "smile.girls.Tina"
3) (integer) 1
// 接收消息
1) "message"
2) "smile.girls.Tina"
3) "love u"
```

需要注意的是，**如果一个客户端订阅了频道以及通过模式也匹配到同样的频道，那么客户端会收到多次消息。**

比如，65 哥订阅了`smile.girls.Tina`频道和`smile.girls.*`模式，那么当 Tina 发布消息到`smile.girls.Tina`频道的时候，65 哥会收到两条票消息，一条消息类型是`message`，一条类型是`pmessage`，消息内容是一样的。

### 4.4.3 原理分析

通过上文我们知道了发布订阅的概念，一共两种模式实现发布订阅，还运用原生指令和 Redisson 进行实战。

接下来，我们要深入理解 Redis 如何实现发布订阅机制，做到知其然知其所以然。

#### 1. 频道实现原理

> 如果是你会使用什么数据结构来实现基于频道名称来查找所有客户端？

我觉得可以使用字典 dict 数据结构来实现，字典的 key 对应被订阅的频道，而字典的值可以使用一个列表，列表里面保存着订阅这个频道的所有客户端。

##### 数据结构

聪明，Redis 使用 `redis.h`中的 `redisServer` 结构体维护每个服务器进程表示服务器状态，`pubsub_channels` 属性是一个字典，用于保存订阅频道的信息。

```c
struct redisServer {
  ...
    dict *pubsub_channels;
  ...
}
```

`pubsub_channels`：**这是一个指向 dict 类型的指针变量，key 是频道名称，value 是一个单项链表结构保存着订阅该频道的客户端。**

```c
typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

如下图 4-16 所示，码哥、谢霸哥订阅了`redis-channel`频道；宅男和 LSP 订阅了`单身派对`频道；Chaya 和李老师订阅了 `三国杀`频道。

![4-16][image-118]

图 4-16

##### 发送消息到频道

生产者调用 `PUBLISH channel messsage` 命令发送消息，Redis 最后会调用 `pubsub.c` 的`pubsubPublishMessageInternal` 函数。

1. 函数接收三个参数：`channel` 表示要发布消息的频道，`message` 表示要发布的消息内容，`type` 表示发布类型。
2. 首先根据 channel 从 `pubsub_channels` 定位到字典`dictEntry` 。
3. 调用 `dictGetVal` 获取一个 list 链表，里面保存监听该频道的所有客户端。
4. 调用 `listNext` 遍历该链表，通过`addReplyPubsubMessage`函数把消息发送给客户端

部分核心源码如下。

```c
/*
 * Publish a message to all the subscribers.
 */
int pubsubPublishMessageInternal(robj *channel, robj *message, pubsubtype type) {
    int receivers = 0;
    dictEntry *de;
    dictIterator *di;
    listNode *ln;
    listIter li;

    /* 通过 dictFind 函数查找字典 dictEntry ，里面保存着指定频道的订阅者列表。*/
    de = dictFind(*type.serverPubSubChannels, channel);
    if (de) {
        // 找到字典值的 list 列表，保存着客户端。
        list *list = dictGetVal(de);
        listNode *ln;
        listIter li;
				// 循环遍历列表中存储的每个订阅者，调用 addReplyPubsubMessage 把消息发送给客户端。
        listRewind(list,&li);
        while ((ln = listNext(&li)) != NULL) {
            client *c = ln->value;
            // 把消息发送给客户端
            addReplyPubsubMessage(c,channel,message,*type.messageBulk);
            // 更新客户端的内存使用情况和桶计数
            updateClientMemUsageAndBucket(c);
            receivers++;
        }
    }

    省略部分源码，主要是关于基于模式实现的发布订阅模式......
    return receivers;
}
```

##### 退订频道

`UNSUBSCRIBE`命令退订指定的频道：对于字典操作来说，根据 key 找到字典 value 指向的 list 列表。遍历列表，删除这个客户端，这样消息就不会发送给这个客户端了。

#### 2.模式实现原理

接下来，我们继续看基于模式实现的发布订阅原理……

当使用 `PUBLISH`发布消息到某个频道的时候，不仅订阅这个频道的所有客户端会收到消息，与这个模式匹配的客户端也会收到消息。

Redis 使用 `server.h` 文件中的`redisServer.pubsub_patterns` 属性来保存模式匹配相关信息。

```c
struct redisServer {
  ...

	dict *pubsub_patterns;
  ...
}
```

也是 dict 字典类型， key 对应`pattern`模式，value 同样也是一个链表类型的结构，存储着匹配这个模式的所有客户端。

##### 订阅模式

当执行 `PSUBSCRIBE smile.girls.*`命令订阅模式的时候，Redis 会执行`pubsubSubscribePattern`函数来实现。

1. 方法参数别分表示匹配该模式的客户端 client，和客户端想要关注的 pattern。
2. `listSearchKey(c->pubsub_patterns,pattern)`：根据 pattern 从 redisServer.pubsub\_patterns 字典查找是客户端是否已经匹配该模式，存在则调用`addReplyPubsubPatSubscribed` 通知客户端已经订阅过了，否则进入 if 分支的逻辑。
3. `dictFind(server.pubsub_patterns,pattern)`：根据模式 `pattern`从字典 `server.pubsub_patterns`找到 dictEntry 哈希桶，为空就调用 `listCreate()`创建客户端链表 `list *clients`，并放到字典中，key = pattern，value = list \*clients 链表。
4. 哈希桶不为空，那么把当前客户端 `client *c` 添加到 `list *clients`链表尾节点。

```c
int pubsubSubscribePattern(client *c, robj *pattern) {
    dictEntry *de;
    list *clients;
    int retval = 0;
    // 根据 pattern 从 redisServer.pubsub_patterns 字典查找 dictEntry
    if (listSearchKey(c->pubsub_patterns,pattern) == NULL) {
        retval = 1;
        listAddNodeTail(c->pubsub_patterns,pattern);
        incrRefCount(pattern);
        /* 将匹配模式的 client 存到存储到 dictEntry 中，key 是 pattern，value 是链表结构*/
        de = dictFind(server.pubsub_patterns,pattern);
        if (de == NULL) {
            clients = listCreate();
            dictAdd(server.pubsub_patterns,pattern,clients);
            incrRefCount(pattern);
        } else {
            clients = dictGetVal(de);
        }
        listAddNodeTail(clients,c);
    }
    /* 通知客户端已经订阅过 */
    addReplyPubsubPatSubscribed(c,pattern);
    return retval;
}
```

所以模式实现的发布订阅也是通过字典来保存模式与客户端的关系，如下图所示。

![图 4-17][image-119]

图 4-17

##### 发布消息

当使用 `PUBLISH` 命令发布消息的时候，除了发布到订阅`channel`的客户端以外，还会将消息发送给与 `pubsub_patterns` 字典中查找匹配模式 key 对应的 value 链表中的客户端。

不管是模式还是频道方式实现的发布/订阅，底层都是都是 `pubsubPublishMessageInternal` 函数实现发布，我把与频道模式相关的代码删除，方便查看。

- `dictGetIterator`：遍历存储模式订阅信息的字典 `server.pubsub_patterns`，查找到模式 pattern 以及模式对应的客户端链表。
- 调用 stringmatchlen 函数进行模式匹配，判断当前 channel 是否与模式匹配。
- 如果匹配的话，调用 `listRewind`、`listNext` 循环遍历该模式对应的订阅者列表，针对每个客户端调用 `addReplyPubsubPatMessage` 函数将模式、频道和消息回复给客户端。

```c
/*
 * Publish a message to all the subscribers.
 */
int pubsubPublishMessageInternal(robj *channel, robj *message, pubsubtype type) {
    int receivers = 0;
    dictEntry *de;
    dictIterator *di;
    listNode *ln;
    listIter li;

   ......省略频道模式的代码

    /* 遍历存储模式订阅信息的字典 server.pubsub_patterns */
    di = dictGetIterator(server.pubsub_patterns);
    if (di) {
        channel = getDecodedObject(channel);
        while((de = dictNext(di)) != NULL) {
          // 查找到模式 pattern 以及模式对应的客户端链表
            robj *pattern = dictGetKey(de);
            list *clients = dictGetVal(de);
            // 调用 stringmatchlen 函数进行模式匹配，判断当前 channel 是否与模式匹配
            if (!stringmatchlen((char*)pattern->ptr,
                                sdslen(pattern->ptr),
                                (char*)channel->ptr,
                                sdslen(channel->ptr),0)) continue;
      			// 如果匹配的话，循环遍历该模式对应的订阅者列表
            listRewind(clients,&li);
            while ((ln = listNext(&li)) != NULL) {
                client *c = listNodeValue(ln);
                // 调用 addReplyPubsubPatMessage 函数将模式、频道和消息回复给客户端
                addReplyPubsubPatMessage(c,pattern,channel,message);
                updateClientMemUsageAndBucket(c);
                receivers++;
            }
        }
        decrRefCount(channel);
        dictReleaseIterator(di);
    }
    return receivers;
}

```

#### 3. 分片（Sharded ）Pub/Sub

Sharded Pub/Sub 是从 Redis 7.0 开始引入的功能，它将分片的概念应用到了 Pub/Sub 机制中。

在 Sharded Pub/Sub 中，分片的 channel 频道会根据与分片键使用相同的算法来分配到槽位（slot）。每个分片消息都必须发送到负责该槽位的节点上。集群会确保发布的分片消息被转发到分片中的所有节点，因此客户端可以通过连接到负责该槽位的 master 或其任意 slave 来订阅分片频道，而不需要与所有节点建立连接。`SSUBSCRIBE、SUNSUBSCRIBE 和 SPUBLISH` 命令用于实现 Sharded Pub/Sub。

Sharded Pub/Sub 在集群模式下有助于扩展 Pub/Sub 的使用。它将消息的传播限制在集群的分片内部，因此通过集群总线传递的数据量相对于全局 Pub/Sub 要少。

在普通 Pub/Sub 中，每条消息都会传播到 Cluster 集群中的每个节点，而 Sharded Pub/Sub 只会在分片内部传播。

通过引入分片概念，Sharded Pub/Sub 在大规模集群环境下提供了更高效和可扩展的消息传递方式。它限制了消息传播的范围，减少了集群总线的负载，并允许用户根据需求增加分片以满足更高的 Pub/Sub 使用需求。

### 4.4.4 使用场景

> Chaya：“说了这么多，Redis 发布订阅能在什么场景发挥作用呢？”

**哨兵间通信**

哨兵集群中，每个哨兵节点利用 Pub/Sub 发布订阅实现哨兵之间的相互发现彼此和找到 Slave。

哨兵与 Master 建立通信后，利用 master 提供发布/订阅机制在`__sentinel__:hello`发布自己的信息，比如 IP、端口……，同时订阅这个频道来获取其他哨兵的信息，就这样实现哨兵间通信。

**消息队列**

之前码哥跟大家分享过如何利用 Redis List 与 Stream 实现消息队列。我们也可以利用 Redis 发布/订阅机制实现**轻量级简单的 MQ 功能**，实现上下游解耦，**需要注意点是 Redis 发布订阅的消息不会被持久化，所以新订阅的客户端将收不到历史消息。**

也不支持 ACK 机制，所以当前业务不能容忍这些缺点，那就使用专业的消息队列，如果能容忍那就能享受 Redis 快带来的优势。

## 4.5 性能必杀技之客户端缓存(Client side caching)

客户端缓存（Client side caching）技术，他的英文名叫： `Redis server-assisted client side caching` 。官方文档对其描述如下。

> 客户端缓存是一种创建极致性能服务的必杀技，在此技术下，应用程序把数据库的数据缓存在应用程序端的内存中，当应用程序访问数据库时候，可直接从本地内存读取，无序连接数据库端，减少了网络 I/O，提升应用程序的响应速度，也减少了数据库的压力。

看到这里，你会不会以为这个就像和本地缓存 Guava、Caffeine 一样，差别只是我不需要引入这个 jar 而已？

Redis 的客户端缓存可没这么简单，客户端缓存最核心的问题就是当 Redis 中的缓存变更或者失效了之后，如果能够及时有效的通知到客户端缓存，来保证数据的一致性。

今天，就聊下 Client side caching（客户端缓存），Redis 为什么需要客户端缓存、实现原理是什么，以及怎么使用。

### 4.5.1 为什么需要客户端缓存

很多公司使用 Redis 做缓存系统，缓存热点数据，提高了数据访问的性能，用来应对“吃瓜事件”。

比如，微博吃瓜事件“吴 1 凡之大方牢房”、“时间管理大师猪猪”、“思聪舔我不得就锤我”、“吴秀波之谈恋爱么，能坐牢的那种”……

#### 无客户端缓存

使用 Redis 存储热点数据，应用程序先查询 Redis，如果 Redis 没有命中缓存则从源数据库端查询，并把数据写到 Redis 缓存中；如果命中 Redis 缓存，则直接从 Redis 端查询数据。这种方式可以高效解决读取数据的业务场景。

![图 4-18][image-120]

图 4-18

虽然使用了 Redis 提升了访问性能，可还存在一些问题。Redis 缓存服务是一个独立服务存在，访问 Redis 获取数据库要经历以下几个步骤。

1. 通过网络 I/O 连接 Redis。
2. Redis 监听连接、读取 socket、执行命令。
3. 将数据序列化通过网络传输把指令执行结果传输给客户端，客户端反序列化数据。

这些操作是对性能有影响的，随着互联网的发展，流量不断的膨胀，很容易达到 Redis 的性能上限。于是乎，客户端缓存诞生了。

#### 有客户端缓存

应用端先查询本地缓存是否命中，若命中则直接把数据返回给应用程序。若没有则访问 Redis ，没有命中则查询源数据库，并把数据同步到本地缓存和 Redis 数据库中。

![图 4-19][image-121]

图 4-19

一般我们可以使用 `Memcachced`、`Guava Cache` 、`Caffeine`等来做第一级别缓存（本地缓存），使用 Redis 作为二级缓存（缓存服务），本地内存避免了网络连接、查询、网络传输、序列化等操作，性能比 Redis 服务快很多，这种模式大大减少数据延迟。

#### 客户端缓存应该保存哪些数据

- 我们不应该缓存不断变化的键。
- 我们不该缓存很少请求的键。
- 我们希望缓存经常请求并以合理速率更改的键。对于没有稳定变化速度的例子。比如，不断被`INCR`修改的全局计数器，就不应该缓存。

### 4.5.2 客户端缓存实现原理

> Chaya：“码哥，当 Redis 中的缓存变更或者失效了之后，如何有效的通知到各个应用程序进程内的缓存，来保证数据的一致性。

Redis 客户端缓存被称为 `Tracking`，可以使用以下命令来开启。

```bash
CLIENT TRACKING ON|OFF [REDIRECT client-id] [PREFIX prefix] [BCAST] [OPTIN] [OPTOUT] [NOLOOP]
```

Redis 6.0 实现 `Tracking` 功能提供了两种模式解决这个问题，分别是客户端使用 `RESP3` 协议版本的**普通模式和广播模式**，以及使用 **RESP2 协议版本的转发**模式。

#### 1. 普通模式

默认模式，当`tracking`开启时， `Redis` 会记录每个客户端访问过哪些 `key`，当 `key`的值发现变化时，服务端可以通过 `RESP3`协议发送失效信息（`invalidation message`）给客户端 。这种方式会消耗服务端的内存。

- Redis Server 端将 Client 访问的 key 以及该 key 对应的客户端 ID 列表存储在一个全局唯一的表（`TrackingTable`）。当表满了，就移除最老的记录，同时对这个客户端发送已过期的失效消息，通知进程更新本地缓存。
- 每个 Redis 客户端有一个唯一的数字 ID 标识，`TrackingTable` 存储着每一个 Client ID，当连接断开后，清除该 ID 对应的记录。
- **TrackingTable 表**中记录的 Key 信息不考虑是哪个 database 的，虽然访问的是 db1 的 key，db2 同名 key 修改时，客户端也会收到过期提示，这样做的目的是减少系统的复杂性，以及表的存储数据量。

> Chaya：“码老师，可以说下这个 TrackingTable 原理么？”

Redis 服务端使用 `TrackingTable` 表存储普通模式下的客户端，它的数据结构是前缀树 ( Radix Tree)。

前缀树是针对稀疏的整型数据查找的多叉搜索树，能快速且节省空间，关于 Radix Tree 的原理可查看 2.6 章节的内容。

Redis 用它存储**key 的指针**和**客户端 ID** 的映射关系。**key 对象的指针是内存地址，也就是一个长整型数据**。客户端缓存的变更操作就是对该数据结构的增删改查。

![图 4-20][image-122]

图 4-20

- 当 Redis 获取一个键值信息时，radix tree 会调用 `enableTracking` 函数记录 key 和 clientID 的映射关系，记录到 `TrackingTable` 中。
- 当 Redis 删除或者修改一个键值信息时。
  - radix tree 根据 key 调用 `trackingInvalidateKey` 方法查找对应的 ClinetID。
  - 调用 `sendTrackingMessage` 方法把失效的键值信息（invalidate 消息） 发送给这些 Clinet ID。
  - 发送完成之后从 TrackingTable 中删除映射关系。
- Client 关闭 track 功能后，遇到大量删除操的时候，一般是懒删除，只将 `CLIENT_TRACKING` 标志位删除。

**注意：服务端对于客户端缓存记录的 key 只会触发一次 invalidate 消息**，也就是说，服务端在某个 key 关联的客户端发送过一次 invalidate 消息后，如果 key 再被修改，此时，服务端不会再次给客户端发送 invalidate 消息。

**只有下次客户端再次执行只读命令被 track，才会进行下一次消息通知** 。

Redis 客户端默认不开启 track 模式，客户端连接到 Redis 服务后，需要先通过指令开启 tracking 模式的功能，否则无法收到失效类型的消息。

```bash
> CLIENT TRACKING ON
OK
```

#### 2. 广播模式

广播模式 (broadcasting) 开启时，服务器不会记住给定客户端访问了哪些键，因此这种模式在服务器端不再消耗过多内存，而是发送更多的失效消息给客户端，即使变更的 key 没有被该客户端缓存。

在这个模式下，服务端会给客户端广播所有 key 的失效情况，如果 key 被频繁修改，服务端会发送大量的失效广播消息，这就会消耗大量的网络带宽资源。

所以，在实际应用中，我们设置让客户端注册**只跟踪指定前缀的 key**，当注册跟踪的 key 前缀匹配被修改，服务端就会把失效消息广播给关注这个 key 前缀的客户端。

```bash
> CLIENT TRACKING ON BCAST prefix myprefix
```

我们在实际应用时，会给同一业务下的 key 设置相同的业务名前缀，所以，我们就可以非常方便地使用广播模式。

与普通模式获取一次键的规则不同，**广播模式下，只要 key 被修改或删除，符合规则的客户端都会收到失效消息，而且是可以多次获取。**

![图 4-21][image-123]

图 4-21

广播模式与普通模式类似，Redis 也是使用 Radix Tree `PrefixTable` 保存客户端订阅的 key 前缀字符串与客户端 ID 的映射关系，每个前缀字符串映射一些客户端 ID。

如果不指定前缀，客户端默认接收所有 key 的失效消息。

#### 3. 重定向模式

普通模式与广播模式，需要客户端使用 `RESP3` 协议支持，是 Redis 6.0 新启用的协议。

对于使用 `RESP2` 协议的客户端来说，实现客户端缓存则需要另一种模式重定向模式（redirect）。

Redis 服务端无法对使用`RESP2` 的客户端直接 PUSH 失效消息，所以需要另一个支持 `RESP3` 协议的客户端告诉 Server 将失效消息通过 Pus/Sub 发布订阅机制通知给 `RESP2` 客户端。

在重定向模式下，想要获得失效消息通知的客户端，需要执行订阅命令 `SUBSCRIBE`订阅用于发送失效消息的频道 `_redis_:invalidate`。

同时，再使用另外一个支持 `RESP3` 协议客户端 ，执行 `CLIENT TRACKING ON BCAST REDIRECT {clientID}`命令，设置服务端将失效消息转发给只支持使用 `RESP2` 协议的客户端。

![图 4-22][image-124]

图 4-22

比如只支持 `RESP2`协议的客户端 B 想要获取失效消息，就需要支持 `RESP3` 协议的客户端 A 告诉服务端将失效消息通过 `_redis_:invalidate`频道转发给客户端 A。我们可以分别在客户端 B 和 A 执行以下命令。

```bash
//客户端B执行，客户端 B 的 ID 号是 606
SUBSCRIBE _redis_:invalidate

//客户端 A 执行
CLIENT TRACKING ON BCAST REDIRECT 606
```

客户端 B 就可以通过 `_redis_:invalidate` 频道获取失效消息来更新本地缓存数据了。

说白了，转发模式使用额 Pub/Sub 发布订阅模式，在转发模式下，key 的作废消息只能被转发到一个客户端上。看到这里你会不会感觉这个转发模式有点鸡肋，实际业务场景很可能是多个客户端存在，只转发一个也有点坑，大家赶紧拥抱 `RESP3`协议吧。

### 4.5.3 源码解析

在 `tracking.c`源码文件中，重点关注 `trackingInvalidateKey`函数和 `sendTrackingMessage`函数。

#### trackingInvalidateKey

```c
void trackingInvalidateKey(client *c, robj *keyobj, int bcast) {
    // 1. 客户端是否执行 CLIENT TRACKING on 开启客户端缓存，没有的话直接返回。
    if (TrackingTable == NULL) return;

    unsigned char *key = (unsigned char*)keyobj->ptr;
    size_t keylen = sdslen(keyobj->ptr);

    // 2. 如果广播模式的基数数不为空，记录要广播的 key。
    if (bcast && raxSize(PrefixTable) > 0)
        trackingRememberKeyToBroadcast(c,(char *)key,keylen);
		// 3. 根据 key 去 TrackingTable 中查找元素
    rax *ids = raxFind(TrackingTable,key,keylen);
    if (ids == raxNotFound) return;

    // 4. 迭代器遍历基数树
    raxIterator ri;
    raxStart(&ri,ids);
    raxSeek(&ri,"^",NULL,0);
    while(raxNext(&ri)) {
        uint64_t id;
        memcpy(&id,ri.key,sizeof(id));
        // 4.1 根据 clientID 查找 Client 实例
        client *target = lookupClientByID(id);
        /* 4.2 如果客户端未开启 track 或者广播模式，跳过 */
        if (target == NULL ||
            !(target->flags & CLIENT_TRACKING)||
            target->flags & CLIENT_TRACKING_BCAST)
        {
            continue;
        }

        .... 省略部分代码

        /* 5. 如果目标客户端是当前客户端，并且正在执行命令。在这种情况下，需要执行 key 失效逻辑，而不是发送。 */
        if (target == server.current_client && server.fixed_time_expire) {
            incrRefCount(keyobj);
            listAddNodeTail(server.tracking_pending_keys, keyobj);
        } else {
            // 6. 发送失效消息
            sendTrackingMessage(target,(char *)keyobj->ptr,sdslen(keyobj->ptr),0);
        }
    }
    raxStop(&ri);

    /* 7. 更新TrackingTable中的总项目数，释放客户端ID集合 (ids)，并从TrackingTable中删除 key。 */
    TrackingTableTotalItems -= raxSize(ids);
    raxFree(ids);
    raxRemove(TrackingTable,(unsigned char*)key,keylen,NULL);
}

```

#### sendTrackingMessage

接着继续看发送失效消息的 `sendTrackingMessage`函数都干了啥。

```c
void sendTrackingMessage(client *c, char *keyname, size_t keylen, int proto) {
    uint64_t old_flags = c->flags;
    c->flags |= CLIENT_PUSHING;

    int using_redirection = 0;
    // 1. 开启了重定向模式
    if (c->client_tracking_redirection) {
        // 1.1 查找并切换到重定向的客户端
        client *redir = lookupClientByID(c->client_tracking_redirection);
        ....省略部分源码
    }


    if (c->resp > 2) {
       // 2. 如果是 RESP3 协议，就 push invalidate 消息给客户端
        addReplyPushLen(c,2);
        addReplyBulkCBuffer(c,"invalidate",10);
    } else if (using_redirection && c->flags & CLIENT_PUBSUB) {
        /* 3. 转发模式则使用 Pub/Sub机制 TrackingChannelName channel 中发送消息 */
        addReplyPubsubMessage(c,TrackingChannelName,NULL,shared.messagebulk);
    } else {
        /* 4. 客户端既没有使用 RESP3 协议，也没有重定向到另一个客户端，直接 return */
        if (!(old_flags & CLIENT_PUSHING)) c->flags &= ~CLIENT_PUSHING;
        return;
    }

    /* Send the "value" part, which is the array of keys. */
    if (proto) {
        addReplyProto(c,keyname,keylen);
    } else {
        addReplyArrayLen(c,1);
        addReplyBulkCBuffer(c,keyname,keylen);
    }
    // 4. 更新客户端的内存使用情况。
    updateClientMemUsageAndBucket(c);
    if (!(old_flags & CLIENT_PUSHING)) c->flags &= ~CLIENT_PUSHING;
}
```

## 4.6 性能必杀技之 Redis I/O 多线程模型

在 2.11 章节《Redis 为什么这么快》中我们已经知道 Redis 使用全局 dict 字典表 + 内存数据库 + 丰富高效的数据结构 + 单线程模型 + I/O 多路复用事件驱动框架使得 Redis 快到飞起。

Redis 的网络 I/O 以及键值对指令读写是由单个线程来执行的，避免了不必要的上再问切换和资源竞争，对于性能提升有很大的帮助。

然而，Redis 官方在 2020 年 5 月正式推出 6.0 版本，引入了 I/O 多线程模型。

> 谢霸哥：“为什么之前是单线程模型？为什么 6.0 引入了 I/O 多线程模型？主要解决了什么问题？”

今天，咱们就详细的聊下 I/O 多线程模型带来的效果到底是黛玉骑鬼火，该强强，该弱弱；还是犹如光明顶身怀绝技的的张无忌，招招都是必杀技。

### 4.6.1 单线程模型真的只有一个线程么？

> 谢霸哥：“码哥， Redis 6.0 之前单线程指的是 Redis 只有一个线程干活么？”

非也，我们通常说的单线程模型指的是 Redis 在处理客户端的请求时，包括获取 (socket 读)、解析、执行、内容返回 (socket 写) 等都由一个顺序串行的主线程处理。

而其他的清理过期键值对数据、释放无用连接、内存淘汰策略执行、`BGSAVE` 生成 RDB 内存快照文件、AOF rewrite 等都是其他线程处理。

命令执行阶段，每一条命令并不会立马被执行，而是进入一个一个 socket 队列，当 socket 事件就绪则交给事件分发器分发到对应的事件处理器处理，单线程模型的命令处理如下图所示。

![图 4-23][image-125]

图 4-23

### 4.6.2 线程模型的演化

> 谢霸哥：“为什么 Redis6.0 之前是单线程模型？”

以下是官方关于为什么 6.0 之前一直使用单线程模型的回答。

- Redis 的性能瓶颈主要在于内存和网络 I/O，CPU 不会是性能瓶颈所在。
- Redis 通过使用 `pipelining` 每秒可以处理 100 万个请求，应用程序的所时候用的大多数命令时间复杂度主要使用 O(N) 或 O(log(N)) 的，它几乎不会占用太多 CPU。
- 单线程模型的代码可维护性高。多线程模型虽然在某些方面表现优异，但是它却引入了程序执行顺序的不确定性，带来了并发读写的一系列问题，增加了系统复杂度、同时可能存在线程切换、甚至加锁解锁、死锁造成的性能损耗。

Redis 通过基于 I/O 多路复用实现的 AE 事件驱动框架将 I/O 事件和事件事件融合在一起，实现高性能网络处理能力，再加上基于内存的数据处理，没有引入多线程的必要。

而且**单线程机制让 Redis 内部实现的复杂度大大降低，Hash 的惰性 Rehash、Lpush 等等线程不安全的命令都可以无锁进行**。

> 谢霸哥：“既然单线程这么好，为什么 6.0 版本引入多线程模型？”

因为随着底层网络硬件性能提升，Redis 的性能瓶颈逐渐体现在网络 I/O 的读写上，**单个线程处理网络读写的速度跟不上底层网络硬件执行的速度**。

因为读写网络的 read/write 系统调用占用了 Redis 执行期间大部分 CPU 时间。所以 Redis 采用多个 I/O 线程来处理网络请求，提高网络请求处理的并行度。

**需要注意的是，Redis 多 IO 线程模型只用来处理网络读写请求，对于 Redis 的读写命令，依然是单线程处理**。

这是因为，**网络 I/O 读写是瓶颈，可通过多线程并行处理可提高性能。而继续使用单线程执行读写命令，不需要为了保证 Lua 脚本、事务、等开发多线程安全机制，实现更简单。**

> 谢霸哥：“码哥，你真是斑马的脑袋，说的头头是道。”

我谢谢您嘞，主线程与 I/O 多线程共同协作处理命令的架构图如下所示。

![图 4-24][image-126]

图 4-24

### 4.6.3 I/O 多线程模型解读

> 谢霸哥：“如何开启多线程呢？”

Redis 6.0 的多线程默认是禁用的，如需开启需要修改 `redis.conf` 配置文件的配置`io-threads-do-reads yes`。

开启多线程后，还要设置线程数才能生效，同样是修改 `redis.conf`配置文件。

```bash
io-threads 4
```

> 谢霸哥：“码老师，线程数是不是越多越好？”

当然不是，关于线程数的设置，官方有一个建议：线程数的数量最好小于 CPU 核心数，起码预留一个空闲核处理，因为 Redis 是主线程处理指令，如果系统出现频繁上下文切换，效率会降低。

比如 4 核的机器建议设置为 2 或 3 个线程，8 核的机器建议设置为 6 个线程，线程数一定要小于机器核数。

> 谢霸哥：“码老师真厉害，就好像卖盆的进村一套一套的。我什么时候也能像你这样连贯又有逻辑的掌握 Redis。”

认真读 Redis 高手心法，长线放风筝慢慢来。

> 谢霸哥：“主线程与 I/O 线程是如何实现协作呢？”

![图 4-25][image-127]

图 4-25

**主要流程**。

1. 主线程负责接收建立连接请求，通过轮询将可读 `socket` 分配给 I/O 线程绑定的等待队列。
2. 主线程**阻塞等待**，直到 I/O 线程完成 `socket` 读取和解析。
3. 主线程执行 I/O 线程读取和解析出来的 Redis 请求命令。
4. 主线程**阻塞等待** I/O 线程将指令执行结果回写回 `socket`完毕。
5. 主线程清空等待队列，等待下一次客户端后续的请求。

思路：**将主线程 IO 读写任务拆分出来给一组独立的线程处理，使得多个 socket 读写可以并行化，但是 Redis 命令还是主线程串行执行。**

大家注意第三和第五步，主线程并不是挂起线程让出 CPU 分片时间。而是通过 for 循环进行忙等，不断的检测所有 I/O 线程处理任务是否已经完成，完成再执行下一步。

#### 源码解析

看完流程图以及主要步骤，接着跟着源码走一个。通过 4.3 章节的学习，我们知道 Redis 是通过 `server.c`的`main`函数启动的，经过一系列的初始化操作后，调用 `aeMain(server.el);`启动事件驱动框架，也就是整个 Redis 的核心。

##### 初始化线程

I/O 多线程模型的开端也是由 `server.c`的`main`方法中的 `InitServerLast`来初始化，该方法内部会调用 `networking.c` 的 `initThreadedIO`来执行实际 I/O 线程初始化工作。

```c
/* networking.c */
void initThreadedIO(void) {
   // 设置成 0 表示激活 I/O 多线程模型
    server.io_threads_active = 0;
    /* I/O 线程处于空闲状态 */
    io_threads_op = IO_THREADS_OP_IDLE;

    /* 如果 redis.conf 的 io-threads 配置为 1 表示使用单线程模型，直接退出 */
    if (server.io_threads_num == 1) return;

    // 线程数超过最大值 128，退出程序
    if (server.io_threads_num > IO_THREADS_MAX_NUM) {
        ....省略
        exit(1);
    }

    for (int i = 0; i < server.io_threads_num; i++) {
        /*  io_threads_list 链表，用于存储该线程要执行的 I/O 操作。*/
        io_threads_list[i] = listCreate();
        // 0 号线程不创建，0 号就是主线程，主线程也会处理任务逻辑。
        if (i == 0) continue;

        // 创建线程，主线程先对子线程上锁，挂起子线程，不让其进入工作模式
        pthread_t tid;
        pthread_mutex_init(&io_threads_mutex[i],NULL);
        setIOPendingCount(i, 0);
        // 挂起子线程，先不进入工作模式，等待主线程发出干活信号再执行任务。
        pthread_mutex_lock(&io_threads_mutex[i]);
        // 创建线程，指定I/O线程的入口函数 IOThreadMain
        if (pthread_create(&tid,NULL,IOThreadMain,(void*)(long)i) != 0) {
            serverLog(LL_WARNING,"Fatal: Can't initialize IO thread.");
            exit(1);
        }
        // I/O 线程数组
        io_threads[i] = tid;
    }
}
```

1. 检查是否开启 I/O 多线程模型：默认不激活 I/O 多线程模型，当 redis.conf 的 io-threads 配置大于 1 并且小于 `IO_THREADS_MAX_NUM`（128） 则表示开启 I/O 多线程模式。
2. 创建 `io_threads_list` 链表，用于保存每个线程需要处理的 I/O 任务。
3. 创建子线程，创建的时候先上锁，挂起子线程不让其进入工作模式，等初始化工作完成再开启。
4. 指定 I/O 线程的入口函数 `IOThreadMain`，I/O 线程开始工作。

##### I/O 线程核心函数

`IOThreadMain` 函数主要负责等待启动信号、执行特定的 I/O 操作，并在完成操作后重置线程状态，以便再次等待下一次启动信号。

```c
void *IOThreadMain(void *myid) {
    /* 每个线程创建一个 id */
    long id = (unsigned long)myid;
    char thdname[16];
    .....
    // 进入无限循环，等待主线程发出干活信号
    while(1) {
        /* 没有使用 sleep 设置等待时间实现忙等，而是循环，耗费 CPU*/
        for (int j = 0; j < 1000000; j++) {
            // 等待待处理的 I/O 操作出现，也就是读写客户端数据
            if (getIOPendingCount(id) != 0) break;
        }

        /*留机会给主线程上锁，挂起当前子线程 */
        if (getIOPendingCount(id) == 0) {
            pthread_mutex_lock(&io_threads_mutex[id]);
            pthread_mutex_unlock(&io_threads_mutex[id]);
            continue;
        }

        serverAssert(getIOPendingCount(id) != 0);

        /* 根据线程 id 以及待分配列表进行任务分配 */
        listIter li;
        listNode *ln;
        listRewind(io_threads_list[id],&li);
        while((ln = listNext(&li))) {
            client *c = listNodeValue(ln);
            if (io_threads_op == IO_THREADS_OP_WRITE) {
                // 将可写客户端任务分配
                writeToClient(c,0);
            } else if (io_threads_op == IO_THREADS_OP_READ) {
                // 读取客户端 socket 数据
                readQueryFromClient(c->conn);
            } else {
                serverPanic("io_threads_op value is unknown");
            }
        }

        listEmpty(io_threads_list[id]);
        setIOPendingCount(id, 0);
    }
}
```

##### 待读取客户端任务分配

Redis 会在主线程 `initServer` 初始化服务器的时候会注册 `beforeSleep`函数，里面会调用 `handleClientsWithPendingReadsUsingThreads`函数实现待处理任务分配逻辑。该函数的主要作用如下。

- 将所有待读的客户端平均分配到不同的 I/O 线程的列表中。
- 通过设置 `io_threads_op` 和调用 `setIOPendingCount` 函数，通知各个 I/O 线程开始处理可读取的客户端数据。
- 主线程也参与处理客户端读取，以确保更好的并发性能。
- 主线程等待所有 I/O 线程完成读取 socket 工作。

**这种设计采用了“扇出 -\> 扇入”的范式，通过将工作分发到多个 I/O 线程，再将结果合并回主线程，以提高并发性能。**

```c
int handleClientsWithPendingReadsUsingThreads(void) {
    ......

    /* 将所有待处理的客户端平均分配到不同的 I/O 线程的列表中*/
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_read,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }

    /* 通过设置 `io_threads_op` 和调用 `setIOPendingCount` 函数，通知各个 I/O 线程开始处理可读取的客户端数据。 */
    io_threads_op = IO_THREADS_OP_READ;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        setIOPendingCount(j, count);
    }

    /* 主线程处理第一个等待队列任务 */
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        readQueryFromClient(c->conn);
    }
    listEmpty(io_threads_list[0]);

    /* 主线程处理完任务后，忙等等待所有 I/O 线程完成读取 socket 工作 */
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }

    ......

    return processed;
}
```

##### 待写回客户端任务分配

与上面类似， `beforeSleep`函数里面会调用 `handleClientsWithPendingWritesUsingThreads`函数实现可写客户端处理任务分配给 I/O 线程，源代码跟 `handleClientsWithPendingReadsUsingThreads`类似，不贴了。差别就是这个函数处理的事情是把响应写回 socket。

- 将所有待写的客户端平均分配到不同的 I/O 线程的列表中。
- 设置 `io_threads_op` 为 `IO_THREADS_OP_READ`通知各个 I/O 线程开始处理可写的客户端数据。
- 主线程也参与处理客户端读取，以确保更好的并发性能。
- 主线程等待所有 I/O 线程完成读取 socket 工作。

#### 模型缺陷

Redis 的多线程网络模型实际上并不是一个标准的 Multi-Reactors/Master-Workers 模型，I/O 线程任务仅仅是通过 socket 读取客户端请求命令并解析，以及把指令执行结果回写给 socket ，没有真正去执行命令。

所有客户端命令最后还需要回到主线程去执行，因此对多核的利用率并不算高，而且每次主线程都必须在分配完任务之后忙轮询等待所有 I/O 线程完成任务之后才能继续执行其他逻辑。

在我看来，Redis 目前的多线程方案更像是一个折中的选择，只是黛玉骑鬼火，还未达到必杀技的阶段。

## 4.7 Redis 内存碎片深度解析与优化策略

通过 `CONFIG SET maxmemory 100mb`或者在 `redis.conf` 配置文件设置 `maxmemory 100mb` 限制 Redis 内存占用。当达到内存最大值值，会触发**内存淘汰策略**删除数据。

除此之外，当 key 达到过期时间，Redis 会通过以下两种删除过期数据。

- 后台定时任务选取部分数据删除。
- 惰性删除。

> 肖材积：“码哥，生产上 Redis 保存了 5GB 的数据，我现在删除了 2GB 数据，为什么进程还是占用了 5GB 内存？”

删除了数据， Redis 进程占用的内存（也叫做 RSS，进程消耗内存页数））不一定会降低。在长时间运行过程中，可能会面临内存碎片问题。本章将深入解析 Redis 内存碎片的成因，以及针对性的优化策略。

### 4.7.1 数据已删，释放的内存去哪了？

> 肖材积：“明明删除了数据，使用 top 命令查看 Redis 进程占用却发现没有降低，内存都去哪了？”

使用 `info memory` 命令获取 Redis 内存相关指标，我列举了几个重要的指标数据。

```bash
127.0.0.1:6379> info memory
# Memory
// Redis 存储数据占用的内存量
used_memory:1132832
// 可读友好形式返回内存总量
used_memory_human:1.08M
// 操作系统角度，进程占用的物理总内存
used_memory_rss:2977792
// used_memory_rss 可读性模式展示
used_memory_rss_human:2.84M
// 内存使用的最大值，表示 used_memory 的峰值
used_memory_peak:1183808

// 以可读的格式返回 used_memory_peak的值
used_memory_peak_human:1.13M
// Lua 引擎所消耗的内存大小。
used_memory_lua:37888
used_memory_lua_human:37.00K
// Redis 能使用的最大内存值，字节为单位。
maxmemory:2147483648
// Redis 能使用的最大内存值
maxmemory_human:2.00G
// 内存淘汰策略
maxmemory_policy:noeviction // 内存淘汰策略

// used_memory_rss / used_memory 的比值，代表内存碎片率
mem_fragmentation_ratio:2.79
```

Redis 进程内存消耗主要由以下部分组成。

- Redis 自身启动所占用的内存。
- 存储对象数据内存。
- 缓冲区内存：主要由 client-output-buffer-limit 客户端输出缓冲区、复制积压缓冲区（backlog\_buffer）、AOF 缓冲区。
- 内存碎片。

![4-26][image-128]

图 4-26

Redis 进程占用的内存很小可以忽略不计，存储数据对象内存是占比最大的一块，里面存储着所有的数据。

缓冲区内存在大流量场景容易失控，造成 Redis 内存不稳定，需要重点关注。

**内存碎片过大会导致明明有空间可用，但是却无法存储数据。**

**碎片率 = used\_memory\_rss 实际使用的物理内存（RSS 值）除以 used\_memory 实际存储数据内存。**

### 4.7.2 什么是内存碎片？

内存碎片是指内存空间中未被利用的小块空闲区域，它们由于大小不一致或位置不连续，而无法被有效地分配给新的数据存储。内存碎片会占用 Redis 的物理内存，但是并不计入 Redis 的逻辑内存。

内存碎片率越大，说明内存碎片越多，内存利用率越低。

举个例子，你跟漂亮小姐姐去电影院看电影，肯定想连在一块坐。

假设有 8 个座位，已经卖出了 4 张票，还有 4 张可以买。可是好巧不巧，买票的人很奇葩，分别间隔一个座位买票。

即使还有 4 个座位空闲，可是你却买不到两个座位连在一块的票，厚礼蟹！

![图 4-27][image-129]

图 4-27

### 4.7.3 内存碎片的形成原因

主要有两个原因：

- 内存分配器的分配策略。
- 频繁的数据更新操作：Redis 频繁对大小不同的键值对做更新操作、大量过期数据删除，释放的空间不够连续导致无法得到复用。

#### 1. 内存分配器的分配策略

Redis 默认的内存分配器采用 jemalloc，可选的分配器还有 glibc、tcmalloc。

**jemalloc 内存分配是按照固定大小的块为单位进行连续内存分配的，而不是按需分配的。**

例如 8 字节、16 字节…..，2 KB，4KB，当申请内存最近接某个固定值的时候，jemalloc 会给它分配最接近固定值大小的空间，这样就会出现内存碎片。

比如程序只需要 1.5 KB，内存分配器会分配 2KB 空间，那么这 0.5KB 就是碎片。

**这么做的目的是减少内存分配次数**，比如申请 22 字节的空间保存数据，jemalloc 就会分配 32 字节，如果后边还要写入 10 字节，就不需要再向操作系统申请空间了，可以使用之前申请的 32 字节。

> Chaya：“Redis 不会立马把内存归还给操作系统的原因是什么？”

**删除 key 的时候，Redis 并不会立马把内存归还给操作系统**，出现这个情况是因为底层内存分配器管理机制，比如大多数已经删除的 key 依然与其他有效的 key 分配在同一个内存页中。

除此之外，**分配器为了复用空闲的内存块：**原有 5GB 的数据中删除了 2 GB 后，当再次添加数据到实例中，复用了之前删除释放出来的 2GB 内存。Redis 的 RSS 会保持稳定，不会增长太多。

#### 2. 键值对大小不一和删改操作

Redis 频繁对大小不同的键值对做更新操作、大量过期数据删除，释放的空间不够连续导致无法得到复用。

键值对的频繁修改和删除，导致内存空间的扩容和释放，比如原本占用 32 字节的字符串，现在修改为占用 20 字节的字符串，那么释放出的 12 字节就是空闲空间。

如果下一个数据存储请求需要申请 13 字节的字符串，那么刚刚释放的 12 字节空间无法使用，导致碎片。

**碎片最大的问题：空间总量足够大，但是这些内存不是连续的，可能大致无法存储数据。**

### 4.7.4 内存碎片解决之道

> Chaya：“如何判断是否有内存碎片呢？”

Redis 提供了`info memory`命令，可以查看 Redis 的内存使用情况，其中有一个字段`mem_fragmentation_ratio`，表示 Redis 的内存碎片率。它的计算公式是：

`mem_fragmentation_ratio = used_memory_rss / used_memory`

其中，`used_memory_rss`表示操作系统实际分配给 Redis 的物理内存空间，里面包含了碎片；`used_memory`表示 Redis 为了保存数据实际申请使用的空间。如果`mem_fragmentation_ratio`大于 1，那么就说明存在内存碎片，这个值越大，内存碎片就越多。如果 1 \< 碎片率 \< 1.5，可以认为是合理的，而大于 1.5 说明碎片已经超过 50%，我们需要采取一些手段解决碎片率过大的问题。

#### 1. 重启大法

Redis 4.0 之前的版本，没有内置内存碎片清理工具，只能通过重启的方式来清理内存碎片。

如果没有开启持久化，数据会丢失；开启持久化的话，需要使用 RDB 或者 AOF 恢复数据，如果只有一个实例，数据大的话会导致恢复阶段长时间无法提供服务，高可用大打折扣。

#### 2. 自动清理内存碎片

> Chaya：“咋办呢？码哥靓仔”

既然你都叫我靓仔了，就倾囊相助告诉你终极杀招：Redis 4.0 之后，提供了一个内存碎片自动清理的功能，可以在不重启的情况下，自动进行碎片清理。

这种方法的原理是，当一块连续的内存空间被划分为好几块不连续的空间的时候，操作系统先把数据依次挪动拼接在一块，并释放原来数据占据的空间，形成一块连续空闲内存空间，从而提高内存利用率。需要注意的是，**Redis 的内存分配器是 jemalloc 才能启用内存碎片自动清理**。

![图 4-28][image-130]

图 4-28

##### 如何开启和配置内存碎片自动清理？

首先，你需要确定 Redis 的内存分配器是否是 jemalloc，可以通过`info memory`命令查看`mem_allocator`字段。如果不是 jemalloc，你需要重新编译 Redis，并指定`MALLOC=jemalloc`参数。

接着，可以通过如下指令动态修改 Redis 配置，而不需要重启 Redis。

```bash
CONFIG SET activedefrag yes
```

如果向永久开启这个功能，就需要修改 `redis.conf`配置文件，配置 `activedefrag yes`。

这只是开启自动清理，我们可以根据需要调整一些参数，来控制碎片清理的触发条件和速度。

##### 清理的触发条件

`active-defrag-ignore-bytes 200mb`：表示内存碎片占用的内存达到 200MB，开始清理。默认值是 100MB。

`active-defrag-threshold-lower 20`：表示内存碎片占用操作系统分配给 Redis 总空间的比例达到 20% 时，开始清理。默认值是 10%。

##### 清理的速度

自动清理虽好，可不要肆意妄为，操作系统把数据移动到新位置，再把原有空间释放是需要消耗资源的。

**Redis 操作数据的指令是单线程，所以在数据复制移动的时候，只能等待清理碎片完成才能处理请求，造成性能损耗。**

通过以下两个参数来控制内存碎片清理和结束时机，避免占用 CPU 过多，减少清理碎片对 Redis 处理请求的性能影响。

- `active-defrag-cycle-min 20`：表示自动清理过程所用 CPU 时间的比例不低于 20%，保证清理能正常开展。默认值是 5%。
- `active-defrag-cycle-max 50`：自动清理过程占用的 CPU 时间比例不能高于 50%，超过的话就立刻停止清理，避免对 Redis 的阻塞，造成高延迟。默认值是 75%，可以根据实际情况调整。

这些参数可以通过`config set`命令动态地修改，也可以通过修改配置文件永久地修改。

# 第 5 章 元婴大成出师实战

## 5.1 Redis 性能排查与解决问题的终极 Checklist

Redis 通常是我们业务系统中一个重要的组件，比如：缓存、账号登录信息、排行榜等。一旦 Redis 请求延迟增加，可能就会导致业务系统“雪崩”。

我在单身红娘婚恋类型互联网公司工作，在双十一推出下单就送女朋友的活动。谁曾想，凌晨 12 点之后用户量暴增，系统出现故障，用户无法下单，老板们上蹿下跳！

经过查找发现 Redis 报 `Could not get a resource from the pool`的错误：获取不到连接资源，并且集群中的单台 Redis 连接量很高。大量的流量没了 Redis 的缓存响应，直接打到了 MySQL，最后数据库也宕机了……

于是各种更改 Redis 最大连接数、连接等待数，虽然报错信息频率有所缓解，但还是**持续报错**。后来经过线下测试，发现 Redis 中某些 key 的 value**字符数据很大，平均 1s 才能返回数据**。今天码哥跟大家一起来分析下 Redis 突然变慢了我们该怎么办？如何确定 Redis 性能出问题了，出现问题要如何调优解决。

### 5.1.1 性能基线测量

最大延迟是指从客户端发出命令到客户端收到命令响应的时间，通常情况下，Redis 能够在微秒级别内快速处理。

然而，在 Redis 性能波动的情况下，延迟可能会增加到几秒甚至十几秒，这时我们明显可以判断 Redis 的性能出现了问题。

对于高端硬件配置，当延迟达到 0.6 毫秒时，我们可能会认为 Redis 性能降低。而在硬件配置相对较差的情况下，我们可能需要延迟达到 3 毫秒才会认为出现性能问题。

> 张无剑：“那么，我们如何判定 Redis 真的变慢呢？”

因此，我们需要对当前环境下的**Redis 基线性能**进行测量，即在系统低压力、无干扰的条件下，获取其基本性能水平。

**当你观察到 Redis 运行时延迟超过基线性能的两倍以上时，可以明确判定 Redis 性能已经下降。**

redis-cli 可执行脚本提供了 `–intrinsic-latency` 选项，用来监测和统计测试期间内的最大延迟（以毫秒为单位），这个延迟可以作为 Redis 的基线性能。

**需要注意的是，你需要在运行 Redis 的服务器上执行，而不是在客户端中执行。**

```sh
./redis-cli --intrinsic-latency 100
Max latency so far: 4 microseconds.
Max latency so far: 18 microseconds.
Max latency so far: 41 microseconds.
Max latency so far: 57 microseconds.
Max latency so far: 78 microseconds.
Max latency so far: 170 microseconds.
Max latency so far: 342 microseconds.
Max latency so far: 3079 microseconds.

45026981 total runs (avg latency: 2.2209 microseconds / 2220.89 nanoseconds per run).
Worst run took 1386x longer than the average latency.
```

> 注意：参数`100`是测试将执行的秒数。我们运行测试的时间越长，我们就越有可能发现延迟峰值。
> 
> 通常运行 100 秒通常是合适的，足以发现延迟问题了，当然我们可以选择不同时间运行几次，避免误差。

我运行的最大延迟是 3079 微秒，所以基线性能是 3079 （3 毫秒）微秒。

需要注意的是，我们要在 Redis 的服务端运行，而不是客户端。这样，可以**避免网络对基线性能的影响**。

此外，可以通过 `-h host -p port` 来连接服务端，如果想监测网络对 Redis 的性能影响，可以使用 Iperf 测量客户端到服务端的网络延迟。

如果网络延迟几百毫秒，说明网络可能有其他大流量的程序在运行导致网络拥塞，需要找运维协调网络的流量分配。

### 5.1.2 慢指令监控

> Chaya：“知道了性能基线后，有什么监控手段知道有慢指令呢？”

我们要避免使用时间复杂度为 `O(n)`的指令，尽可能使用`O(1)`和`O(logN)`的指令。

涉及到集合操作的复杂度一般为`O(N)`，比如集合**全量查询**`HGETALL、SMEMBERS`，以及集合的**聚合操作** `SORT`、`LREM`、 `SUNION` 等。

> Chaya：“代码不是我写的，不知道有没有人用了慢指令，有没有监控呢？”

有两种方式可以排查到。

- 使用 Redis 慢日志功能查出慢命令。
- latency-monitor（延迟监控）工具。

此外，使用 Linux 指令（top、htop、prstat 等）快速检查 Redis 主进程的 CPU 消耗。如果 CPU 使用率很高而流量不高，通常表明使用了慢命令。

#### 慢日志功能

Redis 中的 `slowlog` 命令可以让定位到那些超出指定执行时间的慢命令，默认情况下命令若是执行时间超过 10ms 就会被记录到日志。

slowlog 只会记录其命令执行的时间，不包含 I/O 往返操作时间，也不记录单由网络延迟引起的响应时间。

我们可以**根据基线性能来自定义慢命令的标准（配置成基线性能最大延迟的 2 倍）**，调整触发记录慢命令的阈值。

可以在 redis-cli 中输入以下命令配置记录 6 毫秒以上的指令。

```bash
redis-cli CONFIG SET slowlog-log-slower-than 6000
```

也可以在 Redis.conf 配置文件中设置，以微秒为单位。想要查看所有执行时间比较慢的命令，可以通过使用 Redis-cli 工具，输入 `SLOWLOG GET` 命令查看。

如下查看最后 2 个慢命令，输入 `SLOWLOG GET 2`即可。

```bash
示例：获取最近2个慢查询命令
> SLOWLOG get 2
1) 1) (integer) 6
   2) (integer) 1458734263
   3) (integer) 74372
   4) 1) "HGETALL"
      2) "max.magebyte.blacklist"
2) 1) (integer) 5
   2) (integer) 1458734258
   3) (integer) 5411075
   4) 1) "KEYS"
      2) "max.magebyte.blacklist"

```

以第一个 `HGET` 命令响应数据为例分析，每个 slowlog 实体共 4 个字段：

- 字段 1，1 个整数，表示这个 slowlog 出现的序号，server 启动后递增，当前为 6。
- 字段 2：表示查询执行时的 Unix 时间戳。
- 字段 3，表示查询执行微秒数，当前是 74372 微秒，约 74ms。
- 字段 4，表示指令的命令和参数，如果参数很多或很大,只会显示部分并给数参数个数。当前命令是`HGETALL max.magebyte.blacklist`。

#### Latency Monitoring

Redis 在 2.8.13 版本引入了 Latency Monitoring 功能，用于以秒为粒度监控各种事件的发生频率。

启用延迟监视器的第一步是**设置延迟阈值(单位毫秒)**。只有超过该阈值的时间才会被记录，比如我们根据基线性能（3ms）的 3 倍设置阈值为 9 ms，可以用 redis-cli 设置也可以在 Redis.config 中设置。

```bash
CONFIG SET latency-monitor-threshold 9
```

如获取最近的 latency

```shell
127.0.0.1:6379> debug sleep 2
OK
(2.00s)
127.0.0.1:6379> latency latest
1) 1) "command"
   2) (integer) 1645330616
   3) (integer) 2003
   4) (integer) 2003
```

1. 事件的名称；
2. 事件发生的最新延迟的 Unix 时间戳。
3. 毫秒为单位的时间延迟。
4. 该事件的最大延迟。

### 5.1.3 解决性能问题的终极 Checklist

Redis 的指令由单线程执行，如果主线程执行的操作时间太长，就会导致主线程阻塞。一起分析下都有哪些情况会导致 Redis 性能问题，我们又该如何解决。

#### 1. 网络通信导致的延迟

客户端使用 TCP/IP 连接或 Unix 域连接连接到 Redis。1 Gbit/s 网络的典型延迟约为 200 us。

redis 客户端执行一条命令分 4 个过程：

> 发送命令－〉 命令排队 －〉 命令执行－〉 返回结果

这个过程称为 Round trip time(简称 RTT, 往返时间)，mget mset 有效节约了 RTT，但大部分命令（如 hgetall，并没有 mhgetall）不支持批量操作，需要消耗 N 次 RTT ，这个时候需要 pipeline 来解决这个问题。

**解决方案**

Redis pipeline 将多个命令连接在一起来减少网络响应往返次数。

![图 5-1][image-131]

图 5-1

#### 2. 慢指令

根据上文的慢指令监控到慢查询的指令。可以通过以下两种方式解决。

- 在 Cluster 集群中，将聚合运算等 O(N) 时间复杂度操作放到 slave 上运行或者在客户端完成。
- 使用更高效的命令代替。比如使用增量迭代的方式，避免一次查询大量数据，具体请查看 `SCAN、SSCAN、HSCAN、ZSCAN`命令。

除此之外，生产中禁用 `KEYS` 命令，因为它会遍历所有的键值对，所以操作延时高，只适用于调试。

#### 3. 开启透明大页（Transparent HugePages）

常规的内存页是按照 `4 KB` 来分配，Linux 内核从 2.6.38 开始支持内存大页机制，该机制支持 `2MB` 大小的内存页分配。

Redis 使用 fork 生成 RDB 快照的过程中，Redis 采用**写时复制**技术使得主线程依然可以接收客户端的写请求。

也就是当数据被修改的时候，Redis 会复制一份这个数据，再进行修改。

采用了内存大页，生成 RDB 期间即使客户端修改的数据只有 50B 的数据，Redis 可能需要复制 2MB 的大页。当写的指令比较多的时候就会导致大量的拷贝，导致性能变慢。

**使用以下指令禁用 Linux 内存大页即可解决。**

```shell
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

#### 4. swap 交换区

> 谢霸哥：“什么是 swap 交换区？”

当物理内存不够用的时候，操作系统会将部分内存上的数据交换到 swap 空间上，防止程序因为内存不够用而导致 oom 或者更致命的情况出现。

当应用进程向操作系统请求内存发现不足时，操作系统会把内存中暂时不用的数据交换放在 SWAP 分区中，这个过程称为 SWAP OUT。

当该进程又需要这些数据且操作系统发现还有空闲物理内存时，就会把 SWAP 分区中的数据交换回物理内存中，这个过程称为 SWAP IN。

**内存 swap 是操作系统里将内存数据在内存和磁盘间来回换入和换出的机制，涉及到磁盘的读写。**

> 谢霸哥：“触发 swap 的情况有哪些呢？”

对于 Redis 而言，有两种常见的情况。

- Redis 使用了比可用内存更多的内存。
- 与 Redis 在同一机器运行的其他进程在执行大量的文件读写 I/O 操作（包括生成大文件的 RDB 文件和 AOF 后台线程），文件读写占用内存，导致 Redis 获得的内存减少，触发了 swap。

> 谢霸哥：“我要如何排查因为 swap 导致的性能变慢呢？”

Linux 提供了很好的工具来排查这个问题，当你怀疑由于交换导致的延迟时，只需按照以下步骤排查。

**获取 Redis pid**

我省略部分指令响应的信息，重点关注 process\_id。

```bash
127.0.0.1:6379> INFO Server
# Server
redis_version:7.0.14
process_id:2847
process_supervised:no
run_id:8923cc83412b223823a1dcf00251eb025acab271
tcp_port:6379
```

##### **查找内存布局**

进入 Redis 所在的服务器的 /proc 文件系统目录。

```shell
cd /proc/2847
```

在这里有一个 smaps 的文件，该文件描述了 Redis 进程的内存布局，用 grep 查找所有文件中的 Swap 字段。

```sh
$ cat smaps | egrep '^(Swap|Size)'
Size:                316 kB
Swap:                  0 kB
Size:                  4 kB
Swap:                  0 kB
Size:                  8 kB
Swap:                  0 kB
Size:                 40 kB
Swap:                  0 kB
Size:                132 kB
Swap:                  0 kB
Size:             720896 kB
Swap:                 12 kB
```

**每行 Size 表示 Redis 实例所用的一块内存大小，和 Size 下方的 Swap 对应这块 Size 大小的内存区域有多少数据已经被换出到磁盘上了，如果 Size == Swap 则说明数据被完全换出了。**

可以看到有一个 720896 kB 的内存大小有 12 kb 被换出到了磁盘上（仅交换了 12 kB），这就没什么问题。

Redis 本身会使用很多大小不一的内存块，所以，你可以看到有很多 Size 行，有的很小，就是 4KB，而有的很大，例如 720896KB。不同内存块被换出到磁盘上的大小也不一样。

**敲重点了**

**如果 Swap 一切都是 0 kb，或者零星的 4k ，那么一切正常。**

**当出现百 MB，甚至 GB 级别的 swap 大小时，就表明，此时，Redis 实例的内存压力很大，很有可能会变慢。**

**解决方案**

1. 增加机器内存。
2. 将 Redis 放在单独的机器上运行，避免在同一机器上运行需要大量内存的进程，从而满足 Redis 的内存需求。
3. 增加 Cluster 集群的数量分担数据量，减少每个实例所需的内存。

#### 5. AOF 和磁盘 I/O 导致的延迟

在不死之身高可用章节我们知道 Redis 为了保证数据可靠性，你可以使用 AOF 和 RDB 内存快照实现宕机快速恢复和持久化。

**可以使用 appendfsync **配置将 AOF 配置为以三种不同的方式在磁盘上执行 write 或者 fsync （可以在运行时使用 **CONFIG SET**命令修改此设置，比如：`redis-cli CONFIG SET appendfsync no`）。

- **no**：Redis 不执行 fsync，唯一的延迟来自于 write 调用，write 只需要把日志记录写到内核缓冲区就可以返回。
- **everysec**：Redis 每秒执行一次 fsync，使用后台子线程异步完成 fsync 操作。最多丢失 1s 的数据。
- **always**：每次写入操作都会执行 fsync，然后用 OK 代码回复客户端（实际上 Redis 会尝试将同时执行的许多命令聚集到单个 fsync 中），没有数据丢失。在这种模式下，性能通常非常低，强烈建议使用 SSD 和可以在短时间内执行 fsync 的文件系统实现。

**我们通常只是将 Redis 用于缓存，数据未命中从数据获取，并不需要很高的数据可靠性，建议设置成 no 或者 everysec。**

除此之外，避免 AOF 文件过大 Redis 会进行 AOF 重写缩小的 AOF 文件大小。

你可以把配置项 `no-appendfsync-on-rewrite`设置为 yes，表示在 AOF 重写时不进行 fsync 操作。

也就是说，Redis 实例把写命令写到内存后，不调用后台线程进行 fsync 操作，就直接向客户端返回了。

#### 6. fork 生成 RDB 导致的延迟

Redis 必须 fork 后台进程才能生成 RDB 内存快照文件，fork 操作（在主线程中运行）本身会导致延迟。

Redis 使用操作系统的多进程**写时复制技术 COW(Copy On Write)** 来实现快照持久化，减少内存占用。

![图 5-2][image-132]

图 5-2

但 fork 会涉及到复制大量链接对象，一个 24 GB 的大型 Redis 实例执行 `bgsave`生成 RDB 内存快照文件 需要复制 24 GB / 4 kB \* 8 = 48 MB 的页表。

此外，**slave 在加载 RDB 期间无法提供读写服务，所以主库的数据量大小控制在 2\~4G 左右，让从库快速的加载完成**。

#### 6. 键值对数据集中过期淘汰

Redis 有两种方式淘汰过期数据。

- 惰性删除：当接收请求的时候检测 key 已经过期，才执行删除。
- 定时删除：按照每 100 毫秒的频率删除一些过期的 key。

定时删除的算法如下。

1. 随机采样 `CTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP（`默认设置为 20）\`个数的 key，删除所有过期的 key。

2. 执行之后，如果发现还有超过 25% 的 key 已过期未被删除，则继续执行步骤一。

每秒执行 10 次，一次删除 200 个 key 没啥性能影响。如果触发了第二条，就会导致 Redis 一致在删除过期数据取释放内存。

> 谢霸哥：“码哥，触发条件是什么呀？”

大量的 key 设置了相同的时间参数，同一秒内大量 key 过期，需要重复删除多次才能降低到 25% 以下。

**简而言之：大量同时到期的 key 可能会导致性能波动。**

**解决方案**

如果一批 key 的确是同时过期，可以在 `EXPIREAT` 和 `EXPIRE` 的过期时间参数上，**加上一个一定大小范围内的随机数**，这样，既保证了 key 在一个邻近时间范围内被删除，又避免了同时过期造成的压力。

#### 7. bigkey

> 谢霸哥：“什么是 Bigkey？key 很大么？”

“大”确实是关键字，但是这里的“大”指的是 Redis 中那些存有较大量元素的集合或列表、大对象的字符串占用较大内存空间的键值对数据称为 Bigkey。用几个实际例子来说。

- 一个 String 类型的 Key，它的 value 为 5MB（数据过大）。

- 一个 List 类型的 Key，它的列表数量为 10000 个（列表数量过多）。

- 一个 Zset 类型的 Key，它的成员数量为 10000 个（成员数量过多）。

- 一个 Hash 格式的 Key，它的成员数量虽然只有 1000 个但这些成员的 value 总大小为 10MB（成员体积过大）。

Bigkey 的存在可能会引发以下问题。

- **内存压力增大：** 大键会占用大量的内存，可能导致 Redis 实例的内存使用率过高，Redis 内存不断变大引发 OOM，或者达到 `maxmemory` 设置值引发写阻塞或重要 Key 被淘汰。
- **持久化延迟：** 在进行持久化操作（如 RDB 快照、AOF 日志）时，处理 bigkey 可能导致持久化操作的延迟。
- **网络传输压力：** 在主从复制中，如果有 bigkey 的存在，可能导致网络传输的压力增大。
- bigkey 的读请求占用过大带宽，自身变慢的同时影响到该服务器上的其它服务。

> 谢霸哥：“如何解决 Bigkey 问题呢？”

- **定期检测：** 使用工具如 `redis-cli` 的 `--bigkeys` 参数进行定期扫描和检测。
- **优化数据结构：** 根据实际业务需求，优化使用的数据结构，例如使用 HyperLogLog 替代 Set。
- **清理不必要的数据：** Redis 自 4.0 起提供了 `UNLINK` 命令，该命令能够以非阻塞的方式缓慢逐步的清理传入的 Key，通过 `UNLINK`，你可以安全的删除大 Key 甚至特大 Key。
- **对大 key 拆分：**如将一个含有数万成员的 HASH Key 拆分为多个 HASH Key，并确保每个 Key 的成员数量在合理范围，在 Redis Cluster 集群中，大 Key 的拆分对 node 间的内存平衡能够起到显著作用。

## 5.2 Redis 很强，不懂使用规范就糟蹋了

通过前面所学，我们知道 Redis 为了高性能和节省内存费劲心思。然而，我们只有规范的使用 Redis，才能实现高性能和节省内存，否则再快的 Redis 也禁不起我们瞎折腾。

Redis 使用规范围绕如下几个纬度展开。

- 键值对使用规范。
- 命令使用规范。
- 数据设计规范。
- SDK 使用规范。
- 运维管理规范。

### 5.2.1 键值对使用规范

有两点需要注意。

1. 好的 `key` 名称才能提供可读性强、可维护性高的代码，便于定位问题和查找数据。
2. `value` 要避免出现 `bigkey` 现象，选择高效的序列化和压缩、使用对象共享池、选择高效恰当的数据类型保存数据。

#### 1. key 命名规范

规范的 `key`命名，在遇到问题的时候能够方便定位。Redis 属于没有 `Scheme`的 `NoSQL`数据库。

所以要靠规范来建立其 `Scheme` 语意，就好比根据不同的场景我们建立不同的数据库。

**敲黑板**

把业务模块名或者数据库名作为前缀（好比数据库 `Scheme`），用冒号分隔。

这样我们就可以通过 `key` 前缀来区分不同的业务数据，清晰明了。总结起来就是“数据库名:表名:id”。

比如我们要记录公众号属于技术类型的博主“码哥字节”的粉丝数。

```
set 公众号:技术类:码哥字节 100000
```

> Chaya：“码哥，key 太长的话有什么问题么？”

key 是字符串类型，底层的数据结构是 `SDS`，SDS 结构中会包含字符串长度、分配空间大小等元数据信息。

**字符串长度增加，SDS 的元数据也会占用更多的内存空间。**

所以当字符串太长的时候，我们可以采用适当缩写的形式。

除此之外，禁止 key 包含特殊字符（大括号“{}”除外），由于大括号“{}”为 Redis 的 hash tag 语义，如果使用的是集群实例，Key 名称需要正确地使用大括号避免分片不均的情况。

#### 2. value 规范

防止出现 bigkey，设计合理的 Key 的 Value 的大小，推荐小于 10 KB。过大的 Value 会引发分片不均、热点 Key、实例流量或 CPU 使用率冲高等问题。

**因为 Redis 是单线程执行读写指令，如果出现`bigkey` 的读写操作就会阻塞线程，降低 Redis 的处理效率。**

`bigkey` 包含两种情况。

- 键值对的 `value` 很大，比如 `value`保存了 `2MB`的 `String`数据。
- 键值对的 `value`是集合类型（例如 Hash，Set，List 等），避免其中包含过多元素，建议单 Key 中的元素不要超过 5000 个。

> 谢霸哥：“码哥，如果业务数据就是这么大咋办？比如我就要把《金瓶梅》保存到 Redis。”

我们还可以通过 `gzip` 数据压缩来减小数据大小，以下是 Java 代码。

```java
/**
 * 使用gzip压缩字符串
 */
public static String compress(String str) {
    if (str == null || str.length() == 0) {
        return str;
    }

    try (ByteArrayOutputStream out = new ByteArrayOutputStream();
    GZIPOutputStream gzip = new GZIPOutputStream(out)) {
        gzip.write(str.getBytes());
    } catch (IOException e) {
        e.printStackTrace();
    }
    return new sun.misc.BASE64Encoder().encode(out.toByteArray());
}

/**
 * 使用gzip解压缩
 */
public static String uncompress(String compressedStr) {
    if (compressedStr == null || compressedStr.length() == 0) {
        return compressedStr;
    }
    byte[] compressed = new sun.misc.BASE64Decoder().decodeBuffer(compressedStr);;
    String decompressed = null;
    try (ByteArrayOutputStream out = new ByteArrayOutputStream();
    ByteArrayInputStream in = new ByteArrayInputStream(compressed);
    GZIPInputStream ginzip = new GZIPInputStream(in);) {
        byte[] buffer = new byte[1024];
        int offset = -1;
        while ((offset = ginzip.read(buffer)) != -1) {
            out.write(buffer, 0, offset);
        }
        decompressed = out.toString();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return decompressed;
}
```

#### 3. 使用高效序列化和压缩

为了节省内存，我们可以使用高效的序列化方法和压缩方法去减少 `value`的大小。

`protostuff`和 `kryo`这两种序列化方法，就要比 `Java`内置的序列化方法效率更高。

上述的两种序列化方式虽然省内存，但是序列化后都是二进制数据，可读性太差。

通常我们会序列化成 `JSON`或者 `XML`，为了避免数据占用空间大，可以使用压缩工具（snappy、 gzip）将数据压缩再存到 Redis 中。

#### 4. 使用整数对象共享池

Redis 内部维护了 0 到 9999 这 1 万个整数对象作为一个共享池使用。

即使大量键值对保存了 0 到 9999 范围内的整数，在 Redis 实例中，其实只保存了一份整数对象，可以节省内存空间。

需要注意的是，有两种情况整数对象共享池是不生效的。

1. Redis 中设置了 `maxmemory`，而且启用了 `LRU`策略（`allkeys-lru 或 volatile-lru 策略`）。

   这是因为 LRU 需要统计每个键值对的使用时间，如果不同的键值对都复用一个整数对象就无法统计了。

2. 如果集合类型数据采用 ziplist （7.0 之后是 listpack ）编码，并且集合元素是整数，这个时候，也不能使用共享池。

   因为 ziplist、listpack 使用了紧凑型内存结构，判断整数对象的共享情况效率低。

### 5.2.2 命令使用规范

有的命令的执行会造成很大的性能问题，我们一定要知道。

#### 1. 生产禁用的指令

Redis 是单线程处理请求操作，如果我们执行一些涉及大量操作、耗时长的命令，就会严重阻塞主线程，导致其它请求无法得到正常处理。

**谨慎使用 O(N)复杂度的命令**

`KEYS`：该命令需要对 Redis 的全局哈希表进行全表扫描，严重阻塞 Redis 主线程。应该使用 `SCAN` 来代替，分批返回符合条件的键值对，避免主线程阻塞。

`FLUSHALL`：删除 Redis 实例上的所有数据，如果数据量很大，会严重阻塞 Redis 主线程。

`FLUSHDB`，删除当前数据库中的数据，如果数据量很大，同样会阻塞 Redis 主线程。加上 `ASYNC` 选项，让 `FLUSHALL`，`FLUSHDB` 异步执行。

#### 2. 慎用 MONITOR 命令

`MONITOR` 命令会把监控到的内容持续写入输出缓冲区。如果线上命令的操作很多，输出缓冲区很快就会溢出，这会对 Redis 性能造成影响，甚至引起服务崩溃。

#### 3. 慎用全量操作命令

比如获取集合中的所有元素（HASH 类型的 `hgetall`、`List` 类型的 lrange`、`Set 类型的 `smembers`、`zrange` 等命令）。

这些操作时间复杂度为 O(N)的命令，需要特别注意 N 的值。避免 N 过大，造成 Redis 阻塞以及 CPU 使用率冲高。会对整个底层数据结构进行全量扫描 ，导致阻塞 Redis 主线程。

> 谢霸哥：“码哥，如果业务场景就是需要获取全量数据咋办？”

有两个方式可以解决。

1. 可使用 `hscan、sscan、zscan`这些分批扫描的命令替代。
2. 把大集合拆成小集合，比如按照时间、区域等划分。

禁止使用 del 命令直接删除大 Key，Redis 4.0 后的版本可以通过 UNLINK 命令安全地删除大 Key，该命令是异步非阻塞的。

#### 4. 使用批量提高效率

如果有批量操作，可使用 `mget`、`mset`或`pipeline`，提高效率，但要注意控制一次批量操作的元素个数。

`mget、mset` 和 `pipeline`的区别如下：

- `mget`和 mset`是`原子操作，`pipeline`是非原子操作。
- `pipeline`可以打包不同的命令。

### 5.2.3 数据存储使用规范

#### 1. 冷热数据分离

虽然 Redis 支持使用 RDB 快照和 AOF 日志持久化保存数据，但是，这两个机制都是用来提供数据可靠性保证的，并不是用来扩充数据容量的。

建议将热数据加载到 Redis 中。低频数据可存储在 MySQL 或者 ElasticSearch 中。

#### 2. 业务数据隔离

不要多个业务共用一个 Redis。 一方面避免业务相互影响，另一方面避免单实例膨胀，并能在故障时降低影响面，快速恢复。

#### 3. 设置过期时间

写入 Redis 的数据会一直占用内存，如果数据持续增多，就可能达到机器的内存上限，造成内存溢出，导致服务崩溃。

在数据保存时，我建议你根据业务使用数据的时长，设置数据的过期时间以及内存淘汰策略，可以在 Redis 内存意外写满的时候，仍然正常提供服务。

#### 4. 控制单实例的内存容量

建议设置在 2\~6 GB 。这样一来，无论是 RDB 快照、 AOF 重写，还是主从集群进行数据同步，都能很快完成，不会阻塞正常请求的处理。

Redis 在执行`RewriteAOF`和`BGSAVE`的时候，会 fork 一个进程，过大的内存会导致卡顿。

#### 5. 防止缓存雪崩

避免集中过期 key 导致缓存雪崩。当某一个时刻出现大规模的缓存失效的情况，那么就会导致大量的请求直接打在数据库上面，导致数据库压力巨大，如果在高并发的情况下，可能瞬间就会导致数据库宕机。

### 5.2.4 SDK 使用规范

SDK 操作 Redis 的时候有以下几个注意事项。

1. 使用连接池和长连接：连接的频繁创建和销毁，会浪费大量的系统资源，极限情况会造成宿主机宕机。

2. 避免使用 Lettuce 客户端：Lettuce 客户端在默认配置下有一定性能优势，并且是 spring 的默认客户端，但是 Jedis 客户端在面对连接异常，网络抖动等场景下的异常处理和检测能力明显强于 Lettuce，可靠性更强，建议使用 Jedis。

   - Lettuce 默认未配置集群拓补刷新的配置，会导致 Cluster 集群在发生拓补信息变化（主备倒换，扩容缩容）时，无法识别新的节点信息，导致业务失败。

   - Lettuce 没有连接池校验的功能，无法检测连接池中的连接是否仍然有效，获取失效连接之后会导致业务失败。

3. 客户端容错重试机制：由于 Redis 服务可能因网络波动或基础设置故障的影响，引发主备倒换，命令超时或慢请求等现象，需要在客户端内设计合理的容错重试机制。合理设置容错处理的重试时间，根据业务要求设置，避免过短或者过长。

### 5.2.5 运维规范

1. 生产系统中需要开启 Redis 密码保护机制，使用 Cluster 集群或者哨兵集群，做到高可用。

2. 根据告警基线配置告警：配置节点 cpu、内存、带宽等告警。
3. 对实例设置最大连接数，防止过多客户端连接导致实例负载过高，影响性能。
4. 不开启 AOF 或开启 AOF 配置为每秒刷盘，避免磁盘 I/O 拖慢 Redis 性能。
5. 设置合理的 `repl-backlog` 大小，降低主从全量同步的概率。
6. 设置合理的`slave client-output-buffer-limit` 大小，避免主从复制中断情况发生。
7. 根据实际场景设置合适的内存淘汰策略。

## 5.3 Redis 内存优化必杀技，小内存存储大数据

本篇，码哥跟你分享一些优化神技，当你面试或者工作中遇到如下问题，那就使出今天学到的绝招，一招定乾坤！

> 谢霸哥：“Redis 对内存的优化可谓是精打细算，油盐不断。还有什么杀招能更少的内存保存更多的数据？”

我们从 Redis 是如何保存数据的原理展开，分析键值对的存储结构和原理。

从而延展出每种数据类型底层的数据结构，针对不同场景使用更恰当的数据结构和编码实现更少的内存占用。

为了保存数据， Redis 需要先申请内存，数据过期或者内存淘汰需要回收内存，从而拓展出内存碎片优化。

最后，说下 key、value 使用规范和技巧、 Bitmap 等高阶数据类型，运用这些技巧巧妙解决有限内存去存储更多数据难题……这一套组合拳下来直接封神。

主要优化神技如下。

- 键值对优化。
- 小数据集合的编码优化。
- 使用对象共享池。
- 使用 Bit 比特位或 byte 级别操作。
- 使用 hash 类型优化。
- 内存碎片优化。
- 使用 32 位的 Redis。

### 5.3.2 键值对优化

当我们执行 `set key value` 的命令，`*key`指针指向 SDS 字符串保存 key，而 `value` 的值保存在 `*ptr` 指针指向的数据结构，消耗的内存为 key + value。

第一个优化神技：**降低 Redis 内存使用的最粗暴的方式就是缩减键（key）与值（value）的长度。**

在《Redis 很强，不懂使用规范就糟蹋了》章节中我说过关于键值对的使用规范，对于 key 的命名使用“业务模块名:表名:数据唯一 id”这样的方式方便定位问题。

比如：users:firends:996 表示用户系统中，id = 996 的朋友信息，我们可以简写为`u:fs:996`。**对于 key 的优化可以使用单词简写方式优化内存占用。**对于 value 的优化那就更多了。

- **过滤不必要的数据**：不要大而全的一股脑将所有信息保存，想办法去掉一些不必要的属性，比如缓存登录用户的信息，通常只需要存储昵称、性别、账号等。

- **精简数据**：比如用户的会员类型：0 表示「屌丝」、1 表示 「VIP」、2 表示「VVIP」。而不是存储 VIP 这个字符串。

- **数据压缩：**对数据的内容进行压缩，比如使用 GZIP、Snappy。

- **使用性能好，内存占用小的序列化方式**。比如 Java 内置的序列化不管是速度还是压缩比都不行，我们可以选择 protostuff，kryo 等方式。如下图 Java 常见的序列化工具空间压缩比：

![图 5-3][image-133]

  图 5-3

  \> 码哥：“靓仔们，我们通常使用 json 作为字符串存储在 Redis，用 json 存储与二进制数据存储有什么优缺点呢？”

  json 格式的优点：方便调试和跨语言；缺点是：同样的数据相比字节数组占用的空间更大。一定要 json 格式的话，那就先通过压缩算法压缩 json，再把压缩后的数据存入 Redis。比如 GZIP 压缩后的 json 可降低约 60% 的空间。

### 5.3.3 小数据集合编码优化

key 对象都是 string 类型，value 对象主要有五种基本数据类型 String、List、Set、Zset、Hash。数据类型与底层数据结构的关系如下所示。

![2-55][image-134]

图 5-4

特别说明下在 7.0 版本，**ziplist 压缩列表由 listpack 代替。另外，同一数据类型会根据键的数量和值的大小也有不同的底层编码类型实现。**

Redis 在存储集合数据（Hash、List、Set、SortedSet）满足某些情况下会采用内存压缩技术来实现使用更少的内存存储更多的数据。

**当这些集合中的数据元素数量小于某个值且元素的值占用的字节大小小于某个值的时候，存储的数据会用非常节省内存的方式进行编码，理论上至少节省 10 倍以上内存（平均节省 5 倍以上）。**

比如 Hash 类型里面的数据不是很多，哈希表的时间复杂度是 O(1)，listpack 的时间复杂度是 O(n)，但是使用 listpack 保存数据的话会节省了内存，并且在少量数据情况下效率并不会降低很多。

**所以我们需要尽可能地控制集合元素数量和每个元素的内存大小，这样能充分利用紧凑型编码减少内存占用。**

并且，这些编码对用户和 api 是无感知的，当集合数据超过配置文件的配置的最大值， Redis 会自动转成正常编码。

> 谢霸哥：“码哥，为啥对一种数据类型实现多种不同编码方式？”

主要原因是想通过不同编码实现效率和空间的平衡。比如当我们的存储只有 100 个元素的列表，使用双向链表数据结构时，需要维护大量的内部字段。

比如每个元素需要：前置指针，后置指针，数据指针等，造成空间浪费。

如果采用连续内存结构的压缩列表(ziplist)，将会节省大量内存，而由于数据长度较小，存取操作时间复杂度即使为 O(n) 性能也相差不大，因为 n 值小 与 O(1) 并明显差别。

### 5.3.4 对象共享池

整数我们经常在工作中使用，Redis 在启动的时候默认后生成一个 **0 \~9999 的整数对象共享池用于对象复用，减少内存占用**。

比如执行`set 码哥 18; set 吴彦祖 18;` key 分别是字符串 “码哥”、“吴彦祖”，他们的 value 都指向同一个对象 18。

如果 value 可以使用整数表示的话尽可能使用整数，这样即使大量键值对的 value 属于 0\~9999 范围内的整数，在实例中，其实只有一份数据。

**靓仔们需要注意的是，以下两个情况会导致对象共享池失效。**

- **Redis 中设置了 maxmemory 限制最大内存占用大小且启用了 LRU 策略（allkeys-lru 或 volatile-lru 策略）。**

  因为 LRU 需要记录每个键值对的访问时间，都共享一个整数 对象，LRU 策略就无法进行统计了。

- 集合类型的编码采用 ziplist （7.0 之后由 listpack 编码代替）编码，并且集合内容是整数，也不能共享一个整数对象。

  因为使用了 listpack 紧凑型内存结构存储数据，判断整数对象是否共享的效率很低。

### 5.3.5 使用 Bit 比特位或 byte 级别操作

在一些“二值状态统计”的场景下使用 Bitmap 实现，对于网页 UV 使用 HyperLogLog 来实现，大大减少内存占用。

二值状态统计：也就是集合中的元素的值只有 0 和 1 两种，在签到打卡和用户是否登陆的场景中，只需记录`签到(1)`或 `未签到(0)`，`已登录(1)`或`未登陆(0)`。

假如我们在判断用户是否登陆的场景中使用 Redis 的 String 类型实现（**key -\> userId，value -\> 0 表示下线，1 - 登陆**），假如存储 100 万个用户的登陆状态，如果以字符串的形式存储，就需要存储 100 万个字符串，内存开销太大。

String 类型除了记录实际数据以外，还需要额外的内存记录数据长度、空间使用等信息。

Bitmap 的底层数据结构用的是 String 类型的 SDS 数据结构来保存位数组，Redis 把每个字节数组的 8 个 bit 位利用起来，每个 bit 位 表示一个元素的二值状态（不是 0 就是 1）。

可以将 Bitmap 看成是一个 bit 为单位的数组，数组的每个单元只能存储 0 或者 1，数组的下标在 Bitmap 中叫做 offset 偏移量。

为了直观展示，我们可以理解成 buf 数组的每个字节用一行表示，每一行有 8 个 bit 位，8 个格子分别表示这个字节中的 8 个 bit 位，如下图所示。

![图 5-5][image-135]

图 5-5

### 5.3.6 妙用 Hash 类型优化

**尽可能把数据抽象到一个哈希表里。**比如说系统中有一个用户对象，我们不需要使用 String 类型存储一个用户的昵称、姓名、邮箱、地址等，而是将这个信息存放在一个哈希表里。

如下所示。

```bash
hset users:深圳:999 姓名 码哥
hset users:深圳:999 年龄 18
hset users:深圳:999 爱好 女
```

因为 Redis 的数据类型有很多，不同数据类型都有些相同的元数据要记录（比如最后一次访问的时间、被引用的次数等）。

所以，Redis 会用一个 RedisObject 结构体来统一记录这些元数据，用 \*prt 指针指向实际数据。

**当我们为每个属性都创建 key，就会创建大量的 `redisObejct` 对象占用内存。**用 Hash 类型的话，每个用户只需要设置一个 key。

### 5.3.7 内存碎片优化

Redis 释放的内存空间可能并不是连续的，这些不连续的内存空间很有可能处于一种闲置的状态。

虽然有空闲空间，Redis 却无法用来保存数据，不仅会减少 Redis 能够实际保存的数据量，还会降低 Redis 运行机器的成本回报率。

比如， Redis 存储一个整形数字集合需要一块占用 32 字节的连续内存空间，当前虽然有 64 字节的空闲，但是他们都是不连续的，导致无法保存。

在 4.0 之前版本，我们只能使用重启恢复：重启加载 RDB 或者通过高可用主从切换实现数据的重新加载减少碎片。

在 4.0 之后版本，Redis 提供了自动和手动的碎片整理功能，原理大致是把数据拷贝到新的内存空间，然后把老的空间释放掉，这个是有一定的性能损耗的。

手动整理碎片，执行 `memory purge`命令即可。自动清理内存碎片的细节详见【4.7 章节】。

### 5.3.8 使用 32 位的 Redis

使用 32 位的 redis，对于每一个 key 将使用更少的内存，因为 32 位程序，指针占用的字节数更少。但是 32 的 Redis 整个实例使用的内存将被限制在 4G 以下。我们可以通过 cluster 模式将多个小内存节点构成一个集群，从而保存更多的数据。

另外小内存的节点 fork 生成 rdb 的速度也更快。RDB 和 AOF 文件是不区分 32 位和 64 位的（包括字节顺序）,所以你可以使用 64 位的 Redis 恢复 32 位的 RDB 备份文件，相反亦然。

打完收工，这一套神技下来，只想说一个字“绝”。希望这篇文章，能帮你使用全局视角去破解内存优化难题。

## 5.4 生产王者必备配置详解

我是 Redis， 当程序员用指令 `./redis-server /path/to/redis.conf` 把我启动的时候，第一个参数必须是`redis.conf` 文件的路径。

这个文件很重要，就好像是你们的 DNA，它能控制我的运行情况，不同的配置会有不同的特性和人生，它掌握我的人生命运，控制着我如何完成高可用、高性能。合理的配置能让我更快、更省内存，并发挥我最大的优势让我更安全运行。

以下这些配置大家必知必会，需要大家掌握每个配置背后的技术原理，学会融合贯通并在生产中正确配置，解决问题。避免出现技术悬浮，原理说的叭叭叭，配置像个大傻瓜。

本文配置文件版本是 Redis 7.0。

### 5.4.1 常规通用配置

这些是我的常规配置，每个 Redis 启动必备参数，你一定要掌握，涉及到网络、模块插件、运行模式、日志等。

#### MODULES

这个配置可以加载模块插件增强我的功能，常见的模块有 RedisSearch、RedisBloom 等。关于模块加载可以参考【5.6 布隆过滤器原理与实战】章节集成布隆过滤器便是通过以下配置实现加载布隆过滤器插件。

```shell
loadmodule /opt/app/RedisBloom-2.2.14/redisbloom.so
```

#### NETWORK

这部分都是与网络相关的配置，很重要滴，配置不当将会有安全和性能问题。

##### bind

`bind`用于绑定**本机的网络接口**（网卡），注意是本机。

每台机器可能有多个网卡，每个网卡都有一个 IP 地址。配置了 bind，则表示我只允许来自本机指定网卡的 Redis 请求。

> MySQL：“bind 是用于限制访问你的机器 IP 么？”

非也，**注意，这个配置指的并不是只有 bind 指定的 IP 地址的计算机才能访问我。**如果想限制指定的主机连接我，只能通过防火墙来控制，bind 参数不也能起到这个作用。

举个例子：如果我所在的服务器有两个网卡，每个网卡有一个 IP 地址， IP1，IP2。

配置 `bind IP1`，则表示只能通过这个网卡地址来的网络请求访问我，也可以使用空格分割绑定多个网卡 IP。

我的默认配置是`bind 127.0.0.1 -::1` 表示绑定本地回环地址 IPv4 和 Ipv6。- 表示当 ip 不存在也能启动成功。

##### protected-mode

> MySQL：网络世界很危险滴，你如何保证安全？

默认开启保护模式，如果没有设置密码或者没有 bind 配置，我**只允许在本机连接我，其它机器无法连接**。

如果想让其它机器连接我，有以下三种方式。

1. 配置为 `protected-mode no`（不建议，地球很危险滴，防人之心不可无）。
2. `protected-mode yes`，配置 bind 绑定本机的 IP。
3. `protected-mode yes`，除了设置 `bind` 以外，还可以通过 `requirepass magebyte`设置密码为 `magebyte`， 让其他机器的客户端能使用密码访问我。

**bind、protected-mode、requirepass 之间的关系**

- bind：指定的是我所在服务器网卡的 IP，**不是指定某个可以访问我的机器。**
- protected-mode：保护模式，默认开启，如果没有设置密码或者 bind IP，我只接受本机访问(没密码+保护模式启动=本地访问)。
- requirepass，Redis 客户端连接我通行的密码。

如果参数设置为`bind 127.0.0.1 -::1`，不管 `protected-mode`是否开启，只能本机用 127.0.0.1 连接，其他外机无法连接。

**在生产环境中，为了安全，不要关闭 protected-mode，并设置 `requirepass` 参数配置密码和 bind 绑定机器的网卡 IP。**

##### port 6379

用于指定我监听的客户端 socket 端口号，默认 6379。设置为 0 则不会监听 TCP 连接，我想没人设置为 0 吧。

##### tcp-backlog 511

用于在 `Linux` 系统中控制 TCP 三次握手**已完成连接队列**（完成三次握手后）的长度，如果已完成连接队列已经满则无法放入，客户端会报`read timeout`或者`connection reset by peer`的错。

> MySQL：“在高并发系统中这玩意需要调大些吧？”

是的，我的默认配置是 511，这个配置的值不能大于 Linux 系统定义的 /proc/sys/net/core/somaxconn 值，Linux 默认的是 128。

所以我在启动的时候你会看到这样的警告：`WARNING: The TCP backlog setting of 511 cannot be enforced because kern.ipc.somaxconn is set to the lower value of 128.`

当系统并发量大并且客户端速度缓慢的时候，在高并发系统中，需要设置一个较高的值来避免客户端连接速度慢的问题。

需要分别调整 Linux 和 Redis 的配置。

**建议修改为 2048 或者更大**，Linux 则在 `/etc/sysctl.conf`中添加`net.core.somaxconn = 2048`配置，并且在终端执行 `sysctl -p`即可。

码哥使用 macOS 系统，使用 `sudo sysctl -w kern.ipc.somaxconn=2048`即可。

##### timeout

`timeout 60` 单位是秒，如果在 timout 时间内客户端跟我没有数据交互（客户端不再向我发送任何数据），我将关闭该客户端连接。

**注意事项**

- 0 表示永不断开。
- timeout 对应源码 `server.maxidletime`

##### tcp-keepalive

`tcp-keepalive 300` 单位是秒，官方建议值是 300。这是一个很有用的配置，实现 TCP 连接复用。

**用途**

用于客户端与服务端的长连接，如果设置为非 0，则使用 `SO_KEEPALIVE` 周期性发送 ACK 给客户端，俗话就是**用来定时向客户端发送 tcp\_ack 包来探测客户端是否存活，并保持该连接**。不用每次请求都建立 TCP 连接，毕竟创建连接是比较慢的。

#### 常规配置

这些都是我的常规配置，比较通用，你必须了解。你可以把这些配置写到一个特有文件中，其他节点可以使用 `include /path/to/other.conf` 配置来加载并复用该配置文件的配置。

##### daemonize

配置`daemonize yes`表示使用守护进程的模式运行，默认情况下我是以非守护线程的模式运行（daemonize no），开启守护进程模式，会生成一个 `.pid`文件存储进程号。

你也可以配置 `pidfile /var/run/redis_6379.pid` 参数来指定文件的生成目录，当关闭服务的时候我会自动删除该文件。

##### loglevel

指定我在运行时的日志记录级别。默认是 `loglevel notice`。有以下几个选项可以配置。

- debug：会记录很多信息，主要用于开发和测试。
- verbose：许多用处不大的信息，但是比 debug 少，如果发现生产出现一些问题无从下手，可使用该级别来辅助定位。
- notice：生产一般配置这个级别。
- warning：只会记录非常重要/关键的的日志。

##### logfile

指定日志文件目录，默认是 `logfile ""`，表示只在标准控制台输出。

**需要注意的是，如果使用标准控制台输出，并且使用守护进程的模式运行，日志会发送到 /dev/null。**

##### databases

设置数据库数量，我的默认配置是 `databases 16` 。默认的数据库是 `DB 0`，使用集群模式的时候， database 只有一个，就是 `DB 0`。

### 5.4.2 RDB 快照持久化

> MySQL：“要怎么开启 RDB 内存快照文件实现持久化呢？”

RDB 快照持久化相关的配置，必须掌握，合理配置能我实现宕机快速恢复实现高可用。

#### save

使用 `save <seconds> <changes>` 开启持久化，比如 `save 60 100` 表示 60 秒内，至少执行了 100 个写操作，则执行 RDB 内存快照保存。

不关心是否丢失数据，你也可以通过配置 `save ""` 来禁用 RDB 快照保存，让我性能起飞，冲出三界外。

默认情况的我会按照如下规则来保存 RDB 内存快照。

- 在 3600 秒 (一个小时) 内，至少执行了一次更改。
- 在 300 秒(5 分钟)内，至少执行了 100 个更改。
- 在 60 秒后，至少执行了 10000 个更改。

也可以通过 `save 3600 1 300 100 60 10000` 配置来显示设置。

#### stop-writes-on-bgsave-error

> MySQL：“bgsave 失败的话，停止接收写请求要怎么配置？”

默认配置为 `stop-writes-on-bgsave-error yes`，它的作用是**如果 RDB 内存快照持久化开启并且最后一次 `bgsave` 失败的话就停止接收写请求。**

我通过这种强硬的方式来告知程序员数据持久化不正常了，否则可能没人知道 RDB 快照持久化出问题了。

当 `bgsave` 后台进程能正常工作，我会自动允许写请求。如果你对此已经有相关的监控，即使磁盘出问题（磁盘空间不足、没有权限等）的情况下依旧处理写请求，那么设置成 `no` 即可。

#### rdbcompression

> MySQL：“RDB 内存快照文件比较大，可以压缩么？”

我的默认配置是 `rdbcompression yes`，意味着**对 RDB 内存快照文件中的 String 对象使用 LZF 算法做压缩**。这个非常有用，能大大减少文件大小，受益匪浅呀，建议你开启。

如果你不想损失因为压缩 RDB 内存快照文件的 CPU 资源，那就设置成 `no`，带来的后果就是文件比较大，传输占用更大的带宽（要三思啊，老伙计）。

#### rdbchecksum

默认配置是 `rdbchecksum yes`，从 5.0 版本开始，RDB 文件末尾会写入一个 CRC64 检验码，能起到一定的纠错作用，但是要**丢失大约 10%** 的性能损失，你可以设置成功 `no` 关闭这个功能来获得更快的性能。

关闭了这个功能， RDB 内存快照文件的校验就是 0 ，代码会自动跳过检查。

**推荐你关闭，让我快到令人发指。**

你还可以通过 `dbfilename` 参数来指定 RDB 内存快照文件名，默认是 `dbfilename dump.rdb`。

#### rdb-del-sync-files

默认配置是 `rdb-del-sync-files no`，主从进行全量同步时，通过传输 RDB 内存快照文件实现，没有开启 RDB 持久化的实例在同步完成后会删除该文件，通常情况下保持默认即可。

#### dir

我的工作目录，注意这是目录而不是文件， 默认配置是`dir ./`。比如存放 RDB 内存快照文件、AOF 文件。

### 5.4.3 主从复制

这部分配置很重要，涉及到主从复制的方方面面，是高可用的基石，重点对待啊伙计们。

#### replicaof

**主从复制，使用`replicaof <masterip> <masterport>` 配置将当前实例成为其他 Redis 服务的从节点**。

- masterip，就是 master 的 IP。
- masterport，master 的端口。

有以下几点需要注意。

- 我使用异步实现主从复制，当 Master 节点的 slave 节点数量小于指定的数量时，你可以设置 Master 节点停止处理写请求。
- 主从复制如果断开的时间较短，slave 节点可以执行部分重新同步，需要合理设置 `backlog size`，保证这个缓存区能完整保存断连期间 Master 接受写请求的数据，防止出现全量复制，具体配置后面会细说。
- 主从复制是自动的，不需要用户干预。

#### masterauth

如果当前节点是 slave，且 master 节点配置了 `requirepass` 参数设置了密码，那么 slave 节点必须使用该参数配置为 master 的密码，否则 master 节点将拒绝该 slave 节点的请求。

配置方式为 `masterauth <master-password>`。

#### masteruser

在 6.0 以上版本，如果使用了我的 ACL 安全功能，只配置 `masterauth` 还不够。因为默认用户不能运行 `PSYNC` 命令或者主从复制所需要的其他命令。

这时候，最好配置一个专门用于主从复制的特殊用户，配置方式为 `masteruser <username>`。

#### replica-serve-stale-data

> MySQL：“当 slave 节点与 master 失去连接，导致主从同步失败的时候，还能处理客户端请求么？”

slave 节点可以有以下两种行为来决定是否处理客户端请求。

- 配置为 `yes`，slave 节点可以继续处理客户端请求，但是数据可能是旧的，因为新的没同步过来。也可能是空的，如果是第一次同步的话。
- 配置为 `no`，slave 节点将返回错误 `MASTERDOWN Link with MASTER is down and replica-serve-stale-data is set to no`给客户端。但是以下的指令还是可以执行：`INFO, REPLICAOF, AUTH, SHUTDOWN, REPLCONF, ROLE, CONFIG, SUBSCRIBE,UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB, COMMAND, POST,HOST and LATENCY`。

我的默认配置是 `replica-serve-stale-data yes`。

#### replica-read-only

这个配置用于控制 slave 实例能否接收写指令，在 2.6 版本后默认配置为 `yes`，表示 slave 节点只处理读请求，如果为 `no` 则可读可写。

**我建议保持默认配置，让 slave 节点只作为副本实现高可用。想要提高写性能，使用集群模式横向拓展更好。**

#### repl-diskless-sync

主从复制过程中，新加入的 slave 节点和 slave 节点重连后无法进行增量同步，需要进行一次全量同步，master 节点会生成 RDB 内存快照文件传输给 slave 节点。

所以这个配置是用于控制传输方式的，传输方式有两种。

- Disk-backed（磁盘备份）：master 节点创建新进程将 RDB 内存快照文件写到磁盘，主进程逐步将这个文件传输到不同 slave 节点。
- Diskless（无盘备份）：master 节点创建一个新进程直接把 RDB 内存快照内容写到 Socket，不会将 RDB 内存快照文件持久化到磁盘。

使用磁盘备份的方式，master 保存在磁盘的 RDB 内存快照文件可以让多个 slave 复用。

使用无盘备份的话，当 RDB 内存快照文件传输开始，如果当前有多个`slave` 节点与 master 建立连接，我会使用并行传输的方式将 RDB 内容传输给多个节点。

**默认的配置是 `repl-diskless-sync yes`，表示使用无盘备份。在磁盘速度很慢，而网络超快的情况下，无盘备份会更给力。**如果网络很慢，有可能会出现数据丢失，推荐你改成 no。

#### repl-diskless-sync-delay

使用无盘复制的话，**如果此刻有新的 slave 发起全量同步，需要等待之前的传输完毕才能开启传输。**

所以可以使用配置 `repl-diskless-sync-delay 5` 参数指定一个延迟时间，这个单位是秒，让 master 节点等待一会，让更多 slave 节点连接再执行传输。

**因为一旦开始传输，master 节点无法响应新的 slave 节点的全量复制请求，只能在队列中等待下一次 RDB 内存快照传输。**

想要关闭这个功能，设置为 0 即可。

#### repl-diskless-load

mastar 节点有两种方式传输 RDB，slave 节点也有两种方式加载 master 传输过来的 RDB 数据。

- 传统方式：接受到数据后，先持久化到磁盘，再从磁盘加载 RDB 文件恢复数据到内存中，这是传统方式。
- diskless-load：从 Socket 中一边接受数据，一边解析，实现无盘化。

一共有三个取值可配置。

- disabled：不使用 diskless-load 方式，即采用磁盘化的传统方式。
- on-empty-db：安全模式下使用 diskless-load（也就 slave 节点数据库为空的时候使用 diskless-load）。
- swapdb：使用 diskless-load 方式加载，slave 节点会缓存一份当前数据库的数据，再清空数据库，接着进行 Socket 读取实现加载。缓存一份数据的目的是防止读取 Socket 失败。

**需要注意的是，diskless-load 目前在实验阶段，因为 RDB 内存快照数据并没有持久化到磁盘，因此有可能造成数据丢失；**

**另外，该模式会占用更多内存，可能会导致 OOM。**

#### repl-ping-replica-period

默认配置`repl-ping-replica-period 10` 表示 slave 每 10 秒 PING 一次 master。

#### repl-timeout

很重要的一个参数，slave 与 master 之间的复制超时时间，默认配置是`repl-timeout 60`，表示在 60 秒内 ping 不通，则判定超时。

超时包含以下三种情况。

- slave 角度，全量同步期间，在 repl-timeout 时间内没有收到 master 传输的 RDB 内存快照文件。
- slave 角度，在 repl-timeout 时间内没有收到 master 发送的数据包或者 ping。
- master 角度，在 repl-timeout 时间内没有收到 REPCONF ACK （复制偏移量 offset）确认信息。

当检测到超时，将会关闭 master 与 slave 之间的连接，slave 会发起重新建立主从连接的请求，对于内存数据比较大的系统，可以增大 `repl-timeout` 的值。

你需要注意的是，这个配置一定要大于 `repl-ping-replica-period`的值，否则每次心跳监测都超时。

#### repl-disable-tcp-nodelay

当 slave 与 master 全量同步（slave 发送 psync/sync 指令给 master）完成后，后续的增量同步是否设置成 `TCP_NODELAY`。

如果设置成 `yes`，master 将合并小的 TCP 包从而节省带宽，但是会增加同步延迟（40 ms），造成 master 与 slave 数据不一致；设置成 `no`，则 master 会立即发送数据给 slave，没有延迟。

默认配置 `repl-disable-tcp-nodelay no`。

#### repl-backlog-size

设置主从复制积压缓冲区（backlog） 容量大小，**这是一个环形数组，正常主从同步不涉及到 repl-backlog。当主从断开重连，repl-backlog 的作用就出来了**。

缓冲区用于存放断连期间 master 接受的写请求数据，当主从断开重连，通常不需要执行全量同步，只需要将断连期间的部分数据传递到 slave 即可。

**主从复制积压缓冲区越大，slave 可以承受的断连时间越长。**

默认配置是 `repl-backlog-size 1mb`，建议根据每秒流量大小和断开重连时间长，设置大一点，比如 128 mb。

#### repl-backlog-ttl

用于配置当 master 与 slave 断连多少秒之后，master 清空主从复制积压缓冲区（repl-backlog）。配置成 0 ，表示永远不清空。默认配置`repl-backlog-ttl 3600`。

#### replica-priority

slave 优先级，这个配置是给哨兵使用的，当 master 节点挂掉，哨兵会选择一个 priority 最小的 slave 节点作为新的 master，这个值越小没接越优先选中。

如果是 0，那意味着这个 slave 将不能选中成为 master，默认配置是 `replica-priority 100`。

#### min-slaves-to-write 和 min-slaves-max-lag

这两个配置要一起设置才有意义，如果有一个配置成 0，表示关闭该特性。

先看默认配置含义。

```shell
min-replicas-to-write 3
min-replicas-max-lag 10
```

如果 master 发现超过 3 个 slave 节点连接 master 延迟大于 10 秒，那么 master 就停止接收客户端写请求。**这么做的目的是为了尽可能保证主从数据一致性。**

master 会记录每个 slave 最近一次发来 ping 的时间，掌握每个 slave 的运行情况。

#### tracking-table-max-keys

我在 Redis 6.0 版本，实现了服务端辅助实现客户端缓存的特性，需要追踪客户端有哪些 key。当某个 key 被修改，我需要把这个失效信息发送到对应的客户端将本地缓存失效，这个配置就是用于指定追踪表保存的最大 key 数量，一旦超过这个数量，即使这个 key 没有被修改，为了回收内存我也会强制这个 key 所在的客户端缓存值失效。

设置 0 表示不限制，需要注意的是，如果使用广播模式实现键追踪，则不需要额外内存，忽略这个配置。

使用广播模式的不足就是与这个 key 无关的客户端也会收到失效消息。

### 5.4.4 安全

正是由于我快的一塌糊涂，攻击者一秒钟可以尝试 100 万个密码，所以你应该使用非常健壮的密码。

#### ACL

ACL 日志的最大长度，默认配置 `acllog-max-len 128` 表示最大 128 mb。

另外，使用 `aclfile /etc/redis/users.acl` 配置 ACL 文件所在位置。

#### requirepass

当前 Redis 服务器的访问密码，默认是不需要密码访问，网络危险，必须设置，如 `requirepass magebyte660`设置密码为 “magebyte666”。

#### maxclients

设置客户端同时连接的最大数量，默认设置是 `maxclients 10000`。达到最大值，我将关闭客户端新的连接，并发送一个 `max number of clients reached` 错误给客户端。

### 5.4.5 内存管理

作为用内存保存数据的我，这部分的配置也相当重要。

#### maxmemory

设置使用内存最大字节，当内存达到限制，我将尝试根据配置的内存淘汰策略（参见 maxmemory-policy）删除一些 key。建议你不要设置太大的内存，防止执行 RDB 内存快照文件或者 AOF 重写的时候因数据太大而阻塞过长时间。

推荐最大设置为 `maxmemory 6GB`。

如果淘汰策略是 `noeviction`，当收到写请求，我将回复错误给客户端，读请求依然可以执行。

如果你把我当做一个 LRU 或 LFU 缓存系统的时候，那请用心关注以下配置。

#### maxmemory-policy

设置内存淘汰策略，定义当内存满时如何淘汰 key，默认配置是 `noeviction`。

- volatile-lru -\> 在设置过期时间的 key 中使用近似 LRU 驱逐。
- allkeys-lru -\> 在所有 key 中使用近似 LRU 驱逐。
- volatile-lfu -\> 在过期 key 中使用近似 LFU 驱逐。
- allkeys-lfu -\> 在所有 key 中使用近似 LFU。
- volatile-random -\> 在设置了过期时间的 key 中随机删除一个。
- allkeys-random -\> 在所有的 key 中随机删除一个。
- volatile-ttl -\> 谁快过期就删谁。
- noeviction -\> 不删除任何 key，内存满了直接返回报错。

#### maxmemory-samples

LRU, LFU and minimal TTL algorithms 不是精确的算法，是一个近似的算法(主要为了节省内存)。

所以需要你自己权衡速度和精确度。默认会抽取 5 个 key，选择一个最近最少使用的 key 淘汰，你可以改变这个数量。

默认的 5 可以提供不错的结果。配置 10 会非常接近真实的 LRU 但是会耗费更多的 CPU，配置 3 会更快，但是就不那么精确了。

#### replica-ignore-maxmemory

从 Redis 5.0 开始，默认情况下 slave 节点会忽略 `maxmemory` 配置，除非在故障转移后或手动将其提升为 master。**这意味着只有 master 才会执行内存淘汰策略**，当 master 删除 key 后会发送 `DEL`指令给 slave。

默认配置`replica-ignore-maxmemory yes`。

#### active-expire-effort

我有两种方式删除过期数据。

- 后台周期性选取部分数据删除。
- 惰性删除，当访问请求到某个 key 的时候，发现该 key 已经过期则删除。

这个配置用于指定过期 key 滞留在内存中的比例，默认值是 1，表示最多只能有 10 % 的过期 key 驻留在内存中，值设置的越小，那么一次淘汰周期内需需要消耗的 CPU 将会更多，因为需要删除更多的过期数据。

### 5.4.6 惰性释放

> MySQL：“ 可以使用非阻塞的方式删除 bigkey 么？”

我提供了两种删除 key 的基本命令用于删除数据。

- `DEL` 指令：这是一个阻塞的删除，执行该指令会停止处理写请求，使用同步的方式去回收 DEL 删除的对象的内存。如果这个 key 对应的 value 是一个非常小的对象， `DEL` 执行的时间非常短，时间复杂度为 O(1) 或者 O(log n)。如果 key 对应的 value 非常大，比如集合对象的数据包含百万个元素，服务器将阻塞很长时间（几秒钟）才能完成操作。
- `UNLINK（非阻塞删除）、(异步删除) FLUSHALL ASYNC/FLUSHDB ASYNC`：后台回收内存，这些指令在常量级别时间内执行，会使用一个新的线程在后台渐进的删除并释放内存（Lazy Free 机制）。

#### lazyfree-lazy-eviction

由于 maxmemory 和 maxmemory-policy 策略配置，我会删除一些数据，防止内存爆掉。使用`lazyfree-lazy-eviction yes`表示使用 lazy free 机制，该场景开启 lazy free 可能会导致淘汰数据的内存释放不及时，出现内存超限。

#### lazyfree-lazy-expire

对于设置了 TTL 的键，过期后删除。如果想启用 lazy free 机制删除，则配置 `lazyfree-lazy-eviction yes`。

#### lazyfree-lazy-server-del

针对有些指令在处理已存在的键时，会带有一个隐式的 DEL 键的操作。

如 `rename` 命令，当目标键已存在，我会先删除目标键，如果这些目标键是一个 big key，那可能会出现阻塞删除的性能问题。 此参数设置就是解决这类问题，建议配置 `lazyfree-lazy-server-del yes` 开启。

#### replica-lazy-flush

该配置针对 slave 进行全量数据同步，在加载 master 的 RDB 内存快照文件之前，会先运行 `flashall`清理数据的时候是否采用异步 flush 机制。

推荐你使用 `replica-lazy-flush yes`配置，可减少全量同步耗时，从而减少 master 因输出缓冲区暴涨引起的内存增长。

#### lazyfree-lazy-user-del

意思是是否将 `DEL` 指令的默认行为替换成 lazy free 机制删除，效果就跟 `UNLINK` 一样，只要配置成 `lazyfree-lazy-user-del yes`。

#### lazyfree-lazy-user-flush

`FLUSHDB, FLUSHALL, SCRIPT FLUSH, FUNCTION FLUSH`可以使用额外参数 `ASYNC|SYNC` 决定使用同步还是异步操作，当没有指定这个可选项，可以通过 `lazyfree-lazy-user-flush yes` 表示使用异步删除。

#### I/O 多线程

大家知道我是单线程模型处理读写请求，但是有一些操作可以使用其他线程处理，比如 `UNLINK`，I/O 读写操作。

在 6.0 版本，我提供了 I/O 多线程处理 Socket 读写，利用 I/O 多线程可以提高客户端 Socket 读写性能。

**默认配置是关闭的，我只建议当你的机器至少是 4 核 CPU 或者更多的情况启用，并且配置的线程数少于机器总 CPU 核数，配置超过 8 个线程对提升没什么帮助。**

当你的机器是四核 CPU，那可以尝试配置使用 2\~3 个 I/O 线程，如果是 8 核 CPU，一般只需要配置 6 个线程。

如下配置表示开启 I/O 线程组，线程组的 I/O 线程数量为 3。

```shell
io-threads-do-reads yes
io-threads 3
```

### 5.4.7 AOF 持久化

除了 RDB 内存快照文件作为持久化手段以外，还能使用 AOF(Append only file) 实现持久化，AOF 是一种可选的持久化策略提供更好数据安全性。

默认配置下，我最多只会丢失一秒的数据，你甚至可以配置更高级别，最多只丢失一次 write 操作，但这样会对损耗性能。

#### appendonly

`appendonly yes` 表示开启 AOF 持久化，可以同时开启 AOF 和 RDB 内存快照持久化，如果开启了 AOF ，我会先加载 AOF 用于恢复内存数据。

#### appendfilename

指定 AOF 文件名称，默认名字是 `appendonly.aof`。为了方便，你可以配置 `appenddirname` 指定 AOF 文件存储目录。

#### appendfsync

调用操作系统的 `fsync()`函数告诉操作系统把输出缓冲区的数据持久化到磁盘， AOF 文件刷写的频率有三种。

- no：不去主动调用 fsync()，让操作系统自己决定何时写磁盘。
- always：每次 write 操作之后都调用 fsync()，非常慢，但是数据安全性最高。
- everysec：每秒调用一次 fsync()，一个折中的策略，最多丢失一秒的数据。

**默认配置是 `appendfsync everysec`，推荐大家这么设置，兼顾了速度和数据安全。**

#### no-appendfsync-on-rewrite

当 appendfsync 的配置设置成 `always`或者 `everysec` ，现在有一个后台 save 进程（可能是生成 RDB 内存快照的 bgsave 进程，也有可能是 AOF rewrite 进程）正在进行大量的磁盘 I/O 操作，会造成调用 `fsync()`执行太长，**后续其他想要调用 `fsync()` 的进程就会阻塞。**

为了缓解这个问题，可以使用以下配置 `no-appendfsync-on-rewrite yes` **表示当已经有 `bgsave`和`bgrewriteaof` 后台进程在调用 `fsync()` 时，不再开启新进程执行 AOF 文件写入。**

这样的话，就会出现当前有子进程在做 bgsave 或者其他的磁盘操作时，我就无法继续写 AOF 文件，这意味着可能会丢失更多数据。

如果有延迟问题，请将此选项改为 `yes`。否则将其保留为 `no`。从持久化的角度来看，`no`是最安全的选择。

#### AOF 重写

为了防止 AOF 文件过大，antirez 大佬给我搞了个 AOF 重写机制。

`auto-aof-rewrite-percentage 100` 表示当前 AOF 文件大小超过上一次重写的 AOF 文件大小的百分之多少（如果没有执行过 AOF 重写，那就参照原始 AOF 文件大小），则执行 AOF 文件重写操作。

除了这个配置，你还要配置 `auto-aof-rewrite-min-size 64mb` 用于指定触发 AOF 重写操作的文件大小。

**如果该 AOF 文件大小小于该值，即使文件增长比例达到 100%，我也不会触发 AOF 重写操作，这是为了防止 AOF 文件其实很小，但是满足增长百分比时的多余 AOF 重写操作。**

**如果配置为`auto-aof-rewrite-percentage 0` ，表示禁用 AOF 重写功能，建议大家开启 AOF 重写，防止文件过大。**

#### aof-load-truncated

> MySQL：如果 AOF 文件是损坏的，你还加载数据还原到内存中么？

加载 AOF 文件把数据还原到内存中，文件可能是损坏的，比如文件末尾是错误的。这种情况一般是由于宕机导致，尤其是使用 ext4 文件系统挂载时没配置 `data=ordered` 选项。

在这种情况下，我可以直接报错，或者尽可能的读取可读的 AOF 内容。

如果配置成 `aof-load-truncated yes`，我依然会加载并读取这个损坏的 AOF 文件，并记录一个错误日志通知程序员。

配置成 `aof-load-truncated no`，我就会报错并拒绝启动服务，你需要使用 redis-check-aof 工具修复 AOF 文件，再启动 Redis。如果修复后还是错误，我依然报错并拒绝启动。

#### aof-use-rdb-preamble

**这就是大名鼎鼎的 RDB-AOF 混合持久化功能，配置成 `aof-use-rdb-preamble yes`（必须先开启 AOF），AOF 重写生成的文件将同时包含 RDB 格式的内容和 AOF 格式内容。**

混合持久化是在 AOF 重写完成的，开启混合持久化后，fork 出的子进程先将内存数据以 RDB 的方式写入 AOF 文件，接着把 RDB 格式数据写入 AOF 文件期间收到的增量命令从重写缓冲区以 AOF 格式写到文件中。

写入完成后通知主进程更新统计信息，并把含有 RDB 格式和 AOF 格式的 AOF 文件替换旧的 AOF 文件。

**这样的好处是可以结合 RDB 和 AOF 的优点，实现快速加载同时避免丢失过多数据，缺点是 AOF 文件的 RDB 部分内容不是 AOF 格式，可读性差（都是程序解析读取，哪个傻瓜程序员去读这个呀），强烈推荐你使用这个来保证持久化。**

#### aof-timestamp-enabled

我在 7.0 版本新增的特性，大体就是讲 AOF 现在支持时间戳了，你可以做到基于时间点来恢复数据。

默认是是 `aof-timestamp-enabled no` 表示关闭该特性，你可以按照实际需求选择开启。

### 5.4.7 Cluster 集群

Redis Cluster 集群相关配置，使用集群方式的你必须重视和知晓。别嘴上原理说的头头是道，而集群有哪些配置？如何配置让集群快到飞起，实现真正的高可用却一头雾水，通过下面这些配置详解也让你对集群原理更加深刻。

#### cluster-enabled

普通的 Redis 实例是不能成为集群的一员，想要将该节点加入 Redis Cluster，需要设置 `cluster-enabled yes`。

#### cluster-config-file

`cluster-config-file nodes-6379.conf` 指定集群中的每个节点文件。

集群中的每个节点都有一个配置文件，这个文件并不是让程序员编辑的，是我自己创建和更新的，每个节点都要使用不同的配置文件，一定要确保同一个集群中的不同节点使用的是不同的文件。

#### cluster-node-timeout

设置集群节点不可用的最大超时时间，节点失效检测。集群中当一个节点向另一个节点发送 PING 命令，但是目标节点未在给定的时限内返回 PING 命令的回复时，那么发送命令的节点会将目标节点标记为 PFAIL(possible failuer，可能已失效)；

如果 master 节点超过这个时间还是无响应，则用它的从节点将启动故障迁移，升级成主节点。

默认配置是 `cluster-node-timeout 15000`，单位是毫秒数。

#### cluster-port

该端口是集群总线监听 TCP 连接的端口，默认配置为 `cluster-port 0`，我就会把端口绑定为客户端命令端口 + 10000（客户端端口默认 6379，所以绑定为 16379 作为集群总线端口）。每个 Redis Cluster 节点都需要开放两个端口：

- 一个用于服务于客户端的 TCP 端口，比如 6379.
- 另一个称为集群总线端口，节点使用集群总线进行故障监测、配置更新、故障转移等。**客户端不要与集群总线端口通信，另外请确保在防火墙打开这两个端口，否则 Redis 集群之间将无法通信**。

#### cluster-replica-validity-factor

该配置用于决定当 Redis Cluster 集群中，一个 master 宕机后，如何选择一个 slave 节点完成故障转移自动恢复（failover）。**如果设置为 0 ，则不管 slave 与 master 之间断开多久，都有资格成为 master。**

下面提供了两种方式来评估 slave 的数据是否太旧。

- 如果有多个 slave 可以 failover，他们之间会通过交换信息选出拥有拥有最大复制 offset 的 slave 节点。
- 每个 slave 节点计算上次与 master 节点交互的时间，这个交互包含最后一次 `ping` 操作、master 节点传输过来的写指令、上次与 master 断开的时间等。如果上次交互的时间过去很久，那么这个节点就不会发起 failover。

针对第二点，交互时间可以通过配置定义，如果 slave 与 master 上次交互的时间大于 `(node-timeout * cluster-replica-validity-factor) + repl-ping-replica-period`，该 slave 就不会发生 failover。

例如，\``node-timeout = 30`秒，`cluster-replica-validity-factor=10`，`repl-ping-slave-period=10`秒， 表示 slave 节点与 master 节点上次交互时间已经过去了 310 秒，那么 slave 节点就不会做 failover。

调大 `cluster-replica-validity-factor` 则允许存储过旧数据的 slave 节点提升为 master，调小的话可能会导致没有 slave 节点可以升为 master 节点。

**考虑高可用，建议大家设置为 `cluster-replica-validity-factor 0`。**

#### cluster-migration-barrier

没有 slave 节点的 master 节点称为孤儿 master 节点，这个配置就是用于防止出现孤儿 master。

当某个 master 的 slave 节点宕机后，集群会从其他 master 中选出一个富余的 slave 节点迁移过来，确保每个 master 节点至少有一个 slave 节点，防止当孤立 master 节点宕机时，没有 slave 节点可以升为 master 导致集群不可用。

默认配置为 `cluster-migration-barrier 1`，是一个迁移临界值。

含义是：被迁移的 master 节点至少还有 1 个 slave 节点才能做迁移操作。比如 master A 节点有 2 个以上 slave 节点 ，当集群出现孤儿 master B 节点时，A 节点富余的 slave 节点可以迁移到 master B 节点上。

生产环境建议维持默认值，最大可能保证高可用，设置为非常大的值或者配置 `cluster-allow-replica-migration no` 禁用自动迁移功能。

`cluster-allow-replica-migration` 默认配置为 yes，表示允许自动迁移。

#### cluster-require-full-coverage

**默认配置是 `yes`，表示为当 redis cluster 发现还有哈希槽没有被分配时禁止查询操作。**

这就会导致集群部分宕机，整个集群就不可用了，当所有哈希槽都有分配，集群会自动变为可用状态。

如果你希望 cluster 的子集依然可用，配置成 `cluster-require-full-coverage no`。

#### cluster-replica-no-failover

当配置成 `yes`，在 master 宕机时，slave 不会做故障转移升为 master。

**这个配置在多数据中心的情况下会很有用，你可能希望某个数据中心永远不要升级为 master 节点，否则 master 节点就漂移到其他数据中心了，正常情况设置成 no。**

#### cluster-allow-reads-when-down

默认是 `no`，表示当集群因主节点数量达不到最小值或者哈希槽没有完全分配而被标记为失效时，节点将停止所有客户端请求。

**设置成 `yes`，则允许集群失效的情况下依然可从节点中读取数据，保证了高可用。**

#### cluster-allow-pubsubshard-when-down

配置成 `yes`，表示当集群因主节点数量达不到最小值或者哈希槽没有完全分配而被标记为失效时，pub/sub 依然可以正常运行。

#### cluster-link-sendbuf-limit

设置每个集群总线连接的发送字节缓冲区的内存使用限制，超过限制缓冲区将被清空（主要为了防止发送缓冲区发送给慢速连接时无限延长时间的问题）。

默认禁用，建议最小设置 1gb，这样默认情况下集群连接缓冲区可以容纳至少一条 pubsub 消息（client-query-buffer-limit 默认是 1gb）；

### 5.4.8 性能监控

#### 慢查询日志

慢查询（Slow Log）日志是我用于记录慢查询执行时间的日志系统，只要查询超过配置的时间，都会记录。slowlog 只保存在内存中，因此效率很高，大家不用担心会影响到 Redis 的性能。

执行时间不包括 I/O 操作的时间，比如与客户端建立连接、发送回复等，只记录执行命令执行阶段所需要的时间。

你可以使用两个参数配置慢查询日志系统。

- `slowlog-log-slower-than`：指定对执行时间大于多少微秒（microsecond，1 秒 = 1,000,000 微秒）的查询进行记录，默认是 10000 微妙，推荐你先执行基线测试得到一个基准时间，通常这个值可以设置为基线性能最大延迟的 3 倍。
- `slowlog-max-len`：设定最多保存多少条慢查询的日志，slowlog 本身是一个 FIFO 队列，当超过设定的最大值后，我会把最旧的一条日志删除。默认配置 128，如果设置太大会占用多大内存。

#### 延迟监控

延迟监控（LATENCY MONITOR）系统会在运行时抽样部分命令来帮助你分析 Redis 卡顿的原因。

通过 `LATENCY`命令，可以打印一些视图和报告，系统只会记录大于等于指定值的命令。

默认配置 `latency-monitor-threshold 0`，设置 0 表示关闭这个功能。**没有延迟问题，没必要开启开启监控，因为会对性能造成很大影响。**

在运行过程中你怀疑有延迟性能问题，想要监控的话可以使用 `CONFIG SET latency-monitor-threshold <milliseconds>`开启，单位是毫秒。

### 5.4.9 高级设置

这部分配置主要围绕以下几个方面。

- 指定不同数据类型根据不同条数下使用不同的数据结构存储，合理配置能做到更快和更省内存。
- 客户端缓冲区相关配置。
- 渐进式 rehash 资源控制。
- LFU 调优。
- RDB 内存快照文件、AOF 文件同步策略。

#### Hashes（散列表）

在 Redis 7.0 版本散列表数据类型有两种数据结构保存数据，分别为散列表和 listpack。当数据量很小时，可以使用更高效的数据结构存储，从而达到在不影响性能的情况下节省内存。

- `hash-max-listpack-entries 512`：指定使用 listpack 存储的最大条目数。
- `hash-max-listpack-value 64`：listpack 中，条目 value 值最大字节数，建议设置成 1024。

在 7.0 版本以前，使用的是 ziplist 数据结构，配置如下。

```shell
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
```

#### Lists（列表）

Lists 也可以使用一种特殊方式进行编码来节省大量内存空间。在 Redis 7.0 之后，Lits 底层的数据结构使用 linkedlist 或者 listpack 。

Redis 3.2 版本，List 内部是通过 linkedlist 和 quicklist 实现，quicklist 是一个双向链表， quicklist 的每个节点都是一个 ziplist，从而实现节省内存。

元素少时用 quicklist，元素多时用 linkedlist。listpack 的目的就是用于替代 ziplist 和 quicklist。listpack 也叫**紧凑列表**，它的特点就是用一块连续的内存空间来紧凑地保存数据，同时为了节省内存空间

**list-max-ziplist-size**

7.0 版本之前`list-max-ziplist-size` 用于配置 quicklist 中的每个节点的 ziplist 的大小。 当这个值配置**为正数时表示 quicklist 每个节点的 ziplist 最多可存储元素数量**，超过该值就会使用 linkedlist 存储。

当 `list-max-ziplist-size` **为负数时表示限制每个 quicklistNode 的 ziplist 的内存大小**，超过这个大小就会使用 linkedlist 存储数据，每个值有以下含义：

- -5：每个 quicklist 节点上的 ziplist 大小最大 64 kb \<--- 正常环境不推荐
- -4：每个 quicklist 节点上的 ziplist 大小最大 32 kb \<--- 不推荐
- -3：每个 quicklist 节点上的 ziplist 大小最大 16 kb \<--- 可能不推荐
- -2：每个 quicklist 节点上的 ziplist 大小最大 8 kb \<--- 不错
- -1：每个 quicklist 节点上的 ziplist 大小最大 4kb \<--- 不错

默认值为 -2，也是官方最推荐的值，当然你可以根据自己的实际情况进行修改。

**list-max-listpack-size**

7.0 之后，配置修改为`list-max-listpack-size -2`则表示限制每个 listpack 大小，不再赘述。

**list-compress-depth**

压缩深度配置，用来配置压缩 Lists 的，当 Lists 底层使用 linkedlist 也是可以压缩的，默认是 `list-compress-depth 0`表示不压缩。一般情况下，Lists 的两端访问的频率高一些，所以你可以考虑把中间的数据进行压缩。

不同参数值的含义如下。

- 0，关闭压缩，默认值。
- 1，两端各有一个节点不压缩。
- 2，两端各有两个节点不压缩。
- N，依次类推，两端各有 N 个节点不压缩。

**需要注意的是，head 和 tail 节点永远都不会被压缩。**

#### Sets（无序集合）

Sets 底层的数据结构可以是 intset（整形数组）和 Hashtable（散列表），intset 你可以理解成数组，Hashtable 就是普通的散列表（key 存的是 Sets 的值，value 为 null）。有没有觉得 Sets 使用散列表存储是意想不到的事情？

**set-max-intset-entries**

当集合的元素都是 64 位以内的十进制整数时且长度不超过 `set-max-intset-entries` 配置的值（默认 512），Sets 的底层会使用 intset 存储节省内存。添加的元素大于 `set-max-intset-entries`配置的值，底层实现由 intset 转成散列表存储。

#### SortedSets（有序集合）

在 Redis 7.0 版本之前，有序集合底层的数据结构有 ziplist 和 skipist，之后使用 listpack 代替了 ziplist。

7.0 版本之前，当集合元素个数小于 `zset-max-ziplist-entries`配置，同时且每个元素的值大小都小于`zset-max-ziplist-value`配置（默认 64 字节，推荐调大到 128）时，我将使用 ziplist 数据结构存储数据，有效减少内存使用。与此类似，7.0 版本之后我将使用 listpack 存储。

```shell
## 7.0 之前的配置
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
## 7.0 之后的配置
zset-max-listpack-entries 128
zset-max-listpack-value 64
```

#### HyperLogLog

HyperLogLog 是一种高级数据结构，统计基数的利器。**HyperLogLog 的存储结构分为密集存储结构和稀疏存储结构两种，默认为稀疏存储结构，而我们常说的占用 12K 内存的则是密集存储结构，稀疏结构占用的内存会更小。**

**hll-sparse-max-bytes**

默认配置是 `hll-sparse-max-bytes 3000`，单位是 Byte，这个配置用于决定存储数据使用稀疏数据结构（sparse）还是稠密数据结构（dense）。

如果 HyperLogLog 存储内容大小大于 hll-sparse-max-bytes 配置的值将会转换成稠密的数据结构（dense）。

推荐的值是 0\~3000，这样`PFADD`命令的并不会慢多少，还能节省空间。如果内存空间相对 cpu 资源更缺乏，可以将这个值提升到 10000。

#### Streams（流）

Stream 是 Redis 5.0 版本新增的数据类型。Redis Streams 是一些由基数树（Radix Tree）连接在一起的节点经过 delta 压缩后构成的，这些节点与 Stream 中的消息条目（Stream Entry）并非一一对应，而是**每个节点中都存储着若干 Stream 条目**，因此这些节点也被称为宏节点或大节点。

**stream-node-max-bytes 4096**

单位为 Byte，默认值 4096，用于设定每个宏节点占用的内存上限为 4096，0 表示无限制。

**stream-node-max-entries 100**

用于设定每个宏节点存储元素个数。 默认值 100，0 表示无限制。当一个宏节点存储的 Stream 条目到达上限，新添加的条目会存储到新的宏节点中。

#### rehash

我采用的是渐进式 rehash，这是一个惰性策略，不会一次性把所有数据迁移完，而是分散到每次请求中，这样做的目的是防止数据太多要迁移阻塞主线程。

**在渐进式 rehash 的同时，推荐你使用 `activerehashing yes`开启定时辅助执行 rehash，默认情况下每一秒执行 10 次 rehash 加快迁移速度，尽可能释放内存。**

关闭该功能的话，如果这些 key 不再活跃不被被访问到，rehash 操作可能不再有机会完成，会导致散列表占用更多内存。

#### 客户端输出缓冲区限制

这三个配置是用来强制断开客户端连接的，当客户端没有及时把缓冲区的数据读取完毕，我会认为这个客户端可能完蛋了（一个常见的原因是 Pub/Sub 客户端处理发布者的消息不够快），于是断开连接。

一共分为三种不同类型的客户端，分别设置不同的限制。

- normal（普通），普通客户端，包括 MONITOR 客户端。
- replica（副本客户端），slave 节点的客户端。
- pubsub（发布订阅客户端），至少订阅了一个 pubsub 频道或者模式的客户端。

`client-output-buffer-limit`的语法如下。

```shell
client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
```

<class> 表示不同类型的客户端，当客户端的缓冲区内容大小达到 <hard limit>后我就立马断开与这个客户端的连接，或者达到 <soft limit> 并持续了 <soft seconds>秒后断开。

默认情况下，普通客户端不会限制，只有后异步的客户端才可能发送发送请求的速度比读取响应速度快的问题。比如 pubsub 和 replica 客户端会有默认的限制。

soft limit 或者 hard limit 设置为 0，表示不启用此限制。默认配置如下。

```shell
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

#### client-query-buffer-limit

每个客户端都有一个 query buffer（查询缓冲区或输入缓冲区），用于保存客户端发送命令，Redis Server 从 query buffer 获取命令并执行。

如果程序的 Key 设计不合理，客户端使用大量的 query buffer，导致 Redis 很容易达到 maxmeory 限制。最好限制在一个固定的大小来避免占用过大内存的问题。

如果你需要发送巨大的 multi/exec 请求的时候，那可以适当修改这个值以满足你的特殊需求。

默认配置为 `client-query-buffer-limit 1gb`。

#### maxmemory-clients

这是 7.0 版本特性，每个与服务端建立连接的客户端都会占用内存（查询缓冲区、输出缓冲区和其他缓冲区），大量的客户端可能会占用过大内存导致 OOM，为了避免这个情况，我提供了一种叫做（Client Eviction）客户端驱逐机制用于限制内存占用。

配置方式有两种。

- 具体内存值， `maxmemory-clients 1g`来限制所有客户端占用内存总和。
- 百分比，`maxmemory-clients 5%` 表示客户端总和内存占用最多为 Redis 最大内存配置的 5%。

默认配置是 `maxmemory-clients 0` 表示无限制。

> MySQL：“达到最大内存限制，你会把所有客户端连接都释放么？”

不是的，一旦达到限制，我会优先尝试断开使用内存最多的客户端。

#### proto-max-bulk-len

批量请求（单个字符串的元素）内存大小限制，默认是 `proto-max-bulk-len 512mb`，你可以修改限制，但必须大于等于 1mb。

#### hz

我会在后台调用一些函数来执行很多后台任务，比如关闭超时连接，清理不再被请求的过期的 key，rehash、执行 RDB 内存快照和 AOF 持久化等。

并不是所有的后台任务都需要使用相同的频率来执行，你可以使用 hz 参数来决定执行这些任务的频率。

默认配置是 `hz 10`，表示每秒执行 10 次，更大的值会消耗更多的 CPU 来处理后台任务，带来的效果就是更快的清理过期 key，清理的超时连接更精确。

**这个值的范围是 1\~500，不过并不推荐设置大于 100 的值。大家使用默认值就好，或者最多调高到 100。**

#### dynamic-hz

默认配置是 `dynamic-hz yes`，启用 dynamic-hz 后，将启用自适应 HZ 值的能力。hz 的配置值将会作为基线，Redis 服务中的实际 hz 值会在基线值的基础上根据已连接到 Redis 的客户端数量自动调整，连接的客户端越多，实际 hz 值越高，Redis 执行定期任务的频率就越高。

#### `aof-rewrite-incremental-fsync`

当子进程进行 AOF 重写时，如果配置成 `aof-rewrite-incremental-fsync yes`，每生成 4 MB 数据就执行一次 `fsync`操作，分批提交到硬盘来避免高延迟峰值，推荐开启。

#### rdb-save-incremental-fsync

当我在保存 RDB 内存快照文件时，如果配置成 `db-save-incremental-fsync yes`，每生成 4MB 文件就执行一次 `fsync`操作，分批提交到硬盘来避免高延迟峰值，推荐开启。

#### LFU 调优

这个配置生效的前提是内存淘汰策略设置的是 `volatile-lfu`或`allkeys-lfu`。

- lfu-log-factor 用于调整 Logistic Counter 的增长速度，lfu-log-factor 值越大，Logistic Counter 增长越慢。默认配置 10。

  以下是表格是官方不同 factor 配置下，计数器的改变频率。注意：表格是通过如下命令获得的： `redis-benchmark -n 1000000 incr foo redis-cli object freq foo`。

  | factor | 100 hits | 1000 hits | 100K hits | 1M hits | 10M hits |
  | :----- | :------- | :-------- | :-------- | :------ | :------- |
  | 0      | 104      | 255       | 255       | 255     | 255      |
  | 1      | 18       | 49        | 255       | 255     | 255      |
  | 10     | 10       | 18        | 142       | 255     | 255      |
  | 100    | 8        | 11        | 49        | 143     | 255      |

- lfu-decay-time 用于调整 Logistic Counter 的衰减速度，它是一个以分钟为单位的数值，默认值为 1；lfu-decay-time 值越大，衰减越慢。

### 5.4.9 在线内存碎片整理

> MySQL：“什么是在线内存碎片整理？”

Active (online) defragmentation 在线内存碎片整理指的是自动压缩内存分配器分配和 Redis 频繁做更新操作、大量过期数据删除，释放的空间（不够连续）无法得到复用的内存空间。

通常来说当碎片化达到一定程度（查看下面的配置）Redis 会使用 Jemalloc 的特性创建连续的内存空间， 并在此内存空间对现有的值进行拷贝，拷贝完成后会释放掉旧的数据。 这个过程会对所有的导致碎片化的 key 以增量的形式进行。

**需要注意的是**

1. 这个功能默认是关闭的，并且只有在编译 Redis 时使用我们代码中的 Jemalloc 版本才生效。（这是 Linux 下的默认行为）。
2. 在实际使用中，建议是在 Redis 服务出现较多的内存碎片时启用（内存碎片率大于 1.5），正常情况下尽量保持禁用状态。
3. 如果你需要试验这项特性，可以通过命令 `CONFIG SET activefrag yes`来启用。

**清理的条件**

`activefrag yes`：内存碎片整理总开关，默认为禁用状态 no。

`active-defrag-ignore-bytes 200mb`：内存碎片占用的内存达到 200MB。

`active-defrag-threshold-lower 20`：内存碎片的空间占比超过系统分配给 Redis 空间的 20% 。

**在同时满足上面三项配置时，内存碎片自动整理功能才会启用。**

**CPU 资源占用**

> MySQL：如何避免自动内存碎片整理对性能造成影响？

清理的条件有了，还需要分配清理碎片占用的 CPU 资源，保证既能正常清理碎片，又能避免对 Redis 处理请求的性能影响。

`active-defrag-cycle-min 5`：自动清理过程中，占用 CPU 时间的比例不低于 5%，从而保证能正常展开清理任务。

`active-defrag-cycle-max 20`：自动清理过程占用的 CPU 时间比例不能高于 20%，超过的话就立刻停止清理，避免对 Redis 的阻塞，造成高延迟。

**整理力度**

`active-defrag-max-scan-fields 1000`：碎片整理扫描到`set/hash/zset/list` 时，仅当 `set/hash/zset/list`的长度小于此阀值时，才会将此键值对加入碎片整理，大于这个值的键值对会放在一个列表中延迟处理。

`active-defrag-threshold-upper 100`：内存碎片空间占操作系统分配给 Redis 的总空间比例达此阈值（默认 100%），我会尽最大努力整理碎片。建议你调整为 80。

#### jemalloc-bg-thread

默认配置为 `jemalloc-bg-thread yes`，表示启用清除脏页后台线程。

### 5.4.10 绑定 CPU

你可以将 Redis 的不同线程和进程绑定到特定的 CPU，减少上下文切换，提高 CPU L1、L2 Cache 命中率，实现最大化的性能。

你可以通过修改配置文件或者`taskset`命令绑定。

可分为三个模块。

- 主线程和 I/O 线程：负责命令读取、解析、结果返回。命令执行由主线程完成。
- bio 线程：负责执行耗时的异步任务，如 close fd、AOF fsync 等。
- 后台进程：fork 子进程（RDB bgsave、AOF rewrite bgrewriteaof）来执行耗时的命令。

Redis 支持分别配置上述模块的 CPU 亲合度，默认情况是关闭的。

- `server_cpulist 0-7:2`，I/O 线程（包含主线程）相关操作绑定到 CPU 0、2、4、6。
- `bio_cpulist 1,3`，bio 线程相关的操作绑定到 CPU 1、3。
- `aof_rewrite_cpulist`，aof rewrite 后台进程绑定到 CPU 8、9、10、11。
- `bgsave_cpulist 1,10-11`，bgsave 后台进程绑定到 CPU 1、10、11。

**注意事项**

1. Linux 下，使用 **numactl --hardware** 查看硬件布局，确保支持并开启 NUMA。
2. 线程要尽可能分布在 **不同的 CPU，相同的 node**，设置 CPU 亲和度才有效，否则会造成频繁上下文切换。
3. 你要熟悉 CPU 架构，做好充分的测试。否则可能适得其反，导致 Redis 性能下降。

### 5.4.11 sentinel.conf 哨兵

要配置 Redis 哨兵集群，需要修改 Redis 哨兵配置文件，其默认名称为 sentinel.conf。Redis 哨兵配置文件可以设置哨兵节点的域名、端口号、监视频率、故障切换等参数。

#### protected-mode

Sentinel 的保护模式（protected mode）默认配置是 `no`表示保护模式被禁用，Sentinel 可以从本地主机以外的接口访问，这意味着 Sentinel 可以从除本地主机以外的其他网络接口进行访问。但为了安全起见，应确保通过防火墙等方式限制 Sentinel 的外部访问，出绑定的 ip 地址外。

#### port 26379

此 Sentinel 实例运行的端口。

#### daemonize 与 pidfile

`daemonize` 默认配置是 `no`，Redis Sentinel 不作为守护进程运行。也就是说，它会在前台运行，而不是在后台默默地执行。如果需要将 Redis Sentinel 运行为守护进程，可以设置为 “yes”。

`pidfile` 默认配置是 `/var/run/redis-sentinel.pid`，当 Redis Sentinel 被设置为守护进程时，Redis 会将进程 ID（pid）写入到 `/var/run/redis-sentinel.pid` 文件中，以便后续可以通过这个文件来管理或停止 Redis Sentinel 进程。

#### logfile

这一行配置用于指定日志文件的名称。默认配置是空字符串 `""`，表示不指定具体的日志文件名。这样的设置将强制 Sentinel 将日志输出到标准输出（stdout）。

**注意**，如果你使用标准输出（stdout）进行日志记录但又将 Sentinel 设置为守护进程（daemonize），那么日志将被发送到 `/dev/null`，即被丢弃。

#### `dir <working-directory>`

这个配置项设置了 Sentinel 进程的工作目录。对于长时间运行的进程，通常建议有一个明确定义的工作目录。

#### sentinel monitor

这是关键的一个配置，配置语法是`sentinel monitor <master-name> <ip> <redis-port> <quorum>`。比如配置成 `sentinel monitor mymaster 127.0.0.1 6379 2`。以下是配置各字段的详细解释。

- `sentinel monitor <master-name>`: 这里指定了要监控的 master 实例的名称，这里是 "mymaster"。这个名称在整个 Sentinel 群中必须是唯一的。
- `<ip>`: 这是 Redis 主服务器的 IP 地址。在这个例子中，IP 地址是 `127.0.0.1`，表示 Redis 主服务器运行在本地。
- `<redis-port>`: 这是 Redis master 的端口号。在这个例子中，端口号是 `6379`。
- `<quorum>`: 这是一个定义 Sentinel 节点达成一致所需的最小数量的参数。只有在至少 `<quorum>` 个 Sentinel 节点同意 master 进入 `O_DOWN`（主观下线）状态时，主服务器才会被认为处于 `O_DOWN`（客观下线） 状态。在这个例子中，`2` 表示需要至少两个 Sentinel 同意。

#### sentinel auth-pass、sentinel auth-user

这两个配置是关于在 Sentinel 中设置用于与 master 和 slave 进行身份验证的密码或用户名的部分。

示例如下。

```bash
sentinel auth-pass mymaster secret-0123passw0rd
sentinel auth-user <master-name> <username>
```

- `sentinel auth-pass <master-name> <password>`：这里设置了哨兵与 master 和 slave 进行身份验证的密码。在这个例子中，`mymaster` 是主服务器的名称，`MySUPER--secret-0123passw0rd` 是用于身份验证的密码。这个密码与 master 和 slave 的身份验证密码相匹配。
- `sentinel auth-user <master-name> <username>`：这里设置用于与具有 ACL（Access Control List）功能的 Redis 服务器进行身份验证的用户名。ACL 是 Redis 6.0 及更高版本中引入的功能。如果只提供 `auth-pass`，那么 Sentinel 将使用旧的 `AUTH <pass>` 方法进行身份验证。当提供用户名时，将使用 `AUTH <user> <pass>"`方法进行身份验证。

如果配置 `sentinel auth-user`， Redis 必须配置 ACL（Access Control List）来向 Sentinel 实例提供对其进行最小访问权限的权限。以下是一个示例 ACL 配置。

```bash
user sentinel-user >somepassword +client +subscribe + publish + ping +info +multi +slaveof +config +client +exec on
```

这个 ACL 配置允许 Sentinel 用户执行必要的操作，如客户端连接、订阅和发布消息、执行 PING、获取服务器信息、执行 MULTI、设置服务器为从服务器、配置服务器等。

#### sentinel down-after-milliseconds

默认配置 `sentinel down-after-milliseconds mymaster 30000`，用于设置 Sentinel 在多长时间内没有收到 master 以及该 master 的 slave 的 PING 响应时，将该节点判定为 S\_DOWN 状态（主观下线）的时间，默认是 30 秒。

#### acllog-max-len

`acllog-max-len 128`: 这里设置 ACL Log 的最大条目长度为 128。这表示 ACL Log 将最多存储 128 条记录，超过这个数目后会按照先进先出（FIFO）的原则丢弃旧的记录。你可以根据实际需求和系统资源设置这个值。

#### 哨兵身份验证

这部分是有关 Sentinel 的身份验证和密码设置,这些配置项一起提供了对 Sentinel 实例进行身份验证的方式，以确保 Sentinel 之间的安全通信。

##### `requirepass <password>`

这一项配置用于为 Sentinel 本身设置密码。然而，需要注意的是，如果设置了 `requirepass`，Sentinel 会尝试使用相同的密码对所有其他 Sentinel 进行身份验证。因此，在给定的 Sentinel 组中，你需要为所有的 Sentinel 配置相同的 `requirepass` 密码。

需要注意的是,从 Redis 6.2 开始，不再推荐使用`requirepass` 已经成为 ACL 系统的兼容性层。推荐使用 `sentinel sentinel-user <username>` 和 `sentinel sentinel-pass <password>`。

##### sentinel sentinel-pass、sentinel-user

- `sentinel sentinel-user <username>`: 这一项配置允许你为 Sentinel 配置特定的用户名，以便用于与其他 Sentinel 进行身份验证。
- `sentinel sentinel-pass <password>`: 这是 Sentinel 用于与其他 Sentinel 进行身份验证的密码。如果没有配置 `sentinel-user`，那么 Sentinel 将使用默认的用户 `'default'` 进行身份验证，并使用 `sentinel-pass` 指定的密码。

#### sentinel parallel-syncs

`sentinel parallel-syncs <master-name> <numreplicas>`: 这个配置项用于指定在故障切换期间可以多少个 slave 指向新 master 进行主从同步。在进行同步时，如果 slave 提供读处理，建议设置较低的数字，以避免所有 slave 在同一时间变得不可用。这有助于减轻切换期间的负载。

`mymaster` 是要监视的 master 的名称，`numreplicas` 是可以同时同步的 slave 的数量。

#### sentinel failover-timeout

`sentinel failover-timeout <master-name> <milliseconds>`: 该配置项指定了**故障转移的超时时间**，单位为毫秒。这个超时时间在不同的情况下有不同的作用。

- 当前一个 Sentinel 尝试对同一个主节点执行故障转移失败，并且需要重新开始故障转移时，会等待两倍于故障超时时间的间隔。
- 当一个 slave 根据 Sentinel 的当前配置，正在与错误的 master 进行复制时，会被强制与正确的主节点进行复制，这个过程的时间正好是故障超时时间。
- 当取消一个已经在进行但没有产生任何配置更改的故障转移时，需要的时间也是故障超时时间的一部分。
- 当进行 failover 时，配置所有 slaves 指向新的 master 所需的最大时间。即使过了这个超时，slaves 依然会被正确配置为指向 master。

#### SENTINEL master-reboot-down-after-period

这个配置项的作用在于控制 master 重启后 Sentinel 的是否就进行故障切换。默认配置是`SENTINEL master-reboot-down-after-period mymaster 0`。

- 当 `master_reboot_down_after_period` 被设置为 `0` 时，表示主节点在重启后 Sentinel 不会进行故障切换，即不会因为主节点的 `-LOADING` 响应而触发故障转移。这个配置在 Redis 7.0 之前是唯一支持的行为。
- 如果设置了一个非零的值，表示 Sentinel 在 master 重启后等待的时间（以毫秒为单位），在这个时间范围内如果收到主节点的 `-LOADING` 响应，则 Sentinel 不会触发故障切换，超过这个时间才会考虑触发切换。

## 5.5 缓存击穿、缓存穿透、缓存雪崩怎么解决？

原始数据存储在 DB 中（如 MySQL），但 DB 的读写性能低、延迟高。比如 MySQL 在 4 核 8G 上的 TPS = 5000，QPS = 10000 左右，读写平均耗时 10\~100 ms。

用 Redis 作为缓存系统正好可以弥补 DB 的不足，码哥在自己的 MacBook Pro 2019 上执行 Redis 性能测试如下。

```bash
> redis-benchmark -t set,get -n 100000 -q
SET: 107758.62 requests per second, p50=0.239 msec
GET: 108813.92 requests per second, p50=0.239 msec
```

**TPS 和 QPS 达到 10 万**，于是我们引入缓存架构，**当请求进来的时候，先从缓存中去数据，如果有则直接返回缓存中的数据。**

**如果缓存中没数据，就去数据库中读取数据并写到缓存中，再返回结果。**

这样就天衣无缝了么？缓存的设计不当，将会导致严重后果，本篇将介绍缓存使用中常见的三个问题和解决方案。

- **缓存穿透**指的是数据库本就没有这个数据，请求直奔数据库，缓存系统形同虚设。
- **缓存击穿（失效）**指的是数据库有数据，缓存本应该也有数据，但是缓存过期了，Redis 这层流量防护屏障被击穿了，请求直奔数据库。
- **缓存雪崩**指的是**大量**的热点数据无法在 Redis 缓存中处理（大面积热点数据缓存失效、Redis 宕机），流量全部打到数据库，导致数据库极大压力。

### 5.5.1 缓存击穿（失效）

**高并发流量，访问的这个数据是热点数据，请求的数据在 DB 中存在，但是 Redis 存的那一份已经过期，应用程序需要从 DB 从加载数据并写到 Redis。**

**关键字：单一热点数据、高并发、数据失效**

但是由于高并发，可能会把 DB 压垮，导致服务不可用。

![图 5-6][image-136]

图 5-6

#### 方案一：过期时间 + 随机值

对于热点数据，我们不设置过期时间，这样就可以把请求都放在缓存中处理，充分把 Redis 高吞吐量性能利用起来。或者过期时间再加一个随机值。

设计缓存的过期时间时，使用公式：过期时间=baes 时间+随机时间。

即相同业务数据写缓存时，**在基础过期时间之上，再加一个随机的过期时间**，避免瞬时全部过期，对 DB 造成过大压力。

#### 方案二：预热

预先把热门数据提前存入 Redis 中，并设热门数据的过期时间超大值。

#### 方案三：使用锁

当应用程序查询数据发现 Redis 缓存失效的时候，不是直接立即从数据库加载数据。而是先获取分布式锁，获取锁成功才执行数据库查询和写数据到缓存的操作；获取锁失败，则说明当前有线程在执行数据库查询操作，当前线程睡眠一段时间再重试。

伪代码如下。

```java
public Object getData(String id) {
    String desc = redis.get(id);
    // Redis 缓存未命中
    if (desc == null) {
        // 互斥锁，只有一个请求可以成功
        if (redis(lockName)) {
            try
                // 从数据库取出数据
                desc = getFromDB(id);
                // 写到 Redis
                redis.set(id, desc, 60 * 60 * 24);
            } catch (Exception ex) {
                log.error(ex);
            } finally {
                // 确保最后删除，释放锁
                redis.del(lockName);
                return desc;
            }
        } else {
            // 否则睡眠200ms，接着获取锁
            Thread.sleep(200);
            return getData(id);
        }
    }
}
```

### 5.5.2 缓存穿透

缓存穿透：意味着有特殊请求在查询一个不存在的数据，**即数据不存在 Redis 也不存在于数据库。**

导致每次请求都会**穿透**直接打到数据库，Redis 缓存成了摆设，对数据库产生很大压力从而影响正常服务。

![图 5-7][image-137]

图 5-7

#### 方案一

缓存空值：当请求的数据不存在 Redis 也不存在数据库的时候，设置一个缺省值（比如：-1）。当后续再次进行查询则直接返回空值或者缺省值。

#### 方案二

布隆过滤器：当数据写入数据库的同时将这个 ID 同步到到布隆过滤器中，当请求的 ID 不存在布隆过滤器中则说明该请求查询的数据一定没有在数据库中保存，就不要去数据库查询了。

BloomFilter 要缓存全量的 key，这就要求全量的 key 数量不大，10 亿 条数据以内最佳，因为 10 亿 条数据大概要占用 1.2GB 的内存。

BloomFilter 的算法是，首先分配一块内存空间做 bit 数组，数组的 bit 位初始值全部设为 0。

加入元素时，采用 k 个相互独立的 Hash 函数计算，然后将元素 Hash 映射的 K 个位置全部设置为 1。

检测 key 是否存在，仍然用这 k 个 Hash 函数计算出 k 个位置，如果位置全部为 1，则表明 key 存在，否则不存在。

![图 5-8][image-138]

图 5-8

### 5.5.3 缓存雪崩

缓存雪崩指的是**大量的请求无法在 Redis 缓存系统中处理，请求全部打到数据库，导致数据库压力激增**，甚至宕机。

出现该原因主要有两种。

- 大量热点数据同时过期，导致大量请求需要查询数据库并写到缓存。
- Redis 故障宕机、缓存服务器故障、扩容、缓存系统异常。

**缓存雪崩是发生在大量数据同时失效的场景，而缓存击穿（失效）是在某个热点数据失效的场景，这是他们最大的区别。**

![图 5-9][image-139]

图 5-9

#### 方案一

**过期时间添加随机值**

设置缓存数据的过期时间时，尽量避免同时设置相同的过期时间，防止在某个时间点大量数据同时失效。可以设置一个时间范围内的随机过期时间，分散缓存失效的时间点。

`过期时间 = baes 时间+ 随机时间`（较小的随机数，比如随机增加 1\~5 分钟）。

这样一来，就不会导致同一时刻热点数据全部失效，同时过期时间差别也不会太大，既保证了相近时间失效，又能满足业务需求。

#### 方案二

**接口限流**

我们在**业务系统的请求入口前端控制每秒进入系统的请求数，避免过多的请求被发送到数据库。**

当访问的不是核心数据的时候，在查询的方法上加上**接口限流保护**。比如设置 10000 req/s，这样的话，只有部分请求会发送到数据库，减少了压力。

![图 5-10][image-140]

图 5-10

#### 方案三

**使用多级缓存**

将缓存分为多级，例如本地缓存、分布式缓存，甚至页面缓存，这样即使某一级缓存发生故障或者失效，其他级别的缓存依然可用，降低了缓存失效的风险。比如开启 Redis 客户端缓存机制。

#### 方案四

以上方案都是在 Redis 这个正常运行的情况下的解决方式，对于缓存系统故障导致的缓存雪崩的解决方案有两种。

- 服务熔断和接口限流。
- 构建高可用缓存集群系统。

**服务熔断和限流**

在业务系统中，**针对高并发的使用服务熔断来有损提供服务从而保证系统的可用性。**

**服务熔断就是当从缓存获取数据发现异常，则直接返回错误数据给前端，防止所有流量打到数据库导致宕机。**

**服务熔断和限流属于在发生了缓存雪崩，如何降低雪崩对数据库造成的影响的方案。**

**构建高可用的缓存集群**

缓存系统一定要构建一套 Redis 高可用集群，比如 Redis 哨兵集群或者 Redis Cluster 集群，如果 Redis 的 master 故障宕机了，从节点还可以切换成为主节点，继续提供缓存服务，避免了由于缓存实例宕机而导致的缓存雪崩问题。

## 5.6 Redis 缓存策略与数据库一致性问题深度剖析

Redis 拥有高性能的数据读写功能，被我们广泛用在缓存场景，一是能提高业务系统的性能，二是为数据库抵挡了高并发的流量请求。

今天码哥跟你一起深入探索**缓存的工作机制和缓存一致性应对方案**。在本文正式开始之前，我觉得我们需要先取得以下两点的共识。

1. 缓存必须要有过期时间。
2. 保证数据库跟缓存的最终一致性即可，不必追求强一致性。

### 5.6.1 缓存的使用策略

缓存使用策略包括 Cache-Aside（旁路缓存）、Read/Write Through （读写穿透）和 Write-Behind（异步缓存写入）。

这些策略决定了在何时、如何以及何地将数据加载到缓存中，以及如何处理缓存和底层数据存储之间的更新。

**选择适当的缓存使用策略取决于应用程序的需求和性能目标**。例如，Cache-Aside 适用于读多写少的场景，Read Through 和 Write Through 通常配合一起使用，适用于需要自动加载和同步的场景，而 Write-Behind 适用于需要延迟写入和异步刷新的场景。

#### 1. Cache-Aside (旁路缓存)

Cache-Aside，也称为旁路缓存或手动加载缓存，**读取缓存、读取数据库和更新缓存的操作都在应用系统来完成**，**业务系统最常用的缓存策略**。在这种策略中，缓存的加载和维护由应用程序负责，而不是由缓存系统自动管理。

##### 读取数据

![图 5-11][image-141]

图 5-11

**读取数据**逻辑如下。

1. **读取数据**：应用程序需要读取数据时，先检查缓存是否命中。
2. **缓存未命中：** 如果数据不在缓存中（缓存未命中），应用程序从数据库中获取数据。
3. **加载到缓存：** 从数据库获取数据后，应用程序手动将数据加载到缓存中，以便后续读取相同数据会命中缓存。
4. **返回数据：** 应用程序将获取到的数据返回给调用方。

时序图如下。

![图 5-12][image-142]

图 5-12

##### 写数据

![图 5-13][image-143]

图 5-13

1. 应用程序首先更新数据库数据。
2. 手动更新或清除相应的缓存项，以确保缓存与数据库保持一致。

通常我们选择**删除缓存使缓存数据失效**来确保缓存与数据库的数据一致性。

> Chaya：“为啥不是更新缓存呢？”

**性能问题**

当缓存的更新成本很高，需要访问多张表联合计算，建议直接删除缓存，而不是更新缓存数据来保证一致性。

**安全问题**

在高并发场景下，可能会造成查询查到的数据是旧值，具体待会码哥会分析，大家别急。

**优点**

- 缓存中仅包含应用程序实际请求的数据，有助于保持缓存大小的成本效益。
- 实现简单，并且能获得性能提升。

**缺点**

- 需要手动管理：应用程序需要显式地管理缓存，包括加载、更新和清除。这可能增加开发和维护的复杂性。
- 缓存不一致：由于是手动管理，存在数据在数据存储中更新但缓存未及时更新的情况，导致缓存与数据存储不一致。

**适用场景**

- 对于读频繁、写相对较少的场景。
- 需要对缓存的加载和更新时间进行精细控制的场景

#### 2. Read-Through（读穿透）

Read-Through 缓存策略是一种自动加载缓存数据的策略，当应用程序访问缓存时，如果缓存中不存在所需数据，**缓存系统会自动从数据存储中读取数据并将其加载到缓存中**。

Cache-Aside 和 Read-Through 非常相似，**区别在于前者由应用程序负责读取数据库获取数据和填充缓存；后者将应用程序读取数据库数据以及填充到缓存的责任放到缓存系统中。**

![图 5-14][image-144]

图 5-14

**Read-Through 实现了关注点分离原则。应用程序只与缓存系统交互，由缓存系统来管理自身与数据库之间的数据同步。**

缓存系统会自动向数据存储（通常是数据库）发送读取请求，获取所需数据。

获取数据后，缓存系统将数据加载到缓存中，并返回给应用程序，同时也会在下一次相同数据的访问时直接从缓存中读取。

**实际上，Read-Through 缓存策略是在缓存层和数据存储之间插入了一个透明的加载过程，应用程序无需关心数据加载的具体细节。这个过程完全由缓存系统自动管理。**

**优点**

- **自动加载：** 数据在缓存中不存在时，自动触发加载操作，简化了应用程序的逻辑。
- **透明性：** 应用程序无需关心数据是否在缓存中，读取数据的方式与直接从数据存储中读取数据一致。

**缺点**

- **延迟：** 首次访问未命中时，需要从数据存储中加载数据，可能引入一定的延迟。
- **并发加载：** 多个并发请求首次读取相同数据时，可能导致并发加载多次相同的数据。

**适用场景**

- **读取频繁：** 适用于读取频繁，但相对不经常写入的场景。
- **数据相对稳定：** 适用于数据相对稳定，不经常变化的场景。

#### 3. Write-Through （写穿透）

Write-Through 缓存策略是一种同步直写策略，当应用程序写入数据时，数据会同步写入缓存和数据库。

与 Read-Through 类似，发生写请求时，Write-Through **将写入责任转移到缓存系统，由缓存抽象层来完成写缓存数据和数据库的更新。**

![图 5-15][image-145]

图 5-15

1. **应用程序写入：** 当应用程序需要写入数据时，首先将写入请求发送给缓存层。
2. **数据写入缓存：** 缓存系统首先将数据写入缓存，以确保缓存中的数据是最新的。
3. **同步写入数据库：** 缓存系统接着将相同的写入请求同步发送给数据库。
4. **确认写入完成：** 缓存系统等待数据库的写入操作完成，确保数据库的数据也是最新的。

Write-Through 策略确保每次写入操作都同步更新了缓存和底层数据存储，保持了两者的一致性。这样，应用程序在写入完成后可以立即从缓存中读取相同的数据，而且数据库中也是最新的。

**一般 Write-Through 需要与 Read-Through 配合使用才有意义。**

![图 5-16][image-146]

图 5-16

这个策略颠倒了 Cache-Aside 填充缓存的顺序，不是在缓存未命中后延迟加载到缓存，而是在**数据先写缓存，接着由缓存系统将数据写到数据库**。

**优点**

- **数据一致性：** 缓存与数据库数据总是最新的，降低了数据不一致的风险。
- **查询性能最佳：** 应用程序写入完成后，缓存和数据存储都是最新的。

**缺点**

- **延迟：** 写入操作需要等待数据库的确认，可能引入一定的延迟。
- **高并发写入：** 在高并发写入场景下，同步写入数据存储的过程可能成为瓶颈。

**适用场景**

- **强一致性要求：** 适用于对数据一致性要求较高的场景。
- **读写比较平衡：** 适用于读写操作相对平衡的场景。
- **实时性要求：** 适用于对数据实时性要求较高的场景。

#### 4. Write-Behind

Write-Behind 缓存策略是一种**异步写入策略**，当应用程序写入数据时，数据首先写入缓存，而后台异步任务负责将这些变更批量刷写到数据库。

![图 5-17][image-147]

图 5-17

这个图一眼看去似乎与 Write-Through 一样，其实不是的，**区别在于最后一个箭头的箭头：它从实心变为线。**这意味着缓存系统将**异步更新数据库数据，应用系统只与缓存系统交互**。

应用程序不必等待数据库更新完成，从而提高应用程序性能，因为对数据库的更新是最慢的操作。

1. **应用程序写入：** 当应用程序需要写入数据时，首先将写入请求发送给缓存系统，缓存系统将数据写入缓存，确保应用程序可以快速完成写入操作。
2. **异步队列：** 缓存系统维护一个异步队列，将写入缓存的数据变更操作添加到队列中。
3. **批量刷写到数据库：**异步任务将异步队列中积累的数据变更操作批量刷写到数据库。
4. **确认刷写完成：** 缓存系统更新相应的状态，标记已刷写的数据变更操作，向应用程序返回写入操作的确认，表示写入已经完成。

通过这种方式，Write-Behind 策略实现了异步批量刷写，避免了每次写入都同步更新底层数据存储，提高了写入的性能。异步队列和定期批量刷写的机制保证了数据最终一致性。

**优点**

- **提高写入性能：** 异步批量刷写减少了每次写入都同步刷写到数据库的开销，提高了写入性能。
- **减少写入延迟：** 应用程序写入完成后，无需等待数据库的确认，减少了写入延迟。

**缺点**

- **数据一致性延迟：** 由于是异步刷写，可能存在一定的数据一致性延迟。
- **可能存在丢失数据：** 在发生系统故障或异常情况下，尚未刷写的数据可能会丢失。

**适用场景**

- **适度一致性要求：** 适用于对一致性要求较为灵活的场景，能够容忍一定的数据一致性延迟。
- **写入密集型场景：** 适用于写入操作较为密集的场景，可以通过批量刷写提高写入性能。

### 5.6.2 缓存与数据库一致性是什么？

一致性主要涉及两个方面。

- **读一致性：** 读一致性要求无论从数据库还是从缓存中读取数据，获取到的都是最新、准确的数据。如果一个写操作已经将数据写入数据库，但缓存中仍然存储的是旧数据，这就导致了读操作的不一致。
- **写一致性：** 写一致性要求在进行写操作时，数据库和缓存的数据都要保持同步。如果一个写操作成功写入了数据库，但由于某种原因缓存中的数据未能及时更新，这就引发了写操作的不一致。

> Chaya：“为何会出现数据一致性问题呢？”

鱼与熊掌号不可兼得，把 Redis 作为缓存的时候，当数据发生改变我们需要双写来保证缓存与数据库的数据一致。

数据库跟缓存，毕竟是两套系统，如果要保证强一致性，势必要引入 `2PC` 或 `Paxos` 等分布式一致性协议，或者分布式锁等等，这个在实现上是有难度的，而且一定会对性能有影响。

### 5.6.3 旁路缓存的问题分析

业务场景用的最多的就是 Cache-Aside\`(旁路缓存) 策略，在该策略下，客户端对数据的读取流程是先读取缓存，如果命中则返回；

未命中，则从数据库读取并把数据写到缓存中，所以**读操作不会导致缓存与数据库的不一致。重点是写操作，数据库和缓存都需要修改，而两者就会存在一个先后顺序，可能会导致数据不一致**。

对于写数据，有两个问题需要考虑。

1. 先操作缓存还是先操作数据库？
2. 当数据发生变化时，选择修改缓存（update），还是删除缓存（delete）？

将这两个问题排列组合，会出现四种方案。

1.  先操作缓存，后操作数据库。
2.  先操作数据库，后操作缓存。
3.  先删除缓存，再更新数据库。
4.  先更新数据库，再删除缓存。

接下来的分析你不必死记硬背，关键在于在推演的过程中只需要考虑以下两个场景会不会带来严重问题即可。

- 第一个操作成功，第二个失败会导致什么问题？
- 在高并发情况下会不会造成读取数据不一致？

#### 1. 先操作缓存，后操作数据库

![图 5-18][image-148]

图 5-18

如果先更新缓存成功，写数据库失败，就会导致缓存是最新数据，数据库是旧数据，两者数据不一致。

此时此刻，其他查询立马请求就会获取缓存中的这个脏数据，而这个数据在数据库中却不存在。**数据库都不存在的数据，缓存并返回客户端就毫无意义了。该方案直接 Pass。**

#### 2.先操作数据库，后操作缓存

**更新缓存失败**

这时候我们来推断下，假如**第一步成功，第二步失败**会导致**数据库是最新数据，缓存是旧数据，数据不一致。**

**高并发场景**

谢霸歌经常 996 腰酸脖子疼，bug 越写越多，就在在“爽到飞” app 上预约按摩推拿到家服务，提升下编程技巧。

![图 5-19][image-149]

图 5-19

谢霸哥发起了预约单，假设现在出现了高并发抢单。

1. 98 号技师先下手为强，点击“抢单”按钮，系统执行 `set 谢霸歌的服务技师 = 98` 的指令将数据写入数据库。准备把数据写到 Redis 缓存的时候网络出现波动，卡顿了，**数据还没来得及写到缓存**。
2. 另一边 520 号技师也点击“抢单”，系统把 `set 谢霸哥的服务技师 = 520`写到数据库中，也成功的把这个数据写到 Redis 缓存中了。
3. 这时候之前的 98 号技师的写缓存请求开始执行，顺利将数据 `set 谢霸歌的服务技师 = 98` 写到 Redis 中。

最后发现，数据库的值是 `set 谢霸哥的服务技师 = 520`，而 Redis 的值是 `set 谢霸歌的服务技师 = 98`。所以，**在高并发的场景中，多线程同时写数据再写缓存，就会出现缓存是旧值，数据库是最新值的不一致情况。**

#### 3. 先删缓存，再更新数据库

依旧按照前面说的套路，假设第一个操作成功，第二个操作失败推断下会发生什么？高并发场景下又会发生什么？

**更新数据库失败**

假设现在有两个请求，写请求 A，读请求 B。写请求 A 第一步先删除缓存成功，写数据到数据库失败，就会导致该次**写数据丢失，数据库保存的是旧值**。

接着另一个读请 B 求进来，发现缓存不存在，从数据库读取旧数据并写到缓存中，数据不一致。

**高并发下的问题**

肖菜鸡下单，数据库中存储的初始化数据是 \``set 肖菜鸡的服务技师 = 待定`。

![图 5-20][image-150]

图 5-20

1. 还是 98 号技师先下手为强，应用系统把 Redis 数据删除，准备将 `set 肖菜鸡的服务技师 = 98`写到数据库的时候发生卡顿，没有写到数据库。
2. 这时候，大堂经理向系统发起查询请求，查下肖菜鸡的预约单有没有技师接待，应用系统在 Redis 未查询到数据，就从数据库读取到旧数据 `set 肖菜鸡的服务技师 = 待定`，并写到 Redis 中。
3. 这时候，原先卡顿的 98 号技师线程醒来，继续把数据 `set 肖菜鸡的服务技师 = 98`写到数据库中。

**该方案 pass，因为第一步成功，第二步失败，会造成数据库是旧数据；Redis 中没数据继续从数据库读取到旧值写入缓存，造成数据不一致。**

#### 4. 先更新数据库，再删缓存

经过前面的三个方案，全都被 pass 了，分析下最后的方案到底行不行。该策略可以知道，在写数据库阶段失败的话就直返返回客户端异常，不需要执行缓存操作了。

所以第一步失败不会出现数据不一致的情况。

**删缓存失败**

**重点在于第一步写最新数据到数据库成功，删除缓存失败怎么办？**

可以把这两个操作放在一个事务中，当缓存删除失败，那就把写数据库回滚。

> 高并发场景下不合适，容易出现大事务，造成死锁问题。

如果不回滚，那就出现数据库是新数据，缓存还是旧数据，数据不一致了，咋办？

所以，我们要想办法让缓存删除成功，不然只能等到有效期失效那可不行。

**使用重试机制。**

比如重试三次，三次都失败则记录日志到数据库，使用分布式调度组件做后续的处理。

在高并发的场景下，**重试最好使用异步方式**，比如发送消息到 mq 中间件，实现异步解耦。

亦或是利用 Canal 订阅 MySQL binlog 日志，监听对应的更新请求，执行删除对应缓存操作。

**高并发场景**

![图 5-21][image-151]

图 5-21

1. 98 号技师先下手为强，接下肖菜鸡的这笔生意，数据库执行 `set 肖菜鸡的服务技师 = 98`成功；还是网络卡顿了下，**没来得及执行删除缓存操作**。
2. 主管 Candy 向系统执行读请求，查下肖菜鸡有没有技师接待，发现缓存中有数据 `肖菜鸡的服务技师 = 待定`，直接返回信息给客户端，**主管以为没人接待**，实际上数据库已经保存 `set 肖菜鸡的服务技师 = 98`d 的数据。
3. 原先 98 号技师卡顿恢复，执行删除 Redis 缓存成功。

**读请求可能出现少量读取旧数据的情况，但是很快旧数据就会被删除，之后的请求都能获取最新数据，问题不大。**

还有一种比较极端的情况，缓存自动失效的时候又遇到了高并发读写的情况，假设这会有两个请求，一个线程 A 做查询操作，一个线程 B 做更新操作，那么会有如下情形产生。

![图 5-22][image-152]

图 5-22

1. 缓存的过期时间到期，缓存失效。
2. 线程 A 读请求读取缓存，没命中，查询数据库得到一个旧的值（因为 B 会写新值，相对而言就是旧的值了），准备**把数据写到缓存时发送网络问题卡顿，还没写到 Redis**。
3. 线程 B 执行写操作，将新值写数据库。
4. 线程 B 执行删除缓存。
5. 线程 A 从卡顿中醒来，把查询到的旧值写到入缓存。导致数据不一致。

> Chaya：“这咋玩，还是出现了不一致的情况啊。”

不要慌，发生这个情况的概率微乎其微，发生上述情况的必要条件是。

1. 步骤 （3）的写数据库操作要比步骤（2）读操作耗时短速度快，才可能使得步骤（4）先于步骤（5）。
2. 缓存刚好到达过期时限。

通常 MySQL 单机的 QPS 大概 5K 左右，而 TPS 大概 1k 左右，（ps：Tomcat 的 QPS 4K 左右，TPS = 1k 左右）。

数据库读操作是远快于写操作的（正是因为如此，才做读写分离），所以步骤（3）要比步骤（2）更快这个情景很难出现，同时还要配合缓存刚好失效。

所以，在用旁路缓存策略的时候，对于写操作推荐使用：**先更新数据库，再删除缓存。**

### 5.6.4 数据库与缓存一致性解决方案

**在说解决方案之前，先达成一个共识，缓存是通过牺牲强一致性来提高性能的。**

这是由**CAP 理论**决定的。缓存系统适用的场景就是非强一致性的场景，它属于`CAP`中的`AP`，如果需要数据库和缓存数据保持强一致，就不适合使用缓存。

**针对 Cache-Aside (旁路缓存) 策略，写操作使用先更新数据库，再删除缓存**的情况下，我们来分析下数据一致性解决方案都有哪些？

- 延时双删策略。
- 删除缓存重试机制。
- 读取 binlog 异步删除缓存。

#### 1. 延时双删

不一定要先操作数据库呀，采用缓存延时双删策略就好啦？

1. 先删除缓存。
2. 写数据库。
3. 休眠 500 毫秒，再次删除缓存。

这样子最多只会出现 500 毫秒的脏数据读取时间。关键是这个休眠时间怎么确定呢？

**延迟时间的目的就是确保读请求结束，写请求可以删除读请求造成的缓存脏数据。**

所以我们需要自行评估项目的读数据业务逻辑的耗时，**在读耗时的基础上加几百毫秒作为延迟时间即可**。

#### 2. 删除缓存重试机制

> Chaya：“缓存删除失败怎么办？比如延迟双删的第二次删除失败，那岂不是无法删除脏数据。”

使用重试机制，保证删除缓存成功。比如重试三次，三次都失败则记录日志到数据库并发送警告让人工介入。在高并发的场景下，**重试最好使用异步方式**，比如发送消息到 mq 中间件，实现异步解耦。

![图 5-23][image-153]

图 5-23

第（5）步如果删除失败且未达到重试最大次数则将消息重新入队，直到删除成功，否则就记录到数据库，人工介入。

该方案有个缺点，就是对业务代码中造成侵入，于是就有了下一个方案，启动一个专门订阅 MySQL binlog 的服务读取需要删除的数据进行缓存删除操作。

#### 3. 读取 binlog 异步删除缓存

![图 5-24][image-154]

图 5-24

1. 更新数据库。
2. 数据库会把操作信息记录在 binlog 日志中。
3. 使用 canal 订阅 binlog 日志获取目标数据和 key。
4. 缓存删除系统获取 canal 的数据，解析目标 key，尝试删除缓存。
5. 如果删除失败则将消息发送到消息队列。
6. 缓存删除系统重新从消息队列获取数据，再次执行删除操作。

### 5.6.5 总结

缓存策略的最佳实践是 **Cache Aside Pattern。**分别分为读缓存最佳实践和写缓存最佳实践。

**读缓存**最佳实践：先读缓存，命中则返回；未命中则查询数据库，再写到数据库。

**写缓存**最佳实践：

- 先写数据库，再操作缓存；
- 直接删除缓存，而不是修改，因为**当缓存的更新成本很高，需要访问多张表联合计算，建议直接删除缓存，而不是更新，另外，删除缓存操作简单，副作用只是增加了一次 chache miss，建议大家使用该策略。**

在以上最佳实践下，为了尽可能保证缓存与数据库的一致性，我们可以采用延迟双删。

防止删除失败，我们采用异步重试机制保证能正确删除，异步机制我们可以发送删除消息到 mq 消息中间件，或者利用 canal 订阅 MySQL binlog 日志监听写请求删除对应缓存。

那么，如果我非要保证绝对一致性怎么办，先给出结论：

**没有办法做到绝对的一致性，这是由 CAP 理论决定的，缓存系统适用的场景就是非强一致性的场景，所以它属于 CAP 中的 AP。**

所以，我们得委曲求全，可以去做到 BASE 理论中说的**最终一致性**。其实一旦在方案中使用了缓存，那往往也就意味着我们放弃了数据的强一致性，但这也意味着我们的系统在性能上能够得到一些提升。

## 5.7 Redis 分布式锁演进原理与实战

Redis 分布式锁使用 `SET` 指令就可以实现了么？分布式锁的门道可没那么简单，我们在网上看到的分布式锁方案可能是有问题的。甚至在很多的公司内部也使用着一些存在明显 bug 的分布式锁方案。

今天，我们一步步深入分布式锁的演进原理，在高并发生产环境中如何正确使用分布式锁。

### 5.7.1 为什么需要分布式锁

在说分布式锁之前，我们先说下为什么需要分布式锁。

在单机部署的时候，我们可以使用 Java 中提供的 JUC 锁机制避免多线程同时操作一个共享变量产生的安全问题。 JUC 锁机制只能保证同一个 JVM 进程中的同一时刻只有一个线程操作共享资源。

一个应用部署多个节点，多个进程如果要修改同一个共享资源，为了避免操作乱序导致的并发安全问题，这个时候就需要引入分布式锁，**分布式锁就是用来控制同一时刻，只有一个 JVM 进程中的一个线程可以访问被保护的资源。**

Redis 本身可以被多个客户端共享放任，而且读写性能强大，可以应对高并发的加锁场景，于是 Redis 分布式锁就诞生了。

> 程旭源：“码哥，说个通俗的例子讲解下什么时候需要分布式锁呢？”

诊所只有一个医生，很多患者前来就诊。

医生在同一时刻只能给一个患者提供就诊服务每个患者需要预约挂号，凭借挂的这个 “号”，才能去找医生就诊。否则就会出现医生给肾亏的“肖菜鸡”准备开药时候，脚臭的“谢霸哥”进来把肾亏的药拿走了。

> 程旭源：“分布式锁有哪些特性呢？”

- **互斥性（Mutual Exclusion）：** 任意时刻只有一个客户端能够持有锁。这确保了在同一时刻只有一个进程能够访问共享资源，避免了竞争条件。
- **不可抢占性（Non-Preemptive）：** 一旦一个节点获得了锁，其他节点不能强制抢夺该锁。只有持有锁的节点可以释放它。
- **有限等待（Finite Wait）：** 如果一个节点请求锁失败，它不能无限期地等待。分布式锁应该具有合理的超时机制，确保在一定时间内获取锁，避免死锁。
- **安全性（Safety）**：加锁和解锁必须是同一个客户端（线程），客户端自己不能把别人加的锁给解了。
- **可重入性（Reentrant）：** 允许同一个节点在持有锁的情况下多次请求锁，而不会造成死锁。
- **高可用性（High Availability）：** 分布式锁的实现应该具备高可用性，确保在节点故障或网络分区的情况下，系统仍能正常工作。
- **容错性（Fault Tolerance）：** 分布式锁应该在系统发生故障或网络分区时依然能够正常运作，不会导致数据不一致或其他问题。

### 5.7.2 入门级分布式锁

> 谢霸哥：“公司的老架构师用 `SETNX key value` 命令来实现互斥性。”

`SETNX` 命令是 `SET if Not eXists`的缩写，意思是如果 `key` 不存在，则设置 `value` 给这个`key`，否则啥都不做。命令返回值 1 表示设置成功，0 表示没有设置成功。

```bash
> SETNX lock:orderID:998 1
integer) 1
```

执行完成之后，只需要调用 `DEL` 删除这个锁即可。

```bash
> DEL lock:orderID:998
(integer) 1
```

![图 5-25][image-155]

图 5-25

> 谢霸哥：“码哥你见过龙么？我没见过，但是我被一条龙服务过。架构师对我说这就是分布式锁一条龙服务，让我放心使用。”

事情可没这么简单。这一个方案会因为客户端所在节点崩溃导致无法执行删除指令释放锁，造成死锁的问题。

> 谢霸哥：“架构师不行呀，还说一条龙服务。我可以在获取锁成功的时候设置一个超时时间。”

```bash
> SETNX lock:orderID:998 1  // 获取锁
(integer) 1
> EXPIRE lock:orderID:998 10  // 10s 自动删除
(integer) 1
```

这样依然有问题，**加锁操作和设置超时时间**是分开的，他们不是原子操作。假如加锁成功，但是设置超时时间的指令没机会执行，依然出现死锁问题。

> 谢霸哥：“那咋办，我想被一条龙服务，要解决这个问题。”

Redis 2.6.X 之后，官方拓展了 `SET` 命令的参数，满足了当 key 不存在则设置 value，同时设置超时时间，并且满足原子性。

```bash
SET lockKey 1 NX PX expireTime
```

- `lockKey` 表示锁的资源，value 设置成 1。
- `NX`：表示只有 `lockKey` 不存在的时候才能 `SET` 成功，从而保证只有一个客户端可以获得锁。
- `PX expireTime`设置锁的超时时间，单位是毫秒；也可以使用 `EX seconds`以秒为单位设置超时时间。

```java
//加锁成功
if（jedis.set(lockKey, 1, "NX", "EX", 10) == 1）{
  try {
      do work //执行业务

  } finally {
    //释放锁，这里可能误删其他客户端的锁
     jedis.del(key);
  }
}
```

### 5.7.3 释放了不是自己加的锁

> 谢霸哥：“这样我能稳妥的享受一条龙服务了么？”

No，有可能出现**释放别人的锁**的情况。

1. 客户端 A 获取锁成功，设置超时时间 10 秒。
2. 客户端 A 执行业务逻辑，但是因为某些原因（网络问题、FullGC、代码垃圾性能差）执行很慢，时间超过 10 秒，锁因为超时自动释放了。
3. 客户端 B 加锁成功。
4. 客户端 A 执行 `DEL` 释放锁，相当于把客户端 B 的锁释放了。

原因很简单：客户端加锁时，没有设置一个唯一标识。释放锁的逻辑并不会检查这把锁的归属，直接删除。

解决方法：客户端加锁时设置一个“唯一标识”，可以让 value 存储客户端的唯一标识，比如随机数、 UUID 等；释放锁时判断锁的唯一标识与客户端的标识是否匹配，匹配才能删除。

```bash
SET lockKey randomValue NX PX 3000
```

删除锁的时候判断唯一标识是否匹配伪代码如下。

```java
if (jedis.get(lockKey).equals(random_value)) {
    jedis.del(lockKey);
}
```

加锁、解锁的伪代码如下所示。

```java
try (Jedis jedis = pool.getResource()) {
  //加锁成功
  if（jedis.set(lockKey, randomValue, "NX", "PX", 3000) == 1）{
    do work //执行业务
  }
} finally {
  //判断是不是当前线程加的锁,是才释放
  if (randomValue.equals(jedis.get(keylockKey {
     jedis.del(lockKey); //释放锁
  }
}
```

到这里，很多公司可能都是使用这个方式来实现分布式锁。

> 谢霸哥：“码哥，还有问题。判断锁的唯一标识是否与当前客户端匹配和删除操作不是原子操作。”

聪明。这个方案还存在原子性问题，存在其他客户端的锁给释放。

1. 客户端 A 执行唯一标识匹配成功，还来不及执行`DEL`释放锁操作，**锁过期被释放**。
2. 客户端 B 获取锁成功，设置了自己的客户端唯一标识。
3. 客户端 A 执行 `DEL`删除锁操作，相当于把客户端 B 的锁给删了。

解决方案很简单，我们可以通过 `Lua` 脚本来实现判断和删除的过程。

```lua
// 获取 KEY[1] 中的 value 是否与 ARGV[1] 匹配，匹配则执行 del
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

使用上面的脚本，每个锁都用一个随机值作为唯一标识，当删除锁的客户端的“唯一标识”与锁的 value 匹配的时候，才能执行删除操作。这个方案已经相对完美，我们用的最多的可能就是这个方案了。

```java
String script =
              "if redis.call('get',KEYS[1]) == ARGV[1] then" +
                      "   return redis.call('del',KEYS[1]) " +
                      "else" +
                      "   return 0 " +
                      "end";

try (Jedis jedis = pool.getResource()) {
  //加锁成功
  if（jedis.set(lockKey, randomValue, "NX", "PX", 3000) == 1）{
    do work //执行业务
    return true;
  }
  return false;
} finally {

  Object result = jedis.eval(script, Collections.singletonList(lockKey)
             , Collections.singletonList(randomValue));
  if("1".equals(result.toString())){
     return true;
  }
  return false;
}
```

有两个点需要注意。

1. 释放锁的代码一定要放在 `finally{}` 块中。否则一旦执行业务逻辑过程中抛出异常，程序就无法执行释放锁的流程。只能干等着锁超时释放。
2. 加锁的代码应该写在 `try {}` 代码中，放在 try 外面的话，如果执行加锁异常（客户端网络连接超时），但是实际指令已经发送到服务端并执行，就会导致没有机会执行解锁的代码。

### 5.7.4 可重入锁

当一个线程执行一段代码成功获取锁之后，继续执行时，又遇到加锁的代码，可重入性就就保证线程能继续执行，而不可重入就是需要等待锁释放之后，再次获取锁成功，才能继续往下执行。

```java
public synchronized void a() {
    b();
}
public synchronized void b() {
    // doWork
}
```

假设 X 线程在 a 方法获取锁之后，继续执行 b 方法，如果此时**不可重入**，线程就必须等待锁释放，再次争抢锁。

锁明明是被 X 线程拥有，却还需要等待自己释放锁，然后再去抢锁，这看起来就很奇怪，我释放我自己\~

#### Redis hash 实现可重入

当线程拥有锁之后，往后再遇到加锁方法，直接将加锁次数加 1，然后再执行方法逻辑。

退出加锁方法之后，加锁次数再减 1，当加锁次数为 0 时，锁才被真正的释放。可以看到可重入锁最大特性就是计数，计算加锁的次数。

所以当可重入锁需要在分布式环境实现时，我们也就需要统计加锁次数。

#### 加锁逻辑

我们可以使用 Redis hash 结构实现，key 表示被锁的共享资源， hash 结构的 fieldKey 存储客户端唯一标识，fieldKey 的 value 则保存加锁的次数。

![图 5-26][image-156]

图 5-26

以下脚本来自于 Redisson 的加锁执行脚本，脚本入参如下。

- `KEYS[1]`表示获取的锁资源，比如 `lock:168`。
- `ARGV[1]` 表示表示锁的有效时间（单位毫秒）。
- `ARGV[2]` 表示客户端唯一标识，在 Redisson 中使用 UUID + ThreadID。

```lua
if ((redis.call('exists', KEYS[1]) == 0) or
        (redis.call('hexists', KEYS[1], ARGV[2]) == 1)) then
    redis.call('hincrby', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
		return nil;
end;
return redis.call('pttl', KEYS[1]);
```

**脚本解读**

1. 锁不存在或者锁存在且值与客户端唯一标识匹配，则执行 `'hincrby'` 和 `pexpire`指令，接着 `return nil`。表示的含义就是锁不存在就设置锁并设置锁重入计数值为 1，设置过期时间；锁存在且唯一标识匹配表明当前加锁请求是锁重入请求，锁从如计数 +1，重新锁超时时间。
   2. `redis.call('exists', KEYS[1]) == 0`判断锁是否存在，0 表示不存在。
   3. `redis.call('hexists', KEYS[1], ARGV[2]) == 1)`锁存在的话，判断 hash 结构中 fieldKey 与客户端的唯一标识是否相等。相等表示当前加锁请求是锁重入。
   4. `redis.call('hincrby', KEYS[1], ARGV[2], 1)`将存储在 hash 结构的 `ARGV[2]` 的值 +1，不存在则支持成 1。
   5. `redis.call('pexpire', KEYS[1], ARGV[1])`对 `KEYS[1]` 设置超时时间。
2. 锁存在，但是唯一标识不匹配，表明锁被其他线程持有，调用 `pttl`返回锁剩余的过期时间。

> 谢霸哥：“脚本执行结果返回 nil、锁剩余过期时间有什么目的？”

当且仅当返回 `nil`才表示加锁成功；客户端需要感知锁是否成功的结果。

#### 解锁逻辑

该解锁脚本依然出自 Redisson 框架，脚本入参如下。

- `KEYS[1]`表示锁的资源，比如 `lockKey`。
- `KEYS[2]`解锁用于 Pub/Sub 发布消息的频道名称，比如 `redisson_lock__channel:{lockKey}`。
- `ARGV[1]`，Redisson 定义 0 表示解锁消息。
- `ARGV[2]`，锁的超时时间，默认值是 30 秒。
- `ARGV[3]`，Hash 表的 FieldKey，存储客户端唯一标识 uuid + threadID。
- `ARGV[4]`，发布订阅的指令 `PUBLISH`。

```lua
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then
    return nil;
end;
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1);
if (counter > 0) then
    redis.call('pexpire', KEYS[1], ARGV[2]);
    return 0;
else
    redis.call('del', KEYS[1]);
    redis.call(ARGV[4], KEYS[2], ARGV[1]);
    return 1;
end;
return nil;
```

**脚本解读**

首先使用 `hexists` 判断 Redis 的 Hash 表是否存在 `fileKey`，如果不存在则直接返回 `nil`。

若存在的情况下，且唯一标识匹配，使用 `hincrby` 对计数器数 -1，然后判断计算之后可重入次数，若值大于 0 表示，当前下次呢很难过持有锁存在重入，重新设置超时时间，返回值 1；若值小于等于 0，表明锁释放了，执行 `del`释放锁，并 执行 `redis.call(ARGV[4], KEYS[2], ARGV[1])`，替换成实际执行的命令是 `PUBLISH redisson_lock__channel:lockKey 0`，广播解锁消息，唤醒那些争抢过锁但还处于阻塞中的线程，并返回值 1。

解锁代码执行方式与加锁类似，只不过解锁的执行结果返回类型使用 `Long`。这是因为解锁 lua 脚本中，三个返回值含义如下。

- 1 代表解锁成功，锁被释放。
- 0 代表可重入次数被减 1。
- `nil` 代表其他线程尝试解锁，解锁失败。

### 5.7.5 正确设置锁超时

至此依然存在的一个问题是：加锁后，业务逻辑执行耗时超过了 lockKey 的过期时间，lockKey 会被 Reids 删除。

> 谢霸哥：“锁的超时时间怎么计算合适呢？”

这个时间不能瞎写，一般要根据在测试环境多次测试，然后压测多轮之后，比如计算出接口平均执行时间 200 ms。那么锁的**超时时间就放大为平均执行时间的 3\~5 倍。**

> 谢霸哥：“为啥要放大呢？”

因为如果锁的操作逻辑中有网络 IO 操作、JVM FullGC 等，线上的网络不会总一帆风顺，我们要给网络抖动留有缓冲时间。

> 谢霸哥：“那我设置更大一点，比如设置 1 小时不是更安全？”

设置时间过长，一旦发生宕机重启，就意味着 1 小时内，分布式锁的服务全部节点不可用。你要让运维手动删除这个锁么？只要运维真的不会打你。

> 谢霸哥：“有没有完美的方案呢？不管时间怎么设置都不大合适。”

我们可以让获得锁的线程开启一个**守护线程**，用来给当前客户端快要过期的锁续航，续命的前提是，得判断是不是当前进程持有的锁，如果不是就不进行续。

**如果快要过期，但是业务逻辑还没执行完成，自动对这个锁进行续期，重新设置过期时间。**

这个玩意呢，已经有一个库把这些工作都封装好了，他叫 **Redisson**，并且把这个机制叫做看门狗。

此时此刻，这个方案实际上已经比较完美，能写到这一步已经打败 90% 的程序猿。总结该方案的分布式锁套路。

1. 通过 `SET lockKey randomValue NX PX expire_time`执行加锁，同时启动守护线程为当前快要过期但还没执行完的客户端的锁续命，续命的前提是，得判断是不是当前进程持有的锁，如果不是就不进行续命，不然会出现死锁的情况（客户端 A 获取锁成功，宕机了；其他守护线程一直给这个锁续命）。
2. 客户端执行业务逻辑操作共享资源。
3. 释放锁时，通过 `Lua` 脚本释放锁，先 get 判断锁是否是自己加的，再执行 `DEL`。

![图 5-27][image-157]

图 5-27

#### 看门狗机制

为了解决过期时间确定和业务执行时长不确定性的问题，引入看门狗机制。服务中需要有一个 **daemon 线程定时去守护我的锁的安全性**，比如说锁超时时间设置的是 1s，那么我这个定时任务是每隔 300ms 去 redis 服务端做一次检查，如果我还持有，你就给我续命，就像 session 会话的活跃机制一样。

Redisson watch dog 自动延时机制原理如下所示。

![图 5-28][image-158]

图 5-28

在 Redisson 中自动续期的功能也是使用 lua 脚本来实现。而定时任务使用的是 netty 框架提供的时间轮 `HashedWheelTimeout`，这是一个支持海量任务的高性能定时器。以下是 Redisson 加锁成功后的看门狗机制部分源码。

```java
// 存储所有需要续命的定时任务
private static final ConcurrentMap<String, ExpirationEntry> EXPIRATION_RENEWAL_MAP = new ConcurrentHashMap<>();

protected void scheduleExpirationRenewal(long threadId) {
    ExpirationEntry entry = new ExpirationEntry();
    ExpirationEntry oldEntry = EXPIRATION_RENEWAL_MAP.putIfAbsent(getEntryName(), entry);
    if (oldEntry != null) {
        oldEntry.addThreadId(threadId);
    } else {
        entry.addThreadId(threadId);
        try {
            // 开启守护线程续命
            renewExpiration();
        } finally {
            if (Thread.currentThread().isInterrupted()) {
                // 线程中断，取消获取锁的守护线程定时续期
                cancelExpirationRenewal(threadId);
            }
        }
    }
}
```

1. 首先用一个 `EXPIRATION_RENEWAL_MAP` HashMap 保存一个服务中所有需要续期的分布式锁客户端守护线程信息，key 就是客户端的唯一标识。
2. `getEntryName`就是分布式锁的唯一标识，如果获取锁的客户端没有开启过守护线程，就使用新创建的 entry 存到 `EXPIRATION_RENEWAL_MAP`。
3. 重点关注 `renewExpiration()` 方法：获取当前获取锁的客户端 `ExpirationEntry`，并使用时间轮创建一个定时任务，执行续命功能。`ExpirationEntry` 你可以理解为存储每个获取分布式锁的客户端信息（定时任务、续命次数、线程 ID）
4. `cancelExpirationRenewal(threadId)`，取消守护线程续命，移除 `EXPIRATION_RENEWAL_MAP` 中存储的 `ExpirationEntry`并调用时间轮的 `cancel`函数取消定时任务。

重点关注 `renewExpiration` 开启续命的执行逻辑，关键位置我都加了解释。

```java
private void renewExpiration() {
   // 获取锁的客户端看门狗
    ExpirationEntry ee = EXPIRATION_RENEWAL_MAP.get(getEntryName());
    if (ee == null) {
        return;
    }

  	// 使用时间轮创建一个定时任务为锁续命
    Timeout task = getServiceManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            ExpirationEntry ent = EXPIRATION_RENEWAL_MAP.get(getEntryName());
            if (ent == null) {
                return;
            }
            Long threadId = ent.getFirstThreadId();
            if (threadId == null) {
                return;
            }

            // 异步执行续命逻辑
            CompletionStage<Boolean> future = renewExpirationAsync(threadId);
            // 续命执行出现异常，则移除该定时任务
            future.whenComplete((res, e) -> {
                if (e != null) {
                    log.error("Can't update lock {} expiration", getRawName(), e);
                    EXPIRATION_RENEWAL_MAP.remove(getEntryName());
                    return;
                }
								 // 执行成功的话，递归调用 renewExpiration 重复续命，失败则移除
                if (res) {
                    // reschedule itself
                    renewExpiration();
                } else {
                   // 失败的话就移除续命定时任务。
                    cancelExpirationRenewal(null);
                }
            });
        }
    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
    // 将这个定时任务设置到 ExpirationEntry 中，以便下一轮定时触发执行
    ee.setTimeout(task);
}
```

续命执行的指令封装在 `renewExpirationAsync`方法中。里面执行以下 lua 脚本实现续命。解释下每个参数的含义。

- `KEYS[1]`表示锁的资源，比如 `lockKey`。
- `ARGV[1]` Hash 表的 FieldKey，存储客户端唯一标识 uuid + threadID。
- `ARGV[2]`超时时间。

```lua
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return 1;
end;
return 0;
```

需要注意的地方是：在实现自动续期功能时，还需要设置一个总的过期时间，可以跟 redisson 保持一致，设置成 30 秒。如果业务代码到了这个总的过期时间，还没有执行完，就不再自动续期了。

### 5.7.6 Redis 部署方式对锁的影响

> 谢霸哥：“码哥，到这里分布式锁很完美了吧，没想到分布式锁这么多门道。”

路还很远，之前分析的场景都是锁在单个 Redis 实例中可能产生的问题，并没有涉及到 Redis 主从模式、哨兵集群、或者 Cluster 集群导致的问题。

我们通常使用 Cluster 集群或者哨兵集群的模式部署保证高可用。这两个模式都是基于主从架构数据同步复制实现的数据同步，而 Redis 的主从复制默认是异步的。

我们试想下如下场景会发生什么问题。

1. 客户端 A 在 master 节点获取锁成功。
2. master 还没有把获取锁的信息同步到 slave 的时候，master 宕机。
3. slave 被选举为新 master，数据库没有客户端 A 获取锁的数据。
4. 客户端 B 就能成功的获得客户端 A 持有的锁，违背了分布式锁定义的互斥。

此外，假如我们没有设置 Redis 的持久化，也会出现上面的问题。如果我们启用 AOF 持久化，情况将会改善很多。虽然这个概率极低，但是我们必须得承认这个风险的存在。

### 5.7.7 Redlock 红锁

Redis 的作者为了统一分布式锁的标准，Redis 的作者提出了一种解决方案，叫 Redlock（红锁），但是这个 Redlock 也被国外的一些分布式专家给喷了。因为它也不完美，有“漏洞”。

> 谢霸哥：“红锁是不是这个？”

![5-29][image-159]

图 5-29

泡面吃多了你，`Redlock` 红锁是为了解决主从架构中当出现主从切换导致多个客户端持有同一个锁而提出的一种算法。

想用使用 Redlock，官方建议在不同机器上部署 5 个 Redis 主节点，节点都是完全独立互不相干，也不使用主从复制，使用多个节点是为容错。

获取分布式锁，客户端需要执行一下操作。

1. 客户端获取当前时间 `T1`（毫秒级别）。
2. 尝试在所有的实例中获取相同的锁，键值对相同。
   1. 每个请求都设置一个超时时间（毫秒级别），该超时时间要远小于锁的有效时间，这样便于快速尝试与下一个实例发送请求。
   2. 比如锁的自动释放时间 `10s`，则请求的超时时间可以设置 `5~50` 毫秒内，这样可以防止客户端长时间阻塞。
3. 客户端获取当前时间 `T2` 并减去步骤 1 的 `T1` 来计算出获取锁所用的时间（`T3 = T2 -T1`）。**当且仅当客户端在大多数实例（`N/2 + 1`）获取成功，且获取锁所用的总时间 T3 小于锁的有效时间，才认为加锁成功，否则加锁失败。**
4. 如果第 3 步加锁成功，则执行业务逻辑操作共享资源，**key 的真正有效时间等于有效时间减去获取锁所使用的时间（步骤 3 计算的结果）。**
5. 如果因为某些原因，获取锁失败（没有在至少 N/2+1 个 Redis 实例取到锁或者取锁时间已经超过了有效时间），**客户端应该在所有的 Redis 实例上进行解锁**（即便某些 Redis 实例根本就没有加锁成功）。

**另外部署实例的数量要求是奇数，为了能很好的满足过半原则，如果是 6 台则需要 4 台获取锁成功才能认为成功，所以奇数更合理**。

Redis 作者把这个方案提出后，受到了业界著名的分布式系统专家的**质疑**。两人好比神仙打架，两人一来一回论据充足的对一个问题提出很多论断……

### 5.7.8 Redlock 是与非

Martin Kleppmann 认为分布式锁的目的是为了保护对共享资源的读写，并且应该“高效”和“正确”。

- 高效性：分布式锁应该要满足高效的性能，Redlock 算法向 5 个节点执行获取锁的逻辑性能不高，成本增加，复杂度也高。
- 正确性：分布式锁应该防止并发进程在同一时刻只能有一个线程能对共享数据读写。

对于高效性，我们没必要承担 Redlock 的成本和复杂，运行 5 个 Redis 实例并判断加锁是否满足大多数才算成功。主从架构崩溃恢复极小可能发生，这没什么大不了的。使用单机版就够了，Redlock 太重了，没必要。

**Martin** 认为 **Redlock** 根本达不到安全性的要求，也依旧存在锁失效的问题！

#### Martin 的结论

1. **Redlock** 不伦不类：对于偏好效率来讲，**Redlock** 比较重，没必要这么做，而对于偏好正确性来说，**Redlock** 是不够安全的。
2. 时钟假设不合理：该算法对系统时钟做出了危险的假设（假设多个节点机器时钟都是一致的），如果不满足这些假设，锁就会失效。
3. 无法保证正确性：**Redlock** 不能提供类似 **fencing token** 的方案，所以解决不了正确性的问题。为了正确性，请使用有“共识系统”的软件，例如 **Zookeeper**。

#### Redis 作者 Antirez 的反驳

在 **Redis** 作者的反驳文章中，有 3 个重点：

- 时钟问题：**Redlock** 并不需要完全一致的时钟，只需要大体一致就可以了，允许有「误差」，只要误差不要超过锁的租期即可，这种对于时钟的精度要求并不是很高，而且这也符合现实环境。
- 网络延迟、进程暂停问题。
  - 客户端在拿到锁之前，无论经历什么耗时长问题，**Redlock** 都能够在第 3 步检测出来
  - 客户端在拿到锁之后，发生 **NPC**，那 **Redlock、Zookeeper** 都无能为力
- 质疑 fencing token 机制。

[image-1]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-1.png
[image-2]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-2.png
[image-3]:	https://files.mdnice.com/user/363/1b5b4006-4120-4d67-ac91-395acc7893a4.png
[image-5]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-4.png
[image-6]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-5.png
[image-7]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-6.png
[image-8]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-7.png
[image-9]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-8.png
[image-10]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-9.png
[image-11]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%9B%BE1-11.png
[image-12]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/1-10.drawio%20(1).png
[image-13]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/1-12.drawio.png
[image-14]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/1-13.drawio.png
[image-15]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-1.drawio.png
[image-16]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-2.drawio-2.png
[image-17]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-3.drawio.png
[image-18]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-4.drawio.png
[image-19]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-5.drawio.png
[image-20]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-6.drawio.png
[image-21]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-7.drawio.png
[image-22]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-8.drawio.png
[image-23]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-9.png
[image-24]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-10.drawio.png
[image-25]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-11.drawio.png
[image-26]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/clean-code%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E6%A6%82%E8%BF%B0.png
[image-27]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/clean-codeList%E9%98%9F%E5%88%97.png
[image-28]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-14.drawio.png
[image-29]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-15.drawio.png
[image-30]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-16.drawio.png
[image-31]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-17.drawio.png
[image-32]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-18.drawio.png
[image-33]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-19.drawio.png
[image-34]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/21-8.drawio.png
[image-35]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-21.png
[image-36]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-22.drawio.png
[image-37]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-23.drawio.png
[image-38]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-24.drawio.png
[image-39]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-25.drawio.png
[image-40]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-26.drawio.png
[image-41]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-27.drawio.png
[image-42]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-28.drawio.png
[image-43]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-29.drawio%20(1).png
[image-44]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-30.drawio.png
[image-45]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-31.drawio.png
[image-46]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-32.drawio.png
[image-47]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-33.drawio.png
[image-48]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-34.drawio.png
[image-49]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-35.drawio.png
[image-50]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-36.png
[image-51]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/consumerGroup.png
[image-52]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/MySQL%E9%99%84%E8%BF%91%E7%9A%84%E4%BA%BA.png
[image-53]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Hash%E4%BE%8B%E5%AD%90LBS.png
[image-54]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/SortedSet%E4%BE%8B%E5%AD%90LBS.png
[image-55]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/GeoHash.png
[image-56]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/2-2.drawio-2.png
[image-57]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/RedisObject.png
[image-58]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20230725192010.png
[image-59]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202309092130830.png
[image-60]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220320143811.png
[image-61]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202309122042681.png
[image-62]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F%E8%A7%A3%E5%86%B3.drawio.png
[image-63]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210117162311.png
[image-64]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202309231233179.png
[image-65]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210118225652.png
[image-66]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E9%98%BB%E5%A1%9EIO.png
[image-67]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210121002208.png
[image-68]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210120002119.png
[image-69]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202309232033142.png
[image-70]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis%E5%85%A8%E5%B1%80%E5%93%88%E5%B8%8C%E8%A1%A8.png
[image-71]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/RDB%E5%86%85%E5%AD%98%E5%BF%AB%E7%85%A7.png
[image-72]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%86%99%E6%97%B6%E5%A4%8D%E5%88%B6%E6%8A%80%E6%9C%AF.png
[image-73]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/AOF%E6%97%A5%E5%BF%97.png
[image-74]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/AOF%E6%97%A5%E5%BF%97%E6%A0%BC%E5%BC%8F.png
[image-75]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/AOF%E9%87%8D%E5%86%99.png
[image-76]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/AOF%E9%87%8D%E5%86%99%E8%BF%87%E7%A8%8B.png
[image-77]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202309292206814.png
[image-78]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E6%8C%81%E4%B9%85%E5%8C%96%E6%96%87%E4%BB%B6%E6%81%A2%E5%A4%8D%E6%95%B0%E6%8D%AE.png
[image-79]:	http://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis%E4%B8%BB%E4%BB%8E%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB.png
[image-80]:	http://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis%E5%85%A8%E9%87%8F%E5%A4%8D%E5%88%B6%E7%BB%88%E6%9E%81.png
[image-81]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/repl_backlog_buffer.png
[image-82]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis%E5%A2%9E%E9%87%8F%E5%A4%8D%E5%88%B6.png
[image-83]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/repl_backlog%E4%B8%8Erepl_buffer%E5%8C%BA%E5%88%AB.png
[image-84]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202310182227902.png
[image-85]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202310182302792.png
[image-86]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202310182326480.png
[image-87]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%A2%9E%E9%87%8F%E5%85%A8%E9%87%8F%E6%B5%81%E7%A8%8B%E5%9B%BE.png
[image-88]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202310211223679.png
[image-89]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%93%A8%E5%85%B5%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png
[image-90]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%AE%A2%E8%A7%82%E4%B8%8B%E7%BA%BF%E4%B8%8E%E4%B8%BB%E8%A7%82%E4%B8%8B%E7%BA%BFdrawio.png
[image-91]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E6%96%B0master%E9%80%89%E6%8B%A9%E8%BF%87%E7%A8%8B.png
[image-92]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/pub-sub%E6%9C%BA%E5%88%B6.png
[image-93]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/INFO%E5%91%BD%E4%BB%A4%E8%8E%B7%E5%8F%96slave%E4%BF%A1%E6%81%AF.png
[image-94]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis%E5%93%A8%E5%85%B5%E6%89%A7%E8%A1%8C%E4%B8%BB%E4%BB%8E%E5%88%87%E6%8D%A2.png
[image-95]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E6%B0%B4%E5%B9%B3%E4%B8%8E%E5%9E%82%E7%9B%B4%E6%8B%93%E5%B1%95.png
[image-96]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis%E9%9B%86%E7%BE%A4%E6%9E%B6%E6%9E%84.png
[image-97]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/MEET%E8%BF%87%E7%A8%8B.png
[image-98]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Slot%E4%B8%8E%E5%AE%9E%E4%BE%8B%E7%9A%84%E6%98%A0%E5%B0%84.png
[image-99]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E9%9B%86%E7%BE%A4Leader%E9%80%89%E4%B8%BE.png
[image-100]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AE%9A%E4%BD%8D%E6%95%B0%E6%8D%AE%E6%89%80%E5%9C%A8%E8%8A%82%E7%82%B9.png
[image-101]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/MOVED%E6%8C%87%E4%BB%A4.png
[image-102]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/ASK%E9%94%99%E8%AF%AF.png
[image-103]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-transaction.png
[image-104]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-watch.png
[image-105]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-after.png
[image-106]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-eviction.drawio.png
[image-107]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/LRU.png
[image-108]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311192320146.png
[image-109]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-delete2.png
[image-110]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-timeout.png
[image-111]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311222047559.png
[image-112]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311222137660.png
[image-113]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311251651319.png
[image-114]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/pub_sub.drawio.png
[image-115]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220807161809.png
[image-116]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814112343.png
[image-117]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814114904.png
[image-118]:	/Users/magebte/Desktop/4-16.png
[image-119]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814211411.png
[image-120]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311262050126.png
[image-121]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311262148246.png
[image-122]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311281937045.png
[image-123]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311281959163.png
[image-124]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202311282024607.png
[image-125]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202312022154704.png
[image-126]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202312022221393.png
[image-127]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis%E5%A4%9A%E7%BA%BF%E7%A8%8B.png
[image-128]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis%E5%86%85%E5%AD%98%E5%8D%A0%E7%94%A8.png
[image-129]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%86%85%E5%AD%98%E7%A2%8E%E7%89%87.png
[image-130]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220903223140.png
[image-131]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220220143454.png
[image-132]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%86%99%E6%97%B6%E5%A4%8D%E5%88%B6%E6%8A%80%E6%9C%AF.png
[image-133]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220703170339.png
[image-134]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202309232033142.png
[image-135]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20230725192010.png
[image-136]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%BC%93%E5%AD%98%E5%87%BB%E7%A9%BF2.png
[image-137]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F.png
[image-138]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8.png
[image-139]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%BC%93%E5%AD%98%E9%9B%AA%E5%B4%A9-%E5%90%8C%E6%97%B6%E5%A4%B1%E6%95%88.png
[image-140]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%BC%93%E5%AD%98%E9%9B%AA%E5%B4%A9-%E9%99%90%E6%B5%81.png
[image-141]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522212245.png
[image-142]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522214335.png
[image-143]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522212610.png
[image-144]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522215921.png
[image-145]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522220448.png
[image-146]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522231840.png
[image-147]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220522232852.png
[image-148]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%85%88%E6%9B%B4%E6%96%B0%E7%BC%93%E5%AD%98%E5%90%8E%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE%E5%BA%93.png
[image-149]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E9%AB%98%E5%B9%B6%E5%8F%91%E5%85%88%E5%85%88%E5%86%99%E6%95%B0%E6%8D%AE%E5%BA%93%E5%86%8D%E6%9B%B4%E6%96%B0%E7%BC%93%E5%AD%98.drawio.png
[image-150]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%85%88%E5%88%A0%E7%BC%93%E5%AD%98%E5%86%8D%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE%E5%BA%93.drawio.png
[image-151]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%85%88%E5%86%99%E6%95%B0%E6%8D%AE%E5%BA%93%E5%90%8E%E5%88%A0%E7%BC%93%E5%AD%98.drawio.png
[image-152]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%85%88%E5%86%99%E6%95%B0%E6%8D%AE%E5%BA%93%E5%90%8E%E5%88%A0%E7%BC%93%E5%AD%98%EF%BC%8C%E7%BC%93%E5%AD%98%E5%BF%BD%E7%84%B6%E5%A4%B1%E6%95%88.drawio.png
[image-153]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E9%87%8D%E8%AF%95%E6%9C%BA%E5%88%B6.drawio.png
[image-154]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/binlog%E5%88%A0%E9%99%A4%E7%BC%93%E5%AD%98.drawio.png
[image-155]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%AE%80%E5%8D%95%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.png
[image-156]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis%E9%87%8D%E5%85%A5%E9%94%81.png
[image-157]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E5%AE%88%E6%8A%A4%E7%BA%BF%E7%A8%8B%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.png
[image-158]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redsson.png
[image-159]:	https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/%E7%BA%A2%E9%94%81%E6%90%9E%E7%AC%91.png