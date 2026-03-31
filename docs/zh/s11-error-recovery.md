# S11：错误恢复与韧性 — 崩溃不是结束，是重启的开始

> **格言**："崩溃不是结束，是重启的开始"

[← 上一章](s10-skill-mcp-management.md) | [下一章 →](s12-ralph-loop.md)

---

## 问题：AI Agent 必然会失败。问题是——然后呢？

OMO 的哲学：**能恢复就不失败**。4 个专门的恢复 Hook 覆盖了最常见的故障模式。

## 恢复 Hook 全景

```
┌─────────────────────────────────────────────────┐
│              OMO 错误恢复体系                     │
├──────────────────┬──────────────────────────────┤
│ Edit Error       │ Agent 编辑文件时犯错           │
│ Recovery         │ → 强制 Agent 先读取再重试      │
├──────────────────┼──────────────────────────────┤
│ Session          │ Session 崩溃                   │
│ Recovery         │ → 修复消息 + 自动恢复           │
├──────────────────┼──────────────────────────────┤
│ Context Window   │ Anthropic 上下文溢出           │
│ Limit Recovery   │ → 自动压缩/恢复                │
├──────────────────┼──────────────────────────────┤
│ Delegate Task    │ 委派任务失败                    │
│ Retry            │ → 自动重试                     │
└──────────────────┴──────────────────────────────┘
```

## 1. Edit Error Recovery — 最常见的失败

```typescript
// src/hooks/edit-error-recovery/index.ts:L7-L11
export const EDIT_ERROR_PATTERNS = [
  "oldString and newString must be different",
  "oldString not found",
  "oldString found multiple times",
] as const
```

当 `edit` 工具返回这些错误时，Hook 追加恢复指令：

```typescript
// src/hooks/edit-error-recovery/index.ts:L34-L43
export function createEditErrorRecoveryHook(_ctx: PluginInput) {
  return {
    "tool.execute.after": async (input, output) => {
      if (input.tool.toLowerCase() !== "edit") return

      const outputLower = output.output.toLowerCase()
      const hasEditError = EDIT_ERROR_PATTERNS.some((pattern) =>
        outputLower.includes(pattern.toLowerCase())
      )
      if (hasEditError) {
        output.output += `\n${EDIT_ERROR_REMINDER}`
      }
    },
  }
}
```

恢复指令的内容：

```
[EDIT ERROR - IMMEDIATE ACTION REQUIRED]
1. READ the file immediately to see its ACTUAL current state
2. VERIFY what the content really looks like
3. CONTINUE with corrected action based on the real file content

DO NOT attempt another edit until you've read and verified the file state.
```

这个 Hook 解决了一个反复出现的问题：Agent 假设文件内容是 X，但实际是 Y，然后盲目重试。

## 2. Session Recovery — 会话崩溃恢复

```typescript
// src/hooks/session-recovery/index.ts
// 可恢复的错误类型：
// - tool_result_missing: 工具结果丢失
// - thinking_block_order: thinking 块顺序错误
// - thinking_disabled_violation: thinking 被禁用但 Agent 仍然使用
```

恢复流程：

```
session.error 事件
    │
    ├── isRecoverableError(error)? → 检测错误类型
    │
    ├── handleSessionRecovery(messageInfo)
    │     ├── 修复消息中的问题（空消息、孤立thinking块等）
    │     └── 返回 recovered: boolean
    │
    └── if recovered && 是主 session:
          └── 自动发送 "continue" prompt 继续工作
```

```typescript
// src/index.ts:L250-L260
if (event.type === "session.error") {
  if (sessionRecovery?.isRecoverableError(error)) {
    const recovered = await sessionRecovery.handleSessionRecovery(messageInfo)
    if (recovered && sessionID === getMainSessionID()) {
      await ctx.client.session.prompt({
        path: { id: sessionID },
        body: { parts: [{ type: "text", text: "continue" }] },
      })
    }
  }
}
```

## 3. Context Window Limit Recovery

当 Anthropic API 报告上下文窗口超限：

```typescript
// src/hooks/anthropic-context-window-limit-recovery/index.ts
createAnthropicContextWindowLimitRecoveryHook(ctx, {
  experimental: pluginConfig.experimental,
})
```

## 4. Delegate Task Retry

```typescript
// src/hooks/delegate-task-retry/index.ts
createDelegateTaskRetryHook(ctx)
```

委派任务失败时，在 `tool.execute.after` 阶段自动注入重试逻辑。

## 恢复与 Ralph Loop 的协作

Session Recovery 和 Ralph Loop 之间有**回调连接**：

```typescript
// src/index.ts:L150-L155
if (sessionRecovery && todoContinuationEnforcer) {
  sessionRecovery.setOnAbortCallback(todoContinuationEnforcer.markRecovering)
  sessionRecovery.setOnRecoveryCompleteCallback(
    todoContinuationEnforcer.markRecoveryComplete
  )
}
```

当 Session Recovery 正在恢复时，它会通知 Todo Continuation Enforcer 暂停，避免在恢复过程中触发新的工作。

## Sisyphus 的失败协议

除了 Hook 级别的自动恢复，Sisyphus 的 prompt 也定义了显式的失败协议：

```
After 3 Consecutive Failures:
1. STOP all further edits immediately
2. REVERT to last known working state
3. DOCUMENT what was attempted and what failed
4. CONSULT Oracle with full failure context
5. If Oracle cannot resolve → ASK USER
```

## 关键洞察

1. **四层恢复**：Edit Error → Session → Context Window → Delegate Task，覆盖最常见的故障。
2. **恢复是透明的**：通过追加到 `output.output`，Agent 看到的是"继续指令"，不知道自己被恢复了。
3. **回调协调**：Recovery Hook 之间通过回调协调，避免恢复过程中产生新的故障。
4. **最后的防线是人类**：3 次失败后 → Oracle → 人类。OMO 知道自己的极限。

---

[← 上一章：Skill 与 MCP 管理](s10-skill-mcp-management.md) | [下一章：Ralph Loop →](s12-ralph-loop.md)
