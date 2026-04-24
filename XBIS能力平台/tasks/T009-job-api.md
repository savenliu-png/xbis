# job API 层

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | job API 层 |
| 任务编号 | T009 |
| 所属模块 ⭐ | M3 API 层 |
| 优先级 | P0 |
| 指派给 | @backend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-05 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 invocation 相关接口，但随着架构向任务平台迁移，需要：
- 新的 job 相关接口
- 支持任务状态查询、取消、重试
- 支持任务结果查询

### 2.2 目标用户
- 前端开发者：调用 API 获取任务数据
- 后端开发者：维护 API 接口

### 2.3 预期效果
- 提供完整的 job CRUD 接口
- 支持任务状态轮询
- 兼容旧 invocation 接口

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`packages/shared/src/api/services.ts`

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务列表查询（/api/v1/jobs）
- [ ] 任务详情查询（/api/v1/jobs/:id）
- [ ] 任务创建（/api/v1/jobs）
- [ ] 任务取消（/api/v1/jobs/:id/cancel）
- [ ] 任务重试（/api/v1/jobs/:id/retry）
- [ ] 任务结果查询（/api/v1/jobs/:id/result）
- [ ] 任务日志查询（/api/v1/jobs/:id/logs）

### 4.2 不包含功能（明确排除）
- 任务调度逻辑（由调度器实现）
- 任务执行逻辑（由执行器实现）
- 任务回调机制（由 T034 实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 任务状态枚举
export type JobStatus = 
  | 'pending' 
  | 'queued' 
  | 'running' 
  | 'paused'
  | 'completed' 
  | 'failed' 
  | 'cancelled' 
  | 'timeout';

// 任务列表查询参数
export interface JobListParams {
  status?: JobStatus | JobStatus[];
  abilityId?: string;
  keyword?: string;
  dateFrom?: string;
  dateTo?: string;
  page?: number;
  pageSize?: number;
}

// 任务列表响应
export interface JobListResponse {
  items: Job[];
  total: number;
  statusStats: { status: JobStatus; count: number }[];
}

// 任务详情响应
export interface JobDetailResponse {
  job: Job;
  result?: JobResult;
  stages: JobStage[];
}

// 任务创建请求
export interface JobCreateRequest {
  abilityId: string;
  payload: Record<string, any>;
  executionMode?: 'async' | 'sync';
  callbackUrl?: string;
  timeout?: number;
}

// 任务创建响应
export interface JobCreateResponse {
  jobId: string;
  traceId: string;
  status: JobStatus;
  estimatedDuration?: number;
}

// 任务取消响应
export interface JobCancelResponse {
  jobId: string;
  status: JobStatus;
  cancelledAt: string;
}

// 任务重试响应
export interface JobRetryResponse {
  jobId: string;
  newJobId: string;
  status: JobStatus;
}

// 任务结果响应
export interface JobResultResponse {
  jobId: string;
  status: JobStatus;
  resultData?: Record<string, any>;
  governanceData?: Record<string, any>;
  errorCode?: string;
  errorMessage?: string;
  completedAt?: string;
}

