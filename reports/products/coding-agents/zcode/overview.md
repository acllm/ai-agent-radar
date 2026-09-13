---
title: "ZCode 深度分析"
type: product
product: "ZCode"
category: "coding-agents"
slug: "zcode"
status: draft
as_of: "2026-09-13"
created: "2026-09-13"
updated: "2026-09-13"
first_hand: false
tags: [hot, major-release, pricing-change]
---

# ZCode 深度分析

## TL;DR

- ZCode 是智谱为 GLM-5.3 打造的官方「harness」：Agentic 开发环境（ADE），桌面端形态。
- 核心主张是长任务连续性：GLM-5.3 的 1M 上下文 + Goal 目标模式自动迭代到完成。
- 差异化在多端控制：手机 Remote 与微信 / 飞书 Bot Channel 可跟进同一桌面任务。
- 商业模式是「工具免费 + GLM Coding Plan 订阅」，套餐可在 ZCode、Claude Code 等 20+ 工具通用。
- 两个月内套餐折扣价与倍率多次调整；GLM-5.3 时代的第三方深度实测仍稀缺。

## 1. 产品概览

- **公司 / 团队**：智谱（Z.ai / BigModel）。官网托管于 z.ai 子域，账号体系分国内 BigModel 与海外 Z.ai，第三方来源均将其归属智谱 [1][3][10][11][12]。
- **上线时间与重要节点**：
  - 确切首发日期：待核实；不晚于 2025-12-26（ai-bot.cn 收录时间与其评论区时间线）[11]。
  - 2025-12 前后：GLM-5.2 时期，第三方导航站将其描述为「轻量级 AI IDE」[11]。
  - 2026-07-05：第三方实测版本为 v3.2.2，深度适配 GLM-5.2 [12]。
  - 2026-08-14：v3.7.7 上线旗舰模型 GLM-5.3 [2]。
  - 2026-08-26：v3.9.2 增加 GLM-5.3-Flash 多模态，改进 Computer Use [2]。
  - 2026-09-04：v3.11.2 发布；截至 2026-09-13 为当前版本 [1][2]。
- **一句话定位**：「Official Harness for GLM-5.3」——智谱官方 Agentic Development Environment（ADE），把 GLM-5.3 的长上下文与长程任务能力转化为稳定的桌面开发工作流 [1][3]。
- **目标用户与核心场景**：需要「一句话目标、多轮执行」的开发者与团队：重构模块并保持测试通过、清理编译错误、补测试、代码评审、发布准备 [10][12]；以及希望模型、账号、执行全链路国产化的团队 [12]（该动机为第三方作者观点）。

### 分类判断

本报告将其放入 `coding-agents`：产品主线是规划→编码→验证→评审的开发全流程 [3][10]。需要说明两点：其一，其能力已明显外溢（Computer Use、内置浏览器自动化、文档 / PPT / PDF 与媒体处理 [2][3]），与 Finch 类似，若未来 taxonomy 拆出「桌面个人 Agent / ADE」分类应迁移；其二，它是模型厂商官方 harness，生态位与 Claude Code（Anthropic）、Codex（OpenAI）同构，而非中立的第三方壳。

## 2. 核心能力

