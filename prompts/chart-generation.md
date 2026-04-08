# Python 图表生成规范

## 交付流程

```
cordys-boss CLI 获取数据
       ↓
Python 生成图表 PNG（matplotlib）
       ↓
上传至腾讯云 COS（通过 OpenClaw COS 技能）
       ↓
HTML 报告引用图表 URL
       ↓
HTML 文件上传至 COS
       ↓
Webhook 简报 + HTML 链接推送
```

## 设计系统

### 颜色系统

```python
COLORS = {
    # 主色调
    "primary": "#2563EB",        # 蓝色 - 主要指标
    "primary_light": "#DBEAFE",  # 蓝色浅底
    "secondary": "#7C3AED",      # 紫色 - 次要指标

    # 语义色
    "success": "#059669",        # 绿色 - 健康/达标
    "warning": "#D97706",        # 橙色 - 预警
    "danger": "#DC2626",         # 红色 - 危险/超期
    "info": "#0891B2",           # 青色 - 信息

    # 四角色色
    "ceo": "#1E40AF",            # CEO 深蓝
    "cfo": "#9333EA",            # CFO 紫色
    "coo": "#EA580C",            # COO 橙色
    "cmo": "#059669",            # CMO 绿色

    # 中性色
    "text_primary": "#1E293B",   # 主文字
    "text_secondary": "#64748B", # 次文字
    "border": "#E2E8F0",         # 边框
    "bg_card": "#FFFFFF",        # 卡片背景
    "bg_page": "#F8FAFC",        # 页面背景

    # 图表调色板（用于多系列）
    "palette": [
        "#2563EB", "#7C3AED", "#059669", "#EA580C",
        "#0891B2", "#DC2626", "#D97706", "#6366F1"
    ],

    # 销售层级专用色（从高到低）
    "tier_colors": [
        "#DC2626",  # 300万+   战略客户 红
        "#EA580C",  # 100-300万 核心    橙
        "#D97706",  # 50-100万  高价值  黄
        "#2563EB",  # 20-50万   成长型  蓝
        "#94A3B8",  # <20万     孵化型  灰
    ],

    # 状态色
    "status_colors": {
        "completed": "#059669",
        "in_progress": "#2563EB",
        "shipped": "#0891B2",
        "pending": "#D97706",
        "overdue": "#DC2626",
        "loss": "#9333EA",
    }
}
```

### 字体

```python
import matplotlib
matplotlib.rcParams.update({
    "font.family": ["PingFang SC", "Microsoft YaHei", "SimHei", "sans-serif"],
    "font.size": 12,
    "axes.titlesize": 14,
    "axes.titleweight": "bold",
    "axes.labelsize": 12,
    "xtick.labelsize": 11,
    "ytick.labelsize": 11,
    "legend.fontsize": 11,
    "figure.facecolor": "white",
    "axes.facecolor": "white",
    "axes.edgecolor": "#E2E8F0",
    "axes.grid": True,
    "grid.color": "#F1F5F9",
    "grid.linewidth": 0.8,
})
```

### 尺寸规范

| 图表类型 | 宽(inch) | 高(inch) | DPI | 用途 |
|:---------|:--------:|:--------:|:---:|:-----|
| 标准横向 | 8 | 4.5 | 150 | 条形图、折线图、漏斗图 |
| 标准方形 | 6 | 6 | 150 | 饼图、雷达图 |
| 宽幅 | 10 | 4 | 150 | 热力图、四象限图 |
| 紧凑 | 6 | 3 | 150 | 仪表盘指标卡 |

## 图表类型定义

### 1. 销售层级分布图（水平条形图）

**数据来源**: `customer-portrait` → `salesTierDistribution`

```python
def chart_sales_tier(distribution_items, output_path):
    """
    distribution_items: List[dict] 每项含 name, count, ratio
    从 API 返回的 salesTierDistribution 直接传入
    """
    fig, ax = plt.subplots(figsize=(8, 4.5))

    names = [item["name"] for item in distribution_items]
    counts = [item["count"] for item in distribution_items]
    ratios = [item["ratio"] for item in distribution_items]

    # 按 count 从大到小排序（注意：保持层级顺序，不按数量排）
    colors = COLORS["tier_colors"][:len(names)]

    bars = ax.barh(names[::-1], counts[::-1], color=colors[::-1], height=0.6)

    for bar, count, ratio in zip(bars, counts[::-1], ratios[::-1]):
        ax.text(bar.get_width() + max(counts) * 0.02,
                bar.get_y() + bar.get_height() / 2,
                f"{count}人 ({ratio:.1%})",
                va="center", fontsize=11, color=COLORS["text_primary"])

    ax.set_xlabel("客户数量")
    ax.set_title("客户销售额层级分布")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 2. 工单状态漏斗图（水平条形图）

**数据来源**: `order-overview` → `statusFunnel`

```python
def chart_order_funnel(status_items, output_path):
    """
    status_items: List[dict] 每项含 name, count, ratio
    从 API 返回的 statusFunnel 直接传入
    """
    fig, ax = plt.subplots(figsize=(8, 4.5))

    names = [item["name"] for item in status_items]
    counts = [item["count"] for item in status_items]

    status_color_map = COLORS["status_colors"]
    colors = [status_color_map.get(name, COLORS["primary"]) for name in names]

    bars = ax.barh(names[::-1], counts[::-1], color=colors[::-1], height=0.6)

    for bar, count in zip(bars, counts[::-1]):
        ax.text(bar.get_width() + max(counts) * 0.02,
                bar.get_y() + bar.get_height() / 2,
                f"{count}单",
                va="center", fontsize=11, color=COLORS["text_primary"])

    ax.set_title("工单状态分布")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 3. 商务人效排名图（水平条形双轴）

