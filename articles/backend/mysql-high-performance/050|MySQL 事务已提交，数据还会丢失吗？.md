你好，我是《Redis 高手心法》畅销书作者，可以叫我靓仔。

今天这篇内容与上一篇 MVCC 内容很紧密，你可以翻看上一篇再学习本章内容效果更好，不至于呛死在知识的海洋里。

在 MySQL 数据库的使用过程中，事务是一个绕不开的核心概念。我们常说“事务 ACID 特性”，其中“持久性（Durability）”明确表示：事务一旦提交，其对数据库中数据的修改就应该是永久性的，接下来的操作或故障都不应该对其执行结果有任何影响。

今天这篇文章，我们就从 MySQL 的底层原理出发，一步步拆解“事务提交”的完整流程，找出数据丢失的潜在风险点，同时给出可落地的解决方案。

全文约 4000 字，包含多个核心机制图解，建议先收藏再阅读。

## 一个误区

要搞懂“提交后数据是否会丢”，首先得打破一个认知误区：**事务提交的返回结果，只是“数据库内核确认可以完成持久化”，而非“数据已经真正写入磁盘”**。

很多人以为的事务提交流程是“执行 SQL→ 修改数据 → 写入磁盘 → 返回成功”，但这与 MySQL 的实际实现相去甚远。

为了平衡性能和可靠性，MySQL 引入了“内存缓冲”“日志机制”等多层设计，这就导致“提交成功”和“数据持久化”之间存在时间差，而这个时间差正是数据丢失风险的根源。

在深入分析前，我们先回顾两个基础概念，这是理解后续内容的关键：

- **事务的持久性**：ACID 中的 D，理论上要求事务提交后数据永久不丢，但“永久不丢”是理想状态，实际中需通过技术手段无限趋近这一目标。
- **MySQL 的存储引擎**：只有 InnoDB 支持事务，MyISAM 不支持。本文所有分析均基于 InnoDB 引擎。

## InnoDB 事务提交的核心流程

InnoDB 之所以能在高性能下保障事务安全，核心依赖于“缓冲池**（Buffer Pool）**”和“**重做日志（Redo Log）**”两大机制。

事务提交的全过程，本质就是这两大机制协同工作的过程。我们先通过一张图理清整体流程：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20251214172645130.png)

这里我用一个 UPDATE 语句执行的例子来向你介绍事务执行过程。假如说原本 a = 3，现在要执行 `UPDATE tab SET a = 5 WHERE id = 1`。

该流程清晰地划分了五个核心阶段。

1. **SQL 解析与事务初始化**：MySQL Server 层负责接收 SQL，进行词法分析、语法优化，生成最优的执行计划。
2. **InnoDB 事务执行准备**：InnoDB 引擎接管后，会为当前事务分配唯一 ID，并根据 WHERE 条件为需要修改的行加上**排他锁（X Lock）**， 防止其他事务同时修改相同行。数据加载到 buffer pool 里面。
3. **数据修改与日志记录**（核心）：这是保证事务 ACID 特性的关键环节。
   - **写 Undo Log**：在修改数据前，先将原始数据备份到 Undo Log 中。这确保了事务可以回滚（原子性），同时也是实现 **MVCC（多版本并发控制）** 的基础，使其他事务的读操作不受本事务写操作的影响。
   - **修改内存数据页**：在 Buffer Pool 中直接修改数据，此时数据页变为"脏页"。
   - **写 Redo Log Buffer**：将数据页的**物理变化**记录到重做日志缓冲区。这是 **WAL（Write-Ahead Logging）** 规则的体现，即先写日志，后刷数据。
4. **事务提交（两阶段提交）**：为了确保 Binlog（用于主从复制）和 Redo Log 的一致性，MySQL 使用两阶段提交机制。
   - **Prepare 阶段**：InnoDB 引擎根据 `innodb_flush_log_at_trx_commit` 决定是否 Redo Log 刷盘，并标记状态为 `prepare`。
   - **Commit 阶段**：MySQL Server 写入 Binlog 后，再将 Redo Log 标记为 `commit`。至此，事务才被视为真正提交，锁被释放。
5. **后台异步操作**：事务提交后，被修改的"脏页"并不会立即刷回磁盘，而是由后台线程异步完成，这极大地提升了性能。

**这张图里藏着三个关键问题，也是数据丢失风险的核心：**

1. 为什么不直接把数据写入磁盘，而要先写缓冲池？
2. 重做日志（Redo Log）到底是什么，为什么它的刷盘比数据刷盘更重要？
3. 事务提交时，重做日志是“必须刷盘”还是“可以延迟刷盘”？

我们逐一拆解这三个问题，就能彻底搞懂提交后数据丢失的根源。

## 前置知识

update 事务执行过程，我们看到了几个关键术语：bing log、undo log、redo log、buffer pool。

一起看下这些都是什么玩意。

### undo log

> 余弦：undo log 到底是什么呢？

