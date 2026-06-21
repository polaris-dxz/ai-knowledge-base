# Yuniverse Work Desktop Architecture

> 工程实践笔记 · `apps/desktop`（`@yuniverse/desktop`）架构设计  
> 代码仓库：`yuniverse-work` monorepo  
> 最后更新：2026-06-21（含 Renderer 模块拆分）

## Summary

Yuniverse Work 桌面端是一个 **薄 Electron 壳 + 厚 packages 运行时** 的多 Agent AI Workbench。App 层负责窗口、IPC、OS 集成与 React UI；Agent 核心逻辑下沉到 `packages/*`。运行时通过 **Bridge 策略** 对接 Yuniverse Gateway 或 Claude Code，UI 通过 **快照推送** 与 Agent 解耦。

**前端（Renderer）** 采用单页 Workbench + `openDrawer` 状态导航，按 **12 个功能模块** 组织（见下文「Renderer 模块架构」）。

---

## WHAT

### 解决什么问题

* 提供桌面原生体验的 AI Agent 工作台：对话、终端、Skills、Plugins、MCP
* 在本地运行 Agent 任务，支持多 Task、Workspace 绑定、权限确认
* 打包后独立分发，不依赖 monorepo 源码树（捆绑 Bun/Node/Python/Git + Gateway/Session 服务）

### 面向谁

* 开发者 / 知识工作者（本地 Agent 协作）
* monorepo 贡献者（`apps/desktop` 为产品入口，packages 为能力层）

### 边界

| 在范围内 | 不在范围内 |
|----------|------------|
| Electron 三进程、IPC、打包 | Agent 运行时实现（→ `packages/agent-core`） |
| React Workbench UI | GUI 状态机核心（→ `packages/agent-gui`） |
| Bridge 适配 Gateway / Claude Code | Gateway 协议与服务端（→ `packages/gateway`） |
| MCP 安装/配置/探测 UI | MCP 协议实现（→ `packages/mcp`） |

OpenSpec 约束：**Electron IPC 不得泄漏到 shared packages**（`openspec/specs/desktop-app/spec.md`）。

---

## HOW

### 总体分层

```text
┌─────────────────────────────────────────────────────────┐
│  Renderer (React 19 + Tailwind 4)                        │
│  Workbench · Composer · Settings · MCP/Skills/Plugins UI  │
└───────────────────────────┬─────────────────────────────┘
                            │ window.yuniverseDesktop
                            │ (preload contextBridge)
┌───────────────────────────▼─────────────────────────────┐
│  Preload (src/preload.ts)                               │
│  invoke/on · 白名单 API · 无 nodeIntegration            │
└───────────────────────────┬─────────────────────────────┘
                            │ ipcRenderer ↔ ipcMain
┌───────────────────────────▼─────────────────────────────┐
│  Main Process (src/main/)                               │
│  IPC handlers · Bridge · MCP registry · Cron · OS 能力   │
└───────────────────────────┬─────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        ▼                                       ▼
┌───────────────────┐                 ┌───────────────────────┐
│ yuniverse-bridge  │                 │ claude-code-bridge    │
│ GatewayClient     │                 │ ClaudeCodeSessionRuntime│
└─────────┬─────────┘                 └───────────┬───────────┘
          ▼                                       ▼
   packages/gateway                          integrations/claude-code
   packages/agent-core                       (迁移源，保持可运行)
   packages/agent-gui
   packages/config / storage / mcp / telemetry
```

### 目录结构

```text
apps/desktop/
├── src/main/           # 主进程（窗口、IPC、Bridge、MCP、Cron、系统能力）
├── src/preload.ts      # 唯一渲染层 API 暴露点
├── src/ipc/channels.ts # IPC 通道名 + payload 类型（对齐 @yuniverse/protocol）
├── src/shared/         # 跨进程轻量常量/类型
├── src/renderer/       # React UI（workbench 为中心）
├── scripts/            # build / pack / publish
├── pack-resources/     # prepare-pack 生成的运行时（gitignore）
├── vite.config.ts      # Main + Preload 构建
└── vite.renderer.config.ts
```

Renderer 与 Main/Bridge 分节描述：**Renderer 模块架构**（下文）· **HOW — Main / Bridge / IPC**（后文）。

---

## Renderer 模块架构

