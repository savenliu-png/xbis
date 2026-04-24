# 任务平台（Task Platform）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 任务平台 |
| 任务编号 | FEATURE-202604-004 |
| 所属模块 ⭐ | 用户端 — 任务平台 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 7 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-20 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 "调用记录"（Invocations）概念，仅展示历史调用日志。随着架构向 "任务平台" 迁移，需要一个完整的任务生命周期管理系统，支持任务的创建、执行、监控、审核和结果查看。

### 2.2 目标用户
- 终端用户（开发者）：创建任务、查看执行状态、获取结果
- 终端用户（运营）：监控任务执行、处理审核任务

### 2.3 预期效果
- 用户可以创建异步/同步任务
- 实时查看任务执行状态和时间线
- 支持批量操作和筛选
- 任务失败时可重试或查看错误详情

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 任务列表页 | `/tasks` | 顶部导航「我的任务」 |
| 任务详情页 | `/tasks/:jobId` | 任务列表页点击任务项 |
| 任务创建页 | `/tasks/create` | 任务列表页「新建任务」按钮 |
| 任务监控页 | `/tasks/monitor` | 管理入口（仅管理员） |

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 任务运营 | `/admin/tasks` | 侧边栏「任务运营」 |
| 任务审核 | `/admin/tasks/reviews` | 侧边栏「内容与审核」 |

### 3.3 页面原型/设计稿
- Figma: `https://figma.com/xbis/task-platform`
- 参考实现: `packages/pages/job/JobListTemplate`

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务列表展示（状态、目标 Agent/Skill、执行模式、风险等级）
- [ ] 状态统计栏（全部/执行中/队列中/待审核/已完成/失败）
- [ ] 任务筛选（关键词搜索、状态筛选、时间范围、执行模式）
- [ ] 任务详情查看（基本信息、执行时间线、结果数据）
- [ ] 任务创建（选择能力、填写参数、选择执行模式）
- [ ] 批量操作（批量取消、批量重试、批量导出）
- [ ] 任务监控大盘（管理员：实时任务统计、队列深度、执行器状态）
- [ ] 任务审核（管理员：审核待审核任务、通过/拒绝）

### 4.2 不包含功能（明确排除）
- 任务调度策略配置（管理端高级功能，V2 迭代）
- 任务执行器管理（独立模块）
- 任务告警通知（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 任务列表查询参数
export interface TaskListParams {
  status?: JobStatus | JobStatus[];
  keyword?: string;
  executionMode?: 'sync' | 'async';
  startTime?: string;
  endTime?: string;
  page?: number;
  pageSize?: number;
}

// 任务列表响应
export interface TaskListResponse {
  items: Job[];
  total: number;
  statusStats: { status: JobStatus; count: number; label: string }[];
}

// 任务创建请求
export interface CreateTaskRequest {
  abilityId: string;
  parameters: Record<string, any>;
  executionMode: 'sync' | 'async';
  callbackUrl?: string;
  priority?: 'low' | 'normal' | 'high';
}

// 任务创建响应
export interface CreateTaskResponse {
  success: boolean;
  jobId: string;
  status: JobStatus;
  result?: Record<string, any>;  // sync 模式下直接返回结果
}

// 任务操作请求
export interface TaskActionRequest {
  jobId: string;
  action: 'cancel' | 'retry' | 'approve' | 'reject';
  reason?: string;
}

// 任务时间线事件
export interface TaskTimelineEvent {
  status: JobStatus;
  label: string;
  timestamp: string;
  detail?: string;
  metadata?: Record<string, any>;
}

