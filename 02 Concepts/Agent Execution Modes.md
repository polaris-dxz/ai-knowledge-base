# Agent Execution Modes

## What Is It

Agent Execution Modes 指 AI Agent 系统中通过**模式切换**改变执行语义、工具权限与协作拓扑的机制。不同模式决定 Agent 在同一循环（如 [[TAOR Loop]]）中能做什么、不能做什么。

Claude Code 定义了三种一等模式：

| 模式 | 工具权限 | 执行语义 | 触发方式 |
|------|----------|----------|----------|
| **Default** | 全量工具，有真实副作用 | 标准 TAOR 循环 | 默认 |
| **Plan** | 只读工具（Read、Glob、Grep 等），禁用副作用工具 | 输出执行计划供人审查 | EnterPlanModeTool / ExitPlanModeTool（模型可自主触发） |
| **Coordinator** | Hub-and-Spoke 多 Agent 协作 | Coordinator 拆分任务，Worker 在 worktree 隔离执行 | 复杂任务需并行子任务时 |

## Problem It Solves

* 单一执行模式无法兼顾效率与安全——高风险操作需要人审查，常规任务不应被审查流程拖慢
* 复杂任务需要拆分为多 Agent 并行执行，但与单 Agent 模式共享同一运行时入口
* 模式切换需要在**路由层**而非 UI 层实现，确保工具权限与执行语义的一致性

## Core Ideas

* **模式即路由**：模式切换本质是 [[Agent Routing]] 的一种——改变的是执行路径、工具集与权限策略
* **模型自主切换**：Plan Mode 通过工具调用进入/退出，模型在推理中决定何时需要规划
* **工具权限门控**：每种模式定义允许/禁止的工具集，而非事后审查
* **与 TAOR 的关系**：三种模式共享同一 [[TAOR Loop]] 结构，但循环内的行动空间不同

## Related Concepts

[[Agent Routing]]
[[TAOR Loop]]
[[Plan Mode]]
[[Hub-and-Spoke Agent Topology]]
[[Multi-Agent]]
[[Progressive Tool Disclosure]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：模式切换 = UI 按钮切换
  **实际**：是运行时路由决策，通过工具调用或任务特征触发，影响整个执行语义

* **误解**：三种模式是独立系统
  **实际**：共享同一 TAOR 循环与基础设施，差异在工具权限与协作拓扑

## My Understanding

模式是 Agent 系统的「档位」——不改变引擎（TAOR），但改变传动（工具权限、协作方式）。生产 Agent 至少需要 Default + Plan 两档；多 Agent 协作则需要 Coordinator 档。模式设计应在架构层面是一等公民，而非功能列表的附加项。
