# S08：提示工程 — Prompt 不是写的，是构建的

> **格言**："Prompt 不是写的，是构建的"

[← 上一章](s07-crafted-tools.md) | [下一章 →](s09-claude-code-compat.md)

---

## 问题：500 行的 Sisyphus prompt 是怎么生成的？

不是手写的。它是通过 `dynamic-agent-prompt-builder.ts` **动态组装**的。

## 动态 Prompt 构建

```typescript
// src/agents/sisyphus.ts:L10-L24
function buildDynamicSisyphusPrompt(
  availableAgents: AvailableAgent[],
  availableTools: AvailableTool[] = [],
  availableSkills: AvailableSkill[] = [],
  availableCategories: AvailableCategory[] = []
): string {
  const keyTriggers = buildKeyTriggersSection(availableAgents, availableSkills)
  const toolSelection = buildToolSelectionTable(availableAgents, availableTools, availableSkills)
  const exploreSection = buildExploreSection(availableAgents)
  const librarianSection = buildLibrarianSection(availableAgents)
  const categorySkillsGuide = buildCategorySkillsDelegationGuide(availableCategories, availableSkills)
  const delegationTable = buildDelegationTable(availableAgents)
  const oracleSection = buildOracleSection(availableAgents)
  const hardBlocks = buildHardBlocksSection()
  const antiPatterns = buildAntiPatternsSection()

  return `<Role>...\n${keyTriggers}\n...${toolSelection}\n...`
}
```

## Prompt 组装架构

```
AgentPromptMetadata (每个 Agent 导出)
    │
    ├── keyTrigger    ──→ buildKeyTriggersSection()
    ├── cost          ──→ buildToolSelectionTable()
    ├── triggers      ──→ buildDelegationTable()
    ├── useWhen       ──→ 各 Agent 专属 section
    └── avoidWhen     ──→ 各 Agent 专属 section
    │
    ▼
buildDynamicSisyphusPrompt()
    │
    ▼
Sisyphus 的完整 system prompt
```

## AvailableAgent 接口

```typescript
// src/agents/dynamic-agent-prompt-builder.ts:L3-L7
export interface AvailableAgent {
  name: BuiltinAgentName
  description: string
  metadata: AgentPromptMetadata
}
```

## 工具分类

```typescript
// src/agents/dynamic-agent-prompt-builder.ts:L14-L25
export function categorizeTools(toolNames: string[]): AvailableTool[] {
  return toolNames.map((name) => {
    let category: AvailableTool["category"] = "other"
    if (name.startsWith("lsp_")) category = "lsp"
    else if (name.startsWith("ast_grep")) category = "ast"
    else if (name === "grep" || name === "glob") category = "search"
    else if (name.startsWith("session_")) category = "session"
    else if (name === "slashcommand") category = "command"
    return { name, category }
  })
}
```

## Context Injector — 上下文注入系统

```typescript
// src/features/context-injector/index.ts
export { ContextCollector, contextCollector } from "./collector"
export { createContextInjectorMessagesTransformHook } from "./injector"
```

Context Injector 在 `experimental.chat.messages.transform` 阶段工作，将收集的上下文注入到消息流中。

## Prometheus 的 Prompt — 规划 Agent 的艺术

Prometheus 的 prompt 是 OMO 中最长的（~1000 行），包含：

```
Phase 1: Interview Mode (默认)
  ├── Intent Classification (Trivial/Refactoring/Build/Architecture/...)
  ├── Test Infrastructure Assessment
  └── Draft Management (.sisyphus/drafts/)

Phase 2: Plan Generation
  ├── Metis Consultation (强制)
  ├── Self-Review (Gap Classification)
  └── Momus Loop (可选高精度模式)

Phase 3: Handoff
  └── 引导用户运行 /start-work
```

核心约束：

```typescript
// src/agents/prometheus-prompt.ts
// "**YOU ARE A PLANNER. YOU ARE NOT AN IMPLEMENTER.**"
// "When user says 'fix X', interpret as 'create a work plan to fix X'"
// "You may ONLY create/edit markdown (.md) files"
```

## Atlas 的动态 Prompt

Atlas 的 prompt 也是动态构建的：

```typescript
// src/agents/atlas.ts:L130-L140
function buildDynamicOrchestratorPrompt(ctx?: OrchestratorContext): string {
  return ATLAS_SYSTEM_PROMPT
    .replace("{CATEGORY_SECTION}", categorySection)
    .replace("{AGENT_SECTION}", agentSection)
    .replace("{DECISION_MATRIX}", decisionMatrix)
    .replace("{SKILLS_SECTION}", skillsSection)
    .replace("{{CATEGORY_SKILLS_DELEGATION_GUIDE}}", categorySkillsGuide)
}
```

## 关键洞察

1. **Prompt 是代码生成的**：不是手写字符串，而是通过函数组装。
2. **添加 Agent 自动更新 Prompt**：新 Agent 只需导出 `AgentPromptMetadata`，Sisyphus 的 prompt 自动包含它。
3. **Prompt 是 API**：Prometheus 的 prompt 定义了一个完整的工作流——面试→研究→规划→审查→交付。
4. **Context Injection 是透明的**：上下文在消息传输层被注入，Agent 感知不到。

---

[← 上一章：工具集](s07-crafted-tools.md) | [下一章：Claude Code 兼容层 →](s09-claude-code-compat.md)
