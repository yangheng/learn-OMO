# S06：后台 Agent — tmux 是穷人的容器编排

> **格言**："tmux 是穷人的容器编排"

[← 上一章](s05-hook-system.md) | [下一章 →](s07-crafted-tools.md)

---

## 问题：如何让多个 Agent 真正并行工作？

OpenCode 本身是单 session 的。OMO 通过 `BackgroundManager` + `TmuxSessionManager` 实现了真正的异步并行。

## 双层并行架构

```
┌─────────────────────────────────────┐
│         BackgroundManager           │
│  (逻辑层 — 任务队列 + 并发控制)      │
│                                     │
│  tasks: Map<id, BackgroundTask>     │
│  queuesByKey: Map<key, QueueItem[]> │
│  concurrencyManager: ConcurrencyMgr│
├─────────────────────────────────────┤
│       TmuxSessionManager            │
│  (物理层 — tmux 窗格管理)            │
│                                     │
│  sessions: Map<id, TrackedSession>  │
│  每个子 session → 一个 tmux pane    │
└─────────────────────────────────────┘
```

## BackgroundManager

```typescript
// src/features/background-agent/manager.ts:L56-L68
export class BackgroundManager {
  private tasks: Map<string, BackgroundTask>
  private queuesByKey: Map<string, QueueItem[]> = new Map()
  private processingKeys: Set<string> = new Set()
  private concurrencyManager: ConcurrencyManager

  async launch(input: LaunchInput): Promise<BackgroundTask> {
    const task: BackgroundTask = {
      id: `bg_${crypto.randomUUID().slice(0, 8)}`,
      status: "pending",
      queuedAt: new Date(),
      agent: input.agent,
      parentSessionID: input.parentSessionID,
    }
    this.tasks.set(task.id, task)

    // 加入队列
    const key = this.getConcurrencyKeyFromInput(input)
    const queue = this.queuesByKey.get(key) ?? []
    queue.push({ task, input })
    this.queuesByKey.set(key, queue)

    // 触发处理（fire-and-forget）
    this.processKey(key)
    return task
  }
}
```

### 并发控制

```typescript
// processKey — 按 key 串行处理队列
private async processKey(key: string): Promise<void> {
  if (this.processingKeys.has(key)) return  // 已在处理中
  this.processingKeys.add(key)
  try {
    const queue = this.queuesByKey.get(key)
    while (queue && queue.length > 0) {
      await this.concurrencyManager.acquire(key)  // 等待并发槽
      await this.startTask(item)
      queue.shift()
    }
  } finally {
    this.processingKeys.delete(key)
  }
}
```

**Concurrency Key** 是按 Agent 类型分组的——同类型的任务在同一个队列中串行，不同类型可并行。

## TmuxSessionManager

```typescript
// src/features/tmux-subagent/manager.ts:L12-L28
export class TmuxSessionManager {
  private enabled: boolean
  private sessions: Map<string, TrackedSession>

  constructor(ctx: PluginInput, tmuxConfig: TmuxConfig) {
    this.enabled = tmuxConfig.enabled && isInsideTmux()
    if (this.enabled) this.startPolling()
  }

  async onSessionCreated(event: { sessionID, parentID?, title }) {
    if (!this.enabled || !event.parentID) return  // 只处理子 session
    const result = await spawnTmuxPane(event.sessionID, event.title, this.config, this.serverUrl)
    if (result.success && result.paneId) {
      this.sessions.set(event.sessionID, { ... })
    }
  }
}
```

### tmux 窗格生命周期

```
子 Session 创建 → spawnTmuxPane() → 新 tmux pane 打开
     │
     ├── 每 N ms 轮询状态 (pollSessions)
     │     ├── session 存活 → 更新 lastSeenAt
     │     ├── session idle → closeTmuxPane()
     │     └── session 消失 > grace period → closeTmuxPane()
     │
子 Session 删除 → closeTmuxPane() → tmux pane 关闭
```

## 后台任务的用户可见性

```typescript
// TaskToastManager — 在 OpenCode TUI 中显示 toast
const toastManager = getTaskToastManager()
if (toastManager) {
  toastManager.addTask({
    id: task.id,
    description: input.description,
    agent: input.agent,
    isBackground: true,
    status: "queued",
  })
}
```

## 进程清理

```typescript
// BackgroundManager 注册进程退出清理
private registerProcessCleanup() {
  // 监听 SIGINT, SIGTERM, beforeExit, exit
  // 清理所有运行中的后台任务
}
```

## Explore/Librarian 的并行模式

Sisyphus 的 prompt 强制要求探索类 Agent 始终后台运行：

```
// CORRECT: Always background, always parallel
delegate_task(subagent_type="explore", run_in_background=true, ...)
delegate_task(subagent_type="librarian", run_in_background=true, ...)
// Continue working immediately. Collect with background_output when needed.

// WRONG: Never wait synchronously for explore/librarian
result = delegate_task(..., run_in_background=false)
```

## 关键洞察

1. **火然后忘记**：后台任务启动后立即返回任务 ID，主 Agent 继续工作。
2. **并发 key 隔离**：同类 Agent 串行，不同类并行——避免资源竞争。
3. **tmux 是可选的**：`tmuxConfig.enabled` 可以关闭，后台任务仍然工作，只是没有可视化窗格。
4. **Grace Period**：tmux pane 在 session 消失后有宽限期，避免误杀。

---

[← 上一章：Hook 系统](s05-hook-system.md) | [下一章：工具集 →](s07-crafted-tools.md)
