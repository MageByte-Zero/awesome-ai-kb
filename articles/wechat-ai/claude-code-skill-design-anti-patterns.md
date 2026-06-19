---
title: "我写了 50 个 Claude Code Skill 才发现，前 30 个都白写了"
description: "从 awesome-codex-skills 一周涨 4290 星说起，拆解 Claude Code Skill 的 7 个反模式，附 5 个我每天都在用的 Skill 模板和 Codex 适配指南。"
date: "2026-05-03"
keywords: ["Claude Code Skill", "AI Agent Skill", "OpenAI Codex Skill", "MCP", "AI 编程工具"]
alt_titles:
  - "OpenAI 抄了 Anthropic 的 Skill，但又没完全抄，差异藏在一行配置里"
  - "2026 年了，你还不会写 Claude Code Skill？AI 工具的下一个标准已经定了"
  - "Anthropic 工程师不会告诉你：写 Skill 的真正难点不是 SKILL.md"
  - "5 个 Skill 让我每天少敲 1 万字，附 Claude Code 和 Codex 双适配模板"
platform: "微信公众号"
source: "GitHub Trending Python + Anthropic Blog + OpenAI Codex Docs"
---

![我写了 50 个 Claude Code Skill，前 30 个都白写了 - 码哥字节封面](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/claude-code-skill-design-anti-patterns-cover.png)

# 我写了 50 个 Claude Code Skill 才发现，前 30 个都白写了

凌晨一点，我看着 Claude 又一次完美绕过了我精心写的那个 Skill，第三次了。

prompt 关键词全对上了，描述里写得清清楚楚「用于 Spring Boot 项目的接口设计」，结果它愣是没触发。我开始怀疑自己——是不是 Claude Code 这个东西本身就不靠谱？

第二天起来翻官方文档，又翻了 Anthropic 工程师那篇被很多人转的 Medium 长文，才反应过来：**问题不在 Claude，问题在我自己。我以为 Skill 就是 prompt 模板的升级版，结果完全搞反了它的设计哲学。**

那是去年 11 月的事。后来我陆陆续续写了 50 多个 Skill，回过头看，前面那 30 个基本可以全删了。今天这篇就把这一年踩过的坑、想明白的事一次性讲清楚——尤其是当 OpenAI 这周也下场卷 Skill 的时候，事情就变得有点意思了。

![Anthropic Skill 与 OpenAI Codex Skill 生态对比图：127k vs 6k stars，但 SKILL.md 格式 99% 一致](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/skill-ecosystem-comparison.png)

## Skill 突然变成了 AI 工具的下一个战场

这周 GitHub Trending Python 榜上有个项目一周涨了 4290 星，叫 `awesome-codex-skills`,ComposioHQ 维护的。点进去一看,**这是 OpenAI Codex CLI 的 Skill 仓库,不是 Claude Code 的。**

我第一反应是「Codex 也搞 Skill 了?」第二反应是把两边的 SKILL.md 格式拎出来对比——结果发现两份 frontmatter 几乎一模一样:

```yaml
# Anthropic 的 SKILL.md
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---
```

```yaml
# OpenAI Codex 的 SKILL.md
---
name: my-skill-name
description: What the skill does and when Codex should use it.
---
```

差了两个字母。

但更有意思的是 stars 对比:`anthropics/skills` 127k,`awesome-codex-skills` 6k。**21 倍的差距。** Anthropic 在 2025 年 10 月 16 日上线 Skill,OpenAI 这边在 2026 年 4 月才被社区大规模搬运推广,中间差了半年。其中 Anthropic 官方的 frontend-design Skill 截至今年 3 月已经被安装了 27.7 万次。

做过 10 年后端的人对这个剧本应该很熟——这就是当年 Java EE 的 EJB vs Spring,Servlet vs Spring MVC,JPA vs MyBatis 那个故事的 AI 版本:**先有人定义事实标准,然后所有人跟进,格式趋同,最后赢家是开发者拿到了一个跨平台的能力包。**

那如果你现在还在观望「Skill 是不是 Anthropic 的私有玩具」,可以放心了——它已经在变成 AI 编程工具的事实标准。

## 我写过的几十个 Skill,前面一半都错在哪里

回头看那 30 个被我删掉的 Skill,问题不是技术不行,是设计哲学错了。这一节列 7 个最常见的反模式,**几乎每个都至少踩过一次,有些到现在还在踩。**

### 反模式 1:把 SKILL.md 写成长篇 prompt