> 路径基准：`apps/desktop/src/renderer/`  
> 导航：**无 URL Router**，由 `workbench-context` 的 `openDrawer` + overlay dialogs 驱动。

### 模块总览

```text
renderer/
├── app.tsx                    # 入口 · Sentry · 懒加载 Workbench
├── hooks/                     # IPC 桥接层
├── lib/                       # 持久化 & 纯逻辑（~50 模块）
└── components/
    ├── ui/                    # 基础组件（shadcn 风格）
    ├── markdown/              # Markdown 渲染
    ├── plugin/                # Plugin Provider
    └── workbench/             # 业务模块（本节后文按功能拆分）
```

| # | 模块 | 职责 | 主要 UI | 状态层 | 持久化 / IPC |
|---|------|------|---------|--------|--------------|
| M1 | **Shell & 导航** | 布局、Provider 嵌套、drawer 路由 | `workbench.tsx`, `sidebar.tsx` | `workbench-context`, `sidebar-context` | `yw-sidebar-collapsed` |
| M2 | **Conversation** | 消息列表、Agent 块渲染 | `conversation.tsx`, `message-*`, `agent-*-section` | runtime → `messages` | Web: `task-message-storage` |
| M3 | **Composer** | 输入、附件、slash、发送/停止 | `composer.tsx`, `composer-plus-menu`, `composer-queue` | context: input/chips/queue | `permission-storage`, IPC `sendMessage` |
| M4 | **Tasks & Projects** | 任务切换、项目、工作目录 | `sidebar`, `task-palette`, `welcome-view`, `new-project-*` | context: activeTaskId/recent | `task-storage`, `project-storage`, IPC `switchTask` |
| M5 | **Permissions** | 模式选择 + 运行时弹窗 | `permission-control`, `permission-prompt` | context + runtime prompt | `permission-storage`, IPC `respondPermission` |
| M6 | **Activity** | 工具时间线、Plan、Token 用量 | `activity-panel`, `context-usage-indicator` | runtime: taskPlan/progress | — |
| M7 | **Skills** | 市场 / 安装 / 启用 | `skills-center`, `skill-detail-dialog` | runtime skills + local prefs | `skill-storage`, IPC `readSkillDocument` |
| M8 | **MCP / Connectors** | MCP 市场、安装、运行时 toggle | `connectors-center`, `installed-mcp-*` | runtime connectors + stubs | `mcp-storage`, IPC CRUD + `toggleMcpServer` |
| M9 | **Expert Suites** | 专家套件市场 | `expert-suites-center` | catalog + prefs | `expert-suite-storage` |
| M10 | **Plugins & Theme** | 插件 catalog、主题/branding | `plugin-provider`, `plugins-panel` | `PluginProvider` | IPC `listPlugins`, `activateThemePlugin` |
| M11 | **Hooks** | Agent 生命周期钩子配置 | `hooks-panel`, `hooks-hook-form` | 本地编辑态 | IPC `get/saveHooksConfig` |
| M12 | **Scheduled Tasks** | Cron 任务与运行历史 | `scheduled-tasks-center` | 组件 state + IPC | `cron-task-storage` → main 磁盘 |
| M13 | **IM Channels** | 飞书/钉钉/企微等频道 | `im-channels-center`, `channel-config-dialog` | connections store | `channel-connections-store` |
| M14 | **Settings** | 偏好、系统、运行时、开发者 | `settings-dialog` + 各 `*-panel` | lang/theme/system/shortcuts | localStorage + IPC `get/setSystemSettings` |

> 说明：**无独立 Terminal 应用**；Shell 输出嵌入消息流（`bash-output-panel`）；Bundled runtimes 在 Settings → Runtimes 管理。

### M1 — Shell & 导航

**Provider 嵌套（自外向内）：**

```text
SentryErrorBoundary
└── WorkbenchProvider          # workbench-context.tsx (~1570 行，UI 编排中枢)
    └── PluginProvider
        └── SidebarProvider
            └── WorkbenchInner # workbench.tsx
```

**布局模式：**

