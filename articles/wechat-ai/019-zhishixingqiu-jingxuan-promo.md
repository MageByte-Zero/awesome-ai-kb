---
title: "AI 用的不好，关键在于你的工作流打造的不够强"
alt_titles:
  - "Redis畅销书作者做的AI成长星球，到底有什么"
  - "一个后端架构师的星球：AI编程、Redis、面试、普通人提效，四条线"
  - "程序员和不会代码的人，都能从这个星球拿走东西"
  - "我把过去两年踩过的坑，整理成了四套内容放进星球"
description: "码哥（《Redis高手心法》作者、InfoQ签约作者、腾讯云架构师同盟专家）知识星球「码哥的AI成长进化营」介绍：AI 编程实战 41 讲、AI 超能工具箱 31 讲（非程序员向）、Redis 高手心法电子版、后端面试心法 58 讲，四条主线，长期更新。"
date: "2026-05-17"
keywords: ["Claude Code", "AI编程", "Redis", "后端工程师", "知识星球", "AI工具", "非程序员", "工作流"]
platform: "微信公众号"
source: "知识星球精选推广"
cover: "https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/019-zhishixingqiu-jingxuan-promo-cover.png"
---

![封面](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/019-zhishixingqiu-jingxuan-promo-cover.png)

《Redis 高手心法》出版以后，我陆续收到两类很不一样的私信。

一类来自做后端的工程师：「码哥，Redis 那本写得很有价值，AI 编程这块你为什么不搞一套？现在这方面内容太乱了，看了一堆技巧，拿回项目里用，根本形不成体系。」

另一类来自做产品、运营和内容的朋友：「我也想用 AI 工具提效，但那些教程都在讲「配置环境」「写代码」，完全不知道该从哪里下手，打开一看就关掉了。」

两类问题，都指向同一件事：AI 工具的教程多，但真正适合你的很少。工程师要的是有工程背景的人讲完整工作流，不是「神级技巧 10 条」；非程序员要的是能直接上手、不用写代码的路径，不是上来就让你配终端环境。

所以我决定认真做这件事，做一个长期更新的成长空间：**「码哥的AI成长进化营」**。

在说星球内容之前，我先说清楚我是谁，你才能判断这里的内容值不值你的时间。

我是《Redis 高手心法》的作者，在电子工业出版社出版，目前是畅销级别。我是 InfoQ 的签约作者，在上面写了几年系统设计和后端架构。我在一家日流水亿级的国际互联网公司做后端架构，用了整整一年把 AI 工具真正跑通了自己的工作流。同时也是腾讯云架构师同盟名人堂专家。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/EoJib2tNvVtdZYMBEQ8S0fa57mId5IT6JvoUl4yrkGibR6ERarGcdoibicKEPTSlnLbic44iaCOCy0iaJwyGiak3jG958A/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=0)

这些背书我平时不专门拿出来说，但做这个星球必须说清楚：**这里的内容来自一个在大型后端系统工作、拿 AI 工具真实改造过开发流程的人——不是看了两周 Cursor 就来教课的，是踩过坑整理出来的体系。**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sgicmTR46ZvuupK3TZfiaRgSbiakIPNGicibN20682Uv3oIDV0NDpdCWjCrdomxB6n5I3fiaZlcajMgK50kZaBovBIRZUgtBOL5y1RkQicI4pcsc3Y/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

同时，我还邀请了两位重磅大佬作为嘉宾加入我的星球：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/sgicmTR46ZvvFyApVAK3H9SgXqD3HsAckFBfMc8oKr0fcnekwGvH4F9Q34DStCia3WwAibqMp7pNZFf2eicV0vuq0HfLdgTzaibjgGfXly4ricmb0/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=1)

如果你读了这篇觉得「这不就是在卖课吗」，完全理解。但如果你觉得「这个背景的人来讲 AI 实战，角度可能跟别人不一样」——那可以继续看下去。



## 星球里有什么

目前四条主线，覆盖两类读者。

![四条主线一览](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/019-zhishixingqiu-jingxuan-promo-fig01-four-tracks.png)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/sgicmTR46Zvup69A6XtbGg3aJU6q7jLjPOUZ3ftUUsFr1g6JE2kMxmCFefZCBUsu8V3h9hHepvC0Cl25rx7MfAGxY5NY9eQ3ic30X6todvDyA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

### 第一条：《AI 编程实战：用 Claude Code 重塑你的开发工作流》41 讲

**面向后端工程师和架构师。**

AI 编程的内容铺天盖地，但真正回答「AI 怎么进入我的日常开发流程」的，几乎没有。知道 Cursor 能补代码，但不知道什么时候该让它改、什么时候该自己改。听过 Claude Code 能读文件、跑测试，但不知道它和 ChatGPT 的本质区别是什么。知道 MCP 很火，但不知道它到底是「插件市场」还是「AI 时代的工具调用接口」。

这 41 讲分 6 个模块，从认知、工具上手，一直讲到 MCP、Skill，再到企业落地和架构师视角。

重点说几个大多数教程不讲的地方：

**MCP 和 Skill 是目前价值密度最高的方向。** 会用工具的人很多，会把工具改造成自己工作流的人很少。MCP 让 AI 能接你的 GitHub、数据库、日志系统；Skill 让你把个人经验沉淀成可复用的流程包——你处理某类任务的最佳实践，可以变成 Skill，以后一键调用，也可以在团队里分发。这两块是真正拉开差距的地方。

