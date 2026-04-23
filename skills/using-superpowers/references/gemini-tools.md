# Gemini CLI Tool Mapping

skill 使用 Claude Code 的工具名称。当你在 skill 中遇到它们时，请使用你所在平台的对应项：

| Skill references | Gemini CLI equivalent |
|-----------------|----------------------|
| `Read`（文件读取） | `read_file` |
| `Write`（文件创建） | `write_file` |
| `Edit`（文件编辑） | `replace` |
| `Bash`（运行命令） | `run_shell_command` |
| `Grep`（搜索文件内容） | `grep_search` |
| `Glob`（按名搜索文件） | `glob` |
| `TodoWrite`（任务跟踪） | `write_todos` |
| `Skill` 工具（调用 skill） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（派发 subagent） | 无对应项 —— Gemini CLI 不支持 subagent |

## No subagent support

Gemini CLI 没有与 Claude Code 的 `Task` 工具等价的机制。依赖 subagent 派发的 skill（`subagent-driven-development`、`dispatching-parallel-agents`）会通过 `executing-plans` 回落到单会话执行。

## Additional Gemini CLI tools

以下工具在 Gemini CLI 中可用，但在 Claude Code 中没有对应项：

| Tool | Purpose |
|------|---------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 将事实持久化到 GEMINI.md，跨会话保留 |
| `ask_user` | 向用户请求结构化输入 |
| `tracker_create_task` | 富任务管理（创建、更新、列举、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在做出更改前切换到只读研究模式 |
