# T025 管理端概览页 — 前后端联调测试验收报告

## 一、联调范围

| 类型 | 对象 | 是否新增 | 风险等级 | 是否必须联调 |
|------|------|---------|---------|------------|
| 页面入口 | `/admin/overview` | 是 | 🟢 低 | ✅ |
| API | `GET /admin-api/v1/overview/stats` | 是 | 🟡 中 | ✅ |
| API | `GET /admin-api/v1/overview/trends` | 是 | 🟡 中 | ✅ |
| API | `GET /admin-api/v1/overview/recent-jobs` | 是 | 🟡 中 | ✅ |
| API | `GET /admin-api/v1/overview/alerts` | 是 | 🟡 中 | ✅ |
| API | `POST /admin-api/v1/overview/alerts/:id/read` | 是 | 🟢 低 | ✅ |
| API | `POST /admin-api/v1/overview/alerts/read-all` | 是 | 🟢 低 | ✅ |
| 请求参数 | `period: 'daily'\|'weekly'\|'monthly'` | 是 | 🟢 低 | ✅ |
| 响应结构 | `OverviewStats` | 是 | 🟡 中 | ✅ |
| 响应结构 | `AdminTrendData[]` | 是 | 🟡 中 | ✅ |
| 响应结构 | `{ items: RecentJobItem[], total }` | 是 | 🟡 中 | ✅ |
| 响应结构 | `{ items: AlertInfo[], total }` | 是 | 🟡 中 | ✅ |
| 类型定义 | `OverviewStats / AdminTrendData / AlertInfo` | 是 | 🟢 低 | ✅ |
| 状态流 | `JobStatus` 枚举 | 否（复用） | 🟢 低 | ✅ |
| 状态流 | `AlertLevel` 枚举 | 是 | 🟢 低 | ✅ |
| 权限 | `adminAuthMiddleware` | 否（复用） | 🟢 低 | ✅ |
| 降级逻辑 | executors 表不存在时默认值 | 是 | 🟡 中 | ✅ |
| 旧接口兼容 | `GET /admin-api/v1/dashboard/overview` | 否 | 🟢 低 | ✅ |

---

## 二、API契约核对

### 逐接口核对结果

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 状态 |
|-----|--------|---------|---------|------------|------------|------|
| `/overview/stats` | GET | `adminApi.overview.stats()` | `router.get('/overview/stats')` | ✅ 无参数 | ✅ 见下方 | ✅ |
| `/overview/trends` | GET | `adminApi.overview.trends({ period })` | `router.get('/overview/trends')` + `req.query.period` | ✅ | ✅ 见下方 | ✅ |
| `/overview/recent-jobs` | GET | `adminApi.overview.recentJobs()` | `router.get('/overview/recent-jobs')` | ✅ 无参数 | ✅ 见下方 | ✅ |
| `/overview/alerts` | GET | `adminApi.overview.alerts()` | `router.get('/overview/alerts')` | ✅ 无参数 | ✅ 见下方 | ✅ |
| `/overview/alerts/:id/read` | POST | `adminApi.overview.markAlertRead(id)` | `router.post('/overview/alerts/:id/read')` | ✅ path param | ✅ | ✅ |
| `/overview/alerts/read-all` | POST | `adminApi.overview.markAllAlertsRead()` | `router.post('/overview/alerts/read-all')` | ✅ 无参数 | ✅ | ✅ |

### 详细字段核对

#### 2.1 `GET /overview/stats`

| 字段 | 前端类型 | 后端返回类型 | 一致 |
|------|---------|------------|------|
| totalJobs | number | number (from COUNT) | ✅ |
| jobsToday | number | number (from COUNT) | ✅ |
| jobsGrowth | number | number (硬编码 0) | ✅ |
| totalRevenue | number | number (from SUM, rounded) | ✅ |
| revenueToday | number | number (from SUM, rounded) | ✅ |
| revenueGrowth | number | number (硬编码 0) | ✅ |
| successRate | number | number (计算百分比) | ✅ |
| successRateChange | number | number (硬编码 0) | ✅ |
| activeExecutors | number | number (from COUNT, 默认 0) | ✅ |
| degradedExecutors | number | number (from COUNT, 默认 0) | ✅ |
| offlineExecutors | number | number (from COUNT, 默认 0) | ✅ |

**实际响应**：
```json
{"code":0,"data":{"totalJobs":40,"jobsToday":0,"jobsGrowth":0,"totalRevenue":0,"revenueToday":0,"revenueGrowth":0,"successRate":0,"successRateChange":0,"activeExecutors":0,"degradedExecutors":0,"offlineExecutors":0}}
```

