# T021 账单与发票 — 接口联调检查报告（D1）

## 🧪 联调结果（D1）

👉 状态：**前端待修**

---

## 1️⃣ API契约一致性

### 1.1 URL 对比

| 接口 | 任务卡定义 URL | 代码实现 URL | 是否一致 |
|------|---------------|-------------|---------|
| 发票列表 | `/api/v1/billing/invoices` | `/portal-api/v1/invoices` | ❌ 不一致 |
| 发票申请 | `/api/v1/billing/invoices/apply` | `/portal-api/v1/invoices/apply` | ❌ 不一致 |
| 发票下载 | `/api/v1/billing/invoices/:id/download` | `/portal-api/v1/invoices/:invoiceNo/download` | ❌ 不一致 |
| 发票详情 | 未定义 | `/portal-api/v1/invoices/:invoiceNo` | ⚠️ 代码有但任务卡未定义 |
| 可开票账单 | 未定义 | `/portal-api/v1/invoices/available-billing-orders` | ⚠️ 代码有但任务卡未定义 |

**说明**：任务卡定义的 URL 前缀为 `/api/v1/billing/invoices`，代码实现使用 `/portal-api/v1/invoices`。这是因为项目实际 API 网关路由与任务卡设计时不同——代码实现遵循了项目已有的 `portal-api/v1` 路由约定（与 `userApi.billing.*`、`userApi.coupons.*` 等其他模块一致）。**以代码实现为准，任务卡 URL 为设计期参考。**

### 1.2 Method 对比

| 接口 | 任务卡 Method | 代码 Method | 是否一致 |
|------|-------------|------------|---------|
| 发票列表 | GET | GET | ✅ |
| 发票申请 | POST | POST | ✅ |
| 发票下载 | GET | GET | ✅ |
| 发票详情 | — | GET | ✅ |
| 可开票账单 | — | GET | ✅ |

### 1.3 参数对比

#### 发票列表 `list`

| 参数 | 任务卡定义 | 代码实现 | 是否一致 |
|------|-----------|---------|---------|
| page | `page?: number` | `page?: number` | ✅ |
| pageSize | `pageSize?: number` | `pageSize?: number` | ✅ |
| status | `status?: string` | `status?: string` | ✅ |

#### 发票申请 `apply`

| 参数 | 任务卡定义 | 代码实现 | 是否一致 |
|------|-----------|---------|---------|
| invoiceType | `'personal' \| 'enterprise'` | `'personal' \| 'company'` | ❌ 枚举值不一致 |
| title | `string` | `string` | ✅ |
| taxNumber | `taxNumber?: string` | `taxNo?: string` | ❌ 字段名不一致 |
| email | `email: string`（必填） | `recipientEmail?: string`（可选） | ❌ 字段名+必填性不一致 |
| address | `address?: string` | `address?: string` | ✅ |
| phone | `phone?: string` | `recipientPhone?: string` | ❌ 字段名不一致 |
| bankName | `bankName?: string` | — | ❌ 代码缺失 |
| bankAccount | `bankAccount?: string` | — | ❌ 代码缺失 |
| orderIds | `orderIds: string[]` | `orderNos: string[]` | ❌ 字段名不一致 |

#### 发票下载 `download`

| 参数 | 任务卡定义 | 代码实现 | 是否一致 |
|------|-----------|---------|---------|
| id | path param: `:id` | path param: `:invoiceNo` | ⚠️ 参数名不同但功能等价 |

---

## 2️⃣ 返回结构匹配

### 2.1 Invoice 类型 vs 任务卡 Invoice 定义

| 字段 | 任务卡定义 | 代码 Invoice 类型 | 是否一致 |
|------|-----------|-----------------|---------|
| id | `string` | `string` | ✅ |
| invoiceId | `string` | — | ❌ 代码使用 `invoiceNo` 代替 |
| invoiceNumber | `string` | — | ❌ 代码无此字段 |
| invoiceType | `'personal' \| 'enterprise'` | `'personal' \| 'company'` | ❌ 枚举值不一致 |
| title | `string` | `string` | ✅ |
| taxNumber | `string?` | `taxNo?: string` | ❌ 字段名不一致 |
| amount | `number` | `number` | ✅ |
| currency | `string` | — | ❌ 代码缺失 |
| status | `'pending' \| 'processing' \| 'completed' \| 'failed'` | `'draft' \| 'applied' \| 'issued' \| 'downloaded' \| 'cancelled'` | ❌ 完全不一致 |
| downloadUrl | `string?` | `string?` | ✅ |
| createdAt | `string` | — | ❌ 代码使用 `appliedAt` |
| completedAt | `string?` | `issuedAt?: string` | ❌ 字段名不一致 |

