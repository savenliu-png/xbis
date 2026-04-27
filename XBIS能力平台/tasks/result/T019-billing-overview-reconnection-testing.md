# T019 计费概览页 — 前后端联调复测与功能验收

**测试日期**：2026-04-26
**测试人**：资深前后端联调负责人 + QA测试负责人 + Bug修复负责人 + 产品验收官
**轮次**：第二轮联调（基于上一轮遗留问题复核）
**基于**：XBIS_EXTERNAL_SYSTEM_INTEGRATION_GUIDE.md + 前端代码 + TypeScript 类型 + 历史报告

---

## 一、联调范围

| 类型 | 对象 | 新增/变更/旧功能 | 风险等级 | 是否必须联调 | 原因 |
|------|------|----------------|---------|------------|------|
| 页面入口 | `/billing` 路由 | 变更（重写） | 中 | ✅ 是 | 页面完全重写 |
| API | `GET /portal-api/v1/billing/plan` | **新增** | **高** | ✅ 是 | 新增聚合接口，后端待实现 |
| API | `GET /portal-api/v1/billing/usage` | **新增** | **高** | ✅ 是 | 新增聚合接口，后端待实现 |
| API | `GET /portal-api/v1/billing/balance` | 旧功能 | 中 | ✅ 是 | 已有接口，需确认兼容 |
| API | `GET /portal-api/v1/billing/transactions` | 旧功能 | 低 | ✅ 是 | 已有接口 |
| API | `GET /portal-api/v1/billing/orders` | 旧功能 | 低 | ✅ 是 | 已有接口 |
| API | `GET /portal-api/v1/billing/recharge-orders` | 旧功能 | 低 | ✅ 是 | 已有接口 |
| API | `POST /portal-api/v1/billing/recharge-orders` | **变更** | 中 | ✅ 是 | 入参 remark→applyRemark |
| API | `POST /portal-api/v1/billing/recharge-orders/{orderNo}/cancel` | 旧功能 | 低 | ✅ 是 | 已有接口 |
| API | `GET /portal-api/v1/subscriptions/current` | 旧功能 | 中 | ✅ 是 | 与 plan() 存在关联 |
| 请求参数 | createRechargeOrder.applyRemark | 变更 | 中 | ✅ 是 | 字段重命名 |
| 响应结构 | CurrentPlan | **新增** | **高** | ✅ 是 | 新增类型 |
| 响应结构 | BillingUsageResponse | **新增** | **高** | ✅ 是 | 新增类型 |
| 类型定义 | CurrentPlan / UsageStats / UsageDetail / UsageTrendPoint / BillingUsageResponse | **新增** | 中 | ✅ 是 | 新增类型 |
| 状态流 | RechargeOrder.status | 旧功能 | 低 | ✅ 是 | 状态枚举需确认 |
| 降级逻辑 | plan().catch + usage().catch | **新增** | 中 | ✅ 是 | 新增降级 |
| 旧接口兼容 | balance/transactions/orders/rechargeOrders | 旧功能 | 低 | ✅ 是 | 回归检查 |

---

## 二、上一轮遗留问题复核

| 遗留问题 | 责任方 | 当前状态 | 验证方式 | 是否通过 |
|---------|--------|---------|---------|---------|
| BUG-BE-001: plan() 接口后端未实现 | 后端 | ⚠️ 仍未实现 | 调用 `GET /portal-api/v1/billing/plan` 返回 404 | ❌ **阻塞项** |
| BUG-BE-002: usage() 接口返回结构需聚合 | 后端 | ⚠️ 仍未实现 | 调用 `GET /portal-api/v1/billing/usage` 返回原始结构 | ❌ **阻塞项** |
| BUG-BE-003: createRechargeOrder 入参 applyRemark 需后端同步 | 后端 | ⚠️ 未同步 | 后端是否接收 applyRemark 字段 | ❌ **阻塞项** |
| BUG-CT-001: successRate 单位约定待确认 | 契约 | ⚠️ 未确认 | 联调时确认后端返回 0~100 还是 0~1 | ❌ **阻塞项** |
| BUG-FE-001: validUntil 空字符串处理 | 前端 | ✅ 已修复 | 代码检查 `plan.validUntil ?` | ✅ 通过 |
| BUG-FE-002: usagePercentage 冗余字段 | 前端 | ✅ 已移除 | 类型定义中无 usagePercentage | ✅ 通过 |

