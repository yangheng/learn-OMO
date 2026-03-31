# S06: Background Agents

> **Motto**: "tmux is the poor man's container orchestration"

[← Previous](s05-hook-system.md) | [Next →](s07-crafted-tools.md)

---

## Two-Layer Parallel Architecture

**BackgroundManager** (logical): Task queuing, concurrency control per key, fire-and-forget launch.

```typescript
// src/features/background-agent/manager.ts
class BackgroundManager {
  private tasks: Map<string, BackgroundTask>
  private queuesByKey: Map<string, QueueItem[]>
  private concurrencyManager: ConcurrencyManager

  async launch(input): Promise<BackgroundTask> {
    const task = { id: `bg_${crypto.randomUUID().slice(0, 8)}`, status: "pending", ... }
    // Queue by concurrency key, process asynchronously
  }
}
```

**TmuxSessionManager** (physical): Spawns tmux panes for child sessions, polls status, auto-closes idle panes.

```typescript
// src/features/tmux-subagent/manager.ts
class TmuxSessionManager {
  constructor(ctx, tmuxConfig) {
    this.enabled = tmuxConfig.enabled && isInsideTmux()
  }
  async onSessionCreated(event) {
    if (!event.parentID) return  // Only child sessions get panes
    await spawnTmuxPane(event.sessionID, event.title, this.config, this.serverUrl)
  }
}
```

tmux is optional—background tasks work without it, just without visible panes.

---

[← Previous: Hook System](s05-hook-system.md) | [Next: Crafted Tools →](s07-crafted-tools.md)
