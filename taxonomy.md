# 分类与标签体系（Single Source of Truth）

> 本文件是全仓库组织方式的唯一事实来源。修改本文件等于修改目录语义：
> 新增分类 = 更新下表 + 在 `reports/products/` 下建目录（含 `.gitkeep`）。
> 分类只新增不删除；确需废弃时标记 `status: deprecated` 并保留目录。

## 产品分类（categories）

| slug | 中文名 | 定义 | 代表产品（示例，滚动更新） |
|---|---|---|---|
| `general-assistants` | 通用助理 Agent | 面向大众的通用任务助手，以对话为主要交互 | ChatGPT、Gemini、Kimi |
| `coding-agents` | 编程 Agent | 以软件开发为核心任务的自主/半自主编程智能体 | Cursor、Claude Code、Codex、Devin、Copilot |
| `deep-research` | 深度研究 Agent | 自主检索、多跳推理并产出研究报告 | 各家 Deep Research 类产品 |
| `computer-use` | 计算机操作 Agent | 操作浏览器 / 桌面 GUI 完成端到端任务 | Operator 类、Computer Use 类产品 |
| `agent-frameworks` | 框架与平台 | 供开发者构建 Agent 的框架、SDK、低代码平台 | LangGraph、CrewAI、AutoGen、Dify、Coze |
| `workflow-automation` | 工作流自动化 Agent | 以流程自动化为核心、用 Agent 增强灵活性 | Zapier Agents、n8n |
| `multi-agent` | 多智能体编排 | 多 Agent 协作、角色分工与任务编排系统 | 各类 multi-agent 编排框架/产品 |
| `agent-infra` | Agent 基础设施 | 记忆、工具协议、评测、安全、可观测性等支撑层 | MCP、记忆层、evals 工具 |
| `vertical-agents` | 垂直行业 Agent | 深耕特定行业或职能的场景化 Agent | 客服、销售、数据分析、法律 Agent |

## 标签（tags）

| 标签 | 用途 |
|---|---|
| `hot` | 近期热度显著上升 |
| `verified` | 含一手实测（注明环境与日期） |
| `pricing-change` | 定价 / 商业模式变动 |
| `major-release` | 重要版本或能力发布 |
| `rumor` | 未经官方证实的信息，正文中须显式声明 |

## product-slug 规则

- 小写 kebab-case，与官方产品名对应：`Claude Code` → `claude-code`。
- 同一产品的不同端（网页 / App / CLI）归同一 slug，不拆分。
- slug 用产品名而非公司名：`cursor` 而不是 `anysphere-cursor`；仅当产品名冲突时加公司前缀。
