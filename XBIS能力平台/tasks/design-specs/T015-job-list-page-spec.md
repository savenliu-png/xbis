# T015 任务列表页 — 工程级设计方案

> 任务卡: [T015-job-list-page.md](../T015-job-list-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
TaskPageShell (一级模板)
├── PageHeader
│   ├── 标题: "任务中心"
│   └── 操作按钮: "创建任务"
├── Filter Bar
│   ├── 状态筛选标签
│   ├── 能力筛选
│   ├── 日期筛选
│   └── 关键词搜索
├── BatchActionBar (选中时显示)
│   ├── 批量取消
│   └── 批量删除
├── ContentArea
│   ├── JobList (任务列表)
│   │   └── TaskItem[]
│   └── Pagination
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Input | 搜索框 | 否 |
| Card | 统计卡片 | 否 |
| Tag | 状态标签 | 否 |
| Badge | 状态标识 | 否 |
| Checkbox | 批量选择 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |
| Pagination | 分页 | 否 |
| DatePicker | 日期筛选 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| TaskItem | 任务项 | 否 |
| StatusBadge | 状态标识 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| JobList | 任务列表 | 是 |
| StatusStats | 状态统计 | 是 |
| FilterBar | 筛选栏 | 是 |
| BatchActionBar | 批量操作栏 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| TaskPageShell | 任务页一级模板 |
| ListPageShell | 列表页模板 |

---

## 3. 数据流

```
[页面加载]
  ├── 调用 GET /api/v1/jobs ──► 获取任务列表
  └── 调用 GET /api/v1/jobs/stats ──► 获取状态统计

[用户筛选]
  ├── 选择状态 ──► 更新 filter.status ──► 重新查询
  ├── 选择能力 ──► 更新 filter.abilityId ──► 重新查询
  ├── 选择日期 ──► 更新 filter.dateRange ──► 重新查询
  └── 输入关键词 ──► debounce 300ms ──► 重新查询

[用户批量操作]
  ├── 选择任务 ──► 更新 selectedJobs
  └── 点击批量操作 ──► 调用 POST /api/v1/jobs/batch
```

---

## 4. 状态管理

### 页面状态

```typescript
interface JobListState {
  // 数据状态
  jobs: JobListItem[];
  statusStats: JobStatusStats;
  total: number;

  // 筛选状态
  filter: JobFilterState;

  // 分页状态
  pagination: {
    page: number;
    pageSize: number;
  };

  // 选择状态
  selectedJobs: string[];

  // UI 状态
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/jobs | 页面加载、筛选变化、分页变化 | JobListParams | 显示错误提示 |
| GET /api/v1/jobs/stats | 页面加载 | - | 静默失败 |
| POST /api/v1/jobs/batch | 批量操作 | JobBatchRequest | 展示错误信息 |

---

## 6. 用户交互流程

### 主流程

```
用户进入任务中心
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 显示任务列表
  │   ├── 用户筛选 ──► 更新列表
  │   ├── 用户搜索 ──► 更新列表
  │   ├── 用户选择任务 ──► 显示批量操作栏
  │   └── 用户点击任务 ──► 跳转任务详情
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 无数据 | 无任务记录 | 显示空态 | 提示创建任务 |
| 批量操作失败 | 部分任务状态不允许 | 标记失败项 | 提示具体原因 |
| 搜索无结果 | 关键词无匹配 | 显示空态 | 提示更换关键词 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 大量任务 | 支持虚拟滚动和分页 |
| 状态实时变化 | 支持手动刷新 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 搜索使用 debounce（300ms）
- 列表支持虚拟滚动（>100 条数据时）
- 分页加载
- 状态统计定时刷新

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 大量数据渲染 | 中 | 任务数量可能很大 | 虚拟滚动 + 分页 |
| 状态实时变化 | 中 | 任务状态可能频繁变化 | 手动刷新 + 轮询 |
| 批量操作复杂 | 低 | 批量操作需校验状态 | 状态校验 + 错误提示 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 TaskPageShell + ListPageShell 搭建页面框架

### Step 2: 筛选区开发（1 人日）
- [ ] FilterBar 组件
- [ ] StatusStats 组件
- [ ] 日期筛选

### Step 3: 列表区开发（1 人日）
- [ ] JobList 组件
- [ ] TaskItem 集成
- [ ] 批量选择

### Step 4: 批量操作开发（0.5 人日）
- [ ] BatchActionBar 组件
- [ ] 批量取消/删除

### Step 5: 状态管理 & API 集成（0.5 人日）
- [ ] 实现筛选状态管理
- [ ] 集成 API 调用

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 实现 debounce
- [ ] 自测