第一个 Skill 我写了 1200 行,里面塞满了「请按以下规范输出」「禁止使用以下表达」「示例如下:...」。结果 Claude 加载后表现比裸调用还差。

后来才搞明白:**SKILL.md 越长,触发越不准。** Claude 通过 description 字段决定是否激活这个 Skill,正文只在激活后才读。你把激活条件淹没在 1200 行细则里,模型自己都迷糊该不该用。

官方建议是 SKILL.md 主体控制在 500 行以内,超出的拆到 references/ 子目录。我现在的所有 Skill,主文件不超过 200 行。

### 反模式 2:description 写成功能介绍而不是触发条件

错误写法:「这个 Skill 提供了完整的 Spring Boot 项目代码生成能力,支持多种数据库适配...」

正确写法:「在用户要求生成 Spring Boot 接口/Controller/Service 代码,或提到『新建一个 REST 接口』『加一个查询 API』时使用」

差别在哪里? **第一种描述告诉 Claude「我能做什么」,第二种告诉 Claude「什么时候轮到我做」。** 模型决定触发的逻辑是后者,不是前者。这是文档里写得不那么显眼但极度关键的一点。

### 反模式 3:把工具调用写进 Skill,而不是用 MCP

我有一个 Skill 是「调用公司内部 API 查订单状态」,里面写了一堆「请按以下格式 curl」「然后 jq 解析返回值」。运行起来时不时翻车,因为模型对长 URL 和复杂 jq 表达式的处理能力不稳定。

后来想通了:**Skill 是任务能力包,不是工具调用器。** 工具该用 MCP 暴露,Skill 只负责「什么时候用这个工具、怎么组合多个工具完成任务」。把这两件事混在一起,等于让一个程序员既写业务代码又写 SDK,结果两边都做不好。

### 反模式 4:多个 Skill 边界互相重叠

我曾经同时写过 `code-review`、`pr-review`、`merge-checklist` 三个 Skill,职责高度重叠。Claude 在选哪个时摇摆,有时三个都触发,输出混乱。

设计 Skill 和拆微服务是一个道理——**职责单一,边界清晰**。如果两个 Skill 在 description 里出现了相似关键词,合并它们或重新切分。

### 反模式 5:不写 Gotchas 章节

Skill 主体只写「应该怎么做」,不写「容易踩什么坑」,Claude 就会反复犯同样的错。

那篇 Anthropic 工程师博客里有句原话我一直记到现在:「Skill 里最有价值的内容是 Gotchas 章节——基于 Claude 在使用过程中真实犯过的错。」

我现在的工作流是:每次发现 Claude 在某个 Skill 下犯了一次傻,就把这次的错误模式追加到 Gotchas 里。Skill 是活的,不是写完就算了。

### 反模式 6:示例用伪代码

写示例图省事用了 `// ... your business logic`,结果 Claude 真的就在这一行旁边写出 `// ... your business logic`,一字不差。

**模型是镜子。** 你给它的示例是什么样,它的输出大概率就是什么样。示例必须是完整可运行的代码,包括 import、main、错误处理,所有占位符都得是真实代码。

### 反模式 7:不区分项目级和用户级 Skill

公司的代码风格规范该放在项目级(`.claude/skills/` 或 `.agents/skills/` 在 repo 根目录),个人写作偏好该放在用户级(`~/.claude/skills/`)。混在一起的后果是:你换个项目就把一堆完全不相关的代码风格强行带过去。

![Claude Code Skill 加载与触发流程：description 决定触发，正文不影响](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/skill-loading-mechanism.png)

## 一个高质量 Skill 长什么样:拆解 frontend-design

讲完反模式,看正面案例最快。Anthropic 官方的 `frontend-design` Skill 装机量 27 万,值得拆开看它做对了什么。

打开 `~/.claude/skills/frontend-design/SKILL.md`,你会看到:

```yaml
---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high
  design quality. Use this skill when the user asks to build web components,
  pages, artifacts, posters, or applications (examples include websites,
  landing pages, dashboards, React components, HTML/CSS layouts, or when
  styling/beautifying any web UI). Generates creative, polished code and UI
  design that avoids generic AI aesthetics.
---
```

注意三个细节:

**第一,description 用了「Use this skill when」明确触发条件,后面跟 examples 列出 5-6 个具体场景。** 这一段不是营销文案,是给模型读的「触发规则」。

**第二,主文件不到 300 行,大量内容拆到 references/ 下的子文件。** 如颜色系统在 `references/color-systems.md`,字体规则在 `references/typography.md`,Claude 只在需要时按需读取。这种结构叫渐进式披露(Progressive Disclosure)——主文件给 overview,子文件给细节。

