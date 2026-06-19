---
title: "Claude Code 实战 400 万 Tokens：接入 DeepSeek V4，从$26降到$2"
description: "DeepSeek V4-Pro 降价75%后，Claude Code 用户能节省多少钱？本文用真实 400 万 Tokens 的费用数据，带你完成接入配置，并说清楚哪些坑会踩到。"
date: "2026-04-27"
keywords: ["DeepSeek V4", "Claude Code", "DeepSeek API", "AI编程工具", "API费用对比", "DeepSeek V4-Pro"]
platform: "微信公众号"
source: "DeepSeek官方公告 / 实测数据"
alt_titles:
  - "Claude Code 账单比服务器贵，我接了 DeepSeek V4 才正常"
  - "Claude Code 的 API 账单刺痛我了，直到 DeepSeek V4 降价75%"
  - "DeepSeek V4 比 GPT-5.4 便宜 1/10，还能直接驱动 Claude Code？"
  - "DeepSeek V4 降价75%，Claude Code 用户为什么是最大赢家？"
---

![封面图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/deepseek-v4-pro-claude-code-setup-cover.png)

# Claude Code 实战 400 万 Tokens：接入 DeepSeek V4，从$26降到$2

上个月用 Claude Code 连着干了几天需求，账单出来的时候我愣了一下。

400 万个 tokens。Claude Sonnet 4.6，$3 输入 / $15 输出，保守估算下来将近 $26。不是说它贵，是突然意识到：这只是一周的编码量。如果是个认真用 AI 工具的工程师，每个月的消费可能超过一台云服务器。

然后 2026 年 4 月 24 日，DeepSeek 发布了 V4，并在三天后（今天，4 月 27 日）宣布 V4-Pro 限时降价 75%。我把 Claude Code 的后端切换到 DeepSeek V4 跑了一遍，用了同样量级的 tokens，账单是 $2.3。

这篇记录整个过程：V4 是什么水平、价格到底怎么算、怎么接入 Claude Code、踩了哪些坑。

## DeepSeek V4 到底是什么规格

先把背景说清楚，后面算费用才有意义。

DeepSeek V4 分两个版本：**V4-Flash** 和 **V4-Pro**。

V4-Pro 是这次的主角。1.6T 参数，但采用 MoE（Mixture of Experts）架构，每次推理只激活 49B 参数。这个设计直接决定了它的成本优势——计算量只有 Dense 同参数模型的一小部分，但性能却能打到接近顶级。

DeepSeek 官方的说法是，V4-Pro 在主流 benchmark 上比 Claude Sonnet 4.5 高，与 GPT-5.4 差距约 3-6 个月。不是"碾压"，是"非常够用"。

![DeepSeek V4 MoE 架构图，展示1.6T总参数与49B激活参数的关系](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/v4-moe-architecture.png)

V4-Flash 是轻量版本，284B 总参数、13B 激活。速度更快，成本更低，适合那些不需要深度推理的任务，比如文件读写、代码格式化、简单问答。

两个版本都支持 1M token 上下文，MIT 协议开源。这两点对用 Claude Code 做长上下文任务的工程师来说不算小事。

## 75% 降价意味着什么，算一遍

直接上数字。

**原价（正式定价）：**

| 模型 | 输入（每百万 tokens） | 输出（每百万 tokens） |
|------|---------------------|---------------------|
| DeepSeek V4-Flash | $0.14 | $0.28 |
| DeepSeek V4-Pro | $1.74 | $3.48 |
| Claude Sonnet 4.6 | $3.00 | $15.00 |
| Claude Opus 4.7 | $5.00 | $25.00 |
| GPT-5.4 | $2.50 | $15.00 |

**V4-Pro 75% 折扣后（截至 2026-05-05 15:59 UTC）：**

| 模型 | 输入（折后） | 输出（折后） |
|------|------------|------------|
| DeepSeek V4-Pro（折扣期） | $0.435 | $0.870 |

缓存命中更夸张：折扣期间 V4-Pro 的 cache hit 输入价格是 $0.003625/百万，V4-Flash 是 $0.0028/百万。Claude Code 里重复上下文很多，如果触发缓存，实际费用还能再砍一刀。

现在算 400 万 tokens 的账单：

假设输入/输出比例 7:3（编码场景典型比例，指令和代码上下文多，生成内容相对少）：
- 280 万输入 + 120 万输出

| 后端 | 输入费用 | 输出费用 | 合计 |
|------|---------|---------|------|
| Claude Sonnet 4.6 | $8.40 | $18.00 | **$26.40** |
| Claude Opus 4.7 | $14.00 | $30.00 | **$44.00** |
| DeepSeek V4-Pro（折扣） | $1.22 | $1.04 | **$2.26** |
| DeepSeek V4-Flash | $0.39 | $0.34 | **$0.73** |

