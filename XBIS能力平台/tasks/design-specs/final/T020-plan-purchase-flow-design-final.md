# T020 套餐购买流程 — 最终可开发设计方案

> 任务卡: [T020-plan-purchase-flow.md](../T020-plan-purchase-flow.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "购买套餐"
    └── FormPageShell
        ├── PlanSelection
        │   └── 套餐选择
        ├── PlanComparison
        │   └── 套餐对比
        ├── CouponInput
        │   └── 优惠券输入
        └── PaymentPanel
            └── 支付确认
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Card | 套餐卡片 | 否 |
| Radio | 单选 | 否 |
| Input | 优惠券输入 | 否 |
| Modal | 确认弹窗 | 否 |
| Steps | 步骤条 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| PlanCard | 套餐卡片 | 否 |
| PriceDisplay | 价格展示 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| PlanComparison | 套餐对比区 | 是 |
| CouponInput | 优惠券输入区 | 是 |
| PaymentPanel | 支付面板 | 是 |
| ResultPanel | 结果面板 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[进入购买页面]
    │
    ▼
[选择套餐] ──► GET /portal-api/v1/subscriptions/plans
    │
    ▼
[费用计算] ──► POST /portal-api/v1/subscriptions/change-quote
    │              返回: originalAmount, proratedCredit, couponDeduction,
    │                    payableAmount, availableCoupons, walletBalance,
    │                    insufficientBalance, riskNotes
    ▼
[输入优惠券（可选）] ──► GET /portal-api/v1/coupons/available
    │                      从 availableCoupons 中匹配 couponNo
    │                      匹配后重新调用 change-quote 计算折扣
    ▼
[确认支付] ──► 二次确认 Modal
    │
    ▼
[余额扣款] ──► POST /portal-api/v1/subscriptions/change
    │              返回: changed, planId, planCode, planName,
    │                    payableAmount, orderNo
    ▼
[支付结果处理]
    ├── 成功 ──► 展示成功页面
    ├── 失败 ──► 展示失败页面 ──► 提供重试入口
    └── 余额不足 ──► 提示充值
```

---

## 4. 状态管理

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

## 5. API 调用

| API | Method | Path | 说明 | 后端路由来源 |
|-----|--------|------|------|-------------|
| 套餐列表 | GET | `/portal-api/v1/subscriptions/plans` | 获取套餐列表 | userP0.ts |
| 费用报价 | POST | `/portal-api/v1/subscriptions/change-quote` | 计算费用（含优惠券折扣） | userP0.ts |
| 可用优惠券 | GET | `/portal-api/v1/coupons/available` | 获取可用优惠券列表 | userP0.ts |
| 变更套餐 | POST | `/portal-api/v1/subscriptions/change` | 余额扣款变更套餐 | userP0.ts |
| 查询订单 | GET | `/portal-api/v1/billing/orders/:orderNo` | 查询订单状态 | userP0.ts |

### 设计补充说明

1. **支付方式**：后端当前仅支持账户余额扣款，不支持第三方支付网关。PaymentPanel 展示"账户余额扣款"而非支付方式选择。
2. **优惠券验证**：通过 `change-quote` 返回的 `availableCoupons` 字段获取可用优惠券列表，CouponInput 从中匹配用户输入的 couponNo，无需单独调用 validate API。
3. **余额不足提示**：`change-quote` 返回 `insufficientBalance` 和 `walletBalance`，前端据此展示余额不足提示。
4. **autoRenew**：前端保留自动续费开关 UI，但后端当前不接收此参数，预留未来扩展。
5. **订单状态**：后端 `subscriptions/change` 为同步扣款，成功直接返回 `changed: true`，无 pending 状态。前端映射为 `status: 'paid'`。

---

## 6. 用户交互流程

```
[进入购买页面]
    │
    ▼
[选择套餐]
    │
    ▼
[查看套餐对比（表格形式）]
    │
    ▼
[输入优惠券（Debounce 500ms）]
    │  从 change-quote 返回的 availableCoupons 中匹配
    ▼
[点击支付]
    │
    ▼
[二次确认 Modal]
    │
    ▼
[账户余额扣款]
    │  POST /subscriptions/change 同步扣款
    ▼
[支付结果处理]
```

### 支付安全校验

```
1. 支付前显示二次确认 Modal
2. 创建订单时生成唯一幂等键（idempotency key）
3. 订单创建后锁定表单，防止重复提交
```

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 优惠券无效 | 提示错误，不清除输入 |
| 余额不足 | 展示余额不足提示，引导充值 |
| 支付失败 | 展示失败原因，提供重试 |
| 订单超时 | 自动取消，提示重新创建 |
| 优惠券叠加 | 不支持叠加 |
| 降级风险 | change-quote 返回 riskNotes，降级需二次确认 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 费用计算 < 300ms
- 套餐数据预加载
- 优惠券验证 Debounce
- 无内存泄漏

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 支付安全 | 高 | 支付信息泄露 | 幂等键 + 二次确认 |
| 重复支付 | 中 | 用户重复点击 | 表单锁定 |
| 优惠券滥用 | 中 | 可能重复使用优惠券 | 后端校验 |
| 并发下单 | 低 | 并发可能导致重复订单 | 幂等处理 |

---

## 10. 开发步骤拆分

### Step 1: 套餐选择（0.5 天）
- [ ] PlanSelection + PlanComparison

### Step 2: 优惠券（0.5 天）
- [ ] CouponInput（Debounce 500ms）

### Step 3: 支付流程（1.5 天）
- [ ] PaymentPanel（含二次确认）
- [ ] 支付回调处理

### Step 4: API 集成 & 状态管理（0.5 天）
- [ ] 集成 API 调用
- [ ] 实现支付流程

### Step 5: 性能优化 & 测试（0.5 天）
- [ ] 自测