**第三,有完整的 Anti-patterns 章节,直接列「不要做什么」。** 比如「不要用千篇一律的紫色渐变」「不要在 hero 里堆 emoji」「不要用 Tailwind 默认配色凑合」。这些都是 Anthropic 的设计师在反复迭代中发现 Claude 容易犯的错,被一条条记下来。

这是真正写过几十个 Skill 的人才会有的设计思路——**主文件像架构概要,子文件像 API 文档,Gotchas 像 SRE 的 runbook。** 三层分得清清楚楚,而不是一锅烩。

## 我每天都在用的 5 个 Skill(附完整模板)

下面这 5 个是我目前真正每天都在用、删了会难受的 Skill。直接给完整模板,你可以拷贝到 `~/.claude/skills/{name}/SKILL.md` 改改就用。

**1. `commit-msg-zh`** — 写中文 commit message,严格按 Conventional Commits

```yaml
---
name: commit-msg-zh
description: When the user asks to write a git commit message, generate a commit
  using Conventional Commits spec, but body in Chinese. Trigger on phrases like
  "写个 commit"、"提交信息"、"写一下 commit message".
---

# Commit Message Generator

格式:
- type(scope): 简短描述(英文,小写,< 50 字符)
- 空一行
- 中文 body 解释「为什么改」,不解释「改了什么」(diff 已经说明了)

允许的 type:feat / fix / refactor / docs / test / chore / perf

## Gotchas
- 永远不要写「修改了 xxx 文件」(无信息量)
- 永远不要在 body 里复述 diff
- 不要用「优化」「调整」这种空洞动词,要说出动机
```

**2. `spring-controller-skeleton`** — 生成符合公司规范的 Controller

放在 repo 根目录的 `.claude/skills/spring-controller-skeleton/SKILL.md`,这是项目级 Skill。description 里写清楚触发条件:「用户要求新增 REST 接口、HTTP 接口、Controller 时使用」。主体内容是公司规范——比如统一返回 `Result<T>`、参数校验用 `@Validated`、异常通过 `@ControllerAdvice` 统一处理等。

**3. `redis-key-naming`** — Redis key 命名规则

description:「在涉及 Redis key 设计、setEx、Hash、ZSet 操作时使用」。主体只写命名规则:`{业务域}:{实体}:{ID}` 三段式,带 TTL 必须写注释,严禁出现单字符前缀。

**4. `mysql-explain-reviewer`** — 自动 review SQL 的执行计划

description 写清楚触发条件「在用户贴 EXPLAIN 输出、贴 SQL 询问性能、提到慢查询时使用」。主体是一个 checklist:type 列必须 ≥ ref、Extra 不能出现 Using filesort/Using temporary、rows 列与表大小的比例阈值等。这个 Skill 救了我至少 3 次线上慢查询排查时间。

**5. `tech-blog-zh`** — 把英文文档转成符合「码哥腔调」的中文技术博文

这是给我自己写公众号用的。description:「在用户贴英文技术文档、英文博客并要求改写、转写为中文公众号文章时使用」。主体是写作规范——开头用场景/反直觉/疑问、禁用 AI 套话表、武侠类比指南、码哥腔调示例。这个 Skill 是元 Skill——它在驱动你正在读的这篇文章的写作。

![5 个每天都在用的 Claude Code Skill 协作图：commit-msg-zh、spring-controller-skeleton、redis-key-naming、mysql-explain-reviewer、tech-blog-zh](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/five-skills-workflow.png)

## Codex 和 Claude Code 的 Skill 能不能复用?我做了一个实验

既然两边 SKILL.md frontmatter 几乎一样,我抱着「能不能一鱼两吃」的想法做了个实验。

把 `commit-msg-zh` 这个 Skill 同时软链到两个目录:

```bash
# Claude Code
ln -s ~/skills-shared/commit-msg-zh ~/.claude/skills/commit-msg-zh

# OpenAI Codex
ln -s ~/skills-shared/commit-msg-zh ~/.codex/skills/commit-msg-zh
# 或者放在 ~/.agents/skills/(Codex 也会扫这个目录)
```

测试结果:**90% 兼容,10% 翻车。**

兼容的 90%:绝大部分纯文本指令、规则、checklist 类的 Skill,两边表现几乎一样。`commit-msg-zh`、`redis-key-naming` 这种纯规约型 Skill 完美复用。

翻车的 10%:**涉及工具调用和文件路径的 Skill 不能直接复用。** Codex 和 Claude 对工具的命名、路径解析、子进程隔离机制完全不同。比如 Skill 里写 `请使用 Read 工具读取文件`,Codex 这边是 `read_file`,触发名对不上。

