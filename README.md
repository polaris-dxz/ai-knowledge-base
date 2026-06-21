# AI Knowledge Base

个人技术操作系统（Personal Technical Operating System, PTOS）

> 长期沉淀 AI Agent、AI Infrastructure、AI Platform 领域的技术认知，并演进为可复用的知识图谱。

---

## Purpose

这个仓库不是软件项目，也不是笔记堆叠器。

它是一个**个人技术操作系统**：把阅读、调研、架构分析与工程实践，转化为可关联、可复用、可输出的知识体系。

使命：

* 建立 AI Agent / AI Infrastructure / AI Platform 三个领域的长期认知框架
* 将零散信息演进为结构化知识，而非一次性记录
* 为架构分析、技术调研、项目设计与技术写作提供统一方法论
* 让未来的人与 AI Agent 都能按同一套规则持续贡献

核心原则：

```text
理解 > 收集
抽象 > 复制
知识图谱 > 孤立笔记
思维系统 > 随手记
```

---

## Knowledge Domains

### AI Agent

AI Agent 产品与框架：运行时、工具调用、权限、多 Agent 协作、人机交互。

关注对象：Codex、Claude Code、OpenHands、Cursor、MCP、Computer Use、Tool Calling、Multi-Agent

### AI Infrastructure

大模型训练与推理基础设施：调度、分布式、Serving、资源管理。

关注对象：Ray、KubeRay、LeaderWorkerSet、vLLM、Distributed Inference、Distributed Training、Model Serving

### AI Platform

企业级 AI 平台：模型生命周期、网关、多租户、治理与商业化。

关注对象：Model Registry、Inference Gateway、AI Studio、Multi-Tenant Architecture、IAM、Billing、Observability

---

## Repository Structure

```text
00 Inbox/        快速捕获，待分拣的临时内容
01 Sources/      原始资料（文章、文档、论文、RFC、开源项目）
02 Concepts/     原子概念，一概念一笔记
03 Systems/      产品 / 系统架构拆解
04 Projects/     个人工程实践与决策沉淀
05 Architecture/ 可复用的架构模式
06 Outputs/      面向发布的文章草稿
99 Templates/    笔记模板
```

| 目录 | 职责 |
|------|------|
| `00 Inbox` | 低摩擦输入；内容必须 eventually 迁出，不作永久存储 |
| `01 Sources` | 保留出处与上下文，记录原始材料与自己的阅读结论 |
| `02 Concepts` | 最小知识单元，跨系统复用 |
| `03 Systems` | 对具体产品 / 系统的完整分析 |
| `04 Projects` | 第一手工程经验、权衡与演进过程 |
| `05 Architecture` | 从多个系统中抽象出的模式 |
| `06 Outputs` | 从已有笔记衍生，而非直接从 Source 写稿 |
| `99 Templates` | 统一笔记结构，降低 Agent 与人工协作成本 |

---

## Knowledge Evolution Model

知识按层级演进，每一层都有明确职责：

```text
Source → Concept → System → Architecture → Output
```

| 层级 | 职责 |
|------|------|
| **Source** | 记录原始资料：链接、摘要、关键发现、待验证问题 |
| **Concept** | 提取原子概念：定义、解决的问题、与其他概念的关系 |
| **System** | 分析具体系统：架构、数据流、设计决策、权衡 |
| **Architecture** | 抽象可复用模式：从多个 System 中归纳通用解法 |
| **Output** | 面向受众的表达：文章、演讲、社区内容 |

规则：

* 下层为上层提供证据，上层为下层提供结构
* 不允许跳过中间层直接写 Output（除非明确标注为快速草稿，后续回补）
* 每一层都应通过 Obsidian wiki link 与相邻层连接

---

## AARRA Workflow

研究新系统时，使用 AARRA 工作流：

```text
Acquire → Abstract → Reverse → Review → Articulate
```

| 阶段 | 做什么 |
|------|--------|
| **Acquire** | 收集原始材料：官方文档、源码、论文、Issue、博客；写入 `01 Sources/` |
| **Abstract** | 提取概念与术语；写入 `02 Concepts/`，建立初步链接 |
| **Reverse** | 逆向理解系统：组件、边界、数据流、控制流；写入 `03 Systems/` |
| **Review** | 对照多个来源交叉验证；更新 Concept 与 System，识别矛盾与盲区 |
| **Articulate** | 抽象模式、形成观点、产出可发布内容；写入 `05 Architecture/` 或 `06 Outputs/` |

AARRA 与 Knowledge Evolution Model 的对应关系：

```text
Acquire   → Source
Abstract  → Concept
Reverse   → System
Review    → Concept + System（迭代）
Articulate → Architecture + Output
```

---

## Architecture Analysis Framework

系统分析统一采用五段式框架：

```text
WHAT → HOW → WHY → TRADE-OFF → REFLECTION
```

| 维度 | 问题 |
|------|------|
| **WHAT** | 系统解决什么问题？面向谁？边界是什么？ |
| **HOW** | 架构如何组织？核心组件？请求 / 数据如何流动？ |
| **WHY** | 为何这样设计？关键设计决策背后的约束与假设？ |
| **TRADE-OFF** | 优势与局限？放弃了什么？有哪些替代方案？ |
| **REFLECTION** | 我的理解、与类似系统的对比、可借鉴与可改进之处 |

此框架用于 `03 Systems/` 与 `04 Projects/` 中的分析笔记。

---

## Long-Term Goal

这个仓库应逐步演进为：

1. **Personal Knowledge Graph** — AI Agent / Infra / Platform 三域互联的概念网络
2. **Reusable Architecture Library** — 从真实系统中提炼、经多次验证的架构模式库
3. **Technical Writing System** — 从笔记到发布的流水线，支持知乎、掘金、CSDN、X、GitHub

最终目标不是「记了多少笔记」，而是构建一套**可复用的技术思维系统**。

---

## For AI Agents

AI Agent 的操作规范见 [AGENTS.md](./AGENTS.md)。
