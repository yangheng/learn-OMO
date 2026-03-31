# 第五章：Hook 管道 — 在你不知道的时候增强一切

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch05-hook-pipeline.md).

> **格言**：*"最好的基础设施是你感觉不到它存在，直到它救了你。"*
## 上回
## 问题
## 代码路径
### Hook 的四个入口
### Think Mode：按需升级推理
### Rules Injector：项目规则自动注入
### Context Window Monitor：窗口守卫
### Tool Output Truncator：输出管理
### Directory Agents Injector：项目级 Agent 配置
### Hook 执行链
**每次工具调用都经过两道关卡**：执行前（参数修改、规则注入）和执行后（输出截断、错误恢复、诊断监控）。
### 可配置的开关
## 架构图
## 关键洞察
**Hook 是 OMO 的中间件层。** 就像 Express.js 的中间件一样，每个请求（消息/事件/工具调用）都穿过一系列处理器。但 OMO 的 hook 更精细——它们分为四个生命周期点，每个点都有不同的 hook 串。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch05-hook-pipeline.md).*
