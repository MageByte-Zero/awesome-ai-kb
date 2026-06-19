---
title: "30 秒描述需求，5 分钟出稿，我的述职演示文稿就这么做好了"
description: "frontend-slides 是 GitHub 17100 star 的 Claude Code skill，把文字描述或旧 PPT 文件转成零依赖的单文件 HTML 动画演示。本文带你从安装到出稿跑完全流程，顺带讲清楚旧 PPT 怎么导入、HTML 怎么放映，以及真实使用中的三个坑。"
date: "2026-05-11"
keywords: ["frontend-slides", "AI做PPT", "PPT转HTML", "Claude Code", "AI演示文稿", "职场AI工具"]
platform: "微信公众号"
source: "GitHub Trending"
alt_titles:
  - "17100 个 GitHub star 的工具说：PPT 软件该退休了"
  - "GitHub 上 17100 人收藏的工具，把 PPT 变成什么样让我没想到"
  - "做完演示被说"不够高端"，我试了这个工具，真的无话可说"
  - "2026年还在用 PPT 模板凑演示？这个 AI 工具让我彻底换了路子"
---

# 30 秒描述需求，5 分钟出稿，我的述职演示文稿就这么做好了

上周五下午四点，领导发消息：「下周一要给大区汇报，帮你安排了 15 分钟，你准备一下演示文稿。」

距离下班还有一个小时，距离周一还有两天半。

我打开 PowerPoint，盯着空白页面和那十几个内置模板发了一分钟的呆。「企业蓝」「活力橙」「简约灰」——这些模板码哥见过太多次，领导也见过太多次。用了感觉像是在发月度运营报告，不是在做一次有分量的汇报。

然后我想起了 frontend-slides。

它是 GitHub 上一个 17100 star 的工具，定位是：**用 AI 把文字描述或旧 PPT 文件，直接生成一份零依赖的单文件 HTML 动画演示文稿**。不需要 Canva，不需要 MasterGo，不需要在模板里抠颜色。只需要告诉 AI 你要讲什么，剩下的它来。

最终的演示文稿我在当天六点发给了领导。他回了一句：「这个风格挺好，不像以前那种模板的感觉。」

下面把完整的过程讲给你。

![frontend-slides 职场使用场景示意](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/frontend-slides-html-presentation-cover.png)
*图：用 AI 描述内容，直接生成高端 HTML 演示文稿*

## 先搞清楚它是什么，解决什么问题

**frontend-slides 是一个 Claude Code 技能（skill）**，也就是说，它不是一个独立的网站或 App，而是安装在 Claude Code 里、通过对话驱动的工具。

你跟它说：「帮我做一个关于 Q2 销售复盘的演示，五页，风格现代一点。」它就会生成一个完整的 HTML 文件，里面包含了所有动画、排版、配色——打开浏览器就能全屏放映，不需要安装任何额外软件。

你也可以把一份旧的 PPTX 文件扔给它，它能把文字、图片和备注全部提取出来，再按你选的风格重新排版成 HTML。

**和普通 PPT 模板相比，它的核心差异是三点：**

第一，**零依赖，单文件**。生成的 HTML 文件里内联了所有样式和脚本，复制给任何人，用任何浏览器打开都一模一样，不会出现字体丢失、图片找不到的问题。

第二，**12 种精心设计的风格**，覆盖了深色系（Bold Signal、Electric Studio、Dark Botanical）、浅色系（Notebook Tabs、Vintage Editorial）和特殊风格（Neon Cyber、Swiss Modern），每一种都明确拒绝了「AI 生成的那种大众模板感」。

第三，**从旧 PPT 到新演示的完整闭环**。你已有的内容不用白费，PPT 里的文字和图片都可以继承过来，只是换了一套更高端的视觉呈现。

用之前 vs 用之后：

![用AI工具前后对比，3小时VS20分钟](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/frontend-slides-html-presentation-fig01-cmp.png)
*图：同样的汇报内容，用模板 vs 用 frontend-slides 的时间和视觉效果对比*

## 安装：只需要做一次的准备

