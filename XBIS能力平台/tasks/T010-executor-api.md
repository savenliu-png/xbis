# executor API 层

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | executor API 层 |
| 任务编号 | T010 |
| 所属模块 ⭐ | M3 API 层 |
| 优先级 | P1 |
| 指派给 | @backend-dev |
| 预计工期 | 2 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-10 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏执行器管理的 API 接口，需要：
- 执行器注册、查询、配置接口
- 健康检查数据查询接口
- 执行器与能力绑定接口

### 2.2 目标用户
- 前端开发者：调用 API 获取执行器数据
- 后端开发者：维护 API 接口
- 运维人员：监控执行器健康状态

### 2.3 预期效果
- 提供完整的 executor CRUD 接口
- 支持健康检查数据查询
- 支持执行器与能力绑定

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`packages/shared/src/api/services.ts`

## 4. 功能范围

### 4.1 包含功能
- [ ] 执行器列表查询（/admin-api/v1/executors）
- [ ] 执行器详情查询（/admin-api/v1/executors/:id）
- [ ] 执行器创建（/admin-api/v1/executors）
- [ ] 执行器更新（/admin-api/v1/executors/:id）
- [ ] 执行器删除（/admin-api/v1/executors/:id）
- [ ] 执行器健康检查（/admin-api/v1/executors/:id/health）
- [ ] 执行器绑定能力（/admin-api/v1/executors/:id/bindings）

### 4.2 不包含功能（明确排除）
- 执行器调度逻辑（由调度器实现）
- 执行器监控告警（由监控系统实现）
- 执行器自动扩缩容（由运维系统实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 执行器列表查询参数
export interface ExecutorListParams {
  status?: 'active' | 'inactive' | 'degraded' | 'offline';
  type?: 'docker' | 'kubernetes' | 'vm' | 'serverless';
  keyword?: string;
  pageNo?: number;
  pageSize?: number;
}

// 执行器列表响应
export interface ExecutorListResponse {
  items: Executor[];
  total: number;
  pageNo: number;
  pageSize: number;
  /** 状态统计，后端支持时返回 */
  statusStats?: { status: string; count: number }[];
  /** 类型统计，后端支持时返回 */
  typeStats?: { type: string; count: number }[];
}

// 执行器详情响应
export interface ExecutorDetailResponse {
  executor: Executor;
  health?: ExecutorHealth;
  bindings: AbilityExecutorBinding[];
}

// 执行器创建请求
export interface ExecutorCreateRequest {
  name: string;
  description?: string;
  type: 'docker' | 'kubernetes' | 'vm' | 'serverless';
  endpoint: string;
  /** 能力列表，后端支持默认值 */
  capabilities?: string[];
  /** 最大并发数，后端支持默认值 */
  maxConcurrency?: number;
  version: string;
  metadata?: Record<string, unknown>;
}

// 执行器更新请求
export interface ExecutorUpdateRequest {
  name?: string;
  description?: string;
  endpoint?: string;
  capabilities?: string[];
  maxConcurrency?: number;
  version?: string;
  status?: 'active' | 'inactive' | 'degraded' | 'offline';
  metadata?: Record<string, unknown>;
}

// 执行器健康响应
export interface ExecutorHealthResponse {
  executorId: string;
  status: 'healthy' | 'degraded' | 'unhealthy';
  cpuUsage: number;
  memoryUsage: number;
  diskUsage: number;
  networkLatency: number;
  activeJobs: number;
  queuedJobs: number;
  errorRate: number;
  message?: string;
  checkedAt: string;
  history?: ExecutorHealth[];
}

// 执行器绑定请求
export interface ExecutorBindingRequest {
  abilityId: string;
  /** 优先级，后端支持默认值 */
  priority?: number;
  /** 权重，后端支持默认值 */
  weight?: number;
  /** 是否默认，后端支持默认值 */
  isDefault?: boolean;
}

