# T020 套餐购买流程 — 工程级设计方案

> 任务卡: [T020-plan-purchase-flow.md](../T020-plan-purchase-flow.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 面包屑导航
│   └── 标题: "选择套餐"
├── FormPageShell
│   ├── PlanComparison (套餐对比区)
│   │   └── PlanCard[]
│   ├── CouponInput (优惠券输入)
│   ├── PriceCalculation (费用计算)
│   └── PaymentPanel (支付面板)
└── (空态/加载态/错误态)
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
[页面加载]
  └── 调用 GET /api/v1/billing/plans ──► 获取套餐列表

[用户选择套餐]
  └── 更新 selectedPlan

[用户应用优惠券]
  ├── 调用 POST /api/v1/billing/coupon/validate
  └── 更新 discount

[用户确认支付]
  ├── 调用 POST /api/v1/billing/orders
  └── 返回支付链接，跳转支付

[支付完成]
  └── 返回支付结果，展示成功页面
```

---

## 4. 状态管理

### 页面状态

```typescript
interface PlanPurchaseState {
  // 数据状态
  plans: PlanDetail[];
  selectedPlan?: PlanDetail;
  coupon?: Coupon;
  order?: PurchaseResponse;

  // 表单状态
  couponCode: string;
  paymentMethod: string;
  autoRenew: boolean;

  // UI 状态
  status: 'idle' | 'loading' | 'success' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/billing/plans | 页面加载 | - | 显示错误提示 |
| POST /api/v1/billing/coupon/validate | 应用优惠券 | code, planId | 提示优惠券无效 |
| POST /api/v1/billing/calculate | 费用计算 | planId, couponCode | 提示计算错误 |
| POST /api/v1/billing/orders | 创建订单 | PurchaseRequest | 提示创建失败 |

---

## 6. 用户交互流程

### 主流程

```
用户点击升级
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示套餐列表
  │   ├── 用户选择套餐
  │   ├── 用户应用优惠券（可选）
  │   ├── 用户确认费用
  │   └── 用户确认支付
  │       └── 跳转支付网关
  ├── 支付完成 ──► 展示成功页面
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 优惠券无效 | 过期或不满足条件 | 返回错误 | 提示原因 |
| 支付失败 | 支付网关错误 | 返回错误 | 提示重试 |
| 套餐下架 | 套餐已不可用 | 标记不可用 | 提示选择其他套餐 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 多币种 | 支持货币切换 |
| 优惠券叠加 | 不支持叠加 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 费用计算 < 300ms
- 无内存泄漏

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 支付安全 | 低 | 支付过程需保证安全 | 使用支付网关 SDK |
| 优惠券滥用 | 中 | 可能重复使用优惠券 | 后端校验 |
| 并发下单 | 低 | 并发可能导致重复订单 | 幂等处理 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 FormPageShell 搭建页面框架

### Step 2: 套餐对比开发（1 人日）
- [ ] PlanComparison 组件
- [ ] 套餐对比功能

### Step 3: 优惠券开发（0.5 人日）
- [ ] CouponInput 组件
- [ ] 优惠券验证

### Step 4: 支付面板开发（0.5 人日）
- [ ] PaymentPanel 组件
- [ ] 费用计算展示

### Step 5: 结果面板开发（0.5 人日）
- [ ] ResultPanel 组件
- [ ] 支付成功/失败展示

### Step 6: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现支付流程

### Step 7: 性能优化 & 测试（0.5 人日）
- [ ] 自测
