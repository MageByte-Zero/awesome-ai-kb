---
title: "Claude Code月费$200太贵，三步切换DeepSeek直接降到¥10"
description: "Claude Code是目前最强的AI编程工具，但官方订阅每月高达$200。本文教中国用户三步完成安装、配置cc-switch工具和接入DeepSeek，月成本降至个位数人民币，国内直连可用。"
date: "2026-05-02"
keywords: ["Claude Code", "DeepSeek", "cc-switch", "AI编程工具", "低成本", "国内使用"]
platform: "微信公众号"
source: "实操教程"
alt_titles:
  - "Claude Code为什么能接DeepSeek？扒了底层配置原理才搞懂"
  - "DeepSeek文档里藏了一行配置，能把Claude Code费用压掉90%"
  - "2026年了还不会Claude Code？用DeepSeek一个月不到十块钱"
  - "每月200美元让我望而却步，直到发现可以把Claude Code挂上DeepSeek"
---

> 🖼️ **[封面图]** *微信公众号封面，900×500px*
>
> *渲染后替换为 `![封面图](claude-code-cc-switch-deepseek-cover.png)`*

# Claude Code月费$200太贵，三步切换DeepSeek直接降到¥10

第一次看到 Claude Code 的时候，我觉得这玩意儿真的能改变写代码的方式。但打开价格页面——Max 订阅每月 $200，折合人民币将近 1400 块。

加上国内访问需要解决网络问题，直接把大半个开发者社区劝退了。

后来我发现 Claude Code 有一个几乎没人专门讲清楚的特性：它的底层通信协议是开放的，可以把请求路由到任何兼容 Anthropic 格式的模型上。DeepSeek 在今年正式支持了 Anthropic API 格式，这意味着：用 DeepSeek 的 key，跑 Claude Code 的工作流，体验七八成，成本降九成。

这篇文章是完整的操作路径——从 Node.js 安装到 Claude Code 配上 DeepSeek 真正能用，中间用 cc-switch 做图形化管理。全程不依赖官方账号，国内环境可直连。

---

## 目录

