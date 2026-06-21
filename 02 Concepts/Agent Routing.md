# Agent Routing

## What Is It

Agent Routing 指 AI Agent 系统中**决定请求走向、执行模式、工具曝光与 Prompt 组装**的一整套调度机制。它不是单一的「意图分类器」，而是分布在入口、执行循环、引擎、工具层的多层路由。

## Problem It Solves

* 多端入口（CLI、Web、IDE）若各自维护 Agent 逻辑，系统必然分裂
* 全量工具 / 全量 Prompt 注入会浪费 Token 并增加误选
* 高风险操作需要执行路径上的风险拦截，而非事后补救
* 多 Agent 协作需要可预测的任务分发拓扑

## Core Ideas

* **入口收敛**：差异在装配阶段吸收，核心逻辑只看标准化运行时
* **循环内路由**：TAOR 每轮隐式决定继续工具调用或终止
* **模式路由**：Default / Plan / Coordinator 切换执行语义与工具权限
* **Prompt 路由**：按上下文特征动态激活 Prompt 碎片
* **工具路由**：粗粒度能力感知 → 细粒度工具选择（Progressive Disclosure）
* **反馈路由**：用户行为信号（挫败感、continue 频率）驱动策略调整

## Related Concepts

[[TAOR Loop]]
[[Progressive Tool Disclosure]]
[[Dynamic System Prompt Assembly]]
[[Hub-and-Spoke Agent Topology]]
[[Plan Mode]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：Agent Routing = NLU 意图分类
  **实际**：商业 Agent 的路由更多是分层工程机制，意图分类只是其中一环

* **误解**：路由算法越动态越好
  **实际**：Claude Code 刻意使用固定 Hub-and-Spoke，优先可预测性

## My Understanding

路由是 Agent 产品从 Demo 到生产的分水岭。Demo 只关心「模型能不能调工具」；生产系统必须回答「从哪个入口、以什么模式、暴露哪些工具、注入哪些 Prompt、如何门控权限、如何压缩上下文」。Claude Code 泄露源码的价值在于证明：**这些已知问题都需要万行级工程代码来承接**，而非一个 clever 的路由模型。
