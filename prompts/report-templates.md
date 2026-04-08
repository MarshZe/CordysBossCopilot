# 报告模板系统

> 配套文件：`chart-generation.md`（图表生成规范）、`html-report-design.md`（HTML设计系统）

## 设计原则

1. **双层交付**：Webhook 简报（30秒看完） + HTML 完整报告（可视化分析）
2. **数据真实**：所有数值**必须**来自 API 返回，禁止编造、估算、占位
3. **决策导向**：每个数据点背后是判断，每个判断背后是行动建议
4. **第一性原理**：不做表面数据罗列，追问"为什么"和"所以呢"
5. **移动友好**：简报在手机竖屏 3 屏内看完；HTML 报告移动端优先

## 交付架构

```
┌─────────────────────────┐
│   Webhook 简报（文本）    │  ← 推送至飞书/钉钉/企微
│   · 一句话总结            │  ← 老板30秒决策
│   · 4-6个核心指标         │  ← 数字+趋势+状态
│   · 2-3条必须盯的事       │  ← 风险+行动
│   · HTML报告链接          │  ← 点击查看详情
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│   HTML 完整报告（COS）    │  ← 可视化Dashboard
│   · 核心指标卡片          │
│   · 执行摘要              │
│   · CEO/CFO/COO/CMO板块  │
│   · 每板块含图表+分析+建议│
│   · 图表由Python生成      │
└─────────────────────────┘
```

---

## CLI 调用约束（所有报告必读）

**`bin/cordys-boss` 不支持 `--range` 等命令行 flag。** 时间范围通过 JSON body 传递：

- **analysis 命令**：通过 `granularity` 字段控制时间范围
  ```bash
  cordys-boss analysis customer-overview '{"granularity":"TODAY"}'
  cordys-boss analysis order-cost '{"granularity":"WEEK"}'
  cordys-boss analysis customer-overview '{"granularity":"LAST_WEEK"}'
  ```
- **不传 JSON 时**：默认 `granularity: MONTH`
- **crm statistic 命令**：通过 `combineSearch.conditions` 传递时间筛选
  ```bash
  cordys-boss crm statistic contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"WEEK","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'
  ```
- **home-stat / target 命令**：接受可选 JSON，不传则用默认参数

---

## 一、日报

### 必须调用的 API（共 8 个）

| # | 命令 | 用途 | 关键字段 |
|:-:|:-----|:-----|:---------|
| 1 | `cordys-boss analysis customer-overview '{"granularity":"TODAY"}'` | 今日达人概况 | statCards(total, newThisMonth, abLevel, followed), growthTrend |
| 2 | `cordys-boss analysis customer-overview` | 本月达人概况（默认MONTH） | statCards(total, newThisMonth, followed) |
| 3 | `cordys-boss analysis customer-portrait` | 客户画像分布（默认MONTH） | salesTierDistribution, levelDistribution, platformDistribution |
| 4 | `cordys-boss analysis order-overview '{"granularity":"TODAY"}'` | 今日工单概况 | statCards(total, newThisMonth, shipped, pending), statusFunnel |
| 5 | `cordys-boss analysis order-overview` | 本月工单概况（默认MONTH） | statCards, orderTrend |
| 6 | `cordys-boss analysis order-cost` | 本月成本分析（默认MONTH） | statCards(totalCost, avgPerOwner, avgPerOrder), ownerCostRanking |
| 7 | `cordys-boss home-stat lead` | 线索统计 | todayClue, thisWeekClue, thisMonthClue |
| 8 | `cordys-boss home-stat opportunity` | 商机统计 | todayOpportunityCount, thisMonthOpportunityAmount |

### 日报需生成的图表（共 5 张）

| 图表 | 类型 | 数据来源 | 文件名 |
|:-----|:-----|:---------|:-------|
| 销售层级分布 | 水平条形图 | customer-portrait.salesTierDistribution | sales-tier.png |
| 工单状态分布 | 水平条形图 | order-overview.statusFunnel | order-status.png |
| 达人等级分布 | 环形图 | customer-portrait.levelDistribution | level-distribution.png |
| 平台分布 | 水平条形图 | customer-portrait.platformDistribution | platform-distribution.png |
| 商务成本排名 | 水平条形图 | order-cost.ownerCostRanking | owner-cost-top.png |

