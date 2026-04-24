# XBIS 前端组件架构设计

> 版本：v1.0  
> 适用范围：XBIS 用户端 + 管理端

---

## 1. 架构总览

组件库采用 **四层分层架构**，从底层到上层依次为：

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 4: blocks/      页面块组件 (Page Blocks)              │
│  ─────────────────────────────────────────────────────────  │
│  StatCards, AbilityGrid, BillingSummary, TaskFilterBar...   │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: business/    业务组件 (Business Components)        │
│  ─────────────────────────────────────────────────────────  │
│  AbilityCard, TaskItem, RiskBadge, QuotaBar, PriceDisplay...│
├─────────────────────────────────────────────────────────────┤
│  Layer 2: layout/      布局组件 (Layout Components)          │
│  ─────────────────────────────────────────────────────────  │
│  AppShell, Sidebar, TopNav, PageHeader, ContentArea...      │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: base/        基础组件 (Base Components)            │
│  ─────────────────────────────────────────────────────────  │
│  Button, Card, Input, Tag, Modal, Table, StatusDot...       │
└─────────────────────────────────────────────────────────────┘
```

**依赖规则**：上层可以依赖下层，同层之间可以互相依赖，但禁止反向依赖。

---

## 2. 命名规范

### 2.1 文件/目录命名
- **组件目录**：PascalCase，与组件名完全一致，如 `AbilityCard/`
- **入口文件**：每个组件目录下统一为 `index.tsx`
- **类型文件**：如需独立类型文件，命名为 `types.ts`
- **样式文件**：如需独立样式，命名为 `style.ts` 或 `*.module.css`

### 2.2 组件命名
- **基础组件**：语义化命名，如 `Button`, `Card`, `StatusDot`
- **业务组件**：{Domain}{Component} 形式，如 `AbilityCard`, `TaskItem`, `RiskBadge`
- **布局组件**：结构语义命名，如 `AppShell`, `TopNav`, `ContentArea`
- **页面块组件**：{Domain}{BlockType} 形式，如 `AbilityGrid`, `BillingSummary`, `TaskFilterBar`

### 2.3 Props 命名
- 布尔类型：`isXxx`, `hasXxx`, `showXxx`, `disabled`, `loading`
- 回调类型：`onXxx`（如 `onClick`, `onSubmit`, `onStatusChange`）
- 配置类型：`variant`, `size`, `theme`, `mode`

---

## 3. 各层详细设计

### 3.1 基础组件层 (base/)

**定位**：对 Ant Design 5.x 的二次封装 + 纯 UI 原子组件。不依赖任何业务逻辑，可在任意项目复用。

**设计原则**：
- 所有组件必须接入 Design Tokens（`@xbis/tokens`）
- 保持与 Ant Design API 的兼容性，降低迁移成本
- 提供 `variant` / `size` 等配置化属性，减少样式硬编码

| 组件 | 说明 | 对应 Ant Design 组件 |
|------|------|---------------------|
| Button | 品牌化按钮，支持 variant/size | Button |
| Input | 品牌化输入框，支持 variant | Input |
| Card | 品牌化卡片，支持 variant/padding | Card |
| Badge | 状态徽标，支持 variant | Badge |
| Tag | 语义化标签，支持多色 variant | Tag |
| Empty | 空状态，接入 Token 样式 | Empty |
| Spinner | 加载动画，品牌色 | Spin |
| Avatar | 头像，品牌回退色 | Avatar |
| Tooltip | 提示，支持 variant | Tooltip |
| Modal | 弹窗，品牌圆角/阴影 | Modal |
| Drawer | 抽屉，品牌圆角/阴影 | Drawer |
| Table | 表格，支持 compact/card 模式 | Table |
| Tabs | 标签页，支持 pill/underline 模式 | Tabs |
| Alert | 警告提示，支持 brand variant | Alert |
| Form | 表单，支持 gap 配置 | Form |
| Search | 搜索框，胶囊圆角 | Input.Search |
| CopyButton | 复制按钮，带反馈状态 | Button + Icon |
| StatusDot | 状态圆点，支持 pulse 动画 | 自定义 |
| Divider | 分割线，支持 variant | Divider |
| Skeleton | 骨架屏，支持多种变体 | Skeleton |

### 3.2 业务组件层 (business/)

**定位**：封装 XBIS 领域概念，依赖业务类型（`@xbis/shared` 中的 TypeScript 类型）。用户端 & 管理端共享。

**设计原则**：
- 每个组件对应一个明确的业务实体或状态
- Props 中使用业务类型而非原始类型
- 保持纯展示，数据获取逻辑上提到页面层

| 组件 | 说明 | 依赖类型 |
|------|------|----------|
| AbilityCard | 能力卡片，展示 API 市场能力 | ApiMarketItem |
| TaskItem | 任务列表项 | Job |
| TaskStatusBadge | 任务状态徽章（含脉冲动画） | JobStatus |
| BillingCard | 账单统计卡片 | - |
| PlanCard | 套餐卡片 | SubscriptionPlan |
| ApiKeyItem | API 密钥项 | ApiKey |
| UsageChart | 使用趋势图表 | - |
| QuotaBar | 额度进度条 | - |
| RiskBadge | 风险等级徽章 | RiskLevel |
| ExecutionModeTag | 执行模式标签 | ExecutionMode |
| NotificationItem | 通知项 | - |
| InvoiceItem | 发票项 | - |
| CouponTag | 代金券标签 | - |
| UserMiniCard | 用户迷你卡片 | - |
| AuditActionBar | 审核操作栏 | - |
| SchemaPreview | Schema 预览 | - |
| JobTimeline | 任务时间线 | - |
| PriceDisplay | 价格展示 | PricingType |
| DataMetric | 数据指标 | - |
| AiSparkle | AI 标识徽标 | - |

### 3.3 布局组件层 (layout/)

**定位**：负责页面级骨架结构。用户端 & 管理端共享，通过 props / 配置区分不同端的表现。

| 组件 | 说明 | 用户端 | 管理端 |
|------|------|--------|--------|
| AppShell | 应用外壳（Header + Sider + Content） | 使用 | 使用 |
| Sidebar | 侧边栏导航 | 白色背景 | 深色背景 |
| TopNav | 顶部导航栏 | 使用 | 使用 |
| PageHeader | 页面标题区（含面包屑/操作） | 使用 | 使用 |
| ContentArea | 内容区容器 | 使用 | 使用 |
| BreadcrumbNav | 面包屑导航 | 使用 | 使用 |
| Footer | 页脚 | 可选 | 可选 |
| MobileDrawer | 移动端抽屉菜单 | 使用 | 使用 |
| UserDropdown | 用户下拉菜单 | 使用 | 使用 |
| NotificationBell | 通知铃铛 | 使用 | 使用 |

### 3.4 页面块组件层 (blocks/)

**定位**：由基础组件 + 业务组件组合而成的页面级模块。通常对应页面上的独立区块，可直接嵌入页面使用。

| 组件 | 说明 | 典型使用页面 |
|------|------|-------------|
| StatCards | 统计卡片组 | Dashboard |
| RecentTasks | 最近任务列表 | Dashboard, 任务页 |
| QuickActions | 快捷操作区 | Dashboard |
| AbilityGrid | 能力网格 | 能力中心 |
| TaskFilterBar | 任务筛选栏 | 任务列表 |
| BillingSummary | 账单汇总 | 账单页 |
| UsageTrend | 使用趋势 | 统计页 |
| NotificationList | 通知列表 | 通知页 |
| ApiKeyTable | API 密钥表格 | 开发者中心 |
| AuditLogTable | 审核日志表格 | 审核页 |
| PlanComparison | 套餐对比 | 订阅页 |
| InvoiceTable | 发票表格 | 发票页 |
| UserProfileForm | 用户资料表单 | 个人中心 |
| AbilityDetailPanel | 能力详情面板 | 能力详情页 |
| JobResultViewer | 任务结果查看器 | 任务详情页 |
| SchemaFormBuilder | Schema 表单构建器 | 能力发布页 |
| ExecutorConfigPanel | 执行器配置面板 | 管理端-执行器管理 |

---

## 4. 已存在组件 → 新分层归位表

### 4.1 现有组件清单

当前项目中，用户端和管理端各有一个 `Layout.tsx` 组件，其余页面均为页面级文件，无独立组件层。

| 现有文件 | 当前位置 | 组件类型 | 新分层归位 | 迁移方式 |
|----------|----------|----------|-----------|----------|
| Layout.tsx | `packages/user/src/components/Layout.tsx` | 布局组件 | `packages/components/layout/AppShell/` + `packages/components/layout/Sidebar/` + `packages/components/layout/TopNav/` | **渐进替换**：新页面使用新组件，旧页面逐步迁移 |
| Layout.tsx | `packages/admin/src/components/Layout.tsx` | 布局组件 | 同上 | **渐进替换**：管理端 Layout 逻辑抽象为配置，复用同一套 layout 组件 |

### 4.2 页面内可提取的组件（未来迁移建议）

以下组件目前内嵌在页面文件中，建议在页面重构时逐步提取到对应分层：

| 页面文件 | 内嵌组件/区块 | 建议归位分层 | 优先级 |
|----------|--------------|-------------|--------|
| Dashboard.tsx | 统计卡片组 | `blocks/StatCards` | P1 |
| Dashboard.tsx | 快捷操作按钮组 | `blocks/QuickActions` | P1 |
| Dashboard.tsx | 最近调用记录列表 | `blocks/RecentTasks` | P1 |
| ApiMarket.tsx | 能力卡片网格 | `blocks/AbilityGrid` | P0 |
| ApiMarketDetail.tsx | 能力详情面板 | `blocks/AbilityDetailPanel` | P0 |
| ApiKeys.tsx | 密钥列表项 | `business/ApiKeyItem` | P1 |
| Invocations.tsx | 调用记录筛选栏 | `blocks/TaskFilterBar` | P1 |
| Billing.tsx | 账单统计卡片 | `blocks/BillingSummary` | P1 |
| Subscription.tsx | 套餐对比卡片 | `blocks/PlanComparison` | P1 |
| Invoices.tsx | 发票列表项 | `business/InvoiceItem` | P2 |
| Coupons.tsx | 代金券标签 | `business/CouponTag` | P2 |
| Notifications.tsx | 通知列表项 | `business/NotificationItem` | P1 |
| Account.tsx | 用户资料表单 | `blocks/UserProfileForm` | P1 |
| Statistics.tsx | 使用趋势图表 | `blocks/UsageTrend` | P2 |
| Admin Dashboard | 数据指标卡片 | `blocks/StatCards` | P1 |
| Users.tsx | 用户迷你卡片 | `business/UserMiniCard` | P2 |
| Reviews.tsx | 审核操作栏 | `business/AuditActionBar` | P1 |
| Reviews.tsx | 审核日志表格 | `blocks/AuditLogTable` | P1 |
| ApiPermissions.tsx | Schema 预览 | `business/SchemaPreview` | P1 |
| Plans.tsx | 套餐卡片 | `business/PlanCard` | P1 |
| Monitoring.tsx | 任务时间线 | `business/JobTimeline` | P2 |
| Settings.tsx | Schema 表单构建器 | `blocks/SchemaFormBuilder` | P1 |

### 4.3 迁移方式说明

#### 阶段一：新建组件（当前）
- 在 `packages/components/` 下创建完整的四层组件结构
- 新组件全部使用 Design Tokens（`@xbis/tokens`）
- 新组件全部使用 TypeScript 类型（`@xbis/shared`）

#### 阶段二：新页面使用新组件
- 新增页面（如能力中心、任务平台）直接使用 `@xbis/components`
- 通过 `import { AbilityCard } from '@xbis/components/business'` 引用

#### 阶段三：旧页面逐步迁移
- 旧页面内的内嵌组件，在功能迭代时逐步提取到对应分层
- 提取顺序：先提取 `business/` 层（复用价值高），再提取 `blocks/` 层
- 保留旧页面代码不变，仅将可复用部分下沉到组件库

#### 阶段四：Layout 统一替换
- 当 `layout/` 层组件稳定后，替换用户端和管理端的 `Layout.tsx`
- 替换方式：将现有 `Layout.tsx` 中的硬编码逻辑抽象为配置，传入统一的 `AppShell`

---

## 5. 使用示例

### 5.1 基础组件使用

```tsx
import { Button, Card, Tag, StatusDot } from '@xbis/components/base';

