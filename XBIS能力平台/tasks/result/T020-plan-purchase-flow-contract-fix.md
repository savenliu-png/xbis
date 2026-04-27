# T020 套餐购买流程 - 接口契约级修复文档（Contract Fix）

## 一、问题分类

| # | 问题 | 分类 |
|---|------|------|
| P1 | 设计方案定义 API 为 `/api/v1/billing/*`，后端实际为 `/portal-api/v1/subscriptions/*` | 契约不一致 |
| P2 | 前端 `PlanDetail` 类型与后端 `SubscriptionPlanRow` 字段名不匹配 | 契约不一致 |
| P3 | 前端 `PriceCalculation` 类型与后端 `SubscriptionQuote` 字段名不匹配 | 契约不一致 |
| P4 | 前端 `PurchaseResponse` 类型与后端 `subscriptions/change` 返回结构不匹配 | 契约不一致 |
| P5 | 前端 `CouponInput` 调用 `validateCoupon` 但后端无此路由 | 契约缺失 |
| P6 | 前端展示支付方式选择（支付宝/微信/线下），但后端仅支持余额扣款 | 设计不明确 |
| P7 | 前端 `PurchaseResponse.status` 含 `pending`，但后端同步扣款无 pending | 设计不明确 |
| P8 | 前端 `autoRenew` 开关后端不接收 | 设计不明确 |
| P9 | `SubscriptionPlanRow`/`SubscriptionQuote` 类型定义在页面文件而非 shared/types | 契约缺失 |
| P10 | CouponInput 内部重复调用 calculate，但 change-quote 已返回 availableCoupons | 设计不明确 |

---

## 二、契约决策

| # | 问题 | 方案A | 方案B | 推荐方案 | 理由 |
|---|------|-------|-------|----------|------|
| P1 | API 路径不一致 | 前端适配后端已有路由 | 后端新增前端定义的路由 | **方案A** | 后端路由已上线且被其他页面使用，不能改 |
| P2 | PlanDetail 字段不匹配 | 前端做映射层 | 修改后端返回结构 | **方案A** | 后端返回被 Billing/Subscription 页面共用 |
| P3 | PriceCalculation 字段不匹配 | 前端做映射层 | 修改后端返回结构 | **方案A** | 同 P2 |
| P4 | PurchaseResponse 不匹配 | 前端做映射层 | 后端返回完整 PurchaseResponse | **方案A** | 后端已稳定运行，前端适配成本更低 |
| P5 | validateCoupon 路由缺失 | 前端改用 coupons/available + 前端匹配 | 后端新增 POST coupons/validate | **方案A** | 已新增 coupons/available 路由，前端匹配更灵活 |
| P6 | 支付方式 vs 余额扣款 | 保留 UI 但标记"余额支付"为唯一选项 | 移除支付方式选择，改为"余额扣款"提示 | **方案B** | 后端仅支持余额扣款，展示虚假选项会误导用户 |
| P7 | pending 状态不存在 | 保留 pending 用于未来扩展 | 移除 pending | **方案A** | 保留扩展性无害，usePolling 已支持 |
| P8 | autoRenew 后端不接收 | 前端保留 UI 但不传给后端 | 移除 autoRenew 开关 | **方案A** | 保留 UI 用于未来扩展，后端忽略无害 |
| P9 | 类型定义位置不当 | 移到 shared/types/index.ts | 保留在页面文件中 | **方案A** | shared/types 是项目类型统一出口 |
| P10 | CouponInput 重复调用 calculate | CouponInput 仅验证优惠券，由 PlanPurchasePage 统一调 calculate | CouponInput 内部调用 calculate | **方案A** | 职责单一，避免重复请求；change-quote 已返回 availableCoupons 可直接使用 |

---

## 三、修复内容

### 1️⃣ 代码修改

