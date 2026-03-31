# 第六章：错误恢复与韧性 — 出错时自动修复

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch06-error-recovery.md).

> **格言**：*"AI 不会不犯错，但好的 AI 知道怎么从错误中恢复。"*
## 上回
## 问题
## 代码路径
### Edit Error Recovery：编辑失败自动修复
### Session Recovery：会话崩溃恢复
### Anthropic Context Window Limit Recovery：上下文溢出恢复
### Delegate Task Retry：委派任务重试
### 三层恢复体系
## 架构图
## 关键洞察
**OMO 的恢复策略不是"重试"——是"理解错误后修正"。** Edit Error Recovery 不是简单重试编辑操作，而是强制 agent 先读取文件的实际状态。Session Recovery 不是重启 session，而是修复消息结构后继续。Context Window Recovery 不是放弃，而是自动压缩后重试。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch06-error-recovery.md).*
