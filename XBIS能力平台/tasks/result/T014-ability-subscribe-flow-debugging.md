# T014 能力接入流程 — 接口联调检查报告（D1）

> 任务编号：T014
> 检查阶段：D1 — 接口联调检查
> 检查日期：2026-04-25
> 执行人：资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 🧪 联调结果（D1）

👉 **状态：联调通过**

说明：6 个检查维度全部通过，发现 1 项类型建议项，无阻塞性问题。

---

## 一、API 契约一致性

| 检查项 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|--------|-----------|----------|------|------|
| 获取套餐列表 URL | `GET /api/v1/abilities/:id/plans` | `GET /api/v1/abilities/${id}/plans` | ✅ 一致 | `services.ts:149` |
| 获取套餐列表 Method | GET | GET | ✅ 一致 | — |
| 接入能力 URL | `POST /api/v1/abilities/:id/subscribe` | `POST /api/v1/abilities/${id}/subscribe` | ✅ 一致 | `services.ts:143` |
| 接入能力 Method | POST | POST | ✅ 一致 | — |
| 接入能力 Body | `abilityId`, `planId`, `couponCode` | 完全一致 | ✅ 一致 | `AbilitySubscribePage.tsx:138-142` |
| 重新生成密钥 URL | `POST /api/v1/subscriptions/:id/regenerate-key` | `POST /portal-api/v1/subscriptions/${id}/regenerate-key` | ⚠️ 前缀差异 | 代码使用 `portal-api` 前缀 |
| 重新生成密钥 Method | POST | POST | ✅ 一致 | — |
| 多余参数 | 无 | 无 | ✅ 通过 | — |

**前缀差异说明**：
- 任务卡定义 `/api/v1/subscriptions/:id/regenerate-key`
- 代码实现 `/portal-api/v1/subscriptions/:id/regenerate-key`
- 差异原因：项目现有 `subscriptions` API 均使用 `portal-api` 前缀（`services.ts:370-396`），为保持一致性，重新生成密钥也使用 `portal-api` 前缀
- **结论**：前缀差异为项目内部约定，不影响功能

---

## 二、返回结构匹配

### 2.1 获取套餐列表响应

| 字段 | 任务卡定义 | 代码类型定义 | 状态 | 说明 |
|------|-----------|-------------|------|------|
| `id` | `string` | `string` | ✅ 一致 | — |
| `name` | `string` | `string` | ✅ 一致 | — |
| `description` | `string` | `string` | ✅ 一致 | — |
| `price` | `number` | `number` | ✅ 一致 | — |
| `currency` | `string` | `string` | ✅ 一致 | — |
| `quota` | `number` | `number` | ✅ 一致 | — |
| `overagePrice` | `number` (optional) | `number` (optional) | ✅ 一致 | — |
| `features` | `string[]` | `string[]` | ✅ 一致 | — |
| `isRecommended` | `boolean` | `boolean` | ✅ 一致 | — |

### 2.2 接入能力响应

| 字段 | 任务卡定义 | 代码类型定义 | 状态 | 说明 |
|------|-----------|-------------|------|------|
| `subscriptionId` | `string` | `string` | ✅ 一致 | — |
| `abilityId` | `string` | `string` | ✅ 一致 | — |
| `planId` | `string` | `string` | ✅ 一致 | — |
| `status` | `'active' \| 'pending' \| 'expired'` | `'active' \| 'pending' \| 'expired'` | ✅ 一致 | — |
| `apiKey` | `string` | `string` | ✅ 一致 | — |
| `apiSecret` | `string` | `string` | ✅ 一致 | — |
| `quota` | `number` | `number` | ✅ 一致 | — |
| `usedQuota` | `number` | `number` | ✅ 一致 | — |
| `validUntil` | `string` | `string` | ✅ 一致 | — |
| `createdAt` | `string` | `string` | ✅ 一致 | — |

### 2.3 重新生成密钥响应

| 字段 | 任务卡定义 | 代码类型定义 | 状态 | 说明 |
|------|-----------|-------------|------|------|
| `apiKey` | `string` | `string` | ✅ 一致 | `RegenerateKeyResponse` |
| `apiSecret` | `string` | `string` | ✅ 一致 | `RegenerateKeyResponse` |

---

## 三、数据结构一致性

| 检查项 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|--------|-----------|----------|------|------|
| `PlanOption` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:295-305` |
| `AbilitySubscribeRequest` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:307-311` |
| `AbilitySubscribeDetailResponse` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:313-324` |
| `SubscriptionInfo` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts:326-334` |
| `RegenerateKeyResponse` | 完整定义 | 完整定义 | ✅ 一致 | `ability.ts` |
| 字段命名冲突 | 无 | 无 | ✅ 通过 | — |
| 结构嵌套 | 正确 | 正确 | ✅ 通过 | — |

