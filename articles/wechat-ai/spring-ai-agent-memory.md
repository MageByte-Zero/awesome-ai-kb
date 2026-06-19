---
title: "给 Agent 加了 Redis 还是跨会话失忆，我排查后发现少了一层"
description: "Spring AI AutoMemoryTools 与 Session API 完整解析：从记忆持久化原理到向量库 vs Redis 存储选型，再到上下文压缩策略，手把手带你搭建不会失忆的 Java AI Agent。"
date: "2026-05-02"
keywords: ["Spring AI", "AutoMemoryTools", "Session API", "AI Agent 记忆", "向量数据库", "上下文压缩", "LangChain4j"]
platform: "微信公众号"
source: "Spring.io Blog"
alt_titles:
  - "AutoMemoryTools 为什么不用向量库存记忆？Spring AI 的设计反直觉"
  - "AI Agent 每次重启都失忆？Spring AI 这套记忆机制你没看懂"
  - "Spring AI Session API：大多数人用 ChatMemory 用错了场景"
  - "从失忆到记住一切：Spring AI AutoMemoryTools 与 Session API 实战"
---

给 Agent 加了 Redis 还是跨会话失忆，我排查后发现少了一层

![Spring AI Agent 记忆系统封面图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/spring-ai-agent-memory-cover.png)

前段时间做了个内部 AI 助手，接的是 Spring AI，给 `ChatClient` 挂上了 `MessageWindowChatMemory`，Redis 存对话历史，跑起来感觉很顺。直到有用户反馈：「我上周告诉过它我不喜欢代码里有 magic number，这周它又给我生成了一堆。」

去查了一遍，才发现问题根源不在 Redis，也不在配置——是我对 Spring AI 的记忆体系理解就错了。我以为加一个 `ChatMemory` 就解决了「Agent 记性」的问题，但实际上 Spring AI 的记忆分两层，我只接了一层。

这篇文章是那次排查之后的系统梳理。如果你正在做 Spring AI 的 Agent 项目，或者准备做，这两层记忆的区别是必须搞清楚的事。

## 先说清楚：Spring AI 记忆的两层架构

绝大多数人最先碰到的是 `ChatMemory`，用来保存对话历史，让 LLM 知道上下文。这是**短期记忆**，解决的是「这一次会话里说过什么」。

Spring AI 1.1 之后引入了 Session API，进一步强化了这一层，加上了事件溯源模型和上下文压缩能力。

但另一层——**长期记忆**——靠的是完全不同的机制：`AutoMemoryTools`。它解决的是「跨会话、跨重启，Agent 要记住的那些事」。

两层各司其职，缺一不可：

| 维度 | 短期记忆（Session API / ChatMemory） | 长期记忆（AutoMemoryTools） |
|------|--------------------------------------|----------------------------|
| 生命周期 | 当前会话 | 永久持久化 |
| 存储位置 | 内存 / JDBC / Redis | 文件系统（Markdown 文件） |
| 内容 | 完整对话历史 | 精选事实、偏好、决策 |
| 谁来管理 | Spring AI 框架自动管理 | LLM 自主决定写入/读取 |
| 适合场景 | 让 LLM 记住上下文 | 让 Agent 记住用户偏好和项目决策 |

我当时只接了短期记忆这层，所以用户的代码风格偏好在下次会话里当然没了——压根没人把这个事实写进长期记忆。

![Spring AI 两层记忆体系架构图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/memory-architecture.png)
*图：Spring AI Agent 两层记忆体系架构，蓝色为 AutoMemoryTools 长期记忆层，紫色为 Session API 短期记忆层*

## AutoMemoryTools：让 LLM 自己决定记什么

AutoMemoryTools 是 `spring-ai-agent-utils` 社区库提供的工具集，设计灵感直接来自 Anthropic Claude Code 的记忆系统。核心思路很简单：给 Agent 6 个文件操作工具，让它自己决定哪些事情值得记下来。

### 6 个工具覆盖完整的记忆生命周期

| 工具 | 作用 |
|------|------|
| `MemoryView` | 读文件内容（带行号）或列目录 |
| `MemoryCreate` | 创建带 frontmatter 的记忆文件 |
| `MemoryStrReplace` | 精确替换文件中的某段内容 |
| `MemoryInsert` | 在指定行之后插入内容 |
| `MemoryDelete` | 删除文件或目录 |
| `MemoryRename` | 移动/重命名文件（自动更新索引） |

这 6 个工具和文件系统工具很像，但有一个关键限制：**全部操作被沙箱化在 `memoriesDir` 目录内**。绝对路径会被拒绝，路径穿越会被检测，你不用担心 Agent 跑去写 `/etc/hosts`。