**前端提取**：`extractSingle<OverviewStats>(res)` → 从 `response.data` 提取 → ✅ 正确

#### 2.2 `GET /overview/trends`

| 字段 | 前端类型 | 后端返回类型 | 一致 |
|------|---------|------------|------|
| date | string | string (TO_CHAR) | ✅ |
| jobs | number | number (from COUNT) | ✅ |
| revenue | number | number (from SUM, rounded) | ✅ |
| successRate | number | number (计算百分比) | ✅ |

**实际响应**：
```json
{"code":0,"data":[{"date":"2026-04-21","jobs":26,"revenue":0,"successRate":30.77},{"date":"2026-04-22","jobs":14,"revenue":0,"successRate":85.71}]}
```

**前端提取**：`extractData<AdminTrendData>(res)` → 从 `response.data` 提取数组 → ✅ 正确

#### 2.3 `GET /overview/recent-jobs`

| 字段 | 前端类型 | 后端返回类型 | 一致 |
|------|---------|------------|------|
| items[].id | string | String(row.id) | ✅ |
| items[].jobId | string | row.job_id | ✅ |
| items[].abilityName | string | row.ability_name \|\| '未知能力' | ✅ |
| items[].status | JobStatus | normalizeInvocationStatus(row.status) | ✅ |
| items[].createdAt | string | row.created_at | ✅ |
| total | number | result.rows.length | ✅ |

**实际响应**：
```json
{"code":0,"data":{"items":[{"id":"40","jobId":"wf-run-c1b60a60fffe","abilityName":"工作流编排 API","status":"completed","createdAt":"2026-04-22T02:48:43.407Z"},...],"total":10}}
```

**前端提取**：`extractData<RecentJobItem>(res)` → 检测 `response.data.items` → ✅ 正确

#### 2.4 `GET /overview/alerts`

| 字段 | 前端类型 | 后端返回类型 | 一致 |
|------|---------|------------|------|
| items[].id | string | 合成ID (如 `job-failed-1`) | ✅ |
| items[].level | AlertLevel | 'info'\|'warning'\|'critical' | ✅ |
| items[].message | string | 合成消息 | ✅ |
| items[].source | string | 固定值 | ✅ |
| items[].createdAt | string | row.created_at / new Date().toISOString() | ✅ |
| items[].isRead | boolean | boolean | ✅ |
| total | number | alerts.length | ✅ |

**实际响应**（当前无告警数据）：
```json
{"code":0,"data":{"items":[],"total":0}}
```

**前端提取**：`extractData<AlertInfo>(res)` → 检测 `response.data.items` → ✅ 正确

#### 2.5 `POST /overview/alerts/:id/read`

**实际响应**：
```json
{"code":0,"data":{"id":"test-id","isRead":true}}
```

✅ 前端不使用响应数据，仅依赖乐观更新

#### 2.6 `POST /overview/alerts/read-all`

**实际响应**：
```json
{"code":0,"data":{"allRead":true}}
```

✅ 前端不使用响应数据，仅依赖乐观更新

---

## 三、数据与类型核对

| 字段 | 前端定义 | 后端定义 | 问题 | 修复建议 |
|------|---------|---------|------|---------|
| OverviewStats.totalJobs | number | number (COUNT) | 无 | — |
| OverviewStats.successRate | number | number (百分比 0-100) | 无 | — |
| AdminTrendData.date | string | string (TO_CHAR格式) | 无 | — |
| AdminTrendData.successRate | number | number (百分比 0-100) | 无 | — |
| RecentJobItem.status | JobStatus | normalizeInvocationStatus() | 无 | — |
| AlertInfo.level | AlertLevel | 'info'\|'warning'\|'critical' | 无 | — |
| AlertInfo.isRead | boolean | boolean | 无 | — |
| jobsGrowth | number | 硬编码 0 | ⚠️ 后端未实现增长计算 | 后端需实现环比计算 |
| revenueGrowth | number | 硬编码 0 | ⚠️ 后端未实现增长计算 | 后端需实现环比计算 |
| successRateChange | number | 硬编码 0 | ⚠️ 后端未实现变化计算 | 后端需实现环比计算 |

**注**：3个增长/变化字段后端当前硬编码为 0，前端已正确展示但值始终为 0。这是后端待完善项，非 Bug。

---

## 四、主流程联调

