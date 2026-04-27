# T018 任务状态轮询 — 接口联调检查报告（D1）

> 任务卡: [T018-job-status-polling.md](../T018-job-status-polling.md)
> 设计方案: [T018-job-status-polling-design-final.md](../design-specs/final/T018-job-status-polling-design-final.md)
> 检查日期: 2026-04-26
> 检查人: AI 联调审查官

---

## 🧪 联调结果（D1）

**👉 状态：联调通过**（问题已修复）

---

## 1️⃣ API 契约一致性检查

### 单任务状态查询

| 检查项 | 任务卡定义 | 代码实现 | 结果 |
|--------|-----------|---------|------|
| URL | `GET /api/v1/jobs/:id/status` | `GET ${prefix}/jobs/${id}/status` | ✅ 一致 |
| Method | GET | GET | ✅ 一致 |
| Path 参数 | `:id` | `${encodePathParam(id)}` | ✅ 一致 |
| 权限 | 用户（已登录） | 通过 `apiClient` 自动携带 Token | ✅ 一致 |

### 批量任务状态查询

| 检查项 | 任务卡定义 | 代码实现（修复后） | 结果 |
|--------|-----------|-------------------|------|
| URL | `POST /api/v1/jobs/status` | `POST ${prefix}/jobs/status` | ✅ 一致 |
| Method | POST | POST | ✅ 一致 |
| Body 参数 | `{ jobIds: string[] }` | `{ jobIds: string[] }` | ✅ 一致（已修复） |

---

## 2️⃣ 返回结构匹配检查

### 单任务响应

```json
{
  "success": true,
  "data": {
    "status": "running",
    "updatedAt": "2026-04-24T10:00:05Z"
  }
}
```

| 字段 | 任务卡定义 | 代码类型定义 | 结果 |
|------|-----------|-------------|------|
| `status` | `JobStatus` | `JobStatus` | ✅ 一致 |
| `updatedAt` | `string` | `string` | ✅ 一致 |

### 批量任务响应

```json
{
  "success": true,
  "data": [
    { "jobId": "job-001", "status": "completed", "updatedAt": "..." },
    { "jobId": "job-002", "status": "running", "updatedAt": "..." }
  ]
}
```

| 字段 | 任务卡定义 | 代码类型定义 | 结果 |
|------|-----------|-------------|------|
| `jobId` | `string` | `string` | ✅ 一致 |
| `status` | `JobStatus` | `JobStatus` | ✅ 一致 |
| `updatedAt` | `string` | `string` | ✅ 一致 |

---

## 3️⃣ 数据结构一致性检查

### JobStatus 枚举

```typescript
type JobStatus =
  | 'accepted'
  | 'queued'
  | 'routing'
  | 'running'
  | 'waiting_review'
  | 'completed'
  | 'failed'
  | 'cancelled'
  | 'callback_pending'
  | 'callback_failed';
```

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 状态值完整性 | ✅ | 10 种状态全部定义 |
| 终态判断 | ✅ | `completed/failed/cancelled/callback_failed` |
| 状态转换规则 | ✅ | `validJobStatusTransitions` 已定义 |

---

## 4️⃣ 状态流一致性检查

| 状态 | 任务卡 | 代码实现 | 终态 | 结果 |
|------|--------|---------|------|------|
| `accepted` | ✅ | ✅ | 否 | 一致 |
| `queued` | ✅ | ✅ | 否 | 一致 |
| `routing` | ✅ | ✅ | 否 | 一致 |
| `running` | ✅ | ✅ | 否 | 一致 |
| `waiting_review` | ✅ | ✅ | 否 | 一致 |
| `completed` | ✅ | ✅ | ✅ | 一致 |
| `failed` | ✅ | ✅ | ✅ | 一致 |
| `cancelled` | ✅ | ✅ | ✅ | 一致 |
| `callback_pending` | ✅ | ✅ | 否 | 一致 |
| `callback_failed` | ✅ | ✅ | ✅ | 一致 |

---

## 5️⃣ 错误处理检查

| 检查项 | 结果 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | `try/catch` + 错误计数 |
| 空数据处理 | ✅ | `if (!resultData)` 判断 |
| loading 状态 | ✅ | `isLoading` 状态管理 |
| 超时处理 | ✅ | `maxPollingDuration` 30分钟 |
| 连续失败处理 | ✅ | `maxRetries` 5次后停止 |
| 页面隐藏暂停 | ✅ | Page Visibility API |

---

## 6️⃣ Business Services 层检查

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 是否通过 Service 层 | ✅ | `userApi.jobs.getStatus` / `batchGetStatus` |
| 是否绕过 XBIS | ✅ | 未绕过，使用标准 `apiClient` |
| 是否使用工厂函数 | ✅ | `createJobApi` 统一创建 |

---

## ❌ 问题列表（已修复）

| 类型 | 问题 | 位置 | 影响 | 修复状态 |
|------|------|------|------|----------|
| API参数 | 批量状态查询请求字段名不一致：任务卡定义 `jobIds`，代码实现 `ids` | `services.ts:83` | 高 | ✅ 已修复 |

### 修复详情

**修改前：**
```typescript
batchGetStatus: (ids: string[]) =>
  apiClient.post(`${prefix}/jobs/status`, { ids }),
```

**修改后：**
```typescript
batchGetStatus: (jobIds: string[]) =>
  apiClient.post(`${prefix}/jobs/status`, { jobIds }),
```

---

## 🛠 修改建议

### 前端修改（已完成）

1. ✅ **修复批量状态查询请求字段名**（`ids` → `jobIds`）

### 后端确认项

1. 确认批量状态查询接口 `POST /api/v1/jobs/status` 支持 `jobIds` 参数
2. 确认用户端 `/api/v1/jobs/...` 路由配置正确（或需调整为 `/portal-api/v1/jobs/...`）

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 批量查询字段名不一致 | 已修复 | 修复前高，修复后无风险 |
| 是否影响后续任务 | 否 | T018 为独立基础能力 |
| 是否影响数据模型 | 否 | 仅查询接口，不修改数据 |
| 内存泄漏 | 低 | useEffect 清理定时器 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

所有 Blocking 问题已修复，API 契约一致，可以进入 D2 功能验收。

---

## 📎 附件

- 任务卡: `tasks/T018-job-status-polling.md`
- 设计方案: `tasks/design-specs/final/T018-job-status-polling-design-final.md`
- 代码实现:
  - `packages/shared/src/hooks/usePolling.ts`
  - `packages/shared/src/hooks/useJobPolling.ts`
  - `packages/shared/src/api/services.ts`
