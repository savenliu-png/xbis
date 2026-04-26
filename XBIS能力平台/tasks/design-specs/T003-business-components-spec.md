# T003 业务组件库 — 工程级设计方案

> 任务卡: [T003-business-components.md](../T003-business-components.md)  
> 设计输入: docs/component-architecture.md, docs/dev-rules.md, docs/design-tokens.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

本任务为**基础能力层**建设，无页面结构。输出物为组件库代码包。

```
packages/components/business/       # 业务组件包
├── index.ts                        # 统一导出入口
├── AbilityCard/
│   ├── index.tsx
│   └── types.ts
├── TaskItem/
│   ├── index.tsx
│   └── types.ts
├── TaskStatusBadge/
│   ├── index.tsx
│   └── types.ts
├── BillingCard/
│   ├── index.tsx
│   └── types.ts
├── PlanCard/
│   ├── index.tsx
│   └── types.ts
├── ApiKeyItem/
│   ├── index.tsx
│   └── types.ts
├── UsageChart/
│   ├── index.tsx
│   └── types.ts
├── QuotaBar/
│   ├── index.tsx
│   └── types.ts
├── RiskBadge/
│   ├── index.tsx
│   └── types.ts
├── ExecutionModeTag/
│   ├── index.tsx
│   └── types.ts
├── NotificationItem/
│   ├── index.tsx
│   └── types.ts
├── InvoiceItem/
│   ├── index.tsx
│   └── types.ts
├── CouponTag/
│   ├── index.tsx
│   └── types.ts
├── UserMiniCard/
│   ├── index.tsx
│   └── types.ts
├── AuditActionBar/
│   ├── index.tsx
│   └── types.ts
├── SchemaPreview/
│   ├── index.tsx
│   └── types.ts
├── JobTimeline/
│   ├── index.tsx
│   └── types.ts
├── PriceDisplay/
│   ├── index.tsx
│   └── types.ts
├── DataMetric/
│   ├── index.tsx
│   └── types.ts
└── AiSparkle/
    ├── index.tsx
    └── types.ts
```

---

## 2. 组件拆分

### business/ 层（20 个组件）

| 组件 | 说明 | 依赖类型 | 依赖 base/ 组件 |
|------|------|----------|----------------|
| AbilityCard | 能力卡片 | ApiMarketItem | Card, Tag, StatusDot, Button |
| TaskItem | 任务列表项 | Job | Card, CopyButton, Tag |
| TaskStatusBadge | 任务状态徽章 | JobStatus | StatusDot, Tag |
| BillingCard | 账单统计卡片 | - | Card, DataMetric |
| PlanCard | 套餐卡片 | SubscriptionPlan | Card, Button, Tag |
| ApiKeyItem | API 密钥项 | ApiKey | Card, CopyButton, Tag |
| UsageChart | 使用趋势图表 | - | Empty |
| QuotaBar | 额度进度条 | - | - |
| RiskBadge | 风险等级徽章 | RiskLevel | Tag |
| ExecutionModeTag | 执行模式标签 | ExecutionMode | Tag |
| NotificationItem | 通知项 | - | Card, StatusDot |
| InvoiceItem | 发票项 | - | Card, Tag |
| CouponTag | 代金券标签 | - | Tag |
| UserMiniCard | 用户迷你卡片 | - | Avatar |
| AuditActionBar | 审核操作栏 | - | Button |
| SchemaPreview | Schema 预览 | - | Card |
| JobTimeline | 任务时间线 | - | StatusDot |
| PriceDisplay | 价格展示 | PricingType | - |
| DataMetric | 数据指标 | - | - |
| AiSparkle | AI 标识徽标 | - | - |

### 组件依赖关系

