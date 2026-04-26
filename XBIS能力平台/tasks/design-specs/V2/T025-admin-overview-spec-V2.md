# T025 管理端概览页 — 修正版设计方案（V2）

> 原设计方案: [T025-admin-overview-spec.md](../T025-admin-overview-spec.md)
> 评审报告: [T025-admin-overview-Reviewer.md](../T025-admin-overview-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 确定图表库选择 | 第 2 节组件拆分 |
| 2 | 补充告警已读 API | 第 5 节 API 调用 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确不同数据的刷新策略 | 第 8 节性能优化 |
| 2 | 增加数据导出功能 | 第 6 节用户交互流程 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   └── Title: "管理概览"
    └── PageShell
        ├── StatsCards
        │   └── 统计卡片
        ├── ChartsPanel
        │   └── 图表展示 【已修复】
        ├── AlertsPanel
        │   └── 告警列表 【已修复】
        └── RecentActivities
            └── 最近活动
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Card | 卡片 |
| Badge | 徽章 |
| Button | 操作按钮 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| StatCard | 统计卡片 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| StatsCards | 统计卡片组 | **新建** |
| ChartsPanel | 图表面板 | **新建** — 集成 ECharts |
| AlertsPanel | 告警面板 | **新建** — 含已读功能 |
| RecentActivities | 最近活动 | **新建** |

---

### 3. 数据流

```
[进入 /admin/overview]
    │
    ▼
[并行加载]
    ├── API: GET /admin-api/v1/overview/stats
    ├── API: GET /admin-api/v1/overview/charts
    └── API: GET /admin-api/v1/overview/alerts
    │
    ▼
[渲染概览页]
```

---

### 4. 状态管理

```typescript
interface AdminOverviewState {
  pageState: 'loading' | 'idle' | 'error';
  stats: OverviewStats | null;
  charts: ChartData | null;
  alerts: Alert[];
  error: ApiError | null;
}
```

---

### 5. API 调用（V2 修正）【已修复】

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 统计数据 | GET | `/admin-api/v1/overview/stats` | 查询统计数据 |
| 图表数据 | GET | `/admin-api/v1/overview/charts` | 查询图表数据 |
| 告警列表 | GET | `/admin-api/v1/overview/alerts` | 查询告警列表 |
| 告警已读 | POST | `/admin-api/v1/overview/alerts/:id/read` | 【新增】标记单条已读 |
| 告警全读 | POST | `/admin-api/v1/overview/alerts/read-all` | 【新增】标记全部已读 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入管理概览]
    │
    ▼
[浏览统计卡片]
    │
    ▼
[浏览图表]
    │
    ▼
[浏览告警列表]
    │
    ├── 点击标记已读 【新增】
    └── 点击标记全部已读 【新增】
    │
    ▼
[浏览最近活动]
    │
    ▼
[点击导出数据（可选）] 【新增】
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无告警 | Empty 组件 |
| 加载失败 | ErrorState + 重试 |

---

### 8. 性能优化（V2 修正）【已修复】

```typescript
// 【新增】不同数据的刷新策略
const refreshStrategies = {
  stats: { interval: 300000, label: '5分钟' },      // 统计数据 5 分钟
  charts: { interval: 300000, label: '5分钟' },      // 图表数据 5 分钟
  alerts: { interval: 60000, label: '1分钟' },       // 告警信息 1 分钟
};
```

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 数据实时性 | 低 | 统计数据可能延迟 | 明确刷新策略 |

---

### 10. 开发步骤拆分

#### Step 1: 统计卡片（0.5 天）
- [ ] StatsCards

#### Step 2: 图表与告警（1.5 天）【已修复】
- [ ] ChartsPanel（集成 ECharts）
- [ ] AlertsPanel（含已读 API）

#### Step 3: 导出功能（0.5 天）【已修复】
- [ ] 数据导出（CSV/PDF）

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **图表库** | 未确定 | 明确使用 ECharts |
| **告警已读** | 未设计 API | 新增单条/全部已读 API |
| **刷新策略** | 未明确 | 明确不同数据不同刷新间隔 |
| **数据导出** | 未提及 | 新增导出功能 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（图表库选择、告警已读 API）
2. 建议修改项已补充（刷新策略、导出功能）
3. 风险等级：低
