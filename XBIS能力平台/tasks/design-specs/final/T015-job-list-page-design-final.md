# T015 任务列表页 — 最终可开发设计方案

> 任务卡: [T015-job-list-page.md](../T015-job-list-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   ├── Title: "任务列表"
    │   └── Actions: [创建任务]
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框
        │   ├── 状态筛选
        │   └── 时间范围
        ├── JobTable
        │   └── 任务列表
        └── Pagination
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 任务表格 |
| Tag | 状态标签 |
| Pagination | 分页 |
| Checkbox | 批量选择 |
| DatePicker | 日期筛选 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| TaskItem | 任务项 |
| TaskStatusBadge | 状态徽章 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| JobTable | 任务表格 | **新建** |
| FilterBar | 筛选栏 | **新建** |
| BatchActionBar | 批量操作栏 | **新建** |
| StatusStats | 状态统计 | **新建** |

### layout/ 层

| 模板 | 用途 |
|------|------|
| TaskPageShell | 任务页一级模板 |
| ListPageShell | 列表页模板 |

---

## 3. 数据流

```
[进入 /jobs]
    │
    ▼
[API: GET /api/v1/jobs]
    │
    ▼
[渲染任务列表]
```

---

## 4. 状态管理

```typescript
interface JobListState {
  pageState: 'loading' | 'idle' | 'error';
  jobs: Job[];
  total: number;
  filters: { keyword?: string; status?: string; startDate?: string; endDate?: string; page: number; pageSize: number };
  selectedJobs: string[];
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 任务列表 | GET | `/api/v1/jobs` | 支持筛选分页 |
| 状态统计 | GET | `/api/v1/jobs/stats` | 获取状态统计 |
| 批量操作 | POST | `/api/v1/jobs/batch` | 批量取消/删除 |

---

## 6. 用户交互流程

```
[进入任务列表]
    │
    ▼
[浏览列表]
    │
    ├── 搜索/筛选
    ├── 点击任务查看详情
    ├── 选择任务批量操作
    └── 点击创建任务
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无任务 | Empty 组件 |
| 筛选无结果 | Empty + 清除筛选 |
| 批量操作失败 | 标记失败项，提示具体原因 |

---

## 8. 性能优化

- 列表虚拟滚动（>100 条）
- 筛选 Debounce 300ms
- 分页加载
- 状态统计定时刷新

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 数据量大 | 低 | 任务数量多 | 分页加载 |
| 状态实时变化 | 中 | 任务状态可能频繁变化 | 手动刷新 + 轮询 |
| 批量操作复杂 | 低 | 批量操作需校验状态 | 状态校验 + 错误提示 |

---

## 10. 开发步骤拆分

### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + ListPageShell

### Step 2: 列表组件（1 天）
- [ ] JobTable + FilterBar + BatchActionBar

### Step 3: API 与优化（0.5 天）
- [ ] API 集成 + 虚拟滚动
