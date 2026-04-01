# Cordys Boss Copilot — 高管顾问团

面向**老板 / 核心管理层**的只读经营 Copilot。
它不是一个通用 CRM 助手，也不是帮一线同学点按钮的自动化脚本，而是一个把 CRM 数据转化为**经营判断、风险提示和可视化分析报告**的专属 Skill。

## 产品定位

这个工程解决的核心问题不是"怎么拿到数据"，而是：

1. 稳定接入老板真正关心的全模块数据
2. 四个高管角色（CEO/CFO/COO/CMO）从不同视角提供专业分析
3. 用可视化图表让数据一目了然
4. 输出可直接使用的结论、风险和行动建议

## 核心业务口径

| 概念 | 说明 |
|------|------|
| 合同金额 = GMV | 直播产生的 GMV 录入合同表单，已筛除退款金额 |
| 财务核算 | 关注合同表单中的金额即可，不关注回款表单 |
| 样品单价 | 日常售卖价格，用于成本预估，不是产品实际成本 |

## 四角色顾问体系

| 角色 | 定位 | 核心关注 |
|------|------|---------|
| **CEO** | 全局经营操盘手 | 经营健康度、战略方向、风险排序、资源配置 |
| **CFO** | 财务与投入产出分析师 | 成本控制、GMV/ROI、合同分析、预算预警 |
| **COO** | 运营执行力分析师 | 履约效率、流程瓶颈、团队执行力、目标达成、损耗管控 |
| **CMO** | 达人生态与增长分析师 | 达人结构、平台健康度、增长质量、转化漏斗、客户生命周期 |

### 四角色联动分析

```
                  ┌─────────────────┐
                  │    CEO 决策层    │
                  │ 战略/资源/风险   │
                  └────┬───────┬────┘
                       │       │
            ┌──────────┘       └──────────┐
            ▼                              ▼
  ┌─────────────────┐            ┌─────────────────┐
  │    CFO 财务层    │◄──────────►│    COO 执行层    │
  │ 成本/GMV/ROI     │            │ 效率/目标/履约    │
  └────────┬────────┘            └────────┬────────┘
           │                              │
           └──────────┐  ┌────────────────┘
                      ▼  ▼
            ┌─────────────────┐
            │    CMO 增长层    │
            │ 达人/平台/漏斗   │
            └─────────────────┘
```

## 绝对边界

- 仅面对老板 / 核心管理层开放
- 只读，不执行任何写操作
- 不混入 `CordysCRM` 主仓库，独立演进

## 项目结构

```text
CordysBossCopilot/
├── README.md                          # 项目说明
├── SKILL.md                           # 技能定义（角色体系/命令/规则）
├── .env.example                       # 环境变量模板
├── bin/
│   └── cordys-boss                    # 只读 CLI（5 大命令族 + 统计扩展）
├── references/
│   ├── crm-api.md                     # CRM 通用 API 参考
│   ├── data-analysis-api.md           # 数据分析接口参考
│   ├── sample-order-api.md            # 样品工单只读接口参考
│   ├── contract-analysis-api.md       # 合同（GMV）分析参考（含统计/客户统计）
│   ├── target-api.md                  # 商务目标管理接口参考
│   ├── home-stat-api.md               # 首页统计接口参考
│   └── readonly-boundary.md           # 只读安全边界说明
├── prompts/
│   ├── role-ceo.md                    # CEO 角色分析框架（含决策树/KPI联动）
│   ├── role-cfo.md                    # CFO 角色分析框架（含四象限/决策树）
│   ├── role-coo.md                    # COO 角色分析框架（含流程效率/决策树）
│   ├── role-cmo.md                    # CMO 角色分析框架（含生命周期/决策树）
│   ├── visualization-rules.md         # 可视化数据分析规则（含四象限图）
│   ├── report-templates.md            # 报告模板（日报/周报/月报/专题）
│   ├── boss-insight-rules.md          # 洞察输出规则
│   └── risk-scan-rules.md             # 风险巡检规则
└── examples/
    ├── daily-brief.md                 # 日报示例
    └── weekly-review.md               # 周报示例
```

## CLI 命令族

| 命令族 | 用途 | 示例 |
|--------|------|------|
| `analysis` | 经营数据分析（看盘） | `cordys-boss analysis customer-overview` |
| `sample-order` | 样品工单只读（风险下钻） | `cordys-boss sample-order page` |
| `crm` | CRM 核心模块只读 | `cordys-boss crm page contract` |
| `crm statistic` | 模块统计数据 | `cordys-boss crm statistic contract` |
| `crm customer-stat` | 客户维度统计 | `cordys-boss crm customer-stat contract <id>` |
| `crm customer-page` | 客户子资源分页 | `cordys-boss crm customer-page contract <id>` |
| `crm chart` | 模块图表数据 | `cordys-boss crm chart account` |
| `crm contract-invoice-stat` | 合同发票统计 | `cordys-boss crm contract-invoice-stat <id>` |
| `target` | 商务目标管理 | `cordys-boss target ranking` |
| `home-stat` | 首页统计 | `cordys-boss home-stat lead` |

## 环境变量

创建 `.env`：

```ini
CORDYS_ACCESS_KEY=老板专属只读AccessKey
CORDYS_SECRET_KEY=老板专属只读SecretKey
CORDYS_CRM_DOMAIN=https://你的CRM域名
```

## 报告能力

所有报告通过 webhook 推送到飞书/钉钉/企业微信，按倒金字塔结构设计（结论→证据→详情）。

### 日报（30秒扫完）
一句话结论 + 关键数字 + 目标进度条 + 今日必须盯。至少 3 张图表。

### 周报（3分钟掌握全貌）
- **合同周数据规则**：合同的「开始时间」或「结束时间」落在本周区间内的合同均纳入统计
- 目标进度 → 核心指标看板 → Top3 风险 → 投入产出 → 履约执行 → 达人生态 → 老板 To-Do
- 按决策优先级排序，不按角色分段。至少 8 张图表。

### 月报（10分钟深度复盘）
CEO 摘要 + 月度目标达成 + 财务分析 + 运营分析 + 达人分析 + 下月策略与资源配置。至少 10 张图表。

### 专题分析
- 商务个人复盘（业绩+成本+效率全维度）
- 平台专题分析（达人+成本+转化全平台对比）
- 达人深度分析（GMV贡献+投入分析+合作评估）

### 可视化标准
- 图表用 Markdown 代码块包裹，等宽字体对齐
- 用 `█░` 条形图 + Markdown 表格 + emoji 状态指示
- 每张图表附带一句话结论
- 不用 ASCII 框线艺术（webhook 渲染会错位）
- 支持四象限分析图、漏斗图、热力图

## 分析方法论

### 三层分析法
```
第一层：看盘（analysis 接口 → 经营总览）
第二层：下钻（sample-order/crm 接口 → 具体明细）
第三层：判断（四角色交叉分析 → 结论+风险+建议）
```

### 角色决策树
每个角色都配备了决策树，根据数据自动触发分析路径，确保不遗漏关键风险。

### 交叉分析
当一个问题涉及多个维度时，自动触发交叉分析（如成本异常→CFO发现→COO追溯→CEO决策）。

## 成功标准

- 老板 5 分钟内获得当天经营结论
- AI 回答中"建议动作"占比稳定
- 关键经营问题无需打开 CRM 页面即可回答
- 每份报告都有充足的可视化图表辅助决策
- 四个角色视角能覆盖老板所有经营决策需求
- 交叉分析能发现单一视角无法发现的深层问题
