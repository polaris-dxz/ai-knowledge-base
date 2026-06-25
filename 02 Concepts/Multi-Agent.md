# Multi-Agent

## What Is It

Multi-Agent 指由**多个 AI Agent 实例协作**完成单一复杂任务的架构模式。每个 Agent 拥有独立或部分隔离的上下文，通过任务拆分、并行执行、结果汇总来扩展单 Agent 的能力边界。

它不是「多个 Chatbot 聊天」，而是**有编排语义的多实例系统**：谁协调、谁执行、如何通信、如何隔离，都是一等设计问题。

## Problem It Solves

* 单 Agent 上下文窗口有限，无法一次承载超大代码库或长链路任务
* 复杂任务可分解为可并行的子任务（调研 / 实现 / 测试）
* 需要隔离风险：子任务在独立环境执行，避免互相污染

## Core Ideas

### 协作形态

| 形态 | 特征 | 典型实现 |
|------|------|----------|
| **Subagent** | 主 Agent 派发子任务，子上下文隔离，结果回传 | Claude Code `Task` / Subagent |
| **Coordinator + Worker** | 中心节点拆分与汇总，Worker 并行执行（见 [[Hub-and-Spoke Agent Topology]]） | Claude Code Coordinator Mode（见 [[Agent Execution Modes]]） |
| **Agent Teams** | 多 Agent 协作，支持 Agent 间直接通信 | Claude Code Agent Teams（实验性） |
| **Code Orchestration** | 用代码（`/workflows`）编排阶段，非 LLM 编排 | Claude Code workflows |

### 通信拓扑

详见 [[Hub-and-Spoke Agent Topology]]（生产首选固定拓扑）。研究方向包括动态拓扑（如 G-Designer），灵活但难预测。Claude Code 使用结构化文本 pipe 而非独立消息总线。

### 编排层级

```text
LLM 编排   — 模型在推理中决定派发（灵活，Token 贵，难调试）
代码编排   — 预定义阶段与数据流（可预测，适合生产）
混合编排   — 粗粒度代码 + 细粒度 LLM
```

## Related Concepts

[[AI Agent]]
[[Hub-and-Spoke Agent Topology]]
[[Agent Routing]]
[[TAOR Loop]]
[[Agent Execution Modes]]

## Related Systems

[[Claude Code Architecture]]

待补充：OpenHands、AutoGen、CrewAI 等 System 笔记。

## Common Misunderstandings

* **误解**：Multi-Agent = 多个用户各自开 Chat
  **实际**：是同一任务下的多实例协作，有编排与汇总

* **误解**：Agent 越多越强
  **实际**：通信开销、上下文污染、调试复杂度随 Agent 数上升；生产系统优先可预测拓扑（见 [[Hub-and-Spoke Agent Topology]]）

* **误解**：Multi-Agent 需要 Agent 间自然语言对话
  **实际**：Claude Code 等生产实现多用**结构化结果回传**，而非自由对话

## My Understanding

Multi-Agent 的演进方向（以 Claude Code 为例）：

```text
Subagent → Agent Teams → /workflows（代码编排）
```

趋势是：**用代码取代 LLM 做编排**，子 Agent 输出直接 stage 流转，主上下文零污染。对 yuniverse-work 的启示：

1. 先实现 **Subagent + Hub-and-Spoke**，不急于动态拓扑
2. 子任务隔离（worktree / sandbox / session）是架构必需，不是优化项
3. Multi-Agent 与 [[Agent Routing]] 深度耦合——Coordinator 本质是 Task Router

## Sources

（根概念；可链接 [[Claude Code Agent 路由设计 - 泄露源码分析]] 等 Source）

[[Claude Code Agent 路由设计 - 泄露源码分析]]
