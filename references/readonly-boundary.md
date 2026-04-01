# 只读边界说明

Cordys Boss Copilot 的第一原则是：**只读**。

这不是一个通用 CRM 自动化脚本，而是一个老板专属的经营分析与团队指导工具。

## 允许接入的接口

### 数据分析

- `POST /data-analysis/customer/overview`
- `POST /data-analysis/customer/portrait`
- `POST /data-analysis/order/overview`
- `POST /data-analysis/order/cost`

### 样品工单只读

- `POST /sample-order/page`
- `GET /sample-order/get/{id}`
- `GET /sample-order/recovery/list/{orderId}`
- `GET /sample-order/recovery/pending-items/{orderId}`

### CRM 模块统计（只读）

- `POST /contract/statistic` — 合同统计
- `POST /contract/payment-record/statistic` — 回款记录统计
- `POST /order/statistic` — 订单统计
- `POST /opportunity/statistic` — 商机统计
- `GET /account/contract/statistic/{accountId}` — 客户合同统计
- `GET /account/invoice/statistic/{accountId}` — 客户发票统计
- `GET /account/contract/payment-plan/statistic/{accountId}` — 客户回款计划统计
- `GET /account/contract/payment-record/statistic/{accountId}` — 客户回款记录统计
- `GET /contract/invoice/statistic/{contractId}` — 合同发票统计

### 客户子资源分页（只读）

- `POST /account/opportunity/page` — 客户商机列表
- `POST /account/contract/page` — 客户合同列表
- `POST /account/order/page` — 客户工单列表
- `POST /account/invoice/page` — 客户发票列表
- `POST /account/contract/payment-plan/page` — 客户回款计划列表
- `POST /account/contract/payment-record/page` — 客户回款记录列表

### 模块图表数据（只读）

- `POST /account/chart` — 客户图表
- `POST /lead/chart` — 线索图表
- `POST /opportunity/chart` — 商机图表

### 首页统计（只读）

- `POST /home/statistic/lead` — 线索统计
- `POST /home/statistic/opportunity` — 商机统计
- `POST /home/statistic/opportunity/underway` — 进行中商机统计
- `POST /home/statistic/opportunity/success` — 赢单商机统计
- `GET /home/statistic/department/tree` — 部门权限树

## 明确禁止的接口

以下接口即便后端存在，也不允许在本 Skill 中暴露：

- `POST /sample-order/add`
- `POST /sample-order/update`
- `POST /sample-order/delete/*`
- `POST /sample-order/accept/*`
- `POST /sample-order/reject`
- `POST /sample-order/ship`
- `POST /sample-order/complete/*`
- `POST /sample-order/recovery/chase/*`
- `POST /sample-order/recovery/return-shipped`
- `POST /sample-order/recovery/confirm`
- `POST /sample-order/recovery/mark-lost`

## 为什么必须这么做

### 1. 用户角色不匹配

老板关心的是：
- 风险在哪里
- 资源投入是否合理
- 团队谁需要被提醒

老板不需要通过 AI 直接去发货、签收、催收、改状态。

### 2. 安全边界必须清晰

如果 Skill 允许写操作，会立刻带来：
- 误操作风险
- 审计责任不清
- 数据权限扩大
- 产品目标偏离“经营判断”

### 3. 产品价值会被稀释

一旦开放写操作，AI 很容易退化成“机械执行器”，而不是“经营参谋”。

## CLI 设计要求

- 不提供 `raw` 命令
- 不提供任何写操作子命令
- 所有命令都必须显式落在 allowlist 内
- 默认输出面向老板，不面向执行岗
