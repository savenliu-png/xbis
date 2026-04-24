# 业务组件库（business/）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 业务组件库（business/） |
| 任务编号 | T003 |
| 所属模块 ⭐ | M1 基础设计系统 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-05 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏业务级组件，各页面重复实现相同功能：
- 能力卡片在不同页面实现不一致
- 任务状态标识样式不统一
- 价格展示格式混乱

### 2.2 目标用户
- 前端开发者：使用业务组件快速构建页面
- 产品经理：确保业务展示一致性

### 2.3 预期效果
- 建立 20 个业务组件，覆盖能力/任务/计费/用户等场景
- 所有组件基于 base 层，保持视觉一致
- 提供清晰的 props 接口，易于使用

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局组件 | — | 所有页面 |

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局组件 | — | 所有页面 |

### 3.3 页面原型/设计稿
- 参考：`packages/components/business/index.ts`

## 4. 功能范围

### 4.1 包含功能
- [ ] AbilityCard（能力卡片）
- [ ] AbilityGrid（能力网格布局）
- [ ] TaskItem（任务列表项）
- [ ] TaskStatusBadge（任务状态标识）
- [ ] TaskFilterBar（任务筛选栏）
- [ ] JobTimeline（任务执行时间线）
- [ ] JobResultViewer（任务结果查看器）
- [ ] RiskBadge（风险等级标识）
- [ ] ExecutionModeTag（执行模式标签）
- [ ] PriceDisplay（价格展示）
- [ ] PlanCard（套餐卡片）
- [ ] QuotaBar（配额进度条）
- [ ] BillingSummary（计费概览面板）
- [ ] UsageTrend（用量趋势图表）
- [ ] InvoiceTable（账单列表）
- [ ] InvoiceItem（账单项）
- [ ] CouponTag（优惠券标签）
- [ ] ApiKeyItem（API Key 项）
- [ ] ApiKeyTable（API Key 表格）
- [ ] UserProfileCard（用户资料卡）

### 4.2 不包含功能（明确排除）
- 页面级布局（由 blocks/ 层实现）
- 纯展示组件（由 base/ 层实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// AbilityCard Props
export interface AbilityCardProps {
  ability: ApiMarketItem;
  onClick?: (ability: ApiMarketItem) => void;
  onSubscribe?: (ability: ApiMarketItem) => void;
  showTags?: boolean;
  showPrice?: boolean;
  layout?: 'grid' | 'list';
}

// TaskItem Props
export interface TaskItemProps {
  job: Job;
  onClick?: (job: Job) => void;
  onCancel?: (jobId: string) => void;
  onRetry?: (jobId: string) => void;
  selected?: boolean;
  onSelect?: (jobId: string, selected: boolean) => void;
}

// TaskStatusBadge Props
export interface TaskStatusBadgeProps {
  status: JobStatus;
  size?: 'sm' | 'md';
}

// RiskBadge Props
export interface RiskBadgeProps {
  level: 'low' | 'medium' | 'high' | 'critical';
  size?: 'sm' | 'md';
}

// ExecutionModeTag Props
export interface ExecutionModeTagProps {
  mode: 'sync' | 'async' | 'review-only' | 'both';
  size?: 'sm' | 'md';
}

// PriceDisplay Props
export interface PriceDisplayProps {
  amount: number;
  currency?: string;
  unit?: string;
  size?: 'sm' | 'md' | 'lg';
  color?: 'default' | 'brand' | 'success' | 'error';
}

// PlanCard Props
export interface PlanCardProps {
  plan: SubscriptionPlan;
  isCurrent?: boolean;
  isPopular?: boolean;
  onSelect?: (plan: SubscriptionPlan) => void;
}

// QuotaBar Props
export interface QuotaBarProps {
  used: number;
  total: number;
  unit?: string;
  warningThreshold?: number;
  dangerThreshold?: number;
}

// BillingSummary Props
export interface BillingSummaryProps {
  currentPlan?: UserSubscription;
  quotaUsages: QuotaUsage[];
  balance: BalanceAccount;
}

// InvoiceItem Props
export interface InvoiceItemProps {
  invoice: Invoice;
  onDownload?: (invoiceNo: string) => void;
}

// CouponTag Props
export interface CouponTagProps {
  coupon: Coupon;
  onUse?: (couponNo: string) => void;
}

