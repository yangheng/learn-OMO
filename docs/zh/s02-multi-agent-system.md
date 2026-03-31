# S02：多 Agent 系统 — 每个 Agent 都以神命名，因为它们各有神力

> **格言**："每个 Agent 都以神命名，因为它们各有神力"

[← 上一章](s01-plugin-architecture.md) | [下一章 →](s03-sisyphus-discipline-agent.md)

---

## 问题：一个 Agent 怎么可能擅长所有事情？

答案是：不能。OMO 的解决方案是创建一个**专家团队**，每个成员有明确的职责、成本和限制。

## Agent 类型系统

```typescript
// src/agents/types.ts:L6-L9
export type AgentCategory = "exploration" | "specialist" | "advisor" | "utility"
export type AgentCost = "FREE" | "CHEAP" | "EXPENSIVE"
```

```typescript
// src/agents/types.ts:L37-L45
export type BuiltinAgentName =
  | "sisyphus"
  | "oracle"
  | "librarian"
  | "explore"
  | "multimodal-looker"
  | "metis"
  | "momus"
  | "atlas"
```

## Agent 全景图

```
┌─────────────────────────────────────────────────┐
│                Agent 分类体系                     │
├────────────┬────────────┬────────────┬───────────┤
│ exploration│ specialist │  advisor   │  utility  │
├────────────┼────────────┼────────────┼───────────┤
│ Explore    │ Sisyphus   │ Oracle     │Multimodal │
│ Librarian  │ Sisyphus-  │ Metis      │ Looker    │
│            │ Junior     │ Momus      │           │
│            │ Atlas      │            │           │
│            │ Prometheus │            │           │
├────────────┼────────────┼────────────┼───────────┤
│ 成本: FREE │ 成本: 变化  │ 成本:      │ 成本:     │
│ 并行友好   │            │ EXPENSIVE  │ CHEAP     │
└────────────┴────────────┴────────────┴───────────┘
```

## Agent 详解

### Sisyphus — 纪律之神 (Primary)

```typescript
// src/agents/sisyphus.ts:L162-L171
export function createSisyphusAgent(model, availableAgents?, ...): AgentConfig {
  return {
    description: "Sisyphus - Powerful AI orchestrator from OhMyOpenCode...",
    mode: "primary",
    model,
    maxTokens: 64000,
    prompt,
    color: "#00CED1",
    thinking: { type: "enabled", budgetTokens: 32000 },
  }
}
```

- **角色**：主 Agent，接收用户任务，决定委派还是自己做
- **模式**：`primary`（OpenCode UI 中直接对话的 Agent）
- **核心能力**：Intent 分类 → 代码库评估 → 委派/执行 → 验证

### Oracle — 架构顾问 (Advisor)

```typescript
// src/agents/oracle.ts:L37-L47
export function createOracleAgent(model: string): AgentConfig {
  const restrictions = createAgentToolRestrictions(["write", "edit", "task", "delegate_task"])
  return {
    description: "Read-only consultation agent. High-IQ reasoning specialist...",
    mode: "subagent",
    temperature: 0.1,
    thinking: { type: "enabled", budgetTokens: 32000 },
  }
}
```

- **角色**：只读顾问，不能写代码，专注架构决策
- **限制**：`write`、`edit`、`task`、`delegate_task` 全部被拒绝
- **触发条件**：2+ 次修复失败后、复杂架构决策、安全/性能审查

### Explore — 代码搜索专家 (Exploration)

```typescript
// src/agents/explore.ts:L27-L30
export const EXPLORE_PROMPT_METADATA: AgentPromptMetadata = {
  category: "exploration",
  cost: "FREE",
  keyTrigger: "2+ modules involved → fire `explore` background",
}
```

- **角色**：内部代码库的 grep 替代品
- **特点**：只读、并行友好、必须返回绝对路径

### Librarian — 文档研究员 (Exploration)