<Card variant="elevated" padding="default">
  <h3>能力名称</h3>
  <Tag variant="success">已发布</Tag>
  <StatusDot status="success" pulse />
  <Button variant="primary" size="medium">立即接入</Button>
</Card>
```

### 5.2 业务组件使用

```tsx
import { AbilityCard, TaskItem, RiskBadge } from '@xbis/components/business';
import type { ApiMarketItem, Job } from '@xbis/shared';

<AbilityCard
  ability={apiMarketItem}
  onClick={(ability) => navigate(`/ability/${ability.apiId}`)}
  onSubscribe={(ability) => openSubscribeModal(ability)}
/>

<TaskItem task={job} onClick={(task) => showTaskDetail(task)} />
```

### 5.3 布局组件使用

```tsx
import { AppShell, TopNav, Sidebar, ContentArea, PageHeader } from '@xbis/components/layout';

<AppShell
  topNav={<TopNav logo={<Logo />} title="XBIS" userSlot={<UserDropdown />} />}
  sidebar={<Sidebar items={menuItems} onSelect={handleMenuSelect} />}
  content={
    <ContentArea>
      <PageHeader title="能力中心" breadcrumbs={breadcrumbs} />
      {/* 页面内容 */}
    </ContentArea>
  }
/>
```

### 5.4 页面块组件使用

```tsx
import { StatCards, AbilityGrid, BillingSummary } from '@xbis/components/blocks';