```text
openDrawer == null（主聊天）:
┌──────────┬─────────────────────────────┬──────────────┐
│ Sidebar  │ TaskHeader                  │ ActivityPanel│
│ 新建任务  │ AgentPlanSection (optional) │ (optional)   │
│ 扩展入口  │ Conversation                │              │
│ 最近任务  │ Composer                    │              │
└──────────┴─────────────────────────────┴──────────────┘

openDrawer != null（全屏 Center，Sidebar 隐藏）:
  expertSuites | skills | connectors | imChannels
  scheduledTasks | settings
```

**Drawer 路由表：**

| `openDrawer` | 组件 |
|--------------|------|
| `null` | 主聊天 / `WelcomeView` |
| `"expertSuites"` | `ExpertSuitesCenter` |
| `"skills"` | `SkillsCenter` |
| `"connectors"` | `ConnectorsCenter` |
| `"imChannels"` | `ImChannelsCenter` |
| `"scheduledTasks"` | `ScheduledTasksCenter` |
| `"settings"` | `SettingsPage` |

**Overlay（非 drawer）：** `TaskPalette`、`NewProjectDialog`、`AboutDialog`、`FeedbackDialog`、`ChannelConfigDialog`

**关键文件：** `workbench.tsx`, `workbench-context.tsx`, `sidebar-context.tsx`, `use-workbench-shortcuts.ts`

---

### M2 — Conversation（对话）

渲染 Agent 与用户消息流，消费 runtime 快照映射后的 `WorkMessage`。

| 组件 | 职责 |
|------|------|
| `conversation.tsx` | 列表滚动、TaskHeader |
| `message-content.tsx` | thinking / tool / plan 块 |
| `agent-plan-section.tsx` | Plan 折叠区 |
| `agent-thinking-section.tsx` | Thinking 折叠区 |
| `agent-subagent-section.tsx` | Subagent 折叠区 |
| `bash-output-panel.tsx` | Shell 输出（非独立 Terminal） |
| `markdown-view.tsx` | Markdown |

**Lib：** `parse-message-content.ts`, `map-runtime-snapshot.ts`, `task-activity.ts`

**数据流：** `useDesktopRuntime.messages` → context enrich 附件 → Conversation 渲染

---

### M3 — Composer（输入）

| 组件 | 职责 |
|------|------|
| `composer.tsx` | 主输入、slash、发送/停止 |
| `composer-plus-menu.tsx` | `+` 菜单（附件 / MCP / skill） |
| `composer-queue.tsx` | 排队消息 UI |
| `workspace-directory-picker.tsx` | 任务工作目录 |

**发送流程：**

```text
buildComposerDisplayText + composeComposerMessage
  → Electron: desktopRuntime.sendMessage
  → Web fallback: 本地 conversationMessages
  → 记录 profile-activity · 持久化 attachments
```

**IPC：** `stageComposerAttachment`（附件暂存主进程 userData）

---

### M4 — Tasks & Projects（任务 / 项目）

| 组件 | 职责 |
|------|------|
| `sidebar.tsx` | 最近任务、扩展入口导航 |
| `task-palette.tsx` | 快速切换 / 全局搜索 / 任务内搜索 |
| `welcome-view.tsx` | 无任务空态 |
| `new-project-dialog.tsx` | 创建项目 |
| `task-action-dialogs.tsx` | 重命名 / 归档 |

**Lib：** `task-storage.ts`, `project-storage.ts`, `task-workspace-storage.ts`, `recent-workspace-dirs.ts`

**IPC：** `switchTask`, `bindTaskWorkspace`, `pickProjectFolder`, `revealProjectPath`

**待接入：** `project-sidebar-list.tsx`（已定义但未引用）

---

### M5 — Permissions（权限）

两层权限 UI：

| 层 | 组件 | 说明 |
|----|------|------|
| **模式** | `permission-control.tsx` | ask / auto / all，Composer 内 |
| **运行时** | `permission-prompt.tsx` | Allow/Deny 弹窗，消费 runtime `permissionPrompt` |
| **OS 级** | Settings → System panel | 磁盘/录屏/辅助功能等 |

**IPC：** `respondPermission`, `getPermissionStatuses`, `requestSystemPermission`

---

### M6 — Activity（活动面板）

右侧可选面板：工具调用时间线、context usage、plan 进度。

**组件：** `activity-panel.tsx`, `context-usage-indicator.tsx`  
**Lib：** `activity-turn-model.ts`, `format-token-count.ts`  
**依赖：** `@yuniverse/agent-gui/conversation`

