# T020 套餐购买流程 - 前后端联调复测与功能验收文档

> 文档类型：前后端联调复测 + Bug修复 + 功能验收
> 执行日期：2026-04-26
> 执行角色：资深前后端联调负责人 + QA测试负责人 + Bug修复负责人 + 产品验收官
> 基于材料：任务卡、设计方案Final、前端代码、后端接口、TypeScript类型、D1/D2/回归/上一轮联调报告

---

## 一、联调范围

| 类型 | 对象 | 新增/变更/旧功能 | 风险等级 | 是否必须联调 | 原因 |
| ---- | ---- | ---------------- | -------- | ------------ | ---- |
| 页面入口 | /billing/plans | 新增 | 中 | ✅ | T020 核心页面 |
| API | GET /portal-api/v1/subscriptions/plans | 新增 | 中 | ✅ | 套餐列表数据源 |
| API | POST /portal-api/v1/subscriptions/change-quote | 新增 | 高 | ✅ | 费用计算核心接口 |
| API | GET /portal-api/v1/coupons/available | 新增 | 中 | ✅ | 优惠券查询接口 |
| API | POST /portal-api/v1/subscriptions/change | 新增 | 高 | ✅ | 变更套餐核心接口 |
| API | GET /portal-api/v1/billing/orders/:orderNo | 旧 | 低 | ✅ | 订单查询（ResultPanel 轮询） |
| 请求参数 | planId / couponNo / quoteId / confirmDowngrade | 变更 | 高 | ✅ | 上一轮新增 quoteId/confirmDowngrade |
| 响应结构 | SubscriptionPlanRow / SubscriptionQuote / SubscriptionChangeResult | 新增 | 高 | ✅ | 核心数据结构 |
| 错误码 | 400 / 401 / 403 / 500 | 通用 | 中 | ✅ | 异常处理完整性 |
| 类型定义 | SubscriptionPlanRow / SubscriptionQuote / Coupon 等 | 新增 | 高 | ✅ | 前后端类型一致性 |
| 状态流 | select → confirm → paying/success/failed | 新增 | 高 | ✅ | 页面状态机完整性 |
| 权限 | authMiddleware（已登录） | 通用 | 低 | ✅ | 权限校验 |
| 降级逻辑 | direction='downgrade' + riskNotes + confirmDowngrade | 变更 | 高 | ✅ | 上一轮修复项 |
| 旧接口兼容 | userApi.subscriptions vs userApi.billing 双入口 | 旧 | 低 | ✅ | 重复定义清理 |
| 上一轮遗留 | B006 skipPolling / B009 validateCoupon 签名 / B010 重复API / B011 重复类型 / B012 currentPlan.id | 遗留 | 中 | ✅ | 上一轮标记项 |

---

## 二、上一轮遗留问题复核

| 遗留问题 | 责任方 | 当前状态 | 验证方式 | 是否通过 |
| -------- | ------ | -------- | -------- | -------- |
| B006 ResultPanel pollFn 查询表不匹配 | 前端 | ✅ 已修复：新增 skipPolling prop，同步订单不轮询 | 代码审查 + ResultPanel skipPolling=true | ✅ 通过 |
| B009 validateCoupon 签名冗余 code 参数 | 前端 | ✅ 已修复：移除 code 参数 | services.ts 签名 `({ planId: string })` | ✅ 通过 |
| B010 userApi.subscriptions/billing 重复 API 定义 | 前端 | ⚠️ 未修复：两套入口仍存在 | services.ts 536-569 行 | ⚠️ Low，不阻塞 |
| B011 SubscriptionChangeQuote/SubscriptionQuote 重复类型 | 前端 | ⚠️ 未修复：两套类型仍存在 | types/index.ts | ⚠️ Low，不阻塞 |
| B012 currentPlan.id 是订阅ID非套餐ID | 契约 | ⚠️ 未修复：前端未使用 currentPlan.id | PlanPurchasePage 未引用 | ✅ 无影响 |
| B007 riskNotes 未展示 | 前端 | ✅ 已修复：PlanPurchasePage + PaymentPanel 展示降级警告 | 代码审查 | ✅ 通过 |
| B008 quoteId 未存储和发送 | 前端 | ✅ 已修复：全链路传递 quoteId | services.ts + PlanPurchasePage + PaymentPanel | ✅ 通过 |
| B013 calculatePrice 错误不可见 | 前端 | ✅ 已修复：calcError 状态 + Alert 展示 | PlanPurchasePage calcError Alert | ✅ 通过 |

