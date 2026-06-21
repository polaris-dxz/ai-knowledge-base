# Computer Use

## What Is It

Computer Use 是 AI Agent 通过**图形界面、浏览器或操作系统 API** 操作计算机的能力：点击、输入、滚动、截图、切换应用等，而不仅限于终端命令或文件 API。

它把 Agent 的「行动空间」从开发者工具链扩展到**通用桌面/浏览器环境**，是 Tool Calling 在 UI 层的延伸。

## Problem It Solves

* 大量任务发生在 Web App、桌面 GUI 中，纯 Shell/文件工具无法覆盖
* 非技术用户场景需要「像人一样操作软件」，而非写脚本
* 自动化测试、RPA、跨应用工作流需要视觉 + 交互能力

## Core Ideas

* **感知**：截图 / DOM /  accessibility tree 作为观察输入
* **行动**：坐标点击、键盘输入、浏览器导航等 structured actions
* **循环**：与 TAOR 相同——观察界面状态 → 决定下一步操作 → 执行 → 再观察
* **与 Tool Calling 的关系**：UI 操作封装为工具（如 `click`、`type`、`screenshot`），仍由 Harness 执行
* **安全边界**：屏幕访问、敏感信息、不可逆操作需要权限门控（类比 [[Plan Mode]]）

## Related Concepts

[[AI Agent]]
[[Tool Calling]]
[[TAOR Loop]]
[[Plan Mode]]

## Related Systems

[[Claude Code Architecture]]（以终端为主，Computer Use 能力有限）

待补充：Anthropic Computer Use API、OpenAI Operator、各浏览器 Agent 的 System 笔记。

## Common Misunderstandings

* **误解**：Computer Use = 远程桌面
  **实际**：是 Agent 按步骤决策操作，非连续流式遥控

* **误解**：Computer Use 替代 Bash/文件工具
  **实际**：GUI 操作慢、脆弱、Token 贵；工程任务仍优先终端工具

* **误解**：截图足够理解 UI
  **实际**：常需结合 DOM / a11y tree；纯视觉在复杂布局上易出错

## My Understanding

Computer Use 适合**低结构、跨应用、以 UI 为接口**的任务；软件工程 Agent（如 Claude Code）仍以 Harness + 专用工具为主。两者边界：

| 场景 | 更合适 |
|------|--------|
| 改代码、跑测试、Git | 终端 / 文件工具 |
| 填表单、操作 SaaS、无 API 的 legacy 系统 | Computer Use |

yuniverse-work 桌面端若做 Electron Agent，Computer Use 可能与 **BrowserView / 内嵌 Web** 结合，但不应替代 `packages/agent-tools` 的一等工具。

## Sources

（根概念；后续补充官方 Computer Use 文档 Source 笔记）