### 记忆文件的格式

每一条长期记忆是一个 Markdown 文件，带 YAML frontmatter：

```yaml
---
name: code-style-preference
description: 用户代码风格偏好
type: feedback
---

用户不希望代码里出现 magic number，变量命名倾向于语义化英文而非拼音缩写。

**Why:** 用户在 2026-04-28 的对话中明确纠正了生成的代码。
**How to apply:** 生成任何代码时，常量抽取为 static final，变量名用完整英文单词。
```

记忆类型有 4 种：
- `user`：角色、目标、专业背景、沟通偏好
- `feedback`：用户验证过的纠正和确认过的方法
- `project`：正在推进的工作、决策、截止日期
- `reference`：外部系统的指针（看板、仪表盘地址）

所有记忆文件的入口是 `MEMORY.md`，这是一个索引文件，Agent 在每次会话开始时会先读它，决定要加载哪些相关记忆：

```markdown
# Memory Index

- [code-style-preference.md](code-style-preference.md) — 用户代码风格偏好：不要 magic number
- [project-deadline.md](project-deadline.md) — 当前项目 Q2 上线计划
```

这个「索引优先」的设计很关键：即使长期记忆库积累了几十个文件，每次 Agent 也不需要把所有文件全塞进上下文，而是看索引摘要，按需精确加载。

### 最简接入方式：AutoMemoryToolsAdvisor

```java
// 依赖：spring-ai-agent-utils
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(
        // 1. AutoMemory Advisor：注入系统提示词 + 注册 6 个工具
        AutoMemoryToolsAdvisor.builder()
            .memoriesDir("/home/user/.agent/memories")
            .build(),
        // 2. 短期记忆：保留最近 100 条消息
        MessageChatMemoryAdvisor.builder(
            MessageWindowChatMemory.builder().maxMessages(100).build())
            .build(),
        // 3. 工具调用执行器
        ToolCallAdvisor.builder().disableInternalConversationHistory().build()
    )
    .build();
```

三个 Advisor 叠加，就完成了两层记忆的完整接入。AutoMemoryToolsAdvisor 自动把系统提示词注入进去，LLM 就知道什么时候该读记忆、什么时候该写。

### 什么情况下 LLM 会写记忆？

AutoMemoryTools 配套的系统提示词会指引 LLM 在以下场景触发写入：
- 用户明确纠正了它的某个行为（`feedback` 类型）
- 用户给出了关于自己的新信息（`user` 类型）
- 项目状态有变化（`project` 类型）

值得注意的是，**LLM 自己决定写什么**，不是框架强制写。这意味着记忆质量和你用的模型能力直接相关——Claude Sonnet 能写出很精准的记忆条目，弱模型可能会漏写或写得太宽泛。

还有一个机制叫 `memoryConsolidationTrigger`，可以配置每隔 N 轮对话或每 5% 概率触发一次记忆整理，防止积累太多冗余记忆。

### 踩坑：为什么不要把代码模式存进长期记忆

文档里有一段容易被忽略的警告：**代码模式、git 历史、调试方法不要存进 AutoMemoryTools**。这类内容应该存在代码库本身（注释、CLAUDE.md、README）里，而不是 Agent 的个人记忆。

原因很直接：这些内容会随代码变化，长期记忆文件不会自动同步代码库的变化，很快就会过时，变成噪音。

## Session API：短期记忆的「工业级」升级

在 Spring AI 1.1 之前，短期记忆的主要实现是 `MessageWindowChatMemory`——维护一个固定大小的消息窗口，简单但有明显局限：

- 按消息数量截断，不按 token 数量——实际上下文利用率低
- 截断时可能切断一半的工具调用 + 工具结果，LLM 看到孤立的 tool result 会困惑
- 没有持久化，重启即失

Spring AI 的 Session API（Agentic Patterns 系列第 7 篇）是对这层的彻底重设计，引入了**事件溯源模型**。

### 核心概念：Turn（轮次）是原子单位

Session API 的最大创新是把「一轮对话」定义为原子单位：**一条 UserMessage + 之后所有 AssistantMessage、ToolCall、ToolResult**，直到下一条 UserMessage 出现。

压缩时保证完整轮次要么全保留要么全丢弃，不会出现「保留了工具调用但丢掉了对应的工具结果」的情况。这解决了 `MessageWindowChatMemory` 最头疼的问题。

每个 `SessionEvent` 包装了一条 `Message`，附带了：
- UUID 和 sessionId
- 时间戳
- branch label（用于多 Agent 并行的分支隔离）
- `METADATA_SYNTHETIC` 标志（标记这是 LLM 生成的摘要，不是真实消息）

