# T014 能力订阅流程 — 最终可开发设计方案

> 任务卡: [T014-ability-subscribe-flow.md](../T014-ability-subscribe-flow.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
SubscribeModal
├── AbilityInfo
│   └── 能力基本信息
├── PlanSelection
│   └── 套餐选择
├── PaymentConfirm
│   └── 支付确认
└── SubscribeSuccess
    └── 订阅成功
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
| ApiKeyDisplay | 密钥展示 | 是 |
| PriceDisplay | 价格展示 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| SubscribeModal | 订阅弹窗 | 是 |
| PlanSelection | 套餐选择 | 是 |
| PaymentConfirm | 支付确认 | 是 |
| KeyDisplay | 密钥展示区 | 是 |
| DocPreview | 文档预览区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[点击订阅]
    │
    ▼
[打开 SubscribeModal]
    │
    ▼
[选择套餐]
    │
    ▼
[确认支付]
    │
    ▼
[API: POST /api/v1/abilities/:id/subscribe]
    │
    ▼
[订阅成功]
```

---

## 4. 状态管理

```typescript
interface SubscribeState {
  step: 'select' | 'confirm' | 'success';
  selectedPlan: Plan | null;
  loading: boolean;
  error: ApiError | null;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/abilities/:id/plans | 页面加载 | abilityId | 显示错误提示 |
| POST /api/v1/billing/coupon/validate | 应用优惠券 | code, planId | 提示优惠券无效 |
| POST /api/v1/abilities/:id/subscribe | 确认接入 | planId, couponCode | 展示错误信息 |
| POST /api/v1/subscriptions/:id/regenerate-key | 重新生成密钥 | subscriptionId | 提示错误 |

---

## 6. 用户交互流程

### 主流程

```
用户点击订阅
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示套餐选择
  │   ├── 用户选择套餐
  │   ├── 用户应用优惠券（可选）
  │   ├── 用户确认费用
  │   └── 用户确认接入
  │       └── 展示密钥（仅一次）
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 已接入 | 用户已接入该能力 | 提示已接入 | 展示已有密钥 |
| 套餐不可用 | 套餐已下架 | 标记不可用 | 提示选择其他套餐 |
| 配额不足 | 免费配额已用完 | 提示升级 | 引导选择付费套餐 |
| 已订阅 | 用户已订阅 | 提示已订阅 | 展示已有密钥 |
| 支付失败 | 支付异常 | 提示错误，保留选择 | 提示重试 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 密钥安全 | 仅展示一次，支持重新生成 |
| 套餐对比 | 支持多套餐对比 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 费用计算 < 300ms
- 无内存泄漏
- 套餐数据预加载
- 支付结果轮询

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 密钥泄露 | 低 | 密钥可能被截图或泄露 | 仅展示一次，支持重新生成 |
| 套餐变更 | 中 | 套餐可能在接入过程中变更 | 接入时锁定套餐版本 |
| 并发接入 | 低 | 并发接入可能导致冲突 | 幂等处理 |
| 支付安全 | 高 | 支付信息泄露 | 使用支付网关 |

---

## 10. 开发步骤拆分

### Step 1: 订阅弹窗（1 天）
- [ ] SubscribeModal + PlanSelection

### Step 2: 支付流程（1 天）
- [ ] PaymentConfirm + 支付集成

### Step 3: 密钥展示开发（0.5 人日）
- [ ] KeyDisplay 组件
- [ ] 重新生成密钥

### Step 4: 接入文档开发（0.5 人日）
- [ ] DocPreview 组件

### Step 5: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现步骤状态管理

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 自测
