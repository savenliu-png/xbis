# T025 管理端概览页 — 接口联调检查报告（D1）

## 🧪 联调结果（D1）

👉 状态：**前端待修 + 后端待修（双方待修）**

---

## 1️⃣ API契约一致性

### 检查结果

| # | 前端调用 | 任务卡定义 | 后端实际 | 一致性 |
|---|---------|-----------|---------|--------|
| 1 | `GET /admin-api/v1/overview/stats` | `GET /admin-api/v1/overview/stats` | ❌ **不存在** | ❌ |
| 2 | `GET /admin-api/v1/overview/trends?period=` | `GET /admin-api/v1/overview/trends` | ❌ **不存在** | ❌ |
| 3 | `GET /admin-api/v1/overview/recent-jobs` | `GET /admin-api/v1/overview/recent-jobs` | ❌ **不存在** | ❌ |
| 4 | `GET /admin-api/v1/overview/alerts` | `GET /admin-api/v1/overview/alerts` | ❌ **不存在** | ❌ |
| 5 | `POST /admin-api/v1/overview/alerts/:id/read` | `POST /admin-api/v1/overview/alerts/:id/read` | ❌ **不存在** | ❌ |
| 6 | `POST /admin-api/v1/overview/alerts/read-all` | `POST /admin-api/v1/overview/alerts/read-all` | ❌ **不存在** | ❌ |

### 详细分析

**后端现状**：后端仅有 `/admin-api/v1/dashboard/overview`（GET），返回的数据结构与 T025 前端期望的完全不同：

- 后端 `dashboard/overview` 返回：`{ stats: { totalUsers, todayActiveUsers, activeTokens, todayCalls, successRate, pendingReviews }, trends: [...], topApis: [...], pendingItems: [...] }`
- 前端 `overview/stats` 期望：`OverviewStats { totalJobs, jobsToday, jobsGrowth, totalRevenue, revenueToday, revenueGrowth, successRate, successRateChange, activeExecutors, degradedExecutors, offlineExecutors }`

**结论**：6 个 API 端点在后端均不存在，需要后端新增实现。

---

## 2️⃣ 返回结构匹配

### 2.1 `GET /admin-api/v1/overview/stats`

| 字段名 | 任务卡类型 | 前端类型 | 后端实际 | 匹配 |
|--------|-----------|---------|---------|------|
| totalJobs | number | number | ❌ 不存在 | ❌ |
| jobsToday | number | number | ❌ 不存在 | ❌ |
| jobsGrowth | number | number | ❌ 不存在 | ❌ |
| totalRevenue | number | number | ❌ 不存在 | ❌ |
| revenueToday | number | number | ❌ 不存在 | ❌ |
| revenueGrowth | number | number | ❌ 不存在 | ❌ |
| successRate | number | number | ❌ 不存在 | ❌ |
| successRateChange | number | number | ❌ 不存在 | ❌ |
| activeExecutors | number | number | ❌ 不存在 | ❌ |
| degradedExecutors | number | number | ❌ 不存在 | ❌ |
| offlineExecutors | number | number | ❌ 不存在 | ❌ |

### 2.2 `GET /admin-api/v1/overview/trends`

| 字段名 | 任务卡类型 | 前端类型 | 后端实际 | 匹配 |
|--------|-----------|---------|---------|------|
| date | string | string | ❌ 不存在 | ❌ |
| jobs | number | number | ❌ 不存在 | ❌ |
| revenue | number | number | ❌ 不存在 | ❌ |
| successRate | number | number | ❌ 不存在 | ❌ |

**前端类型名不一致**：任务卡定义 `TrendData`，前端代码使用 `AdminTrendData`。

### 2.3 `GET /admin-api/v1/overview/recent-jobs`

| 字段名 | 任务卡类型 | 前端类型 | 后端实际 | 匹配 |
|--------|-----------|---------|---------|------|
| id | string | string（RecentJobItem.id） | ❌ 不存在 | ❌ |
| jobId | string | string（RecentJobItem.jobId） | ❌ 不存在 | ❌ |
| abilityName | string | string | ❌ 不存在 | ❌ |
| status | JobStatus | JobStatus | ❌ 不存在 | ❌ |
| createdAt | string | string | ❌ 不存在 | ❌ |

**前端类型名不一致**：任务卡定义 `RecentJob`，前端代码使用 `RecentJobItem`（与 `HomePageData.recentJobs` 中的类型名一致，属于复用已有类型）。

### 2.4 `GET /admin-api/v1/overview/alerts`

| 字段名 | 任务卡类型 | 前端类型 | 后端实际 | 匹配 |
|--------|-----------|---------|---------|------|
| id | string | string | ❌ 不存在 | ❌ |
| level | 'info'\|'warning'\|'critical' | AlertLevel | ❌ 不存在 | ❌ |
| message | string | string | ❌ 不存在 | ❌ |
| source | string | string | ❌ 不存在 | ❌ |
| createdAt | string | string | ❌ 不存在 | ❌ |
| isRead | — (未定义) | boolean? | ❌ 不存在 | ⚠️ |

