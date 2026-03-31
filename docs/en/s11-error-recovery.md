# S11: Error Recovery & Resilience

> **Motto**: "A crash isn't the end, it's a restart"

[← Previous](s10-skill-mcp-management.md) | [Next →](s12-ralph-loop.md)

---

## Four Recovery Hooks

| Hook | Trigger | Action |
|------|---------|--------|
| **Edit Error Recovery** | edit tool returns known error patterns | Appends "READ the file first" reminder |
| **Session Recovery** | tool_result_missing, thinking_block_order | Fixes messages + auto-resumes |
| **Context Window Limit** | Anthropic context overflow | Auto-compaction/recovery |
| **Delegate Task Retry** | Delegation failure | Auto-retry logic |

## Edit Error Recovery — The Most Common Fix

```typescript
// src/hooks/edit-error-recovery/index.ts
const EDIT_ERROR_PATTERNS = [
  "oldString and newString must be different",
  "oldString not found",
  "oldString found multiple times",
]
// Detection: if output contains pattern → append recovery instructions
if (hasEditError) {
  output.output += "\n[EDIT ERROR - READ the file immediately, VERIFY, then CONTINUE]"
}
```

## Session Recovery

```typescript
// src/index.ts (session.error handler)
if (sessionRecovery?.isRecoverableError(error)) {
  const recovered = await sessionRecovery.handleSessionRecovery(messageInfo)
  if (recovered && sessionID === getMainSessionID()) {
    await ctx.client.session.prompt({ ..., body: { parts: [{ type: "text", text: "continue" }] } })
  }
}
```

## Recovery ↔ Ralph Loop Coordination

```typescript
sessionRecovery.setOnAbortCallback(todoContinuationEnforcer.markRecovering)
sessionRecovery.setOnRecoveryCompleteCallback(todoContinuationEnforcer.markRecoveryComplete)
```

Recovery pauses todo enforcement during session repair.

---

[← Previous: Skill & MCP Management](s10-skill-mcp-management.md) | [Next: The Ralph Loop →](s12-ralph-loop.md)
