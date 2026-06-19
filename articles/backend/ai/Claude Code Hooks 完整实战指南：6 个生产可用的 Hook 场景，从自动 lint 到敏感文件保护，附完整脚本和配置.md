title: "Claude Code Hooks 实战：用 Shell 脚本给 AI 编程加上「护栏」"
description: "Claude Code Hooks 完整实战指南：6 个生产可用的 Hook 场景，从自动 lint 到敏感文件保护，附完整脚本和配置。掌握 PreToolUse、PostToolUse、SessionStart 三大核心事件"

keywords: ["Claude Code", "Hooks", "AI 编程", "自动化", "Shell 脚本", "开发工具"]

上周我让 Claude Code 帮我重构一个模块，它很贴心地执行了 `rm -rf dist/` 来清理构建产物。问题是，那个目录里还有一份我手动调试时放进去的配置文件——没有提交到 Git，直接没了。

这不是 Claude 的错。它按照正常的工程流程执行了清理操作。但作为人类，我希望有一种机制能在**危险操作执行之前**拦截它，就像 Git 的 pre-commit hook 一样——让自动化流程在关键节点停下来，先检查再执行。

Claude Code 的 Hooks 就是这个东西。它让你可以在 Agent 的生命周期中插入自定义的 Shell 脚本、HTTP 请求甚至 LLM 判断，实现从「信任 Agent」到「信任但验证」的转变。

这篇文章不讲概念，直接给 6 个生产可用的 Hook 场景，每个都附完整脚本和配置。看完直接能用。

## Hooks 到底是什么：Agent 生命周期的切面

如果你写过 Spring 的 AOP 或者用过 Git Hooks，Claude Code Hooks 的概念你一秒就懂：**在 Agent 执行特定操作的前后，自动触发你定义的逻辑。**

整个生命周期长这样：

![hook-lifecycle-events](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/hook-lifecycle-events.png)

核心事件有这几个：

| 事件                | 触发时机        | 你能做什么                 |
| ------------------- | --------------- | -------------------------- |
| `SessionStart`      | 对话开始/恢复   | 注入环境变量、加载上下文   |
| `PreToolUse`        | 工具执行前      | **拦截危险操作**、修改参数 |
| `PostToolUse`       | 工具执行后      | 自动 lint、日志记录        |
| `PermissionRequest` | 权限弹窗时      | 自动批准/拒绝特定操作      |
| `Stop`              | Agent 结束响应  | 阻止过早结束、触发总结     |
| `UserPromptSubmit`  | 用户提交 prompt | 预处理、添加上下文         |

其中 **PreToolUse** 是最常用的——90% 的「护栏」需求都在这里实现。

## 配置文件放在哪：三级作用域

Hooks 的配置是一个 JSON 对象，放在 Claude Code 的 settings 文件里。根据你的需求，有三个位置可选：

```
~/.claude/settings.json              # 全局：所有项目生效
.claude/settings.json                # 项目级：随代码提交，团队共享
.claude/settings.local.json          # 本地：不提交，只对自己生效
```

推荐做法是：**通用的安全策略放全局，项目特定的放 `.claude/settings.json` 提交到仓库**。

配置的基本结构：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

三层嵌套：事件名 → 匹配器 → 处理器数组。`matcher` 用正则匹配工具名，比如 `Bash` 只拦截命令行操作，`Edit|Write` 拦截文件修改，`mcp__.*` 拦截所有 MCP 工具调用。

## 场景一：拦截危险 Shell 命令

这是最高频的需求。创建一个脚本，拦截 `rm -rf`、`DROP TABLE`、`git push --force` 等危险操作。

```bash
#!/bin/bash
# .claude/hooks/block-dangerous-commands.sh

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# 定义危险模式（正则匹配）
DANGEROUS_PATTERNS=(
  'rm\s+-rf\s+/'           # rm -rf 根目录或绝对路径
  'git\s+push\s+.*--force' # force push
  'DROP\s+TABLE'           # 删表
  'DROP\s+DATABASE'        # 删库
  'git\s+reset\s+--hard'   # 丢弃未提交修改
  '>\s*/dev/sd'            # 写入磁盘设备
)

for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -iEq "$pattern"; then
    # 退出码 2 = 阻塞性错误，Claude 会收到拒绝通知
    echo "BLOCKED: 命令匹配危险模式 [$pattern]" >&2
    echo "原始命令: $COMMAND" >&2
    exit 2
  fi
done

# 通过检查，允许执行
exit 0
```

配置：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-commands.sh",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

> **踩坑记录**：退出码的含义和你想象的不一样。退出码 `1` 是**非阻塞**错误（Claude 会忽略并继续执行），退出码 `2` 才是**阻塞**错误（Claude 收到拒绝，停止操作）。我第一次写的时候用了 `exit 1`，结果发现 Claude 完全无视了我的拦截逻辑。