| 能力 | 说明 | 截至日期 | 来源 |
|---|---|---|---|
| 桌面 ADE | macOS（Apple Silicon / Intel）、Windows（x64 / ARM64）、Linux Beta（x64 / ARM64：deb / rpm / AppImage）；Electron 安装包 | 2026-09-13 | [1] |
| ZCode Agent | 自研主智能体，深度适配 GLM-5.3；统一任务、上下文、权限、文件引用与 Review | 2026-09-13 | [1][3] |
| 长上下文 | 基于 GLM-5.3 的稳定 1M 上下文，任务内持续保留目标、文件、终端输出、浏览器上下文、执行模式与 Git 状态 | 2026-09-13 | [3] |
| Goal 目标模式 | `/goal` 设定可校验目标，自动拆解、迭代、每轮校验，完成后收尾；支持 pause / resume / replace / clear | 2026-09-13（命令细节实测于 2026-07-05） | [3][12] |
| 执行模式 | 四档：Ask before changes（默认）/ Edit automatically / Plan / Full access；`Shift + Tab` 切换 | 2026-09-13 | [9][12] |
| 安全确认 | 敏感命令、文件修改、网络请求、脚本执行执行前强制确认；允许 / 始终允许 / 拒绝 / 始终拒绝（部分场景有会话 / 项目粒度）；普通问题默认 5 分钟无响应自动继续（可关闭），权限请求与计划审批始终等待 | 2026-09-13 | [9][12] |
| 浏览器自动化 | 内置浏览器点击、填表、截图，自动验证前端改动；视频录制为 v3.8.1（2026-08-20）新增；v3.11.2 改为任务间窗口独立并记忆窗口尺寸 | 2026-09-13 | [2][11] |
| Computer Use | 搭配 GLM-5.3-Flash 的计算机操作；权限前置提示；支持 macOS Intel 机型 | 2026-08-26 起 | [2] |
| 多模态 | GLM-5.3-Flash 内置（截图理解、图像分析）；v3.11.2 会话内 PDF 上传 / 预览与图片、视频预览 | 2026-09-13 | [1][2] |
| 记忆 | 自动提取项目偏好与团队约定，后续会话自动带入；设置内开关 | 2026-09-13 | [3][11] |
| 仓库 Wiki | 基于仓库生成带源码位置的架构导读 | 2026-09-13 | [11][12] |
| 自动化 | 定时任务；闲时任务在算力富余时段免费执行、不耗套餐额度（逐步推出中） | 2026-09-13 | [3] |
| 远程开发 | SSH、WSL、Docker 远程工作区，Agent 在目标环境执行命令与代码 | 2026-09-13 | [4][11][12] |
| Remote Control | 手机扫码临时接入当前 ZCode 桌面窗口，可浏览并切换该窗口内已打开的 workspaces / tasks / sessions 并持续下指令；手机只是控制面，不自建 runtime、不能新建 SSH / WSL / Docker 连接；同一时间仅一个手机页面可连接；连接链接本身即授权，需手动 Stop 才断开 | 2026-09-13 | [4] |
| Bot Channel | 官方 Bot Channel 文档当前列出 WeChat、Feishu（飞书回复为流式卡片）；DingTalk / Discord / WeCom 为文档标注的后续版本；Telegram 仅出现在官网主页营销区，Bot Channel 文档未见 | 2026-09-13 | [1][5] |
| 子智能体 | 内置 general-purpose（全工具）与只读 Explore；自定义用户级子智能体（Beta，`~/.zcode/agents/<name>.md`，可配模型 / 思考强度 / 工具白名单）；前台并行与后台执行均支持（后台 Explore 仅只读工具）；v3.7.1 起子智能体默认注入 AGENTS.md（Explore 除外） | 2026-09-13 | [8] |
| 扩展体系 | Plugin、Skill、Command、MCP、Hooks；插件可按工作区安装并提示更新 | 2026-09-13 | [2][3] |
| 项目指令 | `AGENTS.md`（`~/.zcode/AGENTS.md` 全局 + 工作区文件拼接）；`CLAUDE.md` 仅 onboarding 一次性迁移 | 2026-07-05 | [12] |
| 模型接入 | BigModel / Z.ai 账号授权；BYOK：Anthropic、OpenAI、OpenRouter、Moonshot、MiniMax、小米 MiMo 及兼容协议自定义端点 | 2026-07-05 | [11][12] |
| Skills 导入 | 官方口径来源：Claude Code、Codex CLI、OpenClaw、Augment、Windsurf；支持 symlink / copy 两种模式，可导入全局或当前项目 | 2026-09-13 | [6] |
| MCP 导入 | 官方口径来源：Claude Code（`~/.claude/settings.json`）、Codex CLI（`~/.codex/config.toml`）、OpenCode（`~/.config/opencode/opencode.json`）、通用 `.agents`（`~/.agents/mcp.json`） | 2026-09-13 | [7] |
| 输入快捷符 | `@` 引用文件、`#` 关联历史、`/` 命令、`$` 技能、`+` 附件；长文本自动转附件 | 2026-07-05 | [11][12] |

> 口径冲突说明（Remote Control 范围）：cnblogs 第三方实录（2026-07-05，v3.2.2）称手机端「只能访问当前打开的工作区」[12]；官方 Remote Control 文档（2026-09-13 访问）称手机连接的是整个桌面窗口、可切换其中已打开的 workspaces / tasks / sessions，但不能触达窗口外目录 [4]。两者冲突，本报告以时间更近的官方文档为准，旧口径作为 v3.2.x 时点记录保留。

