# awesome-ai-kb

码哥（MageByte）AI + 后端技术知识库内容仓库。本仓库是 [kb-builder](https://github.com/MageByte-Zero/kb-builder) 的内容源，收录公众号、博客、专栏等渠道已发布的公开技术文章，供 AI 知识库构建工具索引和检索。

## 目录结构

```
awesome-ai-kb/
├── articles/               # 文章正文（Markdown）
│   ├── backend/            #   后端技术文章（Redis / MySQL / JVM / Kafka / 算法 / 架构等）
│   ├── wechat-ai/          #   公众号 AI 方向文章（Claude Code / MCP / Agent / AI 工作流等）
│   └── workplace-ai/       #   职场 AI 提效文章（待扩充）
├── briefs/                 # 简报
│   ├── papers/             #   论文速读
│   ├── tools/              #   工具荐评
│   └── trends/             #   趋势观察
├── community/              # 社区内容（AMA、Q&A 等）
├── methodology/            # 方法论（写作规范、研究框架等）
├── glossary.md             # AI + 后端技术术语表
└── README.md               # 本文件
```

## 使用方式

本仓库是 kb-builder 的默认内容源。在 kb-builder 项目中运行：

```bash
kb-builder index --source /path/to/awesome-ai-kb
```

即可将本仓库中的所有文章建立向量索引，供 AI 问答和内容检索使用。

## Content License

本仓库中的所有文章内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。

- **署名（BY）**：转载、引用请注明出处「码哥字节 / MageByte」及原文链接。
- **非商业性使用（NC）**：不得将内容用于商业用途。
- **相同方式共享（SA）**：若对内容进行二次创作，须采用相同的许可协议发布。

## 注意事项

- 本仓库仅收录已公开发布的文章，**不含付费专栏付费内容**（如极客时间 / 知识星球付费文章）。
- 仓库内容会持续更新，新文章发布后会定期同步到本仓库。
- 若有文章授权或版权问题，请通过 GitHub Issues 联系。

## 关于码哥

码哥（MageByte）是微信公众号「码哥字节」和「码哥跳动」的作者，专注于：

- **后端技术**：Redis、MySQL、JVM、Kafka、分布式系统、算法与数据结构
- **AI 提效**：Claude Code、MCP、Agent、AI 工作流、编码智能体
- **职场成长**：AI 时代程序员的效率提升与职业发展

微信公众号：码哥字节 / 码哥跳动
