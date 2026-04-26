# T029 管理端任务运营 — 工程级设计方案（Design Spec）

> 任务卡：T029-admin-task-operations.md  
> 状态：设计确认  
> 输出日期：2026-04-24

---

## 1. 页面结构

### 1.1 层级结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "任务运营"
    │   └── Breadcrumb: 概览 / 任务运营
    └── ListPageShell
        ├── SlaDashboard (顶部统计卡片)
        │   ├── SLA 达成率
        │   ├── 平均响应时间
        │   ├── 待处理任务数
        │   └── 已超时任务数
        ├── FilterBar
        │   ├── 搜索框
        │   ├── 状态筛选
        │   ├── 风险等级筛选
        │   ├── 审核模式筛选
        │   └── 时间范围
        ├── DataTable
        │   ├── 任务列表（全量）
        │   └── 操作列（审核/重试/退款/查看）
        └── Pagination
```

### 1.2 模板使用

| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 外层 | AdminPageShell | 管理端统一骨架 |
| 列表区 | ListPageShell | 列表页模板 |
| 统计区 | 自定义 Dashboard | 顶部 SLA 统计卡片 |

### 1.3 布局说明

- **Desktop (≥1200px)**: 侧边栏 256px + 主内容区
- **管理端无需移动端设计**
- 表格列：任务ID | 用户 | 能力 | 状态 | 风险等级 | 审核模式 | 创建时间 | SLA | 操作

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）

| 组件 | 用途 | 位置 |
|------|------|------|
| Button | 操作按钮 | 表格操作列 |
| Input | 搜索框 | 筛选栏 |
| Table | 任务表格 | 主内容区 |
| Tag | 状态/风险标签 | 表格列 |
| Badge | SLA 状态 | SLA 列 |
| Modal | 确认弹窗 | 审核/退款确认 |
| Select | 筛选下拉 | 筛选栏 |
| DatePicker | 时间范围 | 筛选栏 |
| Skeleton | 加载骨架 | 初始加载 |
| Empty | 空态 | 无任务时 |
| Pagination | 分页 | 表格底部 |

### 2.2 business/ 层组件（已存在）

| 组件 | 用途 | 说明 |
|------|------|------|
| StatusBadge | 状态标识 | T003 已开发 |
| TaskItem | 任务项展示 | T003 已开发 |

### 2.3 blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| SlaDashboard | SLA 统计面板 | **新建** — 4 个统计卡片 |
| OperationTable | 运营任务表格 | **新建** — 基于 Table，含批量操作 |
| ReviewPanel | 审核面板 | **新建** — Modal 内审核表单 |
| RefundPanel | 退款面板 | **新建** — Modal 内退款表单 |

---

## 3. 数据流

### 3.1 页面加载数据流

```
[管理员进入 /admin/operations]
    │
    ▼
[并行加载]
    ├── API: GET /admin-api/v1/operations/sla (SLA 统计)
    └── API: GET /admin-api/v1/operations/jobs (任务列表)
    │
    ▼
[渲染页面]
    │
    ▼
[idle 状态]
```

### 3.2 审核操作数据流

```
[管理员点击「审核」]
    │
    ▼
[Modal 打开，展示任务详情]
    │
    ▼
[管理员选择通过/拒绝，填写原因]
    │
    ▼
[API: POST /admin-api/v1/operations/jobs/:id/review]
    │
    ├── 成功 → 关闭 Modal → 更新列表行状态 → Toast 成功
    │
    └── 失败 → Modal 内显示错误
```

### 3.3 退款操作数据流

```
[管理员点击「退款」]
    │
    ▼
[Modal 打开，展示订单信息]
    │
    ▼
[管理员输入退款金额和原因]
    │
    ▼
[API: POST /admin-api/v1/operations/jobs/:id/refund]
    │
    ├── 成功 → 关闭 Modal → 更新列表 → Toast 成功
    │
    └── 失败 → 显示错误（金额超限等）
```

### 3.4 数据流图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Router    │────▶│  ListParams │────▶│  API GET    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                                │
                       ┌────────────────────────┘
                       ▼
              ┌─────────────────┐
              │  OperationPage   │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │  审核    │   │  重试   │   │  退款   │
   │  POST   │   │  POST   │   │  POST   │
   └─────────┘   └─────────┘   └─────────┘
```

---

## 4. 状态管理

### 4.1 页面级状态机

```typescript
type PageState =
  | 'loading'      // 初始加载
  | 'idle'         // 正常浏览
  | 'reviewing'    // 审核中
  | 'retrying'     // 重试中
  | 'refunding'    // 退款中
  | 'error'        // 加载错误
```

### 4.2 状态定义