---

### M7 — Skills

**Center 页：** 市场 / 内置 / 已安装三 Tab。

**Lib：** `skill-storage.ts`, `skill-preferences.ts`, `skill-display.ts`, `skill-file.ts`  
**Context 动作：** `installSkill`, `toggleSkill`, `removeSkill`, `refreshCapabilities`  
**IPC：** `readSkillDocument`

---

### M8 — MCP / Connectors

双层模型（与 Main 侧一致）：

| 层 | Renderer 职责 | Main / Bridge 职责 |
|----|---------------|-------------------|
| **配置层** | 市场、安装、编辑 stub | `mcp-registry`, 磁盘配置 |
| **运行时层** | 已安装列表、enable toggle | bridge `toggleMcpServer`, capabilities |

**Hook：** `use-installed-mcp-servers.ts`  
**Lib：** `mcp-storage.ts`, `connector-catalog.ts`, `connector-preferences.ts`  
**IPC：** `listInstalledMcpServers`, `connectMcpServer`, `persistMcpConnectorsToFile`, `toggleMcpServer`

---

### M9 — Expert Suites

专家套件市场与安装，类似 Skills 但独立 catalog。

**Lib：** `expert-suite-catalog.ts`, `expert-suite-storage.ts`  
**Context：** `startExpertSuiteCreationChat`

---

### M10 — Plugins & Theme

**`plugin-provider.tsx`：** catalog、branding、favicon、theme tokens  
**`plugins-panel.tsx`：** Settings 内管理  
**Lib：** `plugin-runtime.ts`  
**IPC：** `listPlugins`, `togglePlugin`, `activateThemePlugin`, `openPluginsDir`

---

### M11 — Hooks

Agent 生命周期钩子可视化编辑（事件 / matcher / hook 树 + JSON 视图）。

**依赖：** `@yuniverse/config/hook-events`, `@yuniverse/config/hooks-display`  
**IPC：** `getHooksConfig`, `saveHooksConfig`, `validateHooksConfig`

---

### M12 — Scheduled Tasks

Cron 定时任务：我的任务 + 运行历史 + 立即执行。

**Lib：** `cron-task-storage.ts`（IPC 代理）, `scheduled-task.ts`  
**IPC：** `listCronTasks`, `saveCronTask`, `deleteCronTask`, `listCronRunHistory`, `runCronTaskNow`

---

### M13 — IM Channels

IM 平台接入（飞书 / 钉钉 / 企微等）：平台列表 + 扫码/手动配置。

**Lib：** `channel-platforms.ts`, `channel-connections-store.ts`  
**关联：** `mobile-remote-dialog.tsx` 可跳转 IM 配置

---

### M14 — Settings

**入口：** `openDrawer === "settings"` → `settings-dialog.tsx`（SettingsPage）

| Section | Panel | 说明 |
|---------|-------|------|
| preferences | — | 语言、主题 |
| account | `profile-panel` | 需 `profileUIEnabled` |
| usage | `usage-stats-panel` | 用量统计 |
| system | — | OS 权限 |
| runtimes | `bundled-runtimes-panel` | Bun/Node/Git |
| shortcuts | `shortcuts-panel` | 快捷键 |
| developerMode | `developer-*-panel` | env / telemetry / sentry |
| plugins | `plugins-panel` | → M10 |
| hooks | `hooks-panel` | → M11 |

---

### Renderer 状态三层

```text
┌─────────────────────────────────────────────────────────┐
│  workbench-context.tsx     UI 编排 · 产品状态 · 业务流    │
│  (composer 草稿 / drawer / 项目任务 / 权限模式 UI)        │
└───────────────────────────┬─────────────────────────────┘
                            │ 调用
┌───────────────────────────▼─────────────────────────────┐
│  use-desktop-runtime.ts    Agent 会话 IPC · 快照订阅     │
│  (messages / skills / connectors / permissionPrompt)    │
└───────────────────────────┬─────────────────────────────┘
                            │ 合并
┌───────────────────────────▼─────────────────────────────┐
│  lib/*-storage.ts          localStorage 持久化偏好       │
│  (task / skill / mcp / permission / project / …)        │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
              window.yuniverseDesktop (preload → main)
```