### Webhook 简报模板

```markdown
# 📋 经营日报 · {{MM月DD日}} {{周X}}

> 💡 {{一句话经营判断：基于今日数据的核心结论，15字以内}}

📊 **核心指标**

| 指标 | 数值 | 环比 | 状态 |
|:-----|-----:|-----:|:----:|
| 达人总量 | {{total}}人 | {{vs昨日}} | {{🟢🟡🔴}} |
| 本月新增 | {{newThisMonth}}人 | {{vs上月同期}} | {{🟢🟡🔴}} |
| 今日跟进 | {{followed}}人 | — | {{🟢🟡🔴}} |
| 在途工单 | {{pending}}单 | {{vs昨日}} | {{🟢🟡🔴}} |
| 已发货 | {{shipped}}单 | {{vs昨日}} | {{🟢🟡🔴}} |
| 本月样品成本 | {{totalCost}}万 | {{vs上月}} | {{🟢🟡🔴}} |

⚠️ **今日必须盯**

1. {{最紧急的事}} — {{数据依据}} — **建议：{{行动}}**
2. {{第二紧急的事}} — {{数据依据}} — **建议：{{行动}}**

📈 **今日亮点**

- {{正面发现1}}
- {{正面发现2}}

📎 [查看完整可视化报告]({{HTML_REPORT_URL}})

---
🤖 CordysBossCopilot · 数据截至 {{HH:MM}}
```

### HTML 完整报告结构（日报）

```
├── 头部（日报标题 + 日期 + 一句话判断）
├── 核心指标卡片（2×3 网格）
│   ├── 达人总量（total + 环比）
│   ├── 本月新增（newThisMonth + 环比）
│   ├── 跟进覆盖率（followed/total 百分比）
│   ├── 在途工单（pending）
│   ├── 已发货（shipped）
│   └── 本月样品成本（totalCost，注意分→万换算）
├── 执行摘要（3-5条要点，红/黄/绿标识）
├── CEO 板块
│   ├── 【图表】销售层级分布（sales-tier.png）
│   │   └── 洞察：各层级客户数量变化趋势和结构健康度
│   ├── 经营健康度评估
│   └── 建议卡（1-2条）
├── CFO 板块
│   ├── 【图表】商务成本排名（owner-cost-top.png）
│   │   └── 洞察：成本集中度分析，人效差异
│   ├── 本月预算执行情况
│   └── 建议卡（1-2条）
├── COO 板块
│   ├── 【图表】工单状态分布（order-status.png）
│   │   └── 洞察：待处理积压是否超标，寄样效率
│   ├── 执行效率评估
│   │   ├── 寄样完成率 = shipped / (shipped + pending) × 100%
│   │   ├── 待处理积压 = pending 数量
│   │   └── 跟进覆盖率 = followed / total × 100%
│   └── 建议卡（1-2条）
├── CMO 板块
│   ├── 【图表】达人等级分布（level-distribution.png）
│   │   └── 洞察：A+B级占比变化
│   ├── 【图表】平台分布（platform-distribution.png）
│   │   └── 洞察：平台集中度风险
│   ├── 新增达人渠道分析
│   └── 建议卡（1-2条）
└── 页脚
```

---

## 二、周报

### 必须调用的 API（共 12 个）

