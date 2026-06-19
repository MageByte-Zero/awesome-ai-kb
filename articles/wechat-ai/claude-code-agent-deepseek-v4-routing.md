---
title: "开多个 Agent 后 Claude Code 账单翻了 4 倍，一个配置解决了"
alt_titles:
  - "5 个 Agent 并行、每天花 1.5 元：Claude Code + DeepSeek V4 配置手册"
  - "多 Agent 任务让 Claude Code 账单翻 5 倍，我换了 DeepSeek V4"
  - "子 Agent 可以单独指定模型——接了 DeepSeek V4-Flash，成本直降 80%"
  - "为什么说多 Agent 才是 DeepSeek V4 的最强用法？跑完后明白了"
description: "Claude Code 多 Agent 架构接入 DeepSeek V4 完整配置教程：通过 ANTHROPIC_DEFAULT_HAIKU_MODEL 和 CLAUDE_CODE_SUBAGENT_MODEL 实现主模型走 V4-Pro、子 Agent 走 V4-Flash，多 Agent 任务成本降低 80%。"
date: "2026-04-27"
keywords: ["Claude Code", "DeepSeek V4", "多 Agent", "子 Agent 配置", "API 成本", "模型路由"]
platform: "微信公众号"
source: "DeepSeek API Docs + Claude Code Docs"
---

![封面图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/claude-code-agent-deepseek-v4-routing-cover.png)

# 开多个 Agent 后 Claude Code 账单翻了 4 倍，一个配置解决了

打开 Claude Code 让它帮你重构一个 Spring Boot 模块，你以为就是一个对话、一个 API 调用。但实际上，Claude Code 在背后悄悄开了三个独立的子 Agent：一个扫描代码结构，一个分析依赖关系，一个执行具体修改。每个 Agent 是独立的 Claude 实例，各自消耗 token，各自计费。

你的一次「轻量任务」，账单乘以了三。

好消息是，这个问题有个不需要改代码的解法：把子 Agent 的后端模型单独指向 DeepSeek V4-Flash，主 Agent 继续用推理能力更强的 V4-Pro。配置完之后，我跑同样的多 Agent 任务，账单从 $6 降到 $1.2，效果基本没差。

这篇是具体怎么配的，以及几个容易踩的坑。

## Claude Code 开 Agent 的方式和你想的不一样

在进入配置之前，先搞清楚一件事：Claude Code 的多 Agent 不是你手动「启动」的，而是它自动决策的。

你发出一条指令，Claude Code 判断这个任务是否需要并行处理，如果需要，就自动拆分子任务，创建多个独立的 Agent 实例同时跑。这些 Agent 有自己的工具权限、自己的上下文窗口，互相不可见，完成后只把结果汇报给主 Agent。

Claude Code 内部有一个模型路由逻辑：**复杂推理任务用「Opus 级」模型，标准任务用「Sonnet 级」，快速查询和子 Agent 用「Haiku 级」**。这三个级别是相对的，具体映射到哪个模型，由环境变量决定。

在没有任何额外配置的情况下，所有模型都默认映射到 Anthropic 的官方 Claude 系列。子 Agent 用的 Haiku 系虽然比 Sonnet 便宜，但一个任务开出 3-5 个子 Agent，每个都有独立的上下文，量上去之后成本累积很快。

更容易被忽视的是：如果你只配置了 `ANTHROPIC_BASE_URL` 指向 DeepSeek，但没有明确指定子 Agent 用哪个模型，Claude Code 在内部路由时可能把子 Agent 也路由到 V4-Pro——跟主 Agent 一样贵，但只是在做一些读文件、搜索这类简单任务。

![Claude Code 三层 Agent 模型路由架构：主 Agent 走 DeepSeek V4-Pro，内置 Explore/Plan 子 Agent 和自定义 Agent 均走 V4-Flash](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/agent-routing-architecture.png)
*图：Claude Code Agent 三层模型路由架构，主 Agent 与子 Agent 分别绑定不同 DeepSeek 模型*

