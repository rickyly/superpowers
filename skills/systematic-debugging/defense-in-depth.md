# 纵深防御式校验（defense-in-depth）

## 概述

当你修好一个因非法数据引起的 bug 时，只在一处加校验会让人觉得「够用了」。但那一处校验可能被其他代码路径、refactor 或 mock 绕过。

**核心原则**：在数据流经的每一层都做校验。让这个 bug 在结构上不可能发生。

## 为什么要多层

单层校验：「我们修好了这个 bug」
多层校验：「我们让这个 bug 不可能出现」

不同的层能拦住不同的情况：

- 入口校验可以拦住大多数 bug
- 业务逻辑校验可以拦住边界情况
- 环境守卫可以阻止上下文相关的危险操作
- 调试日志可以在其他层都失效时提供线索

## 四层防御

### Layer 1：入口点校验

**目的**：在 API 边界直接拒绝明显非法的输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### Layer 2：业务逻辑校验

**目的**：确保数据对当前操作而言是合理的

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### Layer 3：环境守卫

**目的**：在特定上下文中阻止危险操作

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### Layer 4：调试埋点

**目的**：为事后排查捕获上下文

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 应用这个 pattern

找到 bug 时：

1. **追踪数据流**——坏值从哪里产生？在哪里被用？
2. **列出所有检查点**——数据流经的每一个点
3. **在每一层加校验**——入口、业务、环境、调试
4. **逐层测试**——尝试绕过 Layer 1，验证 Layer 2 能拦住

## 来自真实会话的示例

Bug：空的 `projectDir` 导致 `git init` 运行在源码目录

**数据流**：

1. 测试 setup → 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 执行

**加入的四层防御**：

- Layer 1：`Project.create()` 校验目录非空/存在/可写
- Layer 2：`WorkspaceManager` 校验 projectDir 非空
- Layer 3：`WorktreeManager` 在测试中拒绝在 tmpdir 之外执行 git init
- Layer 4：在 git init 前记录 stack trace

**结果**：1847 个测试全通过，bug 无法复现。

## 关键洞察

四层防御全部都是必要的。测试过程中，每一层都拦住了其他层漏掉的 bug：

- 不同的代码路径绕过了入口校验
- mock 绕过了业务逻辑检查
- 不同平台上的边界情况需要环境守卫
- 调试日志发现了结构性误用

**不要止步于一处校验。** 在每一层都加上检查。
