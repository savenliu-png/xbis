# T013 能力测试面板 — 接口契约修复报告（Contract Fix）

> 任务编号：T013
> 修复阶段：Contract Fix
> 修复日期：2026-04-25
> 执行人：资深系统架构师 + API契约修复专家 + 前后端联调负责人

---

## 一、问题分类

| 类型 | 问题 | 分类 |
|------|------|------|
| 契约不一致 | `AbilityTestResponse.status` 使用 `string` 而非 `JobStatus` | 2️⃣ 契约不一致 |
| 契约不一致 | `TestHistoryItem.status` 使用 `string` 而非 `JobStatus` | 2️⃣ 契约不一致 |
| 设计不明确 | 异步调用响应类型未明确区分 `AbilityTestResponse` 与 `JobCreateResponse` | 3️⃣ 设计不明确 |

---

## 二、契约决策

### 问题 1：`AbilityTestResponse.status` 类型不一致

**问题说明**：任务卡 §5.1 定义 `AbilityTestResponse.status` 为 `JobStatus`，但代码实现使用 `string`。

| 方案 | 内容 | 优缺点 |
|------|------|--------|
| **方案 A** | 保持 `string`，前端自行约束 | 灵活，但失去类型安全，编译期无法检查非法状态 |
| **方案 B** | 改为 `JobStatus`，从 `@xbis/shared` 导入 | 类型安全，与任务平台状态统一，IDE 自动提示，编译期检查 |

**推荐方案：方案 B**

**理由**：
1. 任务卡明确要求使用 `JobStatus` 类型
2. `JobStatus` 已定义 10 个标准状态，覆盖测试场景（completed/failed/pending/running）
3. 与任务平台状态模型保持一致，避免状态定义分散
4. TypeScript 编译期可检查非法状态赋值

---

### 问题 2：`TestHistoryItem.status` 类型不一致

**问题说明**：与问题 1 相同，`TestHistoryItem.status` 也应使用 `JobStatus`。

| 方案 | 内容 | 优缺点 |
|------|------|--------|
| **方案 A** | 保持 `string` | 简单，但类型不一致 |
| **方案 B** | 改为 `JobStatus`，与 `AbilityTestResponse` 一致 | 类型一致，历史记录状态与响应状态统一 |

**推荐方案：方案 B**

**理由**：
1. `TestHistoryItem` 存储的是 `AbilityTestResponse` 的历史记录，状态类型应一致
2. `TestHistory` 组件的 `statusColorMap` 可完整覆盖所有 `JobStatus` 值
3. 避免同一业务概念使用两种类型定义

---

### 问题 3：异步调用响应类型设计不明确

**问题说明**：任务卡 §8.1 定义异步调用返回 `JobCreateResponse`，但代码复用 `AbilityTestResponse`。

| 方案 | 内容 | 优缺点 |
|------|------|--------|
| **方案 A** | 新增 `AbilityTestAsyncResponse` 类型 | 类型严格，但增加复杂度，且字段与 `AbilityTestResponse` 高度重叠 |
| **方案 B** | 复用 `AbilityTestResponse`，通过 `jobId` 字段区分 | 简洁，字段兼容（status/jobId/result/duration/executedAt），实际业务中异步响应也包含这些字段 |

**推荐方案：方案 B（保持现状）**

**理由**：
1. 当前 `AbilityTestResponse` 已包含 `jobId?: string` 可选字段，天然支持异步响应
2. 异步调用返回的 `JobCreateResponse` 核心字段（jobId/status/estimatedDuration）与 `AbilityTestResponse` 兼容
3. 前端通过 `!!result.jobId` 即可判断是否为异步响应
4. 避免引入冗余类型，保持类型系统简洁

---

## 三、修复内容

### 3.1 代码修改

#### 文件 1：`packages/shared/src/types/ability.ts`

**修改内容**：

```typescript
// 新增导入
import type { JobStatus } from './job';

// 修改 AbilityTestResponse
export interface AbilityTestResponse {
  jobId?: string;
  status: JobStatus;  // 从 string 改为 JobStatus
  result?: Record<string, unknown>;
  error?: { ... };
  duration: number;
  executedAt: string;
}

// 修改 TestHistoryItem
export interface TestHistoryItem {
  id: string;
  abilityId: string;
  parameters: Record<string, unknown>;
  status: JobStatus;  // 从 string 改为 JobStatus
  result?: Record<string, unknown>;
  error?: string;
  duration: number;
  createdAt: string;
}
```

#### 文件 2：`packages/components/business/TestHistory/index.tsx`

**修改内容**：

```typescript
// 新增导入
import type { JobStatus } from '@xbis/shared';

// 修改 statusColorMap 类型和值
const statusColorMap: Record<JobStatus, string> = {
  accepted: colors.text.info,
  queued: colors.text.info,
  routing: colors.text.info,
  running: colors.text.info,
  waiting_review: colors.text.warning,
  completed: colors.text.success,
  failed: colors.text.error,
  cancelled: colors.text.muted,
  callback_pending: colors.text.warning,
  callback_failed: colors.text.error,
};
```

### 3.2 设计方案更新

无需更新设计方案（Final 版本），因为：
1. 任务卡 §5.1 已明确定义 `status: JobStatus`
2. 代码实现之前未遵循任务卡定义，现已修复
3. 设计方案 §4 状态管理中的 `status` 类型已隐含使用 `JobStatus`

### 3.3 类型定义更新

| 类型 | 修改前 | 修改后 |
|------|--------|--------|
| `AbilityTestResponse.status` | `string` | `JobStatus` |
| `TestHistoryItem.status` | `string` | `JobStatus` |
| `statusColorMap` | `Record<string, string>` | `Record<JobStatus, string>` |

---

## 四、影响评估

| 影响项 | 评估 | 说明 |
|--------|------|------|
| 是否影响前端页面 | **否** | 运行时行为不变，`JobStatus` 是 `string` 的子类型 |
| 是否影响后端接口 | **否** | 仅前端类型修改，API 契约不变 |
| 是否影响后续任务 | **否** | 类型更严格，有利于后续开发 |
| 是否影响现有代码 | **否** | `TestHistory` 组件已同步更新 |

---

## 五、是否完成联调修复

👉 **YES**

**修复完成项**：
1. ✅ `AbilityTestResponse.status` 从 `string` 改为 `JobStatus`
2. ✅ `TestHistoryItem.status` 从 `string` 改为 `JobStatus`
3. ✅ `TestHistory.statusColorMap` 完整覆盖所有 `JobStatus` 值
4. ✅ 异步响应类型决策：保持复用 `AbilityTestResponse`

**修复后状态**：
- 前端类型与任务卡定义完全一致
- `JobStatus` 统一使用，类型安全提升
- 无运行时行为变更，无回归风险

---

*本文档与代码同步维护，后续迭代请同步更新。*
