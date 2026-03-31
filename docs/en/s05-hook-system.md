# S05: Hook System

> **Motto**: "Hooks are OMO's nervous system"

[← Previous](s04-agent-delegation.md) | [Next →](s06-background-agents.md)

---

## 30+ Hooks Across 4 Lifecycle Phases

| Phase | Key Hooks |
|-------|-----------|
| `event` | session-recovery, context-window-monitor, atlas, ralph-loop |
| `chat.message` | keyword-detector, auto-slash-command, ralph-loop start |
| `tool.execute.before` | comment-checker, rules-injector, prometheus-md-only |
| `tool.execute.after` | edit-error-recovery, delegate-task-retry, tool-output-truncator |

## Pattern: Conditional Creation + Pipeline

```typescript
// Each hook is independently disableable
const editErrorRecovery = isHookEnabled("edit-error-recovery")
  ? createEditErrorRecoveryHook(ctx) : null;

// Pipeline: hooks execute in fixed order, each modifying output
await editErrorRecovery?.["tool.execute.after"](input, output);
```

## Edit Error Recovery (Most Important Hook)

```typescript
// src/hooks/edit-error-recovery/index.ts
const EDIT_ERROR_PATTERNS = [
  "oldString and newString must be different",
  "oldString not found",
  "oldString found multiple times",
]
// When detected: appends "[EDIT ERROR - READ the file first]" to output
```

Forces agent to read the actual file state before retrying.

---

[← Previous: Agent Delegation](s04-agent-delegation.md) | [Next: Background Agents →](s06-background-agents.md)
