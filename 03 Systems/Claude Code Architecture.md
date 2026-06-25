# Claude Code Architecture

## Summary

Claude Code 是 Anthropic 的终端原生 AI Coding Agent，2026 年 3 月因打包失误泄露 ~512K 行 TypeScript 源码。本文基于泄露代码与公开分析，聚焦其**分层路由架构**：五层系统 + 五层路由，核心竞争力是工程密度而非新算法。

## WHAT

**解决什么问题？**

* 让开发者在本地终端以 Agent 方式完成软件工程任务（读文件、执行命令、编辑代码、多步推理）
* 支持多端入口（CLI、Desktop、Web、IDE、SDK）而不分裂核心逻辑
* 在生产环境稳定运行：权限门控、上下文压缩、成本控制

**面向谁？** 软件开发者、AI Agent 系统架构师

**边界：** Harness / 编排层，底层模型为 Claude（Opus/Sonnet/Haiku）；不是新 LLM

## HOW

### 五层架构

```text
入口层     CLI / Desktop / Web / IDE / SDK
运行层     REPL、状态机、Hook、TAOR 循环
引擎层     QueryEngine、动态 Prompt、上下文压缩
工具层     40+ 工具、ToolSearch、MCP、权限门控
基础设施层  认证、缓存、遥测、远程控制
```

各层双向连接，形成闭环而非线性流水线。

### 入口层：main.tsx 运行时装配器

启动前完成：环境预热、会话类型判定、上下文/命令/工具收集、LSP/MCP 初始化。

判定：交互 REPL / Pipe 批处理 / SDK 远程 / 会话恢复 → 决定工具集、上下文注入、权限策略。

### 运行层：TAOR + 模式路由

三种 [[Agent Execution Modes]]（Default / Plan / Coordinator）决定执行语义、工具权限与协作拓扑。

Act 后插入 Reflection（Reflexion 对齐）：检查预期、循环、约束遗漏。

### 引擎层：QueryEngine (~46K 行)

* 上下文拼接与管理
* Prompt Cache（14 断点 + Sticky Latch）
* 流式响应、对话压缩
* 动态 System Prompt 组装（数百碎片按条件激活）

### 工具层

~40 工具，Zod v4 Schema，工具基类 ~29K 行。Progressive Disclosure + 六层权限门控。

### 数据流（Coordinator Mode）

```text
用户任务 → Coordinator 分析
         → TeamCreateTool 创建 Worker（worktree 隔离）
         → SendMessageTool 分配子任务
         → Worker 执行 → 结构化文本回传
         → Coordinator 汇总输出
```

## WHY

* **入口统一收口**：避免 N 个入口 × N 套 Agent 逻辑的分裂
* **QueryEngine 单文件集中**：API 重试、限速、上下文预算同一处推理，避免跨模块不一致
* **固定 Hub-and-Spoke**：生产优先可预测性，而非动态拓扑最优
* **Plan Mode 工具触发**：模型自主判断何时需要规划，降低不可逆操作风险
* **Prompt Cache 断点设计**：拼接顺序即成本优化策略

## TRADE-OFF

**优势**

* 分层清晰，职责可映射到企业 Agent 架构
* 权限门控、压缩、断路器从 Demo 阶段即可参照
* 工程参考系明确（QueryEngine 46K、工具基类 29K 行量级）

**局限**

* 泄露代码已被 DMCA 下架，后续分析依赖二手资料
* Hub-and-Spoke 失去 Agent 间 P2P 通信灵活性
* Reflection 提升成功率但增加 API 调用成本
* 挫败感 / continue 等遥测指标存在伦理与隐私考量

**替代方案**

* 动态 Agent 拓扑（G-Designer 等）——更灵活，更难预测
* RAG/Embedding 代码搜索 —— Claude Code 已切换为 ripgrep agentic search
* 外部调度器硬编码路由 —— Claude Code 选择模型隐式路由

## REFLECTION

Claude Code 验证了「分层路由 + 权限门控 + 上下文压缩」是商业 Agent 的可行范式。对 yuniverse-work 桌面 Agent 的启示：

1. **入口层收敛**：Electron / Web / TUI 应共享运行时，差异在装配阶段消化
2. **模式路由先行**：Plan Mode 类机制应在架构层而非 UI 层
3. **工具按需曝光**：工具注册表 + 渐进式 Schema 注入，控制 Token 与误选
4. **工程估算**：核心 Harness 是万行级代码量，不是几百行 Demo

与 [[Cursor]]、OpenHands 等的对比待补充独立 System 笔记后交叉链接。本项目实践见 [[Yuniverse Work Desktop Architecture]]。

## Related Concepts

[[Agent Routing]]
[[Agent Execution Modes]]
[[TAOR Loop]]
[[Progressive Tool Disclosure]]
[[Dynamic System Prompt Assembly]]
[[Hub-and-Spoke Agent Topology]]
[[Plan Mode]]
[[AI Agent]]
[[MCP]]

## Related Patterns

[[Layered Agent Routing Pattern]]

## Sources

[[Claude Code Agent 路由设计 - 泄露源码分析]]