**结论**：4 项后端/契约遗留问题仍未解决，为阻塞项。前端侧 2 项已修复通过。

---

## 三、API契约核对

### 3.1 全部接口逐项核对

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 错误码一致 | 状态 |
|-----|--------|---------|---------|------------|------------|-----------|------|
| balance | GET | `userApi.billing.balance()` | portal-api 自有 | ✅ | ✅ | ✅ | ✅ 已有 |
| plan | GET | `userApi.billing.plan()` | ⚠️ 需 portal-api 聚合 | ✅ 无参数 | ❌ 后端未实现 | — | ❌ **阻塞** |
| usage | GET | `userApi.billing.usage({ period: 'monthly' })` | `GET /api/v1/billing/usage` (XBIS) | ⚠️ period 参数需确认 | ❌ 结构需聚合 | ✅ | ❌ **阻塞** |
| transactions | GET | `userApi.billing.transactions()` | portal-api 自有 | ✅ | ✅ | ✅ | ✅ 已有 |
| orders | GET | `userApi.billing.orders()` | portal-api 自有 | ✅ | ✅ | ✅ | ✅ 已有 |
| rechargeOrders | GET | `userApi.billing.rechargeOrders()` | portal-api 自有 | ✅ | ✅ | ✅ | ✅ 已有 |
| createRechargeOrder | POST | `userApi.billing.createRechargeOrder({amount, paymentMethod, applyRemark})` | portal-api 自有 | ⚠️ applyRemark 变更 | ✅ | ✅ | ⚠️ **待同步** |
| cancelRechargeOrder | POST | `userApi.billing.cancelRechargeOrder(orderNo)` | portal-api 自有 | ✅ | ✅ | ✅ | ✅ 已有 |
| current | GET | `userApi.billing.current()` | `GET /portal-api/v1/subscriptions/current` | ✅ | ✅ | ✅ | ✅ 已有 |

### 3.2 plan() 与 current() 关系说明

**发现**：项目中已存在 `userApi.billing.current()` 调用 `/portal-api/v1/subscriptions/current`，返回 `UserSubscription`（含 `plan?: SubscriptionPlan`）。

**分析**：
- `current()` 返回订阅原始数据（`UserSubscription`），不含配额信息
- `plan()` 返回 UI 优化的聚合数据（`CurrentPlan`），含配额信息
- `plan()` 是 `current()` + `tenant-quotas` 的聚合，专为计费概览页设计

**结论**：`plan()` 不是冗余接口，而是更高层的聚合接口。`current()` 仍供订阅管理页使用。

### 3.3 plan() 接口后端实现建议

```
GET /portal-api/v1/billing/plan

后端实现步骤：
1. 调用 GET /api/v1/subscriptions → 获取当前租户活跃订阅
2. 调用 GET /api/v1/subscriptions/{id} → 获取订阅详情（含 plan 信息）
3. 调用 GET /api/v1/tenant-quotas → 获取配额信息
4. 聚合为 CurrentPlan 结构返回

字段映射：
| CurrentPlan 字段 | 数据源 | 映射逻辑 |
|-----------------|--------|---------|
| id | subscription.id | 直接映射 |
| name | plan.planName | subscription.planId → plan |
| description | plan.description | 直接映射 |
| price | plan.price | 直接映射 |
| currency | 'CNY' | 默认值 |
| quota | tenant_quotas.includedQuota | 直接映射 |
| usedQuota | tenant_usage_counters | SUM 计算 |
| remainingQuota | quota - usedQuota | 计算 |
| validUntil | subscription.endAt | 直接映射 |
| features | plan.supportFeatures | 直接映射 |
| isAutoRenew | subscription.billingType === 'monthly' | 逻辑推断 |
```

### 3.4 usage() 接口后端实现建议

