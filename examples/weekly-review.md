# 示例：周报生成

## 老板提问

"给我做个这周的经营周报"

## 推荐调用顺序

### 第一步：获取周度分析数据
```bash
bin/cordys-boss analysis customer-overview '{"granularity":"WEEK"}'
bin/cordys-boss analysis customer-portrait '{"granularity":"WEEK"}'
bin/cordys-boss analysis order-overview '{"granularity":"WEEK"}'
bin/cordys-boss analysis order-cost '{"granularity":"WEEK"}'
```

### 第二步：获取合同数据
```bash
# 本周活跃合同（开始或结束时间在本周）
bin/cordys-boss crm page contract '{"combineSearch":{"searchMode":"OR","conditions":[{"value":"WEEK","operator":"DYNAMICS","name":"startTime","multipleValue":false,"type":"TIME_RANGE_PICKER"},{"value":"WEEK","operator":"DYNAMICS","name":"endTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'

# 本周新签合同
bin/cordys-boss crm page contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"WEEK","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'

# 合同整体统计
bin/cordys-boss crm statistic contract
```

### 第三步：获取目标和统计数据
```bash
bin/cordys-boss target home
bin/cordys-boss target team
bin/cordys-boss target ranking
bin/cordys-boss home-stat lead
bin/cordys-boss home-stat opportunity
bin/cordys-boss home-stat opportunity-success
```

## 分析流程

1. **CEO 层**：汇总全部数据，生成经营健康度仪表盘
2. **CFO 层**：合同GMV + 成本 → 计算 ROI → 商务四象限分析
3. **COO 层**：工单状态 + 超期 + 损耗 → 执行力排名
4. **CMO 层**：达人增长 + 结构 + 漏斗 → 增长质量判断
5. **交叉分析**：识别跨维度风险和机会
6. **CEO 总结**：风险排序 + 资源配置 + 下周关键动作

## 期望输出

按照 `prompts/report-templates.md` 中的周报模板输出，确保：

- 经营健康度仪表盘（进度条）
- 核心指标周看板（表格对比）
- CFO 视角（合同+成本+ROI+四象限图）
- COO 视角（履约+排名+损耗+进度条）
- CMO 视角（增长+结构+漏斗+生命周期）
- CEO 总结（风险热力图+资源配置+下周动作）
- 至少 6 张可视化图表

## 周报合同数据核心规则

合同的「开始时间」或「结束时间」落在本周区间内 → 纳入本周统计。

这意味着：
- 本周新开始的合同（startTime 在本周）
- 本周到期的合同（endTime 在本周）
- 都是本周关注的重点合同
