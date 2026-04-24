# 计费系统升级（Billing System Upgrade）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 计费系统升级 |
| 任务编号 | FEATURE-202604-007 |
| 所属模块 ⭐ | 用户端 — 计费中心 / 管理端 — 财务管理 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 4 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-25 |

## 2. 需求背景

### 2.1 问题描述
当前计费系统仅支持简单的按次计费模式，随着能力平台引入多种定价策略（免费、按次、套餐包），需要升级计费中心以支持：
- 多种定价模型的展示和切换
- 套餐包的购买和管理
- 更精细的用量统计和配额监控

### 2.2 目标用户
- 终端用户（开发者）：查看账单、购买套餐、监控用量
- 终端用户（企业管理员）：管理团队配额、查看团队账单
- 平台管理员：配置定价策略、审核发票、管理优惠券

### 2.3 预期效果
- 用户可以清晰查看当前计费模式和用量
- 支持在线购买套餐包和升级计划
- 提供详细的用量统计和趋势分析
- 管理员可灵活配置各类能力的定价

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 计费中心首页 | `/billing` | 顶部导航「计费」 |
| 用量统计页 | `/billing/usage` | 计费中心「用量统计」标签 |
| 账单历史页 | `/billing/invoices` | 计费中心「账单」标签 |
| 套餐购买页 | `/billing/plans` | 计费中心「升级套餐」按钮 |
| 优惠券页 | `/billing/coupons` | 计费中心「我的优惠券」 |

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 定价管理 | `/admin/pricing` | 侧边栏「计费管理」 |
| 发票审核 | `/admin/invoices` | 侧边栏「财务管理」 |
| 优惠券批次 | `/admin/coupons` | 侧边栏「运营工具」 |

### 3.3 页面原型/设计稿
- Figma: `https://figma.com/xbis/billing-system`
- 参考实现: `packages/pages/billing/BillingDashboardTemplate`（新建）

## 4. 功能范围

### 4.1 包含功能
- [ ] 计费概览卡片（当前计划、剩余配额、本月用量、预估费用）
- [ ] 用量趋势图表（近 30 天调用量、按能力分类统计）
- [ ] 配额进度条（各类能力的已用/剩余配额可视化）
- [ ] 计划对比表格（免费版 / 专业版 / 企业版功能对比）
- [ ] 在线购买流程（选择计划 → 应用优惠券 → 确认支付 → 开通成功）
- [ ] 账单列表（历史账单、发票状态、下载 PDF）
- [ ] 优惠券管理（查看、激活、使用记录）
- [ ] 管理端定价配置（设置能力单价、套餐包内容、阶梯价格）

### 4.2 不包含功能（明确排除）
- 支付网关集成（使用现有支付接口，本任务只做前端流程）
- 退款流程（V2 迭代）
- 团队/企业子账户配额分配（独立任务）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 计费概览响应
export interface BillingOverviewResponse {
  currentPlan: {
    id: string;
    name: string;
    level: 'free' | 'pro' | 'enterprise';
    expiredAt?: string;
  };
  quota: {
    abilityId: string;
    abilityName: string;
    total: number;
    used: number;
    unit: 'call' | 'token' | 'byte';
  }[];
  usageThisMonth: {
    totalCalls: number;
    totalCost: number;
    byAbility: { abilityId: string; calls: number; cost: number }[];
  };
}

// 用量统计查询参数
export interface UsageStatsParams {
  abilityId?: string;
  startDate: string;
  endDate: string;
  granularity: 'day' | 'week' | 'month';
}

// 用量统计响应
export interface UsageStatsResponse {
  dates: string[];
  datasets: {
    label: string;
    data: number[];
    color: string;
  }[];
  summary: {
    totalCalls: number;
    totalCost: number;
    avgDaily: number;
  };
}

