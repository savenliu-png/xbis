# T017 任务创建流程 — 接口契约修复（Contract Fix）

## 修复时间
2025-04-26

## 修复依据
- D1 联调报告：`tasks/result/T017-job-create-flow-debugging.md`
- 任务卡：`tasks/T017-job-create-flow.md`
- 设计方案：`tasks/design-specs/final/T017-job-create-flow-design-final.md`

---

## 一、问题分类

| # | 问题 | 分类 |
|---|------|------|
| 1 | `JobCreateRequest` 缺少 `targetAgent`、`targetSkill`、`policyRiskLevel`、`executorId` 字段 | **契约不一致**（Contract Mismatch） |
| 2 | `Ability` 类型缺少 `targetAgent`、`targetSkill` 字段 | **契约不一致**（Contract Mismatch） |
| 3 | `ExecutorSelector` 使用本地 `Executor` 类型与后端字段名不匹配 | **契约不一致**（Contract Mismatch） |
| 4 | `ExecutorSelector` 状态值与后端不匹配 | **契约不一致**（Contract Mismatch） |
| 5 | `executionMode` 包含 `'callback'`，但后端只支持 `'async'`/`'sync'` | **设计不明确**（Ambiguous Design） |
| 6 | `policyRiskLevel` 缺少 `'critical'` 选项 | **契约缺失**（Contract Missing） |

---

## 二、契约决策

### 问题 1：JobCreateRequest 字段不匹配

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 扩展 `JobCreateRequest` 类型，添加 `targetAgent`、`targetSkill`、`policyRiskLevel`、`executorId` 字段 |
| 方案 B | 前端移除多余字段，只发送类型定义的字段 |

**推荐方案 A 理由：**
1. `targetAgent`/`targetSkill` 是 `Job` 实体中的必填字段，创建时应允许指定
2. `policyRiskLevel` 是任务策略配置，应在创建时指定
3. `executorId` 是指定执行器，业务上合理
4. 不破坏已有 API，只是扩展请求类型

### 问题 2：Ability 类型缺少 targetAgent/targetSkill

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 扩展 `Ability` 类型，添加 `targetAgent`、`targetSkill` 可选字段 |
| 方案 B | 前端从 `ability.name` 或其他字段推导 |

**推荐方案 A 理由：**
1. 能力定义中应包含执行目标信息
2. 与 `Job` 实体中的 `targetAgent`/`targetSkill` 保持一致
3. 便于前端展示和后续任务创建

### 问题 3 & 4：Executor 类型不匹配

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | `ExecutorSelector` 使用后端 `Executor` 类型，字段名用 `executorId` |
| 方案 B | 修改后端 `Executor` 类型，使用 `id` 替代 `executorId` |

**推荐方案 A 理由：**
1. 后端类型已定义，不应随意修改
2. 前端应适配后端类型，保持数据一致性
3. 状态值应与后端 `ExecutorStatus` 对齐

### 问题 5：executionMode 包含 'callback'

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 前端移除 `'callback'` 选项，使用 `'async'` + `callbackUrl` 实现回调功能 |
| 方案 B | 后端扩展 `JobExecutionMode` 支持 `'callback'` |

**推荐方案 A 理由：**
1. 后端 `JobExecutionMode` 已定义为 `'async' | 'sync'`
2. 回调功能可通过 `async` + `callbackUrl` 实现
3. 不修改后端类型，保持兼容性

### 问题 6：policyRiskLevel 缺少 'critical'

| 方案 | 说明 |
|------|------|
| **方案 A（推荐）** | 前端添加 `'critical'` 选项 |
| 方案 B | 后端移除 `'critical'` 选项 |

**推荐方案 A 理由：**
1. 后端 `JobPolicyRiskLevel` 已包含 `'critical'`
2. 前端应完整展示所有选项
3. 用户应有选择最高风险等级的权利

---

## 三、修复内容

### 1️⃣ 代码修改

#### 修改 1：job.ts — 扩展 JobCreateRequest

**文件路径：** `packages/shared/src/types/job.ts`