在开始之前，你的电脑需要准备好两件东西：

**第一：Claude Code**

Claude Code 是 Anthropic 出的 AI 编程工具，但你不用会编程，这里只是把它当一个能运行技能的平台来用。

安装方式：到 Claude 官网（[claude.ai/code](https://claude.ai/code)）按指引下载安装。需要有一个 Claude 账号，免费账号有使用限额，付费订阅（Pro，每月 $20）没有限制。

国内注意：Claude 官网需要科学上网。

**第二：frontend-slides 技能**

打开 Claude Code，在对话框里输入：

```
/plugin marketplace add zarazhangrui/frontend-slides
/plugin install frontend-slides@frontend-slides
```

回车执行，等 30 秒左右，安装完成。这是一次性操作，以后每次打开 Claude Code 都可以直接用。

**安装之后，怎么验证成功了？** 在对话框里输入 `/frontend-slides`，如果 AI 回应说准备好了、让你描述需求，就说明安装成功。

## 上手三步：从零到一份完整演示文稿

### 第一步：描述你要讲什么

打开 Claude Code，输入 `/frontend-slides`，然后告诉它你的需求。

不需要按特定格式，用正常说话的方式就行：

> 「帮我做一个 Q2 销售复盘的演示文稿，大概 6 页。主要内容是：第一页封面，第二页 Q2 整体完成率 87%，达成了年度目标的 43%；第三页是三个重点成交案例；第四页是下半年的三个策略方向；第五页是重点客户列表；第六页是结语。风格要现代、深色，不要那种大众感。」

字数多一点没关系，描述越具体，AI 生成的内容越准确。

### 第二步：选风格

AI 收到需求后，会展示 12 种风格预览，你来选一个。

不知道选哪个？这里是码哥在不同场合的推荐：

| 使用场景 | 推荐风格 | 特点 |
|---------|---------|------|
| 向领导汇报、述职 | Bold Signal | 深色背景，标题强调，专业感强 |
| 对外客户提案 | Swiss Modern | 克制、精致，有设计感 |
| 团队内部分享 | Notebook Tabs | 干净、轻盈，不压迫 |
| 产品发布、创意展示 | Electric Studio 或 Neon Cyber | 有视觉冲击力 |
| 年度总结、回顾类 | Vintage Editorial | 有质感，不花哨 |

### 第三步：等待生成，下载 HTML

选完风格，AI 会开始生成。一般 1-3 分钟后，你会得到一个 `.html` 文件。

把这个文件发给同事、发给领导——任何人用任何浏览器打开，效果都一模一样。

**放映方式：**
- 按 `F11`（Windows）或 `Cmd + Ctrl + F`（Mac）全屏
- 用方向键翻页，或者直接点击屏幕
- 不需要 PowerPoint，不需要安装插件

![frontend-slides操作流程，三步完成演示文稿](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/frontend-slides-html-presentation-fig02-flow.png)
*图：从输入描述到得到演示文稿的完整流程*

## 进阶：把旧 PPT 文件直接转换过来

如果你已经有一份现成的 PPTX 文件，想换一套更好看的视觉风格，流程稍微多一步。

**第一步：提取 PPTX 内容**

frontend-slides 自带了一个提取脚本。把你的 PPTX 文件放到一个固定目录，然后在 Claude Code 里告诉 AI：「我有一个 PPTX 文件，路径是 /Users/你的用户名/Documents/述职.pptx，帮我提取内容」。

AI 会自动运行提取脚本（`python scripts/extract-pptx.py`），把 PPT 里的所有文字、图片、备注整理出来，然后展示给你确认。

如果你愿意自己运行，命令是：

```bash
python scripts/extract-pptx.py 述职.pptx
```

需要先安装 python-pptx 库：`pip install python-pptx`

**第二步：确认提取内容，选风格**

AI 会告诉你它识别到了几页、每页的标题是什么、有多少张图片。你确认没问题后，选一个风格，它就开始生成。

**一个需要注意的点：动画不保留。** PPTX 里原本的切换动画和元素动画不会被迁移，HTML 版会用 frontend-slides 自带的动画效果替代，通常更流畅，但风格不同。

**另一个注意点：复杂图表。** 如果你的 PPT 里有 Excel 联动的数据图表，这些图表 AI 会按截图处理，不是可编辑的矢量图。

## 三个真实使用中的坑

**坑一：内容描述太简短，生成的页面质量差**

只告诉 AI「帮我做 Q2 复盘」，生成出来的内容基本是通用占位文字。AI 需要你告诉它具体的数字、具体的案例名、具体的结论。把你脑子里知道的内容直接说给它，越具体越好。

**坑二：HTML 文件在微信里发送会失效**

HTML 文件不能通过微信直接发送给对方打开——微信会把 HTML 当网页处理，通常只显示代码。正确做法是：发到群文件、云盘链接，或者先转成 PDF（浏览器 → 打印 → 保存为 PDF）再发送。如果是当场汇报，直接在自己电脑上打开浏览器放映就行。

**坑三：中文字体有时显示偏差**

HTML 文件在某些 Windows 系统上打开，中文字体可能和你生成时看到的不一样（主要是字重偏细的问题）。解决方法是用 Chrome 打开，Chrome 对字体的支持最稳定。如果需要发给不确定对方用什么浏览器的人，转成 PDF 是最保险的做法。

## 常见问题

**Q：需要付钱吗？frontend-slides 本身是免费的，但 Claude 要付费吗？**

A：frontend-slides 是 MIT 开源，免费使用。但它需要运行在 Claude Code 上，Claude Code 的使用需要 Claude 账号，免费账号每天有使用额度，日常做 1-2 份演示够用。如果你经常用，或者生成内容比较复杂（比如导入大 PPTX），建议订阅 Pro（$20/月）。国内付费可以通过官方网站用信用卡结算。

**Q：生成的演示文稿可以修改内容吗？**

A：可以，但需要一点点基础。HTML 文件用任何文本编辑器（记事本、VS Code 都行）打开，找到对应的文字内容直接改，保存后刷新浏览器即可。不需要会编程，就是改文字。但如果要调整排版、颜色，那就需要一点 CSS 基础，或者直接告诉 AI「帮我把第三页的标题改成 XX，背景换成浅色」，让 AI 帮你改。

**Q：一份演示文稿大概要花多少时间？**

A：如果是从零描述需求，通常是 5-8 分钟（等 AI 生成）。如果你的描述不够具体，需要来回修改，可能要 15-20 分钟。如果是导入 PPTX 转换，额外需要 2-3 分钟提取内容。整体来说，比从模板手动做快 3-5 倍，但不是「零工作量」——你还是要把内容想清楚、提供给 AI。

**Q：12 种风格之外，可以定制颜色或字体吗？**

A：可以。告诉 AI「帮我用 Bold Signal 风格，但主色改成我们公司的橙色 #FF6B00」，它能做到。字体同理，但要注意太小众的字体可能在不同电脑上显示效果不一。建议以 12 种预设风格为基础微调，不要从零自定义，否则容易破坏原来的视觉平衡。

## 参考资料

- [frontend-slides GitHub 仓库](https://github.com/zarazhangrui/frontend-slides)（含 v2.0.0 更新说明和 demo 视频）
- [Claude Code 官方文档](https://claude.ai/code)（安装和使用 Claude Code）
- [python-pptx 文档](https://python-pptx.readthedocs.io/)（PPTX 提取脚本的底层库）

在做演示这件事上，工具确实不是最重要的——内容好、逻辑清才是根本。但一份视觉上明显高出平均水准的演示文稿，会让你在汇报前就多了三分信心，也让对方在你开口之前就觉得「这个人做事认真」。

frontend-slides 现在已经是码哥做对外演示的默认工具了，不用每次在模板里找颜色搭配，把时间省下来放到内容本身。

下篇打算写 Claude Code 的另一个用法——怎么用它来自动处理每周的数据报表，感兴趣的话关注一下，发布了会第一时间推送。

如果你团队里有人每周要做汇报演示，这篇可以直接发给他参考。
