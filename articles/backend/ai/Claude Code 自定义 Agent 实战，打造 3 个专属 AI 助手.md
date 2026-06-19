---
title: "Claude Code 自定义 Agent 实战：给团队造 3 个专属 AI 助手"
description: "Claude Code 自定义 Agent 完整实战：从 /agents 命令到手写 Markdown 配置，手把手创建代码审查、安全扫描、文档生成 3 个 Agent，含工具白名单、模型路由、持久化记忆配置"
date: "2026-04-08"
keywords: ["Claude Code", "自定义Agent", "AI编程", "Sub-agent", "代码审查", "开发工具"]
platform: "掘金"
source: "Claude Code 官方文档"

---

> 🖼️ **[封面图]** *封面图，3:2 横版*
>
> **提示词：** 见 `image-prompts/claude-code-custom-agents-guide-prompts.md`
>
> *替换为 `![封面图](cover.png)`*

Claude Code 内置了 Explore、Plan 和 general-purpose 三个 Sub-agent，日常使用已经够用了。但你有没有过这种体验：每次让 Claude 审查代码，都要重复一遍「关注安全漏洞、检查错误处理、看看有没有 N+1 查询」——本质上你在用对话复述一个 system prompt。

自定义 Agent 就是把这些重复的指令固化下来。一个 Markdown 文件，定义好 system prompt、工具权限和模型选择，以后只要说「用 code-reviewer 审查这个模块」，Claude 就知道该怎么做。

这篇文章不讲理论，直接动手造三个生产可用的 Agent：代码审查员、安全扫描器、文档生成器。每个都附完整配置，看完直接复制到你的项目里。

## 自定义 Agent 的运行机制

先用 30 秒搞清楚它是怎么工作的。

每个自定义 Agent 是一个 `.md` 文件，放在特定目录下。文件里有两部分：**YAML frontmatter**（配置元数据）和 **Markdown body**（system prompt）。当 Claude 判断当前任务适合委托给某个 Agent 时，它会启动一个独立的 context window，加载这个 Agent 的 system prompt，然后让这个 Agent 独立完成任务并返回结果。

![custom-agent-architecture](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/custom-agent-architecture.png)

关键点：

- **独立 context**：Agent 有自己的 context window，不会污染主对话
- **继承 CLAUDE.md**：Agent 会加载项目的 CLAUDE.md 规则
- **工具可限制**：你可以只给 Agent 读权限，不让它改代码
- **模型可选择**：审查这种不需要极强推理的任务，用 Sonnet 省钱又快

文件放哪？和 CLAUDE.md 一样分层：

| 位置                | 作用域   | 优先级               |
| ------------------- | -------- | -------------------- |
| `.claude/agents/`   | 当前项目 | 高（推荐提交到仓库） |
| `~/.claude/agents/` | 所有项目 | 低（个人偏好）       |


## Agent 1：代码审查员

这是最实用的一个。它在只读模式下审查代码，关注质量、安全和性能问题。

创建文件 `.claude/agents/code-reviewer.md`：

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and performance. Use proactively after code changes or when asked to review.
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
color: blue
---

你是一个高级代码审查员。审查代码时遵循以下原则：

## 审查维度（按优先级排序）

1. **安全漏洞**（最高优先级）
   - SQL 注入、XSS、命令注入
   - 硬编码的密钥或凭据
   - 不安全的反序列化
   - 路径遍历

2. **正确性**
   - 空指针 / 边界条件
   - 并发安全（共享状态、死锁风险）
   - 资源泄漏（未关闭的连接、流）
   - 错误处理是否完整

3. **性能**
   - N+1 查询
   - 不必要的数据库调用或网络请求
   - 大对象在循环内创建
   - 缺失的索引（如果能看到 SQL）

4. **可维护性**
   - 方法过长（>30 行）
   - 过深嵌套（>3 层）
   - 命名不清晰

## 输出格式

对每个问题，用这个格式：

**[严重程度: CRITICAL/WARNING/INFO]** `文件路径:行号`
问题描述（一句话）
建议修复方式（具体到代码级别）

## 规则