| 流程 | 操作 | 预期结果 | 实际结果 | 状态 |
|------|------|---------|---------|------|
| 1. 页面加载 | 访问 `/admin/overview` | 4个API并行请求 | 4个API并行请求 | ✅ |
| 2. 统计卡片 | StatsGrid 渲染 | totalJobs=40, totalRevenue=¥0, successRate=0%, activeExecutors=0/0 | 与预期一致 | ✅ |
| 3. 趋势图表 | UsageChart 渲染 | 2天数据折线图 | 2天数据折线图 | ✅ |
| 4. 周期切换 | 点击"周"Tab | 请求 trends?period=weekly | 请求正确 | ✅ |
| 5. 最近任务 | Table 渲染 | 10条任务记录 | 10条记录，含 jobId/abilityName/status/createdAt | ✅ |
| 6. 告警列表 | AlertList 渲染 | 当前无告警 → Empty | Empty "暂无告警" | ✅ |
| 7. 旧驾驶舱 | 访问 `/admin/dashboard` | 正常返回 | 正常返回 stats/pendingItems/topApis | ✅ |

---

## 五、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 |
|------|---------|---------|---------|
| 无认证访问 | `{"code":401,"message":"未提供认证令牌"}` | apiClient 拦截器处理 401 → 跳转登录页 | ✅ |
| 无效 period 参数 | 返回 daily 默认数据 | 正常渲染（降级为 daily） | ✅ |
| executors 表不存在 | stats 正常返回（executors 默认 0） | StatsGrid 显示 0/0 | ✅ |
| alerts 无数据 | `{"items":[],"total":0}` | Empty "暂无告警" | ✅ |
| recentJobs 无数据 | `{"items":[],"total":0}` | Empty "暂无最近任务" | ✅ |
| trends 无数据 | `[]` | UsageChart 空图表 | ✅ |
| API 500 错误 | error(res, ..., 500) | 各区块独立 error 状态 + 重试按钮 | ✅ |
| 全部 API 失败 | 所有返回 500 | AdminPageShell status="error" | ✅ |
| 告警已读 API 失败 | 500 | 乐观更新回滚（isRead 恢复 false） | ✅ |
| 全部已读 API 失败 | 500 | 乐观更新回滚（所有 isRead 恢复 false） | ✅ |
| 组件卸载后 API 返回 | — | mountedRef 检查，不 setState | ✅ |
| 重复切换周期 | 多次请求 | 每次请求独立，无竞态 | ✅ |

---

## 六、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
|---------|---------|---------|---------|---------|---------|---------|
| BUG-001 | Medium | 后端 `jobsGrowth`/`revenueGrowth`/`successRateChange` 硬编码为 0 | 查看统计卡片变化值 → 始终显示 0 | 后端 | 统计卡片变化指标无意义 | 后端实现环比计算 |
| BUG-002 | Medium | 后端 `markAlertRead`/`markAllAlertsRead` 无状态持久化 | 标记已读 → 刷新页面 → 恢复未读 | 后端 | 告警已读状态不持久 | 后端建立 alert_reads 表 |
| BUG-003 | Low | 后端 trends `period=invalid` 不校验，静默降级为 daily | 请求 `?period=abc` → 返回 daily 数据 | 后端 | 无实际影响 | 可选：返回 400 错误 |

**注**：BUG-001 和 BUG-002 为后端功能完善项，不影响前端展示逻辑正确性。前端已正确处理这些字段的展示和交互。

---

## 七、Bug修复内容

### BUG-001: 增长/变化指标硬编码为 0

**问题原因**：后端 P0 路由中 `jobsGrowth`/`revenueGrowth`/`successRateChange` 未实现环比计算，硬编码为 0。

**后端修改建议**：

需在 `adminP0.ts` 的 `/overview/stats` 路由中新增环比查询：

```sql
-- 昨日任务量
SELECT COUNT(*) AS yesterday_jobs
  FROM platform.invocation_records
 WHERE DATE(created_at AT TIME ZONE 'Asia/Shanghai') = CURRENT_DATE - INTERVAL '1 day';

-- 昨日收入
SELECT COALESCE(SUM(cost_amount), 0) AS yesterday_revenue
  FROM platform.invocation_records
 WHERE status = 'completed'
   AND DATE(created_at AT TIME ZONE 'Asia/Shanghai') = CURRENT_DATE - INTERVAL '1 day';

-- 昨日成功率
SELECT COUNT(*) AS total,
       SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) AS success_count
  FROM platform.invocation_records
 WHERE created_at >= NOW() - INTERVAL '48 hours'
   AND created_at < NOW() - INTERVAL '24 hours';
```

计算公式：
- `jobsGrowth = ((jobsToday - yesterdayJobs) / yesterdayJobs) * 100`
- `revenueGrowth = ((revenueToday - yesterdayRevenue) / yesterdayRevenue) * 100`
- `successRateChange = todaySuccessRate - yesterdaySuccessRate`

**优先级**：P2 — 不阻塞上线，后续迭代完善。