// 计划列表响应
export interface PlanListResponse {
  plans: {
    id: string;
    name: string;
    level: 'free' | 'pro' | 'enterprise';
    price: number;
    period: 'month' | 'year';
    features: string[];
    quotas: { abilityId: string; limit: number; unit: string }[];
    isPopular?: boolean;
  }[];
}

// 购买请求
export interface PurchasePlanRequest {
  planId: string;
  period: 'month' | 'year';
  couponCode?: string;
}

// 购买响应
export interface PurchasePlanResponse {
  success: boolean;
  orderId: string;
  paymentUrl?: string;
  activatedAt: string;
  expiredAt: string;
}

// 账单项
export interface InvoiceItem {
  id: string;
  period: string;
  amount: number;
  status: 'pending' | 'paid' | 'overdue';
  paidAt?: string;
  pdfUrl?: string;
  items: { description: string; amount: number }[];
}

// 优惠券
export interface Coupon {
  id: string;
  code: string;
  type: 'percentage' | 'fixed';
  value: number;
  minOrderAmount?: number;
  validFrom: string;
  validUntil: string;
  isUsed: boolean;
  usedAt?: string;
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| subscriptions | ADD | `plan_level` | ENUM | 套餐级别 free/pro/enterprise |
| subscriptions | ADD | `expired_at` | TIMESTAMP | 过期时间 |
| invoices | MODIFY | `pdf_url` | VARCHAR(512) | 发票 PDF 下载链接 |
| coupons | NEW | — | — | 优惠券表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`BillingOverviewResponse`, `UsageStatsParams`, `PurchasePlanRequest`, `InvoiceItem`

## 6. 交互流程

### 6.1 主流程：购买套餐
```
[用户访问计费中心] ──► [展示计费概览]
   │
   ├──► [查看用量统计] ──► [展示趋势图表]
   │
   ├──► [点击「升级套餐」] ──► [展示计划对比]
   │                              │
   │                              ├──► [选择计划 + 周期]
   │                              │
   │                              ├──► [输入优惠券]
   │                              │
   │                              └──► [确认支付] ──► [支付成功] ──► [更新配额]
   │
   └──► [查看账单] ──► [下载发票]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 优惠券无效 | 过期/已使用/不匹配 | 拒绝应用 | 提示优惠券不可用原因 |
| 支付失败 | 余额不足/网关异常 | 保留订单，标记待支付 | 提示重试或更换支付方式 |
| 配额超限 | 用量超过计划限制 | 阻止调用 | 提示升级套餐 |
| 发票生成失败 | 服务异常 | 标记失败，可重试 | 提供「重新生成」按钮 |

### 6.3 边界情况
- 免费用户无账单历史：展示引导升级的空态
- 用量为 0：图表展示空态，不显示 0 值折线
- 套餐即将过期（7 天内）：顶部 Banner 提醒续费
- 优惠券即将过期：列表中标记「即将过期」标签

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Card | 概览卡片容器 | 是 |
| Button | 购买/升级按钮 | 是 |
| Tag | 计划级别标签 | 是 |
| Table | 账单列表 | 是 |
| Tabs | 计费中心标签切换 | 是 |
| Progress | 配额进度条 | 是 |
| Empty | 无账单空态 | 是 |
| Alert | 过期提醒 | 是 |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PriceDisplay | 价格展示 | 是 |
| PlanCard | 计划卡片 | 是 |
| CouponTag | 优惠券标签 | 是 |
| QuotaBar | 配额进度条 | 是 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| BillingSummary | 计费概览面板 | 是 |
| UsageTrend | 用量趋势图表 | 是 |
| InvoiceTable | 账单列表 | 是 |
| PlanComparison | 计划对比表格 | 是 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| ListPageShell | 账单列表页骨架 |
| DetailPageShell | 账单详情页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| PurchaseFlowModal | blocks | 购买流程弹窗（选择计划 → 优惠券 → 支付） |
| QuotaDashboard | blocks | 配额监控大盘 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 计费概览查询
- **Method**: GET
- **Path**: `/portal-api/v1/billing/overview`
- **响应类型**: `BillingOverviewResponse`
- **权限**: 用户（已登录）

#### 用量统计查询
- **Method**: GET
- **Path**: `/portal-api/v1/billing/usage`
- **请求类型**: `UsageStatsParams`
- **响应类型**: `UsageStatsResponse`
- **权限**: 用户（已登录）

#### 计划列表查询
- **Method**: GET
- **Path**: `/portal-api/v1/billing/plans`
- **响应类型**: `PlanListResponse`
- **权限**: 公开（未登录也可查看）

#### 购买套餐
- **Method**: POST
- **Path**: `/portal-api/v1/billing/purchase`
- **请求类型**: `PurchasePlanRequest`
- **响应类型**: `PurchasePlanResponse`
- **权限**: 用户（已登录）

#### 账单列表查询
- **Method**: GET
- **Path**: `/portal-api/v1/billing/invoices`
- **权限**: 用户（已登录）

#### 优惠券列表查询
- **Method**: GET
- **Path**: `/portal-api/v1/billing/coupons`
- **响应类型**: `Coupon[]`
- **权限**: 用户（已登录）

#### 应用优惠券
- **Method**: POST
- **Path**: `/portal-api/v1/billing/coupons/apply`
- **请求类型**: `{ code: string; planId: string }`
- **权限**: 用户（已登录）

### 8.2 修改接口
| 接口 | 变更内容 | 兼容性 |
|------|----------|--------|
| `/portal-api/v1/subscription/*` | 返回增加 `plan_level` 和 `expired_at` | 向前兼容 |

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.billing.overview`, `userApi.billing.usage`, `userApi.billing.purchase`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 计费概览正确展示当前计划、配额和用量
- [ ] 用量趋势图表支持按日/周/月粒度切换
- [ ] 计划对比表格清晰展示各版本差异
- [ ] 购买流程完整：选择计划 → 应用优惠券 → 确认 → 支付成功
- [ ] 优惠券实时校验，无效券给出明确提示
- [ ] 账单列表支持按状态筛选和下载 PDF
- [ ] 套餐过期前 7 天展示顶部提醒 Banner
- [ ] 配额使用超过 80% 时进度条变为橙色，超过 100% 变为红色

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 图表组件使用已封装的 UsageChart 业务组件
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 金额计算使用整数分存储，避免浮点误差
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 计费概览加载 < 1s
- [ ] 图表数据加载 < 1.5s
- [ ] 计划列表缓存 5 分钟（配置数据不常变化）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 多栏布局正常（概览 + 图表 + 详情）
- [ ] 用户端移动端卡片垂直堆叠，图表支持横向滚动

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [ ] **否** — 全新功能，不影响现有页面
- [x] **是** — 影响范围：
  - 影响页面：原 `/subscription` 页面增加新字段展示
  - 影响用户：全部用户（仅展示层变更，无功能移除）
  - 回滚方案：前端字段兼容，后端接口增加可选字段

### 10.2 是否依赖其他任务先完成 ⭐
- [ ] **否** — 可独立开发
- [x] **是** — 依赖任务：
  - [x] `FEATURE-202604-001` — 能力中心 — 已完成
  - [x] `FEATURE-202604-004` — 任务平台 — 已完成
  - [ ] `FEATURE-202604-008` — 后端计费引擎升级 — 开发中

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 支付接口联调延迟 | 中 | 高 | 使用 Mock 支付接口并行开发 |
| 金额精度问题 | 低 | 高 | 统一使用分作为最小单位，展示时转换 |
| 图表性能（大数据量） | 中 | 中 | 数据聚合到日级别，超过 90 天按周聚合 |

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

- 本任务为计费系统增强，不涉及核心架构变更
- 支付流程使用现有支付网关接口，本任务聚焦前端体验优化
- 团队/企业版配额管理为后续独立任务

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