## 3. 使用体验（外部证据）

> 本报告未做一手实测（`first_hand: false`）。以下为第三方体验证据，样本少且多为 GLM-5.2 时期（v3.2.x）版本，不能当作代表性结论。

- **正面（cnblogs「拓荒者IT」，2026-07-05，v3.2.2）**：「把整套开发流程塞进一个桌面应用」，长任务不需要反复人工催促，上下文保持稳 [12]。同一作者也指出代价：需要花时间配置 AGENTS.md、技能、MCP 与模型接入；偶尔写几行代码的场景「大材小用」[12]。
- **负面 / 早期问题（ai-bot.cn 评论区，约 2025-12）**：有用户反馈模型选择器异常（选模型时变为 auto、无法选择）[11]。单条早期评论；后续 changelog 已多次修复模型配置类问题 [2]。
- **教学侧观察（runoob，2026-06 前后）**：面向初学者的安装-配置-任务全流程教程已成型；并提示「官方免费额度很有限，基本不够用」[10]。
- **官网演示口径**：首页展示跨项目任务看板与一个五子棋目标的完成过程（5/5 步、约 2 分钟、8.9 万 token）[1]。属官方营销演示，仅作能力示意。

## 4. 技术架构

- **底层模型**：GLM-5.3（旗舰，2026-08-14 随 v3.7.7 上线）+ GLM-5.3-Flash（多模态，2026-08-26 随 v3.9.2 上线）[2]；另有 GLM-5-turbo 等内置模型，并支持 BYOK 第三方模型 [3][12]。「深度适配 / 深度调优」的官方口径仅针对 GLM 系列 [3]。
- **工具与协议**：MCP（stdio / HTTP / SSE / 粘贴 JSON 配置；支持需要 OAuth 的远程服务如 Figma 官方 MCP；工作区级优先于用户级；`.zcode` 配置存在时同作用域 `.agents` 整体跳过、不合并；面板写入始终回写 `.zcode`）[7]；Plugin / Skill / Command / Hooks 扩展层 [3]；内置浏览器与 Computer Use 作为原生工具 [2][11]。
- **记忆与上下文管理**：1M 长上下文；Memory 提取偏好与约定；`/compact` 压缩当前对话；`AGENTS.md` 承载长期项目指令 [3][12]。Skill 采用「元数据注入 + 按需加载正文」机制：每轮把所有启用技能的名称与描述摘录（≤250 字符）注入上下文，所有技能共享一个固定元数据预算，超限则降级为仅注入名称，自动触发率明显下降 [6]。
- **部署形态**：本地 Electron 桌面应用（安装包经 `cdn-zcode.z.ai` 分发）[1]；计算执行在本地或远程工作区（SSH / WSL / Docker）[4][11][12]；控制面延伸到手机 Remote 与微信 / 飞书 Bot Channel [4][5]。官方文档将这种离桌协作称为「Vibeworking」[3]。
- **迭代节奏**：2026-08-12 至 2026-09-04 间发布 v3.7.6 → v3.11.2 共 8 个版本，约周级发版；修复集中在远程连接、插件加载、模型配置与 UI 稳定性 [2]。

## 5. 定价与商业模式

工具本体免费下载 [1]；收费在模型订阅 GLM Coding Plan（截至 2026-09-13，官方英文站挂牌 [1]）：

| 方案 | 价格 | 包含内容 | 截至日期 |
|---|---|---|---|
| GLM Coding Lite | $12.6 / 月（原价 $18） | 10,000 credits / 周；滚动访问最新旗舰模型与功能；支持 ZCode、Claude Code 等 20+ agent 工具；默认数据隐私 | 2026-09-13 |
| GLM Coding Pro | $56 / 月（原价 $80） | 6× Lite 用量；精选 MCP 工具；更快生成速度 | 2026-09-13 |
| GLM Coding Max | $117.6 / 月（原价 $168） | 14× Lite 用量；最新功能优先；高峰期专属资源 | 2026-09-13 |

