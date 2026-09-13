---
title: "botmux 深度分析"
type: product
product: "botmux"
category: "multi-agent"
slug: "botmux"
status: in-review
as_of: "2026-09-13"
created: "2026-09-13"
updated: "2026-09-13"
first_hand: true
tags: [hot, verified]
---

# botmux 深度分析

## TL;DR

- botmux 是飞书/Lark 到本地 AI 编程 CLI 的自托管桥接控制层。
- 核心取舍是保真：复用完整 CLI 进程、记忆、MCP 与 hooks，而非 SDK 子集。
- 默认 tmux 后端让 CLI 进程常驻，daemon 重启后可重新接管上下文。
- 多 bot 同群显式 @ 路由可实现并行执行、互审和主控 Agent 验收。
- 安全边界取决于部署：有权限分层和写隔离，但默认并非全隔离沙箱。

## 1. 产品概览

- **公司 / 团队**：独立开源项目，GitHub owner 为 `deepcoldy`，npm maintainer 为 `deepcold`；未发现官方声明其代表某个公司运营。截至 2026-09-13，项目采用 MIT License。[11][12]
- **上线时间与重要节点**：
  - npm registry 记录 `botmux` 包创建于 2026-03-11T17:05:48Z；完整 Git 历史的初始提交 author date 为 2026-03-11 03:08+08，初始定位是 “Lark ↔ Claude Code bridge with live streaming cards”。[12][13]
  - 2026-09-09，项目发布 `v3.21.0`；本机 CLI 与 npm latest 均为 3.21.0。tag 说明新增普通群项目协作、CLI 限额后备用 bot 交接、按任务配置模型、MiniMax CLI 适配器等。[12][13]
  - 截至 2026-09-13，完整 Git 历史约 3,347 commits，npm registry 收录 430 个版本，说明项目仍处于高频迭代期。[12][13]
- **一句话定位**：把飞书/Lark 话题变成 AI 编程 CLI 的遥控器；每个话题映射一个独立 CLI 会话，实时输出为飞书流式卡片，并提供可交互 Web 终端。[1]
- **目标用户与核心场景**：把 Claude Code、Codex、OpenCode、Cursor 等长任务 CLI 放在开发机或服务器上运行、需要在手机/飞书中查看进度并介入的开发者；同群部署多个 bot 做代码 review、方案互怼和分工编排的团队；把机器人放进 oncall 群做项目问答与值班处理的团队。[1][6][10]

### 分类判断

本报告将 botmux 归入 `multi-agent`：它的核心价值之一是同群多 bot 的显式 @ 路由、并行会话、handoff/orchestration 与主控验收；同时它不是新的 Agent SDK，而是对已存在 CLI 进程的控制与编排层，因此也带有明显的 `agent-infra` 属性。[1][6]

## 2. 核心能力

| 能力 | 说明 | 截至日期 | 来源 |
|---|---|---|---|
| IM ↔ CLI 会话桥接 | daemon 监听 Lark 事件，新话题 spawn worker，worker 通过适配器拉起 CLI 并挂在 PTY/tmux 后端；输出渲染为流式卡片 | 2026-09-13 | [1][2] |
| 复用完整 CLI 运行时 | 官方定位是“直接桥接 CLI 进程”，复用 CLI 的记忆、上下文、工具调用、权限、plan mode、`/` 命令与 MCP；接口变化时 adapter 仍可能跟进 | 2026-09-13 | [1][11] |
| 多 CLI 适配器 | README 宣称 20+ CLI / Agent；v3.21.0 源码 registry 枚举 31 个 `cliId`（含本地进程、API/云 Agent 与变体），如 Claude Code、Codex、Cursor、Gemini、OpenCode、Copilot、Kimi、Grok、MiniMax 等 | 2026-09-13 | [4][11] |
| tmux 常驻与接管 | 默认后端为 tmux ≥ 3.x；daemon/worker 重启后 CLI 进程仍存活并 re-attach。tmux 不可用时不再静默降级，而是硬拦截；可显式选择 pty 等应急后端 | 2026-09-13 | [5] |
| 可交互 Web 终端 | 每个会话有 xterm.js Web 终端；卡片自动提供只读链接，可写链接需单独获取并带 token。飞书、Web 终端、本地 tmux 看到同一进程 | 2026-09-13 | [7] |
| 多 bot 协作 | 同一台机器可运行多个 bot，每个 bot 可绑定不同 CLI/model；多人群需显式 @ 指定接收方。botmux 默认发现群内机器人，并提供 `bots list`、handoff/orchestration 技能 | 2026-09-13 | [3][6] |
| 会话与权限模型 | 话题群中新话题对应独立 CLI 会话；权限分为 canTalk（对话/读）、canOperate（切目录/重启/关会话/按钮）、owner（授权管理）三层 | 2026-09-13 | [3] |
| 文件沙盒 | Linux 下基于 bubblewrap + overlayfs 做写隔离：真实文件不被直接修改，owner 审阅 diff 后 `/land` 落盘或丢弃；默认不隔离读，敏感路径需 `sandboxHidePaths`，网络默认开放 | 2026-09-13 | [8] |
| Oncall / 定时 / workflow | 支持把群绑定项目目录、配置周期任务、运行实验性 workflow 和多话题协作；复杂 workflow 仍标注为实验性 | 2026-09-13 | [9][10] |

