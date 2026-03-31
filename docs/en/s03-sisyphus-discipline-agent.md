# S03: Sisyphus — The Discipline Agent

> **Motto**: "Humans roll their boulder every day. So do you."

[← Previous](s02-multi-agent-system.md) | [Next →](s04-agent-delegation.md)

---

## Identity

```typescript
// src/agents/sisyphus.ts (prompt)
// "SF Bay Area engineer. Work, delegate, verify, ship. No AI slop."
// "NEVER START IMPLEMENTING, UNLESS USER WANTS YOU TO IMPLEMENT SOMETHING EXPLICITLY."
```

## Three-Phase Workflow

**Phase 0 — Intent Gate** (every message): Classify → Check ambiguity → Validate delegation check. Default bias: **DELEGATE**.

**Phase 1 — Codebase Assessment** (open-ended tasks): Classify codebase state as Disciplined/Transitional/Legacy/Greenfield.

**Phase 2 — Execute**: Explore → Implement → Verify. On 3 consecutive failures: STOP → REVERT → CONSULT Oracle → ASK USER.

**Phase 3 — Completion**: All TODOs done + diagnostics clean + build passes.

## TODO Discipline (Non-Negotiable)

```
- 2+ steps → todowrite FIRST
- Mark in_progress before starting (ONE at a time)  
- Mark completed IMMEDIATELY after each step
- FAILURE TO USE TODOS = INCOMPLETE WORK.
```

## Dynamic Prompt Generation

```typescript
// src/agents/sisyphus.ts:L10-L24
function buildDynamicSisyphusPrompt(availableAgents, availableTools, availableSkills, availableCategories) {
  // Assembles ~500 line prompt from:
  // - Key triggers section (from agent metadata)
  // - Tool selection table
  // - Delegation table
  // - Oracle section
  // - Hard blocks & anti-patterns
}
```

## Boulder State

Progress persisted in `.sisyphus/` via `readBoulderState()`/`writeBoulderState()`. Survives session restarts.

---

[← Previous: Multi-Agent System](s02-multi-agent-system.md) | [Next: Agent Delegation →](s04-agent-delegation.md)