### 4 种压缩策略怎么选

| 策略 | 是否需要 LLM | 适用场景 |
|------|-------------|---------|
| `SlidingWindow` | 否 | 成本敏感，短期上下文 |
| `TurnWindow` | 否 | 对话结构清晰，N 轮之前的内容不重要 |
| `TokenCount` | 否 | 严格的 token 上限控制 |
| `RecursiveSummarization` | 是 | 长对话，历史信息需要保留但要压缩 |

`RecursiveSummarization` 是最智能也最贵的：它不从零总结，而是维护一个滚动压缩历史，每次把新增的「要被淘汰的内容」追加进去重新摘要，避免每次重算。总结结果会被标记为 `METADATA_SYNTHETIC`，用于持续压缩。

实际选型建议：

- 聊天机器人、单轮问答 → `SlidingWindow` 或 `TurnWindow`，够用不花钱
- 复杂任务 Agent，工具调用链很长 → `TurnWindow`（保证原子性）
- 长期工作 Agent，需要保留全局上下文 → `RecursiveSummarization`，但记得算 LLM 费用

```java
// Session API 接入示例
SessionMemoryAdvisor advisor = SessionMemoryAdvisor.builder(sessionService)
    .defaultUserId("user-alice")
    // 20 轮之后触发压缩
    .compactionTrigger(new TurnCountTrigger(20))
    // 压缩策略：保留最近 10 个 event，其余滑走
    .compactionStrategy(
        SlidingWindowCompactionStrategy.builder().maxEvents(10).build()
    )
    .build();
```

### 还有一个隐藏能力：Recall Storage

即使上下文被压缩掉了，Session API 保留了完整的原始事件日志，并且提供了 `conversation_search` 工具，Agent 可以用关键词搜索历史对话。

比如 Agent 在第 80 轮突然需要查找「第 5 轮说过的某个 API 地址」，可以直接调用工具检索，而不需要在上下文里一直保留第 5 轮的内容。这个设计参考了 MemGPT 的 Recall Storage 模式。

![Session API 上下文压缩决策流程图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/session-compaction-flow.png)
*图：Session API 四种压缩策略的选择决策流程，以及 Recall Storage 的保障机制*

### JDBC 持久化：生产环境必须做

Session API 默认的 `spring-ai-session-jdbc` 用两张表存储：

```sql
-- AI_SESSION：会话元数据
-- AI_SESSION_EVENT：append-only 事件日志
```

支持 PostgreSQL、MySQL、MariaDB、H2。对话历史跨重启保留，这才是生产级的短期记忆。

## 向量库 vs Redis vs 文件系统：长期记忆存储选型

除了 AutoMemoryTools 的文件系统方案，社区里有另一套思路：用 Redis 向量库做语义记忆检索。

### Spring AI + Redis 向量存储的架构

这套方案来自 Redis 官方博客，核心是两种记忆类型：

- **EPISODIC（情节记忆）**：个人经历、用户偏好，按时间顺序，精确检索
- **SEMANTIC（语义记忆）**：通用知识、事实，按语义相似度检索

存储用 `RedisVectorStore`：

```java
// Redis 向量索引配置
RedisVectorStore.RedisVectorStoreConfig config = RedisVectorStore.RedisVectorStoreConfig.builder()
    .withIndexName("longTermMemoryIdx")
    .withVectorAlgorithm(Algorithm.HSNW)  // 近似最近邻
    .withContentFieldName("content")
    .withEmbeddingFieldName("embedding")
    .addMetadataField(MetadataField.tag("memoryType"))
    .addMetadataField(MetadataField.tag("userId"))
    .build();
```

向量维度是 384 维 FLOAT32，使用 COSINE 距离。Redis 8 号称可以支持 10 亿向量不降低延迟，在规模上不是瓶颈。

两个关键 Advisor：

```java
// Retrieval Advisor：调用 LLM 之前，向量搜索最相关记忆并注入 system prompt
// Recorder Advisor：LLM 响应之后，提取原子事实并去重存入向量库
```

### 两种方案的对比

| 维度 | AutoMemoryTools（文件系统） | Redis 向量库 |
|------|----------------------------|-------------|
| 搜索方式 | 精确匹配（LLM 自行判断相关性） | 语义向量相似度 |
| 记忆管理 | LLM 自主决策 | Advisor 自动提取 |
| 运维复杂度 | 低（文件读写） | 中（Redis + 向量索引维护） |
| 记忆可读性 | 高（Markdown 文件，人可直接读） | 低（向量 + JSON，不直观） |
| 适合规模 | 小到中（百条级别） | 中到大（万条级别） |
| 冷启动 | LLM 读 MEMORY.md 索引 | 每次请求自动语义检索 |