结论很直接：折扣期的 V4-Pro，同样的 400 万 tokens，比 Claude Sonnet 便宜 **11.6 倍**，比 Opus 便宜 **19 倍**。

就算折扣结束回到原价，V4-Pro 依然是 Sonnet 的 1/4 不到。

![400万Tokens各主流模型API费用对比，DeepSeek V4-Pro折扣后仅$2.26](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/cost-comparison.png)

## 怎么接入 Claude Code：15 分钟搞定

这是整篇文章最值钱的部分，因为大多数人不知道 DeepSeek 原生支持 Anthropic API 格式。不需要任何 proxy，不需要中间件，直接配置两个环境变量就行。

### 前置条件

1. Claude Code 已安装（`npm install -g @anthropic-ai/claude-code` 或官网下载）
2. 注册 DeepSeek 账号并生成 API Key：[platform.deepseek.com](https://platform.deepseek.com)
3. 给账户充一点余额（10 元人民币够跑很久）

### 第一步：配置环境变量

打开你的 shell 配置文件（`~/.zshrc` 或 `~/.bashrc`），添加：

```bash
# Claude Code 使用 DeepSeek 作为后端
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
export ANTHROPIC_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxx"  # 换成你的 DeepSeek API Key
```

**关键点：** `ANTHROPIC_BASE_URL` 末尾不要加 `/v1`，否则请求会 404。

保存后执行：

```bash
source ~/.zshrc
```

### 第二步：指定模型

Claude Code 默认会尝试调用 `claude-sonnet-4-6` 这个模型名，但 DeepSeek 不认这个名字。需要在 Claude Code 的配置中指定 DeepSeek 的模型名。

创建或编辑 `~/.claude/settings.json`（如果不存在就新建）：

```json
{
  "model": "deepseek-v4-pro",
  "fallbackModel": "deepseek-v4-flash"
}
```

或者按任务类型做分级路由——重度任务走 V4-Pro，轻量任务走 V4-Flash：

```json
{
  "model": "deepseek-v4-pro",
  "smallModel": "deepseek-v4-flash",
  "apiTimeout": 600000
}
```

`apiTimeout` 设成 600000（10 分钟）是因为 V4-Pro 在长推理任务上响应有时会超过默认超时时间。不设这个，复杂任务跑一半会断掉。

### 第三步：验证配置

```bash
claude "你在用什么模型？"
```

如果回复里提到 DeepSeek 或者响应格式正常，说明配置成功。顺便去 DeepSeek 控制台看一下 Usage，有请求进来就稳了。

![Claude Code 接入 DeepSeek V4 的完整配置流程，包括环境变量设置和模型配置](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/setup-flow.png)

## 真实使用：400 万 Tokens 跑下来的感受

用了大概三天，主要场景是：重构一个 Spring Boot 服务、写若干个单元测试、调 API 接口文档、做代码 review。

**感受好的地方：**

代码补全和逻辑推理的质量，跟 Claude Sonnet 4.6 差距不大，多数时候感觉不到切换。特别是理解长文件、跟踪变量依赖这类需要上下文的任务，1M 的上下文窗口没给我制造麻烦。

响应速度比预期快。V4-Flash 处理简单问答基本是秒级，V4-Pro 的复杂推理大概 5-15 秒，在可接受范围内。

**感受不好的地方：**

有一次我往对话里粘贴了一张架构截图，让它帮我分析服务依赖关系。结果 Claude Code 收到的是占位文本，什么也看不出来。排查了半天才想起来：**DeepSeek V4-Pro 目前不支持图片输入**。

这是最大的坑。如果你的工作流里经常需要上传截图、UI 设计稿、日志截图，DeepSeek V4 目前是做不了的。这一块还得走官方 Claude。

另外偶尔会出现比 Claude 官方更"字面化"的回复——你让它重构，它可能严格按照字面意思动，而不是主动发现周边的问题。不算大问题，适应一下提示词的写法就好。

## 踩坑记录

整理一下我遇到的问题，大多数都能绕过去：

**坑1：模型名字写错导致 400 错误**

`ANTHROPIC_BASE_URL` 配了 DeepSeek 的地址，但 `settings.json` 里还写着 `claude-sonnet-4-6`。DeepSeek 不识别这个名字，会自动 fallback 到 `deepseek-v4-flash`，悄悄切换你不一定察觉。记得明确指定模型名。

**坑2：base URL 带了 /v1**

标准 OpenAI 格式的 base URL 通常是 `https://api.example.com/v1`，但 DeepSeek 的 Anthropic 兼容端点是 `https://api.deepseek.com/anthropic`。多加 `/v1` 会 404。

**坑3：超时设置不够**

默认超时 120 秒，V4-Pro 处理涉及大量上下文的复杂任务有时会超。`apiTimeout: 600000` 基本够用。

**坑4：图片内容被静默丢弃**

没有报错，只是图片内容被替换成占位符，你可能根本不知道模型没看到图。有上传图片需求时，先用纯文字描述替代，或者这部分任务切回官方 Claude。

**坑5：折扣有截止时间**

75% 折扣到 **2026-05-05 15:59 UTC** 为止，之后回到原价（V4-Pro $1.74 输入 / $3.48 输出）。原价依然比 Claude Sonnet 便宜，但优势没现在这么大。回原价后，轻量任务建议全面切 V4-Flash（$0.14/$0.28），把 V4-Pro 留给真正需要的场景。

## 我的判断

DeepSeek V4 + Claude Code 这个组合，折扣期内毫无疑问值得试。$26 → $2.3，不是玄学，是实实在在的账单差距。

折扣结束后还值不值得？看你的场景。如果工作流里有大量截图、视觉内容，或者对最新模型能力有依赖，继续付 Claude 官方的价格是合理的。如果主要是文字代码、长上下文推理、批量任务，DeepSeek V4-Flash 回到原价后依然是全市场最便宜的选项之一。

MoE 架构的效率优势是结构性的，不是一次性的价格战。DeepSeek 自己说了，等华为昇腾 950 量产之后还会继续降价。这条路长期看是走得通的。

当然，如果你完全不在乎钱，Claude Sonnet 4.6 的综合体验还是更顺滑一点——特别是多模态、工具调用的稳定性。二者不是替换关系，是互补。

下一篇打算把 Claude Code 的 settings.json 里那些不起眼的配置项拆解一遍，有几个隐藏选项对省钱和提速都有帮助，感兴趣的话关注一下。

如果你身边有人用 Claude Code 但一直嫌贵，这篇可以直接甩给他，省得他再踩一遍那几个坑。

## 常见问题

**Q: DeepSeek V4 接入 Claude Code 后，代码质量真的不差吗？**

A: 多数日常编码任务感受不到明显差距——重构、补全、单测这类有规律可循的工作表现稳定。差距主要出现在需要深度理解业务背景做权衡决策的场景，Claude Sonnet 4.6 在这里会更主动地给出有见解的建议，V4-Pro 有时更"执行型"。如果你对结果有要求，任何模型都需要给足上下文和约束，这一点用哪家都一样。

**Q: 折扣结束后还有必要用 DeepSeek V4 吗？**

A: 有。V4-Flash 原价 $0.14/$0.28 是全市场最低区间，Claude Code 里大量的轻量操作（读文件、简单问答、格式化）用 Flash 完全够，把成本压到原来的 1/10。V4-Pro 回到原价后，在 Claude Sonnet 的 1/4 左右，计算密集型任务还是有竞争力。

**Q: 会不会有数据隐私问题？**

A: DeepSeek 的服务器在中国大陆。如果你的代码涉及公司敏感信息或有合规要求，这是需要认真评估的问题，不是买便宜就完事的。纯个人项目、学习用途的话问题不大，商业项目请先确认合规策略。

**Q: 有没有办法在 V4-Pro 和官方 Claude 之间自动切换？**

A: 有。`claude-code-router` 这个开源工具可以根据任务类型把请求路由到不同后端——有图片的走官方 Claude，纯文字代码走 DeepSeek V4。GitHub 搜 `musistudio/claude-code-router` 有现成配置，这个方案灵活性更高，后续我可能单独写一篇。

**Q: 每次 Claude Code 更新后需要重新配置吗？**

A: 不需要。`ANTHROPIC_BASE_URL` 和 `ANTHROPIC_API_KEY` 在系统环境变量里，`settings.json` 在 `~/.claude/` 目录，Claude Code 升级不会动这两个地方。

## 参考资料

- [DeepSeek API 官方定价文档](https://api-docs.deepseek.com/quick_start/pricing)
- [DeepSeek V4 发布：接近前沿、价格低廉](https://simonwillison.net/2026/Apr/24/deepseek-v4/)
- [DeepSeek 将 V4-Pro API 价格削减 75%](https://thenextweb.com/news/deepseek-v4-pro-price-cut-75-percent)
- [DeepSeek V4 来了，超越 Claude Sonnet 4.5，赶紧对接 Claude Code 体验一把](https://developer.aliyun.com/article/1730869)