**国内可用方案专门讲。** 官方 Claude API、网络、模型价格，这些问题不解决，很多教程停留在「看起来很美」。我专门安排了一讲讲 cc-switch 接 DeepSeek V4：哪些任务可以用国产模型跑，哪些任务还是建议上 Claude，怎么在成本和效果之间做取舍。

**企业落地和团队化，不只讲个人爽感。** 接企业日志、连数据库、做脱敏、搭 RAG 知识库、管成本、做监控、推动团队使用——这些才是 AI 编程从个人玩具变成团队能力的关键。如果你是团队 leader 或架构师，这部分应该最有价值。

全专栏用一个贯穿案例串起来：Spring Boot 订单超时关单服务。从最初实现，到重构、补单测、接入 AI 辅助、企业级集成，同一个项目在不同模块反复出现，看完知道怎么落地，不是看完一堆概念不知道怎么用。

---

### 第二条：《AI 超能工具箱：普通人也能玩转 Skill、Plugin 与 MCP》31 讲

**面向不会写代码的产品、运营、内容创作者、销售和创业者。**

AI 工具的教程几乎全是程序员写给程序员看的。「配置终端环境」「用 Agent 帮你写测试」「让 AI 替你 Code Review」——这些对不写代码的人来说是另一个星球的语言。

结果是：大多数人用 AI，只发挥了 10% 的潜力。打开了，问几个问题，觉得「跟 ChatGPT 差不多」，然后关掉了。

这 31 讲专门为你准备，没有一讲要求你写代码。

**AI 工具能做的远不止「问答」。** 有个叫 claude-mem 的工具，可以让 AI 永远记住你——你的职业背景、写作风格、习惯用语，存一次，以后每次直接上。有个叫 chrome-devtools-mcp 的工具，可以让 AI 帮你打开网页、点按钮、填表单——那些每周要重复做 20 遍的无聊操作。用 skill-creator，可以把你自己总结的工作方法「装进」AI，让它以后每次都按你的方式干活。

**周报、提案、会议纪要、数据报告，AI 全程跑。** 不是「帮你写一段」，而是从整理素材、起草、匹配你的口吻，到最终产出可用的内容。

**飞书用户专门讲。** 会议记录整理成纪要、OKR 进度同步、审批催办、日历提醒，全部交给 AI 处理。

我自己用这套工具探索了整整一年。身边做运营的朋友，按着类似的路径走了 6 个月，现在接的副业单子比全职工资还高。她不会写代码，用的就是这类工具。

### 第三条：《Redis 高手心法》完整电子版

**面向后端工程师，也适合备战面试的开发者。**

这是我在电子工业出版社出版的畅销书，实体版定价 ¥100。

星球里放的是重新优化后的版本：原书受印刷工艺限制，很多原理图只能黑白表现，彩色版本被砍掉了。电子版把这些彩色图表全部加回来。买过实体书的读者也值得再读一遍——图的密度和清晰度差很多。

大多数后端工程师都用过 Redis，但真到面试或者生产排障，很多人只会说 set/get、缓存穿透、分布式锁。再往下问，为什么 Redis 快？AOF 和 RDB 怎么选？缓存击穿为什么不是加个锁就完事？分布式锁为什么会误删？Cluster 为什么会有 hash slot？一下就露怯。

这本书讲的是「为什么这样设计」，不是面向面试的背诵版。



### 第四条：《后端面试高手心法》58 讲

**面向想换工作或晋升的后端工程师。**

很多后端工程师不是没能力，而是不会讲。项目做过，但讲不出亮点；线上问题排过，但讲不成闭环；Redis、MySQL、Spring 都用过，但面试时回答像在背资料。

这 58 讲解决的是「表达」问题：简历项目怎么写，系统设计题怎么拆，面试官追问时怎么把业务场景、技术取舍、风险边界讲出来，让他问到你最有把握的方向。



## 这个星球的定位

四条主线，两条面向后端工程师（AI 编程实战 + Redis + 面试），一条面向所有职场人（AI 超能工具箱），一条两者都能用（Redis 这本工程师写的书其实产品和运营读完也有收获）。

你不用四条都走。按照自己当下最需要的来。

星球里配套有答疑区、实战分享区、面试回答改稿，有具体问题可以拿出来讨论。我在里面，不是只放资料不管的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/sgicmTR46Zvv49uibyhp6Z1qYtKTVTa3Iciakz2TycJyRdiapl5o8ibHHYbbia3dtkb6sMXy9FV3RCfRdLlEpeAV0de8UurFBZ41YCIZJPusV7V4Y/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=2)

## 加入方式

目前星球定价 ¥108，限量发放 ¥20 优惠券，使用后实付 **¥88**。优惠券用完即止。

扫描下方二维码加入，进入后有优惠券使用说明：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/sgicmTR46Zvup69A6XtbGg3aJU6q7jLjPOUZ3ftUUsFr1g6JE2kMxmCFefZCBUsu8V3h9hHepvC0Cl25rx7MfAGxY5NY9eQ3ic30X6todvDyA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)



做这个星球，不是因为 AI 编程现在热。

而是因为我越来越确定：在 AI 工具普及的这一年里，有一批工程师已经把 AI 真正跑进了工作流，产出速度和质量都和两年前不一样了。他们不是更聪明，是工作流更成熟了。同样的事情，也在非程序员里发生。

工具和模型会继续迭代，但「把任务拆清楚、把上下文给准确、把经验沉淀下来」这套思维方式不会过时。这是我想用这四套内容认真讲一遍的原因。

