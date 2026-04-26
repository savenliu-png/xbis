# T016 任务详情页 — 工程级设计方案

> 任务卡: [T016-job-detail-page.md](../T016-job-detail-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 面包屑导航
│   └── 操作按钮: 取消/重试
├── DetailPageShell
│   ├── MainContentArea (70%)
│   │   ├── JobInfo (任务基本信息)
│   │   ├── Tabs
│   │   │   ├── TabPane: 时间线
│   │   │   ├── TabPane: 结果
│   │   │   └── TabPane: 日志
│   │   └── ActionBar
│   └── SideContentArea (30%)
│       ├── JobMeta (元信息)
│       └── AbilityInfo (能力信息)
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Card | 信息卡片 | 否 |
| Tag | 状态标签 | 否 |
| Badge | 状态标识 | 否 |
| Tabs | 内容切换 | 否 |
| Timeline | 时间线 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| StatusBadge | 状态标识 | 否 |
| TaskItem | 任务项 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| JobInfo | 任务信息区 | 是 |
| TimelineViewer | 时间线区 | 是 |
| ResultViewer | 结果展示区 | 是 |
| LogViewer | 日志查看区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[页面加载]
  ├── 调用 GET /api/v1/jobs/:id ──► 获取任务详情
  ├── 调用 GET /api/v1/jobs/:id/result ──► 获取任务结果
  └── 调用 GET /api/v1/jobs/:id/logs ──► 获取任务日志

[用户操作]
  ├── 点击取消 ──► 调用 POST /api/v1/jobs/:id/cancel
  └── 点击重试 ──► 调用 POST /api/v1/jobs/:id/retry
```

---

## 4. 状态管理

### 页面状态

```typescript
interface JobDetailState {
  // 数据状态
  job?: JobDetail;
  result?: JobResultDetail;
  logs: JobLogItem[];
  stages: JobTimelineEvent[];

  // UI 状态
  activeTab: 'timeline' | 'result' | 'logs';
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/jobs/:id | 页面加载 | jobId | 显示错误提示 |
| GET /api/v1/jobs/:id/result | 页面加载 | jobId | 静默失败 |
| GET /api/v1/jobs/:id/logs | 切换日志 Tab | jobId | 静默失败 |
| POST /api/v1/jobs/:id/cancel | 点击取消 | jobId | 展示错误信息 |
| POST /api/v1/jobs/:id/retry | 点击重试 | jobId | 展示错误信息 |

---

## 6. 用户交互流程

### 主流程

```
用户点击任务
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 显示任务详情
  │   ├── 用户切换 Tab ──► 展示对应内容
  │   ├── 用户点击取消 ──► 取消任务
  │   └── 用户点击重试 ──► 重试任务
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 任务不存在 | 无效的 jobId | 返回 404 | 提示任务不存在 |
| 取消失败 | 任务已完成 | 返回错误 | 提示状态不允许 |
| 重试失败 | 任务未失败 | 返回错误 | 提示状态不允许 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 长时间任务 | 支持实时更新 |
| 大量日志 | 支持分页和搜索 |
| 大结果数据 | 支持折叠和下载 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 日志查询 < 500ms
- 无内存泄漏

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 大量日志 | 中 | 日志数量可能很大 | 分页加载 |
| 大结果数据 | 中 | 结果数据可能很大 | 折叠展示 + 下载 |
| 状态实时变化 | 中 | 任务状态可能频繁变化 | 手动刷新 + 轮询 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 DetailPageShell 搭建页面框架

### Step 2: 信息区开发（0.5 人日）
- [ ] JobInfo 组件
- [ ] JobMeta 组件

### Step 3: Tab 内容区开发（1.5 人日）
- [ ] TimelineViewer 组件
- [ ] ResultViewer 组件
- [ ] LogViewer 组件

### Step 4: 操作栏开发（0.5 人日）
- [ ] 取消按钮
- [ ] 重试按钮

### Step 5: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现加载/空态/错误态

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 优化日志加载
- [ ] 自测
