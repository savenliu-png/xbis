# T017 任务创建流程 — 接口联调检查（D1）

## 检查时间
2025-04-26

## 检查范围
- 前端页面：`packages/user/src/pages/JobCreate.tsx`
- 表单组件：`packages/components/blocks/JobCreateForm/index.tsx`
- 业务组件：`packages/components/business/AbilitySelector/index.tsx`、`ExecutorSelector/index.tsx`、`PayloadEditor/index.tsx`
- API Service：`packages/shared/src/api/services.ts`
- 类型定义：`packages/shared/src/types/job.ts`、`packages/shared/src/types/ability.ts`、`packages/shared/src/types/executor.ts`

---

## 1️⃣ API契约一致性

### 1.1 能力列表 API

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| URL | `GET /api/v1/abilities` | `GET /api/v1/abilities` | ✅ 一致 |
| Method | GET | GET | ✅ 一致 |
| 调用位置 | JobCreate.tsx:62 | `userApi.abilities.list()` | ✅ 一致 |

### 1.2 能力 Schema API

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| URL | `GET /api/v1/abilities/:id/schema` | `GET /api/v1/abilities/:id/schema` | ✅ 一致 |
| Method | GET | GET | ✅ 一致 |
| 调用位置 | JobCreate.tsx:80 | `userApi.abilities.getSchema(abilityId)` | ✅ 一致 |

### 1.3 执行器列表 API

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| URL | `GET /api/v1/executors` | `GET /api/v1/executors` | ✅ 一致 |
| Method | GET | GET | ✅ 一致 |
| 调用位置 | JobCreate.tsx:63 | `userApi.executors.list()` | ✅ 一致 |

### 1.4 任务创建 API

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| URL | `POST /api/v1/jobs` | `POST /api/v1/jobs` | ✅ 一致 |
| Method | POST | POST | ✅ 一致 |
| Body | `JobCreateRequest` | `JobCreateRequest` | ⚠️ 见下方字段差异 |

### 1.5 字段差异分析

**JobCreateRequest 类型定义（job.ts:173-183）：**
```typescript
export interface JobCreateRequest {
  abilityId: string;
  payload: Record<string, unknown>;
  executionMode?: JobExecutionMode;  // 'async' | 'sync'
  executionTimeoutMs?: number;
  timeout?: number;  // @deprecated
  callbackUrl?: string;
  sourceSystem?: string;
}
```

**前端实际发送的数据（JobCreate.tsx:122-131）：**
```typescript
const jobData: JobCreateRequest = {
  abilityId: formData.abilityId,
  targetAgent: formData.targetAgent,      // ❌ 不在 JobCreateRequest 中
  targetSkill: formData.targetSkill,      // ❌ 不在 JobCreateRequest 中
  payload: formData.payload,
  executionMode: formData.executionMode,
  policyRiskLevel: formData.policyRiskLevel,  // ❌ 不在 JobCreateRequest 中
  ...(formData.callbackUrl ? { callbackUrl: formData.callbackUrl } : {}),
  ...(formData.executorId ? { executorId: formData.executorId } : {}),  // ❌ 不在 JobCreateRequest 中
};
```

| 字段 | 前端发送 | 类型定义 | 状态 |
|------|----------|---------|------|
| abilityId | ✅ | ✅ | ✅ |
| payload | ✅ | ✅ | ✅ |
| executionMode | ✅ | ✅ | ✅ |
| callbackUrl | ✅ | ✅ | ✅ |
| targetAgent | ✅ | ❌ | ❌ 多余字段 |
| targetSkill | ✅ | ❌ | ❌ 多余字段 |
| policyRiskLevel | ✅ | ❌ | ❌ 多余字段 |
| executorId | ✅ | ❌ | ❌ 多余字段 |
| executionTimeoutMs | ❌ | ✅ | ⚠️ 可选，未发送 |
| sourceSystem | ❌ | ✅ | ⚠️ 可选，未发送 |

---

## 2️⃣ 返回结构匹配

