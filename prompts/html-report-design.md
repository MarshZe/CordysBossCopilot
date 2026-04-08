# HTML 报告设计系统

## 设计原则

1. **移动优先**: 报告主要在手机上查看，所有布局以 375px 宽为基准
2. **零依赖**: 不引入外部 CSS/JS 框架，所有样式内联
3. **快速加载**: 图表以 `<img>` 标签引用 COS 链接，支持懒加载
4. **数据驱动**: 每个数据点旁边都有环比、趋势标注
5. **角色分区**: CEO/CFO/COO/CMO 四个角色各自独立板块

## HTML 基础模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{REPORT_TITLE}} · {{DATE}}</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }

    :root {
      --primary: #2563EB;
      --primary-light: #DBEAFE;
      --success: #059669;
      --success-light: #D1FAE5;
      --warning: #D97706;
      --warning-light: #FEF3C7;
      --danger: #DC2626;
      --danger-light: #FEE2E2;
      --info: #0891B2;
      --ceo: #1E40AF;
      --cfo: #9333EA;
      --coo: #EA580C;
      --cmo: #059669;
      --text-primary: #1E293B;
      --text-secondary: #64748B;
      --text-muted: #94A3B8;
      --border: #E2E8F0;
      --bg-page: #F8FAFC;
      --bg-card: #FFFFFF;
      --radius: 12px;
      --shadow: 0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.04);
      --shadow-md: 0 4px 6px rgba(0,0,0,0.06), 0 2px 4px rgba(0,0,0,0.04);
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "PingFang SC",
                   "Microsoft YaHei", "Segoe UI", sans-serif;
      background: var(--bg-page);
      color: var(--text-primary);
      line-height: 1.6;
      font-size: 15px;
      -webkit-font-smoothing: antialiased;
    }

    .container {
      max-width: 800px;
      margin: 0 auto;
      padding: 16px;
    }

    /* 头部 */
    .header {
      background: linear-gradient(135deg, var(--primary) 0%, #1E40AF 100%);
      color: white;
      padding: 24px 20px;
      border-radius: var(--radius);
      margin-bottom: 16px;
      box-shadow: var(--shadow-md);
    }
    .header h1 {
      font-size: 20px;
      font-weight: 700;
      margin-bottom: 4px;
    }
    .header .subtitle {
      font-size: 14px;
      opacity: 0.85;
    }
    .header .headline {
      margin-top: 16px;
      padding: 12px 16px;
      background: rgba(255,255,255,0.15);
      border-radius: 8px;
      font-size: 15px;
      line-height: 1.5;
      backdrop-filter: blur(4px);
    }

    /* 核心指标卡片网格 */
    .metrics-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      margin-bottom: 16px;
    }
    .metric-card {
      background: var(--bg-card);
      border-radius: var(--radius);
      padding: 16px;
      box-shadow: var(--shadow);
      border-left: 4px solid var(--primary);
    }
    .metric-card .label {
      font-size: 12px;
      color: var(--text-secondary);
      text-transform: uppercase;
      letter-spacing: 0.5px;
    }
    .metric-card .value {
      font-size: 24px;
      font-weight: 700;
      color: var(--text-primary);
      margin: 4px 0;
      font-variant-numeric: tabular-nums;
    }
    .metric-card .change {
      font-size: 13px;
      font-weight: 500;
    }
    .metric-card .change.up { color: var(--success); }
    .metric-card .change.down { color: var(--danger); }
    .metric-card .change.flat { color: var(--text-muted); }

    .metric-card.success { border-left-color: var(--success); }
    .metric-card.warning { border-left-color: var(--warning); }
    .metric-card.danger { border-left-color: var(--danger); }

    /* 角色板块 */
    .role-section {
      background: var(--bg-card);
      border-radius: var(--radius);
      margin-bottom: 16px;
      overflow: hidden;
      box-shadow: var(--shadow);
    }
    .role-header {
      padding: 16px 20px;
      color: white;
      font-size: 16px;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .role-header.ceo { background: var(--ceo); }
    .role-header.cfo { background: var(--cfo); }
    .role-header.coo { background: var(--coo); }
    .role-header.cmo { background: var(--cmo); }

    .role-header .role-icon {
      width: 28px;
      height: 28px;
      background: rgba(255,255,255,0.2);
      border-radius: 6px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 14px;
    }

    .role-body { padding: 20px; }

    /* 图表容器 */
    .chart-block {
      margin-bottom: 20px;
    }
    .chart-block h4 {
      font-size: 14px;
      font-weight: 600;
      color: var(--text-primary);
      margin-bottom: 12px;
      padding-bottom: 8px;
      border-bottom: 1px solid var(--border);
    }
    .chart-block img {
      width: 100%;
      height: auto;
      border-radius: 8px;
      display: block;
    }
    .chart-block .chart-insight {
      margin-top: 12px;
      padding: 12px 16px;
      background: var(--bg-page);
      border-radius: 8px;
      font-size: 14px;
      color: var(--text-secondary);
      line-height: 1.6;
      border-left: 3px solid var(--primary);
    }

    /* 建议卡 */
    .advice-card {
      padding: 16px;
      border-radius: 8px;
      margin-bottom: 12px;
      font-size: 14px;
      line-height: 1.6;
    }
    .advice-card.action {
      background: var(--primary-light);
      border-left: 3px solid var(--primary);
    }
    .advice-card.risk {
      background: var(--danger-light);
      border-left: 3px solid var(--danger);
    }
    .advice-card.opportunity {
      background: var(--success-light);
      border-left: 3px solid var(--success);
    }
    .advice-card .advice-title {
      font-weight: 600;
      margin-bottom: 4px;
      color: var(--text-primary);
    }

    /* 数据表格 */
    .data-table {
      width: 100%;
      border-collapse: collapse;
      font-size: 13px;
      margin: 12px 0;
    }
    .data-table th {
      background: var(--bg-page);
      padding: 10px 12px;
      text-align: left;
      font-weight: 600;
      color: var(--text-secondary);
      font-size: 12px;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      border-bottom: 2px solid var(--border);
    }
    .data-table td {
      padding: 10px 12px;
      border-bottom: 1px solid var(--border);
      font-variant-numeric: tabular-nums;
    }
    .data-table tr:hover { background: var(--bg-page); }
    .data-table .num { text-align: right; }
    .data-table .highlight { font-weight: 600; color: var(--primary); }

    /* 状态标签 */
    .tag {
      display: inline-block;
      padding: 2px 8px;
      border-radius: 4px;
      font-size: 12px;
      font-weight: 500;
    }
    .tag.green { background: var(--success-light); color: var(--success); }
    .tag.yellow { background: var(--warning-light); color: var(--warning); }
    .tag.red { background: var(--danger-light); color: var(--danger); }

    /* 执行摘要 */
    .executive-summary {
      background: var(--bg-card);
      border-radius: var(--radius);
      padding: 20px;
      margin-bottom: 16px;
      box-shadow: var(--shadow);
    }
    .executive-summary h3 {
      font-size: 16px;
      margin-bottom: 12px;
    }
    .summary-item {
      display: flex;
      align-items: flex-start;
      gap: 10px;
      margin-bottom: 10px;
      font-size: 14px;
      line-height: 1.5;
    }
    .summary-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      margin-top: 6px;
      flex-shrink: 0;
    }
    .summary-dot.green { background: var(--success); }
    .summary-dot.yellow { background: var(--warning); }
    .summary-dot.red { background: var(--danger); }

    /* 页脚 */
    .footer {
      text-align: center;
      padding: 20px;
      font-size: 12px;
      color: var(--text-muted);
    }

    /* 分隔线 */
    .divider {
      height: 1px;
      background: var(--border);
      margin: 16px 0;
    }

    @media (min-width: 640px) {
      .container { padding: 24px; }
      .metrics-grid { grid-template-columns: repeat(4, 1fr); }
      .metric-card .value { font-size: 28px; }
    }
  </style>
