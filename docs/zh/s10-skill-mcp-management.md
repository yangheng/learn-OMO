# S10：Skill 与 MCP 管理 — 插件的插件，无限套娃

> **格言**："插件的插件，无限套娃"

[← 上一章](s09-claude-code-compat.md) | [下一章 →](s11-error-recovery.md)

---

## 问题：如何让 Agent 学会新技能？

Skill 是给 Agent 注入领域知识的机制。MCP 是让 Agent 调用外部服务的协议。OMO 两者都管理。

## Skill 发现系统

```typescript
// src/index.ts:L104-L115
const [userSkills, globalSkills, projectSkills, opencodeProjectSkills] = await Promise.all([
  includeClaudeSkills ? discoverUserClaudeSkills() : Promise.resolve([]),
  discoverOpencodeGlobalSkills(),
  includeClaudeSkills ? discoverProjectClaudeSkills() : Promise.resolve([]),
  discoverOpencodeProjectSkills(),
])
const mergedSkills = mergeSkills(
  builtinSkills, pluginConfig.skills,
  userSkills, globalSkills, projectSkills, opencodeProjectSkills
)
```

### 五层 Skill 来源

```
优先级（低 → 高）：
┌─────────────────────────┐
│ 1. builtinSkills        │  OMO 内置技能
├─────────────────────────┤
│ 2. pluginConfig.skills  │  配置中定义的技能
├─────────────────────────┤
│ 3. userSkills (CC)      │  ~/.claude/ 用户级
├─────────────────────────┤
│ 4. globalSkills (OC)    │  ~/.config/opencode/skills/
├─────────────────────────┤
│ 5. projectSkills        │  .opencode/skills/ 项目级
└─────────────────────────┘
```

## Builtin Skills

```typescript
// src/features/builtin-skills/index.ts
export function createBuiltinSkills({ browserProvider }): Skill[] {
  // 内置技能，如 playwright 浏览器自动化
  // 可通过 disabled_skills 配置禁用
}
```

内置 Skill 会检查是否与系统 MCP 冲突：

```typescript
// src/index.ts:L119-L126
const builtinSkills = createBuiltinSkills({ browserProvider }).filter((skill) => {
  if (disabledSkills.has(skill.name)) return false
  if (skill.mcpConfig) {
    for (const mcpName of Object.keys(skill.mcpConfig)) {
      if (systemMcpNames.has(mcpName)) return false  // 避免重复
    }
  }
  return true
})
```

## Skill Tool — Agent 加载技能

```typescript
// src/tools/skill/tools.ts
const skillTool = createSkillTool({
  skills: mergedSkills,
  mcpManager: skillMcpManager,
  getSessionID: getSessionIDForMcp,
})
```

Agent 通过 `skill` 工具按需加载技能。技能内容被注入到 Agent 的 system prompt 中。

## SkillMcpManager — MCP 生命周期管理

```typescript
// src/features/skill-mcp-manager/index.ts
export class SkillMcpManager {
  // 管理 Skill 附带的 MCP 服务器
  // 按 session 隔离 MCP 连接
  async disconnectSession(sessionId: string)  // session 删除时清理
}
```

## Skill 在委派中的使用

```typescript
// delegate_task 中 load_skills 参数
delegate_task(
  category="visual-engineering",
  load_skills=["playwright", "frontend-ui-ux"],
  prompt="..."
)
```

Skill 内容通过 `resolveMultipleSkillsAsync` 异步解析，然后通过 `buildSystemContent` 注入：

```typescript
// src/tools/delegate-task/tools.ts
export function buildSystemContent({ skillContent, categoryPromptAppend }) {
  if (skillContent && categoryPromptAppend) {
    return `${skillContent}\n\n${categoryPromptAppend}`
  }
  return skillContent || categoryPromptAppend
}
```

## 关键洞察

1. **五层合并**：Skill 来自内置、配置、用户、全局、项目五个来源，按优先级合并。
2. **MCP 冲突检测**：内置 Skill 的 MCP 如果与系统 MCP 重名，自动跳过。
3. **Session 隔离**：每个 session 有独立的 MCP 连接，删除 session 时自动清理。
4. **Skill 即 Prompt 片段**：技能本质上是被前置到 Agent prompt 中的领域知识。

---

[← 上一章：Claude Code 兼容层](s09-claude-code-compat.md) | [下一章：错误恢复 →](s11-error-recovery.md)
