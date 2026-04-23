# Codex Tool Mapping

skill 使用 Claude Code 的工具名称。当你在 skill 中遇到它们时，请使用你所在平台的对应项：

| Skill references | Codex equivalent |
|-----------------|------------------|
| `Task` 工具（派发 subagent） | `spawn_agent`（参见 [Named agent dispatch](#named-agent-dispatch)） |
| 多次 `Task` 调用（并行） | 多次 `spawn_agent` 调用 |
| Task 返回结果 | `wait` |
| Task 自动完成 | `close_agent` 释放槽位 |
| `TodoWrite`（任务跟踪） | `update_plan` |
| `Skill` 工具（调用 skill） | skill 原生加载 —— 直接遵循指令即可 |
| `Read`、`Write`、`Edit`（文件） | 使用你原生的文件工具 |
| `Bash`（运行命令） | 使用你原生的 shell 工具 |

## Subagent dispatch requires multi-agent support

将以下内容添加到你的 Codex 配置中（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

这会为 `dispatching-parallel-agents` 和 `subagent-driven-development` 等 skill 启用 `spawn_agent`、`wait` 和 `close_agent`。

## Named agent dispatch

Claude Code 的 skill 会引用 `superpowers:code-reviewer` 之类的具名 agent 类型。
Codex 没有具名 agent 注册表 —— `spawn_agent` 会从内置角色（`default`、`explorer`、`worker`）创建通用 agent。

当 skill 要求派发某个具名 agent 类型时：

1. 找到该 agent 的 prompt 文件（例如 `agents/code-reviewer.md`，或 skill 的本地 prompt 模板如 `code-quality-reviewer-prompt.md`）
2. 读取该 prompt 内容
3. 填充任何模板占位符（`{BASE_SHA}`、`{WHAT_WAS_IMPLEMENTED}` 等）
4. 派发一个 `worker` agent，将填充后的内容作为 `message`

| Skill instruction | Codex equivalent |
|-------------------|------------------|
| `Task tool (superpowers:code-reviewer)` | `spawn_agent(agent_type="worker", message=...)`，使用 `code-reviewer.md` 的内容 |
| 带内联 prompt 的 `Task tool (general-purpose)` | `spawn_agent(message=...)`，使用相同的 prompt |

### Message framing

`message` 参数是用户级输入，不是系统提示。为获得最高的指令遵循度，请这样组织结构：

```
Your task is to perform the following. Follow the instructions below exactly.

<agent-instructions>
[filled prompt content from the agent's .md file]
</agent-instructions>

Execute this now. Output ONLY the structured response following the format
specified in the instructions above.
```

- 使用任务委派式框架（"Your task is..."）而非人设式框架（"You are..."）
- 将指令包裹在 XML 标签中 —— 模型会把带标签的块视为权威内容
- 以显式的执行指令收尾，防止对指令本身进行概括

### When this workaround can be removed

这种做法是为了弥补 Codex 插件系统尚不支持 `plugin.json` 中 `agents` 字段的不足。一旦 `RawPluginManifest` 获得 `agents` 字段，插件就可以通过 symlink 指向 `agents/`（与现有的 `skills/` symlink 保持一致），skill 也就能直接派发具名 agent 类型。

## Environment Detection

那些会创建 worktree 或结束分支的 skill，应该在继续操作前用只读的 git 命令检测自身环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 已经处于一个链接的 worktree 中（跳过创建）
- `BRANCH` 为空 → detached HEAD（无法从沙箱中创建分支、推送或开 PR）

参见 `using-git-worktrees` 的 Step 0 和 `finishing-a-development-branch` 的 Step 1，了解每个 skill 如何使用这些信号。

## Codex App Finishing

当沙箱阻止分支/推送操作时（在外部管理的 worktree 中处于 detached HEAD），agent 会提交所有工作，并告知用户使用 App 的原生控件：

- **"Create branch"** —— 为分支命名，然后通过 App UI 进行 commit/push/PR
- **"Hand off to local"** —— 将工作移交到用户的本地检出中

agent 仍然可以运行测试、暂存文件，并输出建议的分支名、commit 信息和 PR 描述供用户复制。
