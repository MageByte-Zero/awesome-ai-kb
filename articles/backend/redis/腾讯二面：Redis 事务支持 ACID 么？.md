> 腾讯面试官：「数据库事务机制了解么？」

「内心独白：小意思，不就 ACID 嘛，转眼一想，我面试的可是技术专家，不会这么简单的问题吧」

程许远：「balabala…… 极其自信且从容淡定的说了一通。」

> 腾讯面试官：「[Redis](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzMDI1NjcyOQ==&action=getalbum&album_id=1918295695426404359&scene=173&from_msgid=2247493190&from_itemidx=1&count=3&nolastread=1#wechat_redirect) 的事务了解么？它的事务机制能实现 ACID 属性么？」

程许远：「挠头，这个……我知道 lua 脚本能实现事务…」

腾讯面试官：「好的，回去等通知吧。」

---

> 码哥，我跟着你学习了 《[Redis 系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzMDI1NjcyOQ==&action=getalbum&album_id=1918295695426404359&scene=173&from_msgid=2247493190&from_itemidx=1&count=3&nolastread=1#wechat_redirect)》斩获了很多 offer，没想到最后败在了 「[Redis](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzMDI1NjcyOQ==&action=getalbum&album_id=1918295695426404359&scene=173&from_msgid=2247493190&from_itemidx=1&count=3&nolastread=1#wechat_redirect) 如何实现事务？」这个问题上。

我们来一步步分析：

1. 什么是事务 ACID？
2. Redis 如何实现事务？
3. Redis 的事务能实现哪些属性？
4. Lua 脚本实现。

# 什么是事务的 ACID

鬼吹灯之《云南虫谷》中的摸金校尉有句话叫「合则生，分则死」，为了寻找雮尘珠他们三人分工明确、**齐心协力共进退**方可成功。

事务（Transaction）是并发控制单位，一个操作序列组合而成，这些操作要么都执行，要么都不执行。

「是一个不可分割的工作单位」。

事务在执行时，会提供专门的属性保证：

- 原子性（Atomicity）：一个事务的多个操作必须完成，或者都不完成（ps：MySQL 的原子性靠什么实现呢？欢迎留言区评论）；

- 一致性（Consistency）：事务执行结束后，数据库的完整性约束没有被破坏，事务执行的前后顺序都是合法数据状态。

  数据库的完整性约束包括但不限于：

  - 实体完整性（如行的主键存在且唯一）；
  - 列完整性（如字段的类型、大小、长度要符合要求）
  - 外键约束；
  - 用户自定义完整性（如转账前后，两个账户余额的和应该不变）。

- 隔离性（Isolation）：事务内部的操作与其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

  讲究的是不同事务之间的相互影响，严格的隔离性对应隔离级别中的可串行化（Serializable）。

- 持久性（Durability）：事务一旦提交，所有的修改将永久的保存到数据库中，即使系统崩溃重启后数据也不会丢失。

> 码哥，了解了 ACID 的具体要求后，Redis 是如何实现事务机制呢？

# Redis 如何实现事务

[MULTI](https://redis.io/commands/multi)、[EXEC](https://redis.io/commands/exec)、[DISCARD ](https://redis.io/commands/discard)和 [WATCH](https://redis.io/commands/watch) 命令是 Redis 实现事务的的基础。

Redis 事务的执行过程包含三个步骤：

1. 开启事务；
2. 命令入队；
3. 执行事务或丢弃；

## 显式开启一个事务

客户端通过 `MULTI` 命令显式地表示开启一个事务，随后的命令将排队缓存，并不会实际执行。

## 命令入队

客户端把事务中的要执行的一系列指令发送到服务端。

需要注意的是，虽然指令发送到服务端，但是 Redis 实例只是把这一系列指令暂存在一个命令队列中，并不会立刻执行。

## 执行事务或丢弃

客户端向服务端发送提交或者丢弃事务的命令，让 Redis 执行第二步中发送的具体指令或者清空队列命令，放弃执行。

Redis 只需在调用 [EXEC ](https://redis.io/commands/exec)时，即可安排队列命令执行。

也可通过 [DISCARD](https://redis.io/commands/discard) 丢弃第二步中保存在队列中的命令。

## Redis 事务案例

通过在线调试网站执行我们的样例代码：https://try.redis.io

### 正常执行

通过 `MULTI` 和 `EXEC` 执行一个事务过程：

```
# 开启事务
> MULTI
OK
# 开始定义一些列指令
> SET “公众号:码哥字节” "粉丝 100 万"
QUEUED
> SET "order" "30"
QUEUED
> SET "文章数" 666
QUEUED
> GET "文章数"
QUEUED
# 实际执行事务
> EXEC
1) OK
2) OK
3) OK
4) "666"
```

我们看到每个读写指令执行后的返回结果都是 `QUEUED`，表示谢谢操作都被暂存到了命令队列，还没有实际执行。

当执行了 `EXEC` 命令，就可以看到具体每个指令的响应数据。

### 放弃事务

通过 `MULTI` 和 `DISCARD`丢弃队列命令：

```
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

> 码哥，Redis 的事务能保证 ACID 特性么？

这个问题问得好，我们一起来分析下。

# Redis 事务满足 ACID？

**Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：**

1. 批量指令在执行 EXEC 命令之前会放入队列暂存；
2. 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行；
3. 事务执行过程中，其他客户端提交的命令不会插入到当前命令执行的序列中。

## 原子性

> 码哥，如果事务执行过程中发生错误了，原子性能保证么？

在事务期间，可能遇到两种命令错误：

- 在执行 `EXEC` 命令前，发送的指令本身就错误。如下：
  - 参数数量错误；
  - 命令名称错误，使用了不存在的命令；
  - 内存不足（Redis 实例使用 `maxmemory`指令配置内存限制）。
- 在执行 `EXEC` 命令后，命令可能会失败。例如，命令和操作的数据类型不匹配（对 String 类型 的 value 执行了 List 列表操作）；
- 在执行事务的 `EXEC` 命令时。 Redis 实例发生了故障导致事务执行失败。

### EXEC 执行前报错

在命令入队时，Redis 就会**报错并且记录下这个错误**。

此时，我们**还能继续提交命令操作**。

等到执行了 `EXEC`命令之后，Redis 就会**拒绝执行所有提交的命令操作，返回事务失败的结果**。

这样一来，**事务中的所有命令都不会再被执行了，保证了原子性。**

如下是指令入队发生错误，导致事务失败的例子：

```
#开启事务
> MULTI
OK
#发送事务中的第一个操作，但是Redis不支持该命令，返回报错信息
127.0.0.1:6379> PUT order 6
(error) ERR unknown command `PUT`, with args beginning with: `order`, `6`,
#发送事务中的第二个操作，这个操作是正确的命令，Redis把该命令入队
> DECR b:stock
QUEUED
#实际执行事务，但是之前命令有错误，所以Redis拒绝执行
> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
```

### EXEC 执行后报错

事务操作入队时，命令和操作的数据类型不匹配，但 Redis 实例没有检查出错误。

但是，在执行完 EXEC 命令以后，Redis 实际执行这些指令，就会报错。

**敲黑板了：Redis 虽然会对错误指令报错，但是事务依然会把正确的命令执行完，这时候事务的原子性就无法保证了！**

> 码哥，为什么 Redis 不支持回滚？

其实，Redis 中并没有提供回滚机制。虽然 Redis 提供了 DISCARD 命令。

但是，这个命令只能用来主动放弃事务执行，把暂存的命令队列清空，起不到回滚的效果。

### EXEC 执行时，发生故障

如果 Redis 开启了 AOF 日志，那么，只会有部分的事务操作被记录到 AOF 日志中。

我们需要使用 redis-check-aof 工具检查 AOF 日志文件，这个工具可以把未完成的事务操作从 AOF 文件中去除。

这样一来，我们使用 AOF 恢复实例后，事务操作不会再被执行，从而保证了原子性。

**简单总结**：

- 命令入队时就报错，会放弃事务执行，保证原子性；
- 命令入队时没报错，实际执行时报错，不保证原子性；
- EXEC 命令执行时实例故障，如果开启了 AOF 日志，可以保证原子性。

## 一致性

一致性会受到错误命令、实例故障发生时机的影响，按照命令出错实例故障两个维度的发生时机，可以分三种情况分析。

### EXEC 执行前，入队报错

事务会被放弃执行，所以可以保证一致性。

### EXEC 执行后，实际执行时报错

有错误的执行不会执行，正确的指令可以正常执行，一致性可以保证。

### EXEC 执行时，实例故障

实例故障后会进行重启，这就和数据恢复的方式有关了，我们要根据实例是否开启了 RDB 或 AOF 来分情况讨论下。

如果我们没有开启 RDB 或 AOF，那么，实例故障重启后，数据都没有了，数据库是一致的。

如果我们使用了 RDB 快照，**因为 RDB 快照不会在事务执行时执行。**

所以，**事务命令操作的结果不会被保存到 RDB 快照中**，使用 RDB 快照进行恢复时，数据库里的数据也是一致的。

如果我们使用了 AOF 日志，而事务操作还没有被记录到 AOF 日志时，实例就发生了故障，那么，使用 AOF 日志恢复的数据库数据是一致的。

如果只有部分操作被记录到了 AOF 日志，我们可以使用 redis-check-aof 清除事务中已经完成的操作，数据库恢复后也是一致的。

## 隔离性

事务执行又可以分成命令入队（EXEC 命令执行前）和命令实际执行（EXEC 命令执行后）两个阶段。

所以在并发执行的时候我们针对这两个阶段分两种情况分析：

1. 并发操作在 `EXEC` 命令前执行，隔离性需要通过 `WATCH` 机制保证；
2. 并发操作在 `EXEC` 命令之后，隔离性可以保证。

> 码哥，什么是 WATCH 机制？

我们重点来看第一种情况：一个事务的 EXEC 命令还没有执行时，事务的命令操作是暂存在命令队列中的。

此时，如果有其它的并发操作，同样的 key 被修改，需要看事务是否使用了 `WATCH` 机制。

WATCH 机制的作用是：在事务执行前，监控一个或多个键的值变化情况，当事务调用 EXEC 命令执行时，WATCH 机制会先检查监控的键是否被其它客户端修改了。

**如果修改了，就放弃事务执行，避免事务的隔离性被破坏。**

同时，客户端可以再次执行事务，此时，如果没有并发修改事务数据的操作了，事务就能正常执行，隔离性也得到了保证。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-transaction.png)

**没有 WATCH**

如果没有 WATCH 机制， 在 EXEC 命令执行前的并发操作对数据读写。

当执行 EXEC 的时候，事务内部要操作的数据已经改变，Redis 并没有做到事务之间的隔离。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-watch.png)

**并发操作在 EXEC 之后接收执行**

至于第二种情况，因为 Redis 是用单线程执行命令，而且，EXEC 命令执行后，Redis 会保证先把命令队列中的所有命令执行完再执行之后的指令。

所以，在这种情况下，并发操作不会破坏事务的隔离性。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/redis-after.png)

## 持久性

如果 Redis 没有使用 RDB 或 AOF，那么事务的持久化属性肯定得不到保证。

如果 Redis 使用了 RDB 模式，那么，在一个事务执行后，而下一次的 RDB 快照还未执行前，如果发生了实例宕机，数据丢失，这种情况下，事务修改的数据也是不能保证持久化的。

如果 Redis 采用了 AOF 模式，因为 AOF 模式的三种配置选项 no、everysec 和 always 都会存在数据丢失的情况。

所以，事务的持久性属性也还是得不到保证。

**不管 Redis 采用什么持久化模式，事务的持久性属性是得不到保证的。**

## 总结

- Redis 具备了一定的原子性，但不支持回滚。
- Redis 不具备 ACID 中一致性的概念。(或者说 Redis 在设计时就无视这点)
- Redis 具备隔离性。
- Redis 无法保证持久性。

**Redis 的事务机制可以保证一致性和隔离性，但是无法保证持久性。**

不过，因为 Redis 本身是内存数据库，持久性并不是一个必须的属性，我们更加关注的还是原子性、一致性和隔离性这三个属性。

原子性的情况比较复杂，**当事务中使用的命令语法有误时，原子性得不到保证**，在其它情况下，事务都可以原子性执行。

## 好文推荐

[Redis 核心篇：为什么这么快](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487752&idx=1&sn=72a1725e1c86bb5e883dd8444e5bd6c4&chksm=c27c533ef50bda288417c31f5210bb16a70361b2e2c344dfffdb079b54b241cd6c62202d9775&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 持久化篇：AOF 与 RDB 如何保证数据高可用](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487758&idx=1&sn=beb5918bb61948b2920907f54510311f&chksm=c27c5338f50bda2ec2accd75156eb878e3150cf64d79e5d67ac38569e9dbba0f6814801aa3ab&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 高可用篇：主从架构数据一致性同步原理](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487769&idx=1&sn=3c975ea118d4e59f72df5beed58f4768&chksm=c27c532ff50bda39055fc4e6dabf5bb0b6cc2945a4cad87782c8e46fdb32bb67beaa38438c65&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 高可用篇：哨兵集群原理](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487780&idx=1&sn=9a0ea0971e661556c4c5e438ab1b081b&chksm=c27c5312f50bda04231254e78736d151f789ef056f43d36f7cd861c70f0cb54b7e26ea03d5d4&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 高可用篇：Cluster 集群原理](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487789&idx=1&sn=7f8245f8b4e4a98aa0a717011f7b7e24&chksm=c27c531bf50bda0da3bcc325b131dac2553eb508fed4175ab1d883fb4946557fb91053ddb525&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 实战篇：巧用 Bitmap 实现亿级海量数据统计](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487813&idx=1&sn=9b346ad34a3b8cf38a3f338e85804800&chksm=c27c5373f50bda65800af3ba92089815323016979aeafa47294072712e1d93131825cb924026&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 实战篇：通过 Geo 类型实现附近的人邂逅女神](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247489024&idx=1&sn=8ae92ff12bc8c1af1617dbfaf91cb1dd&chksm=c27c5436f50bdd20035974cc91a4b5c203f5c3224a83b80c55a3e795476ad2d872454120e6b2&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 新特性篇：多线程模型解读](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247489917&idx=1&sn=6865c14ee10684c2766eba9bc0edd13a&chksm=c27c5b4bf50bd25d6467e86df51fa7584009be05f2eca40eaa3aafdceb079e7b3855498774de&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

[Redis 6.0 新特性篇：客户端缓存带来的革命](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247491153&idx=1&sn=9e7e7abc8aa6670468c413692961e37c&chksm=c27c5c67f50bd5710c697420d1b9903a780aafbbc0c71b66dea6c8cc14d932ae72f3576920c6&scene=21&cur_album_id=1918295695426404359#wechat_redirect)