// 执行器绑定响应
export interface ExecutorBindingResponse {
  bindingId: string;
  executorId: string;
  abilityId: string;
  priority: number;
  weight: number;
  isDefault: boolean;
  createdAt: string;
  updatedAt: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`ExecutorListParams`, `ExecutorListResponse`, `ExecutorDetailResponse`, `ExecutorCreateRequest`, `ExecutorUpdateRequest`, `ExecutorHealthResponse`, `ExecutorBindingRequest`, `ExecutorBindingResponse`

## 6. 交互流程

### 6.1 主流程
```
[前端请求 API] ──► [后端校验参数] ──► [查询/操作数据库] ──► [返回数据]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 参数错误 | 缺少必填参数 | 返回 400 | 提示参数错误 |
| 执行器不存在 | 查询不存在的 executorId | 返回 404 | 提示执行器不存在 |
| 绑定冲突 | 同一能力绑定多个默认执行器 | 返回 409 | 提示绑定冲突 |
| 权限不足 | 无权限访问 | 返回 403 | 提示权限不足 |

### 6.3 边界情况
- 健康检查数据过期：自动清理 30 天前数据
- 执行器离线：保留历史记录，标记状态
- 绑定关系循环依赖：绑定前校验

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

### 8.1 新增接口

#### 执行器列表查询
- **Method**: GET
- **Path**: `/admin-api/v1/executors`
- **请求类型**: `ExecutorListParams`
- **响应类型**: `ExecutorListResponse`
- **权限**: 管理员

**请求示例**:
```json
{
  "status": "active",
  "type": "docker",
  "keyword": "gpu",
  "pageNo": 1,
  "pageSize": 20
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 12,
    "statusStats": [
      { "status": "active", "count": 8 },
      { "status": "inactive", "count": 2 },
      { "status": "degraded", "count": 1 },
      { "status": "offline", "count": 1 }
    ],
    "typeStats": [
      { "type": "docker", "count": 6 },
      { "type": "kubernetes", "count": 4 },
      { "type": "serverless", "count": 2 }
    ]
  }
}
```

#### 执行器详情查询
- **Method**: GET
- **Path**: `/admin-api/v1/executors/:id`
- **响应类型**: `ExecutorDetailResponse`
- **权限**: 管理员

#### 执行器创建
- **Method**: POST
- **Path**: `/admin-api/v1/executors`
- **请求类型**: `ExecutorCreateRequest`
- **响应类型**: `Executor`
- **权限**: 管理员

**请求示例**:
```json
{
  "name": "GPU-Cluster-01",
  "description": "GPU 推理集群",
  "type": "kubernetes",
  "endpoint": "https://k8s-gpu.example.com",
  "capabilities": ["ai", "image-generation"],
  "maxConcurrency": 100,
  "version": "v2.1.0",
  "metadata": {
    "region": "cn-north-1",
    "gpuType": "A100"
  }
}
```

#### 执行器更新
- **Method**: PUT
- **Path**: `/admin-api/v1/executors/:id`
- **请求类型**: `ExecutorUpdateRequest`
- **响应类型**: `Executor`
- **权限**: 管理员

#### 执行器删除
- **Method**: DELETE
- **Path**: `/admin-api/v1/executors/:id`
- **权限**: 管理员

#### 执行器健康检查
- **Method**: GET
- **Path**: `/admin-api/v1/executors/:id/health`
- **响应类型**: `ExecutorHealthResponse`
- **权限**: 管理员

#### 执行器绑定能力
- **Method**: POST
- **Path**: `/admin-api/v1/executors/:id/bindings`
- **请求类型**: `ExecutorBindingRequest`
- **响应类型**: `ExecutorBindingResponse`
- **权限**: 管理员

**请求示例**:
```json
{
  "abilityId": "ability-001",
  "priority": 1,
  "weight": 100,
  "isDefault": true
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.executors.list`, `adminApi.executors.get`, `adminApi.executors.create`, `adminApi.executors.update`, `adminApi.executors.delete`, `adminApi.executors.health`, `adminApi.executors.bind`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 执行器列表查询支持状态/类型/关键词筛选
- [ ] 执行器详情查询返回完整信息（含健康和绑定）
- [ ] 执行器创建/更新/删除正常
- [ ] 执行器健康检查数据查询正常
- [ ] 执行器绑定能力正常，支持优先级和权重
- [ ] 绑定冲突检测正常

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 接口参数校验完整
- [ ] 错误处理规范
- [ ] 接口文档（Swagger）生成

### 9.3 性能验收
- [ ] 列表查询 < 300ms
- [ ] 详情查询 < 200ms
- [ ] 健康检查数据查询 < 500ms

### 9.4 兼容性验收
- [ ] 新接口不影响现有业务

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 新接口，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T007` — executor 数据模型 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 绑定冲突 | 中 | 中 | 绑定前校验默认执行器 |
| 健康检查数据膨胀 | 中 | 中 | 自动清理策略 |
| 权限控制 | 低 | 高 | 严格管理员权限校验 |

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

- 执行器管理仅限管理员权限
- 健康检查数据建议保留 30 天
- 绑定关系支持优先级和权重配置

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