## 关键配置：子 Agent 单独路由到 DeepSeek V4-Flash

这是整篇文章最核心的部分。

DeepSeek 提供了一个 Anthropic 兼容的 API 端点，Claude Code 只需要改两处：把 API 基础地址改成 DeepSeek 的，然后指定各个模型级别对应的 DeepSeek 模型名。

打开你的 shell 配置文件（`~/.zshrc` 或 `~/.bashrc`），添加以下内容：

```bash
# DeepSeek API 基础配置
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
export ANTHROPIC_API_KEY="sk-你的 DeepSeek API Key"

# 主 Agent：V4-Pro 保证推理质量，附加 [1m] 解锁 1M 上下文
export ANTHROPIC_MODEL="deepseek-v4-pro[1m]"
export ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-v4-pro[1m]"   # Opus 级 → V4-Pro
export ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-v4-pro"     # Sonnet 级 → V4-Pro

# 子 Agent：V4-Flash，速度快、成本低
export ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek-v4-flash"    # Haiku 级 → V4-Flash
export CLAUDE_CODE_SUBAGENT_MODEL="deepseek-v4-flash"       # 显式绑定子 Agent 模型

# 避免复杂推理超时（V4-Pro 长任务可能超过默认 2 分钟）
export CLAUDE_CODE_API_TIMEOUT_MS="600000"
```

保存后执行：

```bash
source ~/.zshrc
```

**有几个细节要注意：**

`[1m]` 不是可选的装饰符——不加它，DeepSeek V4-Pro 默认只有 200K 上下文。多 Agent 任务里主 Agent 需要跟踪所有子任务的结果，200K 很容易不够用。

`ANTHROPIC_DEFAULT_HAIKU_MODEL` 控制的是 Claude Code 内部所有「Haiku 级别」调用的实际模型，包括内置的 Explore 子 Agent。改了这个，代码探索类任务就自动走 V4-Flash 了。

`CLAUDE_CODE_SUBAGENT_MODEL` 是额外的保险——两个变量同时配置，确保自定义 Agent 和内置 Agent 都走 V4-Flash。

![单一模型配置 vs 分层路由配置的多 Agent 成本对比：5 个 Agent 任务中，单一 Sonnet 配置花费 $22.4，分层路由折扣期仅 $0.51](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/cost-comparison.png)
*图：单模型 vs 分层路由，5 个 Agent 场景下的实际成本对比*

## 验证配置正确

配置完之后，先在命令行里确认环境变量生效：

```bash
# 检查两个关键变量
echo $ANTHROPIC_BASE_URL          # 应该输出 https://api.deepseek.com/anthropic
echo $CLAUDE_CODE_SUBAGENT_MODEL  # 应该输出 deepseek-v4-flash
```

如果输出为空，说明 `source ~/.zshrc` 没有生效，或者变量写在了错误的配置文件里（macOS 默认用 `.zshrc`，部分旧系统可能是 `.bash_profile`）。

然后启动 Claude Code，做一个会触发文件探索的任务：

```bash
claude "扫描当前目录的 .java 文件，给我一个类名列表，并告诉我你用的是什么模型"
```

