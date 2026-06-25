# TAOR Loop

## What Is It

TAOR（Think → Act → Observe → Repeat）是 Claude Code 运行层的核心循环，ReAct（Yao et al., 2022）的工程化实现：模型推理 → 工具行动 → 观察结果 → 重复，直到任务完成或终止。

## Problem It Solves

* 让 Agent 在真实环境（文件系统、Shell）中**迭代**解决问题，而非一次性生成
* 每轮 Act 的结果反馈到下一轮 Think，形成闭环

## Core Ideas

* 路由嵌入循环内：每轮隐式决定「继续执行」或「终止输出」
* Act 完成后、下一轮 Think 前插入 **Reflection**（对齐 Reflexion 框架）：检查是否达预期、是否循环、是否遗漏约束
* Reflection 是自适应路由节点：检测到偏差则修正下一路径

## Related Concepts

[[Agent Routing]]
[[Agent Execution Modes]]
[[Plan Mode]]
[[AI Agent]]

## Related Systems

[[Claude Code Architecture]]

## Common Misunderstandings

* **误解**：TAOR = 固定步数循环
  **实际**：终止条件由模型输出隐式决定，无外部硬编码调度器

## My Understanding

TAOR 本身不是 2026 年的新 idea，但 Claude Code 展示了如何把 ReAct + Reflection 与 [[Agent Execution Modes]]、权限门控组合成**可长期运行的生产循环**。Reflection 的额外 API 成本是显式 trade-off。