**怎么选：**

如果 Agent 面向个人用户（每个用户几十到几百条记忆），AutoMemoryTools 的文件方案完全够用，而且可读性好，便于调试。

如果 Agent 需要在海量历史记忆中语义检索（比如企业知识库 Agent），Redis 向量方案更合适，但带来了额外的 embedding 调用开销和 Redis 运维负担。

两种方案并不互斥——你可以用 AutoMemoryTools 存「用户偏好」这类精准事实，用 Redis 向量库存「历史交互摘要」这类语义内容。

![AutoMemoryTools 文件系统与 Redis 向量库存储方案对比](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/memory-storage-comparison.png)
*图：两种长期记忆存储方案的全面对比，以及适用场景分析*

## Spring AI vs LangChain4j：记忆能力差距在哪里

国内文章比较两个框架时经常只比「支不支持某个模型」，记忆能力这块鲜有人细讲。

**LangChain4j 的记忆体系：**

LangChain4j 的聊天记忆基于 `ChatMemory` 接口，有 `MessageWindowChatMemory` 和 `TokenWindowChatMemory` 两种实现。`TokenWindowChatMemory` 按 token 数量而非消息数量控制窗口，这一点比 Spring AI 默认的实现更实用。

LangChain4j 没有内置类似 AutoMemoryTools 的跨会话长期记忆方案，需要自己组合 `EmbeddingStore`（支持向量存储）来实现语义记忆检索，灵活但需要更多 boilerplate。

**Spring AI 的优势：**

Spring AI 1.1+ 的 Session API 和 AutoMemoryTools 提供了更高层的抽象，两层记忆体系开箱即用，与 Spring Boot Advisor 链深度集成。对于已经在 Spring 生态里的团队，接入成本极低。

AutoMemoryTools 最大的创新点是**把记忆管理的决策权交给 LLM 本身**，而不是用代码规则决定「什么该存」，这在长期任务场景下效果明显更好——因为 LLM 能理解语义重要性，代码规则做不到。

**LangChain4j 的优势：**

运行时内存占用更小（Quarkus 下 50-100MB vs Spring Boot 的 150-300MB），如果你的 Agent 跑在资源受限的环境，这是明显优势。模型提供商支持更广，有些小众或私有部署模型只有 LangChain4j 有适配。

坦白说，2026 年两个框架都到了 1.x 阶段，功能差距已经在收窄。**选型的核心还是生态适配性**：用 Spring Boot 就用 Spring AI，用 Quarkus 就用 LangChain4j，不用强行跨栈。

## 把两层记忆接在一起：完整 Agent 示例

下面是一个把 Session API 和 AutoMemoryTools 都接上的完整示例，读者可以直接跑：

```java
// 前置条件：Spring AI 1.1+，spring-ai-agent-utils
// 依赖：
//   spring-ai-starter-model-openai (或其他模型)
//   spring-ai-session-jdbc
//   spring-ai-agent-utils (社区库)

@Configuration
public class AgentConfig {

    @Bean
    public ChatClient agentChatClient(
            ChatModel chatModel,
            SessionService sessionService) {

        return ChatClient.builder(chatModel)
            .defaultSystem("""
                你是一个有记忆的 AI 助手。你能记住用户的偏好、项目背景和历史交互。
                在回答问题之前，主动检查是否有相关记忆需要加载。
                当用户纠正你或透露新的偏好时，主动保存到长期记忆。
                """)
            .defaultAdvisors(
                // 长期记忆层：AutoMemoryTools（跨会话持久化）
                AutoMemoryToolsAdvisor.builder()
                    .memoriesDir(System.getProperty("user.home") + "/.agent/memories")
                    .build(),

                // 短期记忆层：Session API（当前会话 + 上下文压缩）
                SessionMemoryAdvisor.builder(sessionService)
                    .defaultUserId("demo-user")
                    .compactionTrigger(new TurnCountTrigger(15))
                    .compactionStrategy(
                        RecursiveSummarizationStrategy.builder()
                            .summarizationModel(chatModel)
                            .build()
                    )
                    .build(),

                // 工具调用执行器
                ToolCallAdvisor.builder()
                    .disableInternalConversationHistory()
                    .build()
            )
            .build();
    }
}

@RestController
@RequestMapping("/api/agent")
public class AgentController {

    private final ChatClient agentChatClient;

    @PostMapping("/chat")
    public String chat(@RequestParam String userId,
                       @RequestParam String message) {
        return agentChatClient.prompt()
            .user(message)
            // Session API 通过 AdvisorContext 传递用户 ID
            .advisorParam(SessionMemoryAdvisor.USER_ID_PARAM, userId)
            .call()
            .content();
    }
}
```

