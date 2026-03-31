# S02: Multi-Agent System — Each Agent is Named After a God for a Reason

> **Motto**: "Each agent is named after a god for a reason"

[← Previous](s01-plugin-architecture.md) | [Next →](s03-sisyphus-discipline-agent.md)

---

## The Roster

```typescript
// src/agents/types.ts
export type BuiltinAgentName =
  | "sisyphus" | "oracle" | "librarian" | "explore"
  | "multimodal-looker" | "metis" | "momus" | "atlas"
```

| Agent | Role | Category | Cost | Mode |
|-------|------|----------|------|------|
| **Sisyphus** | Primary orchestrator | specialist | — | primary |
| **Oracle** | Read-only architecture advisor | advisor | EXPENSIVE | subagent |
| **Librarian** | External docs/code researcher | exploration | CHEAP | subagent |
| **Explore** | Internal codebase search | exploration | FREE | subagent |
| **Atlas** | Todo list orchestrator | specialist | EXPENSIVE | primary |
| **Prometheus** | Interview→Plan consultant | specialist | — | primary |
| **Momus** | Ruthless plan reviewer | advisor | EXPENSIVE | subagent |
| **Metis** | Pre-planning intent classifier | advisor | EXPENSIVE | subagent |
| **Sisyphus-Junior** | Focused task executor | specialist | — | subagent |
| **Multimodal Looker** | PDF/image analyzer | utility | CHEAP | subagent |

## Permission Model

Each agent has tool restrictions via `createAgentToolRestrictions()`:

- **Oracle**: Cannot write, edit, delegate — read-only consultation
- **Explore/Librarian**: Cannot write, edit, delegate, or call agents — pure search
- **Sisyphus-Junior**: Cannot use `task` or `delegate_task` (prevents recursive delegation)

## PromptMetadata System

Each agent exports `AgentPromptMetadata` which is used to dynamically generate Sisyphus's prompt:

```typescript
// src/agents/types.ts:L20-L36
export interface AgentPromptMetadata {
  category: AgentCategory
  cost: AgentCost
  triggers: DelegationTrigger[]
  useWhen?: string[]
  avoidWhen?: string[]
  keyTrigger?: string
}
```

Adding a new agent automatically updates Sisyphus's delegation rules.

---

[← Previous: Plugin Architecture](s01-plugin-architecture.md) | [Next: Sisyphus →](s03-sisyphus-discipline-agent.md)