// ApiKeyItem Props
export interface ApiKeyItemProps {
  apiKey: ApiKeyListItem;
  onEdit?: (keyId: string) => void;
  onDelete?: (keyId: string) => void;
  onToggleStatus?: (keyId: string, status: string) => void;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/components/business/*/types.ts`
- 命名：`AbilityCardProps`, `TaskItemProps`, `TaskStatusBadgeProps`, `RiskBadgeProps`, `ExecutionModeTagProps`, `PriceDisplayProps`, `PlanCardProps`, `QuotaBarProps`, `BillingSummaryProps`, `InvoiceItemProps`, `CouponTagProps`, `ApiKeyItemProps`

## 6. 交互流程

### 6.1 主流程
```
[开发者引入业务组件] ──► [传入业务数据] ──► [组件渲染业务视图] ──► [用户交互] ──► [触发业务回调]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 数据缺失 | 传入的 ability/job 数据不完整 | 展示占位符或降级展示 | 开发时即发现（TypeScript） |
| 非法状态 | 传入未知状态值 | 展示默认样式 | 开发时即发现（TypeScript） |

### 6.3 边界情况
- 能力无图标：展示默认图标
- 任务无结果：展示空状态
- 配额未使用：进度条为 0
- 账单金额为 0：展示「免费」标签

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Card | AbilityCard/PlanCard 容器 | 否（T002） |
| Button | 操作按钮 | 否（T002） |
| Tag | 状态标签 | 否（T002） |
| Badge | 徽标 | 否（T002） |
| Progress | QuotaBar 基础 | 否（T002） |
| Table | InvoiceTable/ApiKeyTable | 否（T002） |
| Empty | 空状态 | 否（T002） |
| Alert | 警告提示 | 否（T002） |
| Tooltip | 文字提示 | 否（T002） |

### 7.2 业务组件（business/）
无（自身就是业务组件层）

### 7.3 页面块组件（blocks/）
无

### 7.4 页面模板（layout/）
无

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| AbilityCard | business | 能力卡片 |
| AbilityGrid | business | 能力网格布局 |
| TaskItem | business | 任务列表项 |
| TaskStatusBadge | business | 任务状态标识 |
| TaskFilterBar | business | 任务筛选栏 |
| JobTimeline | business | 任务执行时间线 |
| JobResultViewer | business | 任务结果查看器 |
| RiskBadge | business | 风险等级标识 |
| ExecutionModeTag | business | 执行模式标签 |
| PriceDisplay | business | 价格展示 |
| PlanCard | business | 套餐卡片 |
| QuotaBar | business | 配额进度条 |
| BillingSummary | business | 计费概览面板 |
| UsageTrend | business | 用量趋势图表 |
| InvoiceTable | business | 账单列表 |
| InvoiceItem | business | 账单项 |
| CouponTag | business | 优惠券标签 |
| ApiKeyItem | business | API Key 项 |
| ApiKeyTable | business | API Key 表格 |
| UserProfileCard | business | 用户资料卡 |

## 8. API 接口 ⭐

无

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] AbilityCard 展示能力名称/描述/分类/风险等级/价格
- [ ] TaskItem 展示任务状态/目标/模式/风险等级/时间
- [ ] TaskStatusBadge 支持所有 JobStatus 状态展示
- [ ] RiskBadge 支持 4 种风险等级（low/medium/high/critical）
- [ ] ExecutionModeTag 支持 4 种模式（sync/async/review-only/both）
- [ ] PriceDisplay 支持金额格式化（千分位/小数位）
- [ ] PlanCard 展示套餐名称/价格/功能列表/当前套餐标识
- [ ] QuotaBar 展示已用/剩余配额，支持阈值变色
- [ ] BillingSummary 展示当前套餐/配额/余额
- [ ] InvoiceTable 支持排序/筛选/分页
- [ ] ApiKeyTable 支持状态切换/删除/编辑
- [ ] 所有组件支持暗色模式

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 基于 base 层组件构建，不直接依赖 Ant Design
- [ ] 组件 props 有 JSDoc 注释
- [ ] 支持 ref 转发

### 9.3 性能验收
- [ ] 组件首屏渲染 < 100ms
- [ ] 大数据量列表支持虚拟滚动

### 9.4 兼容性验收
- [ ] 支持 Chrome/Firefox/Safari/Edge 最新 2 个版本

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T001` — Design Tokens 系统 — 待开发
  - [x] `T002` — 基础组件库 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 与 base 层耦合过紧 | 中 | 中 | 明确分层，business 只依赖 base |
| 组件 props 过多 | 中 | 低 | 使用 composition 模式拆分 |

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

- 优先实现高频组件（AbilityCard/TaskItem/TaskStatusBadge/PriceDisplay/PlanCard/QuotaBar）
- 图表组件（UsageTrend）使用 ECharts/Ant Design Charts 封装
- 与 base 层保持解耦，便于后续替换底层组件库

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
