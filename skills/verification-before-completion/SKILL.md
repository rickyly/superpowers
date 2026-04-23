---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# 完成之前先验证

## 概述

在没有验证的情况下宣称工作完成，是不诚实，而不是高效。

**核心原则：** Evidence before claims, always.

**违背本规则的字面意思，就是违背本规则的精神。**

## 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

如果你在本条消息中没有运行过验证命令，你就不能宣称它通过。

## 准入函数

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## 常见失败

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## 危险信号——停下

- 使用 "should"、"probably"、"seems to"
- 在验证之前就表达满意（"Great!"、"Perfect!"、"Done!" 等等）
- 即将 commit/push/PR 却未经验证
- 轻信 agent 的成功报告
- 依赖不完整的验证
- 想着"就这一次"
- 疲惫，想尽快收工
- **任何在未运行验证的情况下暗示成功的措辞**

## 合理化防御

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## 关键模式

**测试：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**回归测试（TDD 红绿循环）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**构建：**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**需求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent 委派：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## 为什么重要

来自 24 条失败记忆：
- 你的人类伙伴说过 "I don't believe you"——信任破裂
- 未定义的函数被发布——会崩溃
- 缺失的需求被发布——功能不完整
- 时间浪费在虚假的完成声明上 → 被打断 → 返工
- 违背："Honesty is a core value. If you lie, you'll be replaced."

## 何时适用

**ALWAYS before：**
- 任何形式的成功/完成声明
- 任何形式的满意表达
- 任何关于工作状态的正面表述
- 提交、创建 PR、标记任务完成
- 进入下一个任务
- 向 agent 委派工作

**本规则适用于：**
- 原话
- 改写和同义词
- 对成功的暗示
- 任何暗示完成/正确的表达

## 底线

**验证没有捷径。**

运行命令。阅读输出。然后再宣称结果。

这一点没得商量。
