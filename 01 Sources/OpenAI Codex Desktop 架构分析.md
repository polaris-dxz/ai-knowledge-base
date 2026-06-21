
## 基本信息

|项|内容|
|---|---|
|产品|OpenAI Codex Desktop|
|平台|macOS|
|类型|AI Agent 桌面应用|
|技术栈|Electron + React + Rust + WASM + MCP|
|来源|《逆向 OpenAI Codex 桌面应用架构深度解读》|
|整理时间|2026-06-21|

---

# 一、总体架构

Codex Desktop 并不是传统意义上的 AI Chat 应用。

它本质上是一个具备：

- 文件系统访问
    
- Shell 执行
    
- 浏览器控制
    
- MCP 插件扩展
    
- Desktop Automation（Computer Use）
    

能力的 Agent Runtime。

整体架构：

```text
┌────────────────────────────┐
│ React Renderer             │
│ Chat UI / Editor UI        │
└─────────────┬──────────────┘
              │ IPC
┌─────────────▼──────────────┐
│ Electron Main Process      │
│ Window / Permission / IPC  │
└─────────────┬──────────────┘
              │
┌─────────────▼──────────────┐
│ Worker Process             │
│ Network / FS / Subprocess  │
└─────────────┬──────────────┘
              │
┌─────────────▼──────────────┐
│ MCP Runtime                │
└─────────────┬──────────────┘
              │
┌─────────────▼──────────────┐
│ Plugins                    │
│ Browser / Chrome / CUA     │
└────────────────────────────┘
```

核心思想：

Agent Runtime + MCP Tool Ecosystem。

---

# 二、多进程架构

## Main Process

职责：

- Window 管理
    
- IPC 路由
    
- 原生能力调用
    
- 自动更新
    
- Crash 上报
    

Main Process 是整个应用的控制中心。

---

## Renderer Process

技术：

- React
    
- Chromium
    

职责：

- Chat 界面
    
- 设置页面
    
- 编辑器界面
    

特点：

内部嵌入 .NET WASM Runtime。

用于：

- Word 解析
    
- Excel 解析
    
- PowerPoint 解析
    

实现本地 Office 文件预览。

---

## Worker Process

职责：

- HTTP 请求
    
- 文件读写
    
- TCP/TLS 通信
    
- 子进程管理
    
- 加密操作
    

目的：

避免阻塞 UI。

---

# 三、Session 隔离

Codex 为不同场景创建独立 Session：

```text
persist:main
persist:worker
persist:webview
persist:thread-{id}
```

意味着：

每个 Thread 拥有独立：

- Cookie
    
- Cache
    
- LocalStorage
    

实现会话级隔离。

优势：

- 减少状态污染
    
- 提高隐私性
    
- 降低任务间干扰
    

---

# 四、IPC 通信体系

Renderer 无法直接访问 Node API。

因此通过：

```javascript
electronBridge
```

与 Main Process 通信。

核心能力：

- 状态同步
    
- Worker 调度
    
- 菜单控制
    
- 系统通知
    
- 配置获取
    

---

## SharedObject

共享状态中心：

```text
Renderer A
      │
Renderer B
      │
Renderer C
      │
      ▼
 SharedObject
      ▲
      │
 Main Process
```

特点：

- 任意窗口更新
    
- 全局同步广播
    

类似：

- Redux Store
    
- Zustand Store
    
- Electron Global State
    

---

# 五、WASM 架构设计

Codex 中存在两类 WASM。

---

## 前端 WASM

运行环境：

Renderer

用途：

Office 文档渲染。

例如：

- OpenXML
    
- Excel
    
- Word
    

实现浏览器内解析。

---

## Rust WASM

运行环境：

Main Process

用途：

核心业务逻辑。

原因：

Rust 提供：

- 更高性能
    
- 更高安全性
    
- 更低内存风险
    

体现出：

JS 负责界面，  
Rust 负责核心能力。

---

# 六、MCP 插件体系

Codex 的扩展体系建立在 MCP 之上。

---

## 第一层 Marketplace

插件市场。

负责：

- 插件发现
    
- 插件安装
    
- 插件更新
    

---

## 第二层 Plugin

描述文件：

```text
.codex-plugin/plugin.json
```

包含：

- 名称
    
- 作者
    
- MCP Server
    
- 权限
    
- Skill
    

---

## 第三层 Skill

文件：