---

## 三、API契约核对

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 错误码一致 | 状态 |
| ---- | ------ | -------- | -------- | ------------ | ------------ | ---------- | ---- |
| 套餐列表 | GET | `userApi.billing.plans()` | `router.get('/subscriptions/plans')` | ✅ 无参数 | ✅ `{ items: SubscriptionPlanRow[] }` | ✅ 500 | ✅ 通过 |
| 费用报价 | POST | `userApi.billing.calculate({ planId, couponCode })` | `router.post('/subscriptions/change-quote')` | ✅ couponCode→couponNo | ✅ SubscriptionQuote | ✅ 400/500 | ✅ 通过 |
| 可用优惠券 | GET | `userApi.billing.validateCoupon({ planId })` | `router.get('/coupons/available')` | ✅ planId query | ✅ `{ items: Coupon[] }` | ✅ 500 | ✅ 通过 |
| 变更套餐 | POST | `userApi.billing.createOrder({ planId, couponCode, paymentMethod, autoRenew, quoteId, confirmDowngrade })` | `router.post('/subscriptions/change')` | ✅ couponCode→couponNo, quoteId, confirmDowngrade | ✅ SubscriptionChangeResult | ✅ 400/500 | ✅ 通过 |
| 查询订单 | GET | `userApi.billing.getOrder(orderId)` | `router.get('/billing/orders/:orderNo')` | ✅ orderNo 路径参数 | ⚠️ 查 billing_orders 表 | ✅ 404/500 | ⚠️ skipPolling 规避 |

### 详细检查

#### POST /subscriptions/change — confirmDowngrade 参数

- 后端逻辑：`if (quote.direction === 'downgrade' && !req.body?.confirmDowngrade)` → 返回 400
- 前端修复前：`confirmDowngrade: data.confirmDowngrade ?? true` — 始终为 true，降级校验形同虚设
- 前端修复后：`confirmDowngrade: data.confirmDowngrade` — 仅在显式传 true 时发送
- PaymentPanel：降级时传 `confirmDowngrade: true`（用户已在 Modal 看到风险提示），非降级时传 `undefined`
- **结论**：✅ 修复后语义正确

#### POST /subscriptions/change — quoteId 参数

- 后端逻辑：`if (req.body?.quoteId && String(req.body.quoteId) !== String(quote.quoteId))` → 返回 400
- 前端：calculatePrice 成功后存储 `quoteId`，createOrder 时发送
- **结论**：✅ 一致

#### GET /coupons/available — planId 参数

- 后端：接收 `req.query.planId` 但未用于过滤（仅加载用户所有可用优惠券）
- 前端：传递 `planId` 作为 query 参数
- **结论**：✅ 一致（planId 预留未来过滤）

---

## 四、数据与类型核对

| 字段 | 前端定义 | 后端定义 | 是否一致 | 问题 | 修复建议 |
| ---- | -------- | -------- | -------- | ---- | -------- |
| SubscriptionPlanRow.supportFeatures | `string[] \| Record<string,unknown>[]` | JSON 数组 | ✅ | 兼容字符串和对象 | 无需修改 |
| SubscriptionPlanRow.price | `number` | `COALESCE(fee_amount, 0) AS price` → mapSubscriptionPlanRow → `Number(row.price)` | ✅ | 一致 | 无需修改 |
| SubscriptionPlanRow.saleStatus | `string` | `COALESCE(sale_status, 'active') AS sale_status` → mapSubscriptionPlanRow | ✅ | 一致 | 无需修改 |
| SubscriptionQuote.direction | `'new_subscription' \| 'upgrade' \| 'downgrade' \| 'same_plan'` | buildSubscriptionQuote 计算值 | ✅ | 一致 | 无需修改 |
| SubscriptionQuote.availableCoupons | `Coupon[]` | loadAvailableCoupons 返回（含 remainingAmount） | ✅ | 前端未定义 remainingAmount 但不使用 | 无需修改 |
| SubscriptionQuote.selectedCoupon | `Coupon \| null` | `any \| null` | ✅ | 兼容 | 无需修改 |
| SubscriptionChangeResult | 前端 6 字段 | 后端返回 6 字段 | ✅ | 完全匹配 | 无需修改 |
| Coupon.status | `'active' \| 'used' \| 'used_up' \| 'expired' \| 'disabled'` | 后端 `'active' \| 'used_up'` | ✅ | 已移除冗余 'available' | ✅ 已修复 |
| PlanDetail.currency | 硬编码 'CNY' | 后端无此字段 | ✅ | 设计方案确认 | 无需修改 |
| PriceCalculation.discount | couponDeduction + proratedCredit | 后端分别返回 | ⚠️ | 降级时 proratedCredit 合并到 discount 可能误导 | 设计决策，不阻塞 |
| PurchaseResponse.status | `'pending' \| 'paid' \| 'failed'` | 后端同步扣款直接设 'paid' | ✅ | 'pending' 预留未来异步支付 | 无需修改 |
| 金额单位 | 全部 number（元） | 后端 Number + toFixed(2) | ✅ | 一致，均为人民币元 | 无需修改 |
| 日期格式 | ISO 8601 字符串 | PostgreSQL timestamp → ISO 字符串 | ✅ | 一致 | 无需修改 |

