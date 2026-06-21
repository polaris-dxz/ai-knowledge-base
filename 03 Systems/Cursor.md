# Cursor

## Summary

Cursor 是基于 VS Code 的 **AI-native IDE**，将 Agent 能力嵌入编辑器：Tab 补全、Chat、Composer/Agent 模式、Rules、MCP 集成。在本知识库中作为 **AI Agent 域** 的代表产品之一，与 [[Claude Code Architecture]] 形成 CLI vs IDE 的对照。

## WHAT

**解决什么问题？**

* 在编码工作流中减少上下文切换：编辑、对话、Agent 执行同一窗口完成
* 让开发者用自然语言驱动代码理解、修改、重构、调试

**面向谁？** 软件开发者（个人与团队）

**边界：** IDE + Agent Harness；底层模型来自 Anthropic/OpenAI 等第三方；不是独立 LLM

## HOW

### 架构概览（公开信息 + 产品行为）

```text
VS Code 基座
    ↓
Cursor 扩展层（AI 功能、Rules、Indexing）
    ↓
Agent 模式（Composer / Agent）— 工具调用、多文件编辑
    ↓
MCP Client — 外部能力扩展
    ↓
模型 API（Claude、GPT 等）
```

### 核心组件

* **Tab / Inline**：低延迟补全，局部上下文
* **Chat**：对话式问答，可选 @ 文件/文档
* **Agent / Composer**：多步任务，读写仓库、终端、MCP
* **Rules / .cursor/rules**：项目级 Agent 指令（类似 CLAUDE.md）
* **Codebase Indexing**：语义检索（与 Claude Code 的 ripgrep 策略不同）
* **MCP**：插件式外部工具与资源

### 数据流（Agent 任务）

```text
用户意图 → 上下文组装（打开文件、Rules、Index）
        → 模型推理 → Tool Call（读/写/终端/MCP）
        → 结果回注 → 循环或完成
```

## WHY

* **基于 VS Code**：降低迁移成本，复用扩展生态
* **Indexing**：大仓库下提供语义搜索，弥补纯 grep 的不足（代价：索引同步、隐私）
* **Rules 一等公民**：把项目规范注入 Agent，减少每轮重复说明
* **多模型**：用户可选供应商，避免单模型锁定

## TRADE-OFF

**优势**

* IDE 内闭环，开发者 UX 成熟
* 语义 Index + @ 引用，大项目友好
* MCP、Rules 与团队工作流可结合

**局限**

* 黑盒 Harness，不如 Claude Code 泄露源码可深度分析
* Index 有隐私与同步成本；企业场景需额外治理
* Agent 行为受 VS Code 工作区模型约束，非纯终端 Harness

**与 Claude Code 对比（待深入 Review）**

| 维度 | Cursor | Claude Code |
|------|--------|-------------|
| 入口 | IDE | CLI / Desktop / IDE 插件 |
| 代码搜索 | Index + 工具 | ripgrep agentic search 为主 |
| 配置 | .cursor/rules | CLAUDE.md + settings |
| 可分析性 | 闭源 | 曾有源码泄露可参照 |

## REFLECTION

Cursor 代表 **「Agent 嵌入现有工具」** 路径：不另造运行时，而是把 Harness 叠在 IDE 上。对 yuniverse-work 桌面 App：

* 可参考 Rules、MCP、Agent 模式的产品形态
* Harness 工程仍应对标 Claude Code 的分层路由，而非仅 UI 层包装
* 宜在 `03 Systems/` 与 [[Claude Code Architecture]] 成对维护，做架构对比

（个人理解，待补充一手调研与 Source 笔记）

## Related Concepts

[[AI Agent]]
[[MCP]]
[[Tool Calling]]
[[Agent Routing]]
[[Multi-Agent]]

## Related Systems

[[Claude Code Architecture]]

## Sources

待补充：官方文档、Changelog、深度测评 Source 笔记。

[[Claude Code Agent 路由设计 - 泄露源码分析]]（对比参照）
