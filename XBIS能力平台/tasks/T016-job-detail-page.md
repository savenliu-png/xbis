# 任务详情页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 任务详情页 |
| 任务编号 | T016 |
| 所属模块 ⭐ | M5 任务系统 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-12 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏任务详情展示页面，需要：
- 展示任务的完整信息
- 展示执行时间线
- 展示任务结果

### 2.2 目标用户
- 终端用户：查看任务详情
- 开发者：调试任务问题

### 2.3 预期效果
- 提供清晰的任务详情展示
- 支持执行时间线可视化
- 支持结果展示和下载

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 任务详情 | `/jobs/:id` | 任务列表页点击 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/job/JobDetailTemplate`

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务基本信息展示
- [ ] 执行时间线
- [ ] 任务结果展示
- [ ] 任务日志展示
- [ ] 取消/重试操作
- [ ] 结果下载
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 任务创建（由 T017 实现）
- 任务编辑（不支持）
- 任务分享（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 任务详情
export interface JobDetail {
  id: string;
  jobId: string;
  traceId: string;
  requestId: string;
  tenantId: string;
  sourceSystem: string;
  status: JobStatus;
  targetAgent: string;
  targetSkill: string;
  targetAction: string;
  sessionKey: string;
  executionMode: 'async' | 'sync';
  executionTimeoutMs: number;
  policyRiskLevel: 'low' | 'medium' | 'high' | 'critical';
  policyMemoryPolicy: string;
  policyApprovalMode: 'auto' | 'manual';
  payload: Record<string, any>;
  callbackUrl?: string;
  resultSummary?: string;
  createdAt: string;
  updatedAt: string;
  queuedAt?: string;
  runningAt?: string;
  completedAt?: string;
  failedAt?: string;
}

// 执行时间线
export interface JobTimelineEvent {
  id: string;
  stage: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  detail?: string;
  timestamp: string;
  duration?: number;
}

// 任务结果
export interface JobResultDetail {
  jobId: string;
  status: JobStatus;
  resultData?: Record<string, any>;
  governanceData?: Record<string, any>;
  errorCode?: string;
  errorMessage?: string;
  completedAt?: string;
}

// 任务日志
export interface JobLogItem {
  id: string;
  level: 'info' | 'warn' | 'error';
  message: string;
  timestamp: string;
  metadata?: Record<string, any>;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`JobDetail`, `JobTimelineEvent`, `JobResultDetail`, `JobLogItem`

## 6. 交互流程

### 6.1 主流程
```
[用户点击任务] ──► [加载任务详情] ──► [展示信息] ──► [用户查看时间线/结果/日志]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 任务不存在 | 无效的 jobId | 返回 404 | 提示任务不存在 |
| 取消失败 | 任务已完成 | 返回错误 | 提示状态不允许 |
| 重试失败 | 任务未失败 | 返回错误 | 提示状态不允许 |

### 6.3 边界情况
- 长时间任务：支持实时更新
- 大量日志：支持分页和搜索
- 大结果数据：支持折叠和下载
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 信息卡片 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Tabs | 内容切换 | 是（T002） |
| Timeline | 时间线 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatusBadge | 状态标识 | 是（T003） |
| TaskItem | 任务项 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| JobInfo | 任务信息区 | 否 |
| TimelineViewer | 时间线区 | 否 |
| ResultViewer | 结果展示区 | 否 |
| LogViewer | 日志查看区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| JobInfo | blocks | 任务信息区 |
| TimelineViewer | blocks | 时间线区 |
| ResultViewer | blocks | 结果展示区 |
| LogViewer | blocks | 日志查看区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 任务详情查询
- **Method**: GET
- **Path**: `/api/v1/jobs/:id`
- **响应类型**: `JobDetailResponse`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "job": {
      "id": "job-001",
      "jobId": "job-20260424-001",
      "traceId": "trace-abc123",
      "status": "completed",
      "targetAgent": "agent-001",
      "targetSkill": "text-generation",
      "targetAction": "generate",
      "executionMode": "async",
      "payload": { "text": "Hello" },
      "createdAt": "2026-04-24T10:00:00Z",
      "completedAt": "2026-04-24T10:00:05Z"
    },
    "stages": [
      { "stage": "queued", "status": "completed", "timestamp": "2026-04-24T10:00:00Z" },
      { "stage": "running", "status": "completed", "timestamp": "2026-04-24T10:00:01Z", "duration": 4000 },
      { "stage": "completed", "status": "completed", "timestamp": "2026-04-24T10:00:05Z" }
    ],
    "result": {
      "jobId": "job-001",
      "status": "completed",
      "resultData": { "result": "你好" },
      "completedAt": "2026-04-24T10:00:05Z"
    }
  }
}
```

#### 任务日志查询
- **Method**: GET
- **Path**: `/api/v1/jobs/:id/logs`
- **响应类型**: `JobLogsResponse`
- **权限**: 用户（已登录）

#### 任务取消
- **Method**: POST
- **Path**: `/api/v1/jobs/:id/cancel`
- **响应类型**: `JobCancelResponse`
- **权限**: 用户（已登录）

#### 任务重试
- **Method**: POST
- **Path**: `/api/v1/jobs/:id/retry`
- **响应类型**: `JobRetryResponse`
- **权限**: 用户（已登录）

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.jobs.get`, `userApi.jobs.logs`, `userApi.jobs.cancel`, `userApi.jobs.retry`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务基本信息展示完整
- [ ] 执行时间线展示正常
- [ ] 任务结果展示正常
- [ ] 任务日志展示正常
- [ ] 取消操作正常
- [ ] 重试操作正常
- [ ] 结果下载正常
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
- [ ] 日志查询 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T015` — 任务列表页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 大量日志 | 中 | 中 | 分页加载 |
| 大结果数据 | 中 | 中 | 折叠展示 + 下载 |
| 状态实时变化 | 中 | 低 | 手动刷新 + 轮询（T018） |

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

- 参考实现：`packages/pages/job/JobDetailTemplate`
- 日志支持实时更新（由 T018 实现）
- 结果数据支持 JSON 下载

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