- [为什么是 DeepSeek，不是其他模型](#why-deepseek)
- [环境准备：Node.js 和 npm 镜像](#environment)
- [安装 Claude Code](#install-claude-code)
- [cc-switch 是什么，为什么要用它](#cc-switch-intro)
- [安装 cc-switch](#install-cc-switch)
- [配置 DeepSeek 作为后端模型](#configure-deepseek)
- [验证效果：跑一个真实任务](#verify)
- [踩坑记录：这些问题我都遇到过](#pitfalls)
- [常见问题](#faq)

---

## 为什么是 DeepSeek，不是其他模型 {#why-deepseek}

国内可用的大模型里，Qwen、Kimi、GLM 都支持接入 Claude Code，但实测体验差异明显。选 DeepSeek 的原因是三点叠加：

**代码能力强。** DeepSeek V4 Pro 在 SWE-bench 等代码评测上的得分接近 Claude Sonnet 4，远超同等价位的其他国产模型。Claude Code 的工作流——写代码、读文件、执行 shell 命令、理解报错——DeepSeek 都能跟上节奏，不会出现频繁"不理解指令"的情况。

**原生支持 Anthropic 格式。** 这是关键点。DeepSeek 在 API 文档里专门提供了 `https://api.deepseek.com/anthropic` 这个 Base URL，直接对 Anthropic SDK 的请求格式做了兼容处理。不需要格式转换层，也不容易出协议不匹配的错误。

**价格实在。** 当前（2026年5月）的促销价：DeepSeek V4 Pro 输入 ¥3/百万 token，缓存命中降到 ¥0.025/百万 token，输出 ¥6/百万 token。Claude Sonnet 4 的官方价是 $3/百万 token 输入、$15/百万 token 输出，折算人民币大约贵 10-20 倍。日常写代码的场景里，大量请求都是在已有上下文基础上追加操作，缓存命中率高，实际成本会更低。

一个中度使用者（每天写代码 2-3 小时）用 DeepSeek 跑 Claude Code，一个月下来大概 ¥5-15，基本等于不花钱。

> 📊 **[技术配图：对比图 - Claude Sonnet vs DeepSeek V4 Pro 价格与能力对比]**
>
> *Excalidraw 文件：`images/claude-code-cc-switch-deepseek/price-comparison.excalidraw`*
>
> *渲染后替换为 `![Claude Sonnet与DeepSeek V4 Pro价格与代码能力对比](images/claude-code-cc-switch-deepseek/price-comparison.png)`*

---

## 环境准备：Node.js 和 npm 镜像 {#environment}

Claude Code 是 npm 包，依赖 Node.js 18 及以上版本。先确认本地环境：

```bash
node -v   # 输出 v18.x.x 或更高即可
npm -v    # 输出 9.x 以上即可
```

如果还没装 Node.js，去 [https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/) 下载 LTS 版本，Windows 直接安装包，macOS 推荐用 Homebrew：

```bash
brew install node
```

**国内 npm 镜像（重要）**

npm 默认走官方源，在国内下载 Claude Code 会慢甚至失败。先切到淘宝镜像：

```bash
npm config set registry https://registry.npmmirror.com
```

验证是否生效：

```bash
npm config get registry
# 应输出 https://registry.npmmirror.com
```

这一步如果跳过，后面安装 Claude Code 大概率卡在下载阶段，超时报错。

---

## 安装 Claude Code {#install-claude-code}

镜像配好之后，一行命令：

```bash
npm install -g @anthropic-ai/claude-code
```

`-g` 是全局安装，安装完后 `claude` 命令在任意目录可用。安装时间视网络而定，通常 30 秒到 2 分钟。

安装完成验证：

```bash
claude --version
# 输出版本号，如 claude 1.x.x
```

**关于原生安装方式**

Anthropic 最近推出了不依赖 Node.js 的原生安装包（类似 Homebrew tap 或直接下可执行文件）。如果你的机器 Node.js 环境有历史包袱比较麻烦，可以去官方文档查最新原生安装方式，思路一样，只是命令形式不同。

> 📊 **[技术配图：流程图 - Claude Code 安装流程]**
>
> *Excalidraw 文件：`images/claude-code-cc-switch-deepseek/install-flow.excalidraw`*
>
> *渲染后替换为 `![Claude Code国内安装流程图](images/claude-code-cc-switch-deepseek/install-flow.png)`*

---

## cc-switch 是什么，为什么要用它 {#cc-switch-intro}

这里有必要说清楚一件事：Claude Code 本身切换模型是**可以纯靠环境变量**实现的——设几个 `export`，Claude Code 就会把请求发到 DeepSeek。不用 cc-switch 一样能跑。

那为什么推荐 cc-switch？

cc-switch 是一个跨平台桌面工具（Windows/macOS/Linux 都有），专门给 Claude Code 做供应商管理。它解决了几个纯命令行方案的痛点：

**1. 多 key 管理。** 有时候你同时有 DeepSeek 的 key 和 SiliconFlow 的 key，想快速切换试效果。命令行方案要么手动改环境变量，要么写脚本，cc-switch 图形化一键切。

**2. 50+ 供应商预设。** 不需要自己查 Base URL 格式，cc-switch 内置了 DeepSeek、SiliconFlow、StepFun、OpenRouter 等常用供应商的配置模板，填 key 就能用。

**3. 余额监控。** 直接在界面上看各供应商的剩余额度，不用去每个平台的控制台单独确认。

**4. 无缝切换，不用重启。** 这一点很关键——切换供应商后，Claude Code 不需要重启，立刻生效。纯命令行改环境变量需要重新开终端会话，稍微麻烦一些。

**5. 系统托盘常驻。** 轻量化运行，不占主窗口，需要切换的时候托盘右键就能操作。

---

## 安装 cc-switch {#install-cc-switch}

**macOS（推荐 Homebrew）：**

```bash
brew tap farion1231/ccswitch
brew install --cask cc-switch
```

**macOS（手动）：**

去 [https://github.com/farion1231/cc-switch/releases](https://github.com/farion1231/cc-switch/releases) 下载最新的 `.dmg` 文件，拖入 Applications 安装。

**Windows：**

下载 `.msi` 安装包或便携版 `.zip`，安装或解压后运行 `cc-switch.exe`。

**Linux：**

根据发行版选择 `.deb`、`.rpm` 或 `.AppImage`：

```bash
# Debian/Ubuntu
sudo dpkg -i cc-switch_*.deb

# 或者 AppImage（通用）
chmod +x cc-switch-*.AppImage
./cc-switch-*.AppImage
```

**Arch Linux：**

```bash
paru -S cc-switch-bin
```

安装完成后首次启动，cc-switch 会出现在系统托盘。如果 macOS 提示「无法打开来自身份不明开发者的应用」，去「系统设置 → 隐私与安全性」手动允许一次即可。

---

## 配置 DeepSeek 作为后端模型 {#configure-deepseek}

这是整个流程里最关键的部分，分两种方式说：cc-switch 图形化配置和纯命令行配置。

### 方式一：cc-switch 图形化配置（推荐）

**第一步：获取 DeepSeek API Key**

打开 [https://platform.deepseek.com/](https://platform.deepseek.com/)，注册后在「API Keys」页面创建一个 key，格式是 `sk-xxxxxxxxxxxxxxxx`。新用户有一定免费额度。

**第二步：在 cc-switch 中添加 DeepSeek**

打开 cc-switch 主界面，点击「Add Provider」或「添加供应商」。在供应商列表里找到 DeepSeek（cc-switch 内置了预设），点击选中。

填写你的 API Key，其他配置 cc-switch 已经帮你填好：
- Base URL：`https://api.deepseek.com/anthropic`
- 主力模型：`deepseek-v4-pro`
- 快速模型：`deepseek-v4-flash`

保存后，点击该供应商卡片旁边的「Enable」按钮激活。

> 📊 **[技术配图：架构图 - cc-switch 作为 Claude Code 与模型供应商之间的管理层]**
>
> *Excalidraw 文件：`images/claude-code-cc-switch-deepseek/cc-switch-architecture.excalidraw`*
>
> *渲染后替换为 `![cc-switch管理层架构：Claude Code对接多供应商](images/claude-code-cc-switch-deepseek/cc-switch-architecture.png)`*

**第三步：验证配置已应用**

cc-switch 激活供应商后，会自动设置 Claude Code 所需的环境变量。可以在终端里验证：

```bash
echo $ANTHROPIC_BASE_URL
# 应输出 https://api.deepseek.com/anthropic

echo $ANTHROPIC_AUTH_TOKEN
# 应输出你的 DeepSeek key（sk-xxx...）
```

**切换不需要重启 Claude Code**，这是 cc-switch 相比手动配置的核心优势之一。

---

### 方式二：纯命令行配置

如果不想装 cc-switch，也可以直接在 shell 配置文件里设环境变量。

**macOS / Linux（编辑 `~/.zshrc` 或 `~/.bashrc`）：**

```bash
# Claude Code 使用 DeepSeek 作为后端
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=sk-你的DeepSeek_API_Key

# 主力模型用 v4-pro，子任务和快速响应用 v4-flash
export ANTHROPIC_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash
export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash

# 让模型尽量用完整的思考深度
export CLAUDE_CODE_EFFORT_LEVEL=max
```

编辑保存后，执行 `source ~/.zshrc` 让配置生效。

**Windows（PowerShell）：**

```powershell
$env:ANTHROPIC_BASE_URL = "https://api.deepseek.com/anthropic"
$env:ANTHROPIC_AUTH_TOKEN = "sk-你的DeepSeek_API_Key"
$env:ANTHROPIC_MODEL = "deepseek-v4-pro"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL = "deepseek-v4-pro"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "deepseek-v4-pro"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL = "deepseek-v4-flash"
$env:CLAUDE_CODE_SUBAGENT_MODEL = "deepseek-v4-flash"
$env:CLAUDE_CODE_EFFORT_LEVEL = "max"
```

每次开新 PowerShell 会话都要重新设。如果想持久化，可以把这些加到 PowerShell 的 profile 文件（`$PROFILE`）里。

---

**关于模型选择的说明**

`ANTHROPIC_MODEL`、`ANTHROPIC_DEFAULT_OPUS_MODEL`、`ANTHROPIC_DEFAULT_SONNET_MODEL` 这三个变量分别对应 Claude Code 在不同场景下调用的模型档位。Claude Code 内部把任务分为「重型推理」（用 Opus）、「标准」（用 Sonnet）、「轻量快速」（用 Haiku）三类。

把所有档位都设为 `deepseek-v4-pro` 是最稳妥的选择——体验最好，成本也可控。如果想进一步省钱，可以把 Haiku 和子任务模型设为 `deepseek-v4-flash`，轻量任务便宜很多，主任务质量不受影响。

---

## 验证效果：跑一个真实任务 {#verify}

配置完成后，进入一个真实项目目录启动 Claude Code：

```bash
cd ~/your-project
claude
```

首次启动会显示欢迎界面和使用条款，确认后进入交互模式。输入一个能体现多步推理能力的任务：

```
帮我扫描这个项目里所有 TODO 注释，整理成一个 markdown 清单，按文件分组，每条 TODO 附上所在行号
```

这个任务需要 Claude Code 读取多个文件、理解代码上下文、组织输出——是检验 DeepSeek 在 Claude Code 工作流里能力的好用例。

正常情况下，DeepSeek V4 Pro 能在 10-20 秒内完成，输出结果格式清晰。如果任务能正常执行到底，说明配置没问题。

跑完一个任务后，到 DeepSeek Platform 的「使用记录」页确认有 token 消耗记录，这样就闭环了——你能看到实际花了多少。

---

## 踩坑记录：这些问题我都遇到过 {#pitfalls}

**坑一：npm 安装卡死，长时间无响应**

症状：`npm install -g @anthropic-ai/claude-code` 执行后停在某一步不动。

原因：npm 还在走官方源，国内网络下载超时。

解决：先执行 `npm config set registry https://registry.npmmirror.com`，再重试安装命令。

**坑二：启动 claude 后报 401 或「API key invalid」**

症状：执行 `claude` 后立刻报认证错误。

原因九成是环境变量没生效。常见两种情况：
1. 修改了 `~/.zshrc` 但没有执行 `source ~/.zshrc`，当前终端会话里变量还是旧的
2. 用了 GUI 终端（如 iTerm2、Windows Terminal），某些情况下环境变量继承有问题

解决：在启动 claude 的终端里手动执行：

```bash
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=sk-你的key
```

然后再执行 `claude`，验证是否正常。如果这样可以，说明是 shell 配置文件没有被正确加载。

**坑三：cc-switch 切换后请求还是走旧供应商**

症状：在 cc-switch 里切换到 DeepSeek 后，Claude Code 里的请求看起来还是走 Anthropic 官方。

原因：cc-switch 修改的是系统级或会话级环境变量，但如果 Claude Code 已经缓存了会话启动时的变量，需要重新打开一个终端让变量刷新。

注意：cc-switch 官方文档说「Claude Code 不需要重启」——这是相对于其他工具（比如 Gemini CLI）而言的。实际上**终端会话**有时需要重开。重开一个终端窗口，变量立刻刷新。

**坑四：`deepseek-v4-pro` 报错「模型不存在」**

症状：提示找不到指定模型。

原因：DeepSeek 的旧模型名 `deepseek-chat` 和 `deepseek-reasoner` 是 V4 Pro 和 V4 Flash 的别名，计划在 2026 年 7 月 24 日废弃。如果你的教程是从早期文章抄过来的，模型名可能还是旧格式。

解决：把 `ANTHROPIC_MODEL` 等变量里的旧模型名统一改成 `deepseek-v4-pro` 和 `deepseek-v4-flash`。

**坑五：macOS 提示 cc-switch「来自身份不明的开发者」**

这是正常的 macOS Gatekeeper 行为，不是病毒。

解决：进入「系统设置 → 隐私与安全性」，在页面底部找到「cc-switch 被阻止」的提示，点「仍要打开」即可。只需操作一次，后续正常启动。

---

## 常见问题 {#faq}

**Q：必须用 cc-switch 吗，直接设环境变量不行吗？**

A：直接设环境变量完全可以跑。cc-switch 解决的是管理问题：多 key 切换、余额监控、供应商预设。如果你只用 DeepSeek 一个供应商，完全可以把几行 export 加到 shell 配置文件了事，不用装额外工具。cc-switch 更适合同时尝试多个供应商、或者做成本横向对比的用户。

**Q：这个方案可以用 DeepSeek 的免费额度吗？**

A：可以。DeepSeek 新用户有免费 token 额度，通过 API 可以正常使用，不需要充值。免费额度用完后才需要充值，但充值起步金额很低，¥10 左右就够用一两个月。注意免费额度有效期，过期未用就没了。

**Q：接了 DeepSeek 之后，Claude Code 还能用原来的 claude.ai 账号吗？**

A：不能同时用。环境变量 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_AUTH_TOKEN` 决定了请求去哪里。设成 DeepSeek 的配置，Claude Code 就只走 DeepSeek；如果想换回官方，把这两个环境变量清掉（`unset ANTHROPIC_BASE_URL` 等），再重新用官方方式登录。cc-switch 的价值就在这里——它帮你管理多套配置，一键来回切。

**Q：DeepSeek 的代码能力和 Claude Sonnet 差距大吗，用起来体验怎么样？**

A：差距存在，但在日常开发任务里感知不强。DeepSeek V4 Pro 在 SWE-bench 的得分接近 Claude Sonnet 4，处理「读代码 → 定位问题 → 修改 → 解释」这类任务表现稳定。明显感知到差距的场景是复杂的多轮推理、需要理解大型代码库整体架构，以及生成高质量文档。如果你的日常任务偏向代码补全、bug 修复、重构，DeepSeek 基本够用，省下来的钱可以在真正需要 Claude 能力的关键任务上用。

**Q：cc-switch 支持哪些其他供应商，除了 DeepSeek？**

A：cc-switch 内置了 50+ 供应商预设，常见的包括：SiliconFlow（硅基流动）、StepFun（阶跃星辰）、Qwen（通义千问）、Kimi、OpenRouter、Novita AI 等。国内厂商基本都在列表里，充值和访问都没有国内网络问题。切换方式和 DeepSeek 一样，填 key 就能启用。

---

说到底，Claude Code 的核心价值在它的工作流和工具调用能力，不在于背后一定要是 Anthropic 的模型。一旦把模型层和工具层解耦，选择权就回到用户手里了。

DeepSeek 的出现恰好在这个时间点补上了这个空缺：足够强的代码能力、足够低的价格、原生兼容 Anthropic 格式。

我觉得更有意思的问题是：随着国产模型能力的持续提升，「哪个模型最好」会变成一个需要定期重新校准的答案，而不是一个能一劳永逸的选择。cc-switch 这类工具的意义，可能正在于此。

下篇我打算写 Claude Code 的 CLAUDE.md 实战——如何为不同项目定制 AI 编程上下文，让模型真正理解你的代码库风格和约束。感兴趣的话关注一下，发布了会推。

如果你正在纠结 AI 编程工具的费用问题，或者身边有同学还没迈出这一步，这篇可以直接甩给他——省得他再趟一遍这些坑。

---

## 参考资料

- [DeepSeek 官方文档：接入 Claude Code](https://api-docs.deepseek.com/guides/agent_integrations/claude_code)
- [cc-switch GitHub 仓库](https://github.com/farion1231/cc-switch)
- [DeepSeek API 定价页](https://api-docs.deepseek.com/quick_start/pricing)
- [Claude Code 快速开始（官方中文文档）](https://code.claude.com/docs/zh-CN/quickstart)