<StatCards
  stats={[
    { label: '总调用量', value: 12580, change: 12.5 },
    { label: '活跃能力', value: 24, change: 4 },
  ]}
/>

<AbilityGrid
  abilities={abilities}
  onAbilityClick={handleAbilityClick}
  onSubscribe={handleSubscribe}
/>
```

---

## 6. 目录结构

```
packages/components/
├── package.json
├── index.ts                    # 统一导出入口
├── base/
│   ├── index.ts                # base 层导出入口
│   ├── Button/
│   │   └── index.tsx
│   ├── Card/
│   │   └── index.tsx
│   ├── Input/
│   │   └── index.tsx
│   ├── Tag/
│   │   └── index.tsx
│   ├── Badge/
│   │   └── index.tsx
│   ├── Avatar/
│   │   └── index.tsx
│   ├── Tooltip/
│   │   └── index.tsx
│   ├── Modal/
│   │   └── index.tsx
│   ├── Drawer/
│   │   └── index.tsx
│   ├── Table/
│   │   └── index.tsx
│   ├── Tabs/
│   │   └── index.tsx
│   ├── Alert/
│   │   └── index.tsx
│   ├── Form/
│   │   └── index.tsx
│   ├── Search/
│   │   └── index.tsx
│   ├── CopyButton/
│   │   └── index.tsx
│   ├── StatusDot/
│   │   └── index.tsx
│   ├── Divider/
│   │   └── index.tsx
│   ├── Empty/
│   │   └── index.tsx
│   └── Skeleton/
│       └── index.tsx
├── business/
│   ├── index.ts                # business 层导出入口
│   ├── AbilityCard/
│   │   └── index.tsx
│   ├── TaskItem/
│   │   └── index.tsx
│   ├── TaskStatusBadge/
│   │   └── index.tsx
│   ├── BillingCard/
│   │   └── index.tsx
│   ├── PlanCard/
│   │   └── index.tsx
│   ├── ApiKeyItem/
│   │   └── index.tsx
│   ├── UsageChart/
│   │   └── index.tsx
│   ├── QuotaBar/
│   │   └── index.tsx
│   ├── RiskBadge/
│   │   └── index.tsx
│   ├── ExecutionModeTag/
│   │   └── index.tsx
│   ├── NotificationItem/
│   │   └── index.tsx
│   ├── InvoiceItem/
│   │   └── index.tsx
│   ├── CouponTag/
│   │   └── index.tsx
│   ├── UserMiniCard/
│   │   └── index.tsx
│   ├── AuditActionBar/
│   │   └── index.tsx
│   ├── SchemaPreview/
│   │   └── index.tsx
│   ├── JobTimeline/
│   │   └── index.tsx
│   ├── PriceDisplay/
│   │   └── index.tsx
│   ├── DataMetric/
│   │   └── index.tsx
│   └── AiSparkle/
│       └── index.tsx
├── layout/
│   ├── index.ts                # layout 层导出入口
│   ├── AppShell/
│   │   └── index.tsx
│   ├── Sidebar/
│   │   └── index.tsx
│   ├── TopNav/
│   │   └── index.tsx
│   ├── PageHeader/
│   │   └── index.tsx
│   ├── ContentArea/
│   │   └── index.tsx
│   ├── BreadcrumbNav/
│   │   └── index.tsx
│   ├── Footer/
│   │   └── index.tsx
│   ├── MobileDrawer/
│   │   └── index.tsx
│   ├── UserDropdown/
│   │   └── index.tsx
│   └── NotificationBell/
│       └── index.tsx
└── blocks/
    ├── index.ts                # blocks 层导出入口
    ├── StatCards/
    │   └── index.tsx
    ├── RecentTasks/
    │   └── index.tsx
    ├── QuickActions/
    │   └── index.tsx
    ├── AbilityGrid/
    │   └── index.tsx
    ├── TaskFilterBar/
    │   └── index.tsx
    ├── BillingSummary/
    │   └── index.tsx
    ├── UsageTrend/
    │   └── index.tsx
    ├── NotificationList/
    │   └── index.tsx
    ├── ApiKeyTable/
    │   └── index.tsx
    ├── AuditLogTable/
    │   └── index.tsx
    ├── PlanComparison/
    │   └── index.tsx
    ├── InvoiceTable/
    │   └── index.tsx
    ├── UserProfileForm/
    │   └── index.tsx
    ├── AbilityDetailPanel/
    │   └── index.tsx
    ├── JobResultViewer/
    │   └── index.tsx
    ├── SchemaFormBuilder/
    │   └── index.tsx
    └── ExecutorConfigPanel/
        └── index.tsx
