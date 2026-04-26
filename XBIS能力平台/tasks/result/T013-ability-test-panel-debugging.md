# T013 能力测试面板 — 接口联调检查报告（D1）

> 任务编号：T013
> 检查阶段：D1 — 接口联调检查
> 检查日期：2026-04-25
> 执行人：资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 🧪 联调结果（D1）

👉 **状态：联调通过**

说明：6 个检查维度全部通过，发现 2 项类型建议项，无阻塞性问题。

---

## 一、API 契约一致性

| 检查项 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|--------|-----------|----------|------|------|
| URL | `POST /api/v1/abilities/:id/test` | `POST /api/v1/abilities/${id}/test` | ✅ 一致 | `services.ts:148` |
| Method | POST | POST | ✅ 一致 | — |
| Path 参数 | `:id` | `encodePathParam(id)` | ✅ 一致 | 已编码处理 |
| Body 参数 | `abilityId`, `parameters`, `executionMode` | 完全一致 | ✅ 一致 | `AbilityTestPage.tsx:141-145` |
| 参数类型 | `AbilityTestRequest` | `{ abilityId: string; parameters: Record<string, unknown>; executionMode?: 'sync' \| 'async' }` | ✅ 一致 | — |
| 多余参数 | 无 | 无 | ✅ 通过 | — |

**验证依据**：
- 任务卡 §8.1 定义：`POST /api/v1/abilities/:id/test`
- 代码实现：`packages/shared/src/api/services.ts:147-148`
- 调用位置：`packages/pages/ability/AbilityTestPage.tsx:141-145`

---

## 二、返回结构匹配

### 2.1 同步调用响应

| 字段 | 任务卡定义 | 代码类型定义 | 状态 | 说明 |
|------|-----------|-------------|------|------|
| `status` | `JobStatus` | `string` | ⚠️ 建议 | 任务卡使用 `JobStatus`，代码使用 `string` |
| `result` | `Record<string, any>` | `Record<string, unknown>` | ✅ 一致 | 类型更严格 |
| `duration` | `number` | `number` | ✅ 一致 | — |
| `executedAt` | `string` | `string` | ✅ 一致 | — |
| `jobId` | `string` (optional) | `string` (optional) | ✅ 一致 | 异步时返回 |
| `error` | `{ code, message, details }` | `{ code, message, details }` | ✅ 一致 | — |

### 2.2 异步调用响应

| 字段 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|------|-----------|----------|------|------|
| 响应类型 | `JobCreateResponse` | 复用 `AbilityTestResponse` | ⚠️ 建议 | 任务卡定义两种响应类型 |

**验证依据**：
- 任务卡 §8.1 响应示例：`{ success: true, data: { status, result, duration, executedAt } }`
- 代码类型：`packages/shared/src/types/ability.ts:252-263`
- 代码解析：`AbilityTestPage.tsx:147` `(response as unknown as { data?: AbilityTestResponse }).data`

---

## 三、数据结构一致性

| 检查项 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|--------|-----------|----------|------|------|
| `TestParameter` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:237-244` |
| `AbilityTestRequest` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:246-250` |
| `AbilityTestResponse` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:252-263` |
| `TestHistoryItem` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:265-274` |
| 字段命名冲突 | 无 | 无 | ✅ 通过 | — |
| 结构嵌套 | 正确 | 正确 | ✅ 通过 | — |

---

## 四、状态流一致性

| 状态 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|------|-----------|----------|------|------|
| `completed` | ✅ | ✅ | ✅ 一致 | 同步调用成功 |
| `failed` | ✅ | ✅ | ✅ 一致 | 调用失败 |
| `pending` | ✅ | ✅ | ✅ 一致 | 异步任务等待中 |
| `running` | ✅ | ✅ | ✅ 一致 | 异步任务执行中 |

**状态转换验证**：
```
用户点击调用
  ├── 同步模式 ──► completed / failed
  └── 异步模式 ──► pending ──► running ──► completed / failed
```

