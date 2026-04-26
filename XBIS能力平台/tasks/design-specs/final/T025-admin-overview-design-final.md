# T025 管理端概览页 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AdminPageShell (layout)
├── AdminHeader (layout)
│   ├── Logo
│   ├── NavMenu (管理端导航)
│   └── UserDropdown
├── AdminSidebar (layout)
│   └── MenuItems (概览/能力/任务/用户/计费/系统)
└── MainContent
    ├── PageHeader
    │   └── Title: "管理概览"
    ├── StatsGrid (blocks) - 统计卡片网格
    │   └── StatCard x 4 (business)
    ├── TrendSection (blocks) - 趋势图表区
    │   ├── Tabs (base)
    │   └── TrendChart (business) — 集成 ECharts
    ├── SplitLayout (layout)
    │   ├── RecentJobs (blocks) - 最近任务
    │   │   └── JobTable (base Table)
    │   └── AlertList (blocks) - 告警信息
    │       └── AlertItem (business)
    └── Footer (layout)
```

### 1.2 页面模板
- **使用模板**: `AdminPageShell`（管理端专用模板）
- **布局特点**: 侧边栏导航 + 主内容区，无需移动端适配
- **内容宽度**: 主内容区自适应（≥1200px 全展示）

---

## 2. 组件拆分

### 2.1 base/ 层组件（复用）
| 组件 | 来源 | 用途 |
|------|------|------|
| Card | T002 | 统计卡片容器 |
| Badge | T002 | 告警级别标识 |
| Button | T002 | 操作按钮 |
| Table | T002 | 最近任务表格 |
| Skeleton | T002 | 加载骨架屏 |
| Empty | T002 | 空态展示 |
| Tabs | T002 | 趋势图周期切换 |
| Icon | T002 | 统计图标 |

### 2.2 business/ 层组件（新建/复用）
| 组件 | 说明 | 复杂度 |
|------|------|--------|
| **StatCard** | 统计卡片：数值 + 变化率 + 图标 | 中 |
| **TrendChart** | 趋势图表：支持日/周/月切换，集成 ECharts | 高 |
| **AlertItem** | 告警项：级别 + 消息 + 时间 + 已读操作 | 低 |

### 2.3 blocks/ 层组件（新建）
| 组件 | 说明 | 子组件 |
|------|------|--------|
| **StatsGrid** | 统计网格布局 | StatCard x 4 |
| **TrendSection** | 趋势图区块 | TrendChart + Tabs |
| **RecentJobs** | 最近任务区块 | Table + Pagination |
| **AlertList** | 告警列表区块 | AlertItem list |

### 2.4 layout/ 层组件（复用）
| 组件 | 来源 | 说明 |
|------|------|------|
| AdminPageShell | T004 | 管理端页面骨架 |
| AdminSidebar | T004 | 管理端侧边栏 |
| AdminHeader | T004 | 管理端头部 |

---

## 3. 数据流

### 3.1 页面加载流程
```
[路由进入 /admin/overview]
    │
    ▼
[AdminPageShell 挂载]
    │
    ├──► [并行请求 1] GET /admin-api/v1/overview/stats
    │         └──► OverviewStats
    │
    ├──► [并行请求 2] GET /admin-api/v1/overview/trends?period=daily
    │         └──► TrendData[]
    │
    ├──► [并行请求 3] GET /admin-api/v1/overview/recent-jobs
    │         └──► RecentJob[]
    │
    └──► [并行请求 4] GET /admin-api/v1/overview/alerts
              └──► AlertInfo[]
```

### 3.2 数据更新流
```
[定时器触发]
    │
    ▼
[按策略重新拉取数据]
    │
    ├──► stats / charts ──► 5 分钟间隔
    └──► alerts ──► 1 分钟间隔
    │
    ▼
[对比数据变化]
    │
    ├──► 有变化 ──► 更新 UI（带动画）
    └──► 无变化 ──► 保持现状
```

### 3.3 趋势图切换流
```
[用户点击 Tabs: daily/weekly/monthly]
    │
    ▼
[GET /admin-api/v1/overview/trends?period={tab}]
    │
    ▼
[更新 TrendChart 数据]
```

---

## 4. 状态管理

### 4.1 页面级状态
```typescript
interface OverviewPageState {
  // 页面状态
  pageState: 'loading' | 'idle' | 'error';

  // 加载状态
  statsLoading: boolean;
  trendsLoading: boolean;
  jobsLoading: boolean;
  alertsLoading: boolean;

  // 数据状态
  stats: OverviewStats | null;
  trends: TrendData[];
  recentJobs: RecentJob[];
  alerts: AlertInfo[];

  // UI 状态
  trendPeriod: 'daily' | 'weekly' | 'monthly';
  selectedAlert: string | null;

  // 错误状态
  error: ApiError | null;
}
```

### 4.2 状态机
```
[idle] ──► [loading] ──► [success]
              │
              ▼
           [error] ──► [retry] ──► [loading]
```

### 4.3 各区块独立状态
- StatsGrid: loading → skeleton → data / error
- TrendSection: loading → skeleton → chart / error
- RecentJobs: loading → skeleton → table / empty / error
- AlertList: loading → skeleton → list / empty / error

---

## 5. API 调用

### 5.1 接口清单
| 接口 | 方法 | 路径 | 触发时机 | 参数 |
|------|------|------|----------|------|
| 概览统计 | GET | /admin-api/v1/overview/stats | 页面挂载 | 无 |
| 趋势数据 | GET | /admin-api/v1/overview/trends | 页面挂载 + Tab 切换 | `period: string` |
| 最近任务 | GET | /admin-api/v1/overview/recent-jobs | 页面挂载 | 无 |
| 告警信息 | GET | /admin-api/v1/overview/alerts | 页面挂载 | 无 |
| 告警已读 | POST | /admin-api/v1/overview/alerts/:id/read | 点击单条已读 | `id: string` |
| 告警全读 | POST | /admin-api/v1/overview/alerts/read-all | 点击全部已读 | 无 |

### 5.2 错误处理
```
API 错误
    │
    ├──► 401 ──► 跳转登录页
    ├──► 403 ──► 显示无权限页面
    ├──► 500 ──► 显示错误态 + 重试按钮
    └──► 网络错误 ──► 显示错误态 + 重试按钮
