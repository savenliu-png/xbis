# T009 job API 层 — 最终可开发设计方案

> 任务卡: [T009-job-api.md](../T009-job-api.md)
> 设计输入: docs/dev-rules.md, docs/component-architecture.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为 **API 层** 建设，无页面结构。输出物为 API 服务定义。

---

## 2. 组件拆分

无 UI 组件。本任务输出为 API 服务定义。

### API 服务定义

| API | Method | Path | 请求类型 | 响应类型 |
|-----|--------|------|----------|----------|
| 任务列表查询 | GET | `/api/v1/jobs` | JobListParams | JobListResponse |
| 任务详情查询 | GET | `/api/v1/jobs/:id` | - | JobDetailResponse |
| 任务创建 | POST | `/api/v1/jobs` | JobCreateRequest | JobCreateResponse |
| 任务取消 | POST | `/api/v1/jobs/:id/cancel` | - | JobCancelResponse |
| 任务重试 | POST | `/api/v1/jobs/:id/retry` | - | JobRetryResponse |
| 任务结果查询 | GET | `/api/v1/jobs/:id/result` | - | JobResultResponse |
| 任务日志查询 | GET | `/api/v1/jobs/:id/logs` | - | JobLogsResponse |

---

## 3. 数据流

```
前端页面 ──► API 服务 ──► Axios ──► 后端服务
```

---

## 4. 状态管理

无状态管理。API 服务为纯函数。

---

## 5. API 调用

### 任务列表筛选参数

```typescript
interface JobListParams {
  status?: JobStatus;
  abilityId?: string;
  keyword?: string;
  startDate?: string;
  endDate?: string;
  page?: number;
  pageSize?: number;
}

// 任务创建请求
interface JobCreateRequest {
  abilityId: string;
  payload: Record<string, unknown>;
  callbackUrl?: string;
}

export const jobApi = {
  // 查询任务列表（带筛选）
  list: (params?: JobListParams) =>
    apiClient.get('/api/v1/jobs', { params }),

  // 查询任务详情
  get: (id: string) =>
    apiClient.get(`/api/v1/jobs/${id}`),

  // 创建任务
  create: (data: JobCreateRequest) =>
    apiClient.post('/api/v1/jobs', data),

  // 取消任务
  cancel: (id: string) =>
    apiClient.post(`/api/v1/jobs/${id}/cancel`),

  // 查询任务状态
  getStatus: (id: string) =>
    apiClient.get(`/api/v1/jobs/${id}/status`),

  // 批量查询任务状态
  batchGetStatus: (ids: string[]) =>
    apiClient.post('/api/v1/jobs/status', { ids }),
};

// 管理端批量操作 API
export const adminJobApi = {
  // 批量取消任务
  batchCancel: (ids: string[]) =>
    apiClient.post('/admin-api/v1/jobs/batch-cancel', { ids }),

  // 批量重试任务
  batchRetry: (ids: string[]) =>
    apiClient.post('/admin-api/v1/jobs/batch-retry', { ids }),
};
```

### 调用位置

- 页面组件的 `useEffect` 中
- 自定义 Hook 中（推荐）
- 事件处理函数中

### 错误处理

- API 错误由 `apiClient` 拦截器统一处理
- 页面层只需捕获错误并更新状态

---

## 6. 用户交互流程

无直接用户交互。API 由页面层调用。

---

## 7. 边界与异常

| 错误码 | 场景 | 处理方式 |
|--------|------|----------|
| 400 | 参数错误 | 返回校验错误详情 |
| 403 | 权限不足 | 返回 403 |
| 404 | 任务不存在 | 返回 404 |
| 409 | 任务状态不允许操作 | 返回 409 |

---

## 8. 性能优化

- API 响应缓存（30 秒）
- 列表分页加载
- 支持字段筛选（减少传输数据量）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 旧接口兼容 | 高 | 新接口需兼容旧 invocation 接口 | 保留旧端点，内部转发 |
| 状态机复杂 | 中 | JobStatus 状态转换需严格校验 | 后端校验状态转换 |
| 并发创建冲突 | 低 | 并发创建可能导致 ID 冲突 | 使用分布式锁 |
| 批量操作性能 | 中 | 大量任务批量操作可能超时 | 限制批量数量（最大 100） |

---

## 10. 开发步骤拆分

### Step 1: 用户端 API（0.5 人日）
- [ ] list（补充筛选参数）
- [ ] get / create（明确 JobCreateRequest）
- [ ] cancel / getStatus / batchGetStatus

### Step 2: 管理端批量 API（0.5 人日）
- [ ] batchCancel / batchRetry

### Step 3: 类型定义（0.5 人日）
- [ ] API 请求类型
- [ ] API 响应类型
- [ ] 错误类型

### Step 4: 文档 & 测试（0.5 人日）
- [ ] API 文档（Swagger）
- [ ] 单元测试
