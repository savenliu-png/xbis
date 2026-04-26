# T008 ability API 层 — D1 接口联调检查报告

> 任务编号：T008
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-25
> 检查人：AI 联调审查官

---

## 🧪 联调结果（D1）

**👉 状态：联调通过（附 2 项建议优化）**

前端实现与 API 契约 / 数据结构整体一致，未发现阻塞性不一致问题。

---

## ❌ 问题列表（详细）

### 1. API 参数：设计方案中用户端缺少 `create` 端点，但类型已定义

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | 设计方案 Final 版本（T008-ability-api-design-final.md）中用户端 API 未定义 `create` 端点，但 `ability.ts` 中已定义 `AbilityCreateRequest` 类型 |
| **位置** | `tasks/design-specs/final/T008-ability-api-design-final.md` L34-L39 |
| **影响** | 用户端无法创建能力，仅管理端可创建 |
| **当前代码** | 用户端 `userApi.abilities` 仅有 list/get/subscribe/unsubscribe |
| **修改建议** | 确认业务需求：能力创建是否仅限管理员？若是，当前实现正确；若用户也可创建，需补充 `userApi.abilities.create` |

---

### 2. API 参数：管理端 `publish` 方法参数类型为可选，但设计方案中未明确

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | `adminApi.abilities.publish(id, data?)` 中 `data` 为可选参数，但设计方案中 `publish` 未显示参数 |
| **位置** | `packages/shared/src/api/services.ts` L203-L204 |
| **影响** | 若后端要求必须传入 `AbilityPublishRequest`，则可选参数可能导致 400 错误 |
| **当前代码** | ```ts
publish: (id: string, data?: AbilityPublishRequest) =>
  apiClient.post(`/admin-api/v1/abilities/${encodePathParam(id)}/publish`, data || {}),
``` |
| **修改建议** | 与后端确认 `publish` API 是否必须传入 `version` 和 `changeLog`：<br>1. 若必须传入，改为 `data: AbilityPublishRequest`（非可选）<br>2. 若可选，当前实现正确 |

---

## 🛠 修改建议汇总

### 前端修改（建议确认）

1. **确认用户端是否需要 `create` 能力**：
   - 若需要：补充 `userApi.abilities.create(data: AbilityCreateRequest)`
   - 若不需要：当前实现正确，能力创建仅限管理员

2. **确认 `publish` 参数是否可选**：
   - 若必须传入：修改 `data?: AbilityPublishRequest` → `data: AbilityPublishRequest`
   - 若可选：当前实现正确

### 后端确认项

1. `POST /admin-api/v1/abilities/:id/publish` 是否必须传入 `version` 和 `changeLog`？
2. 用户端是否有创建能力的权限？

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | 低 | API 层已完整定义，后续页面任务可直接调用 |
| 是否影响数据模型 | 无 | 类型定义完整，无冲突 |
| 用户端功能缺失风险 | 低 | 若用户端确实不需要创建能力，则无风险 |
| publish 参数风险 | 低 | `data || {}` 兜底，即使不传参数也不会报错 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

理由：
1. API 契约整体一致，URL、Method、参数定义完整
2. 返回结构匹配（`AbilityListResponse` / `AbilityDetailResponse` / `Ability` 等类型已定义）
3. 数据结构符合任务卡定义（ability 核心实体 + 版本 + UI Schema + 执行器绑定）
4. 状态定义一致（`AbilityStatus`: 'published' | 'draft' | 'deprecated'）
5. 错误处理由 apiClient 拦截器统一处理
6. Business Services 层检查通过（通过 `apiClient` 调用，未绕过接入层）
7. 发现的 2 项问题均为建议确认项，不影响核心功能联调

---

## 📎 附录：API 契约对照表

### 用户端 API (`userApi.abilities`)

| API | Method | 前端定义 | 设计方案定义 | 状态 |
|-----|--------|----------|-------------|------|
| list | GET | `/api/v1/abilities` | `/api/v1/abilities` | ✅ 一致 |
| get | GET | `/api/v1/abilities/:id` | `/api/v1/abilities/:id` | ✅ 一致 |
| subscribe | POST | `/api/v1/abilities/:id/subscribe` | `/api/v1/abilities/:id/subscribe` | ✅ 一致 |
| unsubscribe | POST | `/api/v1/abilities/:id/unsubscribe` | `/api/v1/abilities/:id/unsubscribe` | ✅ 一致 |
| create | POST | 未实现 | 未定义 | ⏳ 待确认 |

### 管理端 API (`adminApi.abilities`)

| API | Method | 前端定义 | 设计方案定义 | 状态 |
|-----|--------|----------|-------------|------|
| list | GET | `/admin-api/v1/abilities` | `/admin-api/v1/abilities` | ✅ 一致 |
| get | GET | `/admin-api/v1/abilities/:id` | 未定义 | ✅ 补充 |
| create | POST | `/admin-api/v1/abilities` | 未定义 | ✅ 补充 |
| update | PUT | `/admin-api/v1/abilities/:id` | `/admin-api/v1/abilities/:id` | ✅ 一致 |
| delete | DELETE | `/admin-api/v1/abilities/:id` | `/admin-api/v1/abilities/:id` | ✅ 一致 |
| publish | POST | `/admin-api/v1/abilities/:id/publish` | `/admin-api/v1/abilities/:id/publish` | ⚠️ 参数待确认 |
| deprecate | POST | `/admin-api/v1/abilities/:id/deprecate` | `/admin-api/v1/abilities/:id/deprecate` | ✅ 一致 |

---

## 📎 附录：类型定义完整性检查

| 类型 | 字段 | 状态 |
|------|------|------|
| `AbilityListParams` | pageNo, pageSize, category, status, riskLevel, executionMode, pricingType, keyword, tags | ✅ 完整 |
| `AbilityListResponse` | items, total, pageNo, pageSize | ✅ 完整 |
| `AbilityDetailResponse` | ability, versions, uiSchema?, executorBindings? | ✅ 完整 |
| `AbilityCreateRequest` | name, displayName, description, category, tags?, icon?, riskLevel, executionMode, requestSchema?, responseSchema?, executorId?, timeout?, retryCount?, pricingType, unitPrice?, freeQuota?, overagePrice?, docMarkdown? | ✅ 完整 |
| `AbilityUpdateRequest` | displayName?, description?, category?, tags?, icon?, status?, riskLevel?, executionMode?, requestSchema?, responseSchema?, executorId?, timeout?, retryCount?, pricingType?, unitPrice?, freeQuota?, overagePrice?, docMarkdown? | ✅ 完整 |
| `AbilityPublishRequest` | version?, changeLog? | ✅ 完整 |
| `AbilitySubscribeResponse` | subscriptionId, abilityId, status, subscribedAt, expiresAt? | ✅ 完整 |
