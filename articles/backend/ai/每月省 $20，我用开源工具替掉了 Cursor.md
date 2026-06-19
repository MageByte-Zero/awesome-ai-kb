> 每月 \$20 的 Cursor 订阅费值不值？当你知道同等效果可以用开源工具免费复现，答案可能会变。本文用真实项目测试，带你搭建一套本地 AI 编程工作流。

## 前言

2026 年初，AI 编程工具的竞争格局正在悄悄改变。

Cursor、GitHub Copilot、Windsurf 占据着大多数开发者的屏幕，但与此同时，一批开源 Coding Agent 项目正在 GitHub 上悄悄爆发：

- **open-swe**（LangChain 出品）：异步编码 Agent，单日 481 星
- **Claude Code**：Anthropic 官方 CLI，支持完整代码库理解
- **Cook CLI**：Claude Code 的任务编排层，Hacker News 热议

这些工具组合在一起，能实现什么效果？本文给出实测答案。

## 商业工具 vs 开源方案：先看结论

| 对比维度   | Cursor Pro       | 开源方案（本文）                |
| ---------- | ---------------- | ------------------------------- |
| 月费用     | \$20/月          | \$0（模型 API 费用另计）        |
| 代码库理解 | 全局索引，响应快 | Claude Code 原生支持，质量相当  |
| 多文件编辑 | Composer 功能强  | open-swe 异步并行，更适合大任务 |
| 隐私       | 代码上传至服务器 | 本地运行，代码不离机            |
| 可定制性   | 插件有限         | 完全开放，可接任意模型          |
| 上手成本   | 安装即用         | 需 30 分钟配置                  |

## 核心工具介绍

### Claude Code — 主力 Agent

Anthropic 官方出品的命令行 AI 编程工具。区别于 IDE 插件，它直接在终端运行，能：

- 读取整个代码库上下文
- 自主执行多步骤任务（写代码 → 运行测试 → 修复 Bug → 提交）
- 调用本地工具（bash、git、文件系统）

### open-swe — 异步编码 Agent

LangChain 最新开源项目。核心特点是**异步**：提交任务后去干别的，完成后通知你。

适合场景：大型重构、批量文件处理、跨多仓库联动修改。

### Cook CLI — 任务编排层

轻量级 CLI，把多个 Claude Code 任务串联成工作流，相当于给 AI 写一份「操作手册」。

## 环境搭建

### 安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

配置 API Key：

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
  # 建议写入 ~/.zshrc 持久化
```

验证：

```bash
claude --version
```

安装 open-swe

```bash
git clone https://github.com/langchain-ai/open-swe
  cd open-swe
  pip install -e .
  cp .env.example .env
  # 编辑 .env 填入 API Key
```

▎ 踩坑：open-swe 依赖 Python 3.11+，3.10 会遇到 typing.Self 报错，用 pyenv 切换版本解决。

安装 Cook CLI

`npm install -g cook-cli`

### 实战一：23 个文件迁移，4 分钟全自动

任务：Express.js 项目从 CommonJS 迁移到 ESM。

cd my-express-project
claude

输入指令：

把这个项目从 CommonJS 迁移到 ESM：

    1. 在 package.json 添加 "type": "module"
    2. 把所有 require() 改成 import
    3. 把所有 module.exports 改成 export
    4. 更新相对路径，确保 .js 扩展名完整
    5. 运行 npm test 验证迁移结果
       如果测试失败，自动分析错误并修复，最多重试 3 次

结果：23 个文件，4 分钟，测试全过，全程无人工干预。

关键是把验收标准写进指令——让它跑完测试才算完成，而不只是改完代码。

### 实战二：60 个函数补测试，异步搞定

> 任务：Django 项目补全所有缺失单元测试（约 60 个函数）。

```bash
from open_swe import Agent
agent = Agent(model="claude-opus-4-6")

task = """
分析 ./myapp/views.py 中所有函数的测试覆盖率。
对每个未被覆盖的函数，编写对应的单元测试。
测试文件命名：test_{原文件名}.py
覆盖正常路径和边界情况。
完成后运行 pytest --cov 输出覆盖率报告。
"""

task_id = agent.submit(task, repo_path="./myapp")
print(f"任务已提交: {task_id}，去喝杯咖啡吧")

result = agent.get_result(task_id)
print(result.summary)
```

▎ 踩坑：项目有数据库 migration 依赖时，在 config 设置 max_workers=1 避免并发冲突。

### 实战三：PR 前全套检查，一条命令

创建 Cookfile：

```
name: pre-pr-check
  steps:
    - name: code-review
      prompt: |
        审查暂存区改动，指出潜在 bug、不规范写法、缺失错误处理。
        输出 Markdown 格式审查报告到 .review.md

    - name: add-comments
      prompt: |
        为暂存区改动中所有 public 函数补充 JSDoc 注释
        风格参考 src/utils/example.js

    - name: update-changelog
      prompt: |
        根据 .review.md 和本次改动，在 CHANGELOG.md 顶部插入今日条目

    - name: run-tests
      shell: npm test
```

执行：

`cook run pre-pr-check`

四步串行自动执行，任一步失败立即中断报错。

真实账单

以每天约 2 小时 AI 编码为例：

```
┌────────────────────┬────────┬───────────────────────────────┐
│        方案        │ 月费用 │             备注              │
├────────────────────┼────────┼───────────────────────────────┤
│ Cursor Pro         │ $20    │ 固定，无限次数                │
├────────────────────┼────────┼───────────────────────────────┤
│ Claude API（本文） │ $8-15  │ 按 token，实测约 2M tokens/月 │
├────────────────────┼────────┼───────────────────────────────┤
│ 本地模型（Ollama） │ $0     │ 质量略低，隐私最好            │
└────────────────────┴────────┴───────────────────────────────┘

```

Claude Sonnet 4.6 定价：输入 $3/1M tokens，输出 $15/1M tokens。日常编码以输入为主，实际比想象便宜。

选型决策树

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/b57727353ce61d908d709ce1de8c57a6.png)

注意事项

    1. 超大仓库：配合 .claudeignore 排除无关目录，避免超出上下文窗口
    2. 关键逻辑必须人工 review：AI 偶尔生成看起来正确但逻辑有误的代码
    3. 先小任务摸费用：指令越模糊，Agent 探索成本越高
    4. open-swe 接口较新：生产使用建议锁定版本号

**进阶方向**

- 接入 MCP：让 Claude Code 直接读数据库、调内部 API
- GitHub Actions 集成：PR 自动触发 AI 代码审查
- 多模型路由：简单任务用 Haiku，复杂任务切 Opus，成本再降 60%

**总结**

- Claude Code：日常交互式编程，代码库理解不输 Cursor
- open-swe：大批量异步任务，解放双手
- Cook CLI：工程化工作流集成，一次配置长期复用

三者组合月成本可控在 \$10 以内，同时获得更高隐私保障和定制空间。

开源工具还没到「完全替代」Cursor 的程度，但对有特定需求的开发者，已经生产可用。