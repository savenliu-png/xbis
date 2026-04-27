# T018 任务状态轮询 — 验收检查报告

> 任务卡: [T018-job-status-polling.md](../T018-job-status-polling.md)
> 设计方案: [T018-job-status-polling-design-final.md](../design-specs/final/T018-job-status-polling-design-final.md)
> 验收日期: 2026-04-26
> 验收结果: **通过**

---

## 1. 修改文件清单

| 文件路径 | 修改类型 | 说明 |
|---------|---------|------|
| `packages/shared/src/index.ts` | 修改 | 补充 hooks 统一导出 |
| `packages/shared/src/hooks/index.ts` | 新增 | hooks 统一导出入口 |
| `packages/shared/src/hooks/usePolling.ts` | 修改 | 修复递归闭包陷阱，完善类型定义 |
| `packages/shared/src/hooks/useJobPolling.ts` | 修改 | 修复 API 调用类型，完善返回类型 |
| `packages/shared/src/api/services.ts` | 修改 | 移除未定义的 `includeCreate` 参数 |
| `packages/components/blocks/PollingIndicator/index.tsx` | 新增 | 轮询状态指示器组件 |
| `packages/components/blocks/index.ts` | 修改 | 导出 PollingIndicator |

---

## 2. 新增组件清单

| 组件名称 | 所属层级 | 说明 |
|---------|---------|------|
| PollingIndicator | blocks | 轮询状态指示器，支持单任务/批量任务状态展示 |

---

## 3. API 变更清单

| API 路径 | 方法 | 调用位置 | 说明 |
|---------|------|---------|------|
| `/api/v1/jobs/:id/status` | GET | `useJobPolling.ts` | 单任务状态查询 |
| `/api/v1/jobs/status` | POST | `useBatchJobPolling.ts` | 批量任务状态查询 |

**API 调用方式**: 均通过 `userApi.jobs.getStatus` / `userApi.jobs.batchGetStatus` 调用，符合 Business Services 层规范。

---

## 4. 风险说明

| 风险项 | 评估 | 说明 |
|-------|------|------|
| 是否影响现网 | 否 | 全新 hooks 和组件，不修改现有页面逻辑 |
| 是否影响数据结构 | 否 | 仅读取 job status，不修改数据 |
| 是否影响性能 | 低 | 轮询间隔最小 3s，支持页面隐藏暂停，有智能退避 |
| 内存泄漏风险 | 低 | useEffect 清理定时器，组件卸载时自动停止 |

---

## 5. 自检结果

### C5 AI 强制自检

| 检查项 | 结果 |
|--------|------|
| 递归闭包陷阱 | ✅ 已修复（使用 ref 模式） |
| API 参数错误 | ✅ 已修复（移除未定义参数） |
| Tag variant 类型安全 | ✅ 已修复（使用有效 color/variant） |
| API 响应类型安全 | ✅ 已修复（使用 ApiResponse<T> 泛型） |
| TypeScript 编译 | ✅ T018 相关代码零错误 |

**结果: Passed**

### C6 开发后自检

| 检查项 | 状态 |
|--------|------|
| 组件复用 | ✅ |
| 命名规范 | ✅ |
| 页面结构统一 | ✅ |
| 状态完整（loading/empty/error） | ✅ |
| API 规范 | ✅ |
| 权限兼容 | ✅ |
| 异常处理 | ✅ |
| 是否影响现网 | ✅ |

**结果: Pass**

---

## 6. 验收结果

| 验收项 | 结果 |
|--------|------|
| 页面是否能打开 | ✅ |
| 主流程是否可用 | ✅ |
| API 是否成功调用 | ✅ |
| 是否存在报错 | ✅ |
| UI 是否破坏 | ✅ |
| 是否影响旧功能 | ✅ |

**👉 验收结果: 通过**

---

## 7. 使用示例

### 单任务轮询

```tsx
import { useJobPolling } from '@xbis/shared';
import { PollingIndicator } from '@xbis/components/blocks';

function JobDetailPage({ jobId }: { jobId: string }) {
  const { status, isPolling, isLoading, isError, error, pollCount, start, stop, poll } = useJobPolling(jobId, {
    onStatusChange: (event) => {
      console.log('Status changed:', event.previousStatus, '->', event.currentStatus);
    },
    onStop: (reason) => {
      console.log('Polling stopped:', reason);
    },
  });

  return (
    <div>
      <PollingIndicator
        isPolling={isPolling}
        isLoading={isLoading}
        isError={isError}
        error={error}
        pollCount={pollCount}
        status={status}
        onRefresh={poll}
        onStop={stop}
        onStart={start}
      />
    </div>
  );
}
```

### 批量任务轮询

```tsx
import { useBatchJobPolling } from '@xbis/shared';

function JobListPage({ jobIds }: { jobIds: string[] }) {
  const { statuses, isPolling, start, stop } = useBatchJobPolling(jobIds, {
    onStatusChange: (events) => {
      events.forEach((event) => {
        console.log('Job', event.jobId, ':', event.previousStatus, '->', event.currentStatus);
      });
    },
  });

  // statuses: Record<string, JobStatus>
  return <div>{/* render job list with statuses */}</div>;
}
```

---

## 8. 备注

- 轮询间隔策略：初始 3s，每次增加 2s，最大 30s
- 终态任务自动停止：completed / failed / cancelled / callback_failed
- 页面隐藏时自动暂停轮询（Page Visibility API）
- 连续失败 5 次后自动停止轮询
- 最大轮询时长 30 分钟
