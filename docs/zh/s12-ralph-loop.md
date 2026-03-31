# S12：Ralph Loop — 做完了吗？没有？那继续。

> **格言**："做完了吗？没有？那继续。"

[← 上一章](s11-error-recovery.md)

---

## 问题：如何让 Agent 真正"做完"一件事？

普通 Agent 做完一轮就停了。Ralph Loop 在 session 结束后检查：**你真的做完了吗？**

## Ralph Loop 核心原理

```
用户发消息: "ralph-loop '实现认证系统' --max-iterations=10"
    │
    ▼
startLoop() — 写入状态文件
    │
    ▼
Agent 开始工作...
    │
    ▼
Session idle (Agent 停止响应)
    │
    ▼
event handler 检测 session.idle
    │
    ├── 检测 completion promise: <promise>done</promise>
    │     ├── 找到 → 清除状态, 显示 "Ralph Loop Complete!"
    │     └── 未找到 → 继续循环
    │
    ├── 检查迭代次数 >= max_iterations?
    │     ├── 是 → 停止, 显示 "Max iterations reached"
    │     └── 否 → 注入 continuation prompt
    │
    └── 注入 continuation prompt:
          "[RALPH LOOP 2/10] Your previous attempt did not output
           the completion promise. Continue working on the task."
          → Agent 继续工作 → 回到 "Session idle" 步骤
```

## 状态管理

```typescript
// src/hooks/ralph-loop/index.ts:L69-L80
const startLoop = (sessionID, prompt, loopOptions?) => {
  const state: RalphLoopState = {
    active: true,
    iteration: 1,
    max_iterations: loopOptions?.maxIterations ?? DEFAULT_MAX_ITERATIONS,
    completion_promise: loopOptions?.completionPromise ?? DEFAULT_COMPLETION_PROMISE,
    ultrawork: loopOptions?.ultrawork,
    started_at: new Date().toISOString(),
    prompt,
    session_id: sessionID,
  }
  writeState(ctx.directory, state, stateDir)
}
```

状态通过文件系统持久化，**跨 session 恢复都有效**。

## Completion Promise 检测

Ralph Loop 检测 Agent 是否输出了 completion promise（默认是 `<promise>done</promise>`）：

```typescript
// src/hooks/ralph-loop/index.ts:L30-L45
function detectCompletionPromise(transcriptPath, promise): boolean {
  const content = readFileSync(transcriptPath, "utf-8")
  const pattern = new RegExp(`<promise>\\s*${escapeRegex(promise)}\\s*</promise>`, "is")
  const lines = content.split("\n").filter(l => l.trim())
  for (const line of lines) {
    const entry = JSON.parse(line)
    if (entry.type === "user") continue  // 跳过用户消息
    if (pattern.test(line)) return true
  }
  return false
}
```

**双重检测**：先检查 transcript 文件，再通过 API 检查 session messages：

```typescript
// src/hooks/ralph-loop/index.ts:L55-L68
async function detectCompletionInSessionMessages(sessionID, promise): Promise<boolean> {
  const response = await ctx.client.session.messages({ path: { id: sessionID } })
  const messages = response.data ?? []
  const assistantMessages = messages.filter(msg => msg.info?.role === "assistant")
  const lastAssistant = assistantMessages[assistantMessages.length - 1]
  // 在最后一条 assistant 消息中搜索 promise
  const pattern = new RegExp(`<promise>\\s*${escapeRegex(promise)}\\s*</promise>`, "is")
  return pattern.test(responseText)
}
```

## Continuation Prompt

```typescript
// src/hooks/ralph-loop/index.ts:L23-L28
const CONTINUATION_PROMPT = `[SYSTEM DIRECTIVE - RALPH LOOP {{ITERATION}}/{{MAX}}]

Your previous attempt did not output the completion promise. Continue working on the task.

IMPORTANT:
- Review your progress so far
- Continue from where you left off  
- When FULLY complete, output: <promise>{{PROMISE}}</promise>
- Do not stop until the task is truly done

Original task:
{{PROMPT}}`
```

注入时保留原始 Agent 和 model 设置：

