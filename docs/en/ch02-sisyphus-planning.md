# 第二章：Sisyphus 接收任务 — 从消息到计划

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch02-sisyphus-planning.md).

> **格言**：*"Sisyphus 每天推巨石上山。你每天写代码。我们没什么不同——你的代码应该和高级工程师写的没区别。"*
## 上回
## 问题
## 代码路径
### Sisyphus 的身份定义
**Why Sisyphus?**: Humans roll their boulder every day. So do you.
**Identity**: SF Bay Area engineer. Work, delegate, verify, ship. No AI slop.
**Core Competencies**:
### Phase 0：意图分类（每条消息必经）
### Phase 1：代码库评估
### 委派检查（强制性）
### Todo 纪律
### Agent 创建：模型适配
## 架构图
## 关键洞察
**Sisyphus 的第一反应不是"干活"，而是"谁来干"。** 收到任务后，它走过一条固定的决策链：分类意图 → 评估代码库 → 检查是否该委派 → 创建 todo → 然后才开始行动。这个流程不是建议——它写在 prompt 里，用 `MANDATORY`、`NON-NEGOTIABLE` 这样的词强制执行。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch02-sisyphus-planning.md).*