| 文件路径 | 修改内容 |
|----------|----------|
| `packages/shared/src/types/index.ts` | 新增 `SubscriptionPlanRow`, `SubscriptionQuote`, `SubscriptionChangeResult` 类型定义 |
| `packages/components/blocks/PaymentPanel/index.tsx` | 移除支付方式选择（支付宝/微信/线下转账），改为"账户余额扣款"提示；新增 `walletBalance`/`insufficientBalance` props；使用 `SubscriptionChangeResult` 类型映射 |
| `packages/components/blocks/CouponInput/index.tsx` | 简化职责：仅验证优惠券（从 `availableCoupons` 中匹配），不再调用 calculate API；回调签名从 `(coupon, priceCalc)` 改为 `(coupon)` |
| `packages/pages/billing/PlanPurchasePage.tsx` | 移除本地类型定义，改用 `@xbis/shared` 导出；新增 `availableCoupons`/`walletBalance`/`insufficientBalance` 状态；`calculatePrice` 从 quote 中提取 availableCoupons 和 walletBalance；`handleCouponApplied` 简化为仅接收 coupon，再调用 calculatePrice |
| `tasks/design-specs/final/T020-plan-purchase-flow-design-final.md` | 更新 API 路径、数据流、交互流程、边界与异常；新增设计补充说明 |

### 2️⃣ 设计方案更新

已更新 `T020-plan-purchase-flow-design-final.md`：

- **API 调用表**：从 5 个虚构路径修正为 5 个后端实际路由
- **数据流图**：从"支付网关跳转"改为"账户余额扣款"
- **交互流程**：优惠券验证改为"从 change-quote 返回的 availableCoupons 中匹配"
- **边界与异常**：新增"余额不足"和"降级风险"场景
- **设计补充说明**：5 条关键业务规则说明

### 3️⃣ 类型定义更新

新增 3 个后端契约类型到 `shared/types/index.ts`：

```typescript
// 后端 subscription_plans 表行映射
export interface SubscriptionPlanRow {
  id: string;
  planCode: string;
  planName: string;
  billingType: string;
  price: number;
  includedQuota: number;
  supportFeatures: string[] | Record<string, unknown>[];
  status: string;
  description: string;
  allowedApiIds: string[];
  saleEndAt: string | null;
  saleStatus: string;
  disabledReason: string | null;
}

// 后端 change-quote 返回结构
export interface SubscriptionQuote {
  quoteId: string;
  currentPlan: SubscriptionPlanRow | null;
  targetPlan: SubscriptionPlanRow;
  direction: 'new_subscription' | 'upgrade' | 'downgrade' | 'same_plan';
  originalAmount: number;
  proratedCredit: number;
  couponDeduction: number;
  payableAmount: number;
  riskNotes: string[];
  availableCoupons: Coupon[];
  walletBalance: number;
  insufficientBalance: boolean;
  selectedCoupon: Coupon | null;
}

// 后端 subscriptions/change 返回结构
export interface SubscriptionChangeResult {
  changed: boolean;
  planId: string;
  planCode: string;
  planName: string;
  payableAmount: number;
  orderNo: string;
}
```

---

## 四、影响评估

| 影响项 | 等级 | 说明 |
|--------|------|------|
| 前端页面 | 中 | PaymentPanel UI 变更（移除支付方式选择，新增余额扣款提示）；CouponInput 回调签名变更 |
| 后端接口 | 低 | 仅新增 `GET /coupons/available` 路由，不修改已有接口 |
| 后续任务 | 低 | 类型定义移到 shared/types，其他任务可直接复用 |
| 数据模型 | 低 | 前端类型不变，仅新增映射层 |

---

## 五、是否完成联调修复

👉 **YES**

所有 10 个契约问题已修复：
- 5 个契约不一致问题 → 前端适配后端已有路由 + 映射层
- 2 个契约缺失问题 → 类型移到 shared/types + 新增 coupons/available 路由
- 3 个设计不明确问题 → 契约决策后统一实现（移除虚假支付方式、简化 CouponInput、保留 pending/autoRenew 扩展性）

TypeScript 编译通过，设计方案已同步更新。
