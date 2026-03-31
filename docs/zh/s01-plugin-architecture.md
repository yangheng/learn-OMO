# S01：插件架构 — 一切始于一个 Plugin 接口

> **格言**："一切始于一个 Plugin 接口"

[下一章 →](s02-multi-agent-system.md)

---

## 问题：如何在不修改宿主的情况下注入整个 Agent 生态系统？

OpenCode 提供了一个 `Plugin` 接口。OMO 通过这个单一入口点，注册了 30+ 个 Hook、10+ 个工具、8+ 个 Agent。

## 插件入口

```typescript
// src/index.ts:L1-L5
import type { Plugin } from "@opencode-ai/plugin";

const OhMyOpenCodePlugin: Plugin = async (ctx) => {
  log("[OhMyOpenCodePlugin] ENTRY - plugin loading", { directory: ctx.directory })
  startTmuxCheck();
```

`Plugin` 是一个异步函数，接收 `ctx`（包含 `client`、`directory`、`serverUrl`）。返回一个对象，包含 `tool`、`event`、各种 Hook 生命周期。

## 架构图

```
OpenCode 启动
    │
    ▼
加载 OhMyOpenCodePlugin(ctx)
    │
    ├── 1. 解析配置 (loadPluginConfig)
    │       ├── 用户级: ~/.config/opencode/oh-my-opencode.json
    │       └── 项目级: .opencode/oh-my-opencode.json
    │
    ├── 2. 初始化所有 Hook (30+)
    │       ├── createRalphLoopHook()
    │       ├── createSessionRecoveryHook()
    │       ├── createEditErrorRecoveryHook()
    │       ├── createThinkModeHook()
    │       └── ... (每个都可通过 disabled_hooks 禁用)
    │
    ├── 3. 初始化管理器
    │       ├── BackgroundManager(ctx)     — 后台任务管理
    │       ├── TmuxSessionManager(ctx)    — tmux 窗格管理
    │       └── SkillMcpManager()          — MCP 服务管理
    │
    ├── 4. 注册工具
    │       ├── builtinTools (grep, glob, lsp_*, ast_grep_*)
    │       ├── call_omo_agent
    │       ├── delegate_task
    │       ├── skill, skill_mcp
    │       ├── slashcommand
    │       └── interactive_bash
    │
    └── 5. 返回 Plugin 对象
            ├── tool: { ... }
            ├── event: async (input) => { ... }
            ├── "chat.message": async (input, output) => { ... }
            ├── "tool.execute.before": async (input, output) => { ... }
            ├── "tool.execute.after": async (input, output) => { ... }
            └── config: configHandler
```

## 配置系统

OMO 使用 **两层配置合并**：用户级 + 项目级。

```typescript
// src/plugin-config.ts:L64-L80
export function loadPluginConfig(directory: string, ctx: unknown): OhMyOpenCodeConfig {
  const configDir = getOpenCodeConfigDir({ binary: "opencode" });
  const userBasePath = path.join(configDir, "oh-my-opencode");
  // ...
  let config: OhMyOpenCodeConfig = loadConfigFromPath(userConfigPath, ctx) ?? {};
  const projectConfig = loadConfigFromPath(projectConfigPath, ctx);
  if (projectConfig) {
    config = mergeConfigs(config, projectConfig);
  }
  return config;
}
```

合并逻辑（`mergeConfigs`）对 `disabled_hooks`、`disabled_agents` 等数组字段使用 Set 去重合并：

```typescript
// src/plugin-config.ts:L32-L48
disabled_hooks: [
  ...new Set([
    ...(base.disabled_hooks ?? []),
    ...(override.disabled_hooks ?? []),
  ]),
],
```

## Hook 启用/禁用机制

```typescript
// src/index.ts
const disabledHooks = new Set(pluginConfig.disabled_hooks ?? []);
const isHookEnabled = (hookName: HookName) => !disabledHooks.has(hookName);

const ralphLoop = isHookEnabled("ralph-loop")
  ? createRalphLoopHook(ctx, { ... })
  : null;
```

每个 Hook 都是**可选的**。如果配置中 `disabled_hooks` 包含 `"ralph-loop"`，Ralph Loop 就不会被创建。

## 生命周期 Hook

OMO 返回的 Plugin 对象实现了 OpenCode 的全部生命周期：

| Hook | 触发时机 | OMO 用途 |
|------|---------|---------|
| `event` | 任何事件 | session.created/deleted/error 处理 |
| `chat.message` | 用户发消息 | Ralph Loop 检测、Agent variant 设置 |
| `tool.execute.before` | 工具执行前 | 参数修改、权限控制、注入提醒 |
| `tool.execute.after` | 工具执行后 | 错误恢复、输出截断、诊断注入 |
| `config` | 配置查询 | Agent 配置动态生成 |

## 关键洞察

1. **一个文件引导一切**：`src/index.ts` 是整个 OMO 的控制中心，约 400 行代码组织了 30+ 个 Hook 和 10+ 个工具的初始化。

2. **条件组合模式**：每个功能都是独立创建、条件注入的。这使得 OMO 可以通过配置精确控制每个功能的开关。

3. **Plugin 返回对象是扁平的**：OpenCode 的 Plugin 接口要求返回一个包含 `tool`、`event`、各种 Hook 的对象。OMO 巧妙地在内部通过闭包和管理器类实现了复杂的状态管理。

## OMO vs Claude Code

| 方面 | OMO | Claude Code |
|------|-----|------------|
| 扩展方式 | Plugin 接口 | 内置 |
| 配置 | JSON/JSONC, 两层合并 | YAML |
| 功能开关 | disabled_hooks 数组 | 有限 |
| 定制性 | 极高 (Hook 级别) | 低 |

---

[下一章：多 Agent 系统 →](s02-multi-agent-system.md)
