# 第七章：后台执行 — 并行运行多个 Agent

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch07-background-agents.md).

> **格言**：*"一个 Agent 是线程，多个 Agent 是并发。真正的生产力在于并行。"*
## 上回
## 问题
## 代码路径
### BackgroundManager：后台任务管理器
### 并发控制
### 后台工具
### Sisyphus 的并行模式
**关键规则**：Explore 和 Librarian **总是后台运行**。它们是"grep"——你不会等 grep 命令返回后才继续工作。
### TmuxSessionManager：终端多路复用
### 任务生命周期
### 后台通知
## 架构图
## 关键洞察
**后台 agent 不是"另一个 tab"——是结构化的并发。** BackgroundManager 控制并发数、管理任务生命周期、处理超时和 stale 检测。每个后台 agent 都有独立的 session，互不干扰。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch07-background-agents.md).*