- **免费层**：新用户 5 天试用（GLM-5.3 每天 300 万 + GLM-5-turbo 每天 200 万 token，仅 5 天内有效，非长期每日额度）[3]；订阅者闲时任务免费 [3]；Weekend Plan 限免领取（2026-08-28 v3.10.1 起）[2]。
- **价格变动轨迹**（`pricing-change` 标签依据）：
  - 2026-07-05（第三方实测记录 [12]）：Lite / Pro / Max 折后约 $16.2 / $64.8 / $144（原价 $18 / $72 / $160；Pro 5×、Max 20×），另有非高峰 0.67 系数折算活动（至 2026-07-31）。
  - 2026-09-13（官网 [1]）：折后 $12.6 / $56 / $117.6（原价 $18 / $80 / $168；Pro 6×、Max 14×）。
  - 两个月内折扣价、原价与倍率均有调整；官网明示「Prices and plan benefits may change」[1]。
- **商业模式观察（观点）**：套餐与工具解耦——一个 GLM Coding Plan 可在 ZCode、Claude Code 等 20+ 工具通用 [1]。智谱出售的是模型订阅，ZCode 是官方最优承载（harness），以工具体验拉动订阅，与 Anthropic（Claude 订阅 + Claude Code）、OpenAI（订阅 + Codex CLI）策略同构。
- **人民币 / BigModel 国内侧定价**：待核实（本轮仅核实美元挂牌价）。

## 6. 生态与集成

- **Skills 导入与分发**：技能可从 Claude Code、Codex CLI、OpenClaw、Augment、Windsurf 一键导入（symlink 跟随源变化 / copy 解耦两种模式，可选全局或当前项目）[6]。ZCode 无独立技能市场：团队分发技能需打包为 plugin（扁平 `skills/<skill-name>/SKILL.md` 布局）经插件市场分发，插件源支持 GitHub 仓库、git URL 或本地目录 [6]。
- **MCP 导入与配置**：可从 Claude Code、Codex CLI、OpenCode 及通用 `.agents` 目录一键导入 MCP server [7]。工作区声明的 MCP server 在会话启动时自动连接，团队可将 MCP 配置提交进仓库随克隆生效；官方同时提示打开来源不明的仓库前应检查其 `.zcode/config.json`（MCP server 可执行命令、读写文件、访问网络）；同名时用户配置优先于项目配置 [7]。
- **官方推荐 MCP**：`zai-mcp-server`（视觉理解）、`web-search-prime`（联网搜索）、`web-reader`（网页读取解析），通常需智谱 API Token [7]。
- **插件市场**：插件支持按工作区安装、更新提醒与一键更新（v3.11.2）[2]；v3.10.1 起提示模板可一键安装其引用的插件 [2]。
- **IM 与社区**：Bot Channel 官方文档当前列出 WeChat、Feishu，DingTalk / Discord / WeCom 标注为后续版本 [5]；Telegram 仅见于官网主页营销区 [1]。飞书 / Lark 回复固定为流式卡片（单卡原地更新、工具调用折叠展示），Bot 管理支持回复粒度与 workspace 访问范围限制 [5]。另有 Discord 用户社区与 X 账号 @zcode_ai [3]。
- **生态规模**：插件 / 技能数量与用户规模无公开数据，待核实。

## 7. 优势与局限

### 优势

- 官方 harness 与 GLM-5.3 同步联调，模型、工具、执行工作流一体化，避免「模型与壳割裂」[1][3]。
- Goal 模式 + 1M 上下文直击长任务「断片」痛点，任务内状态不丢 [3][12]。
- 多端控制落地在国内 IM 生态（微信 / 飞书）里差异化明显，长任务可离桌推进 [4][5]。
- 迁移友好：Skills（Claude Code / Codex CLI / OpenClaw / Augment / Windsurf）与 MCP（Claude Code / Codex CLI / OpenCode）均可一键导入，切换成本低 [6][7]。
- 全链路国产（GLM 模型 + 本地执行 + BigModel / Z.ai 账号），满足部分团队的供应链合规诉求 [12]（第三方作者动机，观点性）。
- 闲时任务免费、Weekend Plan 等权益设计有定价创新 [3]。

### 局限与风险

- 深度适配仅限 GLM 系列；BYOK 第三方模型可用，但官方调优承诺不覆盖 [3][12]。
- 价格与套餐倍率短期内多次变动，重度用户的成本可预测性弱 [1][12]。
- 版本迭代快，changelog 显示大量稳定性修复（远程连接、插件加载、模型配置、Windows Computer Use 等）[2]；早期有模型选择异常的用户反馈 [11]。
- 自定义子智能体仅用户级（Beta），项目级不可用；内置角色不可编辑删除 [8]。
- Remote Control 连接链接本身即授权凭据，存在链接转发泄露风险（官方提供二维码刷新止损）[4]；工作区 MCP 自动连接对不受信仓库构成攻击面 [7]。
- Linux 为 Beta [1]。
- 模型与工具同源的单厂商绑定：更换模型供应商时，深度适配红利不可迁移。
- GLM-5.3 时代（2026-08 后）的独立第三方深度实测稀缺，真实能力边界待核实。

