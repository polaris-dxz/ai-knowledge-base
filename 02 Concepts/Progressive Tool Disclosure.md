# Progressive Tool Disclosure

## What Is It

Progressive Tool Disclosure（渐进式工具曝光）是一种工具路由策略：不一次性将全部工具 Schema 注入上下文，而是按会话特征组装 Tool Pool，部分工具延迟加载，并允许模型通过 ToolSearch 按需发现。

## Problem It Solves

* 40+ 工具全量描述浪费 Token，干扰模型选择
* 长会话中 Prompt 体积膨胀
* 工具误选率随选项增多而上升

## Core Ideas

Claude Code 的三段式实现：

1. **`assembleToolPool()`**：按上下文选出可能相关的工具子集
2. **`defer_loading: true`**：部分工具详细描述按需注入
3. **ToolSearch**：模型主动搜索工具注册表

两阶段路由：

```text
粗路由：确定能力大类（文件 / 搜索 / Shell / Agent 协作）
细路由：注入该类工具 Schema → 精确选择
```

## Related Concepts

[[Agent Routing]]
[[Dynamic System Prompt Assembly]]
[[MCP]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：渐进式曝光 = 减少工具数量
  **实际**：工具池仍在，只是描述按需出现；模型仍可搜索全注册表

## My Understanding

这是 UI「按需展示」原则在 Agent 工具层的迁移。对企业多意图场景同样适用：先粗分业务大类，再类内精细分类，降低误分类率。与 Prompt Cache 配合时，不变的工具描述前缀可最大化缓存命中。