| # | 命令 | 用途 | 关键字段 |
|:-:|:-----|:-----|:---------|
| 1 | `cordys-boss analysis customer-overview '{"granularity":"WEEK"}'` | 本周达人概况 | statCards, growthTrend, cumulativeTrend |
| 2 | `cordys-boss analysis customer-overview '{"granularity":"LAST_WEEK"}'` | 上周达人概况 | statCards（用于环比计算） |
| 3 | `cordys-boss analysis customer-portrait '{"granularity":"WEEK"}'` | 本周客户画像 | salesTierDistribution, levelDistribution, platformDistribution, platformLevelMatrix |
| 4 | `cordys-boss analysis customer-portrait '{"granularity":"LAST_WEEK"}'` | 上周客户画像 | salesTierDistribution（用于层级环比） |
| 5 | `cordys-boss analysis order-overview '{"granularity":"WEEK"}'` | 本周工单概况 | statCards, statusFunnel, orderTrend |
| 6 | `cordys-boss analysis order-overview '{"granularity":"LAST_WEEK"}'` | 上周工单概况 | statCards（用于环比） |
| 7 | `cordys-boss analysis order-cost '{"granularity":"WEEK"}'` | 本周成本分析 | statCards, ownerCostRanking, costTrend, purposeDistribution |
| 8 | `cordys-boss analysis order-cost '{"granularity":"LAST_WEEK"}'` | 上周成本分析 | statCards（用于环比） |
| 9 | `cordys-boss home-stat lead` | 线索统计 | thisWeekClue, thisMonthClue |
| 10 | `cordys-boss home-stat opportunity` | 商机统计 | thisWeekOpportunityCount/Amount, thisMonthOpportunityCount/Amount |
| 11 | `cordys-boss target home` | 目标进度 | myTarget, teamSummary, customerTopN, contractTopN, alerts |
| 12 | `cordys-boss crm statistic contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"WEEK","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'` | 本周合同统计 | amount, averageAmount |

### 环比计算规则

**重要**：`StatCardItem.previousValue` 在当前后端实现中为 `null`，环比必须通过对比两个时间段的数据手动计算：

```
环比变化率 = (本期值 - 上期值) / 上期值 × 100%
```

对于每个需要环比的指标，必须同时调用本期和上期的 API。

### 周报需生成的图表（共 10 张）

| 图表 | 类型 | 数据来源 | 文件名 |
|:-----|:-----|:---------|:-------|
| 销售层级分布 | 水平条形图 | customer-portrait.salesTierDistribution | sales-tier.png |
| 达人等级分布 | 环形图 | customer-portrait.levelDistribution | level-distribution.png |
| 平台分布 | 水平条形图 | customer-portrait.platformDistribution | platform-distribution.png |
| 平台×等级热力图 | 热力图 | customer-portrait.platformLevelMatrix | platform-level-heatmap.png |
| 达人增长趋势 | 柱线混合 | customer-overview.growthTrend + cumulativeTrend | customer-growth.png |
| 工单状态漏斗 | 水平条形图 | order-overview.statusFunnel | order-status.png |
| 工单量趋势 | 折线图 | order-overview.orderTrend | order-trend.png |
| 商务成本排名 | 水平条形图 | order-cost.ownerCostRanking | owner-cost-top.png |
| 成本趋势 | 折线图 | order-cost.costTrend | cost-trend.png |
| 转化漏斗 | 水平条形图 | home-stat lead + opportunity 组合 | conversion-funnel.png |

### Webhook 简报模板

```markdown
# 📊 经营周报 · {{MM月DD日}}-{{MM月DD日}} 第{{N}}周

> 💡 {{一句话本周总结：核心结论，20字以内}}

📈 **本周 vs 上周**

| 指标 | 本周 | 上周 | 环比 | 状态 |
|:-----|-----:|-----:|-----:|:----:|
| 达人总量 | {{total}}人 | {{上周total}} | {{变化率}} | {{🟢🟡🔴}} |
| 新增达人 | {{新增}}人 | {{上周新增}} | {{变化率}} | {{🟢🟡🔴}} |
| 跟进覆盖率 | {{率}}% | {{上周率}}% | {{变化}} | {{🟢🟡🔴}} |
| 工单量 | {{total}}单 | {{上周total}} | {{变化率}} | {{🟢🟡🔴}} |
| 寄样完成率 | {{率}}% | {{上周率}}% | {{变化}} | {{🟢🟡🔴}} |
| 样品成本 | {{cost}}万 | {{上周cost}} | {{变化率}} | {{🟢🟡🔴}} |
| 商机金额 | {{金额}}万 | {{上周金额}} | {{变化率}} | {{🟢🟡🔴}} |

📊 **销售层级分布**（客户质量结构）

{{用 visualization-rules.md 中的占比分布图格式展示 salesTierDistribution}}

⚠️ **本周需要关注**

1. {{风险1}} — {{依据}} — **建议：{{行动}}**
2. {{风险2}} — {{依据}} — **建议：{{行动}}**
3. {{风险3}} — {{依据}} — **建议：{{行动}}**

✅ **本周亮点**

1. {{亮点1}}
2. {{亮点2}}

📎 [查看完整可视化报告]({{HTML_REPORT_URL}})

---
🤖 CordysBossCopilot · {{日期}} 周报
```

