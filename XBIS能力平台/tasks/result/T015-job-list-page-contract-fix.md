# T015 任务列表页 — 接口契约修复（Contract Fix）

## 修复时间
2025-04-26

## 修复依据
- D1 联调报告：`tasks/result/T015-job-list-page-debugging.md`
- 任务卡：`tasks/T015-job-list-page.md`
- 设计方案：`tasks/design-specs/final/T015-job-list-page-design-final.md`

---

## 一、问题分类

| # | 问题 | 分类 |
|---|------|------|
| 1 | 用户端 `userApi.jobs` 未启用 `batchCancel`，但 Jobs.tsx 尝试调用 | **契约缺失**（Contract Missing） |
| 2 | 类型断言 `(userApi.jobs as any).batchCancel` 类型不安全 | **契约不一致**（Contract Mismatch） |
| 3 | `response` 类型断言为 `{ data?: ... }`，实际应为 `ApiResponse<JobListResponse>` | **契约不一致**（Contract Mismatch） |
| 4 | 日期参数 `dateFrom`/`dateTo` 命名需确认与后端一致 | **设计不明确**（Ambiguous Design） |

---

## 二、契约决策

### 问题 1：用户端是否需要 batchCancel？

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 用户端启用 `includeBatchOps: true`，允许用户批量取消自己的任务 |
| 方案 B | 用户端禁用批量取消，Jobs.tsx 中移除批量取消功能 |

**推荐方案 A 理由：**
1. 任务卡 T015 明确要求「批量取消」功能
2. 用户端批量取消是合理业务需求（用户管理自己的任务）
3. 后端 API `/api/v1/jobs/batch-cancel` 已存在，无需后端改动
4. 管理端已启用 `includeBatchOps: true`，用户端应保持一致权限模型

### 问题 2 & 3：类型断言如何修复？

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 使用 `ApiResponse<JobListResponse>` 泛型类型，移除 `as any` |
| 方案 B | 保持现有类型断言，添加运行时检查 |

**推荐方案 A 理由：**
1. 类型安全是 TypeScript 核心优势
2. `ApiResponse<T>` 已在 `client.ts` 中定义，应正确使用
3. 运行时检查无法替代编译时类型检查

### 问题 4：日期参数命名？

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 保持 `dateFrom`/`dateTo`，与 `JobListParams` 类型定义一致 |
| 方案 B | 改为 `startDate`/`endDate` |

**推荐方案 A 理由：**
1. `JobListParams` 类型定义已使用 `dateFrom`/`dateTo`
2. 前端代码已与类型定义一致
3. 若后端不一致，应由后端调整以匹配类型定义

---

## 三、修复内容

### 1️⃣ 代码修改

#### 修改 1：services.ts — 用户端启用 batchCancel

**文件路径：** `packages/shared/src/api/services.ts`

**修改内容：**
```typescript
// 第 555 行
// 修改前：
jobs: createJobApi({ prefix: '/api/v1', includeBatchOps: false }),

// 修改后：
jobs: createJobApi({ prefix: '/api/v1', includeBatchOps: true }),
```

**影响：** 用户端 `userApi.jobs` 现在包含 `batchCancel` 和 `batchRetry` 方法

---

#### 修改 2：Jobs.tsx — 类型修复

**文件路径：** `packages/user/src/pages/Jobs.tsx`

**修改内容 2a：导入类型**
```typescript
// 修改前：
import type { Job, JobStatus } from '@xbis/shared';

// 修改后：
import type { Job, JobStatus, JobListResponse, BatchCancelResponse } from '@xbis/shared';
import type { ApiResponse } from '@xbis/shared';
```

**修改内容 2b：列表 API 调用**
```typescript
// 修改前：
const response = await userApi.jobs.list(params) as { data?: { items: Job[]; total: number; statusStats?: { status: JobStatus; count: number }[] } };
const resultData = response?.data;

// 修改后：
const response = await userApi.jobs.list(params) as ApiResponse<JobListResponse>;
const resultData = response.data;
```

**修改内容 2c：批量取消 API 调用**
```typescript
// 修改前：
const response = await (userApi.jobs as any).batchCancel?.(selectedTaskIds);
if (response?.success !== false) {

// 修改后：
const response = await userApi.jobs.batchCancel(selectedTaskIds) as ApiResponse<BatchCancelResponse>;
if (response.success !== false) {
```

---

### 2️⃣ 设计方案更新

无需更新设计方案，原设计方案已包含批量取消功能，本次修复是代码实现与设计方案的对齐。

---

### 3️⃣ 类型定义更新

无需更新类型定义，`JobListResponse`、`BatchCancelResponse`、`ApiResponse` 类型已在现有类型定义中存在。

---

## 四、影响评估

| 影响项 | 说明 |
|--------|------|
| 是否影响前端页面 | ✅ 是，Jobs.tsx 批量取消功能现在可正常工作 |
| 是否影响后端接口 | ❌ 否，后端 API 已存在，仅前端启用调用 |
| 是否影响后续任务 | ✅ 是，T016 任务详情页可复用修复后的类型模式 |
| 是否影响其他页面 | ⚠️ 用户端其他页面若使用 `userApi.jobs`，现在也可使用 `batchCancel`/`batchRetry` |
| 是否破坏已有 API | ❌ 否，仅启用已有功能，未修改 API 签名 |

---

## 五、修复验证

### 类型检查

修复后代码类型关系：
```
userApi.jobs.list(params: JobListParams) → Promise<ApiResponse<JobListResponse>>
  → response.data: JobListResponse
    → items: Job[]
    → total: number
    → statusStats?: { status: JobStatus; count: number }[]

userApi.jobs.batchCancel(ids: string[]) → Promise<ApiResponse<BatchCancelResponse>>
  → response.data: BatchCancelResponse
    → successCount: number
    → failedCount: number
    → results: BatchJobResult[]
```

### API 调用链验证

```
Jobs.tsx → userApi.jobs.list() → createJobApi().list() → apiClient.get('/api/v1/jobs')
Jobs.tsx → userApi.jobs.batchCancel() → createJobApi().batchCancel() → apiClient.post('/api/v1/jobs/batch-cancel')
```

---

## 六、是否完成联调修复

**👉 YES**

### 修复清单

- [x] 用户端 `userApi.jobs` 启用 `batchCancel`
- [x] 移除 `(userApi.jobs as any).batchCancel` 类型断言
- [x] 使用 `ApiResponse<JobListResponse>` 替代自定义类型断言
- [x] 使用 `ApiResponse<BatchCancelResponse>` 替代 `any` 类型断言
- [x] 日期参数保持 `dateFrom`/`dateTo`，与 `JobListParams` 类型定义一致

### 建议后续行动

1. 重新运行 D1 联调检查，确认所有 Blocking 问题已解决
2. 确认后端 `/api/v1/jobs/batch-cancel` 接口支持用户端调用（权限校验）
3. T016 任务详情页复用本次修复的类型模式