## 3. 使用体验（实测）

- **测试环境与日期**：2026-09-13，Linux 主机，botmux CLI `3.21.0`；同一飞书群中部署 OpenCode、ClaudeCode、Codex 三个 bot，均绑定同一仓库工作区。测试只读命令与消息路由，未修改 botmux 配置。[16]
- **上手门槛**：对已熟悉飞书机器人、tmux 和 AI 编程 CLI 的开发者较低；官方提供扫码 setup、自动权限配置和 daemon/dashboard 启动。对非飞书组织、无常驻主机、不熟悉权限边界的用户有明显运维与治理门槛。[1][9]
- **实测亮点**：
  - `botmux status` 显示 supervisor 在线，三个 bot daemon 与 dashboard 均在线；一个 bot 一个 daemon 的生产形态在本机可观察。[2][16]
  - 对 ClaudeCode 与 OpenCode 发出带显式 @ 的分工交接后，两个 bot 分别进入独立工作卡片；`botmux list --plain` 可看到当前任务为每个 bot 生成独立 online tmux session，工作目录相同但进程隔离。[6][16]
  - `botmux history` 能读取群历史并还原用户请求、Codex 交接、子 bot 工作状态，便于主控 Agent 做事后审查。[16]
  - 内置 `botmux-send`、`botmux-history`、`botmux-bots`、`botmux-handoff`、`botmux-orchestrate`、`botmux-ask` 等技能，说明协作协议是产品化能力而非简单消息转发。[16]
  - `botmux bots list` 不只返回名字，还给出 mentionable、reachability、authorization、runtime deployment 等语义，并提醒 “unknown 不等于不可用”，能减少多 Agent 协作误判。[16]
  - @ 决策有护栏且与轮次上下文相关：原始用户轮同时提及 3 个 bot 时，`--mention-back` 被拒绝并要求显式点名；bot→bot 单提及探针轮则可自动 @ 回触发者。报告不将其泛化为所有多人群必拒。[16]
- **实测槽点**：
  - 会话列表、卡片和 Dashboard 汇总了大量任务标题、路径、上下文用量和输出，默认便利性与信息暴露风险并存；正式使用需收紧 `allowedUsers`、`sandboxHidePaths`、Dashboard 暴露面与卡片输出策略。[3][7][8]
  - 多 Agent 同时观察群消息时可能出现重复唤醒或等待输入；本任务初期 ClaudeCode 与 OpenCode 均被用户消息触发，其中一方自行判定无需回复。显式 @ 和 handoff 能缓解，但编排者仍需主动分工。[16]
  - 概念面较大：session scope、mention mode、canTalk/canOperate、sandbox、adopt/relay、workflow、oncall、plugin 等叠加后，新用户需要学习成本。[3][8][9][10]

## 4. 技术架构

```mermaid
flowchart LR
    U["飞书 / Lark 用户"] -->|消息、@mention、卡片按钮| E["Lark 长连接事件"]
    E --> D["daemon（每个 bot 一个）"]
    D -->|新话题/会话 spawn| W["worker"]
    D -->|状态、权限、会话路由| W
    W --> A["CLI adapter"]
    A --> C["AI 编程 CLI 进程<br/>Claude Code / Codex / OpenCode / ..."]
    C --> B["tmux / PTY 后端"]
    W -->|终端渲染 / Markdown| K["飞书流式卡片"]
    W -->|xterm.js / WebSocket| T["Web 终端"]
    C -->|botmux send / skill| K
```