**数据来源**: `order-cost` → `ownerCostRanking` + `crm statistic contract`

```python
def chart_owner_efficiency(owner_ranking, output_path):
    """
    owner_ranking: List[dict] 每项含 name, value(成本/分), secondaryValue(GMV关联)
    注意：value 单位是分，需要 /100 转元
    """
    fig, ax = plt.subplots(figsize=(8, 4.5))

    names = [item["name"] for item in owner_ranking[:10]]
    costs = [item["value"] / 100 for item in owner_ranking[:10]]  # 分→元

    bars = ax.barh(names[::-1], costs[::-1],
                   color=COLORS["primary"], height=0.6, alpha=0.8)

    for bar, cost in zip(bars, costs[::-1]):
        label = f"{cost/10000:.1f}万" if cost >= 10000 else f"{cost:.0f}元"
        ax.text(bar.get_width() + max(costs) * 0.02,
                bar.get_y() + bar.get_height() / 2,
                label, va="center", fontsize=11)

    ax.set_title("商务样品投入排名")
    ax.set_xlabel("样品成本")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 4. 达人等级分布图（环形图）

**数据来源**: `customer-portrait` → `levelDistribution`

```python
def chart_level_distribution(distribution_items, output_path):
    """
    distribution_items: List[dict] 每项含 name, count, ratio
    """
    fig, ax = plt.subplots(figsize=(6, 6))

    names = [item["name"] for item in distribution_items]
    counts = [item["count"] for item in distribution_items]
    colors = COLORS["palette"][:len(names)]

    wedges, texts, autotexts = ax.pie(
        counts, labels=names, colors=colors,
        autopct=lambda pct: f"{pct:.1f}%\n({int(pct/100*sum(counts))}人)",
        startangle=90, pctdistance=0.75,
        wedgeprops={"width": 0.4, "edgecolor": "white", "linewidth": 2}
    )

    for text in autotexts:
        text.set_fontsize(10)

    ax.set_title("达人等级分布")

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 5. 平台分布图（水平条形图）

**数据来源**: `customer-portrait` → `platformDistribution`

```python
def chart_platform_distribution(distribution_items, output_path):
    """与 chart_sales_tier 结构相同，颜色用 palette"""
    fig, ax = plt.subplots(figsize=(8, 4.5))

    names = [item["name"] for item in distribution_items]
    counts = [item["count"] for item in distribution_items]
    colors = COLORS["palette"][:len(names)]

    bars = ax.barh(names[::-1], counts[::-1], color=colors[::-1], height=0.6)

    for bar, count, item in zip(bars, counts[::-1], distribution_items[::-1]):
        ratio = item["ratio"]
        ax.text(bar.get_width() + max(counts) * 0.02,
                bar.get_y() + bar.get_height() / 2,
                f"{count}人 ({ratio:.1%})",
                va="center", fontsize=11)

    ax.set_title("达人平台分布")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 6. 平台×等级热力图

**数据来源**: `customer-portrait` → `platformLevelMatrix`

```python
def chart_platform_level_heatmap(matrix_items, output_path):
    """
    matrix_items: List[dict] 每项含 row(平台), col(等级), count
    """
    import pandas as pd
    import seaborn as sns

    df = pd.DataFrame(matrix_items)
    pivot = df.pivot(index="row", columns="col", values="count").fillna(0)

    fig, ax = plt.subplots(figsize=(10, 4))
    sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlOrRd",
                linewidths=1, linecolor="white",
                cbar_kws={"label": "达人数量"},
                ax=ax)

    ax.set_title("平台 × 等级 交叉分析")
    ax.set_xlabel("客户等级")
    ax.set_ylabel("平台")

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 7. 成本趋势折线图

**数据来源**: `order-cost` → `costTrend`

