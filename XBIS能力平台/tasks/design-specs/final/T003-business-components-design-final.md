# T003 业务组件库 — 最终可开发设计方案

> 任务卡: [T003-business-components.md](../T003-business-components.md)
> 设计输入: docs/component-architecture.md, docs/dev-rules.md, docs/design-tokens.md
> 输出日期: 2026-04-24
> 文档版本: Final

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
├── AiSparkle/
│   ├── index.tsx
│   └── types.ts
└── CopyButton/
    ├── index.tsx
    └── types.ts
```

---

## 2. 组件拆分

### business/ 层组件（分优先级开发）

**第一批（P0 - 核心组件）**：

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| AbilityCard | `Ability` | 能力卡片 |
| TaskItem | `Job` | 任务列表项 |
| TaskStatusBadge | `JobStatus` | 任务状态徽章 |
| PlanCard | `Plan` | 套餐卡片 |
| ApiKeyItem | `AccessKey` | API Key 列表项 |

**第二批（P1 - 重要组件）**：

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| BillingCard | `BillingItem` | 账单卡片 |
| UsageChart | `UsageData` | 用量图表 |
| QuotaBar | `QuotaUsage` | 配额进度条 |
| RiskBadge | `RiskLevel` | 风险等级徽章 |
| ExecutionModeTag | `ExecutionMode` | 执行模式标签 |

**第三批（P2 - 按需开发）**：

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| InvoiceCard | `Invoice` | 发票卡片 |
| NotificationBell | `Notification` | 通知铃铛 |
| CopyButton | - | 复制按钮 |
| DataMetric | `MetricData` | 数据指标展示 |
| PriceDisplay | `PriceData` | 价格展示 |
| AiSparkle | - | AI 特性标识 |
| CouponTag | `Coupon` | 优惠券标签 |

### 组件依赖关系

```
base/ (无依赖)
  ├── Button, Card, Input, Tag, Badge, Avatar, Tooltip
  ├── Modal, Drawer, Table, Tabs, Alert, Form, Search
  ├── StatusDot, Divider, Empty, Skeleton
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
  ├── AiSparkle ──► (纯样式)
  └── CopyButton ──► (纯样式)
```

### 图表库选择

推荐使用 **Recharts**，理由：
- React 原生支持，API 友好
- 体积小，Tree-shaking 友好
- 与 Ant Design 风格一致
- 支持响应式

---

## 3. 数据流

业务组件为纯展示型，数据通过 Props 传入。

```
页面层 ──► Props（业务类型） ──► Business Component ──► UI 渲染
```

### 示例：AbilityCard 数据流

```typescript
interface AbilityCardProps {
  ability: Ability;
  onClick?: (ability: Ability) => void;
  onSubscribe?: (ability: Ability) => void;
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
| 类型不匹配 | 使用 TypeScript 严格模式校验 |

---

## 8. 性能优化

- 组件使用 `React.memo` 避免不必要的重渲染
- 列表场景使用 `key` 属性
- 图表组件使用懒加载
- 按需导出，支持 Tree-shaking

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与现有页面组件重复 | 高 | 现有页面可能已有类似实现 | 开发前必须梳理现有组件，标记可提取的组件 |
| 业务类型定义不完整 | 中 | 类型定义可能遗漏字段 | 与后端对齐 API 契约 |
| 组件过度设计 | 中 | 可能创建过多细粒度组件 | 遵循「单一职责」原则 |
| 图表库体积 | 中 | Recharts 可能增加包体积 | 按需引入，懒加载 |

---

## 10. 开发步骤拆分

### Step 1: 梳理现有组件（0.5 人日）
- [ ] 梳理现有页面中的可复用组件
- [ ] 标记可提取的组件
- [ ] 避免重复实现

### Step 2: 第一批 P0 组件（1.5 人日）
- [ ] AbilityCard
- [ ] TaskItem
- [ ] TaskStatusBadge
- [ ] PlanCard
- [ ] ApiKeyItem

### Step 3: 第二批 P1 组件（1.5 人日）
- [ ] BillingCard
- [ ] UsageChart（集成 Recharts）
- [ ] QuotaBar
- [ ] RiskBadge
- [ ] ExecutionModeTag

### Step 4: 第三批 P2 组件（按需）
- [ ] 其余组件按需开发

### Step 5: 统一导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写组件使用文档