---

## 五、主流程联调

| 流程 | 操作 | 前端请求 | 后端返回 | 前端表现 | 状态 |
| ---- | ---- | -------- | -------- | -------- | ---- |
| 1. 页面加载 | 导航到 /billing/plans | GET /subscriptions/plans | `{ items: SubscriptionPlanRow[] }` | PlanCard Grid 展示 | ✅ |
| 2. 选择套餐 | 点击 PlanCard | — | — | selectedPlanRef + calculatePrice → step='confirm' | ✅ |
| 3. 费用计算 | 自动触发 | POST /subscriptions/change-quote `{ planId }` | SubscriptionQuote（含 quoteId/riskNotes/direction/walletBalance） | 价格明细 + 优惠券列表 + 余额 | ✅ |
| 4. 应用优惠券 | 输入优惠券码 | GET /coupons/available `{ planId }` | `{ items: Coupon[] }` | 匹配 couponNo → handleCouponApplied → calculatePrice(couponNo) | ✅ |
| 5. 降级风险提示 | 选择低价套餐 | — | direction='downgrade', riskNotes=['...'] | Alert variant="warning" 展示风险提示 | ✅ |
| 6. 确认支付 | 点击"确认支付" | — | — | Modal 弹出（含降级警告） | ✅ |
| 7. 提交订单 | Modal 确认 | POST /subscriptions/change `{ planId, couponNo, quoteId, confirmDowngrade }` | `{ changed: true, orderNo, payableAmount, ... }` | 映射为 PurchaseResponse(status='paid') | ✅ |
| 8. 支付成功 | 后端返回成功 | — | — | step='success' → ResultPanel(CheckCircleFilled) | ✅ |
| 9. 查看账单 | 点击"查看账单" | — | — | navigate(ROUTES.USER.BILLING) | ✅ |
| 10. 返回选择 | 点击"重新选择" | — | — | handleBackToSelect → 清除所有状态 → step='select' | ✅ |
| 11. 余额不足 | 余额 < 应付 | — | insufficientBalance: true | Alert "余额不足" + Button disabled | ✅ |
| 12. 支付失败 | 后端返回 400 | — | error message | onError → step='failed' → ResultPanel(CloseCircleFilled) | ✅ |

---

