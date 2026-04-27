# T025 管理端概览页 — 回归测试与验收报告

## 一、影响范围

| 影响类型 | 影响对象 | 风险等级 | 是否需要回归 |
|---------|---------|---------|------------|
| 页面 | `/admin/overview`（新增） | 🟢 低 | ✅ 是 |
| 页面 | `/admin/dashboard`（旧驾驶舱） | 🟢 低 | ✅ 是 |
| 组件 | AlertItem（新增 business） | 🟢 低 | ✅ 是 |
| 组件 | RecentJobs（新增 blocks） | 🟢 低 | ✅ 是 |
| 组件 | AlertList（新增 blocks） | 🟢 低 | ✅ 是 |
| API | `adminApi.overview.*`（新增6个） | 🟡 中 | ✅ 是 |
| API | `adminApi.dashboard.*`（旧接口） | 🟢 低 | ✅ 是 |
| 状态管理 | OverviewPage 内部 useState | 🟢 低 | ✅ 是 |
| 样式 | tokens/index.ts（修复导出） | 🟡 中 | ✅ 是 |
| 样式 | PollingIndicator（修复 radii→radius） | 🟢 低 | ✅ 是 |
| 权限 | Overview 路由使用 isAuthenticated 守卫 | 🟢 低 | ✅ 是 |
| 旧功能路径 | 侧边栏菜单新增"管理概览"项 | 🟢 低 | ✅ 是 |
| 构建 | vite.config.ts（修复别名路径） | 🟡 中 | ✅ 是 |
| 构建 | pages/package.json（补全 exports） | 🟢 低 | ✅ 是 |

---

## 二、回归测试清单

### 2.1 主流程回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 打开 `/admin/overview` 页面 | ✅ | Vite 编译成功，页面正常渲染 |
| StatsGrid 展示4个统计卡片 | ✅ | totalJobs/totalRevenue/successRate/activeExecutors |
| 趋势图表展示 | ✅ | UsageChart 渲染折线图 |
| 趋势 Tab 切换（日/周/月） | ✅ | 重新请求 trends API，period 参数正确传递 |
| 最近任务列表展示 | ✅ | Table + TaskStatusBadge |
| 告警信息列表展示 | ✅ | AlertItem 三级告警 + 已读/未读 |
| 告警点击标记已读 | ✅ | 乐观更新 + 失败回滚 |
| 全部已读按钮 | ✅ | 乐观更新 + 失败回滚 |
| 侧边栏"管理概览"导航 | ✅ | 菜单项在首位，点击跳转正确 |

### 2.2 API / 数据回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| `GET /admin-api/v1/overview/stats` | ✅ | 返回 OverviewStats 结构，字段完整 |
| `GET /admin-api/v1/overview/trends?period=daily` | ✅ | 返回 AdminTrendData[] |
| `GET /admin-api/v1/overview/trends?period=weekly` | ✅ | 返回周维度数据 |
| `GET /admin-api/v1/overview/trends?period=monthly` | ✅ | 返回月维度数据 |
| `GET /admin-api/v1/overview/recent-jobs` | ✅ | 返回 `{ items: RecentJobItem[], total }` |
| `GET /admin-api/v1/overview/alerts` | ✅ | 返回 `{ items: AlertInfo[], total }`，含 isRead |
| `POST /admin-api/v1/overview/alerts/:id/read` | ✅ | 返回 `{ id, isRead: true }` |
| `POST /admin-api/v1/overview/alerts/read-all` | ✅ | 返回 `{ allRead: true }` |
| 旧 `GET /admin-api/v1/dashboard/overview` | ✅ | 不受影响，正常返回 |
| extractData 处理数组响应 | ✅ | 直接返回数组 |
| extractData 处理 `{ items: [] }` 响应 | ✅ | 提取 items |
| extractSingle 处理 `{ data: T }` 响应 | ✅ | 提取 data |

### 2.3 UI / 响应式回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 桌面端 ≥1200px 布局 | ✅ | StatsGrid 4列 + Grid(3fr 2fr) |
| AdminPageShell 页面模板 | ✅ | title/subtitle/breadcrumbs/status 正确 |
| Card 组件 variant="bordered" | ✅ | 趋势图/最近任务/告警均使用 |
| 告警级别视觉区分 | ✅ | critical=红/warning=黄/info=蓝，左侧边框+圆点 |
| Design Tokens 使用 | ✅ | 无硬编码颜色/间距 |
| Button size="sm" | ✅ | 使用组件库规范 size 值 |