**关键点说明：**

1. `AutoMemoryToolsAdvisor` 必须在 `SessionMemoryAdvisor` 之前放，因为长期记忆的系统提示词需要在短期记忆之前注入
2. `disableInternalConversationHistory()` 避免 `ToolCallAdvisor` 自己维护一份多余的对话历史
3. `memoriesDir` 对不同用户应该隔离，实际项目中用 `{userId}` 做子目录

## 常见问题

**Q：AutoMemoryTools 用的是文件系统，部署在多个 Pod 上怎么办？**

A：这是目前社区版 AutoMemoryTools 的一个限制。多 Pod 场景下，你需要把 `memoriesDir` 指向共享存储（NFS、S3 挂载、云文件服务）。或者考虑用 Redis 向量方案替代，Redis 天然支持多节点共享。后续版本可能会提供 JDBC/Redis 作为后端的 AutoMemoryTools 实现。

**Q：Session API 的 RecursiveSummarization 会不会把重要内容总结丢？**

A：有这个风险，但 Session API 有两道保障：(1) 原始事件日志永远保留，即使被压缩，可以通过 `conversation_search` 工具按关键词检索；(2) Synthetic 摘要会标记 `METADATA_SYNTHETIC`，工程师可以监控摘要质量。如果业务对信息完整性要求极高，建议只用 `TurnWindow` 策略，不用 LLM 摘要。

**Q：AutoMemoryTools 的 6 个工具会占用多少 token？**

A：工具定义本身大约 500-800 token（取决于模型如何编码 JSON schema）。每次对话开始时还需要加载 `MEMORY.md` 索引，大约几十到几百 token。整体 overhead 相对于完整对话历史来说比较小，但对极度成本敏感的场景，可以考虑 Option B（手动接入），按需只加载特定工具。

**Q：Spring AI 的 ChatMemory 和 Session API 可以同时用吗？**

A：不建议同时用。Session API 是对 ChatMemory 的替代，不是补充，两个都接会导致对话历史被记两份。选一个：简单项目用 `MessageWindowChatMemory` 加 `MessageChatMemoryAdvisor`；生产项目用 Session API 的 `SessionMemoryAdvisor`。

**Q：LangChain4j 有没有类似 AutoMemoryTools 的东西？**

A：没有开箱即用的等价物。LangChain4j 的 `AiServices` 可以通过注解注入记忆，但长期跨会话记忆需要自己组合 `EmbeddingStore` 和提取逻辑来实现，代码量显著更多。

## 我的判断

AutoMemoryTools + Session API 这个组合是目前 Java 生态里 Agent 记忆最完整的开箱方案。短期记忆解决了「这次对话不乱」，长期记忆解决了「下次还记得你」，两层各司其职，不互相替代。

说实话，在这个方案出来之前，我们在生产上跑了好几个月的 Spring AI Agent，靠着各种临时方案拼凑记忆层——把偏好塞进 system prompt 硬编码、每次请求先 RAG 一遍历史记录。现在有了这套 API，很多事可以丢给框架了。

还没解决的问题是 AutoMemoryTools 的文件系统后端，对云原生多副本部署不够友好。这个等社区推出 JDBC 版本，或者自己先用 Redis 向量方案顶着。

下一篇打算写 Spring AI 的 Tool Calling 机制和 MCP 集成，比记忆这块更复杂，也更有意思。感兴趣的话关注一下，发出来会第一时间推送。

如果你身边有同事正在做 Spring AI 的 Agent 落地，这篇可以直接甩给他，少走点弯路。

## 参考资料

- [Spring AI Agentic Patterns (Part 6): AutoMemoryTools](https://spring.io/blog/2026/04/07/spring-ai-agentic-patterns-6-memory-tools/)
- [Spring AI Agentic Patterns (Part 7): Session API](https://spring.io/blog/2026/04/15/spring-ai-session-management/)
- [AutoMemoryTools 官方文档](https://spring-ai-community.github.io/spring-ai-agent-utils/0.8.0-SNAPSHOT/tools/AutoMemoryTools/)
- [Agent Memory with Spring AI & Redis - Foojay](https://foojay.io/today/agent-memory-with-spring-ai-redis/)
- [Spring AI Chat Memory 官方文档](https://docs.spring.io/spring-ai/reference/api/chat-memory.html)
