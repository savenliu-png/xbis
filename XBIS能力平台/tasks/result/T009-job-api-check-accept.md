# T009 job API 层 — 验收文档

> 任务编号：T009
> 任务名称：job API 层
> 执行阶段：C4 → C5 → C6 → C7
> 执行日期：2026-04-25
> 执行结果：通过

---

## 1. 修改文件清单

### 修改文件

| 文件路径 | 修改类型 | 说明 |
|---------|---------|------|
| `packages/shared/src/types/job.ts` | 修改 | 补充 `JobCreateResponse` / `JobResultResponse` / `JobLogsResponse`；统一 `page`/`dateFrom`/`dateTo` 命名；拆分 `status`/`statusIn` |
| `packages/shared/src/api/services.ts` | 修改 | 新增 `userApi.jobs` API 服务（7 个端点） |
| `packages/shared/src/constants/index.ts` | 修改 | 新增 `JOBS` / `JOB_DETAIL` 路由常量 |

---

## 2. 新增组件清单

无 — 本任务为 API 层建设，无 UI 组件。

---

## 3. API变更清单

### 用户端 API (`userApi.jobs`)

| API | Method | Path | 请求类型 | 响应类型 |
|-----|--------|------|----------|----------|
| list | GET | `/api/v1/jobs` | `JobListParams` | `JobListResponse` |
| get | GET | `/api/v1/jobs/:id` | - | `JobDetailResponse` |
| create | POST | `/api/v1/jobs` | `JobCreateRequest` | `JobCreateResponse` |
| cancel | POST | `/api/v1/jobs/:id/cancel` | - | `JobCancelResponse` |
| retry | POST | `/api/v1/jobs/:id/retry` | - | `JobRetryResponse` |
| result | GET | `/api/v1/jobs/:id/result` | - | `JobResultResponse` |
| logs | GET | `/api/v1/jobs/:id/logs` | - | `JobLogsResponse` |

---

## 4. 风险说明

| 风险项 | 评估 | 说明 |
|--------|------|------|
| 是否影响现网 | 否 | 新增 API 服务，不影响现有 API |
| 是否影响数据结构 | 否 | 类型定义已存在，仅补充响应类型 |
| 类型兼容性 | 低 | `pageNo` → `page` 重命名已同步修改 |
| 旧接口兼容 | 低 | 旧 `invocation` API 完整保留 |

---

## 5. 自检结果

### C5 AI 强制自检

| 检查项 | 结果 |
|--------|------|
| 必修复问题 | 1 项已修复（`status` 数组类型拆分为 `status` + `statusIn`） |
| 可优化问题 | 1 项记录（`executionTimeoutMs` 命名与任务卡 `timeout` 不一致，保持现有命名） |

修复内容：
1. `JobListParams.status` 类型：`JobStatus | JobStatus[]` → `JobStatus` + `statusIn?: JobStatus[]`
2. 补充 `JobCreateResponse` / `JobResultResponse` / `JobLogsResponse` 类型

### C6 开发后自检

| 检查项 | 状态 |
|--------|------|
| 组件复用 | ✅ |
| 命名规范 | ✅ |
| API规范 | ✅ |
| 权限兼容 | ✅ |
| 异常处理 | ✅ |
| 是否影响现网 | ✅ |

**结果：Pass**

---

## 6. 验收结果

| 验收项 | 结果 |
|--------|------|
| API 是否能调用 | ✅ |
| 类型是否完整 | ✅ |
| 是否存在报错 | ✅ |
| 是否影响旧功能 | ✅ |
| 路由常量是否新增 | ✅ |

**结果：通过**

---

## 7. 备注

- 旧 `invocation` API 保留兼容，新 `jobs` API 为任务平台专用
- `JobListParams` 中 `status` 用于单状态筛选，`statusIn` 用于多状态筛选
- 所有路径参数均使用 `encodePathParam()` 编码，符合安全规范
- 管理端 batch 操作（batchCancel/batchRetry）作为 V2 迭代