**问题**：任务卡 `AlertInfo` 未定义 `isRead` 字段，但前端代码添加了 `isRead?: boolean`。这是合理的前端扩展（告警已读功能需要），但需后端同步支持。

---

## 3️⃣ 数据结构一致性

### 3.1 类型命名冲突

| 任务卡定义 | 前端实现 | 冲突情况 |
|-----------|---------|---------|
| `TrendData` | `AdminTrendData` | ⚠️ 命名不一致，但 `AdminTrendData` 更精确（避免与用户端 `UsageTrendPoint` 混淆） |
| `RecentJob` | `RecentJobItem` | ⚠️ 命名不一致，但 `RecentJobItem` 是已有类型（T033 首页复用），合理 |
| `AlertInfo` | `AlertInfo` | ✅ 一致 |
| `OverviewStats` | `OverviewStats` | ✅ 一致 |

### 3.2 结构嵌套检查

| 接口 | 任务卡定义返回结构 | 前端 extractData 处理 | 匹配 |
|------|-----------------|---------------------|------|
| stats | `{ data: OverviewStats }` | `extractSingle` → 支持 `response.data` | ✅ |
| trends | `TrendData[]` | `extractData` → 支持数组 / `{ items: [] }` | ✅ |
| recentJobs | `{ items: RecentJob[], total: number }` | `extractData` → 支持 `{ items: [] }` | ✅ |
| alerts | `{ items: AlertInfo[], total: number }` | `extractData` → 支持 `{ items: [] }` | ✅ |

### 3.3 字段缺失

| 类型 | 缺失字段 | 影响 |
|------|---------|------|
| `OverviewStats` | 无缺失 | — |
| `AdminTrendData` | 无缺失 | — |
| `RecentJobItem` | 无缺失 | — |
| `AlertInfo` | `isRead`（前端新增，任务卡未定义） | 低 — 合理扩展，需后端同步 |

---

## 4️⃣ 状态流一致性

### 4.1 JobStatus 状态枚举

前端 `RecentJobItem.status` 使用 `JobStatus` 类型，完整枚举为：

```
'accepted' | 'queued' | 'routing' | 'running' | 'waiting_review' | 'completed' | 'failed' | 'cancelled' | 'callback_pending' | 'callback_failed'
```

后端 `invocation_records` 表的 `status` 字段值需与 `JobStatus` 枚举完全匹配。

### 4.2 告警级别

前端 `AlertLevel` = `'info' | 'warning' | 'critical'`

需确认后端告警系统是否使用相同的三级分类。常见告警级别可能为 `'low' | 'medium' | 'high'`，与前端定义不一致。

### 4.3 趋势周期参数

前端 `TrendPeriod` = `'daily' | 'weekly' | 'monthly'`

需确认后端 trends 接口是否支持此参数值。

---

## 5️⃣ 错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | 每个区块独立 try/catch，设置 error 状态 |
| 空数据处理 | ✅ | RecentJobs: Empty "暂无最近任务"；AlertList: Empty "暂无告警" |
| loading 处理 | ✅ | 每个区块独立 loading 状态 + Skeleton |
| 超时/失败处理 | ✅ | 各区块独立重试按钮；页面级 Promise.allSettled 容错 |
| 组件卸载保护 | ✅ | mountedRef 防止卸载后 setState |
| 401/403 处理 | ⚠️ | 依赖 apiClient 拦截器统一处理 401，但 403 无专门处理 |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 通过 Service 层调用 | ✅ | 所有 API 调用通过 `adminApi.overview` 命名空间 |
| 是否绕过 XBIS 层 | ✅ | 未绕过，全部走 `apiClient` |
| Service 层定义完整性 | ✅ | 6 个方法均已定义 |

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| **API端点** | `/admin-api/v1/overview/stats` 后端不存在 | services.ts:389, 后端路由 | 🔴 阻塞 — 页面无法获取统计数据 |
| **API端点** | `/admin-api/v1/overview/trends` 后端不存在 | services.ts:391, 后端路由 | 🔴 阻塞 — 趋势图表无数据 |
| **API端点** | `/admin-api/v1/overview/recent-jobs` 后端不存在 | services.ts:393, 后端路由 | 🔴 阻塞 — 最近任务列表无数据 |
| **API端点** | `/admin-api/v1/overview/alerts` 后端不存在 | services.ts:395, 后端路由 | 🔴 阻塞 — 告警信息无数据 |
| **API端点** | `POST /admin-api/v1/overview/alerts/:id/read` 后端不存在 | services.ts:397, 后端路由 | 🟡 非阻塞 — 告警已读功能不可用 |
| **API端点** | `POST /admin-api/v1/overview/alerts/read-all` 后端不存在 | services.ts:399, 后端路由 | 🟡 非阻塞 — 全部已读功能不可用 |
| **数据结构** | `AlertInfo.isRead` 前端新增字段，任务卡未定义 | types/index.ts:1413 | 🟡 需后端同步返回 |
| **数据结构** | `TrendData` vs `AdminTrendData` 命名不一致 | types/index.ts:1398 | 🟢 低 — 前端命名更精确 |
| **数据结构** | `RecentJob` vs `RecentJobItem` 命名不一致 | types/index.ts:1239 | 🟢 低 — 复用已有类型 |
| **状态流** | 告警级别 `AlertLevel` 需后端确认一致 | types/index.ts:1405 | 🟡 需确认 |
| **错误处理** | 403 无专门处理（跳转 403 页面） | OverviewPage.tsx | 🟢 低 — 依赖 AdminPageShell |

