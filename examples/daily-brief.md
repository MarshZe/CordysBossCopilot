# 示例：每日经营速览

## 老板提问

“给我一个今天的经营速览。”

## 推荐调用顺序

1. `bin/cordys-boss analysis customer-overview`
2. `bin/cordys-boss analysis order-overview`
3. `bin/cordys-boss analysis order-cost`
4. 如发现异常，再下钻：
   - `bin/cordys-boss sample-order page '{...}'`

## 期望输出

```text
结论：
- 今天最需要关注的是待回收超期和样品成本集中。

关键事实：
- 达人总数 ...
- 待处理工单 ...
- 本月样品成本 ...

风险提醒：
- ...

建议动作：
- 今天优先追问 ...
```
