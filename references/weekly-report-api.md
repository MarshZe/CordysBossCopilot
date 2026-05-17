# 商务周报 API 参考（Boss Copilot 只读子集）

> 本 Skill 仅暴露查询接口。写操作（保存草稿、提交、重推）不在 Boss Copilot 范围内。

**Base Path**: `/weekly-report`

---

## 只读接口一览

| # | 方法 | 路径 | 说明 |
|---|------|------|------|
| 1 | POST | `/weekly-report/page` | 周报分页列表 |
| 2 | GET | `/weekly-report/detail/{id}` | 周报详情（含明细 + 推送日志） |
| 3 | GET | `/weekly-report/current` | 当前用户指定周周报 |
| 4 | POST | `/weekly-report/dashboard` | 团队周报看板 |
| 5 | GET | `/weekly-report/resolve-week` | 解析周范围（Asia/Shanghai ISO 周） |

---

## 1. 周报分页列表

```
POST /weekly-report/page
```

**请求体** (`WeeklyReportPageRequest`)：

| 字段 | 类型 | 必填 | 说明 |
|------|------|:--:|------|
| current | int | 是 | 当前页，默认 1 |
| pageSize | int | 是 | 每页条数，默认 30 |
| weekKey | String | 否 | ISO 周标识，如 `2026-W20` |
| status | String | 否 | `DRAFT` / `SUBMITTED` |
| pushStatus | String | 否 | `NOT_CONFIGURED` / `PENDING` / `SUCCESS` / `PARTIAL_SUCCESS` / `FAILED` |
| departmentId | String | 否 | 部门 ID |
| userIds | List<String> | 否 | 商务用户 ID 列表 |
| mineOnly | Boolean | 否 | 仅看当前用户，默认 false |

**响应** `Pager<List<WeeklyReportResponse>>`：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | 周报主键 |
| userId | String | 提交人 ID |
| userName | String | 提交人姓名 |
| weekKey | String | ISO 周标识 |
| startDate | Long | 周开始时间戳（毫秒） |
| endDate | Long | 周结束时间戳（毫秒） |
| status | String | `DRAFT` / `SUBMITTED` |
| pushStatus | String | 推送状态 |
| submittedAt | Long | 提交时间 |
| departmentId | String | 部门 ID |
| departmentName | String | 部门名称 |
| pendingDevCount | Integer | 待开发情况条数 |
| connectedNoSampleCount | Integer | 已建联未寄样条数 |
| sampleSentCount | Integer | 已寄样待跟进条数 |
| scheduledServiceCount | Integer | 已排期待服务条数 |
| items | List<WeeklyReportItem> | 明细列表（详情接口更丰富） |
| pushLogs | List<WeeklyReportPushLog> | 推送日志列表 |

---

## 2. 周报详情

```
GET /weekly-report/detail/{id}
```

**路径参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| id | String | 周报主键 |

**响应** `WeeklyReportResponse`（含完整 `items` 明细 + `pushLogs` 推送日志）。

---

## 3. 当前用户指定周周报

```
GET /weekly-report/current?weekKey={weekKey}
```

**查询参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|:--:|------|
| weekKey | String | 否 | ISO 周标识；不传默认本周 |

**响应** `WeeklyReportResponse`；该周无周报时返回 `null`。

---

## 4. 团队周报看板

```
POST /weekly-report/dashboard
```

**请求体** (`WeeklyReportDashboardRequest`，全部可选)：

| 字段 | 类型 | 说明 |
|------|------|------|
| weekKey | String | ISO 周标识；不传默认本周 |
| departmentId | String | 部门 ID |
| userIds | List<String> | 商务用户 ID 列表 |
| status | String | 提交状态过滤 |
| pushStatus | String | 推送状态过滤 |

**响应** (`WeeklyReportDashboardResponse`)：

| 字段 | 类型 | 说明 |
|------|------|------|
| expectedCount | int | 应提交人数 |
| submittedCount | int | 已提交人数 |
| notSubmittedCount | int | 未提交人数 |
| pushFailedCount | int | 推送失败数 |
| pendingDevCount | int | 待开发情况总条数 |
| connectedNoSampleCount | int | 已建联未寄样总条数 |
| sampleSentCount | int | 已寄样待跟进总条数 |
| scheduledServiceCount | int | 已排期待服务总条数 |
| members | List<MemberWeeklyReport> | 人员明细 |