</head>
<body>
  <div class="container">
    <!-- 头部 -->
    <div class="header">
      <h1>{{REPORT_TITLE}}</h1>
      <div class="subtitle">{{DATE}} · {{REPORT_PERIOD}}</div>
      <div class="headline">{{HEADLINE}}</div>
    </div>

    <!-- 核心指标 -->
    <div class="metrics-grid">
      {{METRIC_CARDS}}
    </div>

    <!-- 执行摘要 -->
    <div class="executive-summary">
      <h3>执行摘要</h3>
      {{SUMMARY_ITEMS}}
    </div>

    <!-- CEO 视角 -->
    <div class="role-section">
      <div class="role-header ceo">
        <div class="role-icon">C</div>
        CEO · 战略全局
      </div>
      <div class="role-body">
        {{CEO_CONTENT}}
      </div>
    </div>

    <!-- CFO 视角 -->
    <div class="role-section">
      <div class="role-header cfo">
        <div class="role-icon">F</div>
        CFO · 财务效率
      </div>
      <div class="role-body">
        {{CFO_CONTENT}}
      </div>
    </div>

    <!-- COO 视角 -->
    <div class="role-section">
      <div class="role-header coo">
        <div class="role-icon">O</div>
        COO · 运营执行
      </div>
      <div class="role-body">
        {{COO_CONTENT}}
      </div>
    </div>

    <!-- CMO 视角 -->
    <div class="role-section">
      <div class="role-header cmo">
        <div class="role-icon">M</div>
        CMO · 市场增长
      </div>
      <div class="role-body">
        {{CMO_CONTENT}}
      </div>
    </div>

    <!-- 页脚 -->
    <div class="footer">
      CordysBossCopilot 高管顾问团 · 数据截至 {{TIMESTAMP}}<br>
      所有数据来源于 CordysCRM 系统
    </div>
  </div>