---

## 四、状态流一致性

| 状态 | 任务卡定义 | 代码实现 | 状态 | 说明 |
|------|-----------|----------|------|------|
| `active` | ✅ | ✅ | ✅ 一致 | 接入成功 |
| `pending` | ✅ | ✅ | ✅ 一致 | 待激活 |
| `expired` | ✅ | ✅ | ✅ 一致 | 已过期 |

**状态转换验证**：
```
用户确认接入
  ├── 成功 ──► active ──► 展示密钥
  ├── 待支付 ──► pending ──► 等待支付
  └── 过期 ──► expired ──► 提示续费
```

**验证依据**：
- 任务卡 §6.2 异常流程：已接入、套餐不可用、配额不足
- 代码实现：`AbilitySubscribePage.tsx:131-156` handleSubscribe 函数

---

## 五、错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | `try/catch` 捕获，`setSubscribeError` 设置错误状态 |
| 空数据处理 | ✅ | `if (!result)` 校验，抛出错误 |
| loading 处理 | ✅ | `subscribeLoading` 状态，按钮禁用 |
| 超时/失败处理 | ✅ | 同步调用由 API 层处理超时，错误由 catch 捕获 |
| 套餐加载失败 | ✅ | `plansError` 状态，显示重试按钮 |

**验证依据**：
- `AbilitySubscribePage.tsx:131-156` handleSubscribe 函数包含完整 try/catch/finally
- `AbilitySubscribePage.tsx:84-98` loadPlans 函数包含错误处理
- `AbilitySubscribePage.tsx:324-326` 错误展示使用 Alert 组件

---

## 六、Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 是否通过 Service 层调用 API | ✅ | `userApi.abilities.plans(id)` / `userApi.abilities.subscribe(id, data)` / `userApi.subscriptions.regenerateKey(id)` |
| 是否绕过业务接入层 | ✅ | 未绕过，使用 `userApi` |
| API 定义位置 | ✅ | `packages/shared/src/api/services.ts` |

**验证依据**：
- `packages/shared/src/api/services.ts:143-150` 定义 subscribe/plans API
- `packages/shared/src/api/services.ts:394-395` 定义 regenerateKey API
- `packages/pages/ability/AbilitySubscribePage.tsx:138` 通过 `userApi.abilities.subscribe` 调用

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 严重级别 |
|------|------|------|------|----------|
| API 前缀 | 重新生成密钥使用 `portal-api` 前缀，任务卡定义 `/api/v1` | `services.ts:394` | 与任务卡定义不一致，但符合项目现有约定 | 🟡 建议 |

---

## 🛠 修改建议

### 前端修改

**建议 1：保持现有 `portal-api` 前缀**

```typescript
// 当前实现（推荐保持）
regenerateKey: (subscriptionId: string) =>
  apiClient.post(`/portal-api/v1/subscriptions/${encodePathParam(subscriptionId)}/regenerate-key`),
```

**理由**：
1. 项目现有 `subscriptions` 相关 API 均使用 `portal-api` 前缀（`list`/`plans`/`current`/`history`/`changeQuote`/`change`）
2. 保持前缀一致性，避免后端路由混乱
3. 任务卡定义 `/api/v1` 为通用描述，实际前缀由项目架构决定

### 后端修改

无 — 前端实现与项目现有 API 约定一致。

### 数据结构调整

无 — 当前数据结构满足需求。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | **低** | API 前缀为项目内部约定，不影响功能 |
| 是否影响数据模型 | **无** | 请求/响应数据结构完全一致 |
| 前缀一致性 | **低** | `portal-api` 前缀与现有 subscriptions API 保持一致 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

**通过理由**：
1. API 契约一致性：URL、Method、参数完全一致（前缀差异为项目约定）
2. 返回结构匹配：所有字段名、类型匹配
3. 数据结构一致性：类型定义完整，无冲突
4. 状态流一致性：active/pending/expired 全部匹配
5. 错误处理完整：API error、空数据、loading、超时均有处理
6. Business Services 层合规：通过 `userApi` 调用，未绕过接入层

**遗留建议**（非阻塞）：
- 重新生成密钥 API 前缀使用 `portal-api`，与任务卡 `/api/v1` 定义存在差异，但符合项目现有约定

---

*本文档与代码同步维护，后续迭代请同步更新。*
