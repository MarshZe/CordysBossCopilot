# 样品工单只读 API 参考

本文件只收录老板专属 Copilot 允许接入的样品工单只读接口。

## 接口定位

数据分析接口负责“看盘”，样品工单只读接口负责“下钻”。

也就是：

```text
analysis = 经营总览
sample-order readonly = 具体风险明细
```

## 1. 工单分页列表

### 接口

```http
POST /sample-order/page
```

### 用途

用于查看：
- 当前有哪些工单
- 哪些待处理
- 哪些待回收
- 哪些逾期
- 某个商务名下有哪些工单

### 常见请求体

```json
{
  "current": 1,
  "pageSize": 30,
  "keyword": "",
  "status": "",
  "purpose": ""
}
```

### 关键返回字段

- `data.list`
- `data.total`

工单列表中的老板重点字段：
- `status`
- `purpose`
- `recoveryRequired`
- `recoveryProgress`
- `hasLoss`
- `overdueLevel`
- `totalAmount`

## 2. 工单详情

### 接口

```http
GET /sample-order/get/{id}
```

### 用途

用于回答：
- 某一单现在什么状态
- 明细有哪些
- 是否需要回收
- 总金额多少
- 回收进度如何

### 关键返回字段

- `data.id`
- `data.status`
- `data.purpose`
- `data.recoveryRequired`
- `data.recoveryDeadline`
- `data.items`
- `data.logs`

## 3. 回收流水

### 接口

```http
GET /sample-order/recovery/list/{orderId}
```

### 用途

用于回答：
- 这个工单的回收过程发生了什么
- 谁在什么时间录入了什么回收动作
- 有没有损坏、缺件、异常备注

### 关键返回字段

- `data[]`
  - `productName`
  - `recoveredQuantity`
  - `conditionStatus`
  - `remark`
  - `operatorName`
  - `createTime`

## 4. 待回收明细

### 接口

```http
GET /sample-order/recovery/pending-items/{orderId}
```

### 用途

用于回答：
- 这一单还有哪些产品没回收
- 每个产品还差多少
- 回收风险集中在哪些明细上

### 关键返回字段

- `data[]`
  - `productName`
  - `specification`
  - `quantity`
  - `recoveredQuantity`
  - `pendingQuantity`
  - `recoveryStatus`

## Skill 使用建议

### 推荐问法

- 哪几单超期未回收
- 这单现在什么状态
- 哪些产品还没回收完
- 这单回收过程发生了什么

### 不推荐直接让 AI 做的事

- 帮我发货
- 帮我催收
- 帮我签收
- 帮我标记丢失

这些都属于写操作，不应进入老板专属只读 Copilot。
