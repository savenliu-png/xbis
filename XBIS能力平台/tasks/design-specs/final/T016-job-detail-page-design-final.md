# T016 任务详情页 — 最终可开发设计方案

> 任务卡: [T016-job-detail-page.md](../T016-job-detail-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   ├── Title: "任务详情"
    │   └── Actions: [取消] [重试]
    └── DetailPageShell
        ├── JobInfoCard
        │   └── 任务基本信息
        ├── JobStatusPanel
        │   └── 状态时间线
        └── JobResultPanel
            └── 执行结果
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Card | 信息卡片 |
| Timeline | 状态时间线 |
| Tabs | 内容切换 |
| Tag | 状态标签 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| TaskStatusBadge | 状态徽章 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| JobInfoCard | 任务信息卡片 | **新建** |
| JobStatusPanel | 状态面板 | **新建** |
| JobResultPanel | 结果面板 | **新建** |
| TimelineViewer | 时间线区 | **新建** |
| LogViewer | 日志查看区 | **新建** |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[进入 /jobs/:id]
    │
    ▼
[API: GET /api/v1/jobs/:id]
    │
    ▼
[渲染任务详情]
```

---

## 4. 状态管理

```typescript
interface JobDetailState {
  pageState: 'loading' | 'idle' | 'error';
  job: Job | null;
  result?: JobResultDetail;
  logs: JobLogItem[];
  stages: JobTimelineEvent[];
  activeTab: 'timeline' | 'result' | 'logs';
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 任务详情 | GET | `/api/v1/jobs/:id` | 查询任务详情 |
| 任务结果 | GET | `/api/v1/jobs/:id/result` | 获取任务结果 |
| 任务日志 | GET | `/api/v1/jobs/:id/logs` | 获取任务日志 |
| 取消任务 | POST | `/api/v1/jobs/:id/cancel` | 取消任务 |
| 重试任务 | POST | `/api/v1/jobs/:id/retry` | 重试任务 |

---

## 6. 用户交互流程

```
[进入任务详情]
    │
    ▼
[浏览详情]
    │
    ├── 点击取消
    ├── 点击重试
    ├── 切换 Tab
    └── 查看结果
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 任务不存在 | 404 页面 |
| 加载失败 | ErrorState + 重试 |
| 取消失败 | 提示状态不允许 |
| 重试失败 | 提示状态不允许 |
| 大量日志 | 支持分页和搜索 |
| 大结果数据 | 支持折叠和下载 |

---

## 8. 性能优化

- 详情数据缓存
- 状态轮询（T018）
- 日志查询 < 500ms
- 首屏加载 < 2s

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 状态同步 | 中 | 状态可能不及时 | 轮询机制 |
| 大量日志 | 中 | 日志数量可能很大 | 分页加载 |
| 大结果数据 | 中 | 结果数据可能很大 | 折叠展示 + 下载 |

---

## 10. 开发步骤拆分

### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + DetailPageShell

### Step 2: 详情组件（1 天）
- [ ] JobInfoCard + JobStatusPanel + JobResultPanel

### Step 3: API 集成（0.5 天）
- [ ] API 调用 + 错误处理
