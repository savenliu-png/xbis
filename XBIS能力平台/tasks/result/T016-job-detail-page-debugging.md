# T016 任务详情页 - 接口联调检查报告（D1）

> 任务编号：T016
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-26
> 检查角色：后端架构师 + 前端联调负责人 + API 契约审查官

---

## 🧪 联调结果

**状态：✅ 修复完成，允许进入 D2 验收**

原始问题 5 个，已修复 4 个，1 个经确认无需修复。

---

## ❌ 问题列表（含修复状态）

### 问题 #1：日志 API 参数缺少 `level` 字段（中）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | API 契约 |
| **位置** | `packages/shared/src/api/services.ts` 第 55 行 |

**详情**：
- 任务卡定义：`logs: (id, params?: LogsQueryParams)`，其中 `LogsQueryParams = { page?: number; pageSize?: number; level?: string }`
- 前端实现：`logs: (id, params?: { page?: number; pageSize?: number })`

**修复方式**：
```typescript
logs: (id: string, params?: { page?: number; pageSize?: number; level?: string }) =>
  apiClient.get(`/api/v1/jobs/${encodePathParam(id)}/logs`, { params }),
```

---

### 问题 #2：`JobDetailResponse` 类型缺少 `logs` 字段（中）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | 数据结构 |
| **位置** | `packages/shared/src/types/job.ts` 第 164-169 行 |

**详情**：
- 任务卡定义：`JobDetailResponse { job: Job; result?: JobResult; stages: JobStage[]; logs: JobLog[] }`
- 前端实现缺少 `logs` 字段

**修复方式**：
```typescript
export interface JobDetailResponse {
  job: Job;
  result?: JobResult;
  stages: JobStage[];
  logs?: JobLog[];
}
```

---

### 问题 #3：`JobTimelineViewer` 中 `stage.status` 与 `STAGE_ICONS` key 不匹配（高）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | 状态流 |
| **位置** | `packages/components/blocks/JobTimelineViewer/index.tsx` 第 11-16 行 |

**详情**：
`JobStage.status` 类型为 `JobStatus = 'pending' | 'running' | 'success' | 'failed'`，但 `STAGE_ICONS` 缺少 `success` 映射。

**修复方式**：
```typescript
const STAGE_ICONS: Record<string, React.ReactNode> = {
  completed: <CheckCircleOutlined style={{ color: colors.success.default }} />,
  success: <CheckCircleOutlined style={{ color: colors.success.default }} />,  // 新增
  failed: <CloseCircleOutlined style={{ color: colors.error.default }} />,
  running: <LoadingOutlined style={{ color: colors.brand.default }} />,
  pending: <ClockCircleOutlined style={{ color: colors.text.muted }} />,
};
```

---

### 问题 #4：`STAGE_LABELS` 使用 `stage.stage` 而非 `stage.status`（低）→ 经确认无需修复

| 维度 | 内容 |
|------|------|
| **类型** | 状态流 |
| **位置** | `packages/components/blocks/JobTimelineViewer/index.tsx` 第 81 行 |

**处理结论**：
`stage.stage` 是 `JobStageType`（阶段类型，如 queued/running/completed），`stage.status` 是 `JobStatus`（执行状态，如 pending/running/success/failed）。时间线展示阶段类型（"排队中"、"执行中"、"已完成"）符合设计意图，无需修改。

---

### 问题 #5：`JobLogViewer` 中 `borderRadius: 6` 硬编码（低）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | Design Token |
| **位置** | `packages/components/blocks/JobLogViewer/index.tsx` 第 77 行 |

**修复方式**：
```tsx
// 修复前
borderRadius: 6,

// 修复后
borderRadius: radius.sm,
```

---

## 🛠 修改文件汇总

### 已修改文件

1. `packages/shared/src/api/services.ts`
   - 日志 API 参数添加 `level?: string`

2. `packages/shared/src/types/job.ts`
   - `JobDetailResponse` 添加 `logs?: JobLog[]`

3. `packages/components/blocks/JobTimelineViewer/index.tsx`
   - `STAGE_ICONS` 添加 `success` 映射

4. `packages/components/blocks/JobLogViewer/index.tsx`
   - 导入 `radius`
   - `borderRadius: 6` → `radius.sm`

---

## ⚠️ 风险评估

| 风险 | 说明 | 影响 |
|------|------|------|
| 是否影响后续任务 | 否。类型调整为补充字段，不影响现有功能 | 无 |
| 是否影响数据模型 | 否。仅前端类型定义补充 | 无 |

---

## ✅ 是否允许进入验收（D2）

**YES**

所有前端问题已修复：
- ✅ 问题 #1（日志 API 缺少 `level` 参数）— 已修复
- ✅ 问题 #2（`JobDetailResponse` 缺少 `logs` 字段）— 已修复
- ✅ 问题 #3（`STAGE_ICONS` 缺少 `success` 映射）— 已修复
- ✅ 问题 #5（`JobLogViewer` borderRadius 硬编码）— 已修复

经确认无需修复：
- ✅ 问题 #4（`STAGE_LABELS` 使用 `stage.stage`）— 符合设计意图

**T016 任务详情页 D1 接口联调检查通过，允许进入 D2 验收。**
