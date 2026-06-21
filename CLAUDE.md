# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A **content-only knowledge base** (no code, no build system). It holds published technical articles in Markdown, serving as the content source for [kb-builder](https://github.com/MageByte-Zero/kb-builder) — a vector-indexing tool for AI-powered Q&A and retrieval.

License: CC BY-NC-SA 4.0. Only publicly published content goes here; paid/gated content (极客时间, 知识星球 paid posts) must stay out (see `.gitignore`).

## Content Structure

- `articles/backend/` — 197 backend articles organized by topic subdirectory (`redis/`, `mysql/`, `jvm/`, `kafka/`, `algorithm/`, `architecture/`, etc.)
- `articles/wechat-ai/` — 42 AI-direction articles from the 公众号 (flat directory, numbered filenames like `016-claude-code-large-codebase-tips.md`)
- `articles/workplace-ai/` — Workplace AI productivity (placeholder)
- `briefs/` — Short-form content: `papers/`, `tools/`, `trends/` (all currently empty)
- `community/` — AMA, Q&A (currently empty)
- `methodology/` — Writing standards, research frameworks (currently empty)
- `glossary.md` — AI + backend terminology table (Chinese)

## Article Frontmatter

Every article uses this YAML frontmatter:

```yaml
---
title: "文章标题"
alt_titles:          # optional — alternative headline candidates
  - "备选标题"
description: "一句话摘要，用于索引和 SEO"
date: "YYYY-MM-DD"
keywords: ["关键词1", "关键词2"]
platform: "微信公众号"   # or other source platform
source: "原创"          # or "转载"
cover: "https://..."    # optional cover image URL
---
```

## Key Conventions

- All article content is **Chinese (Simplified)**; the glossary includes bilingual terms.
- `articles/wechat-ai/` uses numbered-prefixed filenames (e.g. `016-`, `033-`) for ordering; `articles/backend/` uses descriptive topic-based filenames.
- `description` in frontmatter is the primary text used for indexing/search — keep it dense and keyword-rich.
- `.gitignore` explicitly excludes paid content directories (`articles/ai-coding-mastery/`, `articles/claude-poweruser/`). Do not remove these entries.

## Working With This Repo

There are no build, lint, or test commands. Typical tasks:

- **Add an article**: create a `.md` file in the appropriate subdirectory with correct frontmatter, then update `CHANGELOG.md`.
- **Update glossary**: edit `glossary.md` directly; keep the two-section format (AI terms / backend terms).
- **Reindex**: the companion tool `kb-builder` is run externally (`kb-builder index --source /path/to/awesome-ai-kb`); this repo itself has no indexing logic.
