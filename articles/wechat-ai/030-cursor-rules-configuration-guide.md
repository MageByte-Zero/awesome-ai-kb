---
title: "你的 Cursor 还在靠缘分猜你的代码规范？Rules 配置完，AI 像换了个人"
alt_titles:
  - "AI 老是改错我的 Spring Boot 代码，配了这 3 个文件之后全好了"
  - "Cursor 用了 3 个月才搞懂：.mdc 文件才是让 AI 真正懂你项目的关键"
  - "让 Cursor 读懂 Spring Boot 项目，后端工程师的 Rules 配置实战"
  - "Cursor Rules 配错了？4 种激活类型没搞懂，AI 帮倒忙的根源就在这"
description: "Cursor Rules 是让 AI 真正理解你项目的核心配置，本文从后端工程师视角出发，手把手教你配置 .mdc 文件、掌握 4 种激活类型、分模块管理规则，并提供完整的 Java Spring Boot 和 Go 项目实战模板。"
date: "2026-06-06"
keywords: ["Cursor Rules", "Cursor 配置", ".mdc 文件", "Spring Boot AI 编程", "Go 项目 Cursor", "AI 编程规范", "Cursor 使用技巧"]
platform: "微信公众号"
source: "原创"
cover: "https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/030-cursor-rules-configuration-guide-cover.png"
---

![封面图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/030-cursor-rules-configuration-guide-cover.png)

# 你的 Cursor 还在靠缘分猜你的代码规范？Rules 配置完，AI 像换了个人

我有一个后端同事，用 Cursor 三个月了，每次让 AI 生成代码都要花半天改格式。

他的 Spring Boot 项目用的是统一异常处理 + 自定义 `Result<T>` 返回体，但 Cursor 每次都给他生成 `ResponseEntity<Object>`，错误处理也是原始的 try-catch 塞了一堆 `e.printStackTrace()`。他说：「这玩意儿就是个代码补全，根本不懂我们项目的规范。」

这个判断只对了一半。Cursor 不懂你的项目，不是 Cursor 的问题，是你没告诉它。

**告诉它的方式，叫 Cursor Rules。**

这篇文章不讲概念，直接给你后端工程师能开箱即用的配置方案。Java Spring Boot 和 Go 项目各一套完整模板，跟着配，AI 帮倒忙的问题基本能解决 80%。

## 旧的方式已经在走下坡路

很多教程还在讲 `.cursorrules` 文件。这个文件放在项目根目录，全局生效，写一堆规则进去。

**问题在于它是一个单文件，不能按场景激活，不能分模块管理。** 你的项目有 Java 后端、有测试、有数据库迁移脚本，这三块的规范完全不同。一个文件全塞进去，既超长又难维护，AI 读取的时候也容易被稀释。

Cursor 的新系统是 `.cursor/rules/` 目录 + `.mdc` 文件，每个 `.mdc` 文件对应一个规则模块，支持 4 种激活类型。`.cursorrules` 还没有强制废弃，但 Cursor 官方已经在引导用户迁移，新功能只加在 `.mdc` 体系上。

现在是切换的好时机。

![Cursor Rules 目录结构与 4 种激活类型关系图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/030-cursor-rules-configuration-guide-fig01-arch.png)
*图：`.cursor/rules/` 目录结构与 4 种激活类型的对应关系*

## 4 种激活类型，用错了比没配还糟

这里有个大多数人绕不过去的坑：不是配了 Rules 就有效，是配对了激活类型才有效。

Cursor 的 `.mdc` 文件支持 4 种激活方式，官方界面里叫 `Rule Type`：

### Always（全局生效）

打开任何文件都加载，适合全项目通用的强制规范。

**适合放什么：** 禁用 `System.out.println`、统一返回体格式、Git commit 规范、所有文件的 package 命名惯例。

**注意：** 别把太多规则塞进 Always，它每次都消耗 context 窗口。超过 50 行，建议拆出去。

### Auto Attached（文件匹配自动触发）

用 glob 模式匹配文件路径，只有打开匹配的文件时才加载。这是用得最多的类型。

**配置示例：**

```
**/*.java     → 触发 Java 规范
**/*.go       → 触发 Go 规范
**/test/**    → 触发测试规范
**/migration/**  → 触发数据库迁移规范
```

**适合放什么：** 各技术栈的语言规范、框架用法惯例、文件结构规定。

### Agent Requested（AI 自主决定）

AI 根据当前上下文判断要不要加载这条规则，你给这个规则写一个 description，AI 读 description 决定是否引入。

