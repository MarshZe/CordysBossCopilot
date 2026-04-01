# 合同（GMV）分析 API 参考

## 业务口径说明

> **合同金额 = 直播 GMV**（已筛除退款金额），是公司核心收入指标。
> 财务核算关注合同表单中的金额信息即可，**不需要关注回款表单**。
> 回款表单不是核心关注点，CFO 视角下以合同金额为准。

## 1. 合同列表查询

### 接口

```http
POST /contract/page
```

### 用途

回答：
- 本月/本周签了多少合同
- 合同金额合计多少（即 GMV）
- 哪些合同本周开始或到期
- 大单动态（金额排名）
- 各商务的合同签约情况

### 请求体

标准分页结构，支持 `combineSearch` 动态筛选。

### 关键返回字段

- `data.list[].name` — 合同名称
- `data.list[].amount` — 合同金额（= GMV）
- `data.list[].startTime` — 合同开始时间
- `data.list[].endTime` — 合同结束时间
- `data.list[].createTime` — 创建时间
- `data.list[].ownerName` — 负责商务
- `data.list[].status` — 合同状态
- `data.list[].accountName` — 关联客户（达人）
- `data.total` — 总数

## 2. 合同详情

### 接口

```http
GET /contract/{id}
```

### 用途

查看单个合同的完整信息，包括关联客户、金额、时间、审批状态等。

## 3. 周报合同数据查询策略

### 核心规则

周报中合同数据的筛选逻辑：合同的「开始时间」或「结束时间」落在本周区间内的合同均纳入统计。

### 本周活跃合同

```bash
bin/cordys-boss crm page contract '{"combineSearch":{"searchMode":"OR","conditions":[{"value":"WEEK","operator":"DYNAMICS","name":"startTime","multipleValue":false,"type":"TIME_RANGE_PICKER"},{"value":"WEEK","operator":"DYNAMICS","name":"endTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'
```

含义：
- 本周新开始的合同（startTime 在本周范围内）
- 本周到期的合同（endTime 在本周范围内）

### 本周新签合同

```bash
bin/cordys-boss crm page contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"WEEK","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'
```

### 本月合同

```bash
bin/cordys-boss crm page contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":"MONTH","operator":"DYNAMICS","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'
```

### 自定义时间范围

```bash
# 查询两个时间戳之间的合同
bin/cordys-boss crm page contract '{"combineSearch":{"searchMode":"AND","conditions":[{"value":[1711872000000,1712476800000],"operator":"BETWEEN","name":"createTime","multipleValue":false,"type":"TIME_RANGE_PICKER"}]}}'
```

## 4. 合同二级模块

### 回款计划（仅供参考，非核心关注）

```bash
bin/cordys-boss crm page contract/payment-plan [keyword|JSON]
```

> 注意：回款表单不是财务核算的核心，GMV 以合同金额为准。

### 回款记录（仅供参考，非核心关注）

```bash
bin/cordys-boss crm page contract/payment-record [keyword|JSON]
```

### 工商抬头

```bash
bin/cordys-boss crm page contract/business-title [keyword|JSON]
```

### 发票

```bash
bin/cordys-boss crm page invoice [keyword|JSON]
```

## 5. 合同统计接口

### 接口

```http
POST /contract/statistic
```

### 用途

回答：
- 合同整体统计数据（总数、总金额、平均金额等）
- 合同状态分布统计
- 合同趋势数据

### CLI 调用

```bash
bin/cordys-boss crm statistic contract [JSON]
```

### 关键返回字段

- 合同总数、总金额
- 各状态合同统计
- 趋势数据

## 6. 客户维度合同统计

### 接口

```http
GET /account/contract/statistic/{accountId}
```

### 用途

查看某个达人客户的合同统计，回答：
- 该达人累计签了多少合同
- 该达人累计 GMV 多少
- 该达人合同状态分布
- 该达人是否为高价值客户

### CLI 调用

```bash
bin/cordys-boss crm customer-stat contract <accountId>
```

## 7. 客户维度发票统计

### 接口

```http
GET /account/invoice/statistic/{accountId}
```

### 用途

查看某个达人客户的发票统计数据。

### CLI 调用

```bash
bin/cordys-boss crm customer-stat invoice <accountId>
```

## 8. CFO 分析场景

### 投入产出比计算

```
ROI = GMV（合同金额合计）/ 样品成本合计
```

推荐调用组合：
```bash
bin/cordys-boss analysis order-cost           # 获取样品总成本
bin/cordys-boss crm page contract             # 获取合同金额合计
```

### GMV 目标达成

```bash
bin/cordys-boss target team                   # 团队目标（含合同金额目标）
bin/cordys-boss target ranking                # 合同金额排名
bin/cordys-boss crm page contract             # 实际合同数据
```

### 合同结构分析

通过合同列表数据可分析：
- 大单 vs 小单分布
- 商务签约效率排名
- 合同周期分布（短期 vs 长期）
- 客户（达人）合同频率

### 达人维度 GMV 深度分析

```bash
bin/cordys-boss crm customer-stat contract <accountId>    # 达人合同统计
bin/cordys-boss crm customer-stat invoice <accountId>     # 达人发票统计
bin/cordys-boss crm statistic contract                    # 合同整体统计
```

通过客户维度统计可回答：
- 哪些达人贡献了最多 GMV
- 达人合作频次和客单价趋势
- 高价值达人识别与维护建议
