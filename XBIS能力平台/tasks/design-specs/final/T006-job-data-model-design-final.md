# T006 job 数据模型 — 最终可开发设计方案

> 任务卡: [T006-job-data-model.md](../T006-job-data-model.md)
> 设计输入: docs/dev-rules.md, docs/component-architecture.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为**数据层**建设，无页面结构。输出物为 TypeScript 类型定义 + 数据库 Schema 文档。

---

## 2. 组件拆分

无 UI 组件。本任务输出为纯类型定义。

### 类型定义清单

| 类型 | 文件 | 说明 |
|------|------|------|
| Job | `types/job.ts` | 任务实体 |
| JobStatus | `types/job.ts` | 任务状态枚举 |
| JobResult | `types/job.ts` | 任务结果 |
| JobStage | `types/job.ts` | 任务阶段 |
| JobLog | `types/job.ts` | 任务日志 |
| JobListParams | `types/job.ts` | 列表查询参数 |
| JobListResponse | `types/job.ts` | 列表响应 |
| JobDetailResponse | `types/job.ts` | 详情响应 |
| JobCreateRequest | `types/job.ts` | 创建请求 |
| JobCancelResponse | `types/job.ts` | 取消响应 |
| JobRetryResponse | `types/job.ts` | 重试响应 |

---

## 3. 数据流

无运行时数据流。类型定义为编译时静态检查。

---

## 4. 状态管理

无状态管理需求。

---

## 5. API 调用

无 API 调用。本任务为类型定义。

---

## 6. 用户交互流程

无用户交互。

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 状态机转换非法 | 使用 TypeScript 联合类型限制 |
| 字段类型冲突 | 使用联合类型或泛型处理 |
| 可选字段缺失 | 使用 `?` 标记可选字段 |

---

## 8. 性能优化

- 类型定义仅在编译时使用，无运行时开销

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与后端 Schema 不一致 | 高 | 类型定义可能滞后于后端变更 | 建立类型同步机制 |
| 状态机复杂 | 中 | JobStatus 状态转换可能复杂 | 定义明确的状态转换规则 |
| 与 invocation 模型冲突 | 高 | 新旧模型可能并存 | 明确迁移策略 |

---

## 10. 开发步骤拆分

### Step 1: 核心类型定义（0.5 人日）
- [ ] Job 接口（补充 abilityId/abilityName）
- [ ] JobStatus 枚举
- [ ] 状态机转换规则

### Step 2: API 类型定义（0.5 人日）
- [ ] JobListParams 类型
- [ ] JobListResponse 类型
- [ ] JobDetailResponse 类型
- [ ] JobCreateRequest 类型
- [ ] JobCancelResponse 类型
- [ ] JobRetryResponse 类型

### Step 3: 数据库 Schema 文档（0.5 人日）
- [ ] job 表 Schema
- [ ] job_result 表 Schema
- [ ] job_stage 表 Schema
- [ ] job_log 表 Schema

### Step 4: 类型导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写类型使用文档

---

## 附录: Job 类型定义

```typescript
interface Job {
  id: string;
  abilityId: string;
  abilityName: string;
  status: JobStatus;
  payload: Record<string, unknown>;
  result?: Record<string, unknown>;
  errorMessage?: string;
  createdAt: string;
  startedAt?: string;
  completedAt?: string;
  createdBy: string;
}

type JobStatus = 'pending' | 'running' | 'completed' | 'failed' | 'retrying' | 'cancelled';

// 状态机转换规则
const validStatusTransitions: Record<JobStatus, JobStatus[]> = {
  pending: ['running', 'cancelled'],
  running: ['completed', 'failed', 'cancelled'],
  completed: [],
  failed: ['retrying'],
  retrying: ['running'],
  cancelled: []
};

// 状态转换校验函数
function canTransition(from: JobStatus, to: JobStatus): boolean {
  return validStatusTransitions[from]?.includes(to) ?? false;
}
```