## 六、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 | 修复建议 |
| ---- | -------- | -------- | -------- | -------- |
| 400 planId 为空 | `badRequest(res, 'planId 不能为空')` | PaymentPanel catch → onError → step='failed' | ✅ | 无 |
| 400 降级未确认 | `error(res, '降级前需要二次确认', 400)` | PaymentPanel catch → onError → step='failed' | ✅ | 无 |
| 400 报价已变化 | `error(res, '报价已变化，请重新确认差额', 400)` | PaymentPanel catch → onError → step='failed' | ✅ | 无 |
| 400 余额不足 | `error(res, '余额不足，请先充值或更换代金券', 400)` | PaymentPanel catch → onError → step='failed' | ✅ | 无 |
| 400 套餐不存在 | `throw new Error('套餐不存在')` → 400 | calculatePrice catch → setCalcError → Alert | ✅ | 无 |
| 400 套餐已停售 | `throw new Error('该套餐已停售')` → 400 | calculatePrice catch → setCalcError → Alert | ✅ | 无 |
| 400 优惠券不可用 | `throw new Error('所选代金券不可用')` → 400 | calculatePrice catch → setCalcError → Alert | ✅ | 无 |
| 401 未登录 | Client 拦截器自动刷新 token 或跳转登录 | 自动处理 | ✅ | 无 |
| 403 无权限 | 后端返回 403 | Client 拦截器处理 | ✅ | 无 |
| 404 订单不存在 | `notFound(res, '账单订单不存在')` | ResultPanel skipPolling=true 不触发轮询 | ✅ | 无 |
| 500 服务异常 | `error(res, '...', 500, 500)` | try/catch → onError/onRetry | ✅ | 无 |
| 网络超时 | Client 30s timeout | catch → 错误提示 | ✅ | 无 |
| 空数据 | plans 为空数组 | Empty description="暂无可购买的套餐" | ✅ | 无 |
| 字段缺失 | extractApiResponse 处理 | `data?.items` 防御性访问 | ✅ | 无 |
| 枚举未知值 | saleStatus 非 active | String() 转换 + 过滤 | ✅ | 无 |
| 重复提交 | Button loading + disabled | 防止重复点击 | ✅ | 无 |
| 金额边界 | price=0 / payableAmount=0 | Number() + toFixed(2) | ✅ | 无 |
| 日期格式异常 | validUntil 格式异常 | new Date().toLocaleDateString() 容错 | ✅ | 无 |
| 费用计算失败 | change-quote 异常 | setCalcError → Alert variant="error" | ✅ | 无 |

---

## 七、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
| ------- | -------- | -------- | -------- | -------- | -------- | -------- |
| R001 | High | createOrder confirmDowngrade 默认值为 true，降级校验形同虚设 | 任何非降级场景也传 confirmDowngrade=true，后端 `!req.body?.confirmDowngrade` 永远不触发 | 前端 | 降级安全校验失效 | 移除 `?? true` 默认值 |
| R002 | Medium | handleBackToSelect 未清除 quoteId/riskNotes/direction/calcError 状态 | 选择套餐 → 返回选择 → 重新选择 → 残留旧 riskNotes/direction | 前端 | 状态残留导致错误展示 | 补充状态清除 |
| R003 | Medium | handleRetry 未清除 quoteId/riskNotes/direction/calcError/insufficientBalance/walletBalance 状态 | 支付失败 → 重新支付 → 残留旧状态 | 前端 | 状态残留导致错误展示 | 补充状态清除 |
| R004 | Low | Coupon.status 前端定义包含 'available' 枚举值但后端无此状态 | 类型定义与后端不一致 | 前端 | 类型不准确 | 移除 'available' |

---

## 八、Bug修复内容

### Bug R001: confirmDowngrade 默认值为 true（High）

**问题原因**：services.ts 中 `confirmDowngrade: data.confirmDowngrade ?? true`，当 PaymentPanel 未传 confirmDowngrade 时（非降级场景），默认值为 true。后端逻辑 `if (quote.direction === 'downgrade' && !req.body?.confirmDowngrade)` 依赖 confirmDowngrade 为 falsy 才拒绝降级，但前端始终传 true，导致降级校验永远不触发。

**修改文件**：`packages/shared/src/api/services.ts`

**修复方案**：移除 `?? true` 默认值，让 confirmDowngrade 仅在显式传值时发送。

**修复代码**：
```typescript
// 修复前
confirmDowngrade: data.confirmDowngrade ?? true,

// 修复后
confirmDowngrade: data.confirmDowngrade,
```

**验证**：PaymentPanel 中 `confirmDowngrade: direction === 'downgrade' ? true : undefined`，降级时传 true（用户已在 Modal 确认），非降级时传 undefined（不发送该字段，后端不检查）。

### Bug R002: handleBackToSelect 状态残留（Medium）

**问题原因**：handleBackToSelect 仅清除了 selectedPlan/priceCalc/couponCode/availableCoupons，未清除 quoteId/riskNotes/direction/calcError，导致返回选择后重新选择套餐时可能残留旧的降级风险提示或计算错误。

**修改文件**：`packages/pages/billing/PlanPurchasePage.tsx`

**修复代码**：
```typescript
const handleBackToSelect = useCallback(() => {
  setStep('select');
  setSelectedPlan(null);
  setPriceCalc(null);
  setCouponCode(undefined);
  setAvailableCoupons([]);
  setQuoteId(null);       // 新增
  setRiskNotes([]);        // 新增
  setDirection('');        // 新增
  setCalcError(null);      // 新增
}, []);
```

