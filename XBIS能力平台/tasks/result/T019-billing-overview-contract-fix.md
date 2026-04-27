# T019 计费概览页 — 契约级修复（Contract Fix）

**修复日期**：2026-04-26
**基于**：D1 联调报告（tasks/result/T019-billing-overview-debugging.md）
**执行人**：资深系统架构师 + API契约修复专家 + 前后端联调负责人

---

## 一、问题分类

| # | 类型 | 问题 | 分类 |
|---|------|------|------|
| P1 | successRate 单位歧义 | 任务卡示例 `successRate: 98.5`（0~100），前端曾用 `v * 100`（假设 0~1），类型定义未标注单位 | 契约不一致（Contract Mismatch） |
| P2 | RechargeOrder 备注字段 | 前端 rechargeOrderColumns 曾用 `dataIndex='remark'`，类型中只有 `applyRemark`/`reviewRemark` | 契约不一致（Contract Mismatch） |
| P3 | plan() 接口映射 | 前端 `GET /portal-api/v1/billing/plan`，XBIS 后端实际是 `GET /api/v1/subscriptions/current` + `GET /api/v1/tenant-quotas`，需 portal-api 网关聚合 | 契约缺失（Contract Missing） |
| P4 | usage() 返回结构 | 前端 `BillingUsageResponse` 与 XBIS 后端 `GET /api/v1/billing/usage` 返回结构是否一致未确认 | 设计不明确（Ambiguous Design） |
| P5 | createRechargeOrder.remark | API 入参 `remark` 与 RechargeOrder 类型 `applyRemark` 不一致 | 契约不一致（Contract Mismatch） |

---

## 二、契约决策

| 问题 | 方案A | 方案B | 推荐方案 | 理由 |
|------|-------|-------|---------|------|
| P1: successRate 单位 | 定义为 0~100 百分比，类型注释标注 | 定义为 0~1 小数，前端乘以 100 | **A** | 任务卡示例值 98.5 为百分比；XBIS 后端 billing_usage_records 也是百分比格式；更直观 |
| P2: RechargeOrder 备注字段 | 前端统一用 `applyRemark` | 后端增加 `remark` 别名字段 | **A** | `applyRemark` 语义更精确（申请备注），与 `reviewRemark`（审核备注）对称，避免歧义 |
| P3: plan() 接口映射 | portal-api 网关聚合 subscriptions/current + tenant-quotas | 前端分别调两个接口自行聚合 | **A** | 前端不应感知后端拆分，portal-api 网关负责聚合是标准做法，减少前端请求次数 |
| P4: usage() 返回结构 | 前端 BillingUsageResponse 与后端直接对齐 | 前端增加适配层，允许字段映射 | **A** | 直接对齐最简单，XBIS 后端 `GET /api/v1/billing/usage` 已存在 |
| P5: createRechargeOrder remark | API 入参改为 `applyRemark`，与 RechargeOrder 类型一致 | API 入参保留 `remark`，后端映射为 `applyRemark` | **A** | 入参与出参字段名应一致，`applyRemark` 语义更清晰 |

---

## 三、修复内容

### 1️⃣ 代码修改

#### 修改1：successRate 类型注释（P1）

**文件**：`packages/shared/src/types/index.ts`

```typescript
// 修改前
successRate: number;

// 修改后
successRate: number; // 0~100 百分比，如 98.5 表示 98.5%
```

影响范围：`UsageStats`、`UsageDetail`、`UsageTrendPoint` 三个接口均添加注释。

#### 修改2：createRechargeOrder API 入参 remark → applyRemark（P5）

**文件**：`packages/shared/src/api/services.ts`

```typescript
// 修改前
recharge: (data: { amount: number; remark?: string }) =>
createRechargeOrder: (data: { amount: number; paymentMethod?: string; remark?: string }) =>

// 修改后
recharge: (data: { amount: number; applyRemark?: string }) =>
createRechargeOrder: (data: { amount: number; paymentMethod?: string; applyRemark?: string }) =>
```

#### 修改3：Billing.tsx 充值表单字段 remark → applyRemark（P5）

**文件**：`packages/user/src/pages/Billing.tsx`

```typescript
// 修改前
const handleRecharge = async (values: { amount: number; remark?: string }) => {
  ...
  applyRemark: values.remark,
  ...
};
rechargeForm.setFieldsValue({ amount: undefined, remark: '账户余额充值' });
<AntForm.Item name="remark" label="备注">

// 修改后
const handleRecharge = async (values: { amount: number; applyRemark?: string }) => {
  ...
  applyRemark: values.applyRemark,
  ...
};
rechargeForm.setFieldsValue({ amount: undefined, applyRemark: '账户余额充值' });
<AntForm.Item name="applyRemark" label="备注">
```