### 2.4 状态回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 页面初始 loading | ✅ | AdminPageShell status="loading" |
| StatsGrid loading | ✅ | loading prop 传入 |
| 趋势图 loading | ✅ | Skeleton variant="card" |
| RecentJobs loading | ✅ | Skeleton variant="card" |
| AlertList loading | ✅ | Skeleton variant="card" lines=3 |
| StatsGrid error | ✅ | Alert type="error" + 重试按钮 |
| 趋势图 error | ✅ | 错误提示 + 重试链接 |
| RecentJobs empty | ✅ | Empty "暂无最近任务" |
| AlertList empty | ✅ | Empty "暂无告警" |
| RecentJobs error | ✅ | Alert type="error" + 重试按钮 |
| AlertList error | ✅ | Alert type="error" + 重试按钮 |
| 页面全部失败 | ✅ | AdminPageShell status="error" |
| 告警已读乐观更新 | ✅ | 先更新 UI，失败回滚 |
| 全部已读乐观更新 | ✅ | 先更新 UI，失败回滚 |

### 2.5 旧功能回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| Dashboard 页面正常打开 | ✅ | `/admin/dashboard` 不受影响 |
| Dashboard API 正常调用 | ✅ | `adminApi.dashboard.overview` 正常返回 |
| 侧边栏其他菜单项 | ✅ | 顺序不变，功能不变 |
| 路由守卫 | ✅ | Overview 与其他页面一致使用 isAuthenticated |
| 登录流程 | ✅ | 不受影响 |
| 通知铃铛 | ✅ | 不受影响 |

---

## 三、发现的 Bug

| Bug编号 | 严重级别 | 问题描述 | 复现路径 | 影响范围 | 修复建议 |
|---------|---------|---------|---------|---------|---------|
| BUG-001 | High | `handleMarkAlertRead` 非乐观更新，API 失败时 UI 不变但用户无反馈 | 点击告警项 → API 失败 → UI 不变 | 告警已读功能 | 改为先乐观更新，失败回滚 |
| BUG-002 | High | `handleMarkAllAlertsRead` 使用闭包捕获 `alerts`，可能导致回滚到过时状态 | 点击全部已读 → API 失败 → 回滚到旧状态 | 告警全部已读功能 | 改为函数式 setState 更新 |
| BUG-003 | Medium | Button `size="small"` 不符合组件库规范（应为 `size="sm"`），导致自定义 padding 不生效 | 查看重试按钮 → 尺寸与设计不符 | 4个 T025 文件中的 Button | 统一改为 `size="sm"` |
| BUG-004 | Medium | 后端 `overview/stats` P0 路由 `executors` 表不存在时整体 500 | 启动后端 → 调用 stats API → 500 | 概览页无法加载 | 将 executors 查询单独 try-catch |
| BUG-005 | Medium | 后端 `overview/trends` P0 路由 `INTERVAL $2` SQL 语法错误 | 调用 trends API → 500 | 趋势图无数据 | 改为字符串拼接 |
| BUG-006 | Medium | 后端 `overview/alerts` P0 路由 `executors` 表不存在时整体 500 | 调用 alerts API → 500 | 告警列表无数据 | 将每个查询单独 try-catch |

---

## 四、Bug 修复内容

### BUG-001: handleMarkAlertRead 非乐观更新

**问题原因**：原实现先调用 API，成功后才更新 UI。API 失败时 UI 不变，用户无反馈，且与"全部已读"行为不一致。

**修复方案**：改为先乐观更新 UI，API 失败时回滚。

**修改文件**：`packages/admin/src/pages/OverviewPage.tsx`

**修复代码**：
```tsx
const handleMarkAlertRead = useCallback(async (id: string) => {
  setAlerts((prev) => prev.map((a) => (a.id === id ? { ...a, isRead: true } : a)));
  try {
    await adminApi.overview.markAlertRead(id);
  } catch {
    setAlerts((prev) => prev.map((a) => (a.id === id ? { ...a, isRead: false } : a)));
  }
}, []);
```

### BUG-002: handleMarkAllAlertsRead 闭包过时状态

**问题原因**：原实现使用 `const previousAlerts = alerts` 捕获闭包中的 `alerts`，但 React 状态更新是异步的，`alerts` 可能是过时的值。回滚时 `setAlerts(previousAlerts)` 会覆盖其他已更新的状态。

**修复方案**：改为函数式 setState，失败时将所有 `isRead` 重置为 `false`。

**修改文件**：`packages/admin/src/pages/OverviewPage.tsx`

**修复代码**：
```tsx
const handleMarkAllAlertsRead = useCallback(async () => {
  setAlerts((prev) => prev.map((a) => ({ ...a, isRead: true })));
  try {
    await adminApi.overview.markAllAlertsRead();
  } catch {
    setAlerts((prev) => prev.map((a) => ({ ...a, isRead: false })));
  }
}, []);
```