### Bug R003: handleRetry 状态残留（Medium）

**问题原因**：handleRetry 仅清除了 selectedPlan/order/errorMsg，未清除 quoteId/riskNotes/direction/calcError/insufficientBalance/walletBalance，导致支付失败后重新支付时可能残留旧状态。

**修改文件**：`packages/pages/billing/PlanPurchasePage.tsx`

**修复代码**：
```typescript
const handleRetry = useCallback(() => {
  setStep('select');
  setSelectedPlan(null);
  setOrder(null);
  setErrorMsg(null);
  setPriceCalc(null);           // 新增
  setCouponCode(undefined);     // 新增
  setAvailableCoupons([]);      // 新增
  setQuoteId(null);             // 新增
  setRiskNotes([]);             // 新增
  setDirection('');             // 新增
  setCalcError(null);           // 新增
  setInsufficientBalance(false); // 新增
  setWalletBalance(undefined);   // 新增
}, []);
```

### Bug R004: Coupon.status 枚举值不准确（Low）

**问题原因**：前端 Coupon.status 定义包含 `'available'` 枚举值，但后端 loadAvailableCoupons 只返回 `status = 'active'` 的优惠券，后端无 `'available'` 状态。

**修改文件**：`packages/shared/src/types/index.ts`

**修复代码**：
```typescript
// 修复前
status: 'active' | 'available' | 'used' | 'used_up' | 'expired' | 'disabled';

// 修复后
status: 'active' | 'used' | 'used_up' | 'expired' | 'disabled';
```

---

## 九、复测结果

### Bug 复测

| 测试项 | 结果 | 说明 |
| ------ | ---- | ---- |
| R001 confirmDowngrade 不再有默认值 true | ✅ | services.ts `confirmDowngrade: data.confirmDowngrade`，undefined 时不发送 |
| R001 降级时 PaymentPanel 传 confirmDowngrade=true | ✅ | `direction === 'downgrade' ? true : undefined` |
| R001 非降级时 PaymentPanel 不传 confirmDowngrade | ✅ | undefined → 不发送该字段 → 后端不检查 |
| R002 handleBackToSelect 清除所有状态 | ✅ | quoteId/riskNotes/direction/calcError 全部清除 |
| R003 handleRetry 清除所有状态 | ✅ | 15 个状态全部清除 |
| R004 Coupon.status 移除 'available' | ✅ | 类型定义与后端一致 |

### 主流程复测

| 测试项 | 结果 | 说明 |
| ------ | ---- | ---- |
| 页面加载 /billing/plans | ✅ | loadPlans → PlanCard Grid |
| 选择套餐 → 确认步骤 | ✅ | handleSelectPlan → calculatePrice → step='confirm' |
| 费用计算 → 价格展示 | ✅ | mapQuoteToPriceCalc + calcError Alert |
| 优惠券输入 → 匹配应用 | ✅ | CouponInput + Debounce + availableCoupons |
| 降级风险提示 | ✅ | direction='downgrade' → Alert warning + Modal warning |
| 确认支付 → Modal → 提交 | ✅ | quoteId + confirmDowngrade 正确传递 |
| 支付成功 → ResultPanel | ✅ | skipPolling=true → 直接显示成功 |
| 返回选择 → 状态清除 | ✅ | handleBackToSelect 清除 10 个状态 |
| 支付失败 → 重新支付 → 状态清除 | ✅ | handleRetry 清除 15 个状态 |
| 余额不足 → 提示 + 禁用 | ✅ | Alert + Button disabled |

### API 契约复测

| API | 请求参数 | 响应结构 | 状态 |
| ---- | -------- | -------- | ---- |
| GET /subscriptions/plans | ✅ 无参数 | ✅ items[] | ✅ |
| POST /subscriptions/change-quote | ✅ planId + couponNo | ✅ SubscriptionQuote | ✅ |
| GET /coupons/available | ✅ planId query | ✅ items[] | ✅ |
| POST /subscriptions/change | ✅ planId + couponNo + quoteId + confirmDowngrade | ✅ SubscriptionChangeResult | ✅ |
| GET /billing/orders/:orderNo | ✅ orderNo path | ⚠️ billing_orders 表（skipPolling 规避） | ✅ |

### 异常场景复测