## 8. 竞品速览

| 维度 | ZCode | Cursor | Claude Code |
|---|---|---|---|
| 形态 | 桌面 ADE（Electron），多端控制 | VS Code Fork IDE | 终端 CLI 为主 |
| 核心模型 | GLM-5.3 深度适配，支持 BYOK | 多模型切换，无单一深度适配（第三方 2025-12 口径） | Anthropic 系模型 |
| 长任务连续性 | Goal 模式 + 1M 上下文，自动迭代到完成 | 待核实（第三方 2025-12 对比称「Agent 能力为辅」，无当前独立证据） | 待核实 |
| 远程控制 | 手机 Remote + 微信 / 飞书 Bot Channel | 待核实（第三方 2025-12 称无原生移动端或 IM 控制） | 终端为主 |
| 定价模式 | GLM Coding Plan $12.6 / 月起（固定月费，20+ 工具通用） | 订阅 $20+ / 月（第三方 2025-12 口径） | 订阅制（Claude Plans），价格细节待核实 |
| 平台 | macOS / Windows / Linux Beta | macOS / Windows / Linux | 跨平台 CLI |

> Cursor 列采自 ai-bot.cn 的对比口径（2025-12）[11]，时间较早且属单一第三方判断，除标注「待核实」者外仅作参考；Claude Code 列仅列基本形态事实，未逐项核实。

## 9. 适用场景推荐

- **适合**：目标可一句话描述、需多轮执行与验证的长任务（重构、清理编译错误、补测试）；经常离桌、需要 IM / 手机跟进任务的开发者；已订阅 GLM Coding Plan、想让同一套餐复用到多个工具的团队；有国产化 / 供应链合规要求的团队。
- **不适合**：以轻量补全、小片段生成为主的用户（IDE 插件更轻）；深度依赖非 GLM 模型特性的工作流（深度适配仅 GLM）；需要稳定 Linux 生产环境的用户（Beta）。

## 信息来源

| # | 来源 URL | 类型（官方/第三方/实测） | 访问日期 |
|---|---|---|---|
| 1 | https://zcode.z.ai/en | 官方（官网主页 / 下载 / 定价） | 2026-09-13 |
| 2 | https://zcode.z.ai/en/changelog | 官方（版本发布记录） | 2026-09-13 |
| 3 | https://zcode.z.ai/en/docs/welcome | 官方（文档 Welcome） | 2026-09-13 |
| 4 | https://zcode.z.ai/en/docs/remote-control | 官方（Remote Control 文档） | 2026-09-13 |
| 5 | https://zcode.z.ai/en/docs/bot-channel | 官方（Bot Channel 文档） | 2026-09-13 |
| 6 | https://zcode.z.ai/en/docs/skill | 官方（Skill 文档） | 2026-09-13 |
| 7 | https://zcode.z.ai/en/docs/mcp-services | 官方（MCP 文档） | 2026-09-13 |
| 8 | https://zcode.z.ai/en/docs/subagents | 官方（Subagents 文档） | 2026-09-13 |
| 9 | https://zcode.z.ai/en/docs/safety-confirm | 官方（Safety Confirmation 文档） | 2026-09-13 |
| 10 | https://www.runoob.com/vibe-coding/zcode-usage.html | 第三方（教程站） | 2026-09-13 |
| 11 | https://ai-bot.cn/sites/69134.html | 第三方（导航站收录，2025-12-26） | 2026-09-13 |
| 12 | https://www.cnblogs.com/youring2/p/21143184 | 第三方（上手实测，2026-07-05） | 2026-09-13 |

## 更新记录

| 日期 | 变更 |
|---|---|
| 2026-09-13 | 初版（官方 3 个页面 + 3 家第三方来源；无一手实测） |
| 2026-09-13 | 细化 Skills / MCP 导入、Remote Control、Bot Channel、子智能体与安全确认的官方口径；补充页面级来源、版本归因与风险边界。 |
