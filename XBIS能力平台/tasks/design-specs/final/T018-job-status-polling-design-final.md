# T018 任务状态轮询 — 最终可开发设计方案

> 任务卡: [T018-job-status-polling.md](../T018-job-status-polling.md)
> 设计输入: docs/dev-rules.md, docs/component-architecture.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为**基础能力层**建设，无独立页面。输出物为轮询 Hook/工具函数。

```
packages/shared/src/hooks/
├── useJobPolling.ts                # 任务状态轮询 Hook
└── usePolling.ts                   # 通用轮询 Hook
```

---

## 2. 组件拆分

无 UI 组件。本任务输出为 React Hook。

### Hook 清单

| Hook | 文件 | 说明 |
|------|------|------|
| useJobPolling | `hooks/useJobPolling.ts` | 任务状态轮询 |
| usePolling | `hooks/usePolling.ts` | 通用轮询 |

---

## 3. 数据流

### 批量/单任务轮询策略

```
列表页：使用批量轮询（POST /api/v1/jobs/status），每 5s 一次
详情页：使用单任务轮询（GET /api/v1/jobs/:id/status），每 3s 一次
同时打开列表和详情时：优先使用详情页轮询，列表页暂停该任务的轮询
```

### 轮询逻辑

```
[启动轮询]
  ├── 初始间隔: 3s
  ├── 每次增加: 2s
  └── 最大间隔: 30s

[轮询逻辑]
  ├── 页面可见 ──► 正常轮询
  ├── 页面隐藏 ──► 暂停轮询
  └── 页面显示 ──► 恢复轮询

[停止条件]
  ├── 任务完成 ──► 停止轮询
  ├── 任务失败 ──► 停止轮询
  └── 连续失败 ──► 停止轮询
```

---

## 4. 状态管理

```typescript
interface PollingConfig {
  interval: number;              // 轮询间隔（毫秒）
  maxRetries?: number;           // 最大重试次数
  stopCondition?: (data: T) => boolean; // 停止条件
  maxPollingDuration?: number;   // 最大轮询时长（毫秒），默认 30 分钟
}

interface PollingResult<T> {
  data: T | undefined;
  isPolling: boolean;
  isLoading: boolean;
  isError: boolean;
  error: Error | null;
  pollCount: number;
}
```

---

## 5. API 调用

### 请求去重机制

```typescript
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

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/jobs/:id/status | 轮询触发 | jobId (path) | 增加错误计数 |
| POST /api/v1/jobs/status | 批量轮询 | jobIds: string[] (body, max 100) | 增加错误计数 |

### 类型定义

```typescript
// 单任务状态查询响应
interface JobStatusQueryResult {
  status: JobStatus;
  updatedAt: string;
}

// 批量任务状态查询响应项
interface BatchJobStatusItem {
  jobId: string;
  status: JobStatus;
  updatedAt: string;
}

type BatchJobStatusQueryResult = BatchJobStatusItem[];
```

---

## 6. 用户交互流程

无直接用户交互。轮询由页面层自动管理。

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 轮询失败 | 增加间隔，继续轮询 |
| 连续失败 | 超过阈值后停止轮询 |
| 页面隐藏 | 暂停轮询 |
| 页面显示 | 恢复轮询 |
| 轮询超时 | 自动停止，提示用户手动刷新 |
| 网络中断 | 检测恢复后自动恢复轮询 |

---

## 8. 性能优化

- 轮询间隔 ≥ 3s
- 轮询请求 < 100ms
- 页面隐藏时无轮询请求
- 使用 `Page Visibility API` 检测页面可见性
- 请求去重
- 超时自动停止

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 轮询频率过高 | 中 | 可能增加服务器压力 | 智能间隔，最小 3s |
| 内存泄漏 | 低 | 组件卸载后未清理定时器 | useEffect 清理 |
| 并发请求 | 中 | 多个组件同时轮询 | 请求去重 |

---

## 10. 开发步骤拆分

### Step 1: 基础轮询 Hook（0.5 天）
- [ ] useJobPolling 基础实现

### Step 2: 去重与超时（0.5 天）
- [ ] 请求去重机制
- [ ] 轮询超时机制

### Step 3: 批量轮询（0.5 天）
- [ ] 批量轮询策略

### Step 4: 页面可见性处理（0.5 天）
- [ ] 集成 Page Visibility API
- [ ] 实现暂停/恢复逻辑

### Step 5: 测试 & 文档（0.5 天）
- [ ] 单元测试
- [ ] 使用文档