**实操结论:** 把 Skill 拆成「pure 指令型」和「带工具调用型」两类。前者通用,后者各写一份。我现在的 Skill 库里大概 60% 是 pure 型,可以一次写两边用。

## Skill vs MCP,到底有什么区别?

这是被问最多的问题之一,简单说清楚。

![Skill / MCP / Agent 三层抽象关系图：Agent 像 Spring Boot Application，Skill 像 Starter，MCP 像 SPI / DI 容器](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/skill-mcp-agent-layers.png)

用 Spring 生态做类比一秒讲明白:

- **MCP 像 Spring SPI / 依赖注入**——它解决「我有什么工具能用」的问题,是横向的能力发现协议。
- **Skill 像 Spring Boot Starter**——它解决「在这个场景下,我应该用哪些工具、按什么顺序、按什么规则」,是纵向的任务能力包。
- **Agent 像 Spring Boot Application**——它是装着 Starter 和 SPI 的运行体,负责接收请求、执行决策。

混淆这三层的常见后果:把 MCP 写得像 Skill(在工具描述里塞业务规则),或者把 Skill 写得像 MCP(在 SKILL.md 里实现工具调用)。**两边都做不好,而且一改就崩。**

判断标准很简单:你写的东西是给「能力」做注册的,还是给「场景」做编排的?前者放 MCP,后者放 Skill。

## 常见问题

**Q:Skill 写完后 Claude 不触发,怎么排查?**

A:90% 是 description 写错了。打开你的 Skill,把 description 当成搜索引擎的关键词去想——用户说什么话的时候这个 Skill 应该匹配?把这些话作为 examples 写进去。description 越接近用户实际说话的方式,触发率越高。

**Q:同一个 Skill,放在 `~/.claude/skills/` 和项目 `.claude/skills/` 有什么区别?**

A:项目级优先于用户级。冲突时项目级覆盖。建议把代码风格、API 规范放项目级(进 git 仓库,团队共享),个人偏好放用户级。

**Q:Skill 数量上限是多少?会不会装太多影响 Claude 启动速度?**

A:没有硬上限,但 Claude 启动时只读 frontmatter,所以装 100 个 Skill 启动开销也很小。真正的 cost 在 description——所有 description 加起来会进上下文,所以 description 要精炼,不要写营销文案。

**Q:能不能用 Claude 自己来生成 Skill?**

A:能,而且强烈推荐。Anthropic 官方提供了 `skill-creator` Skill,跑 `/skill-creator` 就会引导你写出一个合格的 Skill。我现在写新 Skill 都先用它,然后人工调 description 和 Gotchas。

**Q:OpenAI Codex 也支持 Skill 了,我应该投资哪边?**

A:都投资,但优先 Anthropic。原因是生态体量、官方 Skill 质量、社区活跃度都高一个数量级。Codex Skill 当作低成本扩展——只要你的 pure 指令型 Skill 写得规范,加个软链就能两边用,边际成本接近零。

## 我的判断

写 Skill 这一年,最深的感受是:**它正在重新定义「会用 AI 工具」的门槛。**

去年大家比的是 prompt 写得好不好,今年开始比的是 Skill 库丰不丰富、设计得对不对。明年大概率会出现「公司内部 Skill 库」「行业 Skill 标准」这一层抽象——就像今天每家后端团队都有自己的 Maven 私服、内部 SDK 一样。

如果你还在用裸 prompt 跟 Claude Code 一来一回地拉锯,某种程度上就像还在用 ant 编译 Java 项目。能跑,但下个台阶你已经迟到了。

下一篇我打算拆 `frontend-design` 这个 27 万装机量的 Skill 是怎么写出来的——把它的 references/ 目录里那些没人讲过的设计指南一条条翻译过来,关注一下,发布了会第一时间推送。

如果你身边有人在团队里推 Claude Code 但还没把 Skill 当回事,这篇可以直接甩给他,省得他重走我那 30 个 Skill 的弯路。

## 参考资料

- [anthropics/skills - 官方 Skill 仓库](https://github.com/anthropics/skills)
- [ComposioHQ/awesome-codex-skills - OpenAI Codex Skill 集合](https://github.com/ComposioHQ/awesome-codex-skills)
- [Claude Skills 官方公告(2025-10-16)](https://claude.com/blog/skills)
- [Skill authoring best practices - Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Codex Agent Skills - OpenAI Developers](https://developers.openai.com/codex/skills)