### HTML 完整报告结构（周报）

```
├── 头部（周报标题 + 周期 + 一句话判断）
├── 核心指标卡片（2×4 网格）
│   ├── 达人总量（total + 环比）
│   ├── 本周新增（newThisMonth + 环比）
│   ├── 跟进覆盖率（followed/total % + 环比）
│   ├── 工单总量（total + 环比）
│   ├── 寄样完成率（shipped/(shipped+pending) % + 环比）
│   ├── 样品总成本（totalCost + 环比，分→万）
│   ├── 人均成本（avgPerOwner + 环比，分→万）
│   └── 商机金额（opportunityAmount + 环比）
├── 执行摘要（5-8条要点）
│
├── CEO 板块 · 战略全局
│   ├── 【图表】销售层级分布（sales-tier.png）
│   │   └── 洞察：各层级客户数量 + 与上周对比 + 结构健康度判断
│   ├── 【图表】转化漏斗（conversion-funnel.png）
│   │   └── 洞察：最大断裂点分析 + 转化率趋势
│   ├── 经营风险评估（红/黄/绿三色）
│   │   ├── 增长维度：客户增速 vs 目标
│   │   ├── 效率维度：人效指标
│   │   ├── 结构维度：客户质量分层
│   │   └── 履约维度：工单完成率
│   ├── 建议卡 · 本周战略决策
│   │   ├── action: 最高优先级的事
│   │   ├── risk: 需要预防的风险
│   │   └── opportunity: 发现的机会
│   └── 目标进度（如有 target 数据）
│
├── CFO 板块 · 财务效率
│   ├── 【图表】商务成本排名（owner-cost-top.png）
│   │   └── 洞察：成本 Top3 集中度 + 人效差异
│   ├── 【图表】成本趋势（cost-trend.png）
│   │   └── 洞察：成本变化趋势 + 预算执行率
│   ├── 商务人效分析表
│   │   ├── 表格：商务姓名 | 样品成本 | 客户数 | 单客成本 | 状态
│   │   └── 洞察：人效差异的根因（是客户质量差异还是资源分配不均）
│   ├── 建议卡 · 成本优化
│   │   ├── 需要加码的商务（高效率）
│   │   └── 需要复盘的商务（低效率）
│   └── 合同金额统计
│
├── COO 板块 · 运营执行
│   ├── 【图表】工单状态漏斗（order-status.png）
│   │   └── 洞察：积压量趋势 + 是否需要干预
│   ├── 【图表】工单量趋势（order-trend.png）
│   │   └── 洞察：工单量变化 + 是否有季节性波动
│   ├── 寄样执行看板
│   │   ├── 寄样完成率 = shipped / (shipped + pending) × 100%
│   │   ├── 待发货积压 = pending 绝对值 + 环比
│   │   ├── 平均处理时长（如可获取）
│   │   └── 洞察：瓶颈在哪个环节
│   ├── 客户跟进覆盖分析
│   │   ├── 跟进覆盖率 = followed / total × 100%
│   │   ├── 未跟进客户数 = total - followed
│   │   └── 洞察：哪些客户长期未跟进，潜在风险
│   ├── 建议卡 · 执行优化
│   │   └── 具体到人和时间的执行计划
│   └── followTime 字段说明
│       └── 注意：跟进数据使用 followTime 字段，不使用 latestFollowUpTime（后端bug）
│
├── CMO 板块 · 市场增长
│   ├── 【图表】达人等级分布（level-distribution.png）
│   │   └── 洞察：A+B级占比变化 + 结构优化方向
│   ├── 【图表】平台分布（platform-distribution.png）
│   │   └── 洞察：平台集中度 + 风险评估
│   ├── 【图表】平台×等级热力图（platform-level-heatmap.png）
│   │   └── 洞察：哪个平台的高价值达人密度最高
│   ├── 【图表】达人增长趋势（customer-growth.png）
│   │   └── 洞察：增长是在加速还是放缓
│   ├── 客户结构健康度评估
│   │   ├── salesTierDistribution 各层级数量 + 占比
│   │   ├── 高价值客户占比趋势
│   │   └── 洞察：客户资产在升值还是贬值
│   ├── 建议卡 · 增长策略
│   │   ├── 客户开发重点方向
│   │   ├── 平台布局调整建议
│   │   └── 达人升级培育计划
│   └── 线索转化分析
│
└── 页脚
```

