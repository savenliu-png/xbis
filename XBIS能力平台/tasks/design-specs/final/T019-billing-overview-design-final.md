# T019 计费概览页 — 最终可开发设计方案

> 任务卡: [T019-billing-overview.md](../T019-billing-overview.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "计费概览"
    └── DetailPageShell
        ├── BillingSummaryCard
        │   ├── 当前套餐
        │   ├── 剩余配额
        │   └── 有效期
        ├── QuotaUsagePanel
        │   ├── 配额使用进度条
        │   └── 警告提示
        ├── UsageChart
        │   └── 用量趋势图
        └── UsageTable
            └── 用量明细
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Card | 统计卡片 | 否 |
| Progress | 进度条 | 否 |
| Badge | 状态标识 | 否 |
| Tabs | 内容切换 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| BillingCard | 计费卡片 | 否 |
| UsageChart | 用量图表 | 否 |
| QuotaBar | 配额进度条 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| StatsGrid | 统计网格 | 是 |
| BillingSummaryCard | 账单摘要 | 是 |
| QuotaUsagePanel | 配额使用 | 是 |
| UsageStatistics | 用量统计区 | 是 |
| TrendChart | 趋势图区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[进入 /billing]
    │
    ▼
[API: GET /api/v1/billing/overview]
    │
    ▼
[渲染计费概览]
```

---

## 4. 状态管理

```typescript
interface BillingOverviewState {
  pageState: 'loading' | 'idle' | 'error';
  overview: BillingOverview | null;
  quotaWarning: 'none' | 'warning' | 'danger';
  error: ApiError | null;
}

// 配额警告逻辑
const warningThreshold = 0.8;
const dangerThreshold = 0.95;

function getQuotaWarning(usagePercentage: number): 'none' | 'warning' | 'danger' {
  if (usagePercentage > dangerThreshold) return 'danger';
  if (usagePercentage > warningThreshold) return 'warning';
  return 'none';
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 计费概览 | GET | `/api/v1/billing/overview` | 查询计费概览 |
| 当前套餐 | GET | `/api/v1/billing/plan` | 获取当前套餐 |
| 用量统计 | GET | `/api/v1/billing/usage` | 获取用量统计 |

---

## 6. 用户交互流程

```
[进入计费概览]
    │
    ▼
[浏览概览]
    │
    ├── 查看配额使用（含警告提示）
    ├── 查看用量趋势
    ├── 查看用量明细（支持排序/筛选）
    └── 点击升级套餐
```

### 多币种展示方案

```
页面右上角增加币种选择器（CNY/USD）
切换后重新加载数据
默认币种：CNY
```

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 无套餐 | Empty + 引导购买 |
| 配额超限 | 危险警告 + 限制使用 |
| 大量用量数据 | 支持分页 |
| 多币种 | 支持货币切换 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 图表渲染 < 500ms
- 概览数据缓存（5 分钟）
- 图表懒加载
- 无内存泄漏

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 图表性能 | 中 | 大量数据点可能影响性能 | 数据聚合，限制点数 |
| 数据量大 | 中 | 用量数据可能很大 | 分页加载 |
| 多币种 | 低 | 多币种展示复杂 | 统一币种展示 |

---

## 10. 开发步骤拆分

### Step 1: 概览卡片（0.5 天）
- [ ] BillingSummaryCard

### Step 2: 配额与图表（1 天）
- [ ] QuotaUsagePanel（含警告逻辑）
- [ ] UsageChart（集成 ECharts）

### Step 3: 明细表格（0.5 天）
- [ ] UsageTable（支持排序/筛选）

### Step 4: API 集成 & 状态管理（0.5 天）
- [ ] 集成 API 调用
- [ ] 实现加载/空态/错误态

### Step 5: 性能优化 & 测试（0.5 天）
- [ ] 优化图表渲染
- [ ] 自测
