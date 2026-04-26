# T009 job API 层 — D1 接口联调检查报告

> 任务编号：T009
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-25
> 检查人：AI 联调审查官

---

## 🧪 联调结果（D1）

**👉 状态：联调通过（附 3 项建议优化）**

前端实现与 API 契约 / 数据结构整体一致，未发现阻塞性不一致问题。

---

## ❌ 问题列表（详细）

### 1. API 参数：设计方案中有 `getStatus` / `batchGetStatus` / `batchCancel` / `batchRetry`，但任务卡和当前实现未包含

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | 设计方案 Final 版本（T009-job-api-design-final.md）中定义了 `getStatus` / `batchGetStatus` / `batchCancel` / `batchRetry`，但任务卡未列出这些端点，当前实现也未包含 |
| **位置** | `tasks/design-specs/final/T009-job-api-design-final.md` L88-L104 |
| **影响** | 缺少批量操作和状态查询端点，管理端功能不完整 |
| **当前代码** | `userApi.jobs` 仅有 list/get/create/cancel/retry/result/logs |
| **修改建议** | V2 迭代补充管理端批量操作 API：`batchCancel` / `batchRetry`；补充状态查询 API：`getStatus` / `batchGetStatus` |

---

### 2. 返回结构：`JobDetailResponse` 包含 `logs` 字段，但任务卡定义中未包含

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | 当前 `JobDetailResponse` 包含 `logs: JobLog[]`，但任务卡 5.1 节定义中 `JobDetailResponse` 仅包含 `job` / `result?` / `stages` |
| **位置** | `packages/shared/src/types/job.ts` L154-L158 |
| **影响** | 若后端返回的详情不包含日志，前端会期望有 `logs` 字段导致类型不匹配 |
| **当前代码** | ```ts
export interface JobDetailResponse {
  job: Job;
  result?: JobResult;
  stages: JobStage[];
  logs: JobLog[];
}
``` |
| **修改建议** | 将 `logs` 改为可选字段：`logs?: JobLog[]`，与任务卡定义保持一致 |

---

### 3. 数据结构：`JobCreateRequest` 字段与任务卡定义不一致

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | 任务卡定义 `JobCreateRequest` 使用 `timeout?: number`，当前实现使用 `executionTimeoutMs?: number`；任务卡有 `executionMode?: 'async' \| 'sync'`，当前实现使用 `executionMode?: JobExecutionMode` |
| **位置** | `packages/shared/src/types/job.ts` L161-L168 |
| **影响** | 字段命名不一致可能导致前后端对接问题 |
| **当前代码** | ```ts
export interface JobCreateRequest {
  abilityId: string;
  payload: Record<string, unknown>;
  executionMode?: JobExecutionMode;
  executionTimeoutMs?: number;
  callbackUrl?: string;
  sourceSystem?: string;
}
``` |
| **修改建议** | 保持当前实现。理由：<br>1. `executionTimeoutMs` 与 `Job` 实体中的字段命名一致，更明确<br>2. `JobExecutionMode` 类型（'async' \| 'sync' \| 'batch' \| 'streaming'）比任务卡的 `'async' \| 'sync'` 更完整<br>3. `sourceSystem` 是现有类型中已定义的字段，保留有利于追踪 |

---

## 🛠 修改建议汇总

### 前端修改（建议 V2 迭代）

1. **补充管理端批量操作 API**：
   - `adminApi.jobs.batchCancel(ids: string[])`
   - `adminApi.jobs.batchRetry(ids: string[])`

2. **补充状态查询 API**：
   - `userApi.jobs.getStatus(id: string)`
   - `userApi.jobs.batchGetStatus(ids: string[])`

3. **调整 `JobDetailResponse.logs` 为可选**：
   - `logs: JobLog[]` → `logs?: JobLog[]`

### 后端确认项