</body>
</html>
```

## 组件模板

### 指标卡组件

```html
<div class="metric-card {{STATUS_CLASS}}">
  <div class="label">{{LABEL}}</div>
  <div class="value">{{VALUE}}</div>
  <div class="change {{CHANGE_DIR}}">{{CHANGE_TEXT}}</div>
</div>
```

**STATUS_CLASS 规则**：
- 环比增长且为正面指标 → `success`
- 需要关注但不紧急 → `warning`
- 需要立即处理 → `danger`
- 默认 → 无额外类

**CHANGE_DIR 规则**：
- 环比 > 0 → `up`，文本：`↑ +X%`
- 环比 < 0 → `down`，文本：`↓ X%`
- 环比 = 0 → `flat`，文本：`→ 持平`
- 无对比数据 → 不展示 change 行

### 图表+洞察组件

```html
<div class="chart-block">
  <h4>{{CHART_TITLE}}</h4>
  <img src="{{COS_CHART_URL}}" alt="{{CHART_TITLE}}" loading="lazy">
  <div class="chart-insight">{{INSIGHT}}</div>
</div>
```

**INSIGHT 规则**：
- 必须是分析判断，不是数据复述
- 坏的: "A级客户12人，占比15%"
- 好的: "A+B级客户合计35%，较上月改善5个百分点，客户结构在优化中"
- 如有风险，加粗关键词

### 建议卡组件

```html
<div class="advice-card {{TYPE}}">
  <div class="advice-title">{{TITLE}}</div>
  {{CONTENT}}
</div>
```

**TYPE 选择**：
- `action` — 需要执行的行动
- `risk` — 风险预警
- `opportunity` — 发现的机会

### 数据表格组件

```html
<table class="data-table">
  <thead>
    <tr>
      <th>{{COL1}}</th>
      <th class="num">{{COL2}}</th>
      <th class="num">{{COL3}}</th>
      <th>{{COL4}}</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>{{VAL1}}</td>
      <td class="num {{HIGHLIGHT_CLASS}}">{{VAL2}}</td>
      <td class="num">{{VAL3}}</td>
      <td><span class="tag {{TAG_CLASS}}">{{TAG_TEXT}}</span></td>
    </tr>
  </tbody>
</table>
```

## 响应式设计要点

1. **移动端 (< 640px)**:
   - 指标卡 2 列布局
   - 图表宽度 100%
   - 字体不小于 13px
   - 表格允许水平滚动

2. **桌面端 (>= 640px)**:
   - 指标卡 4 列布局
   - 图表保持比例
   - 利用更大空间展示更多数据

## COS 文件结构

```
cos://bucket/cordys-reports/
├── daily/
│   └── {YYYY-MM-DD}/
│       ├── report.html          # HTML 完整报告
│       └── charts/
│           ├── sales-tier.png
│           ├── order-status.png
│           └── ...
├── weekly/
│   └── {YYYY-MM-DD}/
│       ├── report.html
│       └── charts/
│           └── ...
└── monthly/
    └── {YYYY-MM}/
        ├── report.html
        └── charts/
            └── ...
```

## 分析深度要求

### 第一性原理分析法

每个角色板块的分析不能停留在数据罗列层面，必须遵循以下逻辑链：

```
数据事实 → 原因诊断 → 影响评估 → 行动建议
```

**CEO 分析深度**：
- 数据事实：核心经营指标是什么
- 原因诊断：为什么会这样（找到根因，不是表象）
- 影响评估：这对公司3个月后意味着什么
- 行动建议：本周/本月最高优先级的1-2件事

**CFO 分析深度**：
- 数据事实：钱花在哪了，产出了什么
- 原因诊断：ROI 好/差的根因是什么（人效还是渠道问题）
- 影响评估：按当前速率，季度预算是否可控
- 行动建议：哪里加码，哪里收紧，具体到人和金额

**COO 分析深度**：
- 数据事实：工单处理效率、寄样完成率、客户跟进覆盖
- 原因诊断：瓶颈在哪个环节（人力、流程、系统）
- 影响评估：当前执行力能否支撑业务增长目标
- 行动建议：下周必须解决的执行问题

**CMO 分析深度**：
- 数据事实：客户结构、平台分布、增长趋势
- 原因诊断：增长/下滑的驱动因素是什么
- 影响评估：客户资产质量在变好还是变差
- 行动建议：客户开发策略调整方向
