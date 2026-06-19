---
title: "Valkey vs Redis，分叉两年后：大多数团队该换了"
description: "Valkey 分叉 Redis 两年后的真实对比：25.4k stars、10 亿 RPS 集群、230% 吞吐提升。从性能、兼容性、生态到许可证，用数据说清楚该不该迁移"
date: "2026-04-08"
keywords: ["Valkey", "Redis", "缓存", "分布式数据库", "开源", "性能对比"]
platform: "掘金"
source: "Valkey 官方博客 + GitHub"



2024 年 3 月，Redis Labs 把 Redis 的许可证从 BSD 改成了双许可（RSALv2 + SSPLv1）。翻译成人话：云厂商不能再免费拿 Redis 做托管服务卖钱了。

一周之内，Linux 基金会宣布成立 Valkey 项目，AWS、Google Cloud、Oracle、Ericsson 集体站台。Redis 的核心维护者 Madelyn Olson（前 AWS 员工，Redis 项目贡献排名第二）直接跳船加入 Valkey。

现在是 2026 年 4 月，Valkey 分叉两年了。GitHub 25.4k stars，427 个 open issues，204 个 PR 在活跃开发。它不再是一个「Redis 的替代品」——它在用自己的方式重新定义这个品类。

这篇文章用数据说话：性能差多少、兼容性怎么样、迁移有多痛、什么时候该换什么时候别动。

## 先给结论

| 你的场景                       | 推荐           | 理由                                               |
| ------------------------------ | -------------- | -------------------------------------------------- |
| 新项目选型                     | **Valkey**     | BSD 许可证、性能更优、社区更活跃                   |
| 现有 Redis OSS 项目            | **Valkey**     | 兼容性接近 100%，迁移几乎无痛                      |
| 使用 Redis Enterprise 付费功能 | **留 Redis**   | RedisJSON、RedisGraph 等模块暂无等价替代           |
| 云厂商托管服务用户             | **看厂商**     | AWS ElastiCache 已默认 Valkey，腾讯/阿里还是 Redis |
| 需要最新 Redis 7.4+ 特性       | **看具体特性** | Valkey 有自己的特性路线，部分重叠部分分化          |

懒人版：**如果你没有在用 Redis Enterprise 的付费模块，新项目直接选 Valkey，旧项目也值得迁移。**

## 它们各自在解决什么问题

Redis 和 Valkey 现在走的是两条不同的路。

**Redis** 的方向是商业化。Redis 8.0（2025 年发布）把之前收费的 Redis Stack 模块（Search、JSON、TimeSeries、Probabilistic）整合进了核心。听起来很慷慨——但许可证是 SSPL，意味着你不能用它做竞品服务。Redis 的目标很明确：成为一个全功能的商业数据平台，免费给你用但不能和它竞争。

**Valkey** 的方向是纯开源性能怪兽。BSD 许可证，任何人任何用途都可以免费使用。它的精力全砸在两件事上：**极致性能**和**集群可靠性**。不搞花哨的模块堆砌，就是把键值存储这件事做到极限。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/f47c2d7d44160e5fc8954112a458b2e4.png)

## 性能：Valkey 拉开了差距

这是最硬的对比维度。Valkey 在性能上的投入确实凶猛。

### 单节点：230% 吞吐提升

Valkey 8.0 重写了多线程 I/O 架构。核心改动是把 `epoll_wait` 等昂贵的套接字轮询操作从主线程卸载到 I/O 工作线程，同时保持主线程单线程执行命令（不引入锁）。

基准测试数据（AWS C7g.16xlarge，8 I/O 线程，300 万键，512 字节值，650 客户端）：

| 指标     | Valkey 7.2 | Valkey 8.0 | 提升       |
| -------- | ---------- | ---------- | ---------- |
| 吞吐量   | 360K RPS   | 1,190K RPS | **+230%**  |
| 平均延迟 | 1.792ms    | 0.542ms    | **-69.8%** |

这不是小修小补，这是架构级别的性能飞跃。而且因为主线程仍然是单线程执行命令，所有现有的 Redis 客户端库都能无感迁移——不会有多线程带来的兼容性问题。

### 集群：10 亿 RPS

Valkey 9.0 把集群性能推到了一个新高度。2000 个节点（1000 主 + 1000 副本），AWS r7g.2xlarge，跑到了 **10 亿 RPS**（SET 命令，512 字节数据）。

![valkey-io-thread-architecture](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/valkey-io-thread-architecture.png)

关键优化包括四项：

1. **多主节点故障处理**：引入排名机制防止投票冲突，同时 50% 主节点故障时仍能自动恢复
2. **连接节流**：防止数百节点同时故障时的重连风暴
3. **优化故障报告**：用 radix tree 按秒分组，减少冗余 gossip 处理
4. **轻量化 Pub/Sub 头**：从 ~2KB 降到 ~30 字节，去掉了无关的 slot 所有权数据

吞吐量和节点数呈近线性扩展，集群总线开销在默认超时配置下可以忽略。

### Redis 的性能回应

坦白讲，Redis 在性能方面也没闲着。Redis 8.0 声称有 2 倍的吞吐量改进，但具体的基准测试条件和 Valkey 不完全对齐，很难做精确的苹果对苹果对比。

我的判断是：**在纯键值操作场景下，Valkey 的性能优势是确定的**。但如果你用的是 Redis Stack 的高级功能（全文搜索、JSON 操作），两者没有直接可比性——Valkey 刚开始做搜索功能（Valkey Search），成熟度还不够。

## 兼容性：接近 100%，但有分化