- **底层模型**：botmux 不内置模型，也不重写 Agent 推理层；模型、订阅、认证和大部分工具能力由被桥接的 CLI 提供。`model` 字段只对支持模型参数的适配器生效。[1][4]
- **工具与协议**：复用 CLI 原生 function calling、MCP、hooks、skills、plan mode 与 slash commands；官方同时通过结构化 prompt 注入隔离用户内容和系统指令。Lark 侧使用飞书事件、消息 API 与 interactive card。[1][2]
- **记忆与上下文管理**：主要复用 CLI 内建记忆和上下文；tmux 后端保持活进程，daemon 重启后 re-attach 而非冷启动 resume。官方明确当前不能跨 CLI 无损热切，切换需新会话并做 handoff 摘要。[3][5]
- **部署形态**：自托管 daemon/worker，推荐常开开发机、DevBox 或服务器；安装器提供自包含二进制，botmux 本体运行不需要 Node，npm 安装路径需要 Node ≥ 22，目标 CLI 的依赖另计。支持 Linux/macOS，Windows 需 WSL2；默认 tmux ≥ 3.x。[9][11]

### 关键设计判断

botmux 的差异化不在“更聪明的 Agent”，而在 **交互通道与进程生命周期管理**：它把 IM 的事件模型、终端进程、流式卡片、权限审批和多 bot 路由拼接成控制面。官方将其与 SDK wrapper 方案对比，宣称完整 CLI 进程能继承更多原生能力、CLI 升级通常直接受益；这是官方立场，接口和 resume 语义变化仍可能要求 adapter 跟进。[1][11]

第三方源码精读也将其概括为“把 AI 编程 CLI 投影到飞书的会话编排器”，与官方定位相互印证；该文章基于较早版本，本报告只用它做定位交叉验证，不采用其实现细节覆盖当前版行为。[14]

## 5. 定价与商业模式

| 方案 | 价格 | 包含内容 | 截至日期 |
|---|---|---|---|
| 开源自托管 | 未发现官方收费；MIT License | botmux daemon、Dashboard、Web 终端、多 bot 路由、CLI adapters、tmux 会话管理等软件能力 | 2026-09-13 |
| 使用成本 | 用户自担 | 飞书应用/租户权限、常驻主机、目标 CLI 订阅或模型费用、tmux 与网络等基础设施 | 2026-09-13 |

截至 2026-09-13，官方 README、文档与 npm 页面未展示 SaaS 订阅、企业版或托管版定价；以上“未发现”不代表不存在未来商业化计划。[1][11][12]

## 6. 生态与集成

- **IM**：深度绑定飞书/Lark，支持话题群、普通群、私聊、卡片、@mention、附件与语音相关能力；非飞书组织不是目标市场。[1][3]
- **AI CLI / Agent**：官方 README 宣称 20+ 适配器，当前源码 registry 枚举 31 个 `cliId`；既包括本地进程，也包括 Mira、riff 等 API/云 Agent 接入方式。[4][11]
- **终端与生命周期**：默认 tmux，支持 session list、resume/suspend/delete、adopt 本地 tmux 会话、relay 跨群接力；adopt 与 relay 各自边界明确。[5]
- **治理面**：Dashboard 统一查看 daemon、bot、会话与排程；Web 终端读写分权；canTalk/canOperate/owner 分层；Linux 文件沙盒提供写隔离和 diff 落盘审批。[3][7][8]
- **自动化**：Webhook/API 任务触发、schedule、实验性 workflow、多话题协作和插件/hooks 机制，使它可从“遥控器”扩展为轻量 Agent 工作台。[9][10][11]

## 7. 优势与局限

### 优势

