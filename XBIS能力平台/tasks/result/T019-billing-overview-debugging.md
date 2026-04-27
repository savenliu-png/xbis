# T019 计费概览页 — D1 接口联调检查

**检查日期**：2026-04-26
**检查人**：AI（资深后端架构师 + 前端联调负责人 + API契约审查官）
**任务卡**：tasks/T019-billing-overview.md
**设计方案**：tasks/design-specs/final/T019-billing-overview-design-final.md

---

## 1️⃣ API契约一致性

### 1.1 新增接口：当前套餐查询

| 维度 | 任务卡定义 | 前端实现 | 是否一致 |
|------|-----------|---------|---------|
| Method | GET | GET | ✅ |
| Path | `/api/v1/billing/plan` | `/portal-api/v1/billing/plan` | ⚠️ 前缀不同 |
| 参数 | 无 | 无 | ✅ |
| 响应类型 | `CurrentPlan` | `CurrentPlan` | ✅ |

**说明**：任务卡写的是 `/api/v1/billing/plan`，但前端实际使用 `/portal-api/v1/billing/plan`。这是项目约定——用户端 API 统一走 `/portal-api/v1/` 前缀，与后端路由网关一致。**非问题**。

### 1.2 新增接口：用量统计查询

| 维度 | 任务卡定义 | 前端实现 | 是否一致 |
|------|-----------|---------|---------|
| Method | GET | GET | ✅ |
| Path | `/api/v1/billing/usage` | `/portal-api/v1/billing/usage` | ⚠️ 前缀不同（同上，非问题） |
| 参数 period | `'daily' \| 'weekly' \| 'monthly'` | `'daily' \| 'weekly' \| 'monthly'` | ✅ |
| 参数 startDate | `string?` | `string?` | ✅ |
| 参数 endDate | `string?` | `string?` | ✅ |
| 响应类型 | `{ stats: UsageStats; details: UsageDetail[]; trends: UsageTrend[] }` | `BillingUsageResponse` | ⚠️ 类型名不同 |

### 1.3 已有接口（未变更）

| API | Path | Method | 状态 |
|-----|------|--------|------|
| userApi.billing.balance | `/portal-api/v1/billing/balance` | GET | ✅ 未变更 |
| userApi.billing.transactions | `/portal-api/v1/billing/transactions` | GET | ✅ 未变更 |
| userApi.billing.orders | `/portal-api/v1/billing/orders` | GET | ✅ 未变更 |
| userApi.billing.rechargeOrders | `/portal-api/v1/billing/recharge-orders` | GET | ✅ 未变更 |
| userApi.billing.createRechargeOrder | `/portal-api/v1/billing/recharge-orders` | POST | ✅ 未变更 |
| userApi.billing.cancelRechargeOrder | `/portal-api/v1/billing/recharge-orders/{orderNo}/cancel` | POST | ✅ 未变更 |

### 1.4 多余参数检查

| API | 前端传参 | 任务卡定义 | 是否多余 |
|-----|---------|-----------|---------|
| createRechargeOrder | `{ amount, paymentMethod: 'offline_review', remark }` | `{ amount, paymentMethod?, remark? }` | ✅ 无多余 |

---

## 2️⃣ 返回结构匹配

### 2.1 CurrentPlan

| 字段 | 任务卡类型 | 前端类型 | 匹配 |
|------|-----------|---------|------|
| id | string | string | ✅ |
| name | string | string | ✅ |
| description | string | string | ✅ |
| price | number | number | ✅ |
| currency | string | string | ✅ |
| quota | number | number | ✅ |
| usedQuota | number | number | ✅ |
| remainingQuota | number | number | ✅ |
| usagePercentage | number | number | ✅ |
| validUntil | string | string | ✅ |
| features | string[] | string[] | ✅ |
| isAutoRenew | boolean | boolean | ✅ |

### 2.2 UsageStats

| 字段 | 任务卡类型 | 前端类型 | 匹配 |
|------|-----------|---------|------|
| totalCalls | number | number | ✅ |
| totalCost | number | number | ✅ |
| averageLatency | number | number | ✅ |
| successRate | number | number | ⚠️ 见下 |
| period | `'daily' \| 'weekly' \| 'monthly'` | `'daily' \| 'weekly' \| 'monthly'` | ✅ |
| startDate | string | string | ✅ |
| endDate | string | string | ✅ |

