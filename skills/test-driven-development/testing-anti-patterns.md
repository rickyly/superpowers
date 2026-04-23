# 测试反模式

**在以下情况加载本参考：** 编写或修改测试、添加 mock，或想要在生产代码中加入仅用于测试的方法时。

## 概述

测试必须验证真实行为，而不是 mock 的行为。mock 是隔离的手段，不是被测的对象。

**核心原则：** 测代码做什么，不测 mock 做什么。

**严格遵循 TDD 可以防止这些反模式。**

## 铁律

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## 反模式 1：测 mock 的行为

**违例：**
```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么错：**
- 你验证的是 mock 能用，而不是组件能用
- 有 mock 时测试通过，没有 mock 时测试失败
- 对真实行为一无所知

**你的人类伙伴的纠正：** "我们是在测 mock 的行为吗？"

**修法：**
```typescript
// ✅ GOOD: Test real component or don't mock it
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// OR if sidebar must be mocked for isolation:
// Don't assert on the mock - test Page's behavior with sidebar present
```

### 守门函数

```
BEFORE asserting on any mock element:
  Ask: "Am I testing real component behavior or just mock existence?"

  IF testing mock existence:
    STOP - Delete the assertion or unmock the component

  Test real behavior instead
```

## 反模式 2：生产代码中的仅供测试方法

**违例：**
```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Looks like production API!
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// In tests
afterEach(() => session.destroy());
```

**为什么错：**
- 生产类被仅供测试的代码污染
- 如果在生产中被误调用，非常危险
- 违反 YAGNI 和关注点分离
- 混淆了对象生命周期和实体生命周期

**修法：**
```typescript
// ✅ GOOD: Test utilities handle test cleanup
// Session has no destroy() - it's stateless in production

// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// In tests
afterEach(() => cleanupSession(session));
```

### 守门函数

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?"

  IF yes:
    STOP - Don't add it
    Put it in test utilities instead

  Ask: "Does this class own this resource's lifecycle?"

  IF no:
    STOP - Wrong class for this method
```

## 反模式 3：不理解就 mock

**违例：**
```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  // Mock prevents config write that test depends on!
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // Should throw - but won't!
});
```

**为什么错：**
- 被 mock 的方法有测试所依赖的副作用（写配置）
- 为了"保险"而过度 mock 破坏了实际行为
- 测试由于错误的原因通过，或神秘地失败

**修法：**
```typescript
// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  // Mock the slow part, preserve behavior test needs
  vi.mock('MCPServerManager'); // Just mock slow server startup

  await addServer(config);  // Config written
  await addServer(config);  // Duplicate detected ✓
});
```

### 守门函数

```
BEFORE mocking any method:
  STOP - Don't mock yet

  1. Ask: "What side effects does the real method have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (the actual slow/external operation)
    OR use test doubles that preserve necessary behavior
    NOT the high-level method the test depends on

  IF unsure what test depends on:
    Run test with real implementation FIRST
    Observe what actually needs to happen
    THEN add minimal mocking at the right level

  Red flags:
    - "I'll mock this to be safe"
    - "This might be slow, better mock it"
    - Mocking without understanding the dependency chain
```

## 反模式 4：不完整的 mock

**违例：**
```typescript
// ❌ BAD: Partial mock - only fields you think you need
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code uses
};

// Later: breaks when code accesses response.metadata.requestId
```

**为什么错：**
- **部分 mock 掩盖了结构假设** —— 你只 mock 了你知道的字段
- **下游代码可能依赖你没加进去的字段** —— 静默失败
- **测试通过但集成失败** —— mock 不完整，真实 API 是完整的
- **错误的信心** —— 测试对真实行为什么也证明不了

**铁律：** mock 要与真实数据结构完全一致，而不是只 mock 你当前测试用到的字段。

**修法：**
```typescript
// ✅ GOOD: Mirror real API completeness
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // All fields real API returns
};
```

### 守门函数

```
BEFORE creating mock responses:
  Check: "What fields does the real API response contain?"

  Actions:
    1. Examine actual API response from docs/examples
    2. Include ALL fields system might consume downstream
    3. Verify mock matches real response schema completely

  Critical:
    If you're creating a mock, you must understand the ENTIRE structure
    Partial mocks fail silently when code depends on omitted fields

  If uncertain: Include all documented fields
```

## 反模式 5：集成测试作为事后补丁

**违例：**
```
✅ Implementation complete
❌ No tests written
"Ready for testing"
```

**为什么错：**
- 测试是实现的一部分，不是可选的后续工作
- TDD 本来会抓住这个问题
- 没测试就不能声称完成

**修法：**
```
TDD cycle:
1. Write failing test
2. Implement to pass
3. Refactor
4. THEN claim complete
```

## 当 mock 变得过于复杂

**警告信号：**
- mock 的 setup 比测试逻辑更长
- 为了让测试通过而 mock 一切
- mock 缺少真实组件有的方法
- mock 一改，测试就坏

**你的人类伙伴的提问：** "我们这里需要用 mock 吗？"

**考虑：** 用真实组件做集成测试往往比复杂的 mock 更简单

## TDD 如何防止这些反模式

**为什么 TDD 有用：**
1. **先写测试** → 迫使你思考你到底在测什么
2. **看它失败** → 确认测试测的是真实行为，不是 mock
3. **最少实现** → 仅供测试的方法不会溜进来
4. **真实依赖** → 在 mock 之前你看到测试实际需要什么

**如果你在测 mock 的行为，你就违反了 TDD** —— 你加了 mock，却没有先让测试针对真实代码失败过。

## 速查

| 反模式 | 修法 |
|--------------|-----|
| 对 mock 元素做断言 | 测真实组件或不 mock |
| 生产代码中的仅供测试方法 | 移到测试工具 |
| 不理解就 mock | 先理解依赖，最少 mock |
| 不完整的 mock | 完整镜像真实 API |
| 测试作为事后补丁 | TDD —— 先写测试 |
| 过度复杂的 mock | 考虑集成测试 |

## 危险信号

- 断言里检查 `*-mock` 的 test ID
- 只在测试文件里调用的方法
- mock 的 setup 占测试的 50% 以上
- 移掉 mock 测试就失败
- 说不清为什么需要 mock
- "为了保险先 mock 一下"

## 底线

**mock 是用来隔离的工具，不是被测的对象。**

如果 TDD 让你发现自己在测 mock 的行为，说明你走错了方向。

修法：测真实行为，或质疑自己到底为什么要 mock。