**修改内容：**
```typescript
export interface JobCreateRequest {
  abilityId: string;
  payload: Record<string, unknown>;
  executionMode?: JobExecutionMode;
  executionTimeoutMs?: number;
  timeout?: number;
  callbackUrl?: string;
  sourceSystem?: string;

  // 扩展字段：执行目标
  targetAgent?: string;
  targetSkill?: string;

  // 扩展字段：策略配置
  policyRiskLevel?: JobPolicyRiskLevel;

  // 扩展字段：指定执行器
  executorId?: string;
}
```

#### 修改 2：ability.ts — 扩展 Ability

**文件路径：** `packages/shared/src/types/ability.ts`

**修改内容：**
```typescript
export interface Ability {
  // ... 现有字段
  retryCount: number;

  // 执行目标
  targetAgent?: string;
  targetSkill?: string;

  // 计费配置
  pricingType: AbilityPricingType;
  // ...
}
```

#### 修改 3：ExecutorSelector — 使用后端 Executor 类型

**文件路径：** `packages/components/business/ExecutorSelector/index.tsx`

**修改内容：**
- 导入后端 `Executor` 类型：`import type { Executor, ExecutorStatus } from '@xbis/shared'`
- 使用 `executor.executorId` 替代 `executor.id`
- 状态值与 `ExecutorStatus` 对齐：`active` | `inactive` | `degraded` | `offline`

#### 修改 4：JobCreateForm — 移除 callback 模式

**文件路径：** `packages/components/blocks/JobCreateForm/index.tsx`

**修改内容：**
- `ExecutionMode` 类型改为：`'sync' | 'async'`
- 移除 `'callback'` 选项
- 回调 URL 输入框在 `executionMode === 'async'` 时显示

#### 修改 5：JobCreateForm — 添加 critical 风险等级

**文件路径：** `packages/components/blocks/JobCreateForm/index.tsx`

**修改内容：**
- `RISK_LEVEL_OPTIONS` 添加：`{ label: '极高风险', value: 'critical' }`

#### 修改 6：JobCreate.tsx — 适配后端类型

**文件路径：** `packages/user/src/pages/JobCreate.tsx`

**修改内容：**
- 导入后端 `Executor` 类型：`import type { Executor } from '@xbis/shared'`
- 使用 `executor.executorId` 替代 `executor.id`
- 风险等级显示添加 `'critical'` → `'极高风险'`

---

### 2️⃣ 设计方案更新

无需更新设计方案，原设计方案已包含任务创建流程，本次修复是代码实现与类型定义的对齐。

---

### 3️⃣ 类型定义更新

| 文件 | 修改内容 |
|------|----------|
| `packages/shared/src/types/job.ts` | `JobCreateRequest` 添加 `targetAgent`、`targetSkill`、`policyRiskLevel`、`executorId` |
| `packages/shared/src/types/ability.ts` | `Ability` 添加 `targetAgent`、`targetSkill` |

---

## 四、影响评估

| 影响项 | 说明 |
|--------|------|
| 是否影响前端页面 | ✅ 是，JobCreate.tsx 和组件已适配 |
| 是否影响后端接口 | ⚠️ 否，后端接口未变更，仅扩展请求类型 |
| 是否影响后续任务 | ✅ 是，T016 任务详情页可复用修复后的类型 |
| 是否影响已有功能 | ❌ 否，仅扩展类型，不破坏已有字段 |

---

## 五、是否完成联调修复

**👉 YES**

### 修复清单

- [x] `JobCreateRequest` 扩展 `targetAgent`、`targetSkill`、`policyRiskLevel`、`executorId`
- [x] `Ability` 扩展 `targetAgent`、`targetSkill`
- [x] `ExecutorSelector` 使用后端 `Executor` 类型
- [x] `ExecutorSelector` 状态值与后端对齐
- [x] 移除 `'callback'` 执行模式，使用 `'async'` + `callbackUrl`
- [x] 添加 `'critical'` 风险等级选项

### 建议后续行动

1. 重新运行 D1 联调检查，确认所有 Blocking 问题已解决
2. 确认后端 `/api/v1/jobs` 接口支持新扩展的字段
3. T016 任务详情页复用本次修复的类型定义