// 任务监控统计（管理端）
export interface TaskMonitorStats {
  totalToday: number;
  runningNow: number;
  queueDepth: number;
  avgExecutionTime: number;
  successRate: number;
  executorHealth: { executorId: string; status: 'healthy' | 'degraded' | 'down' }[];
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| jobs | ADD | `priority` | ENUM | 任务优先级 low/normal/high |
| jobs | ADD | `callback_url` | VARCHAR(512) | 回调地址 |
| jobs | ADD | `timeline` | JSONB | 执行时间线事件 |
| job_reviews | NEW | — | — | 任务审核记录表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`TaskListParams`, `TaskListResponse`, `CreateTaskRequest`, `TaskTimelineEvent`

## 6. 交互流程

### 6.1 主流程：创建并查看任务
```
[用户访问任务中心] ──► [展示任务列表]
   │
   ├──► [点击「新建任务」] ──► [选择能力]
   │                              │
   │                              └──► [填写参数] ──► [选择执行模式]
   │                                                      │
   │                                                      └──► [提交任务]
   │                                                              │
   │                                                              ├──► [同步] ──► [直接展示结果]
   │                                                              └──► [异步] ──► [返回任务ID]
   │
   └──► [点击任务项] ──► [右侧预览详情]
            │
            ├──► [查看时间线] ──► [追踪执行过程]
            │
            └──► [查看结果] ──► [复制/导出结果]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 任务创建失败 | 参数校验失败 | 阻止提交 | 表单字段级错误提示 |
| 任务执行失败 | 执行器异常 | 标记失败状态 | 展示错误详情和重试按钮 |
| 任务超时 | 执行超过阈值 | 标记超时 | Alert 提示，提供重试 |
| 任务不存在 | 访问已删除任务 | 404 | "该任务不存在或已过期" |
| 无权限操作 | 非任务所有者操作 | 拒绝请求 | "您无权操作此任务" |

### 6.3 边界情况
- 同步任务执行时间 > 30s：自动转为异步，返回 jobId
- 批量操作 > 50 条：分批处理，展示进度
- 任务结果数据 > 1MB：截断展示，提供下载链接
- 队列深度 > 1000：展示排队提示和预计等待时间

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Card | 任务项容器 | 是 |
| Tag | 状态标签 | 是 |
| Button | 操作按钮 | 是 |
| Search | 搜索框 | 是 |
| Empty | 空状态 | 是 |
| Spinner | 加载状态 | 是 |
| Alert | 错误提示 | 是 |
| Modal | 创建任务弹窗 | 是 |
| Table | 批量操作表格 | 是 |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| TaskItem | 任务列表项 | 是 |
| TaskStatusBadge | 任务状态标识 | 是 |
| JobTimeline | 执行时间线 | 是 |
| RiskBadge | 风险等级标识 | 是 |
| ExecutionModeTag | 执行模式标签 | 是 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| TaskFilterBar | 任务筛选栏 | 是 |
| RecentTasks | 最近任务列表 | 是 |
| JobResultViewer | 任务结果查看器 | 是 |
| StatCards | 监控统计卡片 | 是 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| TaskPageShell | 任务列表页骨架 |
| DetailPageShell | 任务详情页骨架 |
| FormPageShell | 任务创建表单页 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| TaskCreateModal | blocks | 创建任务弹窗（选择能力 + 参数表单） |
| TaskMonitorDashboard | blocks | 监控大盘（管理员） |
| BatchActionBar | blocks | 批量操作栏（扩展 AuditActionBar） |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 任务列表查询
- **Method**: GET
- **Path**: `/portal-api/v1/tasks`
- **请求类型**: `TaskListParams`
- **响应类型**: `TaskListResponse`
- **权限**: 用户（已登录，仅查看自己的任务）

#### 任务详情查询
- **Method**: GET
- **Path**: `/portal-api/v1/tasks/:jobId`
- **响应类型**: `Job`
- **权限**: 用户（已登录，仅自己的任务）

#### 任务创建
- **Method**: POST
- **Path**: `/portal-api/v1/tasks`
- **请求类型**: `CreateTaskRequest`
- **响应类型**: `CreateTaskResponse`
- **权限**: 用户（已登录）

#### 任务取消
- **Method**: POST
- **Path**: `/portal-api/v1/tasks/:jobId/cancel`
- **权限**: 用户（已登录，仅自己的任务）

#### 任务重试
- **Method**: POST
- **Path**: `/portal-api/v1/tasks/:jobId/retry`
- **权限**: 用户（已登录，仅自己的任务）

#### 任务审核（管理端）
- **Method**: POST
- **Path**: `/admin-api/v1/tasks/:jobId/review`
- **请求类型**: `{ action: 'approve' | 'reject'; reason?: string }`
- **权限**: 管理员

#### 任务监控统计（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/tasks/monitor/stats`
- **响应类型**: `TaskMonitorStats`
- **权限**: 管理员

### 8.2 修改接口
| 接口 | 变更内容 | 兼容性 |
|------|----------|--------|
| `/portal-api/v1/invocations/*` | 路径迁移为 `/portal-api/v1/tasks/*` | 需同步更新前端调用 |

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.tasks.list`, `userApi.tasks.get`, `userApi.tasks.create`, `userApi.tasks.cancel`, `userApi.tasks.retry`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务列表正确展示所有任务，按时间倒序排列
- [ ] 状态统计栏实时显示各状态任务数量
- [ ] 点击状态标签筛选任务，列表实时更新
- [ ] 关键词搜索支持 jobId、目标 Agent/Skill 模糊匹配
- [ ] 点击任务项右侧展示详情预览（sticky 定位）
- [ ] 详情页展示完整的任务信息、执行时间线、结果数据
- [ ] 时间线正确展示任务状态流转过程
- [ ] 同步任务创建后直接展示结果
- [ ] 异步任务创建后返回 jobId，可后续查询结果
- [ ] 失败任务提供重试按钮，重试后状态重置
- [ ] 批量选择任务后展示批量操作栏（取消/重试/导出）
- [ ] 管理端任务审核支持通过/拒绝操作

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用 `TaskPageShell` 和 `DetailPageShell` 页面模板
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整
- [ ] 定时轮询任务状态（异步任务），支持手动刷新

### 9.3 性能验收
- [ ] 任务列表首屏加载 < 1.5s
- [ ] 状态筛选响应 < 300ms
- [ ] 异步任务状态轮询间隔 3s，页面不可见时暂停轮询
- [ ] 列表支持虚拟滚动（>100 条数据时）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 多栏布局正常
- [ ] 用户端移动端任务列表全宽卡片，详情全屏展开

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [ ] **否** — 全新功能，不影响现有页面
- [x] **是** — 影响范围：
  - 影响页面：原 `/invocations` 页面将被 `/tasks` 替换
  - 影响用户：全部用户
  - 回滚方案：保留原路由 302 重定向到新路由，持续 1 个版本后移除

### 10.2 是否依赖其他任务先完成 ⭐
- [ ] **否** — 可独立开发
- [x] **是** — 依赖任务：
  - [x] `FEATURE-202604-001` — 能力中心 — 已完成
  - [ ] `FEATURE-202604-005` — 后端任务引擎升级 — 开发中
  - [ ] `FEATURE-202604-006` — 执行器管理 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 后端任务引擎接口不稳定 | 中 | 高 | 使用 Mock 数据并行开发，约定接口契约 |
| 异步任务状态同步延迟 | 高 | 中 | 设计合理的轮询策略 + WebSocket 备选方案 |
| 批量操作性能问题 | 中 | 中 | 限制单次批量操作数量（最大 50） |

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

- 本任务为旧调用记录向任务平台迁移的 P0 任务
- 参考实现已存在于 `packages/pages/job/JobListTemplate`
- 任务创建流程依赖能力中心（需先选择能力）
- WebSocket 实时推送为 V2 优化项，P0 版本使用轮询

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