1. **移动优先的 Agent 控制面**：长任务不需要用户守在终端；飞书卡片、状态、上下文用量、输出、停止/关闭/终端入口集中呈现。[1][7]
2. **保真桥接而非重造 SDK**：复用 CLI 的记忆、MCP、hooks、权限与命令生态，避免把上游能力锁定在 botmux 自己的抽象子集里。[1][11]
3. **进程隔离与常驻**：一个 bot 一个 daemon、一个话题一个 CLI 会话、默认 tmux 常驻，使多任务和 daemon 恢复的边界清晰。[2][5]
4. **多 Agent 协作产品化**：群内 bot 发现、显式 @ 路由、handoff/orchestrate 技能和主控验收协议，使不同 CLI/model 可以并行互审。[6][16]
5. **治理细节较完整**：对话权/操作权分层、Web 终端 token、Dashboard 轮换登录、Linux 写隔离沙盒、敏感路径遮罩与网络开关，覆盖了自托管常见风险面。[3][7][8]
6. **开源且迭代活跃**：MIT、完整源码、官方文档、npm 与 GitHub 发布流水线齐备；第三方用户讨论也确认其解决了远程控制 Code Agent 的真实痛点。[11][12][15]

### 局限与风险

1. **强绑定飞书/Lark**：核心交互、卡片、群、机器人、权限和协作协议均围绕飞书；组织不在飞书生态内基本不适用。[1][3]
2. **默认不是防一切沙箱**：Linux sandbox 的定位是防误改和改动可审，不隔离默认读，网络默认开放；`bots.json`、SSH 凭证、其他项目等敏感路径需显式遮罩。CLI 与 Dashboard 在安装用户权限边界内运行，越权配置会放大风险。[8]
3. **运维面不小**：需要常驻主机、tmux、目标 CLI 登录态、飞书应用权限、端口/远程访问策略、版本升级和多 bot 配置；对个人开发者可接受，对企业共享部署需要专门治理。[8][9][11]
4. **高频迭代与适配漂移**：6 个月 430 个 npm 版本、主版本从 1 到 3，说明能力扩张快，也意味着升级回归、配置迁移和 adapter 兼容需要管理。[12][13]
5. **跨 CLI 不等于同一上下文**：官方明确无法跨 CLI 无损热切；多 Agent 协作仍依赖结构化 handoff，不能假设 Claude Code 与 Codex 共享同一原生记忆。[3]
6. **信息暴露与提示注入面**：群聊文本、代码输出、卡片标题、路径和终端截图会跨系统传输；多 Agent 群中一条恶意消息可能被多个 bot 解释，必须配合权限、沙盒、输出审查和最小工作区。[3][7][8]

## 8. 竞品速览

> 下表是定位对比，不是功能完整度评分。Paseo 结论参考本仓库既有 Paseo 报告与官网 [17]；tmux+SSH 与自建 Lark SDK 属于通用替代路径。

| 维度 | botmux | Paseo | tmux + SSH/mosh | 自建 Lark Bot / Agent SDK |
|---|---|---|---|---|
| 核心路线 | 桥接完整本地 CLI 进程到飞书 | 自托管 coding agent control plane，多端客户端 | 直接远程进入终端 | 从消息卡片到 Agent 能力全部自建 |
| 交互 | 飞书原生卡片、@ 路由、Web 终端、Dashboard | 桌面/Web/移动客户端监控与介入 | 终端 UI，移动体验取决于客户端 | 完全自定义 |
| 多 Agent | 同群多 bot、独立 daemon/session、显式 @、handoff/orchestration | 多 Provider 与多会话编排，偏个人/开发者控制面 | 手动切 window/session | 需自研路由、隔离与协作协议 |
| 生命周期 | tmux 常驻、adopt/relay、resume/suspend | daemon 与多端远程管控 | tmux 本身常驻，连接层自管 | 自建 |
| 安全治理 | canTalk/canOperate、Web token、Linux 写隔离沙盒 | 自托管与权限模型（详见 Paseo 报告） | SSH 密钥/网络边界 | 全部自建 |
| 适合边界 | 飞书生态内希望保留原生 CLI 的技术团队 | 需要跨设备/多 Provider coding agent 控制台 | 已有成熟远程开发流程 | 有强定制和集成需求 |

### 与 OpenClaw / SDK wrapper 的关系

官方 README 将 botmux 与“基于 Agent SDK 重新构建”的方案对比，主张完整 CLI 进程能继承 hooks、memory、plan mode、MCP 和 slash commands。该对比是产品立场，不能替代逐项实测；本报告仅采纳其中与架构直接相关的“桥接对象不同”这一点，不采纳“其他方案必然缺失能力”的泛化结论。[1][11]

## 9. 适用场景推荐

### 适合