- 只报告真实问题，不报告风格偏好
- 如果代码质量好，直接说「没有发现问题」，不要硬凑反馈
- 审查完后更新你的 agent memory，记录发现的模式和项目特有的约定
```

注意几个关键配置：

- **`tools: Read, Glob, Grep, Bash`**：只给读权限，这个 Agent 不能改代码。Bash 留着是为了让它能跑 `git diff`、`git log` 来理解变更上下文
- **`model: sonnet`**：审查不需要 Opus 级别的推理能力，Sonnet 更快更便宜
- **`memory: project`**：启用项目级持久记忆。审查员会在 `.claude/agent-memory/code-reviewer/` 目录下积累知识——比如「这个项目用 BizException 不用 RuntimeException」这类模式

使用方式：

```
> 用 code-reviewer 审查 src/main/java/service/ 下最近修改的文件
```

Claude 会自动把任务委托给这个 Agent，Agent 在独立 context 里完成审查后把结果返回。


## Agent 2：安全扫描器

比代码审查员更聚焦，专门找安全问题。配合 Hooks 使用效果更好——每次提交前自动触发。

创建文件 `.claude/agents/security-scanner.md`：

```markdown
---
name: security-scanner
description: Scans code for security vulnerabilities, secrets, and compliance issues. Use before commits or when security review is needed.
tools: Read, Glob, Grep, Bash
model: sonnet
color: red
---

你是一个安全专家。扫描代码时只关注安全问题，不关注代码质量。

## 扫描清单

### 1. 硬编码密钥
扫描所有文件，查找以下模式：
- API key / secret / token（正则：`(?i)(api[_-]?key|secret|token|password)\s*[:=]\s*['"][^'"]+['"]`）
- AWS 凭据（`AKIA[0-9A-Z]{16}`）
- 私钥文件内容（`BEGIN.*PRIVATE KEY`）
- 数据库连接串中的密码

### 2. OWASP Top 10
- **注入**：拼接 SQL、未转义的用户输入直接进命令
- **认证**：硬编码 JWT secret、不过期的 token
- **敏感数据泄露**：日志里打印密码、响应里返回敏感字段
- **XXE**：XML 解析未禁用外部实体
- **SSRF**：用户可控的 URL 未做白名单校验

### 3. 依赖安全
如果有 package.json / pom.xml / go.mod：
- 运行 `npm audit` / `mvn dependency:tree` 检查已知漏洞
- 标记过时的安全相关依赖（如旧版本的 crypto 库）

## 输出格式

**[CRITICAL]** 硬编码 AWS 凭据
- 文件：`src/config/aws.js:15`
- 证据：`AKIAIOSFODNN7EXAMPLE`
- 修复：移至环境变量，使用 AWS SDK 的默认凭据链

扫描完成后给出一个总结：CRITICAL 数量、WARNING 数量、扫描的文件数。
```

使用方式：

```
> 用 security-scanner 扫描整个项目
```

进阶用法：配合 Hooks 在每次 `git commit` 前自动触发（通过 Stop 事件或自定义脚本调用 `claude --agent security-scanner`）。


## Agent 3：文档生成器

自动为代码生成文档。这个 Agent 有写权限，因为它需要创建或更新 README 和 API 文档。

创建文件 `.claude/agents/doc-generator.md`：

```markdown
---
name: doc-generator
description: Generates and updates documentation including README, API docs, and code comments. Use when documentation needs updating after code changes.
tools: Read, Glob, Grep, Write, Edit, Bash
model: sonnet
color: green
---

你是一个技术文档工程师。生成文档时遵循以下原则：

## 文档类型

### README.md
- 项目一句话描述
- 快速开始（3 步以内能跑起来）
- 核心功能列表
- 架构简述（如果项目有多个模块）
- 环境要求和依赖

### API 文档
- 每个公开 API 的请求/响应格式
- 参数说明（类型、是否必填、默认值）
- 错误码表
- curl 示例

### 代码注释
- 只在「为什么这样做」不明显时添加注释
- 不注释「做了什么」——代码本身就是说明
- 复杂算法加注释说明思路
- 临时绕过加 TODO 注释

## 规则