## 场景二：敏感文件保护

防止 Claude 修改 `.env`、密钥文件、锁文件等你不想被动的文件。

```bash
#!/bin/bash
# .claude/hooks/protect-sensitive-files.sh

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# 如果没有文件路径（比如 Bash 命令），直接放行
[ -z "$FILE_PATH" ] && exit 0

# 敏感文件模式
PROTECTED_PATTERNS=(
  '\.env$'
  '\.env\.'
  'credentials'
  'secret'
  '\.pem$'
  '\.key$'
  'package-lock\.json$'
  'pnpm-lock\.yaml$'
  'yarn\.lock$'
  'go\.sum$'
)

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if echo "$FILE_PATH" | grep -iEq "$pattern"; then
    echo "BLOCKED: 不允许修改敏感文件 $FILE_PATH" >&2
    exit 2
  fi
done

exit 0
```

配置（注意 matcher 匹配 Edit 和 Write 两个工具）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-sensitive-files.sh",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

![pretooluse-decision-flow](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/pretooluse-decision-flow.png)

## 场景三：代码修改后自动 Lint

每次 Claude 编辑完文件，自动跑一遍 linter，把结果反馈给它。这样 Claude 可以在同一轮对话里自动修复格式问题。

```bash
#!/bin/bash
# .claude/hooks/auto-lint.sh

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

[ -z "$FILE_PATH" ] && exit 0

# 根据文件类型选择 linter
case "$FILE_PATH" in
  *.js|*.ts|*.jsx|*.tsx)
    RESULT=$(npx eslint --fix "$FILE_PATH" 2>&1) || true
    ;;
  *.py)
    RESULT=$(python -m ruff check --fix "$FILE_PATH" 2>&1) || true
    ;;
  *.go)
    RESULT=$(gofmt -w "$FILE_PATH" 2>&1) || true
    ;;
  *.java)
    # 只检查不修复，把问题反馈给 Claude
    RESULT=$(mvn checkstyle:check -pl "$(dirname "$FILE_PATH")" 2>&1 | tail -5) || true
    ;;
  *)
    exit 0
    ;;
esac

# 如果有 lint 问题，作为 context 反馈给 Claude
if [ -n "$RESULT" ]; then
  jq -n --arg result "$RESULT" --arg file "$FILE_PATH" '{
    hookSpecificOutput: {
      hookEventName: "PostToolUse",
      additionalContext: "Lint 结果 [\($file)]:\n\($result)\n如果有问题请修复。"
    }
  }'
fi

exit 0
```

配置（注意这是 **PostToolUse**，在编辑完成**之后**触发）：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/auto-lint.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## 场景四：SessionStart 自动注入项目上下文

每次对话启动时，自动加载 Git 状态、最近 issue、当前分支等信息，让 Claude 一进来就有上下文。

```bash
#!/bin/bash
# .claude/hooks/inject-context.sh

# 收集项目状态
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "unknown")
RECENT_COMMITS=$(git log --oneline -5 2>/dev/null || echo "no commits")
DIRTY_FILES=$(git diff --name-only 2>/dev/null | head -10)
ISSUES=$(gh issue list -L 3 --json title,number --jq '.[] | "#\(.number) \(.title)"' 2>/dev/null || echo "GitHub CLI 不可用")

# 构建上下文
CONTEXT="
当前分支: $BRANCH
最近 5 次提交:
$RECENT_COMMITS

未提交的修改:
${DIRTY_FILES:-无}

最近的 Issues:
${ISSUES:-无}
"

jq -n --arg ctx "$CONTEXT" '{
  hookSpecificOutput: {
    hookEventName: "SessionStart",
    additionalContext: $ctx
  }
}'

exit 0
```

配置：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/inject-context.sh",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

这个 Hook 有个细节：SessionStart 的 matcher 可以区分 `startup`（新对话）、`resume`（恢复对话）和 `compact`（context 压缩后）。只在 `startup` 时加载完整上下文，避免 resume 时重复注入。

![hook-types-comparison](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/hook-types-comparison.png)

## 场景五：异步审计日志