**MemberWeeklyReport**：

| 字段 | 类型 | 说明 |
|------|------|------|
| userId | String | 商务 ID |
| userName | String | 商务姓名 |
| departmentId | String | 部门 ID |
| departmentName | String | 部门名称 |
| reportId | String | 周报 ID（未提交时为空） |
| weekKey | String | ISO 周标识 |
| status | String | 提交状态 |
| pushStatus | String | 推送状态 |
| submittedAt | Long | 提交时间 |
| pendingDevCount | Integer | 待开发条数 |
| connectedNoSampleCount | Integer | 已建联未寄样条数 |
| sampleSentCount | Integer | 已寄样待跟进条数 |
| scheduledServiceCount | Integer | 已排期待服务条数 |

---

## 5. 解析周范围

```
GET /weekly-report/resolve-week?timestamp={timestamp}
```

**查询参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|:--:|------|
| timestamp | Long | 否 | 毫秒时间戳；不传默认当前时间 |

**响应** (`WeeklyReportRangeResponse`)：

| 字段 | 类型 | 说明 |
|------|------|------|
| weekKey | String | ISO 周键，如 `2026-W19` |
| startDate | Long | 周一 00:00:00.000（Asia/Shanghai 毫秒时间戳） |
| endDate | Long | 周日 23:59:59.999（Asia/Shanghai 毫秒时间戳） |

> ⚠️ 前端禁止本地推导 weekKey，统一调用此接口获取 Asia/Shanghai 权威值。

---

## 数据模型

### 板块枚举（ReportSectionEnum）

| 枚举值 | 板块名称 | 目标类型 | 字段映射 |
|--------|---------|:--:|------|
| `PENDING_DEV` | 待开发情况 | `CLUE` | follow_record = 跟进记录，remark = 备注 |
| `CONNECTED_NO_SAMPLE` | 已建联未寄样 | `CUSTOMER` | follow_record = 跟进记录，remark = 备注 |
| `SAMPLE_SENT` | 已寄样待跟进 | `CUSTOMER` | item_detail_1 = 工单详情，follow_record = 跟进记录，remark = 备注 |
| `SCHEDULED_SERVICE` | 已排期待服务 | `CUSTOMER` | item_detail_1 = 跟进信息，item_detail_2 = 工单信息，item_detail_date = 溯源直播日期，remark = 备注 |

### 状态枚举

**周报状态** (`ReportStatusEnum`)：
- `DRAFT` — 草稿
- `SUBMITTED` — 已提交

**推送状态** (`PushStatusEnum`)：
- `NOT_CONFIGURED` — 未配置 Webhook
- `PENDING` — 待推送
- `SUCCESS` — 全部成功
- `PARTIAL_SUCCESS` — 部分成功
- `FAILED` — 全部失败

---

## CLI 调用示例

```bash
# 周报分页列表
bin/cordys-boss weekly-report page
bin/cordys-boss weekly-report page 2026-W20
bin/cordys-boss weekly-report page '{"weekKey":"2026-W20","status":"SUBMITTED","mineOnly":false}'

# 周报详情
bin/cordys-boss weekly-report detail 123456

# 当前用户本周/指定周周报
bin/cordys-boss weekly-report current
bin/cordys-boss weekly-report current 2026-W20

# 团队看板
bin/cordys-boss weekly-report dashboard
bin/cordys-boss weekly-report dashboard '{"weekKey":"2026-W20","departmentId":"dept01"}'

# 解析周范围
bin/cordys-boss weekly-report resolve-week
bin/cordys-boss weekly-report resolve-week 1746470400000
```

---

## 安全边界

**本 Skill 禁止调用的接口**：

| 方法 | 路径 | 原因 |
|------|------|------|
| POST | `/weekly-report/draft` | 写操作：保存草稿 |
| POST | `/weekly-report/submit` | 写操作：提交周报 |
| POST | `/weekly-report/resend/{id}` | 写操作：重新推送 |
| GET | `/weekly-report/latest-follow` | 需要 SUBMIT 权限，仅用于周报填写时的回填场景 |

Boss Copilot 的定位是**经营分析与决策支持**，不是周报填写工具。老板通过周报查询了解团队执行力、推进态势和管理健康度，而非替商务提交周报。