```python
def chart_cost_trend(trend_items, output_path):
    """
    trend_items: List[dict] 每项含 period, value（单位：分）
    """
    fig, ax = plt.subplots(figsize=(8, 4.5))

    periods = [item["period"] for item in trend_items]
    values = [item["value"] / 100 / 10000 for item in trend_items]  # 分→万元

    ax.plot(periods, values, color=COLORS["primary"],
            marker="o", linewidth=2, markersize=6)
    ax.fill_between(periods, values, alpha=0.1, color=COLORS["primary"])

    for i, (p, v) in enumerate(zip(periods, values)):
        ax.annotate(f"{v:.1f}万", (p, v),
                    textcoords="offset points", xytext=(0, 10),
                    ha="center", fontsize=10)

    ax.set_title("样品成本趋势")
    ax.set_ylabel("成本（万元）")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)
    plt.xticks(rotation=45)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 8. 达人增长趋势折线图

**数据来源**: `customer-overview` → `growthTrend` + `cumulativeTrend`

```python
def chart_customer_growth(growth_items, cumulative_items, output_path):
    """
    growth_items: 新增趋势 List[dict] {period, value}
    cumulative_items: 累计趋势 List[dict] {period, value}
    """
    fig, ax1 = plt.subplots(figsize=(8, 4.5))

    periods = [item["period"] for item in cumulative_items]
    cumulative = [item["value"] for item in cumulative_items]
    growth = [item["value"] for item in growth_items]

    ax1.bar(periods, growth, color=COLORS["primary_light"],
            width=0.6, label="新增", edgecolor=COLORS["primary"])

    ax2 = ax1.twinx()
    ax2.plot(periods, cumulative, color=COLORS["danger"],
             marker="o", linewidth=2, markersize=5, label="累计")

    ax1.set_ylabel("新增达人数")
    ax2.set_ylabel("累计达人数")
    ax1.set_title("达人增长趋势")

    lines1, labels1 = ax1.get_legend_handles_labels()
    lines2, labels2 = ax2.get_legend_handles_labels()
    ax1.legend(lines1 + lines2, labels1 + labels2, loc="upper left")

    ax1.spines["top"].set_visible(False)
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 9. 转化漏斗图

**数据来源**: `home-stat lead` + `home-stat opportunity` + `home-stat opportunity-success` 组合

```python
def chart_conversion_funnel(funnel_data, output_path):
    """
    funnel_data: List[dict] 每项含 name, value
    示例: [
        {"name": "线索", "value": 150},
        {"name": "客户", "value": 80},
        {"name": "商机", "value": 35},
        {"name": "赢单", "value": 12}
    ]
    """
    fig, ax = plt.subplots(figsize=(8, 4.5))

    names = [item["name"] for item in funnel_data]
    values = [item["value"] for item in funnel_data]
    max_val = max(values) if values else 1

    colors = [COLORS["primary"], COLORS["info"], COLORS["warning"], COLORS["success"]]

    y_pos = range(len(names) - 1, -1, -1)
    bars = ax.barh(y_pos, values, color=colors[:len(names)], height=0.6)

    for bar, name, val in zip(bars, names, values):
        pct = val / max_val * 100
        ax.text(bar.get_width() + max_val * 0.02,
                bar.get_y() + bar.get_height() / 2,
                f"{val} ({pct:.0f}%)",
                va="center", fontsize=11)

    ax.set_yticks(y_pos)
    ax.set_yticklabels(names)
    ax.set_title("转化漏斗")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 10. 商务 ROI 四象限散点图

**数据来源**: `order-cost` → `ownerCostRanking` + 合同数据

```python
def chart_roi_quadrant(owner_data, output_path):
    """
    owner_data: List[dict] 每项含 name, cost(元), gmv(元), roi
    """
    fig, ax = plt.subplots(figsize=(8, 6))

    costs = [d["cost"] / 10000 for d in owner_data]
    gmvs = [d["gmv"] / 10000 for d in owner_data]

    median_cost = sum(costs) / len(costs) if costs else 0
    median_gmv = sum(gmvs) / len(gmvs) if gmvs else 0

    for d, cost, gmv in zip(owner_data, costs, gmvs):
        color = COLORS["success"] if gmv > median_gmv else COLORS["danger"]
        if cost <= median_cost and gmv > median_gmv:
            color = COLORS["info"]  # 金牛
        ax.scatter(cost, gmv, s=120, c=color, edgecolors="white",
                   linewidths=1.5, zorder=5)
        ax.annotate(d["name"], (cost, gmv),
                    textcoords="offset points", xytext=(8, 5),
                    fontsize=10)

    ax.axhline(y=median_gmv, color=COLORS["border"], linestyle="--", linewidth=1)
    ax.axvline(x=median_cost, color=COLORS["border"], linestyle="--", linewidth=1)

    ax.text(median_cost * 0.3, median_gmv * 1.5, "金牛", fontsize=12,
            color=COLORS["text_secondary"], ha="center")
    ax.text(median_cost * 1.7, median_gmv * 1.5, "明星", fontsize=12,
            color=COLORS["text_secondary"], ha="center")
    ax.text(median_cost * 0.3, median_gmv * 0.5, "观察", fontsize=12,
            color=COLORS["text_secondary"], ha="center")
    ax.text(median_cost * 1.7, median_gmv * 0.5, "问题", fontsize=12,
            color=COLORS["text_secondary"], ha="center")

    ax.set_xlabel("样品投入（万元）")
    ax.set_ylabel("GMV 产出（万元）")
    ax.set_title("商务投入产出四象限")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)

    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 11. 工单趋势折线图

