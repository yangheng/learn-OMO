# S04：Agent 委派机制

> **格言**："不委派的 Sisyphus 只是一个普通 Agent"

[← 上一章](s03-sisyphus-discipline-agent.md) | [下一章 →](s05-hook-system.md)

---

## 问题：Agent 之间如何通信和分工？

OMO 提供了两个核心工具：`call_omo_agent`（轻量级调用）和 `delegate_task`（全功能委派）。

## 双轨委派架构

```
                  Sisyphus (主 Agent)
                       │
          ┌────────────┼────────────┐
          │                         │
  call_omo_agent              delegate_task
  (探索类专用)                (通用委派)
          │                         │
    ┌─────┴─────┐           ┌──────┴──────┐
    │ explore   │           │  category   │
    │ librarian │           │  (quick,    │
    └───────────┘           │  visual-    │
                            │  engineering│
                            │  ultrabrain)│
                            ├─────────────┤
                            │ subagent_   │
                            │ type        │
                            │ (oracle,    │
                            │  momus,     │
                            │  metis)     │
                            └─────────────┘
```

## call_omo_agent — 轻量级探索

```typescript
// src/tools/call-omo-agent/tools.ts:L42-L55
export function createCallOmoAgent(ctx, backgroundManager): ToolDefinition {
  return tool({
    description,
    args: {
      prompt: tool.schema.string(),
      subagent_type: tool.schema.string(),  // explore 或 librarian only
      run_in_background: tool.schema.boolean(),
      session_id: tool.schema.string().optional(),
    },
    async execute(args, toolContext) {
      if (!includesCaseInsensitive([...ALLOWED_AGENTS], args.subagent_type)) {
        return `Error: Invalid agent type. Only ${ALLOWED_AGENTS.join(", ")} are allowed.`
      }
    }
  })
}
```

**限制**：只允许 `explore` 和 `librarian`。这是故意的——探索类 Agent 是免费的、安全的。

## delegate_task — 全功能委派

```typescript
// src/tools/delegate-task/tools.ts
// 支持两种互斥模式：
// 1. category 模式 — 生成 Sisyphus-Junior 带特定配置
// 2. subagent_type 模式 — 直接调用专家 Agent

delegate_task(
  category="quick",           // 或 subagent_type="oracle"
  load_skills=["playwright"], // 加载的技能
  run_in_background=true,     // 是否后台运行
  prompt="...",               // 任务描述
  session_id="ses_abc123",    // 恢复现有 session
)
```

## Category 系统

`delegate_task` 的 category 模式会生成一个 **Sisyphus-Junior** 实例，带有类别特定的配置：

```typescript
// src/tools/delegate-task/constants.ts
export const DEFAULT_CATEGORIES = {
  quick: { temperature: 0.1, model: "anthropic/claude-sonnet-4-5" },
  "visual-engineering": { temperature: 0.3 },
  ultrabrain: { temperature: 0.1 },
  // ...
}
```

每个 category 有自己的 `temperature`、`model`、`prompt_append`。用户可以在配置中自定义 category。

## Skill 加载

委派时可以通过 `load_skills` 参数给子 Agent 注入技能：

```typescript
// src/tools/delegate-task/tools.ts (buildSystemContent)
export function buildSystemContent(input: BuildSystemContentInput): string | undefined {
  const { skillContent, categoryPromptAppend } = input
  if (skillContent && categoryPromptAppend) {
    return `${skillContent}\n\n${categoryPromptAppend}`
  }
  return skillContent || categoryPromptAppend
}
```

Skill 内容被**前置**到子 Agent 的 system prompt 中。

## 6 Section Prompt 结构

Sisyphus/Atlas 委派时必须遵循的 prompt 结构：

```
1. TASK: 原子化的具体目标
2. EXPECTED OUTCOME: 可验证的交付物
3. REQUIRED TOOLS: 工具白名单
4. MUST DO: 详尽的要求
5. MUST NOT DO: 禁止的行为
6. CONTEXT: 文件路径、模式、约束
```

这个结构确保子 Agent 有足够的上下文来完成任务，而不会越界。

## Session Continuity

每次 `delegate_task` 返回时都包含 `session_id`。恢复现有 session 时：

```typescript
// 子 Agent 已经读过所有文件、知道上下文
delegate_task(
  session_id="ses_xyz789",
  prompt="Fix: Type error on line 42"
)
// 比重新开始节省 70%+ token
```

## BackgroundManager 的角色

后台任务通过 `BackgroundManager` 管理：

```typescript
// src/features/background-agent/manager.ts:L60-L75
export class BackgroundManager {
  private tasks: Map<string, BackgroundTask>
  private concurrencyManager: ConcurrencyManager

  async launch(input: LaunchInput): Promise<BackgroundTask> {
    const task: BackgroundTask = {
      id: `bg_${crypto.randomUUID().slice(0, 8)}`,
      status: "pending",
      // ...
    }
    this.tasks.set(task.id, task)
    // 加入队列，按 concurrency key 处理
  }
}
```

## 关键洞察

1. **双轨设计**：`call_omo_agent` 用于安全的探索任务，`delegate_task` 用于需要写代码的任务。
2. **Category 是模板**：`delegate_task(category="quick")` 本质上是用预设配置创建 Sisyphus-Junior。
3. **Session 是黄金**：恢复 session 而不是重新开始，是 OMO 节省 token 的核心策略。
4. **6 Section Prompt 是合约**：确保委派者和被委派者之间有清晰的接口。

---

[← 上一章：Sisyphus 纪律之神](s03-sisyphus-discipline-agent.md) | [下一章：Hook 系统 →](s05-hook-system.md)
