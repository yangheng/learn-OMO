# 第四章：专家 Agents — 每个角色的真正职责

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch04-specialist-agents.md).

> **格言**：*"不是每个问题都需要同一把锤子。Oracle 思考，Explore 搜索，Librarian 查阅，各司其职。"*
## 上回
## 问题
## 代码路径
### Explore — 代码库的搜索引擎
**关键约束**：Explore 被禁止使用 `write`, `edit`, `task`, `delegate_task`, `call_omo_agent`。纯搜索，无副作用。
### Librarian — 外部世界的窗口
### Oracle — 只读的高级顾问
### Metis — 计划前的顾问
### Momus — 计划的审查官
### Atlas — 编排指挥官
### Agent 权限矩阵
## 架构图
## 关键洞察
**权限就是架构。** OMO 通过工具权限矩阵定义了每个 agent 的能力边界。Oracle 不能写代码，所以它的建议总是"纯建议"——不会意外修改文件。Explore 不能委派，所以它不会试图启动子任务。Sisyphus-Junior 不能再委派，防止了委派递归。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch04-specialist-agents.md).*