**数据来源**: `order-overview` → `orderTrend`

```python
def chart_order_trend(trend_items, output_path):
    """
    trend_items: List[dict] 每项含 period, value
    """
    fig, ax = plt.subplots(figsize=(8, 4.5))

    periods = [item["period"] for item in trend_items]
    values = [item["value"] for item in trend_items]

    ax.plot(periods, values, color=COLORS["coo"],
            marker="o", linewidth=2, markersize=6)
    ax.fill_between(periods, values, alpha=0.1, color=COLORS["coo"])

    ax.set_title("工单量趋势")
    ax.set_ylabel("工单数")
    ax.spines["top"].set_visible(False)
    ax.spines["right"].set_visible(False)
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

### 12. 成本用途分布（环形图）

**数据来源**: `order-cost` → `purposeDistribution`

```python
def chart_cost_purpose(distribution_items, output_path):
    """
    distribution_items: List[dict] 每项含 name, count(金额/分), ratio
    """
    fig, ax = plt.subplots(figsize=(6, 6))

    names = [item["name"] for item in distribution_items]
    values = [item["count"] / 100 for item in distribution_items]  # 分→元
    colors = COLORS["palette"][:len(names)]

    wedges, texts, autotexts = ax.pie(
        values, labels=names, colors=colors,
        autopct=lambda pct: f"{pct:.1f}%",
        startangle=90, pctdistance=0.75,
        wedgeprops={"width": 0.4, "edgecolor": "white", "linewidth": 2}
    )

    ax.set_title("成本用途分布")
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()
```

## 文件命名规范

```
charts/
├── daily-{YYYY-MM-DD}/
│   ├── sales-tier.png          # 销售层级分布
│   ├── order-status.png        # 工单状态
│   ├── owner-cost-top.png      # 商务成本排名
│   ├── level-distribution.png  # 达人等级分布
│   └── platform-distribution.png
├── weekly-{YYYY-MM-DD}/
│   ├── (包含日报所有图表)
│   ├── cost-trend.png          # 成本趋势
│   ├── customer-growth.png     # 达人增长趋势
│   ├── conversion-funnel.png   # 转化漏斗
│   ├── roi-quadrant.png        # ROI四象限
│   ├── order-trend.png         # 工单趋势
│   └── platform-level-heatmap.png
└── monthly-{YYYY-MM-DD}/
    ├── (包含周报所有图表)
    ├── cost-purpose.png        # 成本用途分布
    └── gmv-weekly-trend.png    # GMV周度趋势
```

## 数据单位换算规则

| API 返回字段 | 原始单位 | 展示单位 | 换算 |
|:------------|:---------|:---------|:-----|
| `OrderCostResponse` 中所有金额字段 | **分** | 元/万 | ÷100 |
| `StatCardItem.value` (成本类) | **分** | 元/万 | ÷100 |
| `ContractStatisticResponse.amount` | **元** | 元/万 | 直接用 |
| `DistributionItem.ratio` | 0-1 小数 | 百分比 | ×100 |
| `StatCardItem.changeRate` | 0-1 小数 | 百分比 | ×100 |
| `RankingItem.value` (成本) | **分** | 元/万 | ÷100 |

**金额展示规则**：
- ≥ 10000 元：展示为 `X.X万`（保留一位小数）
- < 10000 元：展示为 `XXXX元`（整数）
- 百分比：保留一位小数（如 `62.5%`）
- 人数/单数：整数，无小数

## 图表质量检查清单

生成图表后必须验证：
- [ ] 所有数据来自 API 返回，无编造
- [ ] 金额单位已正确换算（特别是成本接口的分→元）
- [ ] 中文字体正确显示，无方块
- [ ] 图表标题清晰描述内容
- [ ] 颜色对比度满足可读性要求
- [ ] 导出为 PNG，DPI ≥ 150
- [ ] 文件大小合理（单张 < 500KB）