### BUG-003: Button size="small" 不符合组件库规范

**问题原因**：`ButtonProps.size` 类型定义为 `'sm' | 'md' | 'lg'`，`size="small"` 不在类型中，导致 `sizeMap` 查找返回 `undefined`，自定义 padding 不生效。

**修复方案**：将所有 T025 文件中的 `size="small"` 改为 `size="sm"`。

**修改文件**：
- `packages/admin/src/pages/OverviewPage.tsx`（2处）
- `packages/components/blocks/RecentJobs/index.tsx`（1处）
- `packages/components/blocks/AlertList/index.tsx`（2处）

### BUG-004: 后端 overview/stats executors 表不存在时 500

**问题原因**：`Promise.all` 中包含 `executors` 查询，该表不存在时整个请求失败。

**修复方案**：将 `executors` 查询从 `Promise.all` 中移出，单独 try-catch，失败时返回默认值 0。

**修改文件**：`packages/server/src/routes/adminP0.ts`

### BUG-005: 后端 overview/trends SQL INTERVAL 参数化错误

**问题原因**：PostgreSQL 不支持 `INTERVAL $2` 参数化语法。

**修复方案**：改为字符串拼接 `INTERVAL '${intervalStr}'`。

**修改文件**：`packages/server/src/routes/adminP0.ts`

### BUG-006: 后端 overview/alerts 表不存在时 500

**问题原因**：`Promise.all` 中包含多个可能不存在的表查询，任一失败则整体失败。

**修复方案**：将每个查询单独 try-catch，失败时跳过该类告警。

**修改文件**：`packages/server/src/routes/adminP0.ts`

---

## 五、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| BUG-001 复测：告警点击已读 | ✅ | 乐观更新 → UI 立即变化 → API 失败时回滚 |
| BUG-002 复测：全部已读 | ✅ | 乐观更新 → UI 立即变化 → API 失败时回滚 |
| BUG-003 复测：Button size | ✅ | 所有 Button 使用 `size="sm"` |
| BUG-004 复测：stats API | ✅ | executors 表不存在时返回默认值 0 |
| BUG-005 复测：trends API | ✅ | daily/weekly/monthly 三种周期均正常返回 |
| BUG-006 复测：alerts API | ✅ | 表不存在时返回空数组 |
| 主流程回归 | ✅ | 页面正常加载，4个区块数据正确 |
| 旧功能回归 | ✅ | Dashboard/API/路由不受影响 |
| TypeScript 编译 | ✅ | 无新增错误 |
| Vite 编译 | ✅ | 无新增错误 |

---

## 六、功能验收结论

👉 **Accepted with notes**

### 任务卡验收标准逐项检查

| 验收标准 | 状态 | 说明 |
|---------|------|------|
| 页面展示概览统计（任务量/收入/成功率/执行器） | ✅ | StatsGrid 4列 |
| 趋势图表支持日/周/月切换 | ✅ | Tabs pill 切换 |
| 最近任务列表展示 | ✅ | Table + TaskStatusBadge |
| 告警信息展示（三级告警） | ✅ | critical/warning/info |
| 告警已读/全部已读 | ✅ | 乐观更新 + 失败回滚 |
| loading 状态 | ✅ | 每个区块独立 Skeleton |
| empty 状态 | ✅ | Empty 组件 |
| error 状态 | ✅ | Alert + 重试按钮 |
| 通过 Business Services 层 | ✅ | adminApi.overview |
| 使用 Design Tokens | ✅ | 无硬编码 |
| 不影响现有功能 | ✅ | Dashboard 等不受影响 |

### Notes（备注）

1. **后端数据库表不完整**：`executors` 表不存在时，执行器数据为 0，已做容错处理。需后续创建 `platform.executors` 表。
2. **告警系统为聚合实现**：当前告警数据从 `invocation_records`/`executors`/`manual_review_tasks` 聚合生成，非独立告警表。`markAlertRead`/`markAllAlertsRead` 为无状态实现（不持久化已读状态）。需后续建立 `platform.alerts` + `platform.admin_alert_reads` 表。
3. **Button size="small" 项目级问题**：55 个文件使用 `size="small"` 而非 `size="sm"`，T025 文件已修复，其余需后续统一修复。

---

## 七、是否允许合并

👉 **YES**

- ✅ 所有 Blocker/High Bug 已修复
- ✅ 回归测试全部通过
- ✅ 不影响现有功能
- ✅ TypeScript 编译通过
- ✅ Vite 编译通过
- ✅ 6个 API 端点正常返回数据

**是否允许进入下一任务**：YES

**是否需要后端联调**：部分需要 — 告警已读状态持久化需后端实现 `platform.alerts` 表后联调
