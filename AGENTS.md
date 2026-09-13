# AGENTS.md — AI Agent Radar 协作规范

本仓库用于系统性分析、沉淀和分享市面上流行的 AI Agent 产品。报告由人和 AI Agent 共同维护。**任何 Agent 在本仓库内执行任务前，必须先完整阅读本文件。**

## 1. 项目使命

- 追踪市面上流行的 AI Agent 产品（通用助理、编程、深度研究、Computer Use、框架平台等）。
- 产出四类正式报告：产品深度报告、横向对比、赛道全景、趋势观察；短笔记作为储备素材。
- 保证信息可追溯：所有关键事实必须标注来源与「截至日期」，杜绝编造。

## 2. 开工前必读

| 文件 | 作用 |
|---|---|
| `taxonomy.md` | 产品分类与标签的唯一事实来源 |
| `index.md` | 全部报告的总索引，新增报告后必须登记 |
| `templates/` | 四类报告的标准模板，新报告必须从模板复制起步 |

规则优先级：本文件 > `taxonomy.md` > 各目录 README > 报告正文。

## 3. 目录职责

```text
reports/products/<category>/<product-slug>/   单产品深度报告
reports/comparisons/<category>/               同赛道横向对比
reports/landscapes/                           赛道全景（常青）
reports/trends/YYYY-MM/                       趋势观察（时间点）
notes/                                        短笔记（YYYY-MM-DD-<slug>.md）
sources/YYYY-MM/                              原始素材归档
assets/                                       图片与图表
```

## 4. 命名与路径规范

- 一律小写 kebab-case：`cursor/`、`2026-09-coding-agents-comparison.md`。
- 日期格式：文件名与目录用 `YYYY-MM`（月）或 `YYYY-MM-DD`（日）；front matter 用完整日期。
- 单产品目录内：
  - `overview.md`：常青总览，持续更新，文末维护「更新记录」；
  - `YYYY-MM-<topic>.md`：时间点专题（版本解读、定价快照、实测记录）。
- 横向对比：`reports/comparisons/<category>/YYYY-MM-<topic>.md`。
- 赛道全景：`reports/landscapes/<category>.md`。
- 趋势观察：`reports/trends/YYYY-MM/<topic>.md`。
- 图片：`assets/` 下按报告类型建子目录，文件名含来源与日期。

## 5. 新增报告标准流程

1. 判断报告类型与 category；category 必须已存在于 `taxonomy.md`，否则先新增分类（更新 taxonomy + 建目录）。
2. 复制 `templates/` 对应模板到目标路径，先填 front matter。
3. 检索最新公开信息：官网、官方文档、changelog、权威媒体、一手试用；记录来源 URL 与访问日期。
4. 写作：事实与观点分离；价格、模型版本、能力边界必须标注「截至 YYYY-MM-DD」。
5. 在 `index.md` 对应表格登记一行。
6. 提交 commit：`report: add <topic>`。

## 6. 内容质量红线

- **禁止编造数据或引用不存在的来源**；不确定的结论显式标注「待核实」。
- 每份正式报告的关键事实至少 3 个独立来源（官方 + 第三方混合）。
- 区分「官方宣称」与「实测体验」；实测必须注明环境、版本与日期。
- 涉及评分 / 排名，先在报告内声明评分标准与权重。
- 不提交密钥、内部链接、个人隐私数据。

## 7. 更新规则

- `overview.md` 与 `landscapes/*.md` 是常青文档：直接更新正文，并在文末「更新记录」追加一行。
- 带日期的文件是时间快照：发布后不回改正文，新信息以 `## 更新记录` 追加，或另开新快照。
- `taxonomy.md` 变更保持向后兼容：只新增 slug，不改既有 slug 含义。

## 8. 写作风格

- 中文为主，产品名与专有名词保留英文原文。
- 结论前置：每份正式报告开头必须有 TL;DR（3–5 条）。
- 对比用 Markdown 表格；架构 / 流程可用 Mermaid 图。
- 目标读者：关注 AI Agent 领域的技术从业者与产品经理；避免营销话术。

## 9. Git 规范

- 小改动直接提交 `main`；系列报告可开 `report/<topic>` 分支。
- Commit 前缀：`report:` / `taxonomy:` / `template:` / `index:` / `chore:`。
- 每次提交保持「报告 + 索引」同步：不允许报告已入库而 `index.md` 缺登记。
