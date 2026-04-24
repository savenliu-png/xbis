# 管理端执行器管理

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端执行器管理 |
| 任务编号 | T028 |
| 所属模块 ⭐ | M11 执行器管理 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-20 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏管理端执行器管理页面，管理员需要：
- 查看执行器列表
- 查看执行器详情
- 配置执行器
- 查看健康检查

### 2.2 目标用户
- 管理员：管理执行器
- 运维人员：监控执行器健康

### 2.3 预期效果
- 提供清晰的执行器管理
- 支持健康检查可视化
- 支持配置管理

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 执行器管理 | `/admin/executors` | 管理端导航「执行器管理」 |
| 执行器详情 | `/admin/executors/:id` | 执行器列表点击 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/ExecutorManagement`

## 4. 功能范围

### 4.1 包含功能
- [ ] 执行器列表展示
- [ ] 执行器详情展示
- [ ] 健康检查数据展示
- [ ] 执行器配置
- [ ] 执行器启停
- [ ] 能力绑定管理
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 执行器自动扩缩容（V2 迭代）
- 执行器监控告警（V2 迭代）
- 执行器日志（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 管理端执行器项
export interface AdminExecutorItem {
  id: string;
  executorId: string;
  name: string;
  type: 'docker' | 'kubernetes' | 'vm' | 'serverless';
  endpoint: string;
  status: 'active' | 'inactive' | 'degraded' | 'offline';
  healthStatus: 'healthy' | 'degraded' | 'unhealthy';
  cpuUsage: number;
  memoryUsage: number;
  activeJobs: number;
  maxConcurrency: number;
  version: string;
  lastHealthCheckAt?: string;
  createdAt: string;
}

// 执行器详情
export interface ExecutorDetailAdmin {
  executor: AdminExecutorItem;
  health: ExecutorHealth;
  bindings: AbilityExecutorBinding[];
  history: ExecutorHealth[];
}

// 执行器配置
export interface ExecutorConfig {
  maxConcurrency: number;
  timeout: number;
  retryPolicy: {
    maxRetries: number;
    retryInterval: number;
  };
  resources: {
    cpu: string;
    memory: string;
  };
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AdminExecutorItem`, `ExecutorDetailAdmin`, `ExecutorConfig`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入执行器管理] ──► [加载执行器列表] ──► [展示数据] ──► [管理员操作]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无执行器 | 无执行器 | 显示空态 | 提示注册 |
| 启停失败 | 执行器异常 | 返回错误 | 提示原因 |
| 配置失败 | 配置无效 | 返回错误 | 提示修改 |

### 6.3 边界情况
- 大量执行器：支持分页
- 健康数据多：支持图表
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 搜索框 | 是（T002） |
| Table | 执行器表格 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Switch | 状态切换 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| HealthIndicator | 健康指示器 | 否 |
| ResourceChart | 资源图表 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| ExecutorTable | 执行器表格 | 否 |
| HealthPanel | 健康面板 | 否 |
| ConfigPanel | 配置面板 | 否 |
| BindingPanel | 绑定面板 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AdminPageShell | 管理页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| HealthIndicator | business | 健康指示器 |
| ResourceChart | business | 资源图表 |
| ExecutorTable | blocks | 执行器表格 |
| HealthPanel | blocks | 健康面板 |
| ConfigPanel | blocks | 配置面板 |
| BindingPanel | blocks | 绑定面板 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 执行器列表查询（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/executors`
- **请求类型**: `ExecutorListParams`
- **响应类型**: `ExecutorListResponse`
- **权限**: 管理员

#### 执行器详情查询（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/executors/:id`
- **响应类型**: `ExecutorDetailAdmin`
- **权限**: 管理员

#### 执行器配置更新
- **Method**: PUT
- **Path**: `/admin-api/v1/executors/:id/config`
- **请求类型**: `ExecutorConfig`
- **响应类型**: `AdminExecutorItem`
- **权限**: 管理员

#### 执行器启停
- **Method**: PUT
- **Path**: `/admin-api/v1/executors/:id/status`
- **请求类型**: `{ status: 'active' | 'inactive' }`
- **响应类型**: `AdminExecutorItem`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.executors.list`, `adminApi.executors.get`, `adminApi.executors.updateConfig`, `adminApi.executors.toggleStatus`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 执行器列表展示正常
- [ ] 执行器详情展示正常
- [ ] 健康检查数据展示正常
- [ ] 执行器配置正常
- [ ] 执行器启停正常
- [ ] 能力绑定管理正常
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
- [ ] 健康数据查询 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 管理端无需移动端设计

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发
  - [x] `T010` — executor API 层 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 健康数据量大 | 中 | 中 | 数据聚合 |
| 配置错误 | 中 | 高 | 配置校验 |
| 误操作 | 中 | 高 | 二次确认 |

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

- 执行器启停需二次确认
- 健康数据定时刷新（30s）
- 支持暗色模式

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