**适合放什么：** 可选的、特定场景才需要的规则，比如「使用 Reactor 响应式编程时的规范」、「接入第三方 SDK 时的注意事项」。

### Manual（手动 @引用）

只有你显式在 chat 里 @这个规则文件，AI 才加载。

**适合放什么：** 性能优化检查清单、安全审计要点、Code Review 标准——这些不是写代码时常用的，是特定任务才需要的。

**推荐配比：80% Auto Attached + 10% Always + 10% Manual。**

Agent Requested 初看很智能，但实测下来，AI 的判断不稳定，有时候该用的没加载，不该用的加载了。生产项目里尽量用 Auto Attached，明确匹配，不靠推理。

## 动手：从创建第一个 .mdc 文件开始

先建好目录结构。在项目根目录：

```bash
mkdir -p .cursor/rules
```

**创建方式一（GUI）：** Cmd+Shift+P（Mac）或 Ctrl+Shift+P（Windows），搜索 `New Cursor Rule`，填入名称和类型，Cursor 会自动在 `.cursor/rules/` 下创建文件。

**创建方式二（CLI）：** 在 Cursor 的 chat 里输入 `/rules`（2026-01-08 新增的命令），可以直接在对话中新建和编辑规则。

`.mdc` 文件的结构很简单：

```markdown
---
description: 这条规则的说明（Agent Requested 类型时 AI 靠这个决定是否加载）
globs: **/*.java
alwaysApply: false
---

# 规则标题

规则内容，用自然语言写，AI 能读懂即可。
可以加代码示例，可以加反例。
```

`alwaysApply: true` 等同于 Always 类型；`globs` 有值且 `alwaysApply: false` 等同于 Auto Attached；两个都空 + 有 description 等同于 Agent Requested；手动创建但不写 globs 就是 Manual。

## Java Spring Boot 项目实战模板

以下是码哥在实际 Spring Boot 3.x 项目里用的规则，拿走即用，按你的项目实际情况改。

### 核心规范文件：java-spring.mdc

```markdown
---
description: Java Spring Boot 核心编码规范
globs: "**/*.java"
alwaysApply: false
---

# Java Spring Boot 3.x 编码规范

## 技术栈版本
- Java 21，使用 Virtual Threads
- Spring Boot 3.3.x
- Maven 依赖管理

## 禁止行为
- 禁止使用 System.out.println，统一用 @Slf4j 的 log.info/warn/error
- 禁止直接 throws Exception，必须抛出具体业务异常
- 禁止在 Service 层返回 ResponseEntity，那是 Controller 层的事
- 禁止修改 pom.xml 中的 Spring Boot Parent 版本，除非明确指示
- 禁止生成 // TODO 占位注释，要么给完整实现，要么不生成
- 禁止使用已标记 @Deprecated 的 API

## 统一返回体
所有接口返回 Result<T>，不用 ResponseEntity<Object>：
- 成功：Result.success(data)
- 失败：Result.fail(ErrorCode.XXX)

## 异常处理
- Service 层抛出继承 BaseException 的业务异常
- 统一由 @RestControllerAdvice 的 GlobalExceptionHandler 处理
- 不在 Controller 层写 try-catch

## 包结构（严格遵守）
- controller：只处理请求参数校验和返回体封装
- service / serviceImpl：业务逻辑
- repository：数据访问，只用 JPA Repository 接口
- domain/entity：数据库实体，用 @Entity 注解
- dto：接口入参 DTO，用 @Validated 做参数校验
- vo：接口出参 VO

## 命名规范
- 类名 PascalCase，方法/字段 camelCase
- 接口方法动词开头：get/create/update/delete/query
- 常量 UPPER_SNAKE_CASE，全部放 constant 包下的 Constants 类

## RESTful 风格
- GET 查询，POST 创建，PUT 全量更新，PATCH 部分更新，DELETE 删除
- 路径用小写连字符：/api/v1/user-orders，不用下划线
```

### 测试规范文件：testing.mdc

```markdown
---
description: 单元测试和集成测试规范
globs: "**/test/**/*.java"
alwaysApply: false
---

# 测试编写规范

## 框架
- 单元测试：JUnit 5 + Mockito
- 集成测试：Spring Boot Test + Testcontainers（MySQL/Redis）

## 命名
- 方法名：given_当前状态_when_执行操作_then_期望结果
- 例：given_userExists_when_getUserById_then_returnUserVO

## 强制要求
- Service 层方法必须有对应单元测试
- 测试方法不得共享状态，每个测试独立可运行
- Mock 外部 HTTP 调用，不发真实网络请求
- 禁止 @Autowired 注入，用构造函数注入方便 Mock
```

