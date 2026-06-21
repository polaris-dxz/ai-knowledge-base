# Layered Agent Routing Pattern

## Pattern

Layered Agent Routing Pattern — 在 Agent 系统的多个层级分别嵌入路由决策，而非依赖单一意图分类器或外部调度器。

## Problem

* 多端入口导致 Agent 逻辑分裂
* 全量工具 / Prompt 注入不可扩展
* 高风险操作缺乏执行路径拦截
* Demo 级 Agent 无法直接上生产

## Solution

在五层架构中分别实现路由：

```text
L1 入口路由    运行时装配器 → 会话类型、工具集、权限策略
L2 执行路由    TAOR 循环 + 模式切换（Default / Plan / Coordinator）
L3 引擎路由    动态 Prompt 组装 + Prompt Cache 断点优化
L4 工具路由    Tool Pool 组装 + 渐进式曝光 + ToolSearch
L5 反馈路由    行为遥测 → 策略调整（挫败感、continue 频率）
```

## Advantages

* 每层路由职责单一，可独立演进
* 入口多样、运行时可收敛
* Token 与成本可控（按需加载 + 缓存）
* 生产可预测性高（固定拓扑 + 显式模式）

## Limitations

* 工程复杂度高（Claude Code QueryEngine  alone ~46K 行）
* 跨层调试困难
* 反馈路由依赖埋点设计，指标选择有主观性

## Typical Implementations

* [[Claude Code Architecture]] — 商业 Agent 参考实现
* 企业 AI：钉钉 / 飞书 / Web / 小程序 入口 → 统一 Agent Runtime

## Applicable Scenarios

* 多端 Agent 产品
* 工具数量 > 20 的长会话 Agent
* 需要 Plan-then-Execute 或 Multi-Agent 协作的生产系统
* 从 Demo 向生产迁移的 Enterprise AI 平台

## Related Systems

[[Claude Code Architecture]]

## Related Concepts

[[Agent Routing]]
[[Progressive Tool Disclosure]]
[[Hub-and-Spoke Agent Topology]]

## Sources

[[Claude Code Agent 路由设计 - 泄露源码分析]]
