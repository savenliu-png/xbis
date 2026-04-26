# T015 任务列表页 — 接口联调检查（D1）

## 检查时间
2025-04-26

## 检查范围
- 前端页面：`packages/user/src/pages/Jobs.tsx`
- API Service：`packages/shared/src/api/services.ts`
- 类型定义：`packages/shared/src/types/job.ts`
- API Client：`packages/shared/src/api/client.ts`
- 新增组件：`packages/components/blocks/BatchActionBar/index.tsx`

---

## 1️⃣ API契约一致性

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| URL | `GET /api/v1/jobs` | `GET /api/v1/jobs` | ✅ 一致 |
| Method | GET | GET | ✅ 一致 |
| 参数 page | ✅ 已传递 | `JobListParams.page?: number` | ✅ 一致 |
| 参数 pageSize | ✅ 已传递 | `JobListParams.pageSize?: number` | ✅ 一致 |
| 参数 status | ✅ 已传递 | `JobListParams.status?: JobStatus` | ✅ 一致 |
| 参数 keyword | ✅ 已传递 | `JobListParams.keyword?: string` | ✅ 一致 |
| 参数 dateFrom | ✅ 已传递 | `JobListParams.dateFrom?: string` | ✅ 一致 |
| 参数 dateTo | ✅ 已传递 | `JobListParams.dateTo?: string` | ✅ 一致 |
| 参数 abilityId | ⚠️ 已预留但未使用 | `JobListParams.abilityId?: string` | ⚠️ 前端未暴露 UI |
| 批量取消 URL | `POST /api/v1/jobs/batch-cancel` | `POST /api/v1/jobs/batch-cancel` | ✅ 一致 |
| 批量取消 Method | POST | POST | ✅ 一致 |
| 批量取消 Body | `{ ids: string[] }` | `{ ids: string[] }` | ✅ 一致 |

**说明：** `services.ts` 第 555 行用户端已启用 `includeBatchOps: true`，`batchCancel` 方法已可用。

---

## 2️⃣ 返回结构匹配

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| response.success | ✅ 检查 | `ApiResponse.success: boolean` | ✅ 一致 |
| response.data | ✅ 使用 | `ApiResponse.data?: T` | ✅ 一致 |
| response.data.items | ✅ 使用 | `JobListResponse.items: Job[]` | ✅ 一致 |
| response.data.total | ✅ 使用 | `JobListResponse.total: number` | ✅ 一致 |
| response.data.statusStats | ✅ 使用 | `JobListResponse.statusStats?: {...}` | ✅ 一致 |
| response.data.page | ❌ 未使用 | `JobListResponse.page: number` | ⚠️ 可选字段，不影响功能 |
| response.data.pageSize | ❌ 未使用 | `JobListResponse.pageSize: number` | ⚠️ 可选字段，不影响功能 |

**类型安全验证：**
```typescript
// Jobs.tsx 第 127 行
const response = await userApi.jobs.list(queryParams) as ApiResponse<JobListResponse>;
const resultData = response.data;
```

✅ 使用 `ApiResponse<JobListResponse>` 泛型类型，类型安全。

---

## 3️⃣ 数据结构一致性

| 检查项 | 前端使用 | 类型定义 | 状态 |
|--------|----------|---------|------|
| job.jobId | ✅ | `Job.jobId: string` | ✅ |
| job.status | ✅ | `Job.status: JobStatus` | ✅ |
| job.targetAgent | ✅ | `Job.targetAgent: string` | ✅ |
| job.targetSkill | ✅ | `Job.targetSkill: string` | ✅ |
| job.executionMode | ✅ | `Job.executionMode: JobExecutionMode` | ✅ |
| job.policyRiskLevel | ✅ | `Job.policyRiskLevel: JobPolicyRiskLevel` | ✅ |
| job.payload | ✅ | `Job.payload: Record<string, unknown>` | ✅ |
| job.resultSummary | ✅ | `Job.resultSummary?: string` | ✅ |
| job.callbackUrl | ✅ | `Job.callbackUrl?: string` | ✅ |
| job.createdAt | ✅（TaskItem 中使用） | `Job.createdAt: string` | ✅ |
| job.abilityId | ❌ 未在列表展示 | `Job.abilityId?: string` | ⚠️ 可选字段 |
| job.abilityName | ❌ 未在列表展示 | `Job.abilityName?: string` | ⚠️ 可选字段 |