**验证依据**：
- 任务卡 §6.2 异常流程：调用失败 → 展示错误信息
- 代码实现：`AbilityTestPage.tsx:126-172` handleTest 函数

---

## 五、错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | `try/catch` 捕获，`setTestError` 设置错误状态 |
| 空数据处理 | ✅ | `if (!result)` 校验，抛出错误 |
| loading 处理 | ✅ | `testLoading` 状态，`FormPageShell` 显示 submitting |
| 超时/失败处理 | ✅ | 同步调用由 API 层处理超时，错误由 catch 捕获 |
| JSON 解析错误 | ✅ | `JSON.parse(manualJson)` 在 try 块内，错误会捕获 |

**验证依据**：
- `AbilityTestPage.tsx:126-172` handleTest 函数包含完整 try/catch/finally
- `AbilityTestPage.tsx:265-271` 错误展示使用 Alert 组件

---

## 六、Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 是否通过 Service 层调用 API | ✅ | `userApi.abilities.test(id, data)` |
| 是否绕过业务接入层 | ✅ | 未绕过，使用 `userApi` |
| API 定义位置 | ✅ | `packages/shared/src/api/services.ts` |

**验证依据**：
- `packages/shared/src/api/services.ts:147-148` 定义 test API
- `packages/pages/ability/AbilityTestPage.tsx:141` 通过 `userApi.abilities.test` 调用

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 严重级别 |
|------|------|------|------|----------|
| 类型不一致 | `AbilityTestResponse.status` 使用 `string` 而非 `JobStatus` | `ability.ts:254` | 类型安全性降低 | 🟡 建议 |
| 类型不一致 | `TestHistoryItem.status` 使用 `string` 而非 `JobStatus` | `ability.ts:269` | 类型安全性降低 | 🟡 建议 |
| 响应类型 | 异步调用应返回 `JobCreateResponse`，代码复用 `AbilityTestResponse` | `AbilityTestPage.tsx:147` | 异步响应字段可能不匹配 | 🟡 建议 |

---

## 🛠 修改建议

### 前端修改

**建议 1：使用 JobStatus 类型替代 string**

```typescript
// packages/shared/src/types/ability.ts
import type { JobStatus } from './job';

export interface AbilityTestResponse {
  jobId?: string;
  status: JobStatus;  // 替代 string
  result?: Record<string, unknown>;
  error?: { ... };
  duration: number;
  executedAt: string;
}

export interface TestHistoryItem {
  id: string;
  abilityId: string;
  parameters: Record<string, unknown>;
  status: JobStatus;  // 替代 string
  result?: Record<string, unknown>;
  error?: string;
  duration: number;
  createdAt: string;
}
```

**建议 2：异步调用响应类型处理**

```typescript
// 当前代码已兼容，因为异步响应也包含 status 字段
// 如需严格区分，可添加类型守卫
const isAsyncResponse = (result: AbilityTestResponse): boolean => !!result.jobId;
```

### 后端修改

无 — 前端实现与任务卡 API 定义一致。

### 数据结构调整

无 — 当前数据结构满足需求。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | **低** | 类型建议不影响运行时功能 |
| 是否影响数据模型 | **无** | 当前类型定义满足接口契约 |
| 异步响应类型 | **低** | 异步响应复用同一类型，字段兼容 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

**通过理由**：
1. API 契约一致性：URL、Method、参数完全一致
2. 返回结构匹配：字段名、类型匹配（除 JobStatus 建议项）
3. 数据结构一致性：类型定义完整，无冲突
4. 状态流一致性：completed/failed/pending/running 全部匹配
5. 错误处理完整：API error、空数据、loading、超时均有处理
6. Business Services 层合规：通过 `userApi.abilities.test` 调用，未绕过接入层

**遗留建议**（非阻塞）：
- 将 `AbilityTestResponse.status` 和 `TestHistoryItem.status` 类型从 `string` 改为 `JobStatus`
- 评估异步调用是否需要独立响应类型

---

*本文档与代码同步维护，后续迭代请同步更新。*
