# 任务列表页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 任务列表页 |
| 任务编号 | T015 |
| 所属模块 ⭐ | M5 任务系统 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-10 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 invocation 页面展示任务，但随着架构向任务平台迁移，需要：
- 新的任务列表页面
- 支持状态统计和筛选
- 支持批量操作

### 2.2 目标用户
- 终端用户：查看和管理任务
- 开发者：监控任务状态

### 2.3 预期效果
- 提供清晰的任务列表展示
- 支持多种筛选和排序
- 支持批量操作

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 任务列表 | `/jobs` | 主导航「任务中心」 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/job/JobListTemplate`

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务列表展示
- [ ] 状态统计（顶部卡片）
- [ ] 状态筛选
- [ ] 能力筛选
- [ ] 日期筛选
- [ ] 关键词搜索
- [ ] 排序（时间/状态）
- [ ] 批量操作（取消/删除）
- [ ] 分页
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 任务详情（由 T016 实现）
- 任务创建（由 T017 实现）
- 任务状态轮询（由 T018 实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 任务列表项
export interface JobListItem {
  id: string;
  jobId: string;
  traceId: string;
  abilityId: string;
  abilityName: string;
  status: JobStatus;
  executionMode: 'async' | 'sync';
  policyRiskLevel: 'low' | 'medium' | 'high' | 'critical';
  resultSummary?: string;
  createdAt: string;
  updatedAt: string;
  completedAt?: string;
  failedAt?: string;
}

// 任务筛选条件
export interface JobFilterState {
  status?: JobStatus | JobStatus[];
  abilityId?: string;
  keyword: string;
  dateFrom?: string;
  dateTo?: string;
  sortBy: 'createdAt' | 'updatedAt' | 'status';
  sortOrder: 'asc' | 'desc';
}

// 状态统计
export interface JobStatusStats {
  total: number;
  pending: number;
  running: number;
  completed: number;
  failed: number;
  cancelled: number;
}

// 批量操作请求
export interface JobBatchRequest {
  jobIds: string[];
  action: 'cancel' | 'retry' | 'delete';
}

// 批量操作响应
export interface JobBatchResponse {
  success: string[];
  failed: { jobId: string; reason: string }[];
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`JobListItem`, `JobFilterState`, `JobStatusStats`, `JobBatchRequest`, `JobBatchResponse`

## 6. 交互流程

### 6.1 主流程
```
[用户进入任务列表] ──► [加载状态统计] ──► [加载任务列表] ──► [用户筛选/搜索] ──► [更新列表]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无数据 | 无任务记录 | 显示空态 | 提示创建任务 |
| 批量操作失败 | 部分任务状态不允许 | 标记失败项 | 提示具体原因 |
| 搜索无结果 | 关键词无匹配 | 显示空态 | 提示更换关键词 |

### 6.3 边界情况
- 大量任务：支持虚拟滚动和分页
- 状态实时变化：支持手动刷新
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 搜索框 | 是（T002） |
| Card | 统计卡片 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Checkbox | 批量选择 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |
| DatePicker | 日期筛选 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| TaskItem | 任务项 | 是（T003） |
| StatusBadge | 状态标识 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| JobList | 任务列表 | 否 |
| StatusStats | 状态统计 | 否 |
| FilterBar | 筛选栏 | 否 |
| BatchActionBar | 批量操作栏 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| ListPageShell | 列表页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| JobList | blocks | 任务列表 |
| StatusStats | blocks | 状态统计区 |
| FilterBar | blocks | 筛选栏 |
| BatchActionBar | blocks | 批量操作栏 |

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

#### 批量操作
- **Method**: POST
- **Path**: `/api/v1/jobs/batch`
- **请求类型**: `JobBatchRequest`
- **响应类型**: `JobBatchResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "jobIds": ["job-001", "job-002"],
  "action": "cancel"
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "success": ["job-001"],
    "failed": [
      { "jobId": "job-002", "reason": "任务已完成，无法取消" }
    ]
  }
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.jobs.list`, `userApi.jobs.batch`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务列表展示正常，信息完整
- [ ] 状态统计展示正常
- [ ] 状态筛选正常
- [ ] 能力筛选正常
- [ ] 日期筛选正常
- [ ] 关键词搜索正常
- [ ] 排序功能正常
- [ ] 批量操作正常（取消/删除）
- [ ] 分页功能正常
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
- [ ] 搜索响应 < 300ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发
  - [x] `T009` — job API 层 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 大量数据渲染 | 中 | 中 | 虚拟滚动 + 分页 |
| 状态实时变化 | 中 | 低 | 手动刷新 + 轮询（T018） |
| 批量操作复杂 | 低 | 中 | 状态校验 + 错误提示 |

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

- 参考实现：`packages/pages/job/JobListTemplate`
- 状态统计实时更新（由 T018 实现）
- 批量操作需校验任务状态

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