**使用后实际效果：** 配置前，Cursor 生成的代码里 `System.out.println` 出现频率约 30%，异常处理用原始 try-catch 的比例超 60%。配置后，这两个问题基本消失，需要手动干预的代码量大概少了 40-50%。这是在一个 3 人团队的实际项目里统计的，样本不大，但方向是对的。

## Go 项目实战模板

Go 的规则比 Java 简单，但有几个地方 AI 特别容易犯错。

### 核心规范文件：go-project.mdc

```markdown
---
description: Go 项目编码规范
globs: "**/*.go"
alwaysApply: false
---

# Go 项目编码规范

## 技术栈版本
- Go 1.22+
- 使用 Go Modules，go.mod 中的依赖版本禁止擅自修改
- HTTP 框架：Gin（如项目使用 Fiber，见 go-fiber.mdc）

## 禁止行为
- 禁止用 _ 忽略 error 返回值，必须显式处理
- 禁止在 goroutine 里直接 panic，用 recover 包裹后记录日志
- 禁止使用 interface{} 作为函数参数或返回值，用具体类型或泛型
- 禁止在循环里创建 goroutine 但不用 WaitGroup 或 channel 控制
- 禁止修改 go.mod 文件，除非明确指示添加新依赖

## 代码风格
- 变量名短小：接收者用 1-2 字母缩写（如 u *User → u）
- 包名全小写，单词：userservice 不是 user_service
- 错误变量命名：err，不是 e 或 error
- 接口以行为命名：Reader、Writer、Closer，而不是 IReader

## 错误处理
- 优先使用 fmt.Errorf("操作失败: %w", err) 包装错误
- 业务错误返回 error 类型，不用 panic
- 自定义错误类型实现 error 接口

## 项目结构（按需选择）
- cmd/：程序入口，main.go 保持瘦，只做初始化
- internal/：业务逻辑，不对外暴露
- pkg/：可复用的公共包
- api/：接口定义（protobuf 或 OpenAPI）

## Gin 规范
- 中间件注册在 router.go
- Handler 只做参数绑定和返回，业务逻辑下沉到 service
- 绑定参数用 ShouldBindJSON，不用 BindJSON（后者会直接写响应头）
```

## 让 AI 帮你生成 Rules——这个技巧大多数人没用过

配置 Rules 不一定要从零写。

你可以直接在 Cursor chat 里这样说：

```
分析我的项目结构和现有代码，帮我生成一份适合这个项目的 Cursor Rules 文件。
关注这几点：
1. 当前代码使用的技术栈和版本
2. 命名规范（从现有文件名推断）
3. 错误处理模式（从现有 service 层推断）
4. 我不想让 AI 改动的配置文件

输出格式：.mdc 文件内容，Auto Attached 类型，glob 匹配 *.java
```

AI 会读取你当前打开的文件，结合项目上下文生成一份初版 Rules。不会完全准确，但能省掉 60-70% 的初始配置时间，剩下的手工修正即可。

这个方法特别适合接手老项目：让 AI 先分析现有代码风格，自动总结出「这个项目的潜规则」，写成 Rules 文件，从此新人和 AI 都按同一套标准走。

## 分模块管理：超过 500 行就拆

单个 `.mdc` 文件官方建议不超过 500 行。超出这个限制，AI 读取效率会下降，而且一个大文件也不好维护。

**推荐的拆分结构：**

```
.cursor/rules/
├── always-global.mdc      # Always 类型，全项目基础规范（< 30 行）
├── java-spring.mdc        # Java/Spring Boot 规范
├── go-project.mdc         # Go 规范
├── testing.mdc            # 测试规范
├── sql-migration.mdc      # 数据库迁移规范（Flyway/Liquibase）
├── security-review.mdc    # 安全审计检查清单（Manual 类型）
└── performance-opt.mdc    # 性能优化要点（Manual 类型）
```

`.mdc` 文件还支持 `@file` 语法引用其他规则文件，实现规则链：

```markdown
# 在 java-spring.mdc 里可以引用
@file .cursor/rules/testing.mdc

当生成 Service 层代码时，同时参考测试规范。
```