---

## 三、月报

### 必须调用的 API（共 14 个）

| # | 命令 | 用途 | 关键字段 |
|:-:|:-----|:-----|:---------|
| 1 | `cordys-boss analysis customer-overview` | 本月达人概况（默认MONTH） | statCards, growthTrend, cumulativeTrend |
| 2 | `cordys-boss analysis customer-overview '{"granularity":"LAST_MONTH"}'` | 上月达人概况 | statCards（环比） |
| 3 | `cordys-boss analysis customer-portrait` | 本月客户画像（默认MONTH） | salesTierDistribution, levelDistribution, platformDistribution, platformLevelMatrix |
| 4 | `cordys-boss analysis customer-portrait '{"granularity":"LAST_MONTH"}'` | 上月客户画像 | 各分布数据（环比） |
| 5 | `cordys-boss analysis order-overview` | 本月工单（默认MONTH） | statCards, statusFunnel, orderTrend |
| 6 | `cordys-boss analysis order-overview '{"granularity":"LAST_MONTH"}'` | 上月工单 | statCards（环比） |
| 7 | `cordys-boss analysis order-cost` | 本月成本（默认MONTH） | 全部字段（statCards, ownerCostRanking, costTrend, purposeDistribution） |
| 8 | `cordys-boss analysis order-cost '{"granularity":"LAST_MONTH"}'` | 上月成本 | statCards（环比） |
| 9 | `cordys-boss home-stat lead` | 线索统计 | thisMonthClue, thisYearClue |
| 10 | `cordys-boss home-stat opportunity` | 商机统计 | thisMonthOpportunity*, thisYearOpportunity* |
| 11 | `cordys-boss target home` | 目标进度 | 全部字段 |
| 12 | `cordys-boss crm statistic contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"MONTH","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'` | 本月合同统计 | amount, averageAmount |
| 13 | `cordys-boss crm statistic contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"LAST_MONTH","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'` | 上月合同统计 | amount, averageAmount（环比） |
| 14 | `cordys-boss analysis order-cost '{"granularity":"WEEK"}'` | 周度成本趋势 | costTrend（周粒度） |

### 月报需生成的图表（共 12 张）

| 图表 | 类型 | 数据来源 | 文件名 |
|:-----|:-----|:---------|:-------|
| 销售层级分布 | 水平条形图 | customer-portrait.salesTierDistribution | sales-tier.png |
| 达人等级分布 | 环形图 | customer-portrait.levelDistribution | level-distribution.png |
| 平台分布 | 水平条形图 | customer-portrait.platformDistribution | platform-distribution.png |
| 平台×等级热力图 | 热力图 | customer-portrait.platformLevelMatrix | platform-level-heatmap.png |
| 达人增长趋势 | 柱线混合 | customer-overview.growthTrend + cumulativeTrend | customer-growth.png |
| 工单状态漏斗 | 水平条形图 | order-overview.statusFunnel | order-status.png |
| 工单量趋势 | 折线图 | order-overview.orderTrend | order-trend.png |
| 商务成本排名 | 水平条形图 | order-cost.ownerCostRanking | owner-cost-top.png |
| 成本趋势（周度） | 折线图 | order-cost.costTrend（周粒度） | cost-trend-weekly.png |
| 成本用途分布 | 环形图 | order-cost.purposeDistribution | cost-purpose.png |
| 转化漏斗 | 水平条形图 | home-stat lead + opportunity 组合 | conversion-funnel.png |
| ROI 四象限 | 散点图 | order-cost.ownerCostRanking + contract | roi-quadrant.png |

### Webhook 简报模板

