# T019 计费概览页 — 工程级设计方案

> 任务卡: [T019-billing-overview.md](../T019-billing-overview.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 标题: "计费概览"
│   └── 操作按钮: "升级套餐"
├── DetailPageShell
│   ├── StatsGrid (统计卡片网格)
│   │   ├── StatCard: 当前套餐
│   │   ├── StatCard: 配额使用
│   │   ├── StatCard: 本月用量
│   │   └── StatCard: 本月费用
│   ├── Tabs
│   │   ├── TabPane: 用量趋势
│   │   │   └── UsageChart
│   │   └── TabPane: 用量明细
│   │       └── UsageTable
│   └── QuotaBar (配额进度条)
└── (空态/加载态/错误态)
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

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| StatsGrid | 统计网格 | 是 |
| PlanInfo | 套餐信息区 | 是 |
| QuotaUsage | 配额使用区 | 是 |
| UsageStatistics | 用量统计区 | 是 |
| TrendChart | 趋势图区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[页面加载]
  ├── 调用 GET /api/v1/billing/plan ──► 获取当前套餐
  └── 调用 GET /api/v1/billing/usage ──► 获取用量统计

[用户切换 Tab]
  └── 无需重新加载，本地切换

[用户点击升级]
  └── 跳转 /billing/plans
```

---

## 4. 状态管理

### 页面状态

```typescript
interface BillingOverviewState {
  // 数据状态
  plan?: CurrentPlan;
  stats?: UsageStats;
  details: UsageDetail[];
  trends: UsageTrend[];

  // UI 状态
  activeTab: 'trend' | 'detail';
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/billing/plan | 页面加载 | - | 显示错误提示 |
| GET /api/v1/billing/usage | 页面加载 | period | 显示错误提示 |

---

## 6. 用户交互流程

### 主流程

```
用户进入计费概览
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示计费信息
  │   ├── 用户查看套餐信息
  │   ├── 用户查看用量统计
  │   ├── 用户切换 Tab ──► 查看趋势/明细
  │   └── 用户点击升级 ──► 跳转套餐购买
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 无套餐 | 用户未购买套餐 | 显示空态 | 提示购买 |
| 配额不足 | 用量超过 80% | 警告提示 | 提示升级 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 大量用量数据 | 支持分页 |
| 多币种 | 支持货币切换 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 图表渲染 < 500ms
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

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 DetailPageShell 搭建页面框架

### Step 2: 统计区开发（1 人日）
- [ ] StatsGrid 组件
- [ ] PlanInfo 组件
- [ ] QuotaUsage 组件

### Step 3: 图表区开发（1 人日）
- [ ] UsageChart 组件
- [ ] TrendChart 组件

### Step 4: 明细区开发（0.5 人日）
- [ ] UsageTable 组件

### Step 5: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现加载/空态/错误态

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 优化图表渲染
- [ ] 自测