undo log 是指回滚日志，用一个比喻来说，就是后悔药，它记录着事务执行过程中**被修改前**的数据。

当事务回滚的时候，InnoDB 会根据 undo log 里的数据撤销事务的更改，把数据库恢复到原来的状态。

在爱情的故事里，当你做错了事，可以借助 undo log 触发回滚技能。但是，爱情中的事可能很难执行 undo log，切记切记！

- 对于 INSERT 来说，对应的 undo log 应该是 DELETE。
- 对于 DELETE 来说，对应的 undo log 应该是 INSERT。
- 对于 UPDATE 来说，对应的 undo log 也应该是 UPDATE。比如说有一个数据的值原本是 3，要把它更新成 5。那么对应的 undo log 就是把数据更新回 3。

实际上，对于 **INSERT** 来说，对应的 undo log 记录了该行的主键。

那么后续回滚只需要根据 undo log 里面的主键去原本的聚簇索引里面删掉记录。

对于 **DELETE** 来说，对应的 undo log 记录了该行的主键。因为在事务执行 DELETE 的时候，实际上并没有真的把记录删除，只是把原记录的**删除标记位设置成了 true**。

对于 **UPDATE** 来说，要更加复杂一些。分为两种情况：

- 如果没有更新主键，那么 undo log 里面就记录**原记录的主键和被修改的列的原值**。
- 如果更新了主键，那么可以看作是删除了原本的行，然后插入了一个新行。**因此 undo log 可以看作是一个 DELETE 原数据的 undo log 再加上插入一个新行的 undo log。**

Undo Log 的生命周期与 MVCC 的构建。

```sql
-- 示例：多版本链的形成
-- 事务1 (trx_id=100) 插入记录
BEGIN;
INSERT INTO t1 (id, name, value) VALUES (1, 'A', 100);
COMMIT;

-- 事务2 (trx_id=200) 更新记录
BEGIN;
UPDATE t1 SET value = 200 WHERE id = 1;  -- 生成 undo log1
-- 此时版本链：当前记录(trx_id=200) -> undo log1(trx_id=100)

-- 事务3 (trx_id=300) 再次更新
BEGIN;
UPDATE t1 SET value = 300 WHERE id = 1;  -- 生成 undo log2
-- 此时版本链：当前记录(trx_id=300) -> undo log2(trx_id=200) -> undo log1(trx_id=100)
```

### redo log

InnoDB 引擎在数据库发生更改的时候，把更改操作记录在 redo log 里，以便在数据库发生崩溃或出现其他问题的时候，能够通过 redo log 来重做。

> InnoDB 引擎不是直接修改了数据吗？为什么需要 redo log？

InnoDB 引擎读写都不是直接操作磁盘的，而是读写内存里的 buffer pool，后面再把 buffer pool 里面修改过的数据刷新到磁盘里面。

这是两个步骤，所以就可能会出现 buffer pool 中的数据修改了，但是还没来得及刷新到磁盘数据库就崩溃了的情况。

为了解决这个问题，InnoDB 引擎就引入了 redo log。

**相当于 InnoDB 先把 buffer pool 里面的数据更新了，再写一份 redo log。**

等到事务结束之后，就把 buffer pool 的数据刷新到磁盘里面。

万一事务提交了，但是 buffer pool 的数据没写回去，就可以用 redo log 来恢复。

Redo Log 的核心作用是“故障恢复”：**如果服务器断电，缓冲池中的脏页丢失，重启后 MySQL 会读取 Redo Log，将所有已提交但未刷盘的操作重新执行一遍，从而恢复数据。**

这就是“持久性”的底层保障——只要 Redo Log 已经刷盘，即使数据没刷盘，数据也能恢复。

到这里，我们可以得出一个关键结论：**事务提交后数据是否会丢，本质上取决于 Redo Log 是否已经刷盘**。

如果 Redo Log 没刷盘，即使提示提交成功，断电后数据也会丢失；如果 Redo Log 已经刷盘，即使数据没刷盘，重启后也能通过 Redo Log 恢复。

重做日志是 InnoDB 的核心日志，它记录的是“数据修改的动作”，而非修改后的数据。

比如执行`UPDATE user SET name='zhangsan' WHERE id=1`，Redo Log 不会记录“name 变成了 zhangsan”，而是记录“修改了 user 表中 id=1 的记录的 name 字段，旧值是 lisi，新值是 zhangsan”。

为什么要记录“动作”而不是“结果”？因为“动作”更精简，写入速度更快。

> redo log 不需要写磁盘吗？如果 redo log 也要写磁盘，干嘛不直接修改数据呢？

Redo Log 的刷盘机制比数据刷盘更高效，原因有两个：

