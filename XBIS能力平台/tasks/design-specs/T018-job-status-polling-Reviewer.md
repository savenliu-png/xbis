# T018 任务状态轮询 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: 缺少请求去重机制的具体实现方案
**严重程度: 中**

任务卡提到"并发请求"风险，但设计方案中未明确说明如何实现请求去重。多个组件同时轮询同一任务时，会产生重复请求。

### 问题2: 批量轮询 API 与单任务轮询的协调策略未明确
**严重程度: 中**

同时存在 `GET /api/v1/jobs/:id/status` 和 `POST /api/v1/jobs/status` 两个 API，但未说明何时使用批量轮询、何时使用单任务轮询。

### 问题3: 轮询状态暴露不足
**严重程度: 低**

Hook 返回的 `PollingState` 包含内部状态，但未提供 `isLoading` 或 `isError` 等便于 UI 消费的状态。

### 问题4: 缺少轮询超时机制
**严重程度: 中**

长时间运行任务（如超过 30 分钟）持续轮询会浪费资源，应设置最大轮询时长。

---

## 修改建议

### 建议1: 增加请求去重机制（必须修改）
```typescript
// 使用请求池实现去重
const pollingPool = new Map<string, Promise<unknown>>();

function dedupedPoll(jobId: string, pollFn: () => Promise<unknown>) {
  if (pollingPool.has(jobId)) {
    return pollingPool.get(jobId)!;
  }
  const promise = pollFn().finally(() => pollingPool.delete(jobId));
  pollingPool.set(jobId, promise);
  return promise;
}
```

### 建议2: 明确批量轮询策略（必须修改）
- 列表页：使用批量轮询（`POST /api/v1/jobs/status`），每 5s 一次
- 详情页：使用单任务轮询（`GET /api/v1/jobs/:id/status`），每 3s 一次
- 同时打开列表和详情时：优先使用详情页轮询，列表页暂停该任务的轮询

### 建议3: 补充 UI 友好状态（建议修改）
```typescript
interface PollingResult<T> {
  data: T | undefined;
  isPolling: boolean;
  isLoading: boolean;
  isError: boolean;
  error: Error | null;
  pollCount: number;
}
```

### 建议4: 增加轮询超时机制（必须修改）
```typescript
interface PollingConfig {
  // ... 现有配置
  maxPollingDuration?: number; // 最大轮询时长（毫秒），默认 30 分钟
}
```

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 实现请求去重机制（建议1）
2. 明确批量/单任务轮询策略（建议2）
3. 增加轮询超时机制（建议4）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | N/A | 无 UI 组件 |
| 页面模板使用 | N/A | 无页面 |
| 重复组件 | N/A | 无组件 |
| 分层规范 | ✅ | Hook 属于 shared 层 |
| API 合理性 | ⚠️ | 批量/单任务策略不明确 |
| 状态设计完整性 | ⚠️ | 缺少 UI 友好状态 |
| 能力平台架构 | ✅ | 符合 |
| 过度设计 | ✅ | 不过度 |
| 未声明数据模型 | ⚠️ | 缺少去重/超时机制 |

**风险等级**: 中
