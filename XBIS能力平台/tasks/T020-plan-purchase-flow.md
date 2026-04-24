# 套餐购买流程

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 套餐购买流程 |
| 任务编号 | T020 |
| 所属模块 ⭐ | M6 商业化 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-18 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏套餐购买流程，用户需要：
- 浏览和对比套餐
- 应用优惠券
- 完成支付

### 2.2 目标用户
- 终端用户：购买套餐
- 管理员：配置套餐

### 2.3 预期效果
- 提供清晰的套餐购买流程
- 支持套餐对比
- 支持优惠券应用

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 套餐购买 | `/billing/plans` | 计费概览页「升级」按钮 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/billing/PlanPurchaseFlow`

## 4. 功能范围

### 4.1 包含功能
- [ ] 套餐列表展示
- [ ] 套餐对比
- [ ] 优惠券应用
- [ ] 费用计算
- [ ] 支付确认
- [ ] 支付结果
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 支付网关集成（由后端实现）
- 退款流程（管理端功能）
- 发票申请（由 T021 实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 套餐详情
export interface PlanDetail {
  id: string;
  name: string;
  description: string;
  price: number;
  originalPrice?: number;
  currency: string;
  quota: number;
  overagePrice?: number;
  features: PlanFeature[];
  isRecommended: boolean;
  isPopular: boolean;
}

// 套餐功能
export interface PlanFeature {
  name: string;
  description?: string;
  included: boolean;
  highlight?: string;
}

// 优惠券
export interface Coupon {
  code: string;
  type: 'percentage' | 'fixed';
  value: number;
  minOrderAmount?: number;
  maxDiscount?: number;
  validUntil: string;
}

// 购买请求
export interface PurchaseRequest {
  planId: string;
  couponCode?: string;
  paymentMethod: string;
  autoRenew: boolean;
}

// 购买响应
export interface PurchaseResponse {
  orderId: string;
  planId: string;
  amount: number;
  discount: number;
  finalAmount: number;
  status: 'pending' | 'paid' | 'failed';
  paymentUrl?: string;
  createdAt: string;
}

// 费用计算
export interface PriceCalculation {
  originalPrice: number;
  discount: number;
  finalPrice: number;
  currency: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`PlanDetail`, `PlanFeature`, `Coupon`, `PurchaseRequest`, `PurchaseResponse`, `PriceCalculation`

## 6. 交互流程

### 6.1 主流程
```
[用户点击升级] ──► [浏览套餐] ──► [选择套餐] ──► [应用优惠券] ──► [确认支付] ──► [支付结果]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 优惠券无效 | 过期或不满足条件 | 返回错误 | 提示原因 |
| 支付失败 | 支付网关错误 | 返回错误 | 提示重试 |
| 套餐下架 | 套餐已不可用 | 标记不可用 | 提示选择其他套餐 |

### 6.3 边界情况
- 多币种：支持货币切换
- 优惠券叠加：不支持叠加
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 套餐卡片 | 是（T002） |
| Radio | 单选 | 是（T002） |
| Input | 优惠券输入 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Steps | 步骤条 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanCard | 套餐卡片 | 是（T014） |
| PriceDisplay | 价格展示 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanComparison | 套餐对比区 | 否 |
| CouponInput | 优惠券输入区 | 否 |
| PaymentPanel | 支付面板 | 否 |
| ResultPanel | 结果面板 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| PriceDisplay | business | 价格展示组件 |
| PlanComparison | blocks | 套餐对比区 |
| CouponInput | blocks | 优惠券输入区 |
| PaymentPanel | blocks | 支付面板 |
| ResultPanel | blocks | 结果面板 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 套餐列表查询
- **Method**: GET
- **Path**: `/api/v1/billing/plans`
- **响应类型**: `PlanDetail[]`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": "free",
      "name": "免费版",
      "description": "适合个人开发者试用",
      "price": 0,
      "currency": "CNY",
      "quota": 100,
      "features": [
        { "name": "基础调用", "included": true },
        { "name": "社区支持", "included": true },
        { "name": "优先支持", "included": false }
      ],
      "isRecommended": false,
      "isPopular": false
    },
    {
      "id": "pro",
      "name": "专业版",
      "description": "适合小型团队",
      "price": 99,
      "originalPrice": 129,
      "currency": "CNY",
      "quota": 10000,
      "overagePrice": 0.01,
      "features": [
        { "name": "基础调用", "included": true },
        { "name": "社区支持", "included": true },
        { "name": "优先支持", "included": true, "highlight": "7x24" }
      ],
      "isRecommended": true,
      "isPopular": true
    }
  ]
}
```

#### 优惠券验证
- **Method**: POST
- **Path**: `/api/v1/billing/coupon/validate`
- **请求类型**: `{ code: string; planId: string }`
- **响应类型**: `Coupon`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "code": "SAVE20",
  "planId": "pro"
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "code": "SAVE20",
    "type": "percentage",
    "value": 20,
    "minOrderAmount": 50,
    "maxDiscount": 30,
    "validUntil": "2026-12-31T23:59:59Z"
  }
}
```

#### 费用计算
- **Method**: POST
- **Path**: `/api/v1/billing/calculate`
- **请求类型**: `{ planId: string; couponCode?: string }`
- **响应类型**: `PriceCalculation`
- **权限**: 用户（已登录）

#### 创建订单
- **Method**: POST
- **Path**: `/api/v1/billing/orders`
- **请求类型**: `PurchaseRequest`
- **响应类型**: `PurchaseResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "planId": "pro",
  "couponCode": "SAVE20",
  "paymentMethod": "alipay",
  "autoRenew": true
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "orderId": "order-001",
    "planId": "pro",
    "amount": 99,
    "discount": 19.8,
    "finalAmount": 79.2,
    "status": "pending",
    "paymentUrl": "https://payment.example.com/pay/order-001",
    "createdAt": "2026-04-24T10:30:00Z"
  }
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.billing.plans`, `userApi.billing.validateCoupon`, `userApi.billing.calculate`, `userApi.billing.createOrder`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 套餐列表展示正常
- [ ] 套餐对比功能正常
- [ ] 优惠券应用正常
- [ ] 费用计算正确
- [ ] 支付确认正常
- [ ] 支付结果展示正常
- [ ] 空态/加载态/错误态完整

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用页面模板骨架
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 首屏加载 < 2s
- [ ] 费用计算 < 300ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T019` — 计费概览页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 支付安全 | 低 | 高 | 使用支付网关 SDK |
| 优惠券滥用 | 中 | 中 | 后端校验 |
| 并发下单 | 低 | 中 | 幂等处理 |

## 11. 开发检查清单

### 11.1 开发前
- [ ] 需求已评审并确认
- [ ] 设计稿已确认
- [ ] 数据结构和 API 已评审
- [ ] 依赖任务已完成

### 11.2 开发中
- [ ] 遵循 `docs/dev-rules.md` 规范
- [ ] 遵循 `AI_RULES.md` 规范
- [ ] 自测覆盖所有验收标准

### 11.3 开发后
- [ ] 代码已提交 PR
- [ ] PR 已通过 Code Review
- [ ] 已联调通过
- [ ] 已更新文档（如需要）

## 12. 备注

- 支付流程跳转外部支付网关
- 优惠券不支持叠加使用
- 支持自动续费选项

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
