# Tool Calling

## What Is It

Tool Calling（工具调用）是大语言模型 API 提供的一种能力：模型在推理过程中**输出结构化的工具调用请求**（工具名 + 参数），由运行时执行后将结果**回注上下文**，模型据此继续推理。

它是 AI Agent **Act** 阶段的技术基础——把「说」变成「做」的桥梁。

## Problem It Solves

* 纯文本生成无法直接操作文件系统、执行命令、调用 API
* 需要在模型推理与外部世界之间建立**可解析、可验证、可审计**的接口
* 让 Agent Loop（ReAct / TAOR）中的 Act 步骤有标准实现

## Core Ideas

典型流程：

```text
用户输入 → 模型推理 → tool_call(name, args)
         → Harness 执行工具 → tool_result
         → 回注模型 → 继续推理或终止
```

* **Tool Schema**：JSON Schema / Zod 等描述工具名、参数、说明，注入 System Prompt 或 Tool 列表
* **Parallel Tool Calls**：一次响应可发起多个工具调用
* **Harness 职责**：Schema 管理、执行、权限校验、错误处理、结果格式化——不在模型内
* **与 MCP 的关系**：Tool Calling 是模型层机制；MCP 将外部 Server 能力**适配**为模型可调的 Tool

## Related Concepts

[[MCP]]
[[AI Agent]]
[[TAOR Loop]]
[[Agent Routing]]
[[Progressive Tool Disclosure]]

## Related Systems

[[Claude Code Architecture]]

Claude Code 内置 ~40 工具 + MCP 扩展，均通过 Tool Calling 触发。

## Common Misunderstandings

* **误解**：Tool Calling = Agent
  **实际**：单次 tool call 只是一步 Act；Agent 需要 Loop、权限、上下文管理

* **误解**：工具越多越好
  **实际**：工具过多增加 Token 消耗与误选率，需 Progressive Disclosure

* **误解**：模型直接执行工具
  **实际**：模型只**请求**调用；执行永远在 Harness / 沙箱中，需权限门控

## My Understanding

Tool Calling 是「已知能力」——2013 年代的 function calling 演进至今，各厂商 API 细节不同，但模式一致。工程难点不在「能不能调工具」，而在：

1. **Schema 如何注入**（全量 vs 按需）
2. **权限如何门控**（六层门控、Plan Mode 等）
3. **失败如何恢复**（重试、Reflection、错误回注格式）

yuniverse-work 的 `packages/agent-tools` 应 owns Tool 定义与注册；App 层保持 thin。

## Sources

（根概念；后续可补充 OpenAI / Anthropic Tool Use API 文档 Source 笔记）