```text
SKILL.md
```

作用：

告诉 Agent：

- 什么时候使用工具
    
- 如何使用工具
    
- 使用限制是什么
    

本质：

Tool Documentation for LLM。

---

# 七、内置插件

## browser-use

控制内嵌浏览器。

支持：

- 打开页面
    
- 点击元素
    
- 截图
    
- DOM 操作
    

---

## chrome

控制真实 Chrome 浏览器。

能力：

访问用户已登录网站。

例如：

- GitHub
    
- Notion
    
- Gmail
    

风险显著提升。

---

## computer-use

桌面自动化能力。

核心卖点。

可直接控制 macOS。

---

## latex-tectonic

LaTeX 编译器。

负责：

- PDF 生成
    
- 数学公式渲染
    

---

# 八、Computer Use 架构

这是 Codex 最重要的能力。

---

## 架构设计

采用双进程模式。

### Service

```text
SkyComputerUseService
```

拥有系统权限：

- Accessibility
    
- Screen Recording
    

负责：

- 获取界面树
    
- 获取截图
    
- 模拟鼠标
    
- 模拟键盘
    

---

### Client

```text
SkyComputerUseClient
```

作为 MCP Server。

负责：

- MCP 通信
    
- 请求转发
    

不直接拥有系统权限。

---

## 为什么拆分？

核心原则：

最小权限原则。

即使 Client 被攻击：

攻击者仍无法直接控制桌面。

必须突破 Service。

---

# 九、Computer Use 工作模式

Agent 的执行流程：

```text
观察
 ↓
分析
 ↓
行动
 ↓
验证
 ↓
继续
```

具体：

```text
get_app_state
 ↓
分析 UI
 ↓
click
 ↓
type_text
 ↓
验证结果
```

本质：

Agent Loop。

即：

Observe → Think → Act → Verify

---

# 十、Computer Use 提供的能力

主要 MCP Tool：

|Tool|功能|
|---|---|
|list_apps|获取应用列表|
|get_app_state|获取界面状态|
|click|点击|
|set_value|输入内容|
|scroll|滚动|
|drag|拖拽|
|press_key|按键|
|type_text|输入文字|

已经具备完整桌面操作能力。

---

# 十一、安全设计

## 第一层

macOS TCC

需要用户授权：

- Accessibility
    
- Screen Recording
    

---

## 第二层

应用级审批。

首次访问：

```text
Allow Codex to use Spotify ?
```

用户确认后才允许操作。

---

## 第三层

密码管理器封禁。

例如：

- 1Password
    
- Bitwarden
    
- LastPass
    

永远不允许控制。

---

# 十二、关键设计模式

## 1. Multi Process

Electron 多进程隔离。

---

## 2. MCP First

所有能力统一抽象为 MCP Tool。

---

## 3. Skill Driven

工具通过 Skill 告诉 Agent 如何使用自己。

---

## 4. Human In The Loop

关键操作需要用户批准。

---

## 5. Turn Scoped Permission

每轮对话结束：

Computer Use Session 自动结束。

避免长期持有控制权限。

---

# 十三、架构启示

## MCP 正在成为 Agent 标准

未来 Agent 产品：

```text
Agent Runtime
      ↓
MCP
      ↓
Tools
```

会逐渐成为统一架构。

---

## Computer Use 本质也是 MCP

不是特殊能力。

实际上：

```text
Computer Use
=
Accessibility API
+
Screen Capture
+
Input Simulation
+
MCP
```

---

## Agent 产品正在收敛

Claude Code

Codex

OpenClaw

Cursor Agent

基本都在收敛到：

```text
Frontend
 ↓
Agent Runtime
 ↓
MCP Layer
 ↓
Tool Layer
```

---

## 安全比模型更重要

模型能力已经足够。

真正困难的是：

- Prompt Injection
    
- Credential Security
    
- Tool Permission
    
- Audit
    
- Human Approval
    

未来 Agent 平台的核心竞争力，很可能不再是模型，而是安全边界设计能力。

---

# 一句话总结

Codex Desktop 的本质不是一个 AI 编程工具，而是一个以 MCP 为核心、具备桌面自动化能力的 Agent Runtime 平台；其架构重点不在模型，而在权限控制、工具编排和安全边界设计。

---

## Related Projects

[[Yuniverse Work Desktop Architecture]]（本项目桌面端实践，对照参考本文架构）