```
base/ (无依赖)
  ├── Button, Card, Input, Tag, Badge, Avatar, Tooltip
  ├── Modal, Drawer, Table, Tabs, Alert, Form, Search
  ├── CopyButton, StatusDot, Divider, Empty, Skeleton
  │
  ▼
business/ (依赖 base/ + @xbis/shared types)
  ├── AbilityCard ──► Card, Tag, StatusDot, RiskBadge, ExecutionModeTag, PriceDisplay
  ├── TaskItem ──► Card, CopyButton, TaskStatusBadge
  ├── TaskStatusBadge ──► StatusDot, Tag
  ├── BillingCard ──► Card, DataMetric
  ├── PlanCard ──► Card, Button, Tag
  ├── ApiKeyItem ──► Card, CopyButton, Tag
  ├── UsageChart ──► Empty
  ├── QuotaBar ──► (纯样式)
  ├── RiskBadge ──► Tag
  ├── ExecutionModeTag ──► Tag
  ├── NotificationItem ──► Card, StatusDot
  ├── InvoiceItem ──► Card, Tag
  ├── CouponTag ──► Tag
  ├── UserMiniCard ──► Avatar
  ├── AuditActionBar ──► Button
  ├── SchemaPreview ──► Card
  ├── JobTimeline ──► StatusDot
  ├── PriceDisplay ──► (纯样式)
  ├── DataMetric ──► (纯样式)
  └── AiSparkle ──► (纯样式)
```

---

## 3. 数据流

业务组件为纯展示型，数据通过 Props 传入。

```
页面层 ──► Props（业务类型） ──► Business Component ──► UI 渲染
```

### 示例：AbilityCard 数据流

```typescript
interface AbilityCardProps {
  ability: ApiMarketItem;
  onClick?: (ability: ApiMarketItem) => void;
  onSubscribe?: (ability: ApiMarketItem) => void;
}
```

---

## 4. 状态管理

- 组件内部状态：使用 `useState` 管理局部 UI 状态
- 无全局状态依赖
- 数据获取逻辑上提到页面层

---

## 5. API 调用

无 API 调用。数据通过 Props 传入。

---

## 6. 用户交互流程

| 组件 | 交互 | 行为 | 结果 |
|------|------|------|------|
| AbilityCard | 点击卡片 | onClick 回调 | 跳转能力详情 |
| AbilityCard | 点击「接入」按钮 | onSubscribe 回调 | 打开接入弹窗 |
| TaskItem | 点击复制 | 复制 jobId | 显示复制成功 |
| ApiKeyItem | 点击复制 | 复制密钥 | 显示复制成功 |
| AuditActionBar | 点击通过/拒绝 | onAudit 回调 | 提交审核结果 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 缺少必填 Props | TypeScript 编译时检查 |
| 数据字段缺失 | 使用可选链 + 默认值 |
| 图片加载失败 | 显示 fallback 头像/图标 |

---

## 8. 性能优化

- 组件使用 `React.memo` 避免不必要的重渲染
- 列表场景使用 `key` 属性
- 图表组件使用懒加载

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与现有页面内嵌组件重复 | 高 | 现有页面可能已有类似实现 | 先梳理现有页面，标记可提取组件 |
| 业务类型定义不完整 | 中 | 类型定义可能遗漏字段 | 与后端对齐 API 契约 |
| 组件过度设计 | 中 | 可能创建过多细粒度组件 | 遵循「单一职责」原则 |

---

## 10. 开发步骤拆分

### Step 1: 核心能力组件（AbilityCard, TaskItem, TaskStatusBadge）（1 人日）
- [ ] AbilityCard 组件
- [ ] TaskItem 组件
- [ ] TaskStatusBadge 组件

### Step 2: 商业化组件（BillingCard, PlanCard, ApiKeyItem, PriceDisplay）（1 人日）
- [ ] BillingCard 组件
- [ ] PlanCard 组件
- [ ] ApiKeyItem 组件
- [ ] PriceDisplay 组件

### Step 3: 状态/标识组件（RiskBadge, ExecutionModeTag, StatusDot, AiSparkle）（0.5 人日）
- [ ] RiskBadge 组件
- [ ] ExecutionModeTag 组件
- [ ] StatusDot 组件
- [ ] AiSparkle 组件

### Step 4: 数据展示组件（UsageChart, QuotaBar, DataMetric, JobTimeline）（1 人日）
- [ ] UsageChart 组件
- [ ] QuotaBar 组件
- [ ] DataMetric 组件
- [ ] JobTimeline 组件

### Step 5: 其他组件（NotificationItem, InvoiceItem, CouponTag, UserMiniCard, AuditActionBar, SchemaPreview）（0.5 人日）
- [ ] NotificationItem 组件
- [ ] InvoiceItem 组件
- [ ] CouponTag 组件
- [ ] UserMiniCard 组件
- [ ] AuditActionBar 组件
- [ ] SchemaPreview 组件

### Step 6: 统一导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写组件使用文档
