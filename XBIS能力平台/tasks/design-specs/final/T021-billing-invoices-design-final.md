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

---

## 11. 契约补充说明（Contract Fix D1）

> 以下内容基于后端实际 API 契约（`packages/server/src/routes/userP0.ts`）与前端类型对齐后的补充。

### 11.1 API 契约确认

| 接口 | URL | Method | 说明 |
|------|-----|--------|------|
| 发票列表 | `/portal-api/v1/invoices` | GET | 参数: `{ page?, pageSize?, status? }`，返回: `{ items: InvoiceReview[], total: number }` |
| 发票详情 | `/portal-api/v1/invoices/:invoiceNo` | GET | 返回: `InvoiceReview & { orderNos: string[] }` |
| 发票申请 | `/portal-api/v1/invoices/apply` | POST | 参数: `InvoiceApplyRequest`，返回: `{ invoiceNo, invoiceId, status: 'applied' }` |
| 发票下载 | `/portal-api/v1/invoices/:invoiceNo/download` | GET | 返回: `{ invoiceNo, downloadUrl }` |
| 可开票账单 | `/portal-api/v1/invoices/available-billing-orders` | GET | 返回: `{ items: AvailableBillingOrder[] }` |

### 11.2 字段命名约定（以后端实际契约为准）

| 任务卡定义 | 后端实际字段 | 决策 |
|-----------|------------|------|
| `enterprise` | `company` | 以 `company` 为准（后端 CHECK 约束） |
| `taxNumber` | `taxNo` | 以 `taxNo` 为准（后端列名） |
| `email` | `recipientEmail` | 以 `recipientEmail` 为准（语义化前缀） |
| `phone` | `recipientPhone` | 以 `recipientPhone` 为准（语义化前缀） |
| `orderIds` | `orderNos` | 以 `orderNos` 为准（与 BillingOrder.orderNo 一致） |
| `invoiceId` | `invoiceNo` | 以 `invoiceNo` 为准（业务编号） |
| `pending/processing/completed/failed` | `draft/applied/issued/downloaded/cancelled` | 以后端 5 态为准（更细粒度） |
| — | `bankName` | 新增（企业发票开户行） |
| — | `bankAccount` | 新增（企业发票银行账号） |

### 11.3 修复记录

| 修复项 | 类型 | 说明 |
|--------|------|------|
| `available-billing-orders` SQL 缺少 camelCase 映射 | 后端 Bug | `SELECT bo.*` → 显式列名 + AS 映射 |
| `InvoiceApplyResponse` 缺少 `invoiceId` | 前端类型 | 后端返回 `invoiceId`，前端类型已补充 |
| `InvoiceApplyForm` 无重试机制 | 前端 UX | 加载失败时 Alert 添加重试按钮 |
| `InvoiceApplyRequest` 缺少 `bankName/bankAccount` | 双端 | 类型+表单+后端 INSERT+DB schema 同步补充 |
| `InvoiceReview` 缺少 `bankName/bankAccount` | 前端类型 | 详情展示已补充 |
