# T021 账单与发票 — 工程级设计方案

> 任务卡: [T021-billing-invoices.md](../T021-billing-invoices.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 标题: "账单与发票"
│   └── Tabs: 账单 / 发票
├── ListPageShell
│   ├── FilterBar
│   │   ├── 日期筛选
│   │   └── 状态筛选
│   ├── ContentArea
│   │   ├── InvoiceTable (账单表格)
│   │   └── InvoiceCard[] (发票卡片)
│   └── Pagination
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Table | 账单表格 | 否 |
| Tag | 状态标签 | 否 |
| Modal | 申请弹窗 | 否 |
| Form | 表单容器 | 否 |
| Input | 文本输入 | 否 |
| Select | 下拉选择 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |
| Pagination | 分页 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| InvoiceCard | 账单卡片 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| InvoiceList | 账单列表 | 是 |
| InvoiceDetail | 账单详情 | 是 |
| InvoiceApplyForm | 发票申请表单 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| ListPageShell | 列表页模板 |

---

## 3. 数据流

```
[页面加载]
  ├── 调用 GET /api/v1/billing/invoices ──► 获取账单列表
  └── 调用 GET /api/v1/billing/invoices/records ──► 获取发票列表

[用户筛选]
  ├── 选择日期 ──► 更新 filter.dateRange ──► 重新查询
  └── 选择状态 ──► 更新 filter.status ──► 重新查询

[用户申请发票]
  ├── 填写申请表单
  └── 调用 POST /api/v1/billing/invoices/apply

[用户下载]
  ├── 点击下载账单 ──► 调用 GET /api/v1/billing/invoices/:id/download
  └── 点击下载发票 ──► 调用 GET /api/v1/billing/invoices/:id/download
```

---

## 4. 状态管理

### 页面状态

```typescript
interface BillingInvoicesState {
  // 数据状态
  invoices: InvoiceItem[];
  invoiceRecords: Invoice[];
  total: number;

  // 筛选状态
  filter: {
    dateRange?: [string, string];
    status?: string;
  };

  // 分页状态
  pagination: {
    page: number;
    pageSize: number;
  };

  // UI 状态
  activeTab: 'invoices' | 'records';
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/billing/invoices | 页面加载、筛选变化 | page, pageSize, status | 显示错误提示 |
| GET /api/v1/billing/invoices/records | 页面加载 | - | 显示错误提示 |
| POST /api/v1/billing/invoices/apply | 申请发票 | InvoiceApplyRequest | 提示申请失败 |
| GET /api/v1/billing/invoices/:id/download | 点击下载 | invoiceId | 提示下载失败 |

---

## 6. 用户交互流程

### 主流程

```
用户进入账单与发票
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示账单/发票列表
  │   ├── 用户筛选 ──► 更新列表
  │   ├── 用户申请发票 ──► 打开发票申请表单
  │   └── 用户下载 ──► 下载 PDF
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 无账单 | 无消费记录 | 显示空态 | 提示消费后查看 |
| 申请失败 | 信息不完整 | 返回错误 | 提示补充信息 |
| 下载失败 | 文件不存在 | 返回错误 | 提示联系客服 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 大量账单 | 支持分页 |
| 多币种 | 支持货币切换 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 下载响应 < 3s
- 无内存泄漏

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 发票信息错误 | 中 | 发票信息可能填写错误 | 信息校验 |
| 下载文件大 | 低 | PDF 文件可能较大 | 流式下载 |
| 发票重复申请 | 中 | 可能重复申请发票 | 后端幂等校验 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 ListPageShell 搭建页面框架
- [ ] 配置 Tabs

### Step 2: 账单列表开发（1 人日）
- [ ] InvoiceTable 组件
- [ ] 账单卡片展示

### Step 3: 发票列表开发（0.5 人日）
- [ ] 发票记录展示
- [ ] 发票状态跟踪

### Step 4: 发票申请开发（0.5 人日）
- [ ] InvoiceApplyForm 组件
- [ ] 申请表单校验

### Step 5: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现下载功能

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 自测