### 2.2 InvoiceApplyResponse 类型

| 字段 | 任务卡定义 | 代码实现 | 是否一致 |
|------|-----------|---------|---------|
| invoiceId | `string` | — | ❌ 代码使用 `invoiceNo` |
| status | `'pending'` | `string` | ⚠️ 代码类型过于宽泛 |
| estimatedCompletion | `string` | — | ❌ 代码缺失 |

### 2.3 AvailableBillingOrder 类型

| 字段 | 任务卡定义 | 代码实现 | 是否一致 |
|------|-----------|---------|---------|
| — | 未定义 | `orderNo: string` | ⚠️ 任务卡未定义此类型 |
| — | 未定义 | `payableAmount: number` | ⚠️ 同上 |
| — | 未定义 | `totalAmount: number` | ⚠️ 同上 |
| — | 未定义 | `billingPeriod?: string` | ⚠️ 同上 |
| — | 未定义 | `status?: string` | ⚠️ 同上 |

---

## 3️⃣ 数据结构一致性

### 3.1 核心冲突清单

| # | 冲突类型 | 任务卡定义 | 代码实现 | 影响等级 |
|---|---------|-----------|---------|---------|
| 1 | 枚举值 | `invoiceType: 'enterprise'` | `invoiceType: 'company'` | 🔴 高 — 影响申请提交 |
| 2 | 字段名 | `taxNumber` | `taxNo` | 🔴 高 — 影响申请提交 |
| 3 | 字段名 | `email` (必填) | `recipientEmail` (可选) | 🔴 高 — 影响申请提交 |
| 4 | 字段名 | `phone` | `recipientPhone` | 🟡 中 — 影响申请提交 |
| 5 | 字段名 | `orderIds` | `orderNos` | 🔴 高 — 影响申请提交 |
| 6 | 缺失字段 | `bankName`, `bankAccount` | — | 🟡 中 — 企业发票可能需要 |
| 7 | 缺失字段 | `currency` | — | 🟢 低 — 当前仅支持 CNY |
| 8 | 状态枚举 | `pending/processing/completed/failed` | `draft/applied/issued/downloaded/cancelled` | 🔴 高 — 影响状态展示 |
| 9 | 字段名 | `invoiceId` | `invoiceNo` | 🟡 中 — 影响数据映射 |
| 10 | 缺失字段 | `estimatedCompletion` | — | 🟢 低 — 仅展示用 |

### 3.2 判定依据

代码实现中的类型定义（`Invoice`, `InvoiceReview`, `InvoiceApplyRequest`）是**项目已有的稳定类型**，存在于 `packages/shared/src/types/index.ts` 中，且被 `userApi.invoices.*` 接口使用。这些类型与后端 API 实际返回的数据结构是对齐的。

任务卡中的数据结构是**设计期定义**，与后端实际实现存在差异。**应以代码实现（即后端实际契约）为准**。

---

## 4️⃣ 状态流一致性

### 4.1 任务卡定义的状态

```
pending → processing → completed
                    → failed
```

### 4.2 代码实现的状态

```
draft → applied → issued → downloaded
                 → cancelled
```

### 4.3 状态映射关系

| 任务卡状态 | 代码实际状态 | 语义是否等价 |
|-----------|------------|------------|
| pending | draft / applied | ⚠️ 任务卡 pending 对应两个代码状态 |
| processing | applied | ⚠️ 语义近似 |
| completed | issued / downloaded | ⚠️ 代码细分了"已开具"和"已下载" |
| failed | cancelled | ⚠️ 语义不同：cancelled 是用户取消，failed 是系统失败 |

### 4.4 INVOICE_STATUS_CONFIG 覆盖检查