```typescript
// src/agents/librarian.ts:L15
keyTrigger: "External library/source mentioned → fire `librarian` background",
```

- **角色**：外部文档和开源代码的研究者
- **工具**：GitHub CLI、Context7、Web Search、grep.app
- **特点**：每个声明必须附带 GitHub permalink 作为证据

### Atlas — 编排大师 (Specialist)

```typescript
// src/agents/atlas.ts (ATLAS_SYSTEM_PROMPT)
// "You are Atlas - the Master Orchestrator from OhMyOpenCode.
//  In Greek mythology, Atlas holds up the celestial heavens.
//  You hold up the entire workflow"
```

- **角色**：执行 TODO 列表，通过 `delegate_task()` 编排所有工作
- **规则**：自己**绝不写代码**，只委派和验证
- **特色**：Notepad 系统 (`.sisyphus/notepads/`) 在无状态 Agent 间传递知识

### Prometheus — 规划顾问

- **角色**：面试用户 → 研究 → 生成工作计划
- **绝对约束**：只能写 `.md` 文件（通过 `prometheus-md-only` Hook 强制）
- **工作流**：Interview → Metis 预审 → Plan 生成 → Momus 审查 → `/start-work`

### Momus — 无情批评家 (Advisor)

- **角色**：审查工作计划，确保文档完整性
- **命名**：希腊讽刺之神，连众神的作品都要挑毛病
- **输出**：`OKAY`（通过）或 `REJECT`（附带具体问题）

### Metis — 智慧女神 (Advisor)

- **角色**：规划前的意图分类和风险分析
- **输出**：对 Prometheus 的指令（MUST DO / MUST NOT DO）

### Sisyphus-Junior — 轻量执行者

```typescript
// src/agents/sisyphus-junior.ts:L5-L8
// "Sisyphus-Junior - Focused executor from OhMyOpenCode.
//  Execute tasks directly. NEVER delegate or spawn other agents."
const BLOCKED_TOOLS = ["task", "delegate_task"]
```

- **角色**：被委派的执行者，专注完成单一任务
- **限制**：不能使用 `task` 和 `delegate_task`（防止递归委派）
- **可以**：使用 `call_omo_agent` 召唤 Explore/Librarian 做研究

## Agent 权限模型

```typescript
// src/shared/permission-compat.ts
export function createAgentToolRestrictions(blockedTools: string[]) {
  // 返回 permission 对象，将 blockedTools 设为 "deny"
}
```

| Agent | write | edit | task | delegate_task | call_omo_agent |
|-------|-------|------|------|---------------|----------------|
| Sisyphus | ✅ | ✅ | ✅ | ✅ | ❌ (deny) |
| Oracle | ❌ | ❌ | ❌ | ❌ | ✅ |
| Explore | ❌ | ❌ | ❌ | ❌ | ❌ |
| Librarian | ❌ | ❌ | ❌ | ❌ | ❌ |
| Sisyphus-Junior | ✅ | ✅ | ❌ | ❌ | ✅ |
| Atlas | ✅ | ✅ | ❌ | ✅ | ❌ |

## PromptMetadata 系统

每个 Agent 都导出 `AgentPromptMetadata`，让 Sisyphus 的 prompt 可以**动态生成**：

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

这意味着添加一个新 Agent 时，Sisyphus 的 prompt 会**自动更新**以包含新 Agent 的委派规则。

## 关键洞察

1. **Agent 即团队**：OMO 不是一个全能 Agent，而是一个分工明确的团队。
2. **权限即安全**：通过 `createAgentToolRestrictions` 严格限制每个 Agent 能做什么。
3. **成本感知**：FREE → CHEAP → EXPENSIVE 的分类让 Sisyphus 知道何时值得调用某个 Agent。

---

[← 上一章：插件架构](s01-plugin-architecture.md) | [下一章：Sisyphus 纪律之神 →](s03-sisyphus-discipline-agent.md)