- 中文文档用中文，英文项目用英文
- 不要写「本文档介绍了……」这种废话开头
- 代码示例必须可运行
- 保持文档和代码同步——如果代码变了，更新对应文档
```

![three-agents-comparison](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/three-agents-comparison.png)


## 进阶配置：你可能不知道的功能

### 给 Agent 挂 MCP 工具

自定义 Agent 可以接入 MCP Server，获得主对话没有的工具。比如给测试 Agent 挂一个 Playwright：

```yaml
---
name: browser-tester
description: Tests features in a real browser using Playwright
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
---
```

内联定义的 MCP Server 只在这个 Agent 运行时连接，结束后断开。主对话的 context 里完全看不到这些工具——**零 token 开销**。

### 限制 Agent 可以 spawn 的子 Agent

如果你的 Agent 通过 `--agent` 作为主线程运行，可以限制它能调用哪些其他 Agent：

```yaml
---
name: coordinator
tools: Agent(code-reviewer, security-scanner), Read, Bash
---
```

这是白名单模式。coordinator 只能 spawn code-reviewer 和 security-scanner，不能用其他 Agent。

### 持久化记忆的三个作用域

| 作用域    | 存储位置                      | 适用场景             |
| --------- | ----------------------------- | -------------------- |
| `user`    | `~/.claude/agent-memory/`     | 跨项目通用知识       |
| `project` | `.claude/agent-memory/`       | 项目特有约定（推荐） |
| `local`   | `.claude/agent-memory-local/` | 本地私有，不提交     |

记忆启用后，Agent 的 system prompt 会自动注入 `MEMORY.md` 的前 200 行。你可以在 Agent 的 prompt 里写：「审查完后，把发现的项目约定保存到你的 memory 目录。」

![agent-creation-workflow](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/agent-creation-workflow.png)


## `/agents` 命令：不想手写文件的懒人方案

如果不想手动写 Markdown 文件，Claude Code 提供了交互式的 `/agents` 命令：

```
> /agents
```

它会引导你：

1. 选择「Create new agent」
2. 选作用域（Personal / Project）
3. 选「Generate with Claude」，用自然语言描述你想要的 Agent
4. Claude 自动生成 name、description、system prompt
5. 选工具权限
6. 选模型
7. 选颜色
8. 选记忆作用域
9. 保存

整个过程 30 秒完成。生成的文件和手写的格式完全一样，后续可以手动编辑微调。

**我的建议**：用 `/agents` 快速生成初稿，然后手动打开 `.md` 文件精修 system prompt。Claude 自动生成的 prompt 通常偏泛化，你需要加入项目特有的规则和约定。


## 常见问题

**Q：自定义 Agent 和 Agent Teams 是什么关系？**

自定义 Agent（Sub-agent）是在单个对话里运行的专家实例，由主对话分发任务。Agent Teams 是多个独立对话并行工作、可以双向通信的团队模式。如果你的需求是「让一个专家帮忙做件事然后把结果给我」，用 Sub-agent；如果需要「多个角色协作完成一个大项目」，用 Agent Teams。

**Q：Agent 的 context window 有多大？**

和主对话一样大（Claude 的 200K token 限制）。但因为 Agent 的 context 是独立的、只包含它自己的 system prompt 和工作内容，所以实际可用空间比主对话的「剩余空间」大得多。这也是 Sub-agent 的核心价值——**context 隔离**。

**Q：能不能让 Agent 用 Haiku 来省钱？**

可以，在 frontmatter 里设 `model: haiku`。内置的 Explore agent 就是用 Haiku 跑的。但 Haiku 的推理能力有限，适合简单搜索、格式化这类任务，复杂审查还是用 Sonnet。

**Q：Agent 能读到主对话的历史记录吗？**

不能。Agent 只能看到：1）它自己的 system prompt；2）Claude 分发给它的任务描述；3）它运行过程中的工具调用结果。这就是为什么给 Agent 写 system prompt 要尽量完整——别指望它「知道上下文」。

**Q：怎么查看所有已配置的 Agent？**

在对话里输入 `/agents` 可以看到交互式界面。在命令行里跑 `claude agents` 可以看到按来源分组的完整列表，还会标注哪些被同名高优先级定义覆盖了。

## 我的 Agent 组合推荐

| Agent            | model  | tools | memory  | 触发方式           |
| ---------------- | ------ | ----- | ------- | ------------------ |
| code-reviewer    | sonnet | 只读  | project | 改完代码后手动触发 |
| security-scanner | sonnet | 只读  | 无      | 提交前自动触发     |
| doc-generator    | sonnet | 读写  | 无      | 功能完成后手动触发 |

三个 Agent 总共 3 个 Markdown 文件，不到 200 行配置。但它们覆盖了代码审查、安全检查、文档维护三个最费时间的环节。

关键不是 Agent 有多智能，而是**把重复的人类指令固化成配置**。每省一次 prompt 输入，就省了一次 token 消耗和一次可能的遗漏。

## 参考资料

- [Claude Code Sub-agents 官方文档](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Agent Teams 文档](https://code.claude.com/docs/en/agent-teams)
- [Claude Code 权限配置](https://code.claude.com/docs/en/permissions)