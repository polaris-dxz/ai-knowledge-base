# Plan Mode

## What Is It

Plan Mode 是 Claude Code 的一种执行模式路由：只允许只读工具（Read、Glob、Grep 等），禁用有副作用的工具（Bash、Edit 等），输出供用户审查的执行计划而非直接执行。

## Problem It Solves

* 高风险、不可逆操作（大规模文件改动、危险 Shell 命令）需要人在执行前审查
* 复杂任务需要先规划再执行，降低 Agent 盲目行动的风险

## Core Ideas

* 通过 EnterPlanModeTool / ExitPlanModeTool 触发——**模型可在推理中自主决定**进入 Plan Mode
* 本质是**执行风险路由**：在路由层面阻断危险操作的直接触发路径
* 与 Default Mode（标准 TAOR + 真实副作用）形成模式级切换

## Related Concepts

[[Agent Routing]]
[[TAOR Loop]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：Plan Mode = 单独的 UI 功能
  **实际**：是运行时模式路由，通过工具调用进入/退出

## My Understanding

Plan Mode 把「人类在环」嵌入路由层而非事后 Review，对构建企业 Agent 有直接参考价值：权限与模式应在架构一等公民位置，而非功能列表里的一项。