- 飞书/Lark 已是主要协作工具，且团队接受自托管开发机或服务器。
- 已经订阅或配置多个 AI 编程 CLI，希望在手机上监控、追问、打断和审批。
- 需要 Claude Code、Codex、OpenCode 等不同模型对同一代码/方案并行 review。
- oncall 群需要把机器人绑定项目目录，让多人即问即答，同时限制敏感操作。
- 个人开发者想把本地 tmux 中的长任务接到移动端，并保留本地终端操作习惯。

### 不适合 / 需谨慎

- 不使用飞书/Lark 的组织。
- 只想用单一官方 Agent、没有远程监控需求的轻量用户。
- 不能维护常驻主机、CLI 登录态、tmux、权限和网络暴露策略的团队。
- 强合规环境中禁止代码路径、终端输出、上下文用量或任务元数据进入飞书卡片的场景。
- 期待完整企业多租户 SaaS、集中密钥管理、审计合规和服务级别承诺的团队；当前公开材料未证明这些能力。

## 10. 后续验证清单

1. 实测 Web 终端可写链接在移动端的 Esc、Ctrl+C、方向键与权限确认流程。
2. 验证 daemon 重启前后同一 tmux CLI 进程、上下文与卡片状态的一致性。
3. 对 Linux sandbox 做读遮罩、网络关闭、build 产物、`/land` patch 与冲突场景测试。
4. 用多条恶意/歧义群消息测试多 bot 是否重复执行、越权读取或被提示注入。
5. 压测多 bot 并行时的 token、上下文、卡片更新频率与主机资源占用。
6. 核验企业多租户、密钥管理、审计日志与升级迁移能力。
7. 深入比较 botmux、Paseo、OpenClaw、AgentCLI Bridge 在同一任务集上的完成率与总拥有成本。

## 信息来源

| # | 来源 URL | 类型（官方/第三方/实测） | 访问日期 |
|---|---|---|---|
| 1 | https://deepcoldy.github.io/botmux/ | 官方文档首页 | 2026-09-13 |
| 2 | https://deepcoldy.github.io/botmux/architecture | 官方架构文档 | 2026-09-13 |
| 3 | https://deepcoldy.github.io/botmux/session-model | 官方会话/权限文档 | 2026-09-13 |
| 4 | https://deepcoldy.github.io/botmux/adapters | 官方适配器文档 | 2026-09-13 |
| 5 | https://deepcoldy.github.io/botmux/tmux | 官方 tmux 文档 | 2026-09-13 |
| 6 | https://deepcoldy.github.io/botmux/multi-bot | 官方多 bot 协作文档 | 2026-09-13 |
| 7 | https://deepcoldy.github.io/botmux/web-terminal | 官方 Web 终端文档 | 2026-09-13 |
| 8 | https://deepcoldy.github.io/botmux/sandbox | 官方文件沙盒文档 | 2026-09-13 |
| 9 | https://deepcoldy.github.io/botmux/quickstart | 官方快速接入/前置要求 | 2026-09-13 |
| 10 | https://deepcoldy.github.io/botmux/workflow | 官方 workflow 文档 | 2026-09-13 |
| 11 | https://github.com/deepcoldy/botmux | 官方 GitHub README / 源码 | 2026-09-13 |
| 12 | https://www.npmjs.com/package/botmux | npm 包页 / registry 元数据 | 2026-09-13 |
| 13 | https://github.com/deepcoldy/botmux/commits/master/ | 官方 Git 历史 / v3.21.0 tag | 2026-09-13 |
| 14 | https://inferloop.dev/source-reading/botmux/ | 第三方源码精读（时点见原文） | 2026-09-13 |
| 15 | https://x.com/rccoders/status/2062575520877818128 | 第三方用户实测评价 | 2026-09-13 |
| 16 | local://botmux/3.21.0/lark-multi-bot/2026-09-13 | 一手实测：CLI status/list/history/bots/skill 与本群多 bot 会话 | 2026-09-13 |
| 17 | https://paseo.sh/ | 竞品官方站点（Paseo 对照） | 2026-09-13 |

## 更新记录

| 日期 | 变更 |
|---|---|
| 2026-09-13 | 初版：基于官方文档、源码/npm 元数据、第三方来源与 v3.21.0 三 bot 飞书群实测整理；ClaudeCode 与 OpenCode 分工调研，Codex 完成事实纠偏、结构优化与最终审校。 |