```

### 5.3 轮询机制
```typescript
const refreshStrategies = {
  stats: { interval: 300000, label: '5分钟' },      // 统计数据 5 分钟
  charts: { interval: 300000, label: '5分钟' },      // 图表数据 5 分钟
  alerts: { interval: 60000, label: '1分钟' },       // 告警信息 1 分钟
};
```
- 组件卸载时清除所有定时器

---

## 6. 用户交互流程

### 6.1 主流程
```
管理员进入 /admin/overview
    │
    ▼
页面加载中（Skeleton）
    │
    ▼
展示统计卡片（带动画数字增长）
    │
    ▼
展示趋势图表（默认 daily）
    │
    ▼
展示最近任务表格
    │
    ▼
展示告警列表
```

### 6.2 Tab 切换交互
```
点击「周」Tab
    │
    ▼
TrendChart 显示 loading
    │
    ▼
加载周数据
    │
    ▼
图表平滑过渡到新数据
```

### 6.3 告警交互
```
点击告警项
    │
    ▼
展开告警详情（如有）
    │
    ▼
标记为已读 ──► 调用 POST /alerts/:id/read
    │
    ▼
更新 UI 状态（已读样式）
```

### 6.4 数据导出交互
```
点击导出数据按钮
    │
    ▼
选择导出格式（CSV / PDF）
    │
    ▼
生成并下载文件
```

---

## 7. 边界与异常

### 7.1 空态处理
| 场景 | 展示内容 |
|------|----------|
| 无统计数据 | 显示 "-" 或 "0"，不显示变化率 |
| 无趋势数据 | 显示空图表或提示 "暂无数据" |
| 无最近任务 | Empty 组件 + "暂无最近任务" |
| 无告警 | Empty 组件 + "暂无告警" |

### 7.2 错误态处理
| 场景 | 展示内容 |
|------|----------|
| 统计加载失败 | 卡片显示错误图标 + 重试按钮 |
| 图表加载失败 | 显示错误提示 + 重试按钮 |
| 任务加载失败 | 表格显示错误态 + 重试按钮 |
| 告警加载失败 | 列表显示错误态 + 重试按钮 |

### 7.3 权限处理
- 非管理员访问：跳转 403 页面
- 权限校验：在 AdminPageShell 中统一处理

---

## 8. 性能优化

### 8.1 加载优化
- 4 个 API 请求并行发起
- 各区块独立 loading，不阻塞整体
- Skeleton 占位减少 CLS

### 8.2 渲染优化
- StatCard 数字使用 requestAnimationFrame 动画
- TrendChart 使用 ECharts Canvas 渲染，限制数据点数量（最多 30 个）
- AlertList 虚拟滚动（告警数量 > 50 时）

### 8.3 数据优化
- 趋势数据服务端聚合
- 定时刷新而非实时 WebSocket
- 数据缓存 5 分钟

### 8.4 刷新策略
```typescript
const refreshStrategies = {
  stats: { interval: 300000, label: '5分钟' },
  charts: { interval: 300000, label: '5分钟' },
  alerts: { interval: 60000, label: '1分钟' },
};
```

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 图表性能 | 中 | 数据量大时图表卡顿 | 限制数据点，使用 ECharts Canvas |
| 数据实时性 | 低 | 统计数据可能延迟 | 明确刷新策略（5分钟/1分钟） |
| 权限泄露 | 中 | 管理端数据暴露 | AdminPageShell 统一鉴权 |
| 数字动画性能 | 低 | 大量数字同时动画 | 使用 CSS transform |
| 暗色模式图表 | 低 | 图表颜色需适配暗色 | 使用 Design Token 颜色 |

---

## 10. 开发步骤

### Step 1: 基础准备（0.5 天）
- [ ] 创建 `packages/pages/admin/OverviewPage` 目录
- [ ] 创建页面入口组件
- [ ] 接入 AdminPageShell

### Step 2: 统计卡片（0.5 天）
- [ ] 实现 StatCard 组件
- [ ] 实现 StatsGrid 区块
- [ ] 接入 stats API
- [ ] 实现数字增长动画

### Step 3: 趋势图表（1 天）
- [ ] 引入 ECharts 图表库
- [ ] 实现 TrendChart 组件
- [ ] 实现 TrendSection 区块
- [ ] 接入 trends API
- [ ] 实现 Tab 切换

### Step 4: 最近任务（0.5 天）
- [ ] 实现 RecentJobs 区块
- [ ] 接入 recent-jobs API
- [ ] 实现空态/错误态

### Step 5: 告警列表（0.5 天）
- [ ] 实现 AlertItem 组件（含已读功能）
- [ ] 实现 AlertList 区块
- [ ] 接入 alerts API
- [ ] 接入告警已读/全读 API

### Step 6: 数据导出（0.5 天）
- [ ] 实现数据导出功能（CSV/PDF）

### Step 7: 联调优化（0.5 天）
- [ ] 联调所有 API
- [ ] 性能测试
- [ ] 暗色模式测试
- [ ] 权限测试