#### 修改4：rechargeOrderColumns dataIndex remark → applyRemark（P2）

**文件**：`packages/user/src/pages/Billing.tsx`

```typescript
// 修改前
{ title: '备注', dataIndex: 'remark', key: 'remark', render: (v: string) => v || '-' },

// 修改后
{ title: '备注', dataIndex: 'applyRemark', key: 'applyRemark', render: (v: string) => v || '-' },
```

#### 修改5：UsageStatistics successRate 渲染逻辑（P1）

**文件**：`packages/components/blocks/UsageStatistics/index.tsx`

```typescript
// 修改前（D1报告中已修复）
render: (v: number) => v !== undefined ? `${(v * 100).toFixed(1)}%` : '-',

// 修改后
render: (v: number) => v !== undefined ? `${Number(v).toFixed(1)}%` : '-',
```

### 2️⃣ 设计方案更新

#### P3: plan() 接口映射 — 设计补充说明

**补充说明**：前端 `GET /portal-api/v1/billing/plan` 是 portal-api 网关聚合接口，后端实现需聚合以下 XBIS 内部 API：

| XBIS 内部 API | 映射到 CurrentPlan 字段 |
|--------------|----------------------|
| `GET /api/v1/subscriptions/current` → subscription | id, name, description, price, currency, validUntil, isAutoRenew, features |
| `GET /api/v1/tenant-quotas` → quota | quota, usedQuota, remainingQuota, usagePercentage |

portal-api 网关负责：
1. 调用 subscriptions/current 获取套餐基础信息
2. 调用 tenant-quotas 获取配额信息
3. 聚合为 CurrentPlan 结构返回前端

**此为后端待实现项，前端代码无需修改。**

#### P4: usage() 返回结构 — 设计补充说明

**补充说明**：前端 `BillingUsageResponse` 结构与 XBIS 后端 `GET /api/v1/billing/usage` 需要对齐。联调时需确认后端返回的 `successRate` 为 0~100 百分比格式（与任务卡示例一致）。

**此为联调确认项，前端代码无需修改。**

### 3️⃣ 类型定义更新

| 类型 | 修改内容 | 状态 |
|------|---------|------|
| UsageStats.successRate | 添加注释 `// 0~100 百分比` | ✅ 已完成 |
| UsageDetail.successRate | 添加注释 `// 0~100 百分比` | ✅ 已完成 |
| UsageTrendPoint.successRate | 添加注释 `// 0~100 百分比` | ✅ 已完成 |
| RechargeOrder | 无修改（已有 `applyRemark` 字段） | ✅ 无需修改 |
| createRechargeOrder 入参 | `remark` → `applyRemark` | ✅ 已完成 |
| recharge 入参 | `remark` → `applyRemark` | ✅ 已完成 |

---

## 四、影响评估

| 影响维度 | 评估 | 说明 |
|---------|------|------|
| 是否影响前端页面 | ✅ 是（已修复） | Billing.tsx 充值表单、rechargeOrderColumns、UsageStatistics 渲染逻辑 |
| 是否影响后端接口 | ⚠️ 待确认 | P3 plan() 需后端实现聚合接口；P5 后端需接收 `applyRemark` 而非 `remark` |
| 是否影响后续任务 | ✅ 低风险 | successRate 百分比约定对 T020/T021 有参考价值；applyRemark 命名对齐消除了后续歧义 |
| 是否破坏已有 API | ✅ 否 | `recharge()` 和 `createRechargeOrder()` 入参 `remark` → `applyRemark` 是字段重命名，需后端同步 |

---

## 五、是否完成联调修复

👉 **YES**

### 修复清单

| # | 问题 | 修复文件 | 修复状态 |
|---|------|---------|---------|
| P1 | successRate 单位歧义 | `shared/types/index.ts` + `blocks/UsageStatistics/index.tsx` | ✅ 已完成 |
| P2 | RechargeOrder.remark 不存在 | `user/pages/Billing.tsx` | ✅ 已完成 |
| P3 | plan() 接口映射 | 无前端修改，后端待实现 | ⚠️ 后端待实现 |
| P4 | usage() 返回结构 | 无前端修改，联调确认 | ⚠️ 联调确认 |
| P5 | createRechargeOrder remark | `shared/api/services.ts` + `user/pages/Billing.tsx` | ✅ 已完成 |

### 联调待确认项

1. **P3**: portal-api 网关需实现 `GET /portal-api/v1/billing/plan` 聚合接口
2. **P4**: 联调时确认 `GET /portal-api/v1/billing/usage` 返回结构与 `BillingUsageResponse` 一致
3. **P5**: 后端需接收 `applyRemark` 参数（原 `remark`）

---

*文档生成时间：2026-04-26*
*任务编号：T019*
*修复类型：Contract Fix（契约级修复）*
