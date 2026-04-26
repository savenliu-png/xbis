# T007 executor 数据模型 — 修正版设计方案（V2）

> 原设计方案: [T007-executor-data-model-spec.md](../T007-executor-data-model-spec.md)
> 评审报告: [T007-executor-data-model-Reviewer.md](../T007-executor-data-model-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed

---

## 二、必须修改项（Blocking）

无必须修改项。

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 细化 ExecutorHealth 类型 | 第 4 节状态管理 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为数据模型定义。

---

### 2. 组件拆分

无组件拆分，本任务为数据模型定义。

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

#### ExecutorHealth 类型（V2 细化）【已修复】

```typescript
interface ExecutorHealth {
  status: 'healthy' | 'degraded' | 'unhealthy';
  cpuUsage: number;        // CPU 使用率（0-100）
  memoryUsage: number;     // 内存使用率（0-100）
  diskUsage?: number;      // 磁盘使用率（0-100）【新增】
  uptime: number;          // 运行时长（秒）
  lastCheckAt: string;     // 上次检查时间
  message?: string;        // 健康检查消息
  activeTasks: number;     // 【新增】当前活跃任务数
  queueLength?: number;    // 【新增】队列长度
}
```

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

无边界与异常。

---

### 8. 性能优化

无性能优化。

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 健康数据量大 | 低 | 历史健康数据可能很大 | 限制保留时长 |

---

### 10. 开发步骤拆分

#### Step 1: 核心类型定义（0.5 人日）
- [ ] Executor 接口
- [ ] ExecutorHealth 细化 【已修复】

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **ExecutorHealth** | 未细化字段 | 新增 diskUsage、activeTasks、queueLength |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 原设计已通过评审
2. 建议修改项已补充（Health 字段细化）
3. 风险等级：低
