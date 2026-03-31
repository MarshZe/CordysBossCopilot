# 商务目标管理 API 参考

## 业务背景

商务目标管理模块用于管理销售团队的月度目标，主要包含两个维度：
1. **达人开发数量**：按客户等级分层的月度开发目标
2. **合同金额**：月度 GMV 目标

## 1. 首页目标摘要

### 接口

```http
GET /business-target/home
```

### 用途

快速了解：
- 当月目标整体进度
- 关键维度达成率
- 是否需要预警

### 关键返回字段

- 达人开发各层级目标 & 完成数
- 合同金额目标 & 完成额
- 整体达成率

## 2. 我的目标

### 接口

```http
POST /business-target/my
```

### 请求体

```json
{
  "month": "2026-03",
  "departmentId": null
}
```

### 用途

查看特定人员的目标详情（通常用于老板查看某个商务的目标）。

## 3. 团队目标看板

### 接口

```http
POST /business-target/team
```

### 请求体

```json
{
  "month": "2026-03",
  "departmentId": null
}
```

### 用途

回答：
- 团队整体目标进度
- 各成员目标完成排名
- 哪些维度差距最大
- 需要重点关注哪些人

### 关键返回字段

- 各成员的目标 & 完成数据
- 达人开发各层级数据
- 合同金额目标 & 实际

## 4. 双榜排名

### 接口

```http
POST /business-target/ranking
```

### 请求体

```json
{
  "month": "2026-03",
  "departmentId": null
}
```

### 用途

回答：
- 达人开发数量排名（谁开发得最多/最少）
- 合同金额排名（谁签约最多/最少）
- 排名变动（比上月升/降）

### 关键返回字段

- 达人开发排名列表
- 合同金额排名列表
- 各成员的排名位次

## 5. 历史快照

### 接口

```http
POST /business-target/history
```

### 请求体

```json
{
  "month": "2026-02",
  "departmentId": null
}
```

### 用途

回答：
- 过去几个月的目标完成情况
- 目标完成的趋势（越来越好还是越来越差）
- 历史对比

## 6. 层级选项

### 接口

```http
GET /business-target/level-options
```

### 用途

获取达人开发目标的层级配置（A/B/C/D 级定义等）。

## COO 典型使用场景

### 目标差距分析

```bash
bin/cordys-boss target home              # 整体进度
bin/cordys-boss target team              # 各成员进度
bin/cordys-boss target ranking           # 排名
bin/cordys-boss target history '{"month":"2026-02"}'  # 上月对比
```

### 周会追踪

```bash
bin/cordys-boss target team              # 本月进度
bin/cordys-boss target ranking           # 人效排名
bin/cordys-boss analysis order-cost      # 成本排名（对比产出排名）
```

### 绩效复盘

```bash
bin/cordys-boss target history           # 历史趋势
bin/cordys-boss target ranking           # 当月排名
bin/cordys-boss target level-options     # 层级标准
```

## 注意事项

- 目标数据为月度粒度，查询时需指定 `month` 参数（格式：YYYY-MM）
- 不指定 month 时默认查询当月
- 完成值数据来源：
  - 达人开发完成值 → 来自 `customer_tier_achievement` 表
  - 合同金额完成值 → 来自 contracts 模块聚合
- 目标审批通过后不可直接修改，需提交新版本
