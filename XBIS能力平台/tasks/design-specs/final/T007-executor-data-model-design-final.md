# T007 executor 数据模型 — 最终可开发设计方案

> 任务卡: [T007-executor-data-model.md](../T007-executor-data-model.md)
> 设计输入: docs/dev-rules.md, docs/component-architecture.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为**数据层**建设，无页面结构。输出物为 TypeScript 类型定义 + 数据库 Schema 文档。

---

## 2. 组件拆分

无 UI 组件。本任务输出为纯类型定义。

### 类型定义清单

| 类型 | 文件 | 说明 |
|------|------|------|
| Executor | `types/executor.ts` | 执行器实体 |
| ExecutorHealth | `types/executor.ts` | 执行器健康 |
| ExecutorType | `types/executor.ts` | 执行器类型枚举 |
| ExecutorStatus | `types/executor.ts` | 执行器状态枚举 |
| ExecutorListParams | `types/executor.ts` | 列表查询参数 |
| ExecutorListResponse | `types/executor.ts` | 列表响应 |
| ExecutorDetailResponse | `types/executor.ts` | 详情响应 |
| ExecutorCreateRequest | `types/executor.ts` | 创建请求 |
| ExecutorUpdateRequest | `types/executor.ts` | 更新请求 |
| ExecutorBinding | `types/executor.ts` | 执行器绑定 |

---

## 3. 数据流

无运行时数据流。类型定义为编译时静态检查。

---

## 4. 状态管理

无状态管理需求。

---

## 5. API 调用

无 API 调用。本任务为类型定义。

---

## 6. 用户交互流程

无用户交互。

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 健康数据过期 | 使用可选字段标记 |
| 绑定关系循环 | 类型定义无法约束，需后端校验 |

---

## 8. 性能优化

- 类型定义仅在编译时使用，无运行时开销

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与后端 Schema 不一致 | 高 | 类型定义可能滞后于后端变更 | 建立类型同步机制 |
| 健康数据量大 | 中 | 健康检查数据可能频繁更新 | 定义数据保留策略 |

---

## 10. 开发步骤拆分

### Step 1: 核心类型定义（0.5 人日）
- [ ] Executor 接口
- [ ] ExecutorHealth 细化

### Step 2: API 类型定义（0.5 人日）
- [ ] ExecutorListParams 类型
- [ ] ExecutorListResponse 类型
- [ ] ExecutorDetailResponse 类型
- [ ] ExecutorCreateRequest 类型
- [ ] ExecutorUpdateRequest 类型
- [ ] ExecutorBinding 类型

### Step 3: 数据库 Schema 文档（0.5 人日）
- [ ] executor 表 Schema
- [ ] executor_health 表 Schema
- [ ] executor_binding 表 Schema

### Step 4: 类型导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写类型使用文档

---

## 附录: ExecutorHealth 类型定义

```typescript
interface ExecutorHealth {
  status: 'healthy' | 'degraded' | 'unhealthy';
  cpuUsage: number;        // CPU 使用率（0-100）
  memoryUsage: number;     // 内存使用率（0-100）
  diskUsage?: number;      // 磁盘使用率（0-100）
  uptime: number;          // 运行时长（秒）
  lastCheckAt: string;     // 上次检查时间
  message?: string;        // 健康检查消息
  activeTasks: number;     // 当前活跃任务数
  queueLength?: number;    // 队列长度
}
```
