# 第一章：插件启动 — index.ts 如何挂载一切

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch01-plugin-bootstrap.md).

> **格言**：*"一个好的编排者，在第一行代码里就决定了整场战役的走向。"*
## 起点
## 问题
## 代码路径
### 入口：Plugin 函数签名
### Hook 注册：逐个创建
### Tool 注册：组装工具箱
### 返回值：Plugin 契约
### 事件分发：一个入口，所有 hook
## 架构图
## 关键洞察
**OMO 不是一个 agent，是一个 agent 平台。** `index.ts` 的工作不是思考——是组装。它在启动时把 30+ 个 hooks、10+ 个 tools、6+ 个 agents 全部注册到 OpenCode 里，然后退到幕后。从此以后，每条消息、每次工具调用、每个系统事件，都会经过这张精心编织的网。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch01-plugin-bootstrap.md).*
