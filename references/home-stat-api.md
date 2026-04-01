# 首页统计 API 参考

## 业务背景

首页统计接口提供各模块的核心统计数据，用于快速了解经营全貌。
这些接口返回的是聚合后的统计卡片数据，适合用于日报、周报的快速概览。

## 1. 线索统计

### 接口

```http
POST /home/statistic/lead
```

### 用途

回答：
- 线索总数
- 今天/本周/本月新增线索
- 线索来源分布
- 线索转化情况

### 请求体

支持时间筛选 JSON，或空对象 `{}` 使用默认。

## 2. 商机统计

### 接口

```http
POST /home/statistic/opportunity
```

### 用途

回答：
- 商机总数
- 各阶段分布
- 商机金额总计
- 新增商机趋势

## 3. 进行中商机统计

### 接口

```http
POST /home/statistic/opportunity/underway
```

### 用途

回答：
- 当前正在推进的商机有多少
- 进行中商机的金额合计
- 预计成交时间分布

## 4. 赢单商机统计

### 接口

```http
POST /home/statistic/opportunity/success
```

### 用途

回答：
- 赢单数量和金额
- 赢单率
- 赢单客户类型分布
- 赢单趋势

## 5. 部门权限树

### 接口

```http
GET /home/statistic/department/tree
```

### 用途

获取组织架构树，用于：
- 按部门筛选数据
- 了解团队结构
- 权限范围确认

## CMO 典型使用场景

### 转化漏斗构建

```bash
bin/cordys-boss home-stat lead                  # 线索入口
bin/cordys-boss analysis customer-overview      # 客户层
bin/cordys-boss home-stat opportunity           # 商机层
bin/cordys-boss home-stat opportunity-underway  # 推进中
bin/cordys-boss home-stat opportunity-success   # 赢单层
```

### CEO 经营速览

```bash
bin/cordys-boss home-stat lead
bin/cordys-boss home-stat opportunity
bin/cordys-boss home-stat opportunity-success
bin/cordys-boss target home
```
