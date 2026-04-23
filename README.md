# Superpowers

Superpowers 是一套完整的软件开发方法论，服务于你的编码 agent。它由一组可组合的技能（skill）和一份确保 agent 使用这些技能的初始指令构成。

## 工作原理

一切从你启动编码 agent 的那一刻开始。只要它察觉到你要构建某个东西，就*不会*直接跳进去写代码，而是会先退一步，问你究竟想做什么。

从对话中梳理出一份规格说明后，它会把内容拆成足够短、读得下也消化得了的片段展示给你。

你确认设计方案之后，agent 会拟一份实现计划。计划写得足够清楚，哪怕交给一位热情有余但品味欠佳、判断力不足、对项目一无所知、还抗拒写测试的初级工程师也能照着做。计划强调严格的红-绿 TDD（测试驱动开发）、YAGNI（You Aren't Gonna Need It，你不会需要它）和 DRY（Don't Repeat Yourself，不要重复自己）。

接下来，只要你说"开始"，它就会启动一个 *subagent-driven-development*（子 agent 驱动开发）流程：派出多个 agent 分头处理每个工程任务，检查并审阅它们的成果，再继续向前推进。Claude 能自主工作两小时而不偏离既定计划，这样的情况并不罕见。

这套系统还有不少其他内容，但核心就是如此。因为所有技能都会自动触发，你不需要做任何特别的事，你的编码 agent 天然就拥有 Superpowers。

## 赞助

如果 Superpowers 帮你做成了能赚钱的事，并且你愿意的话，我会非常感激你考虑[赞助我的开源工作](https://github.com/sponsors/obra)。

谢谢！

- Jesse

## 安装

**注意：** 安装方式因平台而异。

### Claude Code 官方插件市场

Superpowers 已上架 [Claude 官方插件市场](https://claude.com/plugins/superpowers)。

从 Anthropic 的官方市场安装插件：

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code（Superpowers 市场）

Superpowers 市场为 Claude Code 提供 Superpowers 及其他相关插件。

在 Claude Code 中，先注册市场：

```bash
/plugin marketplace add obra/superpowers-marketplace
```

然后从该市场安装插件：

```bash
/plugin install superpowers@superpowers-marketplace
```

### OpenAI Codex CLI

- 打开插件搜索界面

```bash
/plugins
```

搜索 Superpowers

```bash
superpowers
```

选择 `Install Plugin`

### OpenAI Codex App

- 在 Codex app 中，点击侧边栏的 Plugins。
- 你会在 Coding 分类下看到 `Superpowers`。
- 点击 Superpowers 旁边的 `+`，按提示操作即可。

### Cursor（通过插件市场）

在 Cursor Agent chat 中，从市场安装：

```text
/add-plugin superpowers
```

或在插件市场中搜索 "superpowers"。

### OpenCode

向 OpenCode 发指令：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**详细文档：**[docs/README.opencode.md](docs/README.opencode.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

更新：

```bash
gemini extensions update superpowers
```

## 基本工作流

1. **brainstorming**（头脑风暴）—— 在写代码前触发。通过提问打磨粗糙的想法，探索备选方案，分段展示设计以供确认。保存设计文档。

2. **using-git-worktrees**（使用 git worktree）—— 设计获批后触发。在新分支上创建隔离工作区，运行项目初始化，验证测试基线干净。

3. **writing-plans**（编写计划）—— 设计获批后触发。把工作拆成每项 2-5 分钟可完成的小任务。每个任务都有明确的文件路径、完整的代码与验证步骤。

4. **subagent-driven-development**（子 agent 驱动开发）或 **executing-plans**（执行计划）—— 拿到计划后触发。为每个任务派出一个全新的子 agent，执行两阶段审查（先查规格符合度，再查代码质量）；或分批执行并设置人工检查点。

5. **test-driven-development**（测试驱动开发）—— 实现过程中触发。强制执行红-绿-重构：先写失败的测试，看它失败；再写最少的代码，看它通过；然后提交。删除测试之前写的代码。

6. **requesting-code-review**（请求代码审查）—— 任务之间触发。对照计划审查，按严重程度上报问题。关键问题会阻断后续流程。

7. **finishing-a-development-branch**（完成开发分支）—— 所有任务完成后触发。验证测试结果，给出选项（合并 / PR / 保留 / 丢弃），清理 worktree。

**agent 在执行任何任务前都会检查是否有相关技能。** 这些是强制流程，不是建议。

## 内容一览

### 技能库

**测试**

- **test-driven-development** —— 红-绿-重构循环（含测试反模式参考）

**调试**

- **systematic-debugging** —— 四阶段根因追踪流程（含 root-cause-tracing、defense-in-depth、condition-based-waiting 等技巧）
- **verification-before-completion** —— 确保问题真的已修复

**协作**

- **brainstorming** —— 苏格拉底式设计打磨
- **writing-plans** —— 详尽的实现计划
- **executing-plans** —— 分批执行并设置检查点
- **dispatching-parallel-agents** —— 并发子 agent 工作流
- **requesting-code-review** —— 审查前自检清单
- **receiving-code-review** —— 回应审查反馈
- **using-git-worktrees** —— 并行开发分支
- **finishing-a-development-branch** —— 合并 / PR 决策流程
- **subagent-driven-development** —— 快速迭代配两阶段审查（先查规格符合度，再查代码质量）

**元技能**

- **writing-skills** —— 按最佳实践创建新技能（含测试方法论）
- **using-superpowers** —— 技能系统入门介绍

## 理念

- **Test-Driven Development（测试驱动开发）**—— 永远先写测试
- **Systematic over ad-hoc（系统化优于临场发挥）**—— 流程胜过猜测
- **Complexity reduction（降低复杂度）**—— 以简洁为首要目标
- **Evidence over claims（证据胜过断言）**—— 先验证，再宣告成功

可阅读[最初的发布公告](https://blog.fsck.com/2025/10/09/superpowers/)。

## 贡献

Superpowers 的常规贡献流程如下。请注意，我们一般不接受新增技能的贡献，且对任何技能的修改都必须在我们支持的所有编码 agent 上都能正常工作。

1. Fork 仓库
2. 切换到 'dev' 分支
3. 基于它创建一个属于你自己工作的分支
4. 遵循 `writing-skills` 技能来创建和测试新的或修改后的技能
5. 提交 PR，务必完整填写 pull request 模板

完整指南见 `skills/writing-skills/SKILL.md`。

## 更新

Superpowers 的更新方式因编码 agent 而略有不同，但通常是自动的。

## 许可证

MIT License —— 详见 LICENSE 文件。

## 社区

Superpowers 由 [Jesse Vincent](https://blog.fsck.com) 和 [Prime Radiant](https://primeradiant.com) 的伙伴们共同打造。

- **Discord**：[加入我们](https://discord.gg/35wsABTejz)，获取社区支持、提问、分享你用 Superpowers 在做的事
- **Issues**：https://github.com/obra/superpowers/issues
- **发布公告**：[订阅](https://primeradiant.com/superpowers/)以获知新版本发布
