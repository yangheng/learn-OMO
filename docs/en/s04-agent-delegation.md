# S04: Agent Delegation

> **Motto**: "A Sisyphus that doesn't delegate is just an agent"

[← Previous](s03-sisyphus-discipline-agent.md) | [Next →](s05-hook-system.md)

---

## Two Delegation Tools

**`call_omo_agent`** — Lightweight, explore/librarian only, supports background execution.

**`delegate_task`** — Full-featured, supports categories (spawn Sisyphus-Junior) or direct agent types, skill loading, session continuity.

```typescript
// delegate_task supports:
delegate_task(
  category="quick",             // OR subagent_type="oracle" (mutually exclusive)
  load_skills=["playwright"],   // Skills prepended to subagent prompt
  run_in_background=true,       // Async execution
  session_id="ses_abc123",      // Resume existing session (saves 70%+ tokens)
  prompt="..."
)
```

## Category System

Categories are templates for Sisyphus-Junior with preset temperature/model:

```typescript
// src/tools/delegate-task/constants.ts
DEFAULT_CATEGORIES = {
  quick: { temperature: 0.1, model: "anthropic/claude-sonnet-4-5" },
  "visual-engineering": { temperature: 0.3 },
  ultrabrain: { temperature: 0.1 },
}
```

## 6-Section Prompt Structure (Mandatory)

Every delegation must include: TASK, EXPECTED OUTCOME, REQUIRED TOOLS, MUST DO, MUST NOT DO, CONTEXT.

## Session Continuity

Every `delegate_task()` returns a `session_id`. Resuming preserves full conversation context—no repeated file reads or setup.

---

[← Previous: Sisyphus](s03-sisyphus-discipline-agent.md) | [Next: Hook System →](s05-hook-system.md)