```markdown
# 📊 经营月报 · {{YYYY年MM月}}

> 💡 {{一句话月度总结：本月经营核心结论}}

📈 **月度核心指标**

| 指标 | 本月 | 上月 | 环比 | 状态 |
|:-----|-----:|-----:|-----:|:----:|
| 达人总量 | {{total}}人 | {{上月}} | {{率}} | {{🟢🟡🔴}} |
| 新增达人 | {{新增}}人 | {{上月}} | {{率}} | {{🟢🟡🔴}} |
| A+B级占比 | {{率}}% | {{上月}}% | {{变化}} | {{🟢🟡🔴}} |
| 跟进覆盖率 | {{率}}% | {{上月}}% | {{变化}} | {{🟢🟡🔴}} |
| 工单总量 | {{total}}单 | {{上月}} | {{率}} | {{🟢🟡🔴}} |
| 寄样完成率 | {{率}}% | {{上月}}% | {{变化}} | {{🟢🟡🔴}} |
| 样品总成本 | {{cost}}万 | {{上月}} | {{率}} | {{🟢🟡🔴}} |
| 人均样品成本 | {{avg}}万 | {{上月}} | {{率}} | {{🟢🟡🔴}} |
| 商机金额 | {{金额}}万 | {{上月}} | {{率}} | {{🟢🟡🔴}} |
| 合同金额 | {{金额}}万 | {{上月}} | {{率}} | {{🟢🟡🔴}} |

📊 **销售层级分布**

{{用 visualization-rules.md 中的占比分布图格式展示 salesTierDistribution}}

🏢 **CEO 核心判断**

{{2-3句话的月度经营评价，聚焦增长质量和结构变化}}

💰 **CFO 成本洞察**

{{1-2句话的成本效率判断}}

📦 **COO 执行评估**

{{1-2句话的运营效率判断}}

🌱 **CMO 增长评估**

{{1-2句话的市场增长判断}}

⚠️ **下月必须解决的问题**

1. {{问题1}} — {{建议}}
2. {{问题2}} — {{建议}}
3. {{问题3}} — {{建议}}

📎 [查看完整可视化报告]({{HTML_REPORT_URL}})

---
🤖 CordysBossCopilot · {{YYYY年MM月}} 月报
```

### HTML 完整报告结构（月报）