- **顺序写**：Redo Log 文件是固定大小的循环写入，始终是顺序追加，而数据刷盘是随机写（要修改磁盘上的任意数据页）。**顺序写的速度远高于随机写，这是 Redo Log 性能的核心优势。**
- **小批量刷盘**：Redo Log Buffer 中的日志可以批量刷盘，而数据刷盘是按数据页（通常 16KB）刷盘，单次刷盘的数据量更大。

redo log 本身也是先写进 redo log buffer，后面再刷新到操作系统的 page cache，或者一步到位刷新到磁盘。

InnoDB 引擎本身提供了参数 `innodb_flush_log_at_trx_commit` 来控制写到磁盘的时机，里面有三个不同值。

- 0：每秒刷新到磁盘，是从 redo log buffer 到磁盘。
- 1：每次提交的时候刷新到磁盘上，也就是最安全的选项，InnoDB 的默认值。
- 2：每次提交的时候刷新到 page cache 里，依赖于操作系统后续刷新到磁盘。

这时候你就应该意识到这样一个问题，除非把 innodb_flush_log_at_trx_commit 设置成 1，否则其他两个都有丢失的风险。

- 0：你提交之后，InnoDB 还没把 redo log buffer 中的数据刷新到磁盘，就宕机了。
- 2：你提交之后，InnoDB 把 redo log 刷新到了 page cache 里面，紧接着宕机了.

在这两个场景下，你的业务都认为事务提交成功了，但是数据库实际上丢失了这个事务。

流程如下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20251214200512967.png)

举个实际案例：某电商平台的订单系统，MySQL 的`innodb_flush_log_at_trx_commit`配置为 2。

某次服务器突然断电，重启后发现有 10 分钟内的订单数据丢失，造成了不小的损失。

事后排查发现，就是因为这 10 分钟内的订单事务虽然提交成功，但 Redo Log 还在 OS Cache 中，没刷到磁盘，断电后 OS Cache 中的数据丢失，无法恢复。

### binlog

Binlog（Binary Log）是 MySQL 的**二进制日志**，它记录了所有对数据库的**数据变更操作**。它是 MySQL Server 级别的日志，也就是说所有引擎都有。

想象一下，它是 MySQL 的"行车记录仪"，完整记录了数据库的每一个变化瞬间。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20251214201335488.png)

在事务执行过程中，写入 binlog 的时机有点巧妙。它和 redo log 的提交过程结合在一起称为 MySQL 的两阶段提交。

- **Prepare（准备）阶段**：存储引擎（如 InnoDB）将事务的`redo log`写入磁盘，并将事务状态标记为`TRX_STATE_PREPARED`（或`PREPARE`）。
- **Commit（提交）阶段**：此阶段又可细分为关键步骤：
  - **写入 Binlog**：首先将事务的`binlog`数据写入磁盘文件。此时数据可能还在操作系统的页面缓存（Page Cache）中。
  - **刷盘 Binlog**：根据`sync_binlog`参数设置，决定何时将`binlog`从缓存强制写入磁盘，这是保证持久性的关键一步。
  - **标记 Commit**：最后，存储引擎将`redo log`中该事务的状态标记为`COMMIT`。值得注意的是，只要`binlog`已安全落盘，即使此步骤因崩溃未完成，事务仍被视为已提交。

事务执行过程，binlog 与 redo log 二阶段提交和崩溃恢复机制流程如下图所示：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20251214201920401.png)

如果 redo log Prepare 执行完毕后，**binlog 已经写成功了**，那么即便 redo log 提交失败，MySQL 也会认为事务已经提交了。

如果 binlog 没写成功，那么 MySQL 就认为提交失败了。

比较简单的记忆方式就是看 **binlog 写成功了没有**。

**binlog 本身有一些完整性校验的规则，所以在 MySQL 看来，写 binlog 要么成功，要么失败，不存在中间状态。**

binlog 也有刷新磁盘的问题，不过你可以通过 sync_binlog 参数来控制它。

- 0：由操作系统决定，写入 page cache 就认为成功了。0 也是默认值，这个时候数据库的性能最好。
- N：每 N 次提交就刷新到磁盘，N 越小性能越差。如果 N = 1，那么就是每次事务提交都把 binlog 刷新到磁盘。

## 总结

为了在保证数据安全和高性能之间取得平衡，有几个关键参数可以调整；

- `sync_binlog`：控制 binlog 刷盘策略。设为 1 最安全（每次提交都刷盘），但性能开销最大；设为大于 1 的值可提升性能，但宕机可能丢失最近 N 个事务的 binlog。
- `innodb_flush_log_at_trx_commit`：控制 redo log 刷盘策略。设为 1 最安全（每次提交都刷盘）；设为 0 或 2 可提升性能，但存在数据丢失风险。

对数据不丢失、一致性要求高的业务，你可以考虑调整 sync_binlog 的值为 1。

另外一个是性能优先，能够容忍一定数据丢失的。

那么你可以考虑将 innodb_flush_log_at_trx_commit 调整为 0 或者 2，同时还可以把 sync_binlog 调整为比较大的值，比如说调到 100。