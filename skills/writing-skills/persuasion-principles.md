# Persuasion Principles for Skill Design

## Overview

LLM 对说服原则的反应与人类相同。理解这种心理学有助于你设计更有效的 skill——不是为了操控，而是为了确保关键实践即使在压力下也会被遵循。

**研究基础**：Meincke et al. (2025) 用 N=28,000 次 AI 对话测试了 7 条说服原则。说服技术让遵守率提高了一倍多（33% → 72%，p < .001）。

## The Seven Principles

### 1. Authority
**它是什么**：对专业知识、凭证或官方来源的遵从。

**它在 skill 中如何起作用：**
- 祈使语气："YOU MUST"、"Never"、"Always"
- 不容商量的框架："No exceptions"
- 消除决策疲劳和合理化

**何时使用：**
- 强制执行纪律的 skill（TDD、验证要求）
- 安全关键实践
- 已确立的最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. Commitment
**它是什么**：与先前行为、陈述或公开宣告保持一致。

**它在 skill 中如何起作用：**
- 要求声明："Announce skill usage"
- 强制明确选择："Choose A, B, or C"
- 使用跟踪：用 TodoWrite 做清单

**何时使用：**
- 确保 skill 真的被遵循
- 多步流程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. Scarcity
**它是什么**：来自时限或稀缺性的紧迫感。

**它在 skill 中如何起作用：**
- 限时要求："Before proceeding"
- 顺序依赖："Immediately after X"
- 防止拖延

**何时使用：**
- 即时验证要求
- 时间敏感的工作流
- 防止「我等会儿再做」

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. Social Proof
**它是什么**：与他人的行为或被视为常态的做法保持一致。

**它在 skill 中如何起作用：**
- 普适模式："Every time"、"Always"
- 失败模式："X without Y = failure"
- 建立规范

**何时使用：**
- 记录普适实践
- 警告常见失败
- 强化标准

**示例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. Unity
**它是什么**：共同身份、「我们感」、群体归属。

**它在 skill 中如何起作用：**
- 协作语言："our codebase"、"we're colleagues"
- 共同目标："we both want quality"

**何时使用：**
- 协作工作流
- 建立团队文化
- 非层级化实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. Reciprocity
**它是什么**：回报所受恩惠的义务。

**它如何起作用：**
- 慎用——容易显得操控性强
- 在 skill 中很少需要

**何时避免：**
- 几乎总是避免（其他原则更有效）

### 7. Liking
**它是什么**：倾向于与我们喜欢的人合作。

**它如何起作用：**
- **不要**用于强制遵守
- 与诚实反馈文化冲突
- 制造谄媚

**何时避免：**
- 在纪律强制场景中一律避免

## Principle Combinations by Skill Type

| Skill 类型 | 使用 | 避免 |
|------------|-----|-------|
| 纪律强制型 | Authority + Commitment + Social Proof | Liking、Reciprocity |
| 指导/技术型 | 适度 Authority + Unity | 重度权威 |
| 协作型 | Unity + Commitment | Authority、Liking |
| 参考型 | 只要清晰 | 所有说服原则 |

## Why This Works: The Psychology

**明确界线的规则减少合理化：**
- "YOU MUST" 消除决策疲劳
- 绝对语言消除「这是例外吗？」的疑问
- 明确的反合理化反制堵住具体漏洞

**实施意图产生自动行为：**
- 明确触发 + 必需动作 = 自动执行
- "When X, do Y" 比 "generally do Y" 更有效
- 减少遵守的认知负担

**LLM 是 parahuman（类人）：**
- 在包含这些模式的人类文本上训练
- 训练数据中权威语言常在遵从之前出现
- 承诺序列（陈述 → 行动）被频繁建模
- 社会认同模式（每个人都做 X）建立规范

## Ethical Use

**正当：**
- 确保关键实践被遵循
- 创作有效文档
- 防止可预见的失败

**不正当：**
- 为个人利益操控
- 制造虚假紧迫感
- 基于愧疚的强制遵守

**检验标准**：如果用户完全理解了这种技术，它是否仍然服务于他们的真实利益？

## Research Citations

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 七条说服原则
- 影响力研究的实证基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 用 N=28,000 次 LLM 对话测试 7 条原则
- 使用说服技术后遵守率从 33% 提升到 72%
- Authority、commitment、scarcity 最有效
- 验证了 LLM 行为的 parahuman 模型

## Quick Reference

设计 skill 时，自问：

1. **这是什么类型？**（纪律 vs 指导 vs 参考）
2. **我想改变什么行为？**
3. **适用哪些原则？**（纪律型通常是 authority + commitment）
4. **我是不是组合了太多？**（别把七个全用上）
5. **这合乎道德吗？**（是否服务用户的真实利益？）