| 层 | 持有 | 不持有 |
|----|------|--------|
| **workbench-context** | composer 草稿、导航、本地 skills/connectors 副本、快捷键 | Agent 原始事件流 |
| **use-desktop-runtime** | 快照映射后的 session 状态 | UI 布局、drawer |
| **lib/*-storage** | 用户偏好、任务元数据 | 运行时 authoritative 配置（→ main IPC） |

**Electron / Web 双模式：** `isWeb = !window.yuniverseDesktop`；Web 用 localStorage 模拟消息，不订阅 snapshot。

---

### Renderer 模块依赖图

```text
                    ┌─────────────┐
                    │  M1 Shell   │
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌────────────┐  ┌────────────┐  ┌────────────┐
    │ M2 Conv    │  │ M3 Composer│  │ M6 Activity│
    └────────────┘  └──────┬─────┘  └────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         M5 Permission  M4 Tasks    M7–M14 Centers
              │            │       (drawer 全屏页)
              └────────────┴───────────┘
                           │
                    use-desktop-runtime
                           │
                      preload / IPC
```

---

## HOW — Main / Bridge / IPC

### 三进程职责（Main / Preload）

**Main (`src/main/index.ts`)** 启动顺序：

1. 日志 / Sentry / 应用名
2. `ensureDesktopRuntimeEnv()` — 打包运行时、Bun、`CLAUDE_CODE_ROOT`、bundled skills/plugins
3. Bootstrap IPC（Sentry config、Cron 等，不依赖 services）
4. `createMainWindow()` — `contextIsolation: true`, `nodeIntegration: false`
5. `createDesktopServices()` — `DesktopGuiController` + Runtime Bridge
6. 注册完整 IPC + Cron scheduler + Gateway init

**Preload** — `contextBridge.exposeInMainWorld('yuniverseDesktop', …)`，所有渲染层访问 Agent/OS 能力经此白名单。

### IPC 设计

* **通道定义**：`src/ipc/channels.ts`，命名空间 `desktop:*`（60+ channels）
* **类型来源**：大量 re-export 自 `@yuniverse/protocol`
* **通信模式**：
  * `invoke/handle` — 请求-响应（发消息、Settings、MCP CRUD）
  * `webContents.send` — 主→渲染推送（`snapshotUpdated`）
* **Handler 分层**：
  1. Bootstrap（renderer 启动前）
  2. 模块级注册（Cron、附件 staging，支持 dev HMR）
  3. Services 就绪后（Agent、权限、Workspace、MCP、Plugins、Hooks、Settings）
* **快照广播**：`broadcast-snapshot.ts` 缓存 `lastSnapshot`，窗口 `did-finish-load` 时 replay，避免竞态

### Runtime Bridge（策略模式）

`services.ts` 定义 `Bridge` 接口，由 `@yuniverse/config` 的 `resolveBridgeKind()` 选择：

| Bridge | 实现 | 特点 |
|--------|------|------|
| `yuniverse` | `yuniverse-bridge.ts` | GatewayClient、多 Task、session workspace、完整 capabilities |
| `claude-code` | `claude-code-bridge.ts` | ClaudeCodeSessionRuntime、单 session、功能子集 |

两者均：

* 持有 `DesktopGuiController`（`@yuniverse/agent-gui`）
* 将 Agent 事件映射为 GUI 快照，`debounce 50ms` 后 `broadcastSnapshot`
* 处理 permission prompt、context usage、系统通知

**IPC 不直接调 Gateway/Claude Code**，一律经 `services.ts` facade。

### 核心数据流

```text
用户输入 (Composer)
  → window.yuniverseDesktop.sendMessage
  → ipcMain → services.sendDesktopMessage
  → bridge.submitMessage
  → GatewayClient / ClaudeCodeSessionRuntime
  → agent events → DesktopGuiController 更新
  → broadcastSnapshot → renderer onSnapshotUpdated
  → map-runtime-snapshot → Workbench UI
```

### 关键子系统

| 子系统 | 位置 | 说明 |
|--------|------|------|
| **多 Task / Workspace** | `yuniverse-bridge` + `session-workspace.ts` | per-task cwd、sessionId；托管目录 vs 用户绑定目录 |
| **MCP 双层** | bridge（运行时 toggle）+ main `mcp-registry`（安装/配置/探测） | 对齐 [[MCP]] |
| **Plugins & Themes** | `plugin-host.ts` + renderer `plugin-runtime.ts` | `@yuniverse/agent-plugins` 发现 bundled + user |
| **Cron** | 独立 IPC + scheduler | 定时触发 Agent，回调注入 bridge |
| **Bundled Runtimes** | `pack-runtime-env.ts` + `prepare-pack.ts` | Bun、Node 20+、CPython、PortableGit、gateway/session server |
| **可观测性** | Sentry 双端 + electron-log + Langfuse 配置 | 诊断日志导出 IPC |
| **系统能力** | 权限、power-save blocker、原生菜单 | macOS 磁盘/录屏/辅助功能等 |

### packages 依赖方向

```text
apps/desktop
  → @yuniverse/agent-core      # Session runtime、Bun 解析、Claude Code 适配
  → @yuniverse/agent-gui       # DesktopGuiController、terminal view model
  → @yuniverse/gateway          # GatewayClient
  → @yuniverse/protocol         # 快照/流式消息/能力类型
  → @yuniverse/config           # bridge kind、MCP/hooks/cron/settings store
  → @yuniverse/storage          # Session persistence
  → @yuniverse/mcp              # MCP registry
  → @yuniverse/agent-plugins    # Plugin 发现与 theme
  → @yuniverse/telemetry        # Sentry
```

### 构建与打包

* **构建**：Vite 6 + `vite-plugin-electron/simple`（非 electron-vite）
  * Main → `dist/main/index.js`
  * Preload → `dist/preload.js`
  * Renderer → `dist/renderer/`
  * build 时复制 skills/plugins 到 `dist/resources/`
* **打包**：`electron-builder` + `prepare-pack.ts` staging 运行时 → `extraResources`
* **注意**：`pack:*` 必须用 Node ≥20，不能用 Bun 跑 electron-builder

---

## WHY

### 薄 App + 厚 Packages

* 符合 monorepo 迁移策略：`integrations/claude-code` 保持可运行，新能力长进 `packages/*`
* Web/TUI 未来可复用同一 agent-gui / agent-core 边界，desktop 只多 Electron 层

### Bridge 策略模式

* 开发期可切换 Claude Code 兼容路径与 Yuniverse Gateway 路径
* IPC 与 UI 只依赖 `Bridge` 接口 + 快照，不绑定单一运行时

### 快照推送 vs 渲染层轮询

* Agent 事件频率高，push snapshot 减少 IPC 往返
* Renderer 无 Node，状态只读快照，降低安全面

### Preload 白名单

* `contextIsolation` + 单一 `yuniverseDesktop` API，对齐 Electron 安全最佳实践
* 对比 Codex Desktop 多进程 Worker 模型（见 [[OpenAI Codex Desktop 架构分析]]），我们选择 **Main 内 Bridge + 子进程运行时**，复杂度更低

### Bundled Runtimes

* 用户机器未必有 Bun/Python/Git；打包自包含 Agent 执行环境
* Settings 可开关 bundled runtimes，灵活 PATH

---

## TRADE-OFF

### 优势

* 清晰 IPC 边界，packages 可单测、可被 Web/TUI 复用
* Bridge 可切换，迁移风险可控
* 快照模式 UI 与 Agent 解耦，Workbench 迭代不碰 runtime
* 完整打包链路（Linux/macOS/Windows）

### 局限

* Main 进程职责重（IPC + Bridge + MCP + Cron + OS），`ipc.ts` / `services.ts` 持续膨胀
* 双 Bridge 维护成本；claude-code bridge 功能子集需文档化差异
* Renderer localStorage 与 main `@yuniverse/config` 文件 store 分工，状态同步边界需小心
* **`workbench-context.tsx` 体量过大（~1570 行）**，14 个功能模块的状态编排集中在一处，模块边界在代码层尚未物理拆分
* 无 URL 路由，`openDrawer` 状态机扩展新 Center 页需改 context + workbench 布局
* 无独立 Worker Process（与 Codex Desktop 对比），重 IO/子进程任务仍在 Main 编排

### 替代方案（未采用或部分采用）

| 方案 | 未完全采用原因 |
|------|----------------|
| electron-vite 一体化 | 已选 Vite + vite-plugin-electron，renderer 独立 config 更灵活 |
| Renderer 直接连 Gateway | 违反 IPC 边界；且无法做 OS 权限与 MCP 安装 |
| 每 Task 独立 BrowserWindow | 当前单窗口 Workbench + 多 Task 状态机 |

---

## REFLECTION

### 与知识库中其他系统对照

| 维度 | Yuniverse Desktop | Claude Code | Codex Desktop（Source） |
|------|-------------------|-------------|------------------------|
| 入口 | Electron GUI | CLI / Desktop / IDE | Electron GUI |
| Harness | Bridge + Controller + Snapshot | 五层路由 + QueryEngine | Main + Worker + MCP |
| 工具层 | Gateway capabilities + MCP | 40+ 内置 + MCP | MCP Plugin 生态 |
| 可分析性 | 自有源码 | 曾有泄露可参考 | 逆向分析 |

可复用的架构模式：[[Layered Agent Routing Pattern]]、[[Progressive Tool Disclosure]]（Gateway capabilities 按需暴露）。

### 演进方向（待验证）

1. **Main 瘦身**：Cron、MCP 配置等是否进一步模块化或下沉 packages
2. **Bridge 统一**：claude-code bridge 长期是兼容层还是移除
3. **Multi-Agent UI**：Coordinator / Subagent 在 Workbench 的产品化（见 [[Multi-Agent]]）
4. **Computer Use**：若做 BrowserView 自动化，与现有 MCP/终端工具边界（见 [[Computer Use]]）

### 文档与代码同步

* 工程 README：`apps/desktop/README.md`（构建/打包/日志）
* OpenSpec：`openspec/specs/desktop-app/spec.md`（IPC 边界）
* 本笔记：架构决策与 trade-off；细节以代码为准

---

## Related Concepts

[[AI Agent]]
[[MCP]]
[[Tool Calling]]
[[Agent Routing]]
[[Layered Agent Routing Pattern]]
[[Multi-Agent]]
[[Plan Mode]]

## Related Systems

[[Claude Code Architecture]]
[[Cursor]]

## Related Patterns

[[Layered Agent Routing Pattern]]

## Sources

[[OpenAI Codex Desktop 架构分析]]（对照参考，非本项目实现）

## Code References

### Renderer（按模块）

| 模块 | 路径 |
|------|------|
| M1 Shell | `src/renderer/components/workbench/workbench.tsx`, `workbench-context.tsx` |
| M2 Conversation | `src/renderer/components/workbench/conversation.tsx`, `message-content.tsx` |
| M3 Composer | `src/renderer/components/workbench/composer.tsx` |
| M4 Tasks | `src/renderer/components/workbench/sidebar.tsx`, `task-palette.tsx` |
| M5 Permissions | `src/renderer/components/workbench/permission-*.tsx` |
| M6 Activity | `src/renderer/components/workbench/activity-panel.tsx` |
| M7 Skills | `src/renderer/components/workbench/skills-center.tsx` |
| M8 MCP | `src/renderer/components/workbench/connectors-center.tsx` |
| M9 Expert Suites | `src/renderer/components/workbench/expert-suites-center.tsx` |
| M10 Plugins | `src/renderer/components/plugin/plugin-provider.tsx` |
| M11 Hooks | `src/renderer/components/workbench/hooks-panel.tsx` |
| M12 Cron | `src/renderer/components/workbench/scheduled-tasks-center.tsx` |
| M13 IM | `src/renderer/components/workbench/im-channels-center.tsx` |
| M14 Settings | `src/renderer/components/workbench/settings-dialog.tsx` |
| Runtime 桥接 | `src/renderer/hooks/use-desktop-runtime.ts` |
| 快照映射 | `src/renderer/lib/map-runtime-snapshot.ts` |

### Main / 平台

| 路径 | 说明 |
|------|------|
| `apps/desktop/src/main/index.ts` | 主进程入口 |
| `apps/desktop/src/main/services.ts` | Bridge 工厂与 facade |
| `apps/desktop/src/main/yuniverse-bridge.ts` | Gateway 路径 |
| `apps/desktop/src/main/claude-code-bridge.ts` | Claude Code 路径 |
| `apps/desktop/src/ipc/channels.ts` | IPC 契约 |
| `apps/desktop/src/preload.ts` | 渲染层 API |
| `openspec/specs/desktop-app/spec.md` | 平台约束 |
