# S01: Plugin Architecture — It All Starts with a Plugin Interface

> **Motto**: "It all starts with a Plugin interface"

[Next →](s02-multi-agent-system.md)

---

## The Entry Point

OMO is a single `Plugin` function that returns tools, event handlers, and lifecycle hooks.

```typescript
// src/index.ts:L1-L5
import type { Plugin } from "@opencode-ai/plugin";

const OhMyOpenCodePlugin: Plugin = async (ctx) => {
  log("[OhMyOpenCodePlugin] ENTRY - plugin loading", { directory: ctx.directory })
  startTmuxCheck();
```

## What It Returns

```
return {
  tool: { builtinTools, call_omo_agent, delegate_task, skill, ... },
  event: async (input) => { /* session lifecycle, error recovery */ },
  "chat.message": async (input, output) => { /* ralph loop, agent variants */ },
  "tool.execute.before": async (input, output) => { /* permissions, injection */ },
  "tool.execute.after": async (input, output) => { /* error recovery, truncation */ },
  config: configHandler,
}
```

## Configuration: Two-Layer Merge

```typescript
// src/plugin-config.ts:L64-L80
// User-level: ~/.config/opencode/oh-my-opencode.json
// Project-level: .opencode/oh-my-opencode.json
// Project overrides user. disabled_hooks arrays are Set-merged.
```

## Hook Enable/Disable

```typescript
const disabledHooks = new Set(pluginConfig.disabled_hooks ?? []);
const isHookEnabled = (hookName) => !disabledHooks.has(hookName);
// Each hook is conditionally created: null if disabled
const ralphLoop = isHookEnabled("ralph-loop") ? createRalphLoopHook(ctx, {...}) : null;
```

Every feature is independently toggleable.

---

[Next: Multi-Agent System →](s02-multi-agent-system.md)
