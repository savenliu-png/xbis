# 管理端任务运营

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端任务运营 |
| 任务编号 | T029 |
| 所属模块 ⭐ | M12 任务运营 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 4 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 `/admin/reviews` 页面，但需要增强以支持：
- 任务列表审核
- 任务重试
- 退款处理
- SLA 监控

### 2.2 目标用户
- 管理员：审核任务
- 运营人员：处理退款
- 运维人员：监控 SLA

### 2.3 预期效果
- 提供完整的任务运营页面
- 支持审核和重试
- 支持退款和 SLA 监控

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 任务运营 | `/admin/operations` | 管理端导航「任务运营」 |

### 3.3 页面原型/设计稿
- 参考：现有 `/admin/reviews` 页面

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务列表（全量）
- [ ] 任务审核（通过/拒绝）
- [ ] 任务重试
- [ ] 退款处理
- [ ] SLA 监控
- [ ] 批量操作
- [ ] 任务详情查看
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 自动审核（V2 迭代）
- 批量退款（V2 迭代）
- 报表导出（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 运营任务项
export interface OperationJobItem {
  id: string;
  jobId: string;
  userId: string;
  username: string;
  abilityId: string;
  abilityName: string;
  status: JobStatus;
  policyRiskLevel: 'low' | 'medium' | 'high' | 'critical';
  policyApprovalMode: 'auto' | 'manual';
  executionMode: 'async' | 'sync';
  payload: Record<string, any>;
  resultSummary?: string;
  createdAt: string;
  updatedAt: string;
  slaStatus?: 'met' | 'breached' | 'pending';
  slaDeadline?: string;
}

// 审核请求
export interface JobReviewRequest {
  jobId: string;
  action: 'approve' | 'reject';
  reason?: string;
}

// 退款请求
export interface JobRefundRequest {
  jobId: string;
  amount: number;
  reason: string;
}

// SLA 统计
export interface SlaStats {
  totalJobs: number;
  metJobs: number;
  breachedJobs: number;
  pendingJobs: number;
  averageResponseTime: number;
  slaPercentage: number;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`OperationJobItem`, `JobReviewRequest`, `JobRefundRequest`, `SlaStats`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入任务运营] ──► [加载任务列表] ──► [展示数据] ──► [管理员审核/重试/退款]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 审核失败 | 任务状态已变 | 返回错误 | 提示刷新 |
| 退款失败 | 金额超限 | 返回错误 | 提示调整 |
| SLA 告警 | 任务即将超时 | 标记告警 | 提示优先处理 |

### 6.3 边界情况
- 大量任务：支持分页和筛选
- 批量操作：支持批量审核
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 搜索框 | 是（T002） |
| Table | 任务表格 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatusBadge | 状态标识 | 是（T003） |
| TaskItem | 任务项 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| OperationTable | 运营表格 | 否 |
| ReviewPanel | 审核面板 | 否 |
| RefundPanel | 退款面板 | 否 |
| SlaDashboard | SLA 面板 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AdminPageShell | 管理页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| OperationTable | blocks | 运营表格 |
| ReviewPanel | blocks | 审核面板 |
| RefundPanel | blocks | 退款面板 |
| SlaDashboard | blocks | SLA 面板 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 运营任务列表查询
- **Method**: GET
- **Path**: `/admin-api/v1/operations/jobs`
- **请求类型**: `JobListParams`
- **响应类型**: `{ items: OperationJobItem[]; total: number }`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "job-001",
        "jobId": "job-20260424-001",
        "userId": "user-001",
        "username": "developer",
        "abilityId": "text-generation",
        "abilityName": "文本生成",
        "status": "pending",
        "policyRiskLevel": "high",
        "policyApprovalMode": "manual",
        "executionMode": "async",
        "payload": { "text": "Hello" },
        "createdAt": "2026-04-24T10:00:00Z",
        "updatedAt": "2026-04-24T10:00:00Z",
        "slaStatus": "pending",
        "slaDeadline": "2026-04-24T10:05:00Z"
      }
    ],
    "total": 256
  }
}
```

#### 任务审核
- **Method**: POST
- **Path**: `/admin-api/v1/operations/jobs/:id/review`
- **请求类型**: `JobReviewRequest`
- **响应类型**: `OperationJobItem`
- **权限**: 管理员

#### 任务重试
- **Method**: POST
- **Path**: `/admin-api/v1/operations/jobs/:id/retry`
- **响应类型**: `JobRetryResponse`
- **权限**: 管理员

#### 退款处理
- **Method**: POST
- **Path**: `/admin-api/v1/operations/jobs/:id/refund`
- **请求类型**: `JobRefundRequest`
- **响应类型**: `{ refundId: string; status: string }`
- **权限**: 管理员

#### SLA 统计查询
- **Method**: GET
- **Path**: `/admin-api/v1/operations/sla`
- **响应类型**: `SlaStats`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.operations.jobs`, `adminApi.operations.review`, `adminApi.operations.retry`, `adminApi.operations.refund`, `adminApi.operations.sla`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务列表展示正常（全量）
- [ ] 任务审核正常（通过/拒绝）
- [ ] 任务重试正常
- [ ] 退款处理正常
- [ ] SLA 监控正常
- [ ] 批量操作正常
- [ ] 任务详情查看正常
- [ ] 空态/加载态/错误态完整

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用页面模板骨架
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 首屏加载 < 2s
- [ ] 列表页支持虚拟滚动（>100 条数据时）
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 管理端无需移动端设计

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **是** — 影响范围：原 `/admin/reviews` 增强
  - 影响页面：`/admin/reviews`
  - 影响用户：管理员
  - 回滚方案：保留旧页面，逐步切换

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发
  - [x] `T009` — job API 层 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 误操作 | 中 | 高 | 二次确认 |
| 数据量大 | 中 | 中 | 分页加载 |
| 并发操作 | 低 | 中 | 乐观锁 |

## 11. 开发检查清单

### 11.1 开发前
- [ ] 需求已评审并确认
- [ ] 设计稿已确认
- [ ] 数据结构和 API 已评审
- [ ] 依赖任务已完成

### 11.2 开发中
- [ ] 遵循 `docs/dev-rules.md` 规范
- [ ] 遵循 `AI_RULES.md` 规范
- [ ] 自测覆盖所有验收标准

### 11.3 开发后
- [ ] 代码已提交 PR
- [ ] PR 已通过 Code Review
- [ ] 已联调通过
- [ ] 已更新文档（如需要）

## 12. 备注

- 改造现有 `/admin/reviews` 页面
- 保留旧页面，逐步切换
- SLA 告警需优先处理

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