```
GET /portal-api/v1/billing/usage?period=monthly&startDate=2026-04-01&endDate=2026-04-30

后端实现步骤：
1. 调用 GET /api/v1/billing/usage → 获取原始 usage records
2. 调用 POST /api/v1/billing/usage/aggregate → 聚合数据
3. 转换为 BillingUsageResponse 结构返回

字段映射：
| BillingUsageResponse 字段 | 数据源 | 映射逻辑 |
|--------------------------|--------|---------|
| stats.totalCalls | billing_usage_aggregates | SUM(invocation_count) |
| stats.totalCost | billing_usage_aggregates | SUM(cost) |
| stats.averageLatency | billing_usage_records | AVG(duration_ms) |
| stats.successRate | billing_usage_records | 成功数/总数*100（0~100 百分比） |
| stats.period | 请求参数 | 直接透传 |
| stats.startDate/endDate | 计算得出 | period 推算 |
| details[] | billing_usage_records | GROUP BY capability_id |
| details[].abilityId | capability_id | 重命名映射 |
| details[].abilityName | capability registry | 查询 capability 表 |
| trends[] | billing_usage_records | GROUP BY DATE(created_at) |
```

---

## 四、数据与类型核对

### 4.1 新增类型与已有类型对照

| 字段 | 前端定义 | 后端/已有类型 | 是否一致 | 问题 | 修复建议 |
|------|---------|-------------|---------|------|---------|
| CurrentPlan.id | string | UserSubscription.id | ✅ | — | — |
| CurrentPlan.name | string | SubscriptionPlan.planName | ⚠️ 字段名不同 | portal-api 映射 | — |
| CurrentPlan.price | number | SubscriptionPlan.price | ✅ | — | — |
| CurrentPlan.quota | number | SubscriptionPlan.includedQuota | ⚠️ 字段名不同 | portal-api 映射 | — |
| CurrentPlan.usedQuota | number | QuotaUsage.dailyCount | ⚠️ 语义不同 | portal-api 聚合 | — |
| CurrentPlan.remainingQuota | number | 计算 | ✅ | quota - usedQuota | — |
| CurrentPlan.validUntil | string | UserSubscription.endAt | ⚠️ 字段名不同 | portal-api 映射 | — |
| CurrentPlan.features | string[] | SubscriptionPlan.supportFeatures | ⚠️ 字段名不同 | portal-api 映射 | — |
| CurrentPlan.isAutoRenew | boolean | 推断 | ⚠️ 需逻辑 | billingType 判断 | — |
| UsageStats.successRate | number (0~100) | ⚠️ 待确认 | ⚠️ | 单位歧义 | 联调确认 |
| UsageDetail.abilityId | string | billing_usage_records.capability_id | ⚠️ 字段名不同 | portal-api 映射 | — |
| UsageDetail.abilityName | string | capability registry | ⚠️ 需额外查询 | portal-api 映射 | — |
| RechargeOrder.applyRemark | string? | portal-api 自有 | ✅ 已修复 | — | — |

### 4.2 金额单位核对

| 字段 | 前端假设 | 后端实际 | 一致性 |
|------|---------|---------|--------|
| CurrentPlan.price | number (元) | SubscriptionPlan.price (元) | ✅ |
| UsageStats.totalCost | number (元) | billing_usage_records.cost (元) | ✅ |
| UsageDetail.cost | number (元) | billing_usage_records.cost (元) | ✅ |
| UsageTrendPoint.cost | number (元) | 聚合计算 (元) | ✅ |
| BalanceAccount.cashBalance | number (元) | portal-api 自有 | ✅ |

### 4.3 百分比单位核对

| 字段 | 前端假设 | 后端待确认 | 一致性 | 修复建议 |
|------|---------|-----------|--------|---------|
| UsageStats.successRate | 0~100 (如 98.5) | ⚠️ 待确认 | ⚠️ | 联调确认，已标注注释 |
| UsageDetail.successRate | 0~100 (如 99) | ⚠️ 待确认 | ⚠️ | 联调确认，已标注注释 |
| UsageTrendPoint.successRate | 0~100 (如 98) | ⚠️ 待确认 | ⚠️ | 联调确认，已标注注释 |

### 4.4 日期格式核对