| 代码状态 | INVOICE_STATUS_CONFIG | 前端展示 | 是否完整 |
|---------|----------------------|---------|---------|
| draft | ✅ `草稿 / default` | ✅ | ✅ |
| applied | ✅ `已申请 / info` | ✅ | ✅ |
| issued | ✅ `已开具 / warning` | ✅ | ✅ |
| downloaded | ✅ `已下载 / success` | ✅ | ✅ |
| cancelled | ✅ `已取消 / error` | ✅ | ✅ |

**结论**：代码状态枚举比任务卡更细粒度，且全部有对应的前端展示配置。代码实现的状态流更贴合实际业务（区分"已申请"和"已开具"，区分"已下载"和"已开具"）。**以代码为准，无需修改。**

---

## 5️⃣ 错误处理

| 检查项 | Invoices.tsx | InvoiceApplyForm | InvoiceTable | 是否完整 |
|--------|-------------|-----------------|-------------|---------|
| API error | ✅ try/catch + message.error | ✅ try/catch + ordersError state | — | ✅ |
| 空数据 | ✅ ListPageShell status='empty' | ✅ Select placeholder | ✅ (由 ListPageShell 处理) | ✅ |
| loading | ✅ ListPageShell status='loading' | ✅ ordersLoading + Select loading | ✅ loading prop | ✅ |
| 超时/失败 | ✅ catch 兜底 | ✅ catch 兜底 | — | ✅ |
| 重试 | ✅ onRetry → loadData | ❌ 无重试按钮 | — | ⚠️ |

**InvoiceApplyForm 的可开票账单加载失败后无重试机制**，仅显示 Alert 但无法重新加载。

---

## 6️⃣ Business Services 层检查

| API 调用 | 是否通过 Service 层 | Service 方法 | 是否绕过 |
|---------|-------------------|-------------|---------|
| 发票列表 | ✅ | `userApi.invoices.list()` | 否 |
| 发票详情 | ✅ | `userApi.invoices.detail()` | 否 |
| 发票申请 | ✅ | `userApi.invoices.apply()` | 否 |
| 发票下载 | ✅ | `userApi.invoices.download()` | 否 |
| 可开票账单 | ✅ | `userApi.invoices.availableBillingOrders()` | 否 |

**结论**：全部通过 Business Services 层调用，未绕过。

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 数据结构 | `invoiceType` 枚举值 `'enterprise'` vs `'company'` | types/index.ts InvoiceApplyRequest | 任务卡写 enterprise，代码写 company；若后端接受 enterprise 则前端提交错误 |
| 数据结构 | `taxNumber` vs `taxNo` 字段名不一致 | types/index.ts InvoiceApplyRequest | 若后端期望 taxNumber 则前端提交字段名错误 |
| 数据结构 | `email`(必填) vs `recipientEmail`(可选) 字段名+必填性不一致 | types/index.ts InvoiceApplyRequest | 若后端期望 email 则前端提交字段名错误；且邮箱应为必填 |
| 数据结构 | `phone` vs `recipientPhone` 字段名不一致 | types/index.ts InvoiceApplyRequest | 若后端期望 phone 则前端提交字段名错误 |
| 数据结构 | `orderIds` vs `orderNos` 字段名不一致 | types/index.ts InvoiceApplyRequest | 若后端期望 orderIds 则前端提交字段名错误 |
| 数据结构 | 缺少 `bankName`, `bankAccount` 字段 | types/index.ts InvoiceApplyRequest | 企业发票可能需要开户行信息 |
| 数据结构 | `InvoiceApplyResponse` 缺少 `estimatedCompletion` 字段 | types/index.ts | 申请成功后无法展示预计完成时间 |
| 状态枚举 | 任务卡 `pending/processing/completed/failed` vs 代码 `draft/applied/issued/downloaded/cancelled` | types/index.ts Invoice.status | 状态语义不匹配，但代码更贴合实际业务 |
| 错误处理 | InvoiceApplyForm 加载可开票账单失败后无重试 | InvoiceApplyForm/index.tsx | 用户无法恢复加载 |
| API契约 | 任务卡 URL 前缀 `/api/v1/billing/` vs 代码 `/portal-api/v1/` | services.ts | 路由前缀不同，以代码为准 |

---

## 🛠 修改建议

### 前端修改（必须确认后端实际契约后执行）

**核心原则**：任务卡是设计期文档，代码类型是项目已有的稳定定义。以下修改需要**与后端确认实际 API 契约**后决定是否执行。

