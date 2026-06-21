# Dynamic System Prompt Assembly

## What Is It

Dynamic System Prompt Assembly 指运行时根据**执行模式、工具集合、项目配置**等上下文特征，从数百 Prompt 碎片中动态组装 System Prompt，而非使用单一固定系统提示词。

## Problem It Solves

* 不同模式（Default / Plan / Coordinator）需要不同约束与能力描述
* 不同项目（CLAUDE.md、MCP、LSP）需要不同上下文注入
* 固定 Prompt 无法兼顾所有场景，全量拼接则 Token 成本过高

## Core Ideas

* 请求上下文特征 → 决定哪些 Prompt 碎片被「路由激活」
* 与 **Prompt Caching** 配合：14 个缓存断点 + Sticky Latch，不变内容放断点前，动态内容放断点后
* 缓存失效 = 真实 API 成本增加，拼接顺序本身是路由优化问题

## Related Concepts

[[Agent Routing]]
[[Progressive Tool Disclosure]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：动态组装 = 每次完全重写 System Prompt
  **实际**：是碎片池 + 条件激活，大量前缀可缓存复用

## My Understanding

QueryEngine（~46K 行）集中管理所有模型 API 相关逻辑（重试、限速、上下文预算），避免跨模块状态不一致——这是「把 Prompt 路由逻辑放在一个文件里」的工程理由，而非架构洁癖。