**团队协作重点：** Rules 文件要提交到 Git。在 `README.md` 里加一行说明 Rules 文件的位置和更新方式，新人克隆项目后直接就有完整的 AI 规范配置。这是把团队经验固化进版本控制的方式，比口口相传靠谱得多。

![Cursor 配置前后效果对比流程图](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/030-cursor-rules-configuration-guide-fig02-flow.png)
*图：从「无 Rules → AI 随机生成」到「有 Rules → AI 遵守项目规范」的对比*

## 踩坑记录：这几个问题我见过不止一次

**坑一：glob 路径写错，规则从不触发**

Auto Attached 类型最常见的问题。`globs: *.java` 只匹配根目录的 Java 文件，项目里绝大多数文件在子目录下，需要写成 `**/*.java`。

另一个常见错误：路径用了绝对路径，比如 `/src/main/java/**/*.java`。Cursor 的 glob 是相对项目根目录的，直接写 `src/main/java/**/*.java` 就行，前面不加斜杠。

**坑二：Rules 写得太长，AI 只用了前半段**

我自己犯过这个错：把所有规范塞进一个文件，洋洋洒洒 800 行。AI context 窗口有限，后面的内容事实上没被重视。按功能拆分，每个文件只讲一件事，是解法。

**坑三：Always 类型用太多，性能下降**

Always 规则每次请求都消耗 context，配了 10 个 Always 规则之后感觉 Cursor 变慢了，回答质量也下降了。把 70% 的 Always 改成 Auto Attached 之后，明显改善。

**坑四：忘了把 .cursor 目录加进 .gitignore 的例外**

一些项目的 `.gitignore` 里有 `.cursor/`，导致 Rules 文件没提交到仓库。检查一下：

```bash
git check-ignore -v .cursor/rules/java-spring.mdc
```

如果有输出，说明被忽略了，在 `.gitignore` 里加：

```
!.cursor/rules/
```

## 常见问题

**Q：项目已经用了 `.cursorrules`，要立刻迁移吗？**

不用立刻迁移，`.cursorrules` 目前还能用。但建议新写的规则都放到 `.cursor/rules/` 下，旧文件慢慢迁移。两套系统可以同时存在，Cursor 会合并加载。

**Q：Rules 文件里能写代码吗？**

可以。放一个「正确示例 vs 错误示例」的对比，AI 学起来比纯文字规范快得多。示例 20-30 行够了，太长反而分散注意力。

**Q：规则之间冲突了怎么办？**

Cursor 会把所有加载的 Rules 合并进 system prompt，冲突时 AI 按最后出现的规则优先。最简单的解法是给规则加优先级说明：「如果以下规则与其他规则冲突，以本文件为准。」

**Q：团队不同成员有不同的编码习惯，Rules 怎么统一？**

先在团队内讨论出一份「最小公约数」规范，只写所有人都认可的强制项，争议内容先不写进 Rules。把 Rules review 纳入 Code Review 流程，Rules 文件变更需要全员审核。

**Q：.mdc 文件的 description 字段有字数限制吗？**

没有硬性限制，但建议 50 字以内。description 是 Agent Requested 类型时 AI 判断是否加载的依据，写得越精准，AI 的判断越准确。太长的描述反而会干扰判断。

## 参考资料

- [Cursor Rules 官方文档](https://docs.cursor.com/context/rules)
- [Cursor Changelog - Rules CLI 命令（2026-01-08）](https://changelog.cursor.com)
- [Awesome CursorRules - 社区规则模板集合](https://github.com/PatrickJS/awesome-cursorrules)

说白了，Cursor Rules 不是什么高深技术，就是把你团队口耳相传的「不成文规定」写成 AI 能读懂的文档。Java 项目禁止 `var`、Go 项目必须处理 error、REST 接口统一返回 `Result<T>`——这些规矩本来就存在，只是以前新人要踩一遍才知道，现在把它写进 `.mdc`，AI 和新人都不用再踩了。

下篇打算聊「Cursor 的 Memory 功能和 Rules 怎么配合用」，感兴趣的关注一下，发了第一时间推给你。身边有同事也在头疼 Cursor 老是生成不符合规范的代码，可以把这篇甩给他，省他自己摸索。

回复「prompt」获取本文完整 Cursor Rules 模板合集（含 Spring Boot / Go / 测试 / 安全审计 4 套）。

![.cursorrules 与新系统对比、4 种激活类型对比表](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/030-cursor-rules-configuration-guide-fig03-cmp.png)
*图：旧系统 vs 新系统对比，4 种激活类型使用场景速查*
