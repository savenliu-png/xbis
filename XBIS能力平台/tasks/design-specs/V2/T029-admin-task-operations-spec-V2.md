# T029 管理端任务运营 — 修正版设计方案（V2）

> 原设计方案: [T029-admin-task-operations-spec.md](../T029-admin-task-operations-spec.md)
> 评审报告: [T029-admin-task-operations-Reviewer.md](../T029-admin-task-operations-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加操作日志记录 | 第 5 节 API 调用 |
| 2 | 完善批量操作错误处理 | 第 6 节用户交互流程 |
| 3 | 增加退款金额校验 | 第 5 节 API 调用 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确 SLA 计算公式 | 第 4 节状态管理 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "任务运营"
    │   └── Breadcrumb: 概览 / 任务运营
    └── ListPageShell
        ├── SlaDashboard
        │   └── SLA 统计卡片
        ├── FilterBar
        │   ├── 搜索框
        │   ├── 状态筛选
        │   ├── 风险等级筛选
        │   ├── 审核模式筛选
        │   └── 时间范围
        ├── OperationTable
        │   └── 任务列表（含批量操作）
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 任务表格 |
| Tag | 状态/风险标签 |
| Badge | SLA 状态 |
| Modal | 确认弹窗 |
| Select | 筛选下拉 |
| DatePicker | 时间范围 |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Pagination | 分页 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| StatusBadge | 状态标识 |
| TaskItem | 任务项展示 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| SlaDashboard | SLA 统计 | 4 个统计卡片 |
| OperationTable | 运营表格 | 含批量操作 |
| ReviewPanel | 审核面板 | Modal 内审核表单 |
| RefundPanel | 退款面板 | Modal 内退款表单 |

---

### 3. 数据流

```
[进入 /admin/operations]
    │
    ▼
[并行加载]
    ├── API: GET /admin-api/v1/operations/sla
    └── API: GET /admin-api/v1/operations/jobs
    │
    ▼
[渲染页面]
```

---

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface AdminOperationsState {
  pageState: 'loading' | 'idle' | 'reviewing' | 'retrying' | 'refunding' | 'error';
  slaStats: SlaStats | null;
  jobs: OperationJobItem[];
  total: number;
  filters: { keyword?: string; status?: string; riskLevel?: string; approvalMode?: string; startDate?: string; endDate?: string; page: number; pageSize: number };
  selectedRowKeys: string[];
  currentJob: OperationJobItem | null;
  reviewModalOpen: boolean;
  refundModalOpen: boolean;
  error: ApiError | null;
}

// 【新增】SLA 计算公式
const slaFormulas = {
  achievementRate: '按时完成的任务数 / 总任务数',
  avgResponseTime: '所有任务响应时间的平均值',
  pendingCount: '状态为 pending + running 的任务数',
  overdueCount: '超过 SLA 时间仍未完成的任务数',
};
```

---

### 5. API 调用（V2 修正）【已修复】

| API | Method | Path | 说明 |
|-----|--------|------|------|
| SLA 统计 | GET | `/admin-api/v1/operations/sla` | 查询 SLA 统计 |
| 任务列表 | GET | `/admin-api/v1/operations/jobs` | 查询任务列表 |
| 任务审核 | POST | `/admin-api/v1/operations/jobs/:id/review` | 审核任务 |
| 任务重试 | POST | `/admin-api/v1/operations/jobs/:id/retry` | 重试任务 |
| 退款处理 | POST | `/admin-api/v1/operations/jobs/:id/refund` | 退款处理 |
| 操作日志 | POST | `/admin-api/v1/operations/logs` | 【新增】记录操作日志 |

#### 退款金额校验（V2 新增）【已修复】

```typescript
// 退款金额校验规则
// 1. 退款金额 > 0
// 2. 退款金额 ≤ 订单剩余可退金额
// 3. 退款金额 ≤ 订单总金额
```

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入任务运营]
    │
    ▼
[浏览任务列表]
    │
    ├── 点击「审核」
    │       │
    │       ▼
    │   [Modal 展示任务详情]
    │       │
    │       ▼
    │   [选择通过/拒绝 + 填写原因]
    │       │
    │       ▼
    │   [提交审核 ──► 记录操作日志] 【新增】
    │
    ├── 勾选多行 ──► 批量操作
    │       │
    │       ▼
    │   [批量审核/重试]
    │       │
    │       ▼
    │   [结果展示：成功数/失败数/失败原因] 【新增】
    │
    └── 点击「退款」
            │
            ▼
        [Modal 展示订单信息]
            │
            ▼
        [输入退款金额（带校验）] 【已修复】
            │
            ▼
        [提交退款 ──► 记录操作日志] 【新增】
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 审核失败（状态已变） | 提示「任务状态已变更，请刷新」 |
| 退款失败（金额超限） | 提示「退款金额超过订单金额」 |
| SLA 告警 | 行高亮 + Badge 标记 |

---

### 8. 性能优化

- 列表虚拟滚动（>100 条）
- SLA 统计缓存（5 分钟）
- 筛选 Debounce 300ms

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 误操作审核/退款 | 高 | 影响用户资金和任务执行 | 二次确认 + 操作日志 |
| 数据量大 | 中 | 全量任务列表可能很大 | 强制分页 |
| 并发操作 | 中 | 多人同时处理同一任务 | 乐观锁 |

---

### 10. 开发步骤拆分

#### Step 1: 基础结构（0.5 天）
- [ ] 页面骨架 + SLA Dashboard

#### Step 2: 列表与筛选（1 天）
- [ ] OperationTable + 筛选栏
- [ ] 批量操作

#### Step 3: 操作面板（1.5 天）【已修复】
- [ ] ReviewPanel（含操作日志）
- [ ] RefundPanel（含金额校验）
- [ ] 重试逻辑

#### Step 4: API 与优化（1 天）
- [ ] API 集成 + 虚拟滚动
- [ ] 错误处理完善

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **操作日志** | 未设计 | 新增操作日志记录 |
| **批量操作** | 未完善错误处理 | 新增部分失败结果展示 |
| **退款校验** | 未明确 | 新增金额校验规则 |
| **SLA 公式** | 未说明 | 明确计算公式 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（操作日志、批量错误处理、退款校验）
2. 建议修改项已补充（SLA 公式）
3. 风险等级：中（可控）
