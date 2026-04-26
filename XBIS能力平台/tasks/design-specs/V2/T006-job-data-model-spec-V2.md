# T006 job 数据模型 — 修正版设计方案（V2）

> 原设计方案: [T006-job-data-model-spec.md](../T006-job-data-model-spec.md)
> 评审报告: [T006-job-data-model-Reviewer.md](../T006-job-data-model-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed

---

## 二、必须修改项（Blocking）

无必须修改项。

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 定义状态机转换规则 | 第 4 节状态管理 |
| 2 | 明确 Job 与 Ability 关联 | 第 4 节状态管理 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为数据模型定义。

---

### 2. 组件拆分

无组件拆分，本任务为数据模型定义。

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

#### Job 类型定义（V2 补充）【已修复】

```typescript
interface Job {
  id: string;
  abilityId: string; // 【新增：关联能力】
  abilityName: string; // 【新增：能力名称（冗余，便于展示）】
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

// 【新增】状态机转换规则
const validStatusTransitions: Record<JobStatus, JobStatus[]> = {
  pending: ['running', 'cancelled'],
  running: ['completed', 'failed', 'cancelled'],
  completed: [],
  failed: ['retrying'],
  retrying: ['running'],
  cancelled: []
};

// 【新增】状态转换校验函数
function canTransition(from: JobStatus, to: JobStatus): boolean {
  return validStatusTransitions[from]?.includes(to) ?? false;
}
```

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

无边界与异常。

---

### 8. 性能优化

无性能优化。

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 状态机违规 | 中 | 非法状态转换 | 使用 `canTransition` 校验 |

---

### 10. 开发步骤拆分

#### Step 1: 核心类型定义（0.5 人日）
- [ ] Job 接口（补充 abilityId/abilityName）
- [ ] JobStatus 枚举
- [ ] 状态机转换规则 【新增】

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **Ability 关联** | 未明确 | 新增 `abilityId` 和 `abilityName` |
| **状态机规则** | 未定义 | 新增 `validStatusTransitions` 和 `canTransition` |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 原设计已通过评审
2. 建议修改项已补充（状态机规则、Ability 关联）
3. 风险等级：低
