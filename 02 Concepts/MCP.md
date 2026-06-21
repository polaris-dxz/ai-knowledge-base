# MCP

## What Is It

MCP（Model Context Protocol）是 Anthropic 推动的开放协议，用于**标准化 AI 应用与外部工具、数据源之间的连接方式**。它定义 Client / Server 角色、能力发现、资源暴露与工具调用的通用接口，使 Agent 可以 plug-in 式接入数据库、API、文件系统、SaaS 等服务。

MCP 解决的不是「如何执行一次函数调用」，而是**如何让工具与上下文资源以统一、可组合的方式暴露给 Agent**。

## Problem It Solves

* 每个 Agent 产品各自实现工具集成，重复造轮子、难以复用
* 工具描述、权限、连接方式缺乏跨产品标准
* 外部能力（Git、Slack、数据库、浏览器）接入 Agent 成本高

## Core Ideas

* **Client / Server**：Agent 侧为 MCP Client，能力提供方为 MCP Server
* **Capabilities**：Tools（可执行动作）、Resources（可读上下文）、Prompts（模板）
* **Discovery**：Client 启动时发现 Server 提供的能力清单
* **Transport**：stdio、HTTP/SSE 等传输层，与协议语义分离
* **与 Tool Calling 的关系**：MCP 是**集成层协议**；Tool Calling 是**模型 API 层的调用机制**。Agent Harness 常将 MCP Server 的工具映射为模型可见的 Tool Schema

## Related Concepts

[[Tool Calling]]
[[AI Agent]]
[[Agent Routing]]
[[Progressive Tool Disclosure]]

## Related Systems

[[Claude Code Architecture]]

Claude Code、Cursor、Codex 等均支持 MCP Client 集成。

## Common Misunderstandings

* **误解**：MCP = Tool Calling
  **实际**：Tool Calling 是 LLM 输出 structured tool invocation；MCP 是连接外部系统的协议标准

* **误解**：MCP 替代所有 SDK 集成
  **实际**：内置工具（Read、Bash 等）仍可直接在 Harness 中实现；MCP 用于**可插拔的外部能力**

* **误解**：接上 MCP 就解决了 Agent 能力问题
  **实际**：还需 Harness 层的权限门控、Tool Pool 组装、Schema 注入策略（见 [[Progressive Tool Disclosure]]）

## My Understanding

MCP 是 AI Agent 生态的「USB 接口」——价值在互操作与生态，不在协议本身多复杂。对 yuniverse-work 而言：

* **packages/mcp/** 应作为明确边界，而非各 App 各自连 MCP
* 需要区分：**内置工具** vs **MCP 扩展工具** 的权限与路由策略
* MCP Server 数量增长后，必然遇到 Progressive Disclosure 问题

## Sources

（根概念；后续可补充 MCP 官方 Spec Source 笔记）