### BUG-002: 告警已读不持久化

**问题原因**：后端 `markAlertRead`/`markAllAlertsRead` 为无状态实现，不存储已读状态。

**后端修改建议**：

1. 新建 `platform.admin_alert_reads` 表：
```sql
CREATE TABLE platform.admin_alert_reads (
  id BIGSERIAL PRIMARY KEY,
  admin_id BIGINT NOT NULL REFERENCES platform.admin_users(id),
  alert_id VARCHAR(255) NOT NULL,
  read_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(admin_id, alert_id)
);
```

2. 修改 `alerts` 路由，查询时 JOIN `admin_alert_reads` 判断 `isRead`。
3. 修改 `markAlertRead`，INSERT 到 `admin_alert_reads`。
4. 修改 `markAllRead`，批量 INSERT。

**优先级**：P2 — 不阻塞上线，后续迭代完善。

### BUG-003: period 参数不校验

**问题原因**：后端 `trends` 路由对 `period` 参数不做校验，无效值静默降级为 daily。

**后端修改建议**（可选）：
```typescript
const validPeriods = ['daily', 'weekly', 'monthly'];
if (!validPeriods.includes(period)) {
  return error(res, '无效的 period 参数', 400, 400);
}
```

**优先级**：P3 — 无实际影响，可选修复。

---

## 八、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| `GET /overview/stats` 真实数据库 | ✅ | 返回 totalJobs=40, 字段完整 |
| `GET /overview/trends?period=daily` | ✅ | 返回 2 天数据 |
| `GET /overview/trends?period=weekly` | ✅ | 正常返回 |
| `GET /overview/trends?period=monthly` | ✅ | 正常返回 |
| `GET /overview/recent-jobs` | ✅ | 返回 10 条记录 |
| `GET /overview/alerts` | ✅ | 返回空数组 |
| `POST /overview/alerts/:id/read` | ✅ | 返回 {id, isRead: true} |
| `POST /overview/alerts/read-all` | ✅ | 返回 {allRead: true} |
| 无认证访问 | ✅ | 返回 401 |
| 旧 Dashboard API | ✅ | 不受影响 |
| TypeScript 编译 | ✅ | 无新增错误 |
| 前端 extractData/extractSingle | ✅ | 正确处理后端响应包裹结构 |
| 乐观更新 + 回滚 | ✅ | 告警已读/全部已读交互正确 |
| mountedRef 防泄漏 | ✅ | 组件卸载后不 setState |

---

## 九、功能验收结论

| 验收项 | 状态 | 说明 |
|--------|------|------|
| 页面正常打开 | ✅ | `/admin/overview` 路由注册，Vite 编译成功 |
| 统计卡片展示 | ✅ | 4个指标（任务量/收入/成功率/执行器） |
| 趋势图表展示 | ✅ | UsageChart + 日/周/月切换 |
| 最近任务列表 | ✅ | Table + TaskStatusBadge |
| 告警信息列表 | ✅ | 三级告警 + 已读/未读 |
| 告警已读交互 | ✅ | 乐观更新 + 失败回滚 |
| loading 状态 | ✅ | 每个区块独立 Skeleton |
| empty 状态 | ✅ | Empty 组件 |
| error 状态 | ✅ | Alert + 重试按钮 |
| API 通过 Business Services | ✅ | adminApi.overview 命名空间 |
| Design Tokens | ✅ | 无硬编码颜色/间距 |
| 不影响现有功能 | ✅ | Dashboard 等不受影响 |
| API 契约一致 | ✅ | 6个接口前后端完全一致 |
| 类型定义一致 | ✅ | TypeScript 类型与后端 DTO 匹配 |
| 增长指标 | ⚠️ | 后端硬编码为 0，需后续实现环比计算 |
| 告警已读持久化 | ⚠️ | 后端无状态实现，需后续建表 |

👉 **Accepted with notes**

---

## 十、是否允许合并

- **是否允许合并**：✅ **YES**
- **是否允许进入下一任务**：✅ **YES**
- **是否需要继续修复**：NO（无 Blocker/High Bug）
- **是否需要后端继续处理**：YES（P2：增长指标环比计算 + 告警已读持久化）
- **是否需要产品确认**：NO

### 后端待完善项（P2，不阻塞上线）

| 项 | 说明 | 优先级 |
|----|------|--------|
| 增长指标计算 | `jobsGrowth`/`revenueGrowth`/`successRateChange` 实现环比 | P2 |
| 告警已读持久化 | 建立 `admin_alert_reads` 表，存储已读状态 | P2 |
| period 参数校验 | 无效值返回 400 而非静默降级 | P3 |