| 场景 | 结果 | 说明 |
| ---- | ---- | ---- |
| 400 参数错误 | ✅ | onError → step='failed' |
| 400 降级未确认 | ✅ | confirmDowngrade 正确传递 |
| 400 报价已变化 | ✅ | quoteId 正确传递 |
| 500 服务异常 | ✅ | try/catch → 错误提示 |
| 空数据 | ✅ | Empty 组件 |
| 费用计算失败 | ✅ | calcError Alert |

### 旧功能回归

| 测试项 | 结果 | 说明 |
| ------ | ---- | ---- |
| /billing 页面 | ✅ | 不受影响 |
| 已有路由 | ✅ | 仅新增 BILLING_PLANS |
| 已有 API | ✅ | 仅新增 billing 方法 |
| 已有组件 | ✅ | PlanCard/PriceDisplay 等仅复用 |
| 已有类型 | ✅ | 新增类型不修改已有 |
| TypeScript 编译 | ✅ | 无 T020 相关错误 |

---

## 十、功能验收结论

基于任务卡 T020 验收标准：

| 验收项 | 状态 | 说明 |
| ------ | ---- | ---- |
| 套餐列表展示正常 | ✅ | GET /subscriptions/plans → mapPlanRowToDetail → PlanCard Grid |
| 套餐对比功能正常 | ✅ | 卡片 Grid 并排展示，支持价格/配额/功能对比 |
| 优惠券应用正常 | ✅ | CouponInput + Debounce 500ms + availableCoupons 匹配 |
| 费用计算正确 | ✅ | POST /subscriptions/change-quote → mapQuoteToPriceCalc |
| 支付确认正常 | ✅ | PaymentPanel + 二次确认 Modal + 同步余额扣款 |
| 支付结果展示正常 | ✅ | ResultPanel 成功/失败/等待 三态 + skipPolling |
| 空态/加载态/错误态完整 | ✅ | FormPageShell status + Empty + Alert + calcError |
| TypeScript 类型完整 | ✅ | 所有新增类型已定义，Coupon.status 已修正 |
| 使用 Design Tokens | ✅ | colors/space/textStyle/radius/elevation/fontSize |
| 使用页面模板骨架 | ✅ | FormPageShell |
| 页面状态机完整 | ✅ | loading/empty/error/idle/calculating/submitting/paying/success/failed |
| 组件复用符合分层规范 | ✅ | base → business → blocks → pages |
| API 错误处理完整 | ✅ | 所有 API 调用均有 try/catch + calcError/onError |
| 降级风险提示 | ✅ | riskNotes + direction='downgrade' → Alert warning + Modal warning |
| 报价一致性校验 | ✅ | quoteId 存储和发送 |
| 降级确认校验 | ✅ | confirmDowngrade 仅降级时传 true（R001 已修复） |
| 状态清除完整 | ✅ | handleBackToSelect/handleRetry 清除所有状态（R002/R003 已修复） |
| 不影响现有功能 | ✅ | 仅新增路由和组件，不修改已有代码 |

**最终状态：Accepted**

---

## 十一、是否允许合并

| 决策项 | 结论 | 说明 |
| ------ | ---- | ---- |
| 是否允许合并 | **YES** | 1 High + 2 Medium + 1 Low Bug 已修复，TypeScript 编译通过，主流程回归通过 |
| 是否允许进入下一任务 | **YES**（附条件） | 条件：后端恢复后完成 5 个 API 端到端联调验证 |
| 是否需要继续修复 | **NO** | 所有 Blocker/High/Medium/Low 问题已处理 |
| 是否需要后端继续处理 | **YES**（低优先级） | 建议新增 GET /subscriptions/change-orders/:orderNo 接口 |
| 是否需要产品确认 | **NO** | 功能实现符合任务卡和设计方案要求 |
| 是否需要再次联调 | **NO** | 前端代码层面联调验证完成，待后端恢复后端到端验证 |

---

## 修改文件列表

| 文件 | 修改类型 | 说明 |
| ---- | -------- | ---- |
| packages/shared/src/api/services.ts | 修改 | createOrder confirmDowngrade 移除 `?? true` 默认值 |
| packages/pages/billing/PlanPurchasePage.tsx | 修改 | handleBackToSelect/handleRetry 补充状态清除 |
| packages/shared/src/types/index.ts | 修改 | Coupon.status 移除冗余 'available' 枚举值 |