---

## 4️⃣ 状态流一致性

| 状态值 | 前端筛选 | 类型定义 | TaskStatusBadge | 状态 |
|--------|----------|---------|-----------------|------|
| accepted | ✅ | ✅ | ✅ | ✅ |
| queued | ✅ | ✅ | ✅ | ✅ |
| routing | ✅ | ✅ | ✅ | ✅ |
| running | ✅ | ✅ | ✅ | ✅ |
| waiting_review | ✅ | ✅ | ✅ | ✅ |
| completed | ✅ | ✅ | ✅ | ✅ |
| failed | ✅ | ✅ | ✅ | ✅ |
| cancelled | ✅ | ✅ | ✅ | ✅ |
| callback_pending | ✅ | ✅ | ✅ | ✅ |
| callback_failed | ✅ | ✅ | ✅ | ✅ |

**状态转换规则：** `validJobStatusTransitions` 已在 `job.ts` 中定义，前端未直接使用（由后端控制状态流转）。

---

## 5️⃣ 错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | try/catch + setError |
| 空数据（empty） | ✅ | TaskPageShell 内置 empty 状态 |
| loading 状态 | ✅ | usePageState 管理 |
| 超时/失败 | ✅ | apiClient 有 30s 超时，错误由拦截器统一处理 |
| 批量操作错误 | ✅ | apiClient 拦截器处理，页面层 catch 后刷新 |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 是否通过 Service 层调用 | ✅ | `userApi.jobs.list()` / `userApi.jobs.batchCancel()` |
| 是否绕过 XBIS | ✅ | 未绕过，使用 apiClient |
| API 工厂函数 | ✅ | `createJobApi({ prefix: '/api/v1', includeBatchOps: true })` |

---

## ❌ 问题列表

| # | 类型 | 问题 | 位置 | 影响 |
|---|------|------|------|------|
| 1 | Warning | `abilityId` 参数已预留但 UI 未暴露 | `Jobs.tsx:87` | 能力筛选功能不可用 |
| 2 | Warning | `dateFrom`/`dateTo` 参数已预留但 UI 未暴露 | `Jobs.tsx:88-89` | 日期筛选功能不可用 |
| 3 | Warning | `response.data.page` / `pageSize` 未使用 | `Jobs.tsx:127` | 低影响，前端自行维护分页 |
| 4 | Warning | `JobResultViewer` 未显示时间线 | `Jobs.tsx:244` | 详情预览缺少时间线展示 |

---

## 🛠 修改建议

### 前端修改（可选优化）

#### 问题 1：能力筛选
```typescript
// 如需实现，添加能力选择器
const [selectedAbilityId, setSelectedAbilityId] = useState<string | undefined>();

// 在 queryParams 中添加
...(selectedAbilityId ? { abilityId: selectedAbilityId } : {}),
```

#### 问题 2：日期筛选
```typescript
// 如需实现，添加日期选择器
const [dateRange, setDateRange] = useState<[string, string] | null>(null);
```

#### 问题 3：分页校验
```typescript
// 可选：使用后端返回的 page 校验当前页
const serverPage = resultData?.page || 1;
if (serverPage !== currentPage) {
  setCurrentPage(serverPage);
}
```

### 后端修改
- 无需修改

---

## ⚠️ 风险评估

| 风险项 | 说明 |
|--------|------|
| 是否影响后续任务 | 否，当前实现已满足 T015 需求 |
| 是否影响数据模型 | 否 |
| 遗留功能 | 能力筛选、日期筛选、排序可在后续迭代补充 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

### 通过理由：
1. API 契约一致：URL、Method、参数、返回结构均匹配
2. 类型安全：使用 `ApiResponse<JobListResponse>` 泛型，无 `any` 断言
3. 状态流一致：10 种状态前后端定义完全匹配
4. 错误处理完整：loading/empty/error/idle 四种状态齐全
5. Business Services 层规范：通过 `userApi.jobs` 调用，未绕过接入层
6. 批量操作可用：`includeBatchOps: true` 已启用，`batchCancel` 可正常调用

### 遗留问题（非阻塞性）：
- [ ] 能力筛选 UI 未暴露（Low）
- [ ] 日期筛选 UI 未暴露（Low）
- [ ] 排序功能未实现（Medium）
