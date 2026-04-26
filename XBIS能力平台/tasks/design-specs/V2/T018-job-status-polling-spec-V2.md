# T018 任务状态轮询 — 修正版设计方案（V2）

> 原设计方案: [T018-job-status-polling-spec.md](../T018-job-status-polling-spec.md)
> 评审报告: [T018-job-status-polling-Reviewer.md](../T018-job-status-polling-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 实现请求去重机制 | 第 5 节 API 调用 |
| 2 | 明确批量/单任务轮询策略 | 第 3 节数据流 |
| 3 | 增加轮询超时机制 | 第 4 节状态管理 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 补充 UI 友好状态 | 第 4 节状态管理 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为 Hook 开发。

---

### 2. 组件拆分

无组件拆分，本任务为 Hook 开发。

---

### 3. 数据流（V2 修正）【已修复】

#### 批量/单任务轮询策略【已修复】

```
列表页：使用批量轮询（POST /api/v1/jobs/status），每 5s 一次
详情页：使用单任务轮询（GET /api/v1/jobs/:id/status），每 3s 一次
同时打开列表和详情时：优先使用详情页轮询，列表页暂停该任务的轮询
```

---

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface PollingConfig {
  interval: number;              // 轮询间隔（毫秒）
  maxRetries?: number;           // 最大重试次数
  stopCondition?: (data: T) => boolean; // 停止条件
  maxPollingDuration?: number;   // 【新增】最大轮询时长（毫秒），默认 30 分钟
}

// 【已修复】补充 UI 友好状态
interface PollingResult<T> {
  data: T | undefined;
  isPolling: boolean;
  isLoading: boolean;            // 【新增】
  isError: boolean;              // 【新增】
  error: Error | null;
  pollCount: number;             // 【新增】
}
```

---

### 5. API 调用（V2 修正）【已修复】

```typescript
// 【新增】请求去重机制
const pollingPool = new Map<string, Promise<unknown>>();

function dedupedPoll(jobId: string, pollFn: () => Promise<unknown>) {
  if (pollingPool.has(jobId)) {
    return pollingPool.get(jobId)!;
  }
  const promise = pollFn().finally(() => pollingPool.delete(jobId));
  pollingPool.set(jobId, promise);
  return promise;
}

export function useJobPolling(jobId: string, config?: PollingConfig) {
  // 实现轮询逻辑，包含去重和超时
}
```

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 轮询超时 | 自动停止，提示用户手动刷新 |
| 网络中断 | 检测恢复后自动恢复轮询 |

---

### 8. 性能优化

- 请求去重
- 超时自动停止

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 并发请求 | 中 | 多个组件同时轮询 | 去重机制 |

---

### 10. 开发步骤拆分

#### Step 1: 基础轮询 Hook（0.5 天）
- [ ] useJobPolling 基础实现

#### Step 2: 去重与超时（0.5 天）【已修复】
- [ ] 请求去重机制
- [ ] 轮询超时机制

#### Step 3: 批量轮询（0.5 天）【已修复】
- [ ] 批量轮询策略

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **请求去重** | 未实现 | 新增 `pollingPool` 去重机制 |
| **轮询策略** | 未明确 | 明确列表页/详情页不同策略 |
| **超时机制** | 未定义 | 新增 `maxPollingDuration` |
| **UI 状态** | 仅 `isPolling` | 新增 `isLoading`/`isError`/`pollCount` |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（去重机制、轮询策略、超时机制）
2. 建议修改项已补充（UI 友好状态）
3. 风险等级：中（可控）