// 任务日志响应
export interface JobLogsResponse {
  jobId: string;
  logs: JobLog[];
  total: number;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`JobListParams`, `JobListResponse`, `JobDetailResponse`, `JobCreateRequest`, `JobCreateResponse`, `JobCancelResponse`, `JobRetryResponse`, `JobResultResponse`, `JobLogsResponse`

## 6. 交互流程

### 6.1 主流程
```
[前端请求 API] ──► [后端校验参数] ──► [查询/操作数据库] ──► [返回数据]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 参数错误 | 缺少必填参数 | 返回 400 | 提示参数错误 |
| 任务不存在 | 查询不存在的 jobId | 返回 404 | 提示任务不存在 |
| 状态不允许 | 取消已完成的任务 | 返回 409 | 提示状态冲突 |
| 权限不足 | 无权限访问 | 返回 403 | 提示权限不足 |

### 6.3 边界情况
- 大数据量：支持分页和缓存
- 高并发：支持限流和降级
- 状态机校验：确保状态转换合法

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

### 8.1 新增接口

#### 任务列表查询
- **Method**: GET
- **Path**: `/api/v1/jobs`
- **请求类型**: `JobListParams`
- **响应类型**: `JobListResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "status": ["running", "pending"],
  "abilityId": "ability-001",
  "keyword": "测试",
  "dateFrom": "2026-04-01T00:00:00Z",
  "dateTo": "2026-04-30T23:59:59Z",
  "page": 1,
  "pageSize": 20
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 256,
    "statusStats": [
      { "status": "pending", "count": 12 },
      { "status": "running", "count": 5 },
      { "status": "completed", "count": 200 },
      { "status": "failed", "count": 39 }
    ]
  }
}
```

#### 任务详情查询
- **Method**: GET
- **Path**: `/api/v1/jobs/:id`
- **响应类型**: `JobDetailResponse`
- **权限**: 用户（已登录）

#### 任务创建
- **Method**: POST
- **Path**: `/api/v1/jobs`
- **请求类型**: `JobCreateRequest`
- **响应类型**: `JobCreateResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "abilityId": "ability-001",
  "payload": {
    "text": "Hello World",
    "language": "zh"
  },
  "executionMode": "async",
  "callbackUrl": "https://example.com/callback",
  "timeout": 30000
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "jobId": "job-20260424-001",
    "traceId": "trace-abc123",
    "status": "pending",
    "estimatedDuration": 5000
  }
}
```

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

#### 任务结果查询
- **Method**: GET
- **Path**: `/api/v1/jobs/:id/result`
- **响应类型**: `JobResultResponse`
- **权限**: 用户（已登录）

#### 任务日志查询
- **Method**: GET
- **Path**: `/api/v1/jobs/:id/logs`
- **响应类型**: `JobLogsResponse`
- **权限**: 用户（已登录）

### 8.2 修改接口
| 接口 | 变更内容 | 兼容性 |
|------|----------|--------|
| `/portal-api/v1/invocations/*` | 路径迁移为 `/api/v1/jobs/*` | 保留旧端点，内部转发 |

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.jobs.list`, `userApi.jobs.get`, `userApi.jobs.create`, `userApi.jobs.cancel`, `userApi.jobs.retry`, `userApi.jobs.result`, `userApi.jobs.logs`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务列表查询支持状态/能力/关键词/日期筛选
- [ ] 任务详情查询返回完整信息（含阶段和结果）
- [ ] 任务创建正常，返回 jobId 和 traceId
- [ ] 任务取消正常，仅允许取消 pending/running 状态
- [ ] 任务重试正常，创建新 job 并关联原 job
- [ ] 任务结果查询正常，completed 返回结果，failed 返回错误
- [ ] 任务日志查询正常，支持分页
- [ ] 旧接口兼容，内部转发到新接口

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 接口参数校验完整
- [ ] 错误处理规范
- [ ] 接口文档（Swagger）生成

### 9.3 性能验收
- [ ] 列表查询 < 300ms
- [ ] 详情查询 < 200ms
- [ ] 创建任务 < 500ms
- [ ] 支持缓存

### 9.4 兼容性验收
- [ ] 旧接口可正常访问
- [ ] 新旧接口数据一致

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 新接口，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T006` — job 数据模型 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 旧接口兼容问题 | 中 | 高 | 保留旧端点，内部转发 |
| 状态机复杂 | 中 | 中 | 严格校验状态转换 |
| 并发创建冲突 | 低 | 中 | 使用分布式锁 |

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

- 旧接口保留 1 个版本后移除
- 新接口使用 RESTful 规范
- 建议添加接口版本控制
- 状态机转换需严格校验

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
