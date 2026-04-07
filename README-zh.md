<!--
  🌐 SEO & Keywords
  Oh My OpenCode 源码解读 | OMO 教程 | OpenCode 插件系统 | Sisyphus Agent 编排器
  多专家Agent协作框架 | Claude Code兼容层 | AI代码重构教程 | OpenCLAW生态
-->
[English](./README.md) | [中文](./README-zh.md)

# learn-OMO: 深入拆解 Oh My OpenCode 源码架构

> 跟随一条真实任务从零走到终点，理解 OMO（Oh My OpenCode）如何用 Sisyphus 编排器 + 多专家Agent + 自动恢复机制，完成复杂代码重构。
>
> **关键词**：Oh My OpenCode, OMO, OpenCode, Sisyphus, Claude Code, AI Agent, 多Agent协作, Ralph Loop, 自动重试, AST-grep, LSP工具, 插件系统, 代码重构, OpenCLAW

## 这是什么

**这不是 API 文档，也不是功能列表。这是一个故事：**

你对 OMO 说"帮我重构这个模块"，然后跟着这条任务走过整个系统——从 plugin 启动（Plugin Bootstrap）到 Ralph Loop 自动循环重试直到任务完成。

每一章从上一章结束的地方开始，每一章跟着代码走，每一章只讲一件事。

## 核心特性对比

| 特性 | 原始 OpenCode | OpenCode + OMO | Claude Code |
|------|---------------|----------------|-------------|
| 多专家Agent | ❌ 不支持 | ✅ 8+ 专业Agent | ✅ 内置 |
| 自动重试循环（ Ralph Loop） | ❌ 不支持 | ✅ Ralph Loop | ❌ 不支持 |
| AST语法树搜索 | ❌ 不支持 | ✅ ast-grep（25+语言） | ❌ 不支持 |
| 完整 LSP 工具链 | ❌ 不支持 | ✅ 定义/引用/重命名 | ⚠️ 部分支持 |
| Plan→Review 流水线 | ❌ 不支持 | ✅ Prometheus→Metis→Momus | ❌ 不支持 |
| 错误自动恢复 | ❌ 不支持 | ✅ 3种恢复Hook | ❌ 不支持 |
| 可定制化程度 | 中等 | 极高（30+ Hook点） | 低 |

## 架构全景

![Architecture](./docs/images/architecture.webp)

## 两种阅读路径

**🏗️ 高级开发者？想快速理解整体架构** → [架构设计文档](./docs/en/architecture.md)

**📖 刚接触 OMO？想从第一章开始学** → [第一章：插件启动](./docs/en/ch01-plugin-bootstrap.md)

## 完整章节列表

| # | 章节 | 核心内容 |
|---|------|----------|
| 01 | [插件启动](./docs/zh/ch01-plugin-bootstrap.md) | OMO plugin 如何加载进 OpenCode，注册 hooks/tools/agents |
| 02 | [Sisyphus 接收任务](./docs/zh/ch02-sisyphus-planning.md) | Sisyphus 分析用户意图、评估代码库、制定执行计划 |
| 03 | [委派系统](./docs/zh/ch03-delegation.md) | delegate_task 如何把任务分发给 Sisyphus-Junior + skills |
| 04 | [专家Agents](./docs/zh/ch04-specialist-agents.md) | Oracle/Explore/Librarian 三种专家Agent如何协作 |
| 05 | [Hook管道](./docs/zh/ch05-hook-pipeline.md) | think-mode、rules注入、context管理如何透明增强系统 |
| 06 | [错误恢复](./docs/zh/ch06-error-recovery.md) | 编辑失败、session崩溃、上下文溢出如何自动修复 |
| 07 | [后台执行](./docs/zh/ch07-background-agents.md) | 如何并行运行多个agent，实现并发搜索与执行 |
| 08 | [动态Prompt](./docs/zh/ch08-dynamic-prompts.md) | 每个agent的prompt如何根据可用资源现场动态拼装 |
| 09 | [增强工具](./docs/zh/ch09-crafted-tools.md) | AST-grep、LSP、interactive bash如何超越原生能力 |
| 10 | [Claude Code兼容层](./docs/zh/ch10-cc-compatibility.md) | Claude Code用户如何零成本迁移到 OMO |
| 11 | [Ralph Loop](./docs/zh/ch11-ralph-loop.md) | 自动循环直到任务完成——OMO 的杀手级特性 |

## 如何阅读

**从头到尾读** — 章节之间是连贯的故事线，每一章都建立在前一章之上。

**对照代码读** — 每个代码片段都标注了文件路径和行号，打开源码对照理解更深。

## 相关项目

- **Oh My OpenCode**：[https://github.com/opensoft/oh-my-opencode](https://github.com/opensoft/oh-my-opencode)
- **OpenCode**：[https://github.com/opencode-ai/opencode](https://github.com/opencode-ai/opencode)
- **OpenCLAW**：[https://openclaw.ai](https://openclaw.ai)

## License

MIT