### 2.1 能力列表返回

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| response.data.items | ✅ | `AbilityListResponse.items: Ability[]` | ✅ |
| response.data.total | ❌ 未使用 | `AbilityListResponse.total` | ⚠️ 可选 |

### 2.2 执行器列表返回

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| response.data.items | ✅ | `ExecutorListResponse.items: Executor[]` | ✅ |

**注意：** 前端 `ExecutorSelector` 使用本地定义的 `Executor` 类型，与后端 `Executor` 类型字段名存在差异：

| 字段 | 前端（ExecutorSelector） | 后端（executor.ts） | 状态 |
|------|------------------------|---------------------|------|
| id | ✅ | executorId | ❌ 字段名不匹配 |
| name | ✅ | ✅ | ✅ |
| type | ✅ | ✅ | ✅ |
| status | 'active' \| 'inactive' \| 'busy' | 'active' \| 'inactive' \| 'degraded' \| 'offline' | ⚠️ 状态值不匹配 |
| description | ✅ | ✅ | ✅ |

### 2.3 任务创建返回

| 检查项 | 前端实现 | API定义 | 状态 |
|--------|----------|---------|------|
| response.data.jobId | ✅ | `JobCreateResponse.jobId` | ✅ |
| response.success | ✅ | `ApiResponse.success` | ✅ |

---

## 3️⃣ 数据结构一致性

### 3.1 Ability 类型一致性

| 字段 | 前端使用 | 类型定义 | 状态 |
|------|----------|---------|------|
| abilityId | ✅ | `Ability.abilityId` | ✅ |
| name | ✅ | `Ability.name` | ✅ |
| targetAgent | ✅ | `Ability` 中无此字段 | ❌ 类型不匹配 |
| targetSkill | ✅ | `Ability` 中无此字段 | ❌ 类型不匹配 |
| description | ✅ | `Ability.description` | ✅ |
| tags | ✅ | `Ability.tags` | ✅ |

**问题：** `Ability` 类型定义中没有 `targetAgent` 和 `targetSkill` 字段，但前端代码中使用了 `ability.targetAgent` 和 `ability.targetSkill`。

### 3.2 JobCreateFormData 类型

前端自定义类型包含字段：
- `abilityId`, `targetAgent`, `targetSkill`, `payload`, `executionMode`, `policyRiskLevel`, `callbackUrl`, `executorId`

与 `JobCreateRequest` 对比：
- `targetAgent`, `targetSkill`, `policyRiskLevel`, `executorId` 不在 `JobCreateRequest` 中

---

## 4️⃣ 状态流一致性

任务创建流程不涉及状态流转（创建后直接进入 `accepted` 状态），此检查项不适用。

---

## 5️⃣ 错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | try/catch + setError |
| 空数据（empty） | ✅ | AbilitySelector/ExecutorSelector 内置空状态 |
| loading 状态 | ✅ | FormPageShell 管理 loading/submitting |
| 表单验证 | ✅ | PayloadEditor JSON 校验 + 必填校验 |
| 提交失败 | ✅ | catch 中 setValidationError |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 是否通过 Service 层调用 | ✅ | `userApi.abilities.list()`、`userApi.jobs.create()` 等 |
| 是否绕过 XBIS | ✅ | 未绕过，使用 apiClient |
| API 工厂函数 | ✅ | `createAbilityApi`、`createJobApi`、`createExecutorApi` |

---

## ❌ 问题列表

