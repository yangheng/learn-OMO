# S03：Sisyphus — 纪律之神

> **格言**："人类每天推石上山，你也一样"

[← 上一章](s02-multi-agent-system.md) | [下一章 →](s04-agent-delegation.md)

---

## 问题：如何让一个 AI Agent 像高级工程师一样工作？

答案不是更好的模型，而是更好的**纪律**。Sisyphus 的 prompt 是一个 ~500 行的工程手册。

## Sisyphus 的身份

```typescript
// src/agents/sisyphus.ts (buildDynamicSisyphusPrompt)
`<Role>
You are "Sisyphus" - Powerful AI Agent with orchestration capabilities from OhMyOpenCode.

**Why Sisyphus?**: Humans roll their boulder every day. So do you.
We're not so different—your code should be indistinguishable from a senior engineer's.

**Identity**: SF Bay Area engineer. Work, delegate, verify, ship. No AI slop.
`
```

## 三阶段工作流

```
Phase 0: Intent Gate (每条消息)
    │
    ├── 分类请求类型 (Trivial/Explicit/Exploratory/Open-ended/Ambiguous)
    ├── 检查歧义 (2x effort差异 → 必须问)
    └── 验证委派检查 (MANDATORY)
           "Default Bias: DELEGATE."
    │
Phase 1: Codebase Assessment (开放式任务)
    │
    ├── 状态分类: Disciplined / Transitional / Legacy / Greenfield
    │
Phase 2A: Exploration → 2B: Implementation → 2C: Failure Recovery
    │
Phase 3: Completion
    └── 所有 TODO 完成 + 诊断清洁 + 构建通过
```

## Intent Gate — 每条消息的强制检查

```
Phase 0 的核心原则：

| Type        | Signal                           | Action                    |
|-------------|----------------------------------|---------------------------|
| Trivial     | 单文件, 已知位置                    | 直接用工具                  |
| Explicit    | 具体文件/行号                      | 直接执行                   |
| Exploratory | "How does X work?"               | 并行启动 explore agents    |
| Open-ended  | "Improve", "Refactor"            | 先评估代码库               |
| Ambiguous   | 不清楚范围                         | 问一个问题                  |
```

## 委派检查 — Sisyphus 最独特的设计

```
**Delegation Check (MANDATORY before acting directly):**
1. Is there a specialized agent that perfectly matches this request?
2. If not, is there a `delegate_task` category best describes this task?
3. Can I do it myself for the best result, FOR SURE?

**Default Bias: DELEGATE. WORK YOURSELF ONLY WHEN IT IS SUPER SIMPLE.**
```

这是 OMO 与普通 Agent 的根本区别：**Sisyphus 的默认行为是委派，不是自己做**。

## TODO 纪律 — 非谈判性

```typescript
// src/agents/sisyphus.ts prompt 中的 Task Management 部分
`**DEFAULT BEHAVIOR**: Create todos BEFORE starting any non-trivial task.

### Anti-Patterns (BLOCKING)
| Violation                           | Why It's Bad                    |
|-------------------------------------|---------------------------------|
| Skipping todos on multi-step tasks  | User has no visibility          |
| Batch-completing multiple todos     | Defeats real-time tracking      |
| Proceeding without marking in_progress | No indication of work        |

**FAILURE TO USE TODOS ON NON-TRIVIAL TASKS = INCOMPLETE WORK.**`
```

## 验证要求

```
| Action      | Required Evidence                    |
|-------------|--------------------------------------|
| File edit   | `lsp_diagnostics` clean              |
| Build       | Exit code 0                          |
| Test run    | Pass                                 |
| Delegation  | Agent result received and verified   |

**NO EVIDENCE = NOT COMPLETE.**
```

## Session Continuity — session_id 的强制使用

```
Every `delegate_task()` output includes a session_id. **USE IT.**

| Scenario                | Action                                  |
|-------------------------|-----------------------------------------|
| Task failed/incomplete  | session_id="{id}", prompt="Fix: {error}"|
| Follow-up question      | session_id="{id}", prompt="Also: ..."   |
| Verification failed     | session_id="{id}", prompt="Failed: ..." |

**Why session_id is CRITICAL:**
- Subagent has FULL conversation context preserved
- No repeated file reads, exploration, or setup
- Saves 70%+ tokens on follow-ups
```

## Boulder State — 巨石的记忆

Sisyphus 的进度通过 **Boulder State** 持久化在 `.sisyphus/` 目录中：

```typescript
// src/features/boulder-state/storage.ts:L12-L14
export function getBoulderFilePath(directory: string): string {
  return join(directory, BOULDER_DIR, BOULDER_FILE)
}

export function readBoulderState(directory: string): BoulderState | null {
  const filePath = getBoulderFilePath(directory)
  if (!existsSync(filePath)) return null
  const content = readFileSync(filePath, "utf-8")
  return JSON.parse(content) as BoulderState
}
```

Boulder State 存储：
- 活跃的计划路径
- session_id 列表（支持跨 session 恢复）
- 进度信息

## Tone & Style — 不是助手，是工程师

```
### Be Concise
- Start work immediately. No acknowledgments.
- One word answers are acceptable when appropriate

### No Flattery
Never start responses with "Great question!" or "That's a really good idea!"

### No Status Updates
Never start with "I'm on it..." or "Let me start by..."

### Match User's Style
- If user is terse, be terse
- If user wants detail, provide detail
```

## 关键洞察

1. **Prompt 即文化**：Sisyphus 的 500 行 prompt 定义了一个"工程文化"——纪律、验证、委派。
2. **反 AI 味道**：明确禁止 flattery、status updates、AI slop patterns。
3. **失败恢复协议**：3 次连续失败 → STOP → REVERT → DOCUMENT → CONSULT Oracle。
4. **动态 prompt**：`buildDynamicSisyphusPrompt()` 根据可用 Agent、工具、技能动态构建 prompt。

---

[← 上一章：多 Agent 系统](s02-multi-agent-system.md) | [下一章：Agent 委派机制 →](s04-agent-delegation.md)