```

---

## 7. 与现有项目的关系

### 7.1 不重叠原则
- 新组件库 `packages/components/` 与现有 `packages/user/src/components/Layout.tsx` 和 `packages/admin/src/components/Layout.tsx` **功能不重叠**
- 现有 Layout 是页面级硬编码组件，新 Layout 是配置化、可复用的组件库
- 现有页面文件（`packages/*/src/pages/*.tsx`）保持不变，仅在新开发时引用新组件

### 7.2 渐进式采用
- **Step 1**：新功能/新页面直接使用 `@xbis/components`
- **Step 2**：旧页面在迭代维护时，将内嵌组件逐步提取到组件库
- **Step 3**：当组件库覆盖率达到 80% 以上时，统一替换 Layout 层

### 7.3 pnpm workspace 配置

在根目录 `pnpm-workspace.yaml` 中已包含 `packages/*`，`packages/components` 会自动被纳入 workspace。

用户端和管理端的 `package.json` 中添加依赖：

```json
{
  "dependencies": {
    "@xbis/components": "workspace:*"
  }
}
```

---

## 8. 附录：组件依赖关系图

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
  │
  ▼
layout/ (依赖 base/ + business/)
  ├── AppShell ──► (Ant Layout)
  ├── Sidebar ──► (Ant Menu)
  ├── TopNav ──► (纯样式)
  ├── PageHeader ──► BreadcrumbNav
  ├── ContentArea ──► (纯样式)
  ├── BreadcrumbNav ──► (Ant Breadcrumb)
  ├── Footer ──► (纯样式)
  ├── MobileDrawer ──► (Ant Drawer)
  ├── UserDropdown ──► Avatar
  └── NotificationBell ──► Badge, Button
  │
  ▼
blocks/ (依赖 base/ + business/ + layout/)
  ├── StatCards ──► Card, DataMetric
  ├── RecentTasks ──► Card, Button, TaskItem
  ├── QuickActions ──► Card, Button
  ├── AbilityGrid ──► Empty, AbilityCard
  ├── TaskFilterBar ──► Search, Tag
  ├── BillingSummary ──► Card, BillingCard, QuotaBar
  ├── UsageTrend ──► Card, UsageChart
  ├── NotificationList ──► Card, Button, Empty, NotificationItem
  ├── ApiKeyTable ──► Card, ApiKeyItem
  ├── AuditLogTable ──► Card, Table, AuditActionBar, UserMiniCard
  ├── PlanComparison ──► Card, PlanCard
  ├── InvoiceTable ──► Card, InvoiceItem
  ├── UserProfileForm ──► Card, Input, Button, Form, UserMiniCard
  ├── AbilityDetailPanel ──► Card, Tag, Button, RiskBadge, ExecutionModeTag, SchemaPreview, PriceDisplay, AiSparkle
  ├── JobResultViewer ──► Card, Tag, TaskStatusBadge, JobTimeline
  ├── SchemaFormBuilder ──► Card, Input, Button, Form
  └── ExecutorConfigPanel ──► Card, Input, Button, Tag, StatusDot
```

---

*本文档与组件代码同步维护，新增组件时请同步更新此文档。*
