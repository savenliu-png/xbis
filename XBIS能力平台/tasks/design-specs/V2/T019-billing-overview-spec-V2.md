# T019 计费概览页 — 修正版设计方案（V2）

> 原设计方案: [T019-billing-overview-spec.md](../T019-billing-overview-spec.md)
> 评审报告: [T019-billing-overview-Reviewer.md](../T019-billing-overview-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确图表库选择 | 第 2 节组件拆分 |
| 2 | 补充配额警告逻辑 | 第 4 节状态管理 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加用量明细排序/筛选 | 第 2 节组件拆分 |
| 2 | 明确多币种展示方案 | 第 6 节用户交互流程 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

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
        │   └── 警告提示 【已修复】
        ├── UsageChart
        │   └── 用量趋势图 【已修复】
        └── UsageTable
            └── 用量明细 【已修复】
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Card | 信息卡片 |
| Progress | 进度条 |
| Table | 用量表格 |
| Button | 操作按钮 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| BillingCard | 账单卡片 |
| QuotaBar | 配额进度条 |
| UsageChart | 用量图表 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| BillingSummaryCard | 账单摘要 | **新建** |
| QuotaUsagePanel | 配额使用 | **新建** |

---

### 3. 数据流

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

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface BillingOverviewState {
  pageState: 'loading' | 'idle' | 'error';
  overview: BillingOverview | null;
  quotaWarning: 'none' | 'warning' | 'danger'; // 【新增】配额警告状态
  error: ApiError | null;
}

// 【新增】配额警告逻辑
const warningThreshold = 0.8;
const dangerThreshold = 0.95;

function getQuotaWarning(usagePercentage: number): 'none' | 'warning' | 'danger' {
  if (usagePercentage > dangerThreshold) return 'danger';
  if (usagePercentage > warningThreshold) return 'warning';
  return 'none';
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 计费概览 | GET | `/api/v1/billing/overview` | 查询计费概览 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入计费概览]
    │
    ▼
[浏览概览]
    │
    ├── 查看配额使用（含警告提示）
    ├── 查看用量趋势
    ├── 查看用量明细（支持排序/筛选）【已修复】
    └── 点击升级套餐
```

#### 多币种展示方案（V2 新增）【已修复】

```
页面右上角增加币种选择器（CNY/USD）
切换后重新加载数据
默认币种：CNY
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无套餐 | Empty + 引导购买 |
| 配额超限 | 危险警告 + 限制使用 |

---

### 8. 性能优化

- 概览数据缓存（5 分钟）
- 图表懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 图表性能 | 低 | 大数据量图表渲染慢 | 数据聚合 |

---

### 10. 开发步骤拆分

#### Step 1: 概览卡片（0.5 天）
- [ ] BillingSummaryCard

#### Step 2: 配额与图表（1 天）【已修复】
- [ ] QuotaUsagePanel（含警告逻辑）
- [ ] UsageChart（集成 ECharts）

#### Step 3: 明细表格（0.5 天）【已修复】
- [ ] UsageTable（支持排序/筛选）

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **图表库** | 未声明 | 明确使用 ECharts |
| **配额警告** | 未设计 | 新增 80%/95% 阈值警告 |
| **用量明细** | 基础表格 | 支持排序/筛选 |
| **多币种** | 未说明 | 增加币种选择器方案 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（图表库选择、配额警告逻辑）
2. 建议修改项已补充（排序/筛选、多币种）
3. 风险等级：低