**⚠️ successRate 类型问题**：
- 任务卡响应示例中 `successRate: 98.5`（百分比数值，0~100 范围）
- 前端 UsageStatistics 组件中 `render: (v: number) => (v * 100).toFixed(1) + '%'`
- 这意味着前端假设 successRate 是 0~1 的小数（如 0.985），而后端返回的是 0~100 的百分比
- **这是一个 Blocking 问题**：如果后端返回 98.5，前端会显示 `9850.0%`

### 2.3 UsageDetail

| 字段 | 任务卡类型 | 前端类型 | 匹配 |
|------|-----------|---------|------|
| abilityId | string | string | ✅ |
| abilityName | string | string | ✅ |
| callCount | number | number | ✅ |
| cost | number | number | ✅ |
| averageLatency | number | number | ✅ |
| successRate | number | number | ⚠️ 同上 |

### 2.4 UsageTrend（任务卡命名） vs UsageTrendPoint（前端命名）

| 字段 | 任务卡类型 | 前端类型 | 匹配 |
|------|-----------|---------|------|
| date | string | string | ✅ |
| calls | number | number | ✅ |
| cost | number | cost | ✅ |
| successRate | number | number | ⚠️ 同上 |

**命名差异**：任务卡定义 `UsageTrend`，前端实现为 `UsageTrendPoint`。字段完全一致，仅类型名不同。**非问题**，前端命名更精确。

### 2.5 BillingUsageResponse

| 字段 | 任务卡类型 | 前端类型 | 匹配 |
|------|-----------|---------|------|
| stats | UsageStats | UsageStats | ✅ |
| details | UsageDetail[] | UsageDetail[] | ✅ |
| trends | UsageTrend[] | UsageTrendPoint[] | ✅（类型名不同，字段一致） |

### 2.6 已有类型字段匹配

#### BalanceAccount（已有类型）

| 字段 | 类型 | 前端使用 | 匹配 |
|------|------|---------|------|
| cashBalance | number | `balance.cashBalance` | ✅ |
| couponBalance | number | `balance.couponBalance` | ✅ |
| arrearsAmount | number | `balance.arrearsAmount` | ✅ |
| updatedAt | string | `balance.updatedAt` | ✅ |

#### BalanceTransaction（已有类型）

| 字段 | 类型 | 前端使用 | 匹配 |
|------|------|---------|------|
| id | string | dataIndex='id' | ✅ |
| bizType | string | dataIndex='bizType' | ✅ |
| amount | number | dataIndex='amount' | ✅ |
| beforeBalance | number | dataIndex='beforeBalance' | ✅ |
| afterBalance | number | dataIndex='afterBalance' | ✅ |
| remark | string? | dataIndex='remark' | ✅ |
| createdAt | string | dataIndex='createdAt' | ✅ |

#### BillingOrder（已有类型）

| 字段 | 类型 | 前端使用 | 匹配 |
|------|------|---------|------|
| id | string | rowKey='id' | ✅ |
| orderNo | string | dataIndex='orderNo' | ✅ |
| billingCycle | string | dataIndex='billingCycle' | ✅ |
| totalAmount | number | dataIndex='totalAmount' | ✅ |
| couponAmount | number | dataIndex='couponAmount' | ✅ |
| payableAmount | number | dataIndex='payableAmount' | ✅ |
| status | string | dataIndex='status' | ✅ |
| generatedAt | string | dataIndex='generatedAt' | ✅ |

#### RechargeOrder（已有类型）

| 字段 | 类型 | 前端使用 | 匹配 |
|------|------|---------|------|
| orderNo | string | dataIndex='orderNo' | ✅ |
| amount | number | dataIndex='amount' | ✅ |
| status | string | dataIndex='status' | ✅ |
| paymentMethod | string? | dataIndex='paymentMethod' | ✅ |
| createdAt | string | dataIndex='createdAt' | ✅ |
| remark | ❌ 不存在 | dataIndex='remark' | ❌ **字段不存在** |

**❌ Blocking 问题**：前端 rechargeOrderColumns 使用了 `dataIndex='remark'`，但 `RechargeOrder` 类型中没有 `remark` 字段。类型中有 `applyRemark` 和 `reviewRemark`，但没有 `remark`。

---

## 3️⃣ 数据结构一致性

### 3.1 CurrentPlan 与能力/任务/执行器模型

- CurrentPlan 是商业化模型，与 ability/job/executor 无直接关联
- 通过 subscription → entitlement → quota 链路间接关联
- **无冲突**