| # | 类型 | 问题 | 位置 | 影响 |
|---|------|------|------|------|
| 1 | **Blocking** | `JobCreateRequest` 缺少 `targetAgent`、`targetSkill`、`policyRiskLevel`、`executorId` 字段 | `job.ts:173-183` | 前端发送的数据与后端类型不匹配，可能导致后端拒绝请求 |
| 2 | **Blocking** | `Ability` 类型缺少 `targetAgent`、`targetSkill` 字段 | `ability.ts:19-61` | 前端代码使用 `ability.targetAgent` 会编译报错/运行时 undefined |
| 3 | **Blocking** | `ExecutorSelector` 使用本地 `Executor` 类型与后端 `Executor` 类型字段名不匹配（`id` vs `executorId`） | `ExecutorSelector/index.tsx:13-19` | 执行器选择后传递的 `executor.id` 可能不匹配后端数据 |
| 4 | **Blocking** | `ExecutorSelector` 状态值与后端不匹配（`'busy'` vs `'degraded'`/`'offline'`） | `ExecutorSelector/index.tsx:17` | 状态判断逻辑可能失效 |
| 5 | Warning | `JobCreateRequest` 未发送 `executionTimeoutMs` 和 `sourceSystem` | `JobCreate.tsx:122-131` | 使用后端默认值 |
| 6 | Warning | `JobCreateFormData.executionMode` 包含 `'callback'`，但 `JobExecutionMode` 只有 `'async'` \| `'sync'` | `JobCreateForm/index.tsx:18` | 类型不匹配，后端可能不支持 `'callback'` |
| 7 | Warning | `JobCreateFormData.policyRiskLevel` 包含 `'low'` \| `'medium'` \| `'high'`，但 `JobPolicyRiskLevel` 包含 `'critical'` | `JobCreateForm/index.tsx:19` | 前端缺少 `'critical'` 选项 |

---

## 🛠 修改建议

### 前端修改

#### 修复 Blocking #1：调整 JobCreateRequest 使用方式

**方案 A（推荐）：** 扩展 `JobCreateRequest` 类型定义以包含前端需要的字段

**方案 B：** 前端只发送 `JobCreateRequest` 定义的字段，移除多余字段

```typescript
// 修改 JobCreate.tsx 中的 handleSubmit
const jobData: JobCreateRequest = {
  abilityId: formData.abilityId,
  payload: formData.payload,
  executionMode: formData.executionMode === 'callback' ? 'async' : formData.executionMode, // 转换 callback 为 async
  callbackUrl: formData.callbackUrl,
  // 移除：targetAgent, targetSkill, policyRiskLevel, executorId
};
```

#### 修复 Blocking #2：Ability 类型补充字段或前端适配

```typescript
// 方案 A：扩展 Ability 类型
export interface Ability {
  // ... 现有字段
  targetAgent?: string;  // 新增
  targetSkill?: string;  // 新增
}

// 方案 B：前端使用 ability.name 替代 targetAgent/targetSkill
```

#### 修复 Blocking #3 & #4：ExecutorSelector 使用后端类型

```typescript
// 使用后端 Executor 类型
import type { Executor } from '@xbis/shared';

// 适配字段名
const executorId = executor.executorId; // 而非 executor.id
```

### 后端修改

- 如需支持前端字段，扩展 `JobCreateRequest` 类型定义
- 确认 `executionMode` 是否支持 `'callback'` 值

---

## ⚠️ 风险评估

| 风险项 | 说明 |
|--------|------|
| 是否影响后续任务 | ✅ 是，T016 任务详情页可能复用相同的类型定义 |
| 是否影响数据模型 | ⚠️ 如修改 `JobCreateRequest`，需确保后端接口兼容 |
| 执行器类型不匹配 | 可能导致执行器选择功能异常 |

---

## ✅ 是否允许进入验收（D2）

**👉 NO**

### 必须修复的 Blocking 问题：

1. **Blocking #1**：`JobCreateRequest` 类型定义与前端发送的数据字段不匹配
2. **Blocking #2**：`Ability` 类型缺少 `targetAgent`/`targetSkill` 字段
3. **Blocking #3**：`ExecutorSelector` 使用本地类型与后端类型字段名不匹配
4. **Blocking #4**：`ExecutorSelector` 状态值与后端不匹配

### 修复后需重新检查：
- [ ] `JobCreateRequest` 字段前后端一致
- [ ] `Ability` 类型包含前端需要的字段
- [ ] `Executor` 类型前后端一致
- [ ] 执行模式 `'callback'` 后端支持确认