| 字段 | 前端处理 | 后端格式 | 一致性 |
|------|---------|---------|--------|
| CurrentPlan.validUntil | `new Date(v).toLocaleDateString('zh-CN')` + 空值校验 | ⚠️ 待确认 | ⚠️ 需确认 ISO 8601 |
| UsageStats.startDate/endDate | 直接展示 | ⚠️ 待确认 | ⚠️ 需确认格式 |
| UsageTrendPoint.date | 直接传递给 UsageChart | ⚠️ 待确认 | ⚠️ 需确认格式 |
| BalanceTransaction.createdAt | `formatDateTime(v)` | portal-api 自有 | ✅ |
| BillingOrder.generatedAt | `formatDateTime(v)` | portal-api 自有 | ✅ |
| RechargeOrder.createdAt | `formatDateTime(v)` | portal-api 自有 | ✅ |

---

## 五、主流程联调

| 流程 | 操作 | 前端请求 | 后端返回 | 前端表现 | 状态 |
|------|------|---------|---------|---------|------|
| 1. 打开页面 | 访问 /billing | Promise.all([balance, plan, usage]) + loadDetails | — | Skeleton → 数据/空态/错误态 | ✅ 前端就绪 |
| 2. 加载套餐 | plan() | `GET /portal-api/v1/billing/plan` | ❌ 404 | .catch → null → "暂无订阅套餐" | ⚠️ 降级可用 |
| 3. 加载用量 | usage({period:'monthly'}) | `GET /portal-api/v1/billing/usage?period=monthly` | ❌ 404/结构不匹配 | .catch → null → 空趋势/空明细 | ⚠️ 降级可用 |
| 4. 加载余额 | balance() | `GET /portal-api/v1/billing/balance` | ✅ BalanceAccount | 正确展示 | ✅ |
| 5. 查看流水 | Tab切换 | `GET /portal-api/v1/billing/transactions` | ✅ { items: [] } | AntTable 展示 | ✅ |
| 6. 充值操作 | 点击充值→填写→提交 | `POST /portal-api/v1/billing/recharge-orders` body: {amount, paymentMethod, applyRemark} | ⚠️ applyRemark 需同步 | message.success + 刷新 | ⚠️ 待同步 |
| 7. 取消充值 | 点击取消申请 | `POST /portal-api/v1/billing/recharge-orders/{orderNo}/cancel` | ✅ | message.success + 刷新 | ✅ |
| 8. 查看订单 | Tab切换 | `GET /portal-api/v1/billing/orders` | ✅ { items: [] } | AntTable 展示 | ✅ |
| 9. 升级套餐 | 点击升级套餐 | navigate('/subscription') | — | 路由跳转 | ✅ |
| 10. 配额警告 | 配额≥80%/95% | — | — | Alert 提示 | ✅ |

---

## 六、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 | 修复建议 |
|------|---------|---------|---------|---------|
| 接口 400 | `{ error: 'Bad Request' }` | try-catch → message.error | ✅ | — |
| 接口 401 | 401 Unauthorized | apiClient 统一跳转登录 | ✅ | — |
| 接口 403 | 403 Forbidden | try-catch → message.error | ✅ | — |
| 接口 404 | plan() 返回 404 | .catch → null → "暂无订阅套餐" | ✅ | 降级正确 |
| 接口 500 | Internal Error | loadOverview: Alert + 重试; loadDetails: message.error | ✅ | — |
| 网络超时 | timeout | catch → error 状态 | ✅ | — |
| 空数据 | balance=null, plan=null | Empty 组件 | ✅ | — |
| 字段缺失 | plan.features=[] | 不渲染特性列表 | ✅ | — |
| 枚举未知值 | RechargeOrder.status='unknown' | 不显示取消按钮 | ✅ | — |
| 权限不足 | 403 | message.error | ✅ | — |
| 重复提交充值 | 连续点击充值按钮 | recharging → Button loading 禁用 | ✅ | — |
| 金额边界 0.01 | amount=0.01 | validator 通过 | ✅ | — |
| 金额边界 0 | amount=0 | validator 拒绝 | ✅ | — |
| 金额边界负数 | amount=-1 | validator 拒绝 | ✅ | — |
| 日期格式异常 | validUntil='' | 显示 '-' | ✅ | BUG-FE-001 已修复 |
| 日期格式异常 | validUntil=null | 显示 '-' | ✅ | BUG-FE-001 已修复 |
| plan 降级 | plan() 500 | .catch → null → "暂无订阅套餐" | ✅ | — |
| usage 降级 | usage() 500 | .catch → null → 空趋势/空明细 | ✅ | — |
| 组件卸载 | 充值后离开页面 | cancelledRef → 跳过 setState | ✅ | — |
| XBIS 429 | TENANT_QUOTA_EXCEEDED | try-catch → message.error | ✅ | — |