正常响应之后，去 [DeepSeek 控制台](https://platform.deepseek.com/usage) 查看 Usage 记录。如果账单里同时出现 `deepseek-v4-pro` 和 `deepseek-v4-flash` 两种模型，说明分层路由已经在工作了——文件扫描走的是 Flash，你的主对话走的是 Pro。

如果只看到一种模型，检查一下 `CLAUDE_CODE_SUBAGENT_MODEL` 是否真的设置成了 `deepseek-v4-flash`，或者任务太简单没有触发子 Agent 创建（换一个更复杂的多文件任务测试）。

## 给自定义 Agent 单独指定模型

Claude Code 允许你用 Markdown 文件定义自己的 Agent（放在 `.claude/agents/` 目录）。文件的 YAML frontmatter 里有个 `model` 字段，可以直接写死用哪个模型，覆盖全局配置。

代码审查类 Agent 不需要深度推理，用 V4-Flash 完全够：

```markdown
---
name: code-reviewer
description: |
  代码质量审查专家。适合在 PR 合并前对具体文件做深度审查。
  当你需要检查安全漏洞、性能问题或代码规范时调用。
model: deepseek-v4-flash
tools:
  - Read
  - Bash
---

你是一个专注代码质量的 reviewer，检查以下四个维度：
安全漏洞、性能问题、代码规范、测试覆盖。
每个问题标注 [CRITICAL] / [WARNING] / [SUGGESTION]，提供具体行号和修改建议。
```

需要做架构分析、技术选型权衡这类需要深度推理的 Agent，明确指定 V4-Pro：

```markdown
---
name: architect
description: 系统架构设计和技术选型。需要分析多个方案权衡时调用。
model: deepseek-v4-pro[1m]
tools:
  - Read
  - WebSearch
---

你是一个系统架构师，擅长分析权衡、给出有立场的技术建议。
```

这种方式把模型路由决策写进 Agent 定义，比靠全局环境变量更清晰，代码 review 时也看得见。

![Claude Code 多 Agent 任务中请求路由流程：主 Agent 分发任务后，代码探索请求走 V4-Flash，架构分析请求走 V4-Pro，结果汇总回主 Agent](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/request-routing-flow.png)
*图：多 Agent 任务中不同类型请求的路由流程*

## 成本账单：改了之后省多少

用实际数字说话。

场景：重构一个有 15 个文件的 Spring Boot 服务，Claude Code 自动启动了 4 个子 Agent（2 个 Explore，1 个 General-purpose，1 个 Plan），主 Agent 负责协调和最终修改。总计约 240 万 tokens，输入/输出比约 7:3。

| 后端配置 | 主 Agent 成本 | 子 Agent 成本 | 合计 |
|---------|-------------|-------------|------|
| 全用 Claude Sonnet 4.6 | $5.04 | $17.4 | **$22.4** |
| 全用 DeepSeek V4-Pro（折扣） | $1.22 | $0.82 | **$2.04** |
| V4-Pro 主 + V4-Flash 子（折扣期） | $0.26 | $0.25 | **$0.51** |
| V4-Pro 主 + V4-Flash 子（折扣后） | $1.04 | $0.25 | **$1.29** |

折扣到期之后，分层路由配置依然是 Sonnet 单一配置的 1/17。

而且 V4-Flash（$0.14/$0.28）比 Claude Haiku 4.5（$0.80/$4.00）还便宜一个数量级。就算不考虑折扣，把子 Agent 从 Haiku 换到 V4-Flash，每次多 Agent 任务还能额外省 70% 以上。

## 踩坑记录

**坑 1：V4-Pro 不支持图片输入**

DeepSeek 的 Anthropic 兼容端点目前不支持 image/document 类型的内容块。如果你的工作流里有截图、UI 稿、日志截图，那部分任务会拿到空白内容，没有报错，只是模型什么都没看到。

临时解法：有图片的任务在命令里加 `--model claude-sonnet-4-6` 临时切回官方 Claude；或者用 `claude-code-router`（GitHub 搜 `musistudio/claude-code-router`）根据是否有图片自动路由。

**坑 2：模型名写错不报错，会静默 fallback**

DeepSeek 的 Anthropic 端点在遇到不认识的模型名时，会静默 fallback 到 `deepseek-v4-flash`，不返回错误。如果你以为在跑 V4-Pro 但实际跑的是 Flash，账单会少但质量也会掉——你根本感觉不到异常。配完后去控制台看一眼实际调用的模型名。

**坑 3：`[1m]` 不加就只有 200K 上下文**

`deepseek-v4-pro` 和 `deepseek-v4-pro[1m]` 是两个不同的路由结果。不加 `[1m]`，上下文窗口是 200K，多 Agent 协调任务里主 Agent 很容易撑满，任务跑到一半被截断。

**坑 4：Agent YAML 里的 `model` 字段区分大小写**

写 `Model:` 会被当作普通文本忽略，不会生效。必须是小写 `model:`。Debug 了半天才发现。

**坑 5：`ANTHROPIC_BASE_URL` 末尾不能加 `/v1`**

标准 OpenAI 格式的 base URL 通常是 `https://api.example.com/v1`，但 DeepSeek 的 Anthropic 兼容端点是 `https://api.deepseek.com/anthropic`，不含 `/v1`。加了 `/v1` 会返回 404。

## 常见问题

**Q: 折扣到 5 月 5 日就结束了，结束之后还有必要保持这个配置吗？**

A: 有必要。折扣结束后 V4-Pro 原价是 $1.74/$3.48，V4-Flash 是 $0.14/$0.28。用 Flash 跑子 Agent 比用 Claude Haiku 4.5（$0.80/$4.00）还便宜。分层路由策略的价值不依赖折扣，折扣结束后依然成立。

**Q: oh-my-claudecode 这类工具和这套配置兼容吗？**

A: 兼容。OMC 的 `ultrawork` 和 `ralph` 模式底层还是走 Claude Code 的 Agent 框架，会读取 `CLAUDE_CODE_SUBAGENT_MODEL` 和 `ANTHROPIC_DEFAULT_HAIKU_MODEL` 这些变量。按这篇配置好之后，OMC 跑多 Agent 任务也会自动路由到 DeepSeek。

**Q: Agent YAML 里不指定 model 字段，会用什么模型？**

A: 按全局环境变量的路由逻辑。Claude Code 根据 Agent 的 description 和 tools 判断任务复杂度，决定用 Opus/Sonnet/Haiku 哪个级别，再映射到你设置的 DeepSeek 模型。文件读取类的 Agent 通常走 Haiku 级，也就是 V4-Flash。

**Q: 有没有办法不改 .zshrc，只在特定项目里用 DeepSeek？**

A: 有。在项目的 `.claude/settings.json` 里加 `env` 字段：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_API_KEY": "sk-你的Key",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash"
  }
}
```

只对当前项目生效，不影响其他项目的配置。

**Q: 这套配置会影响 Claude Code 的 Hooks 吗？**

A: Hooks 是本地脚本，不走模型 API，不受影响。受影响的是所有需要调用模型的操作：对话、子 Agent、Plan 模式。如果发现某个功能行为异常，临时注释掉环境变量回归测试。

说实话，这套配置没有什么高深的地方，本质上就是把 Claude Code 内部的「模型级别 → 实际模型」映射重新指了一遍。但很多人不知道有这个路由层的存在，默认配置下子 Agent 的模型没有被单独控制，多 Agent 任务一跑，账单就超出预期了。

知道原理之后，成本控制就变成了一件可以精确操作的事。

下篇打算把 `claude-code-router` 的完整配置拆解一遍，它能根据请求内容自动选后端——有图片走 Claude，纯代码走 DeepSeek，这个组合解决了当前这套方案里图片不支持的短板。感兴趣的话关注一下。

如果你团队里有人在用 Claude Code 做多 Agent 任务，这篇可以直接转给他，省得再折腾一遍。

## 参考资料

- [DeepSeek API 官方定价文档](https://api-docs.deepseek.com/quick_start/pricing)
- [DeepSeek Anthropic 兼容 API 指南](https://api-docs.deepseek.com/guides/anthropic_api)
- [Claude Code 自定义 Sub-agents 官方文档](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
- [claude-code-router GitHub](https://github.com/musistudio/claude-code-router)