记录 Claude 执行的所有操作，不阻塞正常流程。关键是 `"async": true`——异步执行，不影响 Agent 速度。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "async": true,
            "command": "echo \"$(date +%Y-%m-%dT%H:%M:%S) | $(jq -r '.tool_name') | $(jq -r '.tool_input | tostring' | head -c 200)\" >> \"$CLAUDE_PROJECT_DIR\"/.claude/audit.log"
          }
        ]
      }
    ]
  }
}
```

没有 matcher 意味着**所有工具调用**都会被记录。输出像这样：

```
2026-04-08T14:23:01 | Edit | {"file_path":"/src/main/java/Service.java","old_string":"...
2026-04-08T14:23:05 | Bash | {"command":"mvn compile -pl dlm-framework/dlm-rule"}
2026-04-08T14:23:12 | Read | {"file_path":"/src/test/java/ServiceTest.java"}
```

在出问题的时候，这个日志能帮你回溯 Claude 的每一步操作。

## 场景六：HTTP Hook 对接外部合规系统

如果你的团队有合规审核系统（比如安全扫描服务），可以用 HTTP Hook 在执行前请求外部 API：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/api/validate-command",
            "headers": {
              "Authorization": "Bearer $COMPLIANCE_TOKEN"
            },
            "allowedEnvVars": ["COMPLIANCE_TOKEN"],
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

HTTP Hook 会把工具调用的完整 JSON 作为 POST body 发送给你的服务。你的服务返回 `{"decision": "block", "reason": "..."}` 就能拦截操作。

这在企业环境里特别有用——安全团队可以维护一个中心化的策略服务，所有开发者的 Claude Code 实例都通过 HTTP Hook 对接。

## 进阶：四种 Hook 类型怎么选

Claude Code 支持四种 Hook 类型，适用场景不同：

| 类型      | 执行方式        | 适用场景                         | 延迟       |
| --------- | --------------- | -------------------------------- | ---------- |
| `command` | 本地 Shell 脚本 | 大多数场景，文件检查、lint、日志 | 极低       |
| `http`    | HTTP POST 请求  | 对接外部系统、合规审核           | 取决于网络 |
| `prompt`  | 发送给 LLM 评估 | 需要语义理解的判断               | 较高       |
| `agent`   | 启动 Sub-agent  | 需要多步推理的复杂验证           | 最高       |

**90% 的场景用 `command` 就够了。** `prompt` 和 `agent` 类型虽然强大，但每次触发都会消耗额外 token，不建议放在高频事件（如 PostToolUse）上。

![pretooluse-decision-flow](https://magebyte.oss-cn-shenzhen.aliyuncs.com/myself/pretooluse-decision-flow.png)

## 常见问题

**Q：Hook 脚本的 stdin 里具体传了什么？**

每种工具的输入结构不同。Bash 工具传 `{"tool_input": {"command": "...", "description": "...", "timeout": ...}}`，Edit 工具传 `{"tool_input": {"file_path": "...", "old_string": "...", "new_string": "..."}}`。所有 Hook 还会收到 `session_id`、`cwd`、`permission_mode` 等公共字段。用 `jq -r '.tool_input.command'` 提取你需要的字段。

**Q：Hook 执行失败会怎样？**

取决于退出码。退出码 0 = 成功，退出码 2 = 阻塞（Claude 停止操作），**其他任何退出码**（包括 1）= 非阻塞错误，仅记录到 debug 日志，Claude 继续执行。这是新手最容易踩的坑——别用 `exit 1` 来拦截。

**Q：怎么调试 Hook？**

在 Claude Code 里输入 `/hooks` 可以查看所有已加载的 Hook 配置。脚本的 stderr 输出会出现在 debug 日志里。建议开发时先在终端单独测试脚本：`echo '{"tool_input":{"command":"rm -rf /"}}' | bash .claude/hooks/block-dangerous-commands.sh`，检查退出码和输出。

**Q：Hook 会拖慢 Claude 的执行速度吗？**

同步 Hook 会。所以要注意两点：1）设置合理的 `timeout`（默认 600 秒太长了，大多数场景 5-10 秒足够）；2）对于不需要阻塞的操作（如审计日志），使用 `"async": true` 异步执行。

**Q：能在 Hook 里修改 Claude 的操作参数吗？**

可以。PreToolUse 的 Hook 可以在 JSON 输出里返回 `updatedInput` 字段来修改工具参数。比如你可以把 `rm -rf dist/` 改成 `rm -rf dist/ --interactive`，或者给 Bash 命令自动加上 `set -e`。但谨慎使用——静默修改用户意图可能造成困惑。

## 我的建议

Hooks 本质上是**给 AI Agent 加 middleware**。和 Web 开发里的中间件一样，最好的 Hook 是你写完就忘了它存在——它在背后默默工作，只在真正危险的时候跳出来拦你一下。

我个人的最小化配置是三个 Hook：

1. **PreToolUse/Bash**：拦截 `rm -rf`、`--force`、`DROP` 等危险模式
2. **PreToolUse/Edit|Write**：保护 `.env` 和锁文件
3. **PostToolUse/Edit|Write**（async）：自动 lint + 审计日志

这三个覆盖了 95% 的「AI 编程事故」场景。剩下的 5% 靠 Git。

## 参考资料

- [Claude Code Hooks 官方文档](https://code.claude.com/docs/en/hooks)
- [Claude Code Hooks 示例仓库](https://github.com/anthropics/claude-code/tree/main/examples/hooks)
- [Claude Code Settings 配置参考](https://code.claude.com/docs/en/settings)