```
├── 头部（月报标题 + 月份 + 一句话判断）
├── 核心指标卡片（2×5 网格）
│   ├── 达人总量 + 环比
│   ├── 月新增达人 + 环比
│   ├── A+B级占比 + 环比
│   ├── 跟进覆盖率 + 环比
│   ├── 工单总量 + 环比
│   ├── 寄样完成率 + 环比
│   ├── 样品总成本 + 环比
│   ├── 人均样品成本 + 环比
│   ├── 商机金额 + 环比
│   └── 合同金额 + 环比
├── 执行摘要（8-10条要点）
│
├── CEO 板块 · 战略全局与经营复盘
│   ├── 【图表】销售层级分布（sales-tier.png）
│   │   └── 洞察：月度层级迁移分析（哪些层级增长/流失了多少客户）
│   ├── 【图表】转化漏斗（conversion-funnel.png）
│   │   └── 洞察：全链路转化效率 + 最大瓶颈
│   ├── 经营健康度雷达评估
│   │   ├── 增长力：新增达人数环比 + 达成率
│   │   ├── 质量度：A+B级占比 + 高价值客户占比
│   │   ├── 效率度：人均产出 + 单客成本
│   │   ├── 履约度：寄样完成率 + 跟进覆盖率
│   │   └── 转化度：线索→商机→合同 转化率
│   ├── 第一性原理分析
│   │   ├── 根因分析：本月经营结果的底层驱动因素
│   │   ├── 假设验证：上月的策略是否生效
│   │   └── 趋势预判：按当前趋势，下月会怎样
│   ├── 建议卡 · 下月战略方向（3条）
│   └── 目标进度 + 差距分析
│
├── CFO 板块 · 财务效率与投产分析
│   ├── 【图表】成本趋势-周度（cost-trend-weekly.png）
│   │   └── 洞察：成本是在收敛还是发散
│   ├── 【图表】成本用途分布（cost-purpose.png）
│   │   └── 洞察：钱花在了哪，花得值不值
│   ├── 【图表】商务成本排名（owner-cost-top.png）
│   │   └── 洞察：人效差异的根因
│   ├── 【图表】ROI 四象限（roi-quadrant.png）
│   │   └── 洞察：每个商务的投产位置 + 策略建议
│   ├── 商务人效分析表（完整版）
│   │   ├── 表格：商务 | 样品成本 | 客户数 | 商机金额 | 合同金额 | ROI | 状态
│   │   └── 洞察：最佳实践（效率最高的商务在做什么）
│   ├── 预算执行分析
│   │   ├── 月度预算 vs 实际支出
│   │   ├── 按当前速率，季度预算可控性
│   │   └── 建议：预算调整方向
│   └── 建议卡 · 下月成本策略（2-3条）
│
├── COO 板块 · 运营效率与执行力
│   ├── 【图表】工单状态漏斗（order-status.png）
│   │   └── 洞察：月度工单处理效率
│   ├── 【图表】工单量趋势（order-trend.png）
│   │   └── 洞察：工单量趋势 + 容量规划
│   ├── 寄样执行月度看板
│   │   ├── 寄样完成率月度值 + 环比
│   │   ├── 待发货积压趋势
│   │   ├── 各商务处理效率对比
│   │   └── 洞察：流程瓶颈识别
│   ├── 客户跟进月度分析
│   │   ├── 跟进覆盖率趋势
│   │   ├── 高价值客户跟进覆盖
│   │   ├── 长期未跟进客户预警
│   │   └── 洞察：跟进资源是否匹配客户价值
│   ├── 建议卡 · 下月执行改进（2-3条）
│   └── 流程优化建议
│
├── CMO 板块 · 市场增长与客户资产
│   ├── 【图表】达人增长趋势（customer-growth.png）
│   │   └── 洞察：增长曲线分析（加速/减速/线性）
│   ├── 【图表】达人等级分布（level-distribution.png）
│   │   └── 洞察：月度等级迁移 + 结构变化
│   ├── 【图表】平台分布（platform-distribution.png）
│   │   └── 洞察：平台多元化程度 + 机会评估
│   ├── 【图表】平台×等级热力图（platform-level-heatmap.png）
│   │   └── 洞察：最具潜力的平台×等级组合
│   ├── 客户资产质量月度评估
│   │   ├── salesTierDistribution 各层级 + 与上月对比
│   │   ├── 高价值客户净增减
│   │   ├── 客户流失预警（长期无跟进+低活跃度）
│   │   └── 洞察：客户资产在升值还是贬值
│   ├── 线索全链路分析
│   │   ├── 线索量 + 转化率
│   │   ├── 各阶段断裂点
│   │   └── 洞察：增长杠杆在哪个环节
│   ├── 建议卡 · 下月增长策略（3条）
│   │   ├── 客户开发策略
│   │   ├── 平台布局调整
│   │   └── 达人培育升级
│   └── 市场趋势与机会
│
└── 页脚
```

---

## 四、分析指标计算公式

以下指标不由后端直接提供，需要 Agent 从 API 数据中计算：

### 寄样完成率
```
寄样完成率 = shipped / (shipped + pending) × 100%
数据来源: order-overview.statCards 中 key=shipped 和 key=pending
```

### 跟进覆盖率
```
跟进覆盖率 = followed / total × 100%
数据来源: customer-overview.statCards 中 key=followed 和 key=total
```

### A+B级占比
```
A+B级占比 = abLevel / total × 100%
数据来源: customer-overview.statCards 中 key=abLevel（这是占比值，已经是百分比）
```

### 人均样品成本
```
人均样品成本 = avgPerOwner (单位: 分) ÷ 100
数据来源: order-cost.statCards 中 key=avgPerOwner
```

### 单均样品成本
```
单均样品成本 = avgPerOrder (单位: 分) ÷ 100
数据来源: order-cost.statCards 中 key=avgPerOrder
```

### 环比变化率
```
环比 = (本期值 - 上期值) / 上期值 × 100%
必须调用两个时间范围的 API，不能依赖 StatCardItem.previousValue（当前为null）
```

### 转化漏斗
```
线索数 = thisMonthClue.value
客户数 = customer-overview.statCards.total
商机数 = thisMonthOpportunityCount.value
合同金额 = contract.amount

漏斗转化率:
  线索→客户 = 客户数 / 线索数 × 100%
  客户→商机 = 商机数 / 客户数 × 100%
```

---

## 五、数据完整性规则

### 必须遵守