```typescript
interface AdminOperationsState {
  // 页面状态
  pageState: PageState;

  // SLA 统计
  slaStats: SlaStats | null;

  // 列表数据
  jobs: OperationJobItem[];
  total: number;

  // 筛选条件
  filters: {
    keyword?: string;
    status?: string;
    riskLevel?: string;
    approvalMode?: string;
    startDate?: string;
    endDate?: string;
    page: number;
    pageSize: number;
  };

  // 选中行（批量操作）
  selectedRowKeys: string[];

  // 当前操作的任务
  currentJob: OperationJobItem | null;

  // Modal 状态
  reviewModalOpen: boolean;
  refundModalOpen: boolean;

  // 错误信息
  error: ApiError | null;
}
```

---

## 5. API 调用

### 5.1 API 列表

| API | Method | Path | 触发时机 | 错误处理 |
|-----|--------|------|----------|----------|
| SLA 统计 | GET | `/admin-api/v1/operations/sla` | 页面加载 | 静默失败，显示占位 |
| 任务列表 | GET | `/admin-api/v1/operations/jobs` | 页面加载/筛选 | 显示错误态 |
| 任务审核 | POST | `/admin-api/v1/operations/jobs/:id/review` | 提交审核 | Modal 内显示错误 |
| 任务重试 | POST | `/admin-api/v1/operations/jobs/:id/retry` | 点击重试 | Toast 错误 |
| 退款处理 | POST | `/admin-api/v1/operations/jobs/:id/refund` | 提交退款 | Modal 内显示错误 |

### 5.2 请求参数

```typescript
// GET /admin-api/v1/operations/jobs
interface JobListParams {
  keyword?: string;
  status?: string;
  riskLevel?: string;
  approvalMode?: string;
  startDate?: string;
  endDate?: string;
  page?: number;
  pageSize?: number;
}

// POST /admin-api/v1/operations/jobs/:id/review
interface JobReviewRequest {
  jobId: string;
  action: 'approve' | 'reject';
  reason?: string;
}

// POST /admin-api/v1/operations/jobs/:id/refund
interface JobRefundRequest {
  jobId: string;
  amount: number;
  reason: string;
}
```

---

## 6. 用户交互流程

### 6.1 主流程：审核任务

```
[进入任务运营页]
    │
    ▼
[浏览任务列表]
    │
    ▼
[点击「审核」按钮]
    │
    ▼
[Modal 展示任务详情]
    │
    ▼
[选择通过/拒绝 + 填写原因]
    │
    ▼
[提交审核]
    │
    ▼
[更新列表状态]
```

### 6.2 批量操作

```
[勾选多行复选框]
    │
    ▼
[顶部显示批量操作栏]
    │
    ▼
[选择批量审核/重试]
    │
    ▼
[Modal 确认]
    │
    ▼
[API 批量操作]
    │
    ▼
[更新列表]
```

---

## 7. 边界与异常

### 7.1 空态处理

| 场景 | 处理方式 |
|------|----------|
| 无任务 | Empty 组件 |
| 筛选无结果 | Empty + 清除筛选 |
| SLA 无数据 | 显示占位符 `--` |

### 7.2 错误处理

| 场景 | 处理方式 |
|------|----------|
| 审核失败（状态已变） | Modal 提示「任务状态已变更，请刷新」 |
| 退款失败（金额超限） | Modal 提示「退款金额超过订单金额」 |
| SLA 告警 | 行高亮 + Badge 标记 |

---

## 8. 性能优化

- 列表虚拟滚动（>100 条）
- SLA 统计缓存（5 分钟）
- 筛选 Debounce 300ms

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| **误操作审核/退款** | 影响用户资金和任务执行 | 二次确认 Modal；敏感操作记录日志 |
| **数据量大** | 全量任务列表可能很大 | 强制分页；最大 100 条/页 |
| **并发操作** | 多人同时处理同一任务 | 乐观锁；操作前校验任务状态 |

---

## 10. 开发步骤拆分

### Step 1: 基础结构（0.5 天）
- [ ] 创建页面 `/admin/operations`
- [ ] 搭建页面骨架 + SLA Dashboard

### Step 2: 列表与筛选（1 天）
- [ ] 实现 OperationTable
- [ ] 实现筛选栏（含时间范围）
- [ ] 实现批量操作

### Step 3: 操作面板（1.5 天）
- [ ] 实现 ReviewPanel（审核 Modal）
- [ ] 实现 RefundPanel（退款 Modal）
- [ ] 实现重试逻辑

### Step 4: API 与优化（1 天）
- [ ] 实现所有 API 调用
- [ ] 实现虚拟滚动
- [ ] 错误处理完善

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/admin/TaskOperations/
├── index.tsx
├── TaskOperationsPage.tsx
├── components/
│   ├── SlaDashboard.tsx
│   ├── OperationTable.tsx
│   ├── ReviewPanel.tsx
│   └── RefundPanel.tsx
└── hooks/
    └── useTaskOperations.ts
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/admin.tsx
```
