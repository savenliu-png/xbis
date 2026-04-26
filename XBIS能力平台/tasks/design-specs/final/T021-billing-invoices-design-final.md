# T021 账单与发票 — 最终可开发设计方案

> 任务卡: [T021-billing-invoices.md](../T021-billing-invoices.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "账单与发票"
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框
        │   ├── 日期筛选
        │   └── 状态筛选
        ├── InvoiceTable
        │   └── 账单列表
        └── Pagination
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 账单表格 |
| Tag | 状态标签 |
| Pagination | 分页 |
| Modal | 弹窗 |
| Form | 表单容器 |
| Radio | 单选 |
| Skeleton | 加载骨架 |
| Empty | 空态 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| InvoiceCard | 发票卡片 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| InvoiceTable | 账单表格 | **新建** |
| FilterBar | 筛选栏 | **新建** — 含关键词搜索 |
| InvoiceApplyForm | 发票申请表单 | **新建** — 分组形式 |

---

## 3. 数据流

```
[进入 /invoices]
    │
    ▼
[API: GET /api/v1/billing/invoices]
    │
    ▼
[渲染账单列表]
```

---

## 4. 状态管理

```typescript
interface InvoiceListState {
  pageState: 'loading' | 'idle' | 'error';
  invoices: Invoice[];
  total: number;
  filters: { keyword?: string; startDate?: string; endDate?: string; status?: string; page: number; pageSize: number };
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 账单列表 | GET | `/api/v1/billing/invoices` | 支持筛选分页 |
| 申请发票 | POST | `/api/v1/billing/invoices` | 申请发票 |
| 下载发票 | GET | `/api/v1/billing/invoices/:id/download` | 下载发票 PDF |

---

## 6. 用户交互流程

```
[进入账单页面]
    │
    ▼
[浏览账单列表]
    │
    ├── 搜索/筛选
    ├── 点击下载（显示进度）
    └── 点击申请发票
            │
            ▼
        [分组表单]
            │
            ▼
        [提交申请]
```

### 发票申请表单分组

```
表单分组：
├── 发票类型（个人/企业单选）
├── 基本信息
│   ├── 抬头
│   ├── 税号
│   └── 邮箱
└── 企业信息（仅企业类型显示）
    ├── 地址
│   ├── 电话
│   ├── 开户行
│   └── 账号
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无账单 | Empty 组件 |
| 下载失败 | 提示错误，可重试 |

---

## 8. 性能优化

- 列表分页加载
- 下载进度提示

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 发票信息错误 | 中 | 发票信息填写错误 | 表单校验 |

---

## 10. 开发步骤拆分

### Step 1: 列表页（0.5 天）
- [ ] InvoiceTable + FilterBar（含搜索）

### Step 2: 申请表单（0.5 天）
- [ ] InvoiceApplyForm（分组形式）

### Step 3: 下载功能（0.5 天）
- [ ] 下载进度提示
