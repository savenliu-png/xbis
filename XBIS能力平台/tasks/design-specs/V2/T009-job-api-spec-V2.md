# T009 job API 层 — 修正版设计方案（V2）

> 原设计方案: [T009-job-api-spec.md](../T009-job-api-spec.md)
> 评审报告: [T009-job-api-Reviewer.md](../T009-job-api-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 补充筛选参数 | 第 5 节 API 调用 |
| 2 | 明确 JobCreateRequest 结构 | 第 5 节 API 调用 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 补充批量操作 API | 第 5 节 API 调用 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为 API 层定义。

---

### 2. 组件拆分

无组件拆分，本任务为 API 层定义。

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

无状态管理需求。

---

### 5. API 调用（V2 修正）【已修复】

```typescript
// 任务列表筛选参数 【已修复】
interface JobListParams {
  status?: JobStatus;
  abilityId?: string;
  keyword?: string;
  startDate?: string;
  endDate?: string;
  page?: number;
  pageSize?: number;
}

// 任务创建请求 【已修复】
interface JobCreateRequest {
  abilityId: string; // 【已修复：明确包含 abilityId】
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

// 管理端批量操作 API 【新增】
export const adminJobApi = {
  // 批量取消任务
  batchCancel: (ids: string[]) => 
    apiClient.post('/admin-api/v1/jobs/batch-cancel', { ids }),
  
  // 批量重试任务
  batchRetry: (ids: string[]) => 
    apiClient.post('/admin-api/v1/jobs/batch-retry', { ids }),
};
```

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

| 错误码 | 场景 | 处理方式 |
|--------|------|----------|
| 400 | 参数错误 | 返回校验错误详情 |
| 403 | 权限不足 | 返回 403 |
| 404 | 任务不存在 | 返回 404 |
| 409 | 任务状态不允许操作 | 返回 409 |

---

### 8. 性能优化

- API 响应缓存（30 秒）
- 列表分页加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 批量操作性能 | 中 | 大量任务批量操作可能超时 | 限制批量数量（最大 100） |

---

### 10. 开发步骤拆分

#### Step 1: 用户端 API（0.5 人日）
- [ ] list（补充筛选参数）
- [ ] get / create（明确 JobCreateRequest）
- [ ] cancel / getStatus / batchGetStatus

#### Step 2: 管理端批量 API（0.5 人日）【新增】
- [ ] batchCancel / batchRetry

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **筛选参数** | 未定义 | 新增 `JobListParams` |
| **JobCreateRequest** | 未明确 abilityId | 明确包含 `abilityId` 和 `payload` |
| **批量操作** | 未定义 | 新增 `adminJobApi.batchCancel` / `batchRetry` |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（补充筛选参数、明确 JobCreateRequest）
2. 建议修改项已补充（批量操作 API）
3. 风险等级：低
