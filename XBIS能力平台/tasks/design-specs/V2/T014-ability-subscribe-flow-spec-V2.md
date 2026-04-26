# T014 能力订阅流程 — 修正版设计方案（V2）

> 原设计方案: [T014-ability-subscribe-flow-spec.md](../T014-ability-subscribe-flow-spec.md)
> 评审报告: [T014-ability-subscribe-flow-Reviewer.md](../T014-ability-subscribe-flow-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed

---

## 二、必须修改项（Blocking）

无必须修改项。

---

## 三、建议修改项（Optional）

无建议修改项。

---

## 四、修正版设计方案（V2）

### 1. 页面结构

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

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Modal | 弹窗 |
| Radio | 单选 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| PlanCard | 套餐卡片 |
| PriceDisplay | 价格展示 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| SubscribeModal | 订阅弹窗 | **新建** |
| PlanSelection | 套餐选择 | **新建** |
| PaymentConfirm | 支付确认 | **新建** |

---

### 3. 数据流

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

### 4. 状态管理

```typescript
interface SubscribeState {
  step: 'select' | 'confirm' | 'success';
  selectedPlan: Plan | null;
  loading: boolean;
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力订阅 | POST | `/api/v1/abilities/:id/subscribe` | 订阅能力 |

---

### 6. 用户交互流程

```
[点击订阅按钮]
    │
    ▼
[选择套餐]
    │
    ▼
[确认支付]
    │
    ▼
[订阅成功]
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 已订阅 | 提示已订阅 |
| 支付失败 | 提示错误，保留选择 |

---

### 8. 性能优化

- 套餐数据预加载
- 支付结果轮询

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 支付安全 | 高 | 支付信息泄露 | 使用支付网关 |

---

### 10. 开发步骤拆分

#### Step 1: 订阅弹窗（1 天）
- [ ] SubscribeModal + PlanSelection

#### Step 2: 支付流程（1 天）
- [ ] PaymentConfirm + 支付集成

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **设计内容** | 无修改 | 保持原设计 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 原设计已通过评审
2. 无阻塞项
3. 风险等级：低
