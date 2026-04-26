# T009 job API 层 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: 缺少任务列表筛选参数
**严重程度: 中**

任务列表需要支持按状态、能力、时间范围筛选，但 `JobListParams` 未定义。

### 问题2: 缺少批量操作 API
**严重程度: 低**

管理端任务运营需要批量取消/重试功能，但 API 中未定义批量操作接口。

### 问题3: 任务创建请求的 abilityId 字段未明确
**严重程度: 低**

`JobCreateRequest` 中应明确包含 `abilityId` 和输入参数 `payload`。

---

## 修改建议

### 建议1: 补充筛选参数（必须修改）
```typescript
interface JobListParams {
  status?: JobStatus;
  abilityId?: string;
  keyword?: string;
  startDate?: string;
  endDate?: string;
  page?: number;
  pageSize?: number;
}
```

### 建议2: 明确 JobCreateRequest 结构（必须修改）
```typescript
interface JobCreateRequest {
  abilityId: string;
  payload: Record<string, unknown>;
  callbackUrl?: string;
}
```

### 建议3: 补充批量操作 API（建议修改）
```typescript
export const adminJobApi = {
  batchCancel: (ids: string[]) => apiClient.post('/admin-api/v1/jobs/batch-cancel', { ids }),
  batchRetry: (ids: string[]) => apiClient.post('/admin-api/v1/jobs/batch-retry', { ids }),
};
```

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 补充筛选参数（建议1）
2. 明确 JobCreateRequest 结构（建议2）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | N/A | 无组件 |
| 页面模板使用 | N/A | 无页面 |
| 重复组件 | N/A | 无组件 |
| 分层规范 | ✅ | 符合 API 分层 |
| API 合理性 | ⚠️ | 缺少筛选参数 |
| 状态设计完整性 | N/A | 无状态 |
| 能力平台架构 | ✅ | 符合 |
| 过度设计 | ✅ | 不过度 |
| 未声明数据模型 | ⚠️ | JobCreateRequest 不完整 |

**风险等级**: 低