---

## 🛠 修改建议

### 前端修改

**无需修改** — 前端实现与任务卡定义一致，API 调用路径、参数、数据结构均正确。前端已做好对接准备，只等后端实现。

### 后端修改（必须）

需要在 `packages/server/src/routes/adminP0.ts` 中新增 6 个路由：

```typescript
// 1. GET /admin-api/v1/overview/stats
router.get('/overview/stats', adminAuthMiddleware, async (_req, res) => {
  // 返回 OverviewStats 结构
  // { totalJobs, jobsToday, jobsGrowth, totalRevenue, revenueToday, revenueGrowth,
  //   successRate, successRateChange, activeExecutors, degradedExecutors, offlineExecutors }
});

// 2. GET /admin-api/v1/overview/trends
router.get('/overview/trends', adminAuthMiddleware, async (req, res) => {
  // 接受 query 参数 period: 'daily' | 'weekly' | 'monthly'
  // 返回 AdminTrendData[] 结构
  // [{ date, jobs, revenue, successRate }]
});

// 3. GET /admin-api/v1/overview/recent-jobs
router.get('/overview/recent-jobs', adminAuthMiddleware, async (_req, res) => {
  // 返回 { items: RecentJobItem[], total: number }
  // RecentJobItem: { id, jobId, abilityName, status: JobStatus, createdAt }
});

// 4. GET /admin-api/v1/overview/alerts
router.get('/overview/alerts', adminAuthMiddleware, async (_req, res) => {
  // 返回 { items: AlertInfo[], total: number }
  // AlertInfo: { id, level: 'info'|'warning'|'critical', message, source, createdAt, isRead }
});

// 5. POST /admin-api/v1/overview/alerts/:id/read
router.post('/overview/alerts/:id/read', adminAuthMiddleware, async (req, res) => {
  // 标记单条告警已读
});

// 6. POST /admin-api/v1/overview/alerts/read-all
router.post('/overview/alerts/read-all', adminAuthMiddleware, async (_req, res) => {
  // 标记全部告警已读
});
```

### 数据结构调整

| 调整项 | 说明 |
|--------|------|
| `AlertInfo` 增加 `isRead` 字段 | 后端 alerts 查询需返回 `isRead` 布尔值 |
| 告警级别统一为 `info/warning/critical` | 后端告警系统需使用此三级分类 |
| trends 接口支持 `period` 参数 | 后端需支持 daily/weekly/monthly 聚合 |

---

## ⚠️ 风险评估

| 风险项 | 级别 | 说明 |
|--------|------|------|
| 后端 6 个 API 未实现 | 🔴 高 | 页面完全无法加载数据，阻塞联调 |
| 告警系统可能不存在 | 🟡 中 | 后端可能没有告警数据表，需新建 |
| executor 健康状态查询 | 🟡 中 | `activeExecutors/degradedExecutors/offlineExecutors` 需从 executor 健康检查聚合 |
| 与 dashboard/overview 数据重叠 | 🟢 低 | 现有 `dashboard/overview` 返回不同结构，两者可共存 |

### 是否影响后续任务

- **T026 管理端能力列表** — 不受影响
- **T028 管理端执行器管理** — `overview/stats` 中的 executor 健康数据与 T028 有数据依赖
- **T029 管理端任务操作** — `overview/recent-jobs` 与 T029 有数据依赖

### 是否影响数据模型

- 需新增 `platform.alerts` 表（如不存在）
- 需新增 `platform.admin_alert_reads` 表（记录管理员已读状态）

---

## ✅ 是否允许进入验收（D2）

👉 **NO**

**原因**：后端 6 个 API 端点均未实现，前端页面无法获取任何数据。需后端完成 API 实现后方可进入 D2 验收。

**前置条件**：
1. 后端实现 6 个 overview API 端点
2. 后端确认 `AlertLevel` 枚举与前端一致
3. 后端确认 `JobStatus` 枚举与前端一致
4. 后端确认 `AlertInfo.isRead` 字段返回
