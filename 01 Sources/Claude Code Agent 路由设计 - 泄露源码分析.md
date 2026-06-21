# Claude Code Agent 路由设计 - 泄露源码分析

## Source

知乎专栏 · 【技术综述与趋势】从 51 万行泄露源码，看 Anthropic 如何工程化落地 Agent 路由

## Author

（专栏作者，待补充）

## Link

- 原文（可访问）：https://zhuanlan.zhihu.com/p/2028527517414302665
- 用户提供链接（抓取时被知乎 403 拦截）：https://zhuanlan.zhihu.com/p/2038336345500819809

> 若两链接不是同一篇文章，请补充正文后更新本 Source 笔记。

## Reading Date

2026-06-21

## Summary

2026 年 3 月，Anthropic 因 `.npmignore` 配置遗漏，Claude Code 约 51.2 万行 TypeScript 源码被意外公开。本文聚焦其中**路由设计**：从入口层到多 Agent 委派层逐层拆解，并与 ReAct、Reflexion 等学术框架对照。

核心论点：Claude Code 的竞争力不在于新算法，而在于把已知 idea **高密度工程化**——QueryEngine ~46K 行、工具基类 ~29K 行、六层权限门控、渐进式工具曝光、动态 System Prompt 组装。

## Key Findings

### 五层架构

| 层级 | 职责 |
|------|------|
| 入口层 | CLI / Desktop / Web / IDE / SDK 统一收口 |
| 运行层 | REPL、状态机、Hook、TAOR 循环 |
| 引擎层 | QueryEngine、动态 Prompt、上下文压缩 |
| 工具层 | 40+ 工具、ToolSearch、MCP、权限门控 |
| 基础设施层 | 认证、缓存、遥测、远程控制 |

### 五层路由

1. **入口路由**：`main.tsx` 作为运行时装配器，判定 REPL / Pipe / SDK / 会话恢复，完成环境预热、工具与 MCP 初始化
2. **执行路由**：TAOR 循环内隐式决策继续/终止；区分 Default / Plan / Coordinator 三种模式
3. **引擎路由**：数百 Prompt 碎片按模式、工具、项目配置动态组装；14 个 Prompt Cache 断点
4. **工具路由**：`assembleToolPool()` + `defer_loading` + ToolSearch，渐进式曝光
5. **反馈路由**：挫败感指标、continue 计数器驱动隐式策略改进

### 三种执行模式

- **Default**：标准 TAOR，工具有真实副作用
- **Plan Mode**：只读工具，通过 Enter/ExitPlanMode 触发，输出计划供人审查
- **Coordinator Mode**：Hub-and-Spoke，Coordinator + Worker（git worktree 隔离），结构化文本回传

### 工程数据点

- QueryEngine.ts ≈ 46,000 行
- 工具基类 ≈ 29,000 行
- 内置 ~40 工具，Zod v4 Schema
- Reflection 环节对齐 Reflexion 框架（2023）

## Questions

- [ ] 泄露源码当前是否已被 DMCA 完全下架？后续分析需依赖二次资料？
- [ ] Coordinator Mode 与后续 Agent Teams、`/workflows` 的演进关系？
- [ ] 六层权限门控的具体层级划分需对照官方文档或源码片段验证
- [ ] 用户提供的 URL 与本文是否为同一篇文章？

## Related Concepts

[[Agent Routing]]
[[TAOR Loop]]
[[Progressive Tool Disclosure]]
[[Dynamic System Prompt Assembly]]
[[Hub-and-Spoke Agent Topology]]
[[Plan Mode]]
[[AI Agent]]

## Related Systems

[[Claude Code Architecture]]

## Related Patterns

[[Layered Agent Routing Pattern]]