Valkey 从 Redis 7.2.4 分叉，API 协议层面完全兼容。所有 Redis 客户端库（Jedis、Lettuce、ioredis、redis-py）都能直接连 Valkey，不需要改一行代码。

**已确认兼容的：**

- 所有 Redis 命令（截至 7.2.4 版本）
- RDB/AOF 持久化格式
- 集群协议和 Sentinel
- Lua 脚本
- Pub/Sub
- Spring Data（有了专门的 Spring Data Valkey 模块）

**开始分化的：**

- Valkey 新增了 Hash 字段级 TTL（Redis 没有）
- Valkey 新增了 `COMMANDLOG` 和 `SLOT-STATS`（集群运维工具）
- Redis 8.0 新增的 JSON、Search、TimeSeries 模块（Valkey 不兼容，走自己的实现路线）
- 两者的配置参数开始出现差异（比如 I/O 线程相关的配置）

![compatibility-matrix](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/compatibility-matrix.png)

## 生态与社区

| 维度         | Redis              | Valkey                                       |
| ------------ | ------------------ | -------------------------------------------- |
| 许可证       | RSALv2 + SSPLv1    | BSD 3-Clause                                 |
| GitHub Stars | ~67k（含历史积累） | 25.4k（两年）                                |
| 治理         | Redis Ltd 控制     | Linux 基金会，开放治理                       |
| 核心维护者   | Redis 公司员工     | AWS、Google、Oracle、Ericsson 工程师         |
| 云厂商支持   | Redis Cloud        | AWS ElastiCache、Google Memorystore          |
| 客户端库     | 所有主流语言       | 复用 Redis 客户端 + 新增 Swift/Go 专属客户端 |
| Spring 集成  | Spring Data Redis  | Spring Data Valkey（2026.4 发布）            |

社区活跃度数据来看，Valkey 两年攒到 25.4k stars，有 1.1k forks。考虑到 Redis 历史上积累了十几年的 stars，Valkey 的增速是相当惊人的。

关键信号是 Spring Data Valkey 的发布——Spring 生态是 Java 后端的基石。有了官方的 Spring Data 模块，意味着 Java 社区不再需要把 Valkey 当作「兼容 Redis 的替代品」来使用，它有了自己的一等公民身份。

## 迁移成本：比你想象的低

如果你现在用的是 Redis OSS（不是 Redis Enterprise），迁移到 Valkey 基本上是：

1. **换二进制文件**：下载 Valkey 替换 Redis Server，配置文件格式兼容
2. **客户端不用改**：所有 Redis 客户端库直连 Valkey，连接字符串只需要改 host
3. **数据不用迁移**：RDB/AOF 格式兼容，直接加载

真正需要注意的只有两点：

- 如果你用了 Redis 7.4+ 的新功能，确认 Valkey 是否支持
- 如果你用了 Redis Stack 模块（JSON、Search 等），Valkey 还没有完全对标的替代

## 常见问题

**Q：我现在用 Spring Boot + Lettuce 连 Redis，迁移到 Valkey 要改代码吗？**

不用。Lettuce 客户端直接兼容 Valkey，把连接地址改成 Valkey 实例就行。如果你想用新发布的 Spring Data Valkey 模块，需要替换依赖（从 `spring-boot-starter-data-redis` 换成 `spring-boot-starter-data-valkey`），但这不是必须的——旧依赖也能连 Valkey。

**Q：Valkey 支持 Redis Sentinel 吗？**

支持，完全兼容。Sentinel 协议没有变化。

**Q：性能数据看着很猛，但我的实际场景能有这么大提升吗？**

230% 的提升是在高并发、大客户端数的极限场景下测出来的。如果你的 Redis 实例日常只有几百 QPS，迁移后性能感知可能不明显。Valkey 的性能优势主要体现在高并发场景——客户端数 >100、QPS >10K 时差距会比较显著。

**Q：Valkey 有 Redis Cluster 的所有功能吗？**

有，而且更好。Valkey 9.0 在集群稳定性上做了大量改进，特别是多节点故障恢复和原子化 slot 迁移。如果你现在用的是 Redis Cluster 并且时不时遇到故障切换问题，Valkey 值得一试。

**Q：Redis 会不会把许可证改回来？**

根据 Redis CEO 的公开表态，不会。SSPL 许可证是 Redis 商业化战略的核心，改回去等于自断财路。

## 我的判断

Redis 改许可证是一步合理但短视的商业决策。短期内它保护了 Redis Ltd 的营收，但长期看，它把整个开源社区推给了 Valkey。

Valkey 两年的发展速度证明了一件事：**当一个足够重要的基础设施项目被商业利益绑架时，开源社区有能力重建一个更好的替代品。** 有 Linux 基金会的治理、有云厂商的工程师投入、有 BSD 许可证的无限制使用——Valkey 的护城河不是技术，是信任。

对于大多数后端团队来说，现在是迁移的好时机。兼容性风险已经被两年的生产验证覆盖，Spring Data Valkey 的发布补上了 Java 生态的最后一块拼图，性能还有明显提升。

唯一让我犹豫的是 Redis Stack 的功能差距。如果你重度依赖 RedisJSON 或 RediSearch，Valkey Search 还需要再观察一两个版本。其他场景，直接换。

## 参考资料

- [Valkey 官方博客](https://valkey.io/blog/)
- [Valkey GitHub](https://github.com/valkey-io/valkey) — 25.4k stars
- [Scaling a Valkey Cluster to 1 Billion RPS](https://valkey.io/blog/1-billion-rps/)
- [Unlock 1 Million RPS with Valkey](https://valkey.io/blog/unlock-one-million-rps/)
- [Spring Data Valkey 公告](https://valkey.io/blog/) — 2026.4.1