1. **不编造数据**：如 API 返回空或错误，在报告中标注"数据获取失败"，不要填充假数据
2. **单位换算**：成本类字段（OrderCostResponse 系列）单位是**分**，展示时必须 ÷100
3. **环比计算**：必须调用两个时间段的 API 进行手动对比
4. **金额格式**：≥ 10000元 展示为 "X.X万"；< 10000元 展示为 "XXXX元"
5. **跟进字段**：使用 `followTime` 而非 `latestFollowUpTime`（后端已知 bug）
6. **销售层级**：使用 API 返回的 `salesTierDistribution.name` 作为层级名称，不硬编码

### API 错误处理

```
如果某个 API 调用失败或返回空数据：
1. 在简报中该指标标注 "—"
2. 在 HTML 报告中该图表位置显示 "数据暂不可用"
3. 在执行摘要中说明哪些数据缺失
4. 不要因为部分数据缺失就跳过整个报告
```

### 图表 + 分析质量检查

生成报告前的自检清单：
- [ ] 所有数值来自 API 返回
- [ ] 成本金额已从分换算为元/万
- [ ] 环比通过两期数据计算，非编造
- [ ] 每张图表附带分析洞察（判断，非复述）
- [ ] 每个角色板块有具体可执行的建议
- [ ] 建议具体到人/时间/金额，非空泛建议
- [ ] HTML 链接正确指向 COS 文件

---

## 六、分析框架指引

### CEO 分析框架

用第一性原理审视经营：

```
1. 我们在赚钱吗？
   → 合同金额趋势 + 商机pipeline + 转化率

2. 我们的客户资产在增值还是贬值？
   → 销售层级迁移 + A+B级占比变化 + 高价值客户净增减

3. 我们的增长是健康的还是虚胖的？
   → 新增客户 vs 成本增速 → 客户获取效率
   → 新增客户的质量（等级分布）vs 数量

4. 最大的风险在哪？
   → 客户集中度 + 平台集中度 + 人员集中度
   → 未跟进客户积压 → 客户流失风险

5. 下一步最高杠杆的事是什么？
   → 投入产出比最高的1-2个动作
```

### CFO 分析框架

用投入产出比审视每一分钱：

```
1. 钱花在哪了？花得值不值？
   → 成本用途分布 + 各商务投入排名

2. 谁的钱花得最好？为什么？
   → ROI 四象限 + 最佳实践提取

3. 谁的钱花得不好？根因是什么？
   → 问题象限分析：是客户质量差？是方法不对？是新人？

4. 按当前速率，预算会不会超？
   → 月度burn rate + 季度预测

5. 哪里加码能产生最大回报？
   → 高 ROI 商务 + 高转化渠道
```

### COO 分析框架

用执行力审视每个流程节点：

```
1. 工单处理效率达标吗？
   → 寄样完成率 + 待处理积压量 + 趋势

2. 客户都被照顾到了吗？
   → 跟进覆盖率 + 未跟进客户列表 + 高价值客户覆盖

3. 瓶颈在哪个环节？
   → 工单状态漏斗 → 哪个状态堆积最多

4. 人力配置合理吗？
   → 各商务工单量分布 + 处理效率差异

5. 下周/下月执行改进的优先级？
   → 影响客户最大的执行短板
```

### CMO 分析框架

用增长视角审视客户资产：

```
1. 客户资产的质量在变好还是变差？
   → 销售层级结构变化 + 高价值客户占比趋势

2. 增长动力来自哪里？
   → 各平台增量贡献 + 各等级增量贡献

3. 哪个平台最有潜力？
   → 平台×等级热力图 → 高价值达人密度

4. 转化链路的最大断裂在哪？
   → 转化漏斗 → 断裂点 → 改进方向

5. 增长在加速还是减速？
   → 增长趋势曲线 → 增长率变化
```

---

## 七、Webhook 渲染约束

- **不用 ASCII 框线艺术**：`━━╔══║╚` 在移动端会错位
- **Markdown 表格**：用标准 Markdown 表格，不用 ASCII 对齐
- **宽度 < 40 字符**：手机竖屏一行最多 40 个半角字符
- **简报 ≤ 3 屏**：核心信息 + 链接，详情在 HTML
- **状态指示**：🟢 健康 / 🟡 关注 / 🔴 预警
- **趋势指示**：↑ 上升 / ↓ 下降 / → 持平
