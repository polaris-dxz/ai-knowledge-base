# AI Agent

## What Is It

AI Agent 是以大语言模型为推理核心、能够**感知环境、调用工具、多步执行、达成目标**的软件系统。它不是单纯的 Chatbot（单轮问答），也不是固定 Workflow（硬编码链路），而是**模型驱动的自主循环**：推理 → 行动 → 观察 → 重复，直到任务完成或终止。

在本知识库中，AI Agent 是 **AI Agent 域** 的根概念，覆盖 Codex、Claude Code、Cursor、OpenHands 等产品与框架。

## Problem It Solves

* 将自然语言意图转化为可执行的软件工程 / 研究 / 操作行为
* 在开放环境中处理多步、有副作用、需工具协作的复杂任务
* 降低人机协作成本：人描述目标，Agent 负责执行与迭代

## Core Ideas

* **Agent Loop**：Think → Act → Observe → Repeat（ReAct / TAOR）
* **Tool Calling**：通过函数/API/MCP 扩展 Agent 能力边界
* **Harness**：包裹模型的运行时——权限、上下文、工具注册、会话管理（如 Claude Code、Codex CLI）
* **Multi-Agent**：任务拆分、并行执行、Coordinator + Worker 协作
* **Human-in-the-Loop**：[[Agent Execution Modes]]（Plan Mode）、权限确认、审查高风险操作
* **Computer Use**：通过 GUI/浏览器/终端操作计算机环境

## Related Concepts

[[Agent Routing]]
[[Agent Execution Modes]]
[[TAOR Loop]]
[[MCP]]
[[Tool Calling]]
[[Multi-Agent]]
[[Computer Use]]
[[Plan Mode]]
[[Progressive Tool Disclosure]]

## Related Systems

[[Claude Code Architecture]]
[[Cursor]]

## Related Projects

[[Yuniverse Work Desktop Architecture]]

待补充：Codex、OpenHands 等 System 笔记。

## Common Misunderstandings

* **误解**：Agent = 聊天机器人
  **实际**：Chatbot 无工具、无循环；Agent 可在环境中持续行动

* **误解**：Agent = Workflow 自动化
  **实际**：Workflow 路径固定；Agent 由模型在每步决定下一步

* **误解**：Agent 的核心是更好的 Prompt
  **实际**：生产 Agent 的竞争力往往在 Harness 工程（路由、权限、压缩、工具层），而非 Prompt 技巧 alone

## My Understanding

AI Agent 域的关注点应分为三层：

1. **模型层** — 推理与规划能力
2. **Harness 层** — 运行时、工具、权限、上下文（本知识库重点）
3. **产品层** — 入口形态（CLI、IDE、Desktop）、用户体验、商业化

做 yuniverse-work 桌面 Agent 时，Harness 层的设计（入口收敛、模式路由、工具曝光）比追新模型更重要。Agent 不是「套壳 LLM」，而是**可长期运行的工程系统**。

## Sources

（根概念，由多 Source / System 笔记支撑，不绑定单一来源）
