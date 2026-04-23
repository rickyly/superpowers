---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
如果你是作为 subagent 被派发来执行特定任务的，跳过此 skill。
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
如果你认为某个 skill 有哪怕 1% 的可能适用于你正在做的事情，你 ABSOLUTELY MUST 调用该 skill。

如果某个 skill 适用于你的任务，你没有选择权。你 MUST 使用它。

这没有商量余地。这不是可选项。你无法为自己找借口摆脱这一点。
</EXTREMELY-IMPORTANT>

## Instruction Priority

Superpowers skill 会覆盖默认的系统提示行为，但**用户指令始终享有最高优先级**：

1. **用户的明确指令**（CLAUDE.md、GEMINI.md、AGENTS.md、直接请求）—— 最高优先级
2. **Superpowers skill** —— 在冲突时覆盖默认系统行为
3. **默认系统提示** —— 最低优先级

如果 CLAUDE.md、GEMINI.md 或 AGENTS.md 说 "don't use TDD"，而某个 skill 说 "always use TDD"，则遵循用户的指令。用户说了算。

## How to Access Skills

**在 Claude Code 中：** 使用 `Skill` 工具。当你调用某个 skill 时，其内容会被加载并呈现给你 —— 直接遵循即可。永远不要对 skill 文件使用 Read 工具。

**在 Copilot CLI 中：** 使用 `skill` 工具。Skill 会从已安装的插件中自动发现。`skill` 工具的用法与 Claude Code 的 `Skill` 工具相同。

**在 Gemini CLI 中：** Skill 通过 `activate_skill` 工具激活。Gemini 在会话开始时加载 skill 元数据，并按需激活完整内容。

**在其他环境中：** 请查阅你所在平台的文档了解 skill 的加载方式。

## Platform Adaptation

Skill 使用 Claude Code 的工具名称。非 Claude Code 平台：参见 `references/copilot-tools.md`（Copilot CLI）、`references/codex-tools.md`（Codex）以获取工具对应关系。Gemini CLI 用户会通过 GEMINI.md 自动加载工具映射。

# Using Skills

## The Rule

**在任何回复或动作之前调用相关或被请求的 skill。** 即使只有 1% 的可能某个 skill 适用，也应该调用该 skill 进行检查。如果调用后发现该 skill 不适合当前情况，你不必使用它。

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## Red Flags

下面这些想法意味着 STOP —— 你在找借口：

| 想法 | 现实 |
|---------|---------|
| "这只是一个简单的问题" | 问题也是任务。检查有无 skill。 |
| "我得先获取更多上下文" | skill 检查要在澄清提问之前完成。 |
| "让我先探索一下代码库" | skill 会告诉你如何探索。先检查。 |
| "我可以快速看一下 git/文件" | 文件缺少对话上下文。检查有无 skill。 |
| "让我先收集信息" | skill 会告诉你如何收集信息。 |
| "这用不上正式的 skill" | 如果 skill 存在，就用它。 |
| "我记得这个 skill" | skill 会演进。读当前版本。 |
| "这算不上一个任务" | 动作 = 任务。检查有无 skill。 |
| "这个 skill 杀鸡用牛刀了" | 简单的事会变复杂。用它。 |
| "我先做这一件事再说" | 做任何事**之前**都要检查。 |
| "这感觉挺有成效" | 无纪律的行动浪费时间。skill 可以避免这一点。 |
| "我知道那是什么意思" | 知道概念 ≠ 使用 skill。调用它。 |

## Skill Priority

当多个 skill 都可能适用时，按以下顺序使用：

1. **流程类 skill 优先**（brainstorming、debugging）—— 它们决定**如何**接手任务
2. **实现类 skill 其次**（frontend-design、mcp-builder）—— 它们指导执行

"Let's build X" → 先 brainstorming，再用实现类 skill。
"Fix this bug" → 先 debugging，再用领域相关的 skill。

## Skill Types

**刚性的**（TDD、debugging）：严格遵循。不要借故抛弃纪律。

**灵活的**（模式类）：将原则适配到上下文。

skill 本身会告诉你它属于哪一类。

## User Instructions

指令说的是**做什么**，而不是**怎么做**。"Add X" 或 "Fix Y" 并不意味着跳过工作流。
