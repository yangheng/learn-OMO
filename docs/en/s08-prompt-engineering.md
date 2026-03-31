# S08: Prompt Engineering

> **Motto**: "Prompts aren't written, they're built"

[← Previous](s07-crafted-tools.md) | [Next →](s09-claude-code-compat.md)

---

## Dynamic Prompt Assembly

Sisyphus's ~500 line prompt is **generated**, not hand-written:

```typescript
// src/agents/sisyphus.ts
function buildDynamicSisyphusPrompt(availableAgents, availableTools, availableSkills, availableCategories) {
  const keyTriggers = buildKeyTriggersSection(availableAgents, availableSkills)
  const toolSelection = buildToolSelectionTable(availableAgents, availableTools, availableSkills)
  const delegationTable = buildDelegationTable(availableAgents)
  // ...assembles complete prompt from sections
}
```

## How It Works

Each agent exports `AgentPromptMetadata` (category, cost, triggers, keyTrigger). The `dynamic-agent-prompt-builder.ts` reads these metadata objects and generates:

- **Key Triggers section**: "2+ modules → fire explore", "external library → fire librarian"
- **Tool Selection table**: Resources sorted by cost (FREE → CHEAP → EXPENSIVE)
- **Delegation table**: Domain → trigger → agent mapping

Adding a new agent = automatic prompt update for Sisyphus.

## Prometheus — The Longest Prompt (~1000 lines)

Prometheus defines a complete workflow in its prompt:
1. Interview Mode (intent classification, test infrastructure assessment)
2. Plan Generation (Metis consultation, self-review, gap classification)
3. Momus Loop (optional high-accuracy review)
4. Handoff (`/start-work`)

**Absolute constraint**: "YOU ARE A PLANNER. YOU DO NOT WRITE CODE."

## Context Injector

Works at `experimental.chat.messages.transform` phase, injecting collected context transparently.

---

[← Previous: Crafted Tools](s07-crafted-tools.md) | [Next: Claude Code Compatibility →](s09-claude-code-compat.md)