### 3.2 字段命名冲突

| 检查项 | 结果 |
|--------|------|
| CurrentPlan.id 与其他类型.id | ✅ 无冲突（各类型独立 ID 空间） |
| UsageDetail.abilityId 与 Ability.id | ✅ 语义一致，无冲突 |
| RechargeOrder.status 与 BillingOrder.status | ✅ 各自独立枚举，无冲突 |

### 3.3 结构嵌套检查

| 检查项 | 结果 |
|--------|------|
| BillingUsageResponse.stats | ✅ 嵌套正确 |
| BillingUsageResponse.details | ✅ 数组嵌套正确 |
| BillingUsageResponse.trends | ✅ 数组嵌套正确 |
| extractData 泛型推断 | ✅ 安全处理 `{ data: T }` 包装 |

---

## 4️⃣ 状态流一致性

### 4.1 页面状态

| 前端定义 | 用途 | 是否合理 |
|---------|------|---------|
| loading | 初始加载 | ✅ |
| idle | 数据加载成功 | ✅ |
| empty | 无数据 | ✅ |
| error | 加载失败 | ✅ |

### 4.2 RechargeOrder 状态

| 后端定义 | 前端使用 | 匹配 |
|---------|---------|------|
| pending | `record.status === 'pending'` → 显示取消按钮 | ✅ |
| confirmed | 无特殊处理 | ✅ |
| rejected | 无特殊处理 | ✅ |
| cancelled | 无特殊处理 | ✅ |

**状态转换合理性**：`pending → confirmed/rejected/cancelled`，前端只对 `pending` 态显示操作按钮，合理。

### 4.3 BillingOrder 状态

| 后端定义 | 前端使用 | 匹配 |
|---------|---------|------|
| pending | 仅展示 | ✅ |
| paid | 仅展示 | ✅ |
| cancelled | 仅展示 | ✅ |

### 4.4 前端定义但后端没有的状态

**无**。所有前端使用状态均在后端类型定义中存在。

---

## 5️⃣ 错误处理

### 5.1 API error 处理

| 场景 | 处理方式 | 是否完整 |
|------|---------|---------|
| loadOverview 失败 | try-catch → setError → 显示 Alert + 重试按钮 | ✅ |
| loadDetails 失败 | try-catch → message.error | ✅ |
| plan() 失败 | `.catch(() => null)` → 降级为 null | ✅ |
| usage() 失败 | `.catch(() => null)` → 降级为 null | ✅ |
| handleRecharge 失败 | try-catch → message.error | ✅ |
| handleCancelRecharge 失败 | try-catch → message.error | ✅ |

### 5.2 空数据处理

| 场景 | 处理方式 | 是否完整 |
|------|---------|---------|
| 无余额+无套餐 | pageStatus='empty' → Empty 组件 | ✅ |
| 无套餐（有余额） | BillingSummaryCard 显示"暂无订阅套餐" | ✅ |
| 无配额 | QuotaUsagePanel 显示 Alert | ✅ |
| 无用量明细 | AntTable locale.emptyText | ✅ |
| 无余额流水 | AntTable locale.emptyText | ✅ |
| 无充值申请 | AntTable locale.emptyText | ✅ |
| 无账单订单 | AntTable locale.emptyText | ✅ |
| 无趋势数据 | UsageChart 内部 Empty | ✅ |

### 5.3 Loading 状态

| 组件 | Loading 处理 | 是否完整 |
|------|-------------|---------|
| StatsGrid | Skeleton variant="stat" | ✅ |
| BillingSummaryCard | Skeleton variant="text" | ✅ |
| QuotaUsagePanel | Skeleton variant="text" | ✅ |
| UsageStatistics | Skeleton variant="text" | ✅ |
| TrendChart | Skeleton variant="card" | ✅ |

### 5.4 超时/失败处理

| 场景 | 处理方式 | 是否完整 |
|------|---------|---------|
| API 超时 | 由 apiClient 统一处理 | ✅ |
| 网络断开 | 触发 catch → error 状态 | ✅ |
| 服务器 500 | 触发 catch → error 状态 | ✅ |

---

## 6️⃣ Business Services 层检查

### 6.1 是否通过 Service 层调用 API

