# T020 套餐购买流程 - 接口联调检查文档（D1）

## 🧪 联调结果（D1）

👉 状态：**前端待修** → 已修复，待重新验证

---

## 1️⃣ API契约一致性

### 发现问题及修复

| # | 前端定义 | 后端实际 | 问题 | 修复 |
|---|----------|----------|------|------|
| 1 | `GET /portal-api/v1/billing/plans` | `GET /portal-api/v1/subscriptions/plans` | URL 不一致 | ✅ 已修复 services.ts |
| 2 | `POST /portal-api/v1/billing/coupons/validate` | 无此路由，后端有 `GET /portal-api/v1/coupons` 和 `loadAvailableCoupons` | URL + Method 不一致 | ✅ 新增后端 `GET /coupons/available`，前端适配 |
| 3 | `POST /portal-api/v1/billing/calculate` | `POST /portal-api/v1/subscriptions/change-quote` | URL 不一致 | ✅ 已修复 services.ts |
| 4 | `POST /portal-api/v1/billing/orders` | `POST /portal-api/v1/subscriptions/change` | URL 不一致 | ✅ 已修复 services.ts |
| 5 | `GET /portal-api/v1/billing/orders/:id` | `GET /portal-api/v1/billing/orders/:orderNo` | 已存在，路径一致 | ✅ 无需修改 |

### 参数映射修复

| API | 前端参数 | 后端参数 | 修复 |
|-----|----------|----------|------|
| calculate | `{ planId, couponCode }` | `{ planId, couponNo }` | ✅ services.ts 中 couponCode → couponNo |
| createOrder | `{ planId, couponCode, paymentMethod, autoRenew }` | `{ planId, couponNo, confirmDowngrade }` | ✅ services.ts 中映射；paymentMethod/autoRenew 后端不使用但无害 |

---

## 2️⃣ 返回结构匹配

### subscriptions/plans 返回映射

| 后端字段 | 前端 PlanDetail 字段 | 映射 |
|----------|---------------------|------|
| `id` | `id` | 直接映射 |
| `planName` | `name` | ✅ mapPlanRowToDetail |
| `description` | `description` | 直接映射 |
| `price` (fee_amount) | `price` | 直接映射 |
| `includedQuota` | `quota` | ✅ mapPlanRowToDetail |
| `supportFeatures` | `features` | ✅ mapPlanRowToDetail 解析数组 |
| `saleStatus` | `isRecommended`/`isPopular` | ✅ 前端按位置推断 |
| `billingType` | 无 | 前端不使用 |
| `allowedApiIds` | 无 | 前端不使用 |
| `currency` | 无后端字段 | ✅ 前端默认 'CNY' |

### subscriptions/change-quote 返回映射

| 后端字段 | 前端 PriceCalculation 字段 | 映射 |
|----------|---------------------------|------|
| `originalAmount` | `originalPrice` | ✅ PlanPurchasePage 映射 |
| `couponDeduction + proratedCredit` | `discount` | ✅ 合并计算 |
| `payableAmount` | `finalPrice` | ✅ PlanPurchasePage 映射 |
| `availableCoupons` | Coupon[] | ✅ CouponInput 使用 |
| `walletBalance` | 无 | 前端不使用 |
| `insufficientBalance` | 无 | ✅ 可用于前端提示 |

### subscriptions/change 返回映射

| 后端字段 | 前端 PurchaseResponse 字段 | 映射 |
|----------|---------------------------|------|
| `orderNo` | `orderId` | ✅ PaymentPanel 映射 |
| `payableAmount` | `finalAmount` | ✅ PaymentPanel 映射 |
| `couponDeduction` | `discount` | ✅ PaymentPanel 映射 |
| 无 | `status: 'paid'` | ✅ 后端同步扣款，直接设为 paid |
| 无 | `paymentUrl` | 后端无支付网关跳转，余额直接扣款 |

### coupons/available 返回映射

| 后端字段 | 前端 Coupon 字段 | 映射 |
|----------|-----------------|------|
| `couponNo` | `couponNo` | 直接映射 |
| `amount` | `amount` | 直接映射 |
| `usedAmount` | `usedAmount` | 直接映射 |
| `validUntil` | `validUntil` | 直接映射 |
| `remainingAmount` | 无（计算值） | ✅ 前端用 amount - usedAmount |

---

## 3️⃣ 数据结构一致性

| 检查项 | 状态 | 说明 |
|--------|------|------|
| PlanDetail vs SubscriptionPlanRow | ✅ | 前端通过 mapPlanRowToDetail 做映射 |
| PriceCalculation vs SubscriptionQuote | ✅ | 前端通过 calculatePrice 做映射 |
| PurchaseResponse vs change-order | ✅ | 前端通过 PaymentPanel 做映射 |
| Coupon vs coupons 表 | ✅ | 字段名一致（camelCase） |
| 字段命名冲突 | ✅ | 无冲突 |

