# 数据分析 API 参考

本文件描述老板专属 Copilot 中直接接入的经营分析接口。

## 为什么这组接口是一等能力

这些接口不是普通列表接口，它们已经完成了：
- 聚合
- 趋势
- 分布
- 排名

所以它们应被 Skill 当成“经营能力层”直接使用，而不是让 AI 先查列表、再自己拼指标。

## 1. 达人经营概览

### 接口

```http
POST /data-analysis/customer/overview
```

### 用途

回答：
- 达人总数多少
- 本月新增多少
- A+B 级达人多少
- 本月跟进多少
- 整体趋势如何

### 请求体

```json
{
  "granularity": "MONTH",
  "startTime": null,
  "endTime": null,
  "ownerIds": [],
  "departmentId": "",
  "platformValues": []
}
```

### 关键返回字段

- `data.statCards`
- `data.growthTrend`
- `data.cumulativeTrend`

## 2. 达人画像

### 接口

```http
POST /data-analysis/customer/portrait
```

### 用途

回答：
- 达人结构是什么样的
- 哪个平台达人最多
- 账号类型分布如何
- 销售额层级是否健康
- 合作模式是否失衡

### 关键返回字段

- `data.levelDistribution`
- `data.accountTypeDistribution`
- `data.platformDistribution`
- `data.salesTierDistribution`
- `data.cooperationModeDistribution`
- `data.platformLevelMatrix`

## 3. 工单经营概览

### 接口

```http
POST /data-analysis/order/overview
```

### 用途

回答：
- 工单总数
- 本月新建 / 发货 / 待处理
- 工单状态漏斗
- 紧急程度分布

### 关键返回字段

- `data.statCards`
- `data.orderTrend`
- `data.statusFunnel`
- `data.urgencyDistribution`

## 4. 工单成本分析

### 接口

```http
POST /data-analysis/order/cost
```

### 用途

回答：
- 样品总成本
- 人均成本
- 单工单均成本
- 按用途分类成本
- 商务成本排名
- 产品寄样排名
- 数据完整度风险

### 关键返回字段

- `data.statCards`
- `data.costTrend`
- `data.purposeDistribution`
- `data.ownerCostRanking`
- `data.productRanking`
- `data.dataCoverageRate`

## Skill 使用原则

### 先分析，后下钻

推荐顺序：

```text
老板问经营问题
-> 先走 analysis 接口
-> 找到异常点
-> 必要时再走 sample-order 只读接口下钻
```

### 不要退回列表拼装

不推荐：

```text
page sample-order
-> 自己统计
-> 自己做趋势
```

推荐：

```text
analysis order-overview
analysis order-cost
```

### 默认输出结构

- 结论
- 关键事实
- 风险提醒
- 建议动作
