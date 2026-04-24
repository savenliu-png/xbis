# executor 数据模型

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | executor 数据模型 |
| 任务编号 | T007 |
| 所属模块 ⭐ | M2 数据层 |
| 优先级 | P0 |
| 指派给 | @backend-dev |
| 预计工期 | 2 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-04-27 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏执行器管理的数据模型，需要：
- 存储执行器注册信息
- 记录执行器健康状态
- 支持执行器与能力的绑定关系

### 2.2 目标用户
- 后端开发者：使用统一的数据模型开发 API
- 前端开发者：依赖类型定义开发页面

### 2.3 预期效果
- 建立 executor 表，支持执行器注册
- 建立 executor_health 表，支持健康检查
- 支持执行器与能力的绑定

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`docs/方案&配置/01 xbis-front 系统重构方案.md`

## 4. 功能范围

### 4.1 包含功能
- [ ] 创建 executor 表
- [ ] 创建 executor_health 表
- [ ] 创建 ability_executor_binding 表
- [ ] 健康检查记录逻辑

### 4.2 不包含功能（明确排除）
- 执行器调度逻辑（由调度器实现）
- 执行器监控告警（由监控系统实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// executor 表
export interface Executor {
  id: string;
  executorId: string;
  name: string;
  description?: string;
  type: 'docker' | 'kubernetes' | 'vm' | 'serverless';
  endpoint: string;
  status: 'active' | 'inactive' | 'degraded' | 'offline';
  capabilities: string[];
  maxConcurrency: number;
  currentLoad: number;
  version: string;
  metadata?: Record<string, any>;
  createdAt: string;
  updatedAt: string;
  lastHealthCheckAt?: string;
}

// executor_health 表
export interface ExecutorHealth {
  id: string;
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
}

// ability_executor_binding 表
export interface AbilityExecutorBinding {
  id: string;
  abilityId: string;
  executorId: string;
  priority: number;
  weight: number;
  isDefault: boolean;
  createdAt: string;
  updatedAt: string;
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| executors | CREATE | — | — | 执行器主表 |
| executor_health | CREATE | — | — | 健康检查记录表 |
| ability_executor_bindings | CREATE | — | — | 能力执行器绑定表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`Executor`, `ExecutorHealth`, `AbilityExecutorBinding`

## 6. 交互流程

### 6.1 主流程
```
[创建 executor 表] ──► [创建 executor_health 表] ──► [创建 ability_executor_binding 表] ──► [健康检查记录]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 健康检查失败 | 执行器无响应 | 标记为 unhealthy | 监控告警 |
| 绑定冲突 | 同一能力绑定多个默认执行器 | 拒绝绑定 | 错误提示 |

### 6.3 边界情况
- 执行器离线：保留历史记录，标记状态
- 健康检查数据过期：自动清理 30 天前数据

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

无

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] executor 表创建成功，字段完整
- [ ] executor_health 表创建成功，支持健康指标存储
- [ ] ability_executor_binding 表创建成功，支持绑定关系
- [ ] 健康检查记录逻辑正常
- [ ] 数据清理逻辑正常

### 9.2 技术验收
- [ ] 数据库迁移可回滚
- [ ] 索引优化，查询性能 < 100ms
- [ ] 字段命名符合规范

### 9.3 性能验收
- [ ] 单表数据量 10w 时查询 < 100ms
- [ ] 健康检查写入性能 > 1000 QPS

### 9.4 兼容性验收
- [ ] 新表不影响现有业务

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否（新表）** — 新建表，不影响现有数据

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **否** — 可独立开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 健康检查数据膨胀 | 中 | 中 | 自动清理策略 |
| 绑定关系循环依赖 | 低 | 高 | 绑定前校验 |

## 11. 开发检查清单

### 11.1 开发前
- [x] 需求已评审并确认
- [x] 设计稿已确认
- [x] 数据结构和 API 已评审
- [x] 依赖任务已完成

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

- 全新表，无历史包袱
- 健康检查数据建议保留 30 天
- 绑定关系支持优先级和权重配置

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