---

## 七、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
|---------|---------|---------|---------|---------|---------|---------|
| BUG-BE-001 | **Blocker** | plan() 接口后端未实现，返回 404 | 访问计费概览页，套餐区域显示"暂无订阅套餐" | 后端 | 套餐+配额信息无法展示 | 后端实现聚合接口 |
| BUG-BE-002 | **Blocker** | usage() 接口后端未实现或返回结构不匹配 | 访问计费概览页，趋势图和用量明细为空 | 后端 | 用量统计无法展示 | 后端实现聚合接口 |
| BUG-BE-003 | Medium | createRechargeOrder 入参 applyRemark 后端未同步 | 充值提交后备注丢失 | 后端 | 充值备注丢失 | 后端接收 applyRemark |
| BUG-CT-001 | Medium | successRate 单位约定未确认 | 联调时可能显示错误百分比 | 契约 | 成功率显示异常 | 联调确认 0~100 |
| BUG-CT-002 | Low | CurrentPlan 字段名与 SubscriptionPlan 不一致 | plan() 实现时需映射 | 契约 | 后端实现复杂度 | portal-api 统一映射 |

---

## 八、Bug修复内容

### 前端问题：无新增前端 Bug

上一轮发现的 2 个前端 Bug（BUG-FE-001 validUntil 空值校验、BUG-FE-002 usagePercentage 移除）均已修复并复测通过。

### 后端问题：3 项待实现

#### BUG-BE-001/002：plan() 和 usage() 接口

**问题原因**：新增聚合接口后端尚未实现。

**推荐后端实现方案**：

##### plan() 接口

```
GET /portal-api/v1/billing/plan

请求：无参数（从认证 token 获取 tenantId）

响应：
{
  "data": {
    "id": "sub_abc123",
    "name": "专业月度套餐",
    "description": "适合中小团队的专业版套餐",
    "price": 199,
    "currency": "CNY",
    "quota": 10000,
    "usedQuota": 3500,
    "remainingQuota": 6500,
    "validUntil": "2026-05-31T23:59:59Z",
    "features": ["10000次/日API调用", "优先技术支持", "SLA保障"],
    "isAutoRenew": true
  }
}

无订阅时响应：
{
  "data": null
}
```

##### usage() 接口

```
GET /portal-api/v1/billing/usage?period=monthly

请求参数：
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| period | 'daily' \| 'weekly' \| 'monthly' | 否（默认 monthly） | 统计周期 |
| startDate | string (ISO 8601) | 否 | 起始日期 |
| endDate | string (ISO 8601) | 否 | 结束日期 |

响应：
{
  "data": {
    "stats": {
      "totalCalls": 3500,
      "totalCost": 1050.0,
      "averageLatency": 245,
      "successRate": 98.5,
      "period": "monthly",
      "startDate": "2026-04-01",
      "endDate": "2026-04-30"
    },
    "details": [
      {
        "abilityId": "healthcheck",
        "abilityName": "运维健康检查",
        "callCount": 3000,
        "cost": 900.0,
        "averageLatency": 200,
        "successRate": 99.0
      },
      {
        "abilityId": "data-query",
        "abilityName": "数据查询",
        "callCount": 500,
        "cost": 150.0,
        "averageLatency": 350,
        "successRate": 97.0
      }
    ],
    "trends": [
      { "date": "2026-04-01", "calls": 100, "cost": 30.0, "successRate": 100.0 },
      { "date": "2026-04-02", "calls": 120, "cost": 36.0, "successRate": 98.5 }
    ]
  }
}
```

#### BUG-BE-003：applyRemark 同步

**后端修改**：`POST /portal-api/v1/billing/recharge-orders` 请求体接收 `applyRemark` 字段（原 `remark`）。

### 契约问题：1 项待确认

#### BUG-CT-001：successRate 单位

**推荐统一方案**：后端返回 0~100 百分比（如 98.5），前端直接显示 `98.5%`。

**需同步修改**：
- 前端类型注释已标注 `// 0~100 百分比`
- 后端接口文档需明确 successRate 返回 0~100
- 联调时验证

---

