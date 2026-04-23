# Copilot CLI Tool Mapping

skill 使用 Claude Code 的工具名称。当你在 skill 中遇到它们时，请使用你所在平台的对应项：

| Skill references | Copilot CLI equivalent |
|-----------------|----------------------|
| `Read`（文件读取） | `view` |
| `Write`（文件创建） | `create` |
| `Edit`（文件编辑） | `edit` |
| `Bash`（运行命令） | `bash` |
| `Grep`（搜索文件内容） | `grep` |
| `Glob`（按名搜索文件） | `glob` |
| `Skill` 工具（调用 skill） | `skill` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（派发 subagent） | `task`（参见 [Agent types](#agent-types)） |
| 多次 `Task` 调用（并行） | 多次 `task` 调用 |
| Task 状态/输出 | `read_agent`、`list_agents` |
| `TodoWrite`（任务跟踪） | `sql` 配合内置的 `todos` 表 |
| `WebSearch` | 无对应项 —— 使用 `web_fetch` 搭配搜索引擎 URL |
| `EnterPlanMode` / `ExitPlanMode` | 无对应项 —— 保持在主会话中 |

## Agent types

Copilot CLI 的 `task` 工具接受一个 `agent_type` 参数：

| Claude Code agent | Copilot CLI equivalent |
|-------------------|----------------------|
| `general-purpose` | `"general-purpose"` |
| `Explore` | `"explore"` |
| 具名插件 agent（如 `superpowers:code-reviewer`） | 从已安装的插件中自动发现 |

## Async shell sessions

Copilot CLI 支持持久化的异步 shell 会话，这在 Claude Code 中没有直接对应项：

| Tool | Purpose |
|------|---------|
| `bash` 配合 `async: true` | 在后台启动一个长时间运行的命令 |
| `write_bash` | 向运行中的异步会话发送输入 |
| `read_bash` | 从异步会话中读取输出 |
| `stop_bash` | 终止一个异步会话 |
| `list_bash` | 列出所有活跃的 shell 会话 |

## Additional Copilot CLI tools

| Tool | Purpose |
|------|---------|
| `store_memory` | 持久化关于代码库的事实以供未来会话使用 |
| `report_intent` | 更新 UI 状态栏中当前的意图 |
| `sql` | 查询会话的 SQLite 数据库（todos、元数据） |
| `fetch_copilot_cli_documentation` | 查阅 Copilot CLI 文档 |
| GitHub MCP tools（`github-mcp-server-*`） | 原生 GitHub API 访问（issue、PR、代码搜索） |
