# T021 账单与发票 — 修正版设计方案（V2）

> 原设计方案: [T021-billing-invoices-spec.md](../T021-billing-invoices-spec.md)
> 评审报告: [T021-billing-invoices-Reviewer.md](../T021-billing-invoices-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 发票申请表单分组 | 第 2 节组件拆分 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加下载进度提示 | 第 6 节用户交互流程 |
| 2 | 增加发票状态轮询 | 第 3 节数据流 |
| 3 | 增加关键词搜索 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "账单与发票"
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框 【新增】
        │   ├── 日期筛选
        │   └── 状态筛选
        ├── InvoiceTable
        │   └── 账单列表
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 账单表格 |
| Tag | 状态标签 |
| Pagination | 分页 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| InvoiceCard | 发票卡片 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| InvoiceTable | 账单表格 | **新建** |
| FilterBar | 筛选栏 | **新建** — 含关键词搜索 |
| InvoiceApplyForm | 发票申请表单 | **新建** — 分组形式 |

---

### 3. 数据流

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

### 4. 状态管理

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

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 账单列表 | GET | `/api/v1/billing/invoices` | 支持筛选分页 |
| 申请发票 | POST | `/api/v1/billing/invoices` | 申请发票 |
| 下载发票 | GET | `/api/v1/billing/invoices/:id/download` | 下载发票 PDF |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入账单页面]
    │
    ▼
[浏览账单列表]
    │
    ├── 搜索/筛选 【已修复】
    ├── 点击下载（显示进度）【已修复】
    └── 点击申请发票
            │
            ▼
        [分组表单] 【已修复】
            │
            ▼
        [提交申请]
```

#### 发票申请表单分组（V2 修正）【已修复】

```
表单分组：
├── 发票类型（个人/企业单选）
├── 基本信息
│   ├── 抬头
│   ├── 税号
│   └── 邮箱
└── 企业信息（仅企业类型显示）
    ├── 地址
    ├── 电话
    ├── 开户行
    └── 账号
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无账单 | Empty 组件 |
| 下载失败 | 提示错误，可重试 |

---

### 8. 性能优化

- 列表分页加载
- 下载进度提示

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 发票信息错误 | 中 | 发票信息填写错误 | 表单校验 |

---

### 10. 开发步骤拆分

#### Step 1: 列表页（0.5 天）
- [ ] InvoiceTable + FilterBar（含搜索）

#### Step 2: 申请表单（0.5 天）【已修复】
- [ ] InvoiceApplyForm（分组形式）

#### Step 3: 下载功能（0.5 天）【已修复】
- [ ] 下载进度提示

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **发票表单** | 9 个字段堆叠 | 分组形式（类型/基本信息/企业信息） |
| **关键词搜索** | 未设计 | 新增搜索框 |
| **下载进度** | 未设计 | 新增进度提示 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（发票表单分组）
2. 建议修改项已补充（搜索、下载进度）
3. 风险等级：低
