# T020 套餐购买流程 — 修正版设计方案（V2）

> 原设计方案: [T020-plan-purchase-flow-spec.md](../T020-plan-purchase-flow-spec.md)
> 评审报告: [T020-plan-purchase-flow-Reviewer.md](../T020-plan-purchase-flow-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加支付安全校验 | 第 6 节用户交互流程 |
| 2 | 设计支付回调处理流程 | 第 3 节数据流 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 优惠券输入增加防抖 | 第 6 节用户交互流程 |
| 2 | 明确套餐对比设计 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "购买套餐"
    └── FormPageShell
        ├── PlanSelection
        │   └── 套餐选择
        ├── PlanComparison
        │   └── 套餐对比 【已修复】
        ├── CouponInput
        │   └── 优惠券输入 【已修复】
        └── PaymentPanel
            └── 支付确认 【已修复】
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Radio | 套餐选择 |
| Input | 优惠券输入 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| PlanCard | 套餐卡片 |
| PriceDisplay | 价格展示 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| PlanComparison | 套餐对比 | **新建** — 表格对比形式 |
| CouponInput | 优惠券输入 | **新建** |
| PaymentPanel | 支付面板 | **新建** |

---

### 3. 数据流（V2 修正）【已修复】

```
[进入购买页面]
    │
    ▼
[选择套餐]
    │
    ▼
[输入优惠券（可选）]
    │
    ▼
[确认支付]
    │
    ▼
[API: POST /api/v1/billing/orders]
    │
    ▼
[支付网关跳转]
    │
    ▼
[支付回调处理] 【新增】
    ├── 成功回调 ──► 查询订单状态 ──► 展示成功页面
    ├── 失败回调 ──► 展示失败页面 ──► 提供重试入口
    └── 用户手动返回 ──► 轮询订单状态 ──► 展示结果
```

---

### 4. 状态管理

```typescript
interface PurchaseState {
  step: 'select' | 'confirm' | 'paying' | 'success' | 'failed';
  selectedPlan: Plan | null;
  couponCode: string | null;
  discount: number;
  orderId: string | null;
  loading: boolean;
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 创建订单 | POST | `/api/v1/billing/orders` | 创建购买订单 |
| 查询订单 | GET | `/api/v1/billing/orders/:id` | 查询订单状态 |
| 验证优惠券 | POST | `/api/v1/billing/coupons/validate` | 验证优惠券 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入购买页面]
    │
    ▼
[选择套餐]
    │
    ▼
[查看套餐对比（表格形式）] 【已修复】
    │
    ▼
[输入优惠券（Debounce 500ms）] 【已修复】
    │
    ▼
[点击支付]
    │
    ▼
[二次确认 Modal] 【新增】
    │
    ▼
[支付网关跳转]
    │
    ▼
[支付结果处理] 【新增】
```

#### 支付安全校验（V2 新增）【已修复】

```
1. 支付前显示二次确认 Modal
2. 创建订单时生成唯一幂等键（idempotency key）
3. 订单创建后锁定表单，防止重复提交
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 优惠券无效 | 提示错误，不清除输入 |
| 支付失败 | 展示失败原因，提供重试 |
| 订单超时 | 自动取消，提示重新创建 |

---

### 8. 性能优化

- 套餐数据预加载
- 优惠券验证 Debounce

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 支付安全 | 高 | 支付信息泄露 | 幂等键 + 二次确认 |
| 重复支付 | 中 | 用户重复点击 | 表单锁定 |

---

### 10. 开发步骤拆分

#### Step 1: 套餐选择（0.5 天）
- [ ] PlanSelection + PlanComparison

#### Step 2: 优惠券（0.5 天）【已修复】
- [ ] CouponInput（Debounce 500ms）

#### Step 3: 支付流程（1.5 天）【已修复】
- [ ] PaymentPanel（含二次确认）
- [ ] 支付回调处理

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **支付安全** | 未设计 | 新增二次确认 + 幂等键 |
| **支付回调** | 未设计 | 新增回调处理流程 |
| **优惠券** | 实时验证 | 增加 Debounce 500ms |
| **套餐对比** | 未明确 | 明确表格对比形式 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（支付安全校验、支付回调处理）
2. 建议修改项已补充（Debounce、套餐对比）
3. 风险等级：中（可控）
