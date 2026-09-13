# AI Agent Radar 📡

系统性分析市面上流行的 AI Agent 产品，沉淀**可分享、可追溯**的分析报告。

## 报告类型

| 类型 | 目录 | 说明 |
|---|---|---|
| 产品深度报告 | `reports/products/<category>/<product>/` | 单产品常青总览 + 时间点专题 |
| 横向对比 | `reports/comparisons/<category>/` | 同赛道多产品对比评测 |
| 赛道全景 | `reports/landscapes/` | 某一赛道的玩家地图与竞争格局 |
| 趋势观察 | `reports/trends/YYYY-MM/` | 跨赛道趋势与月度观察 |
| 短笔记 | `notes/` | 单点观察与素材卡片，可成长为正式报告 |
| 原始素材 | `sources/YYYY-MM/` | 链接清单、截图、发布会要点等原始信息 |

## 目录结构

```text
ai-agent-radar/
├── README.md            # 本文件：项目定位与导航
├── AGENTS.md            # 人机协作规范（Agent 开工前必读）
├── taxonomy.md          # 分类与标签体系：唯一事实来源
├── index.md             # 报告总索引：新增报告必须登记
├── templates/           # 四类报告标准模板
├── reports/
│   ├── products/        # 产品深度报告（按赛道分子目录）
│   ├── comparisons/     # 横向对比
│   ├── landscapes/      # 赛道全景
│   └── trends/          # 趋势观察（按月份）
├── notes/               # 短笔记（YYYY-MM-DD-<slug>.md）
├── sources/             # 原始素材归档（按月份）
└── assets/              # 图片与图表
```

## 快速开始：写第一份报告

1. 读 [AGENTS.md](AGENTS.md)（协作规范）和 [taxonomy.md](taxonomy.md)（分类体系）。
2. 从 `templates/` 复制对应模板到目标路径。
3. 写作：结论前置、事实标注来源与截至日期。
4. 在 [index.md](index.md) 登记一行。
5. 提交：`git commit -m "report: add <topic>"`。

## 索引

所有已发布报告见 [index.md](index.md)。
