# 第八章：动态 Prompt 组装 — 每个 Agent 的 Prompt 都是现场拼的

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch08-dynamic-prompts.md).

> **格言**：*"写死的 prompt 是债务，动态的 prompt 是资产。"*
## 上回
## 问题
## 代码路径
### 动态构建器入口
### buildKeyTriggersSection：触发器
### buildToolSelectionTable：工具选择矩阵
### buildDelegationTable：委派矩阵
### Agent Prompt Metadata：自描述元数据
### buildCategorySkillsDelegationGuide：Skill 系统
### 最终组装
## 架构图
## 关键洞察
**每个 agent 都参与了 Sisyphus prompt 的编写。** 当 Oracle 定义了 `ORACLE_PROMPT_METADATA` 里的 triggers 和 useWhen 时，它实际上在告诉 Sisyphus："在这些情况下叫我"。这不是 Sisyphus 硬编码的——是每个 agent **自我声明**后，由构建器自动编排。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch08-dynamic-prompts.md).*