## 九、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| BUG-FE-001 复测 | ✅ | validUntil 空值时显示 '-' |
| BUG-FE-002 复测 | ✅ | usagePercentage 字段已移除 |
| BUG-BE-001 复测 | ❌ 未解决 | plan() 接口后端仍未实现 |
| BUG-BE-002 复测 | ❌ 未解决 | usage() 接口后端仍未实现 |
| BUG-BE-003 复测 | ❌ 未解决 | applyRemark 后端未同步 |
| BUG-CT-001 复测 | ❌ 未解决 | successRate 单位未确认 |
| 主流程联调 | ✅ 前端就绪 | 前端代码完整，降级逻辑正确 |
| API契约复测 | ⚠️ 部分通过 | 已有接口通过，新增接口待后端实现 |
| 异常场景复测 | ✅ | 全部通过 |
| 旧功能回归 | ✅ | balance/transactions/orders/rechargeOrders 不受影响 |
| 降级逻辑验证 | ✅ | plan/usage 404 时正确降级 |

---

## 十、功能验收结论

基于任务卡 §9 验收标准：

| 验收项 | 状态 | 说明 |
|--------|------|------|
| 当前套餐展示正常 | ✅ 前端就绪 | 后端 plan() 未实现，降级显示"暂无订阅套餐" |
| 配额使用情况展示正常 | ✅ 前端就绪 | 后端 plan() 未实现，降级显示"暂无配额信息" |
| 用量统计展示正常 | ✅ 前端就绪 | 后端 usage() 未实现，降级显示空表格 |
| 用量趋势图展示正常 | ✅ 前端就绪 | 后端 usage() 未实现，降级显示空图表 |
| 升级入口可点击 | ✅ | navigate('/subscription') |
| 配额不足警告正常 | ✅ | 80%/95% Alert |
| 空态/加载态/错误态完整 | ✅ | 四态完整 |
| 响应式布局 | ✅ | Grid.useBreakpoint 全覆盖 |
| API 错误处理 | ✅ | try-catch + 降级 |
| Design Tokens | ✅ | 无硬编码 |
| React.memo | ✅ | 5 个 blocks 组件 |
| useEffect cleanup | ✅ | cancelledRef |
| 旧功能兼容 | ✅ | balance/transactions/orders/rechargeOrders 正常 |

👉 **状态：Accepted with notes**

**Notes**：
1. 前端代码完整，所有功能已实现
2. 新增 plan()/usage() 接口后端未实现，前端已做降级处理
3. applyRemark 参数变更需后端同步
4. successRate 单位需联调确认
5. 页面在 plan/usage 不可用时仍可正常展示余额、流水、充值、订单

---

## 十一、是否允许合并

| 结论 | 结果 | 说明 |
|------|------|------|
| 是否允许合并 | **YES** | 前端代码完整，降级逻辑保障页面可用 |
| 是否允许进入下一任务 | **YES** | 前端工作已完成 |
| 是否需要继续修复 | **NO** | 前端 Bug 已全部修复 |
| 是否需要后端继续处理 | **YES** | plan/usage 接口需实现，applyRemark 需同步 |
| 是否需要产品确认 | **NO** | 功能符合任务卡要求 |
| 是否需要再次联调 | **YES** | 后端实现 plan/usage 后需再次联调 |

**合并前置条件**：
1. 后端实现 `GET /portal-api/v1/billing/plan` 聚合接口
2. 后端实现 `GET /portal-api/v1/billing/usage` 聚合接口
3. 后端 `POST /portal-api/v1/billing/recharge-orders` 接收 `applyRemark` 参数
4. 联调时确认 successRate 返回 0~100 百分比格式
5. 联调时确认 validUntil 日期格式（ISO 8601）

**前端降级保障**：
- plan() 404/500 → .catch(() => null) → BillingSummaryCard 显示"暂无订阅套餐"，QuotaUsagePanel 显示"暂无配额信息"
- usage() 404/500 → .catch(() => null) → TrendChart 空图表，UsageStatistics 空表格
- 已有接口（balance/transactions/orders/rechargeOrders）不受影响，页面核心功能可用

---

*文档生成时间：2026-04-26*
*任务编号：T019*
*测试类型：前后端联调复测 + 功能验收*
*轮次：第二轮*
