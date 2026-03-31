# S05：Hook 系统 — Hook 是 OMO 的神经系统

> **格言**："Hook 是 OMO 的神经系统"

[← 上一章](s04-agent-delegation.md) | [下一章 →](s06-background-agents.md)

---

## 问题：如何在不修改 Agent 的情况下改变它的行为？

答案是 **Hook**——在事件发生时自动触发的行为修改器。OMO 有 30+ 个 Hook。

## Hook 架构

```
OpenCode 事件流
    │
    ▼
┌───────────────────────────────┐
│         event handler          │
│  ├── session.created           │
│  ├── session.deleted           │
│  ├── session.error             │
│  └── message.updated           │
├───────────────────────────────┤
│      chat.message handler      │
│  ├── keywordDetector           │
│  ├── claudeCodeHooks           │
│  ├── autoSlashCommand          │
│  ├── startWork                 │
│  └── ralphLoop (检测启动/取消)  │
├───────────────────────────────┤
│    tool.execute.before handler │
│  ├── questionLabelTruncator    │
│  ├── commentChecker            │
│  ├── rulesInjector             │
│  ├── prometheusMdOnly          │
│  └── sisyphusJuniorNotepad     │
├───────────────────────────────┤
│    tool.execute.after handler  │
│  ├── toolOutputTruncator       │
│  ├── commentChecker            │
│  ├── editErrorRecovery  ←────── 核心！
│  ├── delegateTaskRetry         │
│  ├── emptyTaskResponseDetector │
│  └── taskResumeInfo            │
└───────────────────────────────┘
```

## Hook 注册模式

所有 Hook 遵循统一模式：

```typescript
// src/index.ts 中的注册
const editErrorRecovery = isHookEnabled("edit-error-recovery")
  ? createEditErrorRecoveryHook(ctx)
  : null;

// tool.execute.after 中的调用
await editErrorRecovery?.["tool.execute.after"](input, output);
```

`?.` 可选链保证了被禁用的 Hook 不会执行。

## 核心 Hook 详解

### Edit Error Recovery — 编辑错误自动恢复

```typescript
// src/hooks/edit-error-recovery/index.ts:L7-L11
export const EDIT_ERROR_PATTERNS = [
  "oldString and newString must be different",
  "oldString not found",
  "oldString found multiple times",
] as const

export const EDIT_ERROR_REMINDER = `
[EDIT ERROR - IMMEDIATE ACTION REQUIRED]
You made an Edit mistake. STOP and do this NOW:
1. READ the file immediately to see its ACTUAL current state
2. VERIFY what the content really looks like
3. CONTINUE with corrected action based on the real file content
`
```

当 `edit` 工具失败时，Hook 在输出后追加恢复指令，强制 Agent 先读取文件再重试。

### Think Mode — 扩展思考

```typescript
// src/hooks/think-mode/index.ts
export function createThinkModeHook() {
  // 监听事件，检测 "think" 关键词
  // 动态切换 Agent 的 thinking 配置
}
```

### Comment Checker — 代码注释质量检查

在 `tool.execute.before` 和 `tool.execute.after` 都有 Hook，确保代码中不包含低质量注释。

### Rules Injector — 规则注入

自动注入项目级和用户级规则到 Agent 上下文。

### Session Recovery — 会话崩溃恢复

```typescript
// src/hooks/session-recovery/index.ts
// 检测可恢复的错误类型：
// - tool_result_missing
// - thinking_block_order
// - thinking_disabled_violation
// 自动修复消息并恢复 session
```

### Anthropic Context Window Limit Recovery

当 Anthropic API 报告上下文窗口溢出时，自动恢复。

### Delegate Task Retry — 委派重试

委派失败时自动重试。

### Auto Slash Command — 自动斜杠命令检测

```typescript
// src/hooks/auto-slash-command/index.ts
createAutoSlashCommandHook({ skills: mergedSkills })
```

检测用户输入的斜杠命令并自动转发。

### Background Notification — 后台任务通知

当后台任务完成时通知主 session。

### Atlas Hook — 知识注入

```typescript
// src/hooks/atlas/index.ts
createAtlasHook(ctx, { directory: ctx.directory, backgroundManager })
```

Atlas Agent 的状态管理和知识注入。

## Hook 的执行顺序

在 `tool.execute.after` 中：

```typescript
// src/index.ts:L280-L295
await claudeCodeHooks["tool.execute.after"](input, output);
await toolOutputTruncator?.["tool.execute.after"](input, output);
await contextWindowMonitor?.["tool.execute.after"](input, output);
await commentChecker?.["tool.execute.after"](input, output);
await editErrorRecovery?.["tool.execute.after"](input, output);
await delegateTaskRetry?.["tool.execute.after"](input, output);
await taskResumeInfo["tool.execute.after"](input, output);
```

顺序是固定的：先截断输出，再检查错误，再注入恢复指令。

## Hook 通信模式

Hook 之间通过修改 `output` 对象通信：

```typescript
// editErrorRecovery 直接修改 output.output
if (hasEditError) {
  output.output += `\n${EDIT_ERROR_REMINDER}`
}
```

这是一个**管道模式**：每个 Hook 接收前一个 Hook 修改后的 output。

## 关键洞察

1. **Hook 是行为修改器**：不改变 Agent 的 prompt，而是在运行时修改输入/输出。
2. **可选性设计**：每个 Hook 都可以独立禁用，不影响其他功能。
3. **管道模式**：Hook 按顺序执行，每个 Hook 可以修改 output 传递给下一个。
4. **恢复 > 失败**：OMO 的核心哲学——能恢复就不失败。

---

[← 上一章：Agent 委派机制](s04-agent-delegation.md) | [下一章：后台 Agent →](s06-background-agents.md)