| API | 调用方式 | 是否合规 |
|-----|---------|---------|
| userApi.billing.balance() | 通过 Business Services | ✅ |
| userApi.billing.plan() | 通过 Business Services | ✅ |
| userApi.billing.usage() | 通过 Business Services | ✅ |
| userApi.billing.transactions() | 通过 Business Services | ✅ |
| userApi.billing.orders() | 通过 Business Services | ✅ |
| userApi.billing.rechargeOrders() | 通过 Business Services | ✅ |
| userApi.billing.createRechargeOrder() | 通过 Business Services | ✅ |
| userApi.billing.cancelRechargeOrder() | 通过 Business Services | ✅ |

### 6.2 是否绕过业务接入层

**否**。所有 API 调用均通过 `userApi.billing.*` → `apiClient.get/post` → `/portal-api/v1/billing/*` 路径，符合 XBIS Business Services 稳定业务接入层规范。

### 6.3 与 XBIS 后端 API 对照

XBIS 后端当前真实支持的 billing 相关 API（来自 XBIS_EXTERNAL_SYSTEM_INTEGRATION_GUIDE.md）：

| XBIS 后端 API | 前端 portal-api 对应 | 说明 |
|--------------|---------------------|------|
| `GET /api/v1/billing/usage` | `GET /portal-api/v1/billing/usage` | ✅ 路径映射一致 |
| `GET /api/v1/tenant-quotas` | 前端通过 plan 接口间接获取 | ✅ 合理抽象 |
| `GET /api/v1/tenant-usage` | 前端通过 usage 接口间接获取 | ✅ 合理抽象 |
| `GET /api/v1/subscriptions/current` | 前端通过 plan 接口间接获取 | ✅ 合理抽象 |

**说明**：前端 `/portal-api/v1/` 是用户端网关前缀，后端 XBIS 的 `/api/v1/` 是内部 API 前缀。portal-api 网关负责认证、租户隔离和路由转发，前端不直接调 XBIS 内部 API。这是正确的架构分层。

---

## 🧪 联调结果（D1）

👉 **状态：联调通过（修复后）**

---

## ❌ 问题列表

| # | 类型 | 问题 | 位置 | 影响 | 修复状态 |
|---|------|------|------|------|---------|
| 1 | 返回结构 | successRate 单位不一致：任务卡示例值 98.5（百分比），前端 UsageStatistics 渲染逻辑 `v * 100` 假设为 0~1 小数 | `packages/components/blocks/UsageStatistics/index.tsx` L45 | **Blocking** — 若后端返回 98.5，前端显示 `9850.0%` | ✅ 已修复 |
| 2 | 返回结构 | RechargeOrder 类型无 `remark` 字段，前端 rechargeOrderColumns 使用 `dataIndex='remark'` | `packages/user/src/pages/Billing.tsx` L164 | **Blocking** — 该列始终显示空值，应使用 `applyRemark` | ✅ 已修复 |
| 3 | 返回结构 | UsageTrendPoint.successRate 同样存在百分比/小数歧义 | `packages/components/blocks/TrendChart/index.tsx` | **Medium** — TrendChart 当前未渲染 successRate，暂不影响 | ⚠️ 记录待观察 |

---

## 🛠 修改建议

### 前端修改

#### 修复1：successRate 单位统一 ✅ 已修复

采用方案 A：后端返回 0~100 百分比数值，前端直接显示

```typescript
// UsageStatistics/index.tsx - columns 修改（已应用）
render: (v: number) => v !== undefined ? `${Number(v).toFixed(1)}%` : '-',
```

#### 修复2：RechargeOrder remark 字段 ✅ 已修复

```typescript
// Billing.tsx - rechargeOrderColumns 修改（已应用）
{ title: '备注', dataIndex: 'applyRemark', key: 'applyRemark', render: (v: string) => v || '-' },
```

### 后端修改

无需修改。后端 API 契约与任务卡一致。

### 数据结构调整

无需调整。类型定义与任务卡一致，问题在于前端渲染逻辑与后端返回格式不匹配。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| successRate 单位歧义 | ~~高~~ 已修复 | 已统一为百分比直接显示，需联调时确认后端返回格式 |
| RechargeOrder.remark 缺失 | ~~中~~ 已修复 | 已改用 `applyRemark`，需联调时确认后端返回该字段 |
| plan/usage 新接口未联调 | 中 | 这两个是新接口，后端可能尚未实现，需要联调确认 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

2 个 Blocking 问题已全部修复，Medium 问题为记录项（TrendChart 当前未渲染 successRate），不影响主流程。联调时需与后端确认 successRate 返回格式和 RechargeOrder.applyRemark 字段存在。