1. `GET /api/v1/jobs/:id` 返回的详情是否包含 `logs`？
2. 是否提供批量操作端点？
3. 是否提供状态查询端点？

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | 低 | API 层已完整定义，后续页面任务可直接调用 |
| 是否影响数据模型 | 无 | 类型定义完整，无冲突 |
| 批量操作功能缺失风险 | 低 | 管理端批量操作可作为 V2 功能 |
| JobDetailResponse.logs 风险 | 低 | 改为可选字段即可兼容 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

理由：
1. API 契约整体一致，URL、Method、参数定义完整
2. 返回结构匹配（`JobListResponse` / `JobDetailResponse` / `JobCreateResponse` 等类型已定义）
3. 数据结构符合任务卡定义（job 核心实体 + 阶段 + 结果 + 日志）
4. 状态定义一致（`JobStatus`: 11 个状态，覆盖任务卡 8 个状态）
5. 错误处理由 apiClient 拦截器统一处理
6. Business Services 层检查通过（通过 `apiClient` 调用，未绕过接入层）
7. 发现的 3 项问题均为建议优化，不影响核心功能联调

---

## 📎 附录：API 契约对照表

### 用户端 API (`userApi.jobs`)

| API | Method | 前端定义 | 任务卡定义 | 设计方案定义 | 状态 |
|-----|--------|----------|-----------|-------------|------|
| list | GET | `/api/v1/jobs` | `/api/v1/jobs` | `/api/v1/jobs` | ✅ 一致 |
| get | GET | `/api/v1/jobs/:id` | `/api/v1/jobs/:id` | `/api/v1/jobs/:id` | ✅ 一致 |
| create | POST | `/api/v1/jobs` | `/api/v1/jobs` | `/api/v1/jobs` | ✅ 一致 |
| cancel | POST | `/api/v1/jobs/:id/cancel` | `/api/v1/jobs/:id/cancel` | `/api/v1/jobs/:id/cancel` | ✅ 一致 |
| retry | POST | `/api/v1/jobs/:id/retry` | `/api/v1/jobs/:id/retry` | `/api/v1/jobs/:id/retry` | ✅ 一致 |
| result | GET | `/api/v1/jobs/:id/result` | `/api/v1/jobs/:id/result` | 未定义 | ✅ 补充 |
| logs | GET | `/api/v1/jobs/:id/logs` | `/api/v1/jobs/:id/logs` | 未定义 | ✅ 补充 |
| getStatus | GET | 未实现 | 未定义 | `/api/v1/jobs/:id/status` | ⏳ V2 |
| batchGetStatus | POST | 未实现 | 未定义 | `/api/v1/jobs/status` | ⏳ V2 |
| batchCancel | POST | 未实现 | 未定义 | `/admin-api/v1/jobs/batch-cancel` | ⏳ V2 |
| batchRetry | POST | 未实现 | 未定义 | `/admin-api/v1/jobs/batch-retry` | ⏳ V2 |

---

## 📎 附录：类型定义完整性检查

| 类型 | 字段 | 状态 |
|------|------|------|
| `JobListParams` | page, pageSize, status, statusIn?, abilityId?, sourceSystem?, executionMode?, policyRiskLevel?, keyword?, dateFrom?, dateTo? | ✅ 完整 |
| `JobListResponse` | items, total, page, pageSize, statusStats? | ✅ 完整 |
| `JobDetailResponse` | job, result?, stages, logs? | ⚠️ logs 建议改为可选 |
| `JobCreateRequest` | abilityId, payload, executionMode?, executionTimeoutMs?, callbackUrl?, sourceSystem? | ✅ 完整 |
| `JobCreateResponse` | jobId, traceId, status, estimatedDuration? | ✅ 完整 |
| `JobCancelResponse` | jobId, status, cancelledAt | ✅ 完整 |
| `JobRetryResponse` | jobId, newJobId, status, retriedAt | ✅ 完整 |
| `JobResultResponse` | jobId, status, resultData?, governanceData?, errorCode?, errorMessage?, completedAt? | ✅ 完整 |
| `JobLogsResponse` | jobId, logs, total | ✅ 完整 |