#### 1. 确认 `InvoiceApplyRequest` 字段名（🔴 高优先级）

需与后端确认以下字段的实际名称：

| 代码当前 | 后端可能期望 | 建议 |
|---------|------------|------|
| `invoiceType: 'company'` | `'enterprise'`? | 确认后端枚举值 |
| `taxNo` | `taxNumber`? | 确认后端字段名 |
| `recipientEmail` (可选) | `email` (必填)? | 确认字段名和必填性 |
| `recipientPhone` | `phone`? | 确认后端字段名 |
| `orderNos` | `orderIds`? | 确认后端字段名 |

#### 2. 补充 `bankName` / `bankAccount`（🟡 中优先级）

如果后端支持企业发票的开户行信息，需在 `InvoiceApplyRequest` 和 `InvoiceApplyForm` 中补充：

```typescript
// types/index.ts — InvoiceApplyRequest 补充
bankName?: string;
bankAccount?: string;
```

```tsx
// InvoiceApplyForm — 企业信息分组补充
<AntdForm.Item name="bankName" label="开户行">
  <Input placeholder="请输入开户行" />
</AntdForm.Item>
<AntdForm.Item name="bankAccount" label="银行账号">
  <Input placeholder="请输入银行账号" />
</AntdForm.Item>
```

#### 3. InvoiceApplyForm 添加重试机制（🟡 中优先级）

```tsx
// InvoiceApplyForm — 可开票账单加载失败时增加重试按钮
{ordersError && (
  <Alert
    variant="error"
    message={ordersError}
    action={<Button size="small" onClick={loadOrders}>重试</Button>}
    style={{ marginBottom: space['3'] }}
  />
)}
```

需要将 `loadOrders` 从 `useEffect` 内部提取为组件级函数。

#### 4. `InvoiceApplyResponse` 补充 `estimatedCompletion`（🟢 低优先级）

```typescript
export interface InvoiceApplyResponse {
  invoiceNo: string;
  status: string;
  estimatedCompletion?: string;
}
```

### 后端修改

无需后端修改。前端类型定义应与后端实际返回对齐，当前代码中的类型是项目已有的稳定定义。

### 数据结构调整

如果后端确认使用任务卡中定义的字段名（`taxNumber`, `email`, `phone`, `orderIds`, `enterprise`），则需要同步修改：

1. `packages/shared/src/types/index.ts` — 更新 `InvoiceApplyRequest` 字段名
2. `packages/components/blocks/InvoiceApplyForm/index.tsx` — 更新表单字段名
3. `packages/shared/src/constants/index.ts` — 更新 `INVOICE_TYPE_LABEL` 枚举映射

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 字段名不匹配导致申请提交失败 | 🔴 高 | 如果后端期望 `taxNumber` 而前端发送 `taxNo`，申请将失败 |
| 枚举值不匹配导致申请提交失败 | 🔴 高 | 如果后端期望 `enterprise` 而前端发送 `company`，申请将失败 |
| 邮箱必填性不匹配 | 🟡 中 | 如果后端要求 `email` 必填，前端可能提交空值 |
| 状态枚举不匹配 | 🟢 低 | 代码状态比任务卡更细粒度，展示无问题 |
| 缺少开户行字段 | 🟢 低 | 当前不影响核心流程，后续可补充 |

### 是否影响后续任务

- **不影响数据模型**：代码中的类型定义是项目稳定类型，不会被其他任务修改
- **可能影响 T019 计费概览**：Billing 页面中引用了 `BillingOrder` 类型，与发票关联的 `AvailableBillingOrder` 是独立类型，不影响

---

## ✅ 是否允许进入验收（D2）

👉 **NO** — 需先与后端确认以下 3 项关键契约后才能进入验收：

1. **`InvoiceApplyRequest` 字段名**：确认后端实际接受 `taxNo` 还是 `taxNumber`、`recipientEmail` 还是 `email`、`recipientPhone` 还是 `phone`、`orderNos` 还是 `orderIds`
2. **`invoiceType` 枚举值**：确认后端接受 `'company'` 还是 `'enterprise'`
3. **Invoice 状态枚举**：确认后端返回的状态是 `draft/applied/issued/downloaded/cancelled` 还是 `pending/processing/completed/failed`

确认后，根据结果执行修改，再进入 D2 验收。
