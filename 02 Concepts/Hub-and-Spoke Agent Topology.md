# Hub-and-Spoke Agent Topology

## What Is It

Hub-and-Spoke（中心辐射）是一种多 Agent 通信拓扑：一个 Coordinator 作为中心节点负责任务拆分、分配与汇总；多个 Worker 在隔离环境中执行；Worker 之间无点对点通信，结果通过结构化文本回填 Coordinator 上下文。

## Problem It Solves

* 复杂任务需要并行子任务，但 Agent 间自由通信难以预测和调试
* 需要隔离执行环境（Claude Code 使用 git worktree）避免冲突

## Core Ideas

Claude Code Coordinator Mode：

* Coordinator 通过 TeamCreateTool / SendMessageTool 管理 Worker
* Worker 在独立 worktree 执行，上下文隔离
* 通信 = 结构化文本 pipe，**无独立 Agent-to-Agent 消息总线**
* 对比 G-Designer（2024）等动态拓扑研究：Claude Code 选择固定拓扑，牺牲灵活性换可预测性

类比：Task Router + Worker Pool，类似 Kafka Consumer Group 或 Celery Worker。

## Related Concepts

[[Agent Routing]]
[[TAOR Loop]]
[[Multi-Agent]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：多 Agent = Agent 间自由对话
  **实际**：Hub-and-Spoke 禁止 Worker 直连，所有协调经中心节点

## My Understanding

对企业 AI 团队：早期不要追求动态拓扑最优，先用 Hub-and-Spoke 跑通任务路由，再考虑扩展。可预测性在生产环境通常比理论最优拓扑更重要。