```typescript
// src/hooks/ralph-loop/index.ts (event handler)
// 从最后的消息中获取 agent 和 model
let agent, model
const messages = await ctx.client.session.messages({ path: { id: sessionID } })
for (let i = messages.length - 1; i >= 0; i--) {
  const info = messages[i].info
  if (info?.agent || info?.model) {
    agent = info.agent
    model = info.model
    break
  }
}

await ctx.client.session.prompt({
  path: { id: sessionID },
  body: { agent, model, parts: [{ type: "text", text: finalPrompt }] },
})
```

## Ultrawork 模式

Ralph Loop 有一个变体叫 **ULW (Ultrawork) Loop**：

```typescript
// 通过 /ulw-loop 命令启动
ralphLoop.startLoop(sessionID, prompt, { ultrawork: true })

// continuation prompt 前缀 "ultrawork"
const finalPrompt = newState.ultrawork
  ? `ultrawork ${continuationPrompt}`
  : continuationPrompt
```

## 错误处理

```typescript
// src/hooks/ralph-loop/index.ts (event handler)
if (event.type === "session.error") {
  if (error?.name === "MessageAbortedError") {
    // 用户手动中止 → 清除 loop
    clearState(ctx.directory, stateDir)
    return
  }
  // 其他错误 → 标记恢复中，5秒后恢复
  sessionState.isRecovering = true
  setTimeout(() => { sessionState.isRecovering = false }, 5000)
}
```

## 触发方式

Ralph Loop 可以通过三种方式触发：

1. **Chat message 模板**：`"You are starting a Ralph Loop" + "<user-task>..."` 
2. **Slash command**：`/ralph-loop "task" --max-iterations=10`
3. **ULW command**：`/ulw-loop "task"`

```typescript
// src/index.ts:L195-L210
if (ralphLoop && input.tool === "slashcommand") {
  const command = args?.command?.replace(/^\//, "").toLowerCase()
  if (command === "ralph-loop" && sessionID) {
    ralphLoop.startLoop(sessionID, prompt, { maxIterations, completionPromise })
  } else if (command === "cancel-ralph" && sessionID) {
    ralphLoop.cancelLoop(sessionID)
  } else if (command === "ulw-loop" && sessionID) {
    ralphLoop.startLoop(sessionID, prompt, { ultrawork: true, maxIterations })
  }
}
```

## Toast 通知

```typescript
// 循环完成
await ctx.client.tui.showToast({
  body: { title: "Ralph Loop Complete!", 
          message: `Task completed after ${state.iteration} iteration(s)`,
          variant: "success" }
})

// 达到最大迭代
await ctx.client.tui.showToast({
  body: { title: "Ralph Loop Stopped", 
          message: `Max iterations (${state.max_iterations}) reached`,
          variant: "warning" }
})

// 每次迭代
await ctx.client.tui.showToast({
  body: { title: "Ralph Loop", 
          message: `Iteration ${newState.iteration}/${newState.max_iterations}`,
          variant: "info" }
})
```

## 关键洞察

1. **文件系统持久化**：Ralph Loop 状态写入文件，即使 OpenCode 重启也能恢复循环。
2. **Completion Promise 是合约**：Agent 必须输出 `<promise>done</promise>` 才算真正完成。
3. **双重检测**：transcript 文件 + API 消息，确保不会漏检。
4. **保留上下文**：continuation prompt 注入时保留原始 Agent 和 model，确保一致性。
5. **用户可中止**：`MessageAbortedError` 立即清除循环，尊重用户意图。

## Ralph Loop 的哲学

这个名字来自 [Wreck-It Ralph](https://en.wikipedia.org/wiki/Wreck-It_Ralph)——一个每天都在"砸东西"的角色，但他从不放弃。

Ralph Loop 体现了 OMO 的核心哲学：
- **永不放弃**：最大迭代次数是安全阀，不是放弃的理由
- **自我验证**：completion promise 迫使 Agent 自我评估
- **可观测性**：toast 通知让用户随时知道进度
- **优雅退出**：用户随时可以取消，状态立即清理

这就是为什么用户说 **"It just works until the task is done."**

---

[← 上一章：错误恢复与韧性](s11-error-recovery.md)

---

🎉 **恭喜完成所有 12 个章节！** 你现在对 Oh My OpenCode 的架构有了全面的理解。

回到 [README](../../README-zh.md) | [English README](../../README.md)