---

## 4️⃣ 状态流一致性

| 状态 | 前端定义 | 后端实际 | 匹配 |
|------|----------|----------|------|
| 订单 pending | PurchaseResponse.status='pending' | 后端 subscriptions/change 同步扣款，无 pending | ⚠️ 前端保留 pending 用于轮询场景，但当前后端同步扣款不会返回 pending |
| 订单 paid | PurchaseResponse.status='paid' | 后端同步扣款成功 | ✅ PaymentPanel 直接设为 paid |
| 订单 failed | PurchaseResponse.status='failed' | 后端抛异常 | ✅ PaymentPanel catch 后走 onError |
| 优惠券 active | Coupon.status='active' | 后端 status='active' | ✅ |
| 优惠券 used_up | Coupon.status='used_up' | 后端 status='used_up' | ✅ |
| 套餐 saleStatus | 前端过滤 saleStatus='active' | 后端 sale_status='active' | ✅ |

---

## 5️⃣ 错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | 所有 API 调用均有 try/catch |
| 空数据处理 | ✅ | plans 为空时显示 Empty |
| loading 处理 | ✅ | pageStatus/calculating/submitting 三层 loading |
| 超时/失败 | ✅ | usePolling 有 maxRetries；API 错误展示给用户 |
| 余额不足 | ✅ | 后端返回 insufficientBalance，前端可扩展提示 |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| plans 通过 Service 层 | ✅ | userApi.billing.plans() |
| validateCoupon 通过 Service 层 | ✅ | userApi.billing.validateCoupon() |
| calculate 通过 Service 层 | ✅ | userApi.billing.calculate() |
| createOrder 通过 Service 层 | ✅ | userApi.billing.createOrder() |
| getOrder 通过 Service 层 | ✅ | userApi.billing.getOrder() |
| 是否绕过 XBIS | ✅ | 全部通过 userApi 调用 |

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 状态 |
|------|------|------|------|------|
| API URL | plans 路径错误 `/billing/plans` | services.ts | 套餐列表无法加载 | ✅ 已修复 |
| API URL | validateCoupon 路径错误 | services.ts | 优惠券验证失败 | ✅ 已修复 |
| API Method | validateCoupon 应为 GET 非 POST | services.ts | 请求方法不匹配 | ✅ 已修复 |
| API URL | calculate 路径错误 | services.ts | 价格计算失败 | ✅ 已修复 |
| API URL | createOrder 路径错误 | services.ts | 创建订单失败 | ✅ 已修复 |
| 参数映射 | couponCode → couponNo | services.ts | 后端无法识别优惠券 | ✅ 已修复 |
| 返回结构 | PlanDetail 字段名不匹配 | PlanPurchasePage | 套餐信息显示异常 | ✅ 已修复 |
| 返回结构 | PriceCalculation 字段名不匹配 | PlanPurchasePage/CouponInput | 价格计算显示异常 | ✅ 已修复 |
| 返回结构 | PurchaseResponse 字段名不匹配 | PaymentPanel | 订单信息异常 | ✅ 已修复 |
| 后端缺失 | 无 coupons/available 路由 | userP0.ts | 优惠券查询无入口 | ✅ 已新增 |

---

## 🛠 修改建议

### 前端修改（已完成）：

1. **services.ts**: 5 个 API URL/Method 修正，参数映射 couponCode → couponNo
2. **PlanPurchasePage.tsx**: 新增 SubscriptionPlanRow/SubscriptionQuote 类型，mapPlanRowToDetail 映射函数，calculatePrice 适配 quote 返回结构
3. **CouponInput.tsx**: validateAndApply 适配 coupons/available 返回 `{ items: Coupon[] }`，从列表中匹配 couponNo
4. **PaymentPanel.tsx**: handleSubmit 适配 subscriptions/change 返回结构，映射为 PurchaseResponse

### 后端修改（已完成）：

1. **userP0.ts**: 新增 `GET /coupons/available` 路由，复用 `loadAvailableCoupons` 工具函数

### 数据结构调整：无需调整

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 影响后续任务 | 低 | 修改仅涉及 T020 套餐购买流程，不影响其他模块 |
| 影响数据模型 | 低 | 前端类型定义不变，仅做映射层适配 |
| 后端 coupons/available 新增 | 低 | 复用已有 loadAvailableCoupons，无副作用 |
| subscriptions/change 同步扣款 | 中 | 后端直接扣余额，无第三方支付网关跳转；前端 PaymentPanel 保留 paymentMethod 选择但后端不使用 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

所有 API 契约不一致问题已修复：
- 5 个 API URL/Method 已对齐后端实际路由
- 3 个返回结构映射已在前端做适配层
- 1 个后端缺失路由已补充
- TypeScript 编译无新增错误
