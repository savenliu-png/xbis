# T014 能力接入流程 — 工程级设计方案

> 任务卡: [T014-ability-subscribe-flow.md](../T014-ability-subscribe-flow.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 面包屑导航
│   └── 标题: "能力接入"
├── FormPageShell
│   ├── Steps (步骤条)
│   │   ├── Step 1: 选择套餐
│   │   ├── Step 2: 确认费用
│   │   └── Step 3: 获取密钥
│   ├── StepContent
│   │   ├── PlanSelector (套餐选择)
│   │   ├── ConfirmPanel (费用确认)
│   │   └── KeyDisplay (密钥展示)
│   └── DocPreview (接入文档)
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
| ApiKeyDisplay | 密钥展示 | 是 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| PlanSelector | 套餐选择区 | 是 |
| ConfirmPanel | 确认面板 | 是 |
| KeyDisplay | 密钥展示区 | 是 |
| DocPreview | 文档预览区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[页面加载]
  └── 调用 GET /api/v1/abilities/:id/plans ──► 获取套餐列表

[用户选择套餐]
  └── 更新 selectedPlan

[用户应用优惠券]
  ├── 调用 POST /api/v1/billing/coupon/validate
  └── 更新 discount

[用户确认接入]
  ├── 调用 POST /api/v1/abilities/:id/subscribe
  └── 返回 subscription + apiKey

[展示密钥]
  └── 仅展示一次，支持重新生成
```

---

## 4. 状态管理

### 页面状态

```typescript
interface AbilitySubscribeState {
  // 数据状态
  plans: PlanOption[];
  selectedPlan?: PlanOption;
  coupon?: Coupon;
  subscription?: AbilitySubscribeResponse;

  // 表单状态
  couponCode: string;
  autoRenew: boolean;

  // 步骤状态
  currentStep: number;

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
| GET /api/v1/abilities/:id/plans | 页面加载 | abilityId | 显示错误提示 |
| POST /api/v1/billing/coupon/validate | 应用优惠券 | code, planId | 提示优惠券无效 |
| POST /api/v1/abilities/:id/subscribe | 确认接入 | planId, couponCode | 展示错误信息 |
| POST /api/v1/subscriptions/:id/regenerate-key | 重新生成密钥 | subscriptionId | 提示错误 |

---

## 6. 用户交互流程

### 主流程

```
用户点击接入
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

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 密钥泄露 | 低 | 密钥可能被截图或泄露 | 仅展示一次，支持重新生成 |
| 套餐变更 | 中 | 套餐可能在接入过程中变更 | 接入时锁定套餐版本 |
| 并发接入 | 低 | 并发接入可能导致冲突 | 幂等处理 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 FormPageShell 搭建页面框架
- [ ] 配置 Steps 步骤条

### Step 2: 套餐选择开发（1 人日）
- [ ] PlanSelector 组件
- [ ] 套餐对比功能

### Step 3: 费用确认开发（0.5 人日）
- [ ] ConfirmPanel 组件
- [ ] 优惠券应用

### Step 4: 密钥展示开发（0.5 人日）
- [ ] KeyDisplay 组件
- [ ] 重新生成密钥

### Step 5: 接入文档开发（0.5 人日）
- [ ] DocPreview 组件

### Step 6: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现步骤状态管理

### Step 7: 性能优化 & 测试（0.5 人日）
- [ ] 自测
