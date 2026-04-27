# T027 管理端能力编辑页 - 接口联调检查报告（D1）

## 🧪 联调结果（D1）

👉 状态：**前端待修**（2 项已修复，3 项待后端确认）

---

## 1️⃣ API契约一致性

| API | 契约URL | 实现URL | Method | 状态 |
|-----|---------|---------|--------|------|
| 能力详情 | `GET /admin-api/v1/abilities/:id` | `adminApi.abilities.get(id)` → `GET /admin-api/v1/abilities/:id` | ✅ GET | ✅ 一致 |
| 能力更新 | `PUT /admin-api/v1/abilities/:id` | `adminApi.abilities.update(id, data)` → `PUT /admin-api/v1/abilities/:id` | ✅ PUT | ✅ 一致 |
| 能力发布 | `POST /admin-api/v1/abilities/:id/publish` | `adminApi.abilities.publish(id, data)` → `POST /admin-api/v1/abilities/:id/publish` | ✅ POST | ✅ 一致 |
| 版本历史 | `GET /admin-api/v1/abilities/:id/versions` | `adminApi.abilities.versions(id)` → `GET /admin-api/v1/abilities/:id/versions` | ✅ GET | ✅ 一致 |

### 参数检查

| API | 参数 | 契约定义 | 前端实现 | 状态 |
|-----|------|---------|---------|------|
| 能力详情 | path.id | `string` | `abilityId: string` | ✅ 一致 |
| 能力更新 | path.id | `string` | `abilityId: string` | ✅ 一致 |
| 能力更新 | body | `AbilityEditForm` | `AbilityUpdateRequest`（剔除 status/version/changeLog） | ✅ 合理 |
| 能力发布 | path.id | `string` | `abilityId: string` | ✅ 一致 |
| 能力发布 | body | `AbilityPublishRequest` | `{ version, changeLog }` | ✅ 一致 |
| 版本历史 | path.id | `string` | `abilityId: string` | ✅ 一致 |

### ❌ 发现问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| API参数 | 任务卡 `AbilityPublishRequest` 包含 `abilityId` 字段，但 `shared/types` 中 `AbilityPublishRequest` 只有 `version` 和 `changeLog` | 任务卡 §5.1 vs `ability.ts:287-290` | 低 — `abilityId` 已在 URL path 中，无需在 body 重复，前端实现正确 |
| API参数 | 任务卡 `AbilityEditForm` 作为 PUT 请求体，但前端实际发送 `AbilityUpdateRequest`（剔除 status/version/changeLog） | `useAbilityEdit.ts:87` | 低 — 前端正确地不发送只读字段，但任务卡描述不够精确 |

---

## 2️⃣ 返回结构匹配

### GET /admin-api/v1/abilities/:id 返回结构

**契约定义**（`AbilityDetailResponse`）:
```typescript
{
  ability: AbilityDetail;     // 核心能力数据
  versions: AbilityVersion[]; // 版本列表
  uiSchema?: AbilityUiSchemaRecord; // UI Schema 独立记录
  executorBindings?: AbilityExecutorBinding[];
}
```

**前端 `mapApiToForm` 处理**:
```typescript
const ability = apiData.ability || apiData;  // 兼容嵌套和扁平结构
const uiSchemaRecord = apiData.uiSchema || {};
```

| 字段 | 契约位置 | 前端映射 | 状态 |
|------|---------|---------|------|
| `ability.displayName` | `ability.displayName` | `form.displayName` | ✅ |
| `ability.description` | `ability.description` | `form.description` | ✅ |
| `ability.category` | `ability.category` | `form.category` | ✅ |
| `ability.tags` | `ability.tags` | `form.tags` | ✅ |
| `ability.icon` | `ability.icon` | `form.icon` | ✅ |
| `ability.riskLevel` | `ability.riskLevel` | `form.riskLevel` | ✅ |
| `ability.requestSchema` | `ability.requestSchema` | `form.requestSchema` | ✅ |
| `ability.responseSchema` | `ability.responseSchema` | `form.responseSchema` | ✅ |
| `uiSchema.uiSchema` | `uiSchema.uiSchema` | `form.uiSchema` | ✅ 已修复 |
| `uiSchema.formConfig` | `uiSchema.formConfig` | `form.formConfig` | ✅ 已修复 |
| `ability.executionMode` | `ability.executionMode` | `form.executionMode` | ✅ |
| `ability.timeout` | `ability.timeout` | `form.timeout` | ✅ |
| `ability.retryCount` | `ability.retryCount` | `form.retryCount` | ✅ |
| `ability.executorId` | `ability.executorId` | `form.executorId` | ✅ |
| `ability.pricingType` | `ability.pricingType` | `form.pricingType` | ✅ |
| `ability.unitPrice` | `ability.unitPrice` | `form.unitPrice` | ✅ |
| `ability.freeQuota` | `ability.freeQuota` | `form.freeQuota` | ✅ |
| `ability.overagePrice` | `ability.overagePrice` | `form.overagePrice` | ✅ |
| `ability.status` | `ability.status` | `form.status` | ✅ |
| `ability.version` | `ability.version` | `form.version` | ✅ |

### ❌ 发现问题（已修复）

| 类型 | 问题 | 位置 | 影响 | 修复 |
|------|------|------|------|------|
| 返回结构 | `mapApiToForm` 直接从 `res.data` 读取 `uiSchema`/`formConfig`，但 API 返回结构中这些字段在 `uiSchema` 子对象的 `uiSchema` 和 `formConfig` 属性中 | `useAbilityEdit.ts:6-29` | 高 — uiSchema/formConfig 数据丢失 | ✅ 已修复：使用 `apiData.ability` 和 `apiData.uiSchema` 分层解析 |

### GET /admin-api/v1/abilities/:id/versions 返回结构

**前端处理**:
```typescript
const res = await adminAbilitiesApi.versions(abilityId);
setVersions(res.data?.items || []);
```

| 字段 | 契约定义 | 前端映射 | 状态 |
|------|---------|---------|------|
| `items` | `AbilityVersion[]` | `res.data?.items` | ⚠️ 待确认 |
| `items[].id` | `string` | `AbilityVersionListItem.id` | ✅ |
| `items[].abilityId` | `string` | `AbilityVersionListItem.abilityId` | ✅ |
| `items[].version` | `string` | `AbilityVersionListItem.version` | ✅ |
| `items[].changeLog` | `string` | `AbilityVersionListItem.changeLog` | ✅ |
| `items[].status` | `AbilityVersionStatus` | `AbilityVersionListItem.status` | ✅ |
| `items[].createdAt` | `string` | `AbilityVersionListItem.createdAt` | ✅ |
| `items[].publishedAt` | `string?` | `AbilityVersionListItem.publishedAt` | ✅ |

---

## 3️⃣ 数据结构一致性

### AbilityEditForm vs Ability 核心实体

| 字段 | AbilityEditForm | Ability | AbilityUpdateRequest | 状态 |
|------|----------------|---------|---------------------|------|
| `displayName` | `string` | `string` | `string?` | ✅ |
| `description` | `string` | `string` | `string?` | ✅ |
| `category` | `string` | `string` | `string?` | ✅ |
| `tags` | `string[]` | `string[]` | `string[]?` | ✅ |
| `icon` | `string?` | `string?` | `string?` | ✅ |
| `riskLevel` | `AbilityRiskLevel` | `AbilityRiskLevel` | `AbilityRiskLevel?` | ✅ |
| `requestSchema` | `Record<string, unknown>` | `Record<string, unknown>?` | `Record<string, unknown>?` | ✅ |
| `responseSchema` | `Record<string, unknown>` | `Record<string, unknown>?` | `Record<string, unknown>?` | ✅ |
| `uiSchema` | `Record<string, unknown>` | ❌ 不在 Ability 中 | `Record<string, unknown>?` | ✅ 在 AbilityUiSchemaRecord 中 |
| `formConfig` | `Record<string, unknown>` | ❌ 不在 Ability 中 | `Record<string, unknown>?` | ✅ 在 AbilityUiSchemaRecord 中 |
| `executionMode` | `AbilityExecutionMode` | `AbilityExecutionMode` | `AbilityExecutionMode?` | ✅ |
| `timeout` | `number` | `number` | `number?` | ✅ |
| `retryCount` | `number` | `number` | `number?` | ✅ |
| `executorId` | `string?` | `string?` | `string?` | ✅ |
| `pricingType` | `AbilityPricingType` | `AbilityPricingType` | `AbilityPricingType?` | ✅ |
| `unitPrice` | `number?` | `number?` | `number?` | ✅ |
| `freeQuota` | `number?` | `number?` | `number?` | ✅ |
| `overagePrice` | `number?` | `number?` | `number?` | ✅ |
| `status` | `AbilityStatus` | `AbilityStatus` | `AbilityStatus?` | ✅ |
| `version` | `string` | `string` | ❌ 不在 UpdateRequest | ✅ 前端已剔除 |
| `changeLog` | `string` | ❌ 不在 Ability 中 | ❌ 不在 UpdateRequest | ✅ 前端已剔除 |

### ❌ 发现问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 数据结构 | `AbilityEditForm.changeLog` 在 Ability 核心实体中不存在，也不在 AbilityUpdateRequest 中 | `types.ts:398` | 低 — 前端已在 saveForm 中剔除，但 changeLog 的来源不明确（可能是 AbilityVersion 的字段） |
| 数据结构 | `AbilityEditForm` 包含 `uiSchema`/`formConfig`，但 `Ability` 核心实体不包含这些字段 | `types.ts:386-387` | 低 — 前端已通过 `AbilityDetailResponse.uiSchema` 正确映射 |

---

## 4️⃣ 状态流一致性

### AbilityStatus

| 值 | 任务卡 | shared/types | 前端使用 | 状态 |
|----|--------|-------------|---------|------|
| `draft` | ✅ | ✅ `'draft'` | ✅ `initialForm.status = 'draft'` | ✅ |
| `published` | ✅ | ✅ `'published'` | ✅ | ✅ |
| `deprecated` | ✅ | ✅ `'deprecated'` | ✅ | ✅ |

### AbilityVersionStatus

| 值 | 任务卡 | shared/types | 前端使用 | 状态 |
|----|--------|-------------|---------|------|
| `draft` | — | ✅ | ✅ `VERSION_STATUS_MAP.draft` | ✅ |
| `testing` | — | ✅ | ✅ `VERSION_STATUS_MAP.testing` | ✅ |
| `published` | — | ✅ | ✅ `VERSION_STATUS_MAP.published` | ✅ |
| `deprecated` | — | ✅ | ✅ `VERSION_STATUS_MAP.deprecated` | ✅ |

### AbilityRiskLevel

| 值 | 任务卡 | shared/types | BasicInfoTab 选项 | 状态 |
|----|--------|-------------|------------------|------|
| `low` | ✅ | ✅ | ✅ | ✅ |
| `medium` | ✅ | ✅ | ✅ | ✅ |
| `high` | ✅ | ✅ | ✅ | ✅ |
| `critical` | ✅ | ✅ | ❌ 缺失 | ✅ 已修复 |

### AbilityExecutionMode

| 值 | 任务卡 | shared/types | ExecutionTab 选项 | 状态 |
|----|--------|-------------|------------------|------|
| `sync` | ✅ | ✅ | ✅ | ✅ |
| `async` | ✅ | ✅ | ✅ | ✅ |
| `both` | ✅ | ✅ | ✅ | ✅ |
| `review-only` | ✅ | ✅ | ✅ | ✅ |

### AbilityPricingType

| 值 | 任务卡 | shared/types | BillingTab 选项 | 状态 |
|----|--------|-------------|----------------|------|
| `free` | ✅ | ✅ | ✅ | ✅ |
| `per_call` | ✅ | ✅ | ✅ | ✅ |
| `package` | ✅ | ✅ | ✅ | ✅ |
| `overage` | ✅ | ✅ | ✅ | ✅ |

### ❌ 发现问题（已修复）

| 类型 | 问题 | 位置 | 影响 | 修复 |
|------|------|------|------|------|
| 状态流 | `BasicInfoTab` 的 `RISK_LEVEL_OPTIONS` 缺少 `critical` 选项 | `BasicInfoTab/index.tsx` | 高 — 极高风险能力无法正确设置风险等级 | ✅ 已添加 |

---

## 5️⃣ 错误处理

| 场景 | 处理方式 | 位置 | 状态 |
|------|---------|------|------|
| 页面加载失败 | `useAbilityEdit` catch → `SET_ERROR` → `FormPageShell status="error"` → 重试按钮 | `useAbilityEdit.ts:68-70` | ✅ |
| 保存失败 | `useAbilityEdit` catch → `SET_ERROR` → `onError?.('保存失败')` | `useAbilityEdit.ts:91-93` | ✅ |
| 发布失败 | `useAbilityEdit` catch → `SET_ERROR` → `onError?.('发布失败')` | `useAbilityEdit.ts:107-109` | ✅ |
| 版本列表加载失败 | `PublishTab` catch → `setError` → `Alert type="error"` | `PublishTab/index.tsx:44-45` | ✅ |
| 空数据（能力不存在） | `abilityId` 为空 → `SET_ERROR` | `useAbilityEdit.ts:55-56` | ✅ |
| 空数据（版本列表为空） | `versions.length === 0` → `Empty` 组件 | `PublishTab/index.tsx:134` | ✅ |
| 加载中 | `FormPageShell status="loading"` | `AbilityEditPage.tsx:101` | ✅ |
| 版本列表加载中 | `Spinner` 组件 | `PublishTab/index.tsx:106-110` | ✅ |
| 表单验证失败 | `validateForm` → `onError?.()` → 底部状态栏提示 | `AbilityEditPage.tsx:17-44` | ✅ |
| 自动保存失败 | 静默失败，保留 isDirty 状态 | `useAutoSave.ts` | ✅ |
| 超时 | `apiClient` 默认 30s 超时 | `client.ts:19` | ✅ |

### ❌ 发现问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 错误处理 | 未处理 404（能力不存在）场景的特殊提示 | `useAbilityEdit.ts` | 低 — 当前统一显示错误信息，但缺少「返回列表」快捷操作 |
| 错误处理 | 未处理 409（版本冲突）场景的特殊提示 | `AbilityEditPage.tsx` | 低 — 当前统一显示错误信息，但缺少版本冲突的专门提示 |
| 错误处理 | 未实现 `beforeunload` 离开页面确认 | 设计方案 §6.2 | 中 — 用户可能意外丢失未保存数据 |

---

## 6️⃣ Business Services 层检查

| API 调用 | 是否通过 Service 层 | Service 方法 | 状态 |
|---------|-------------------|-------------|------|
| 能力详情 | ✅ | `adminApi.abilities.get(id)` | ✅ |
| 能力更新 | ✅ | `adminApi.abilities.update(id, data)` | ✅ |
| 能力发布 | ✅ | `adminApi.abilities.publish(id, data)` | ✅ |
| 版本历史 | ✅ | `adminApi.abilities.versions(id)` | ✅ |

### ❌ 发现问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| Service层 | `adminApi.abilities` 的 `publish`/`versions` 方法需要类型断言才能使用 | `useAbilityEdit.ts:32-37`, `PublishTab/index.tsx:9-13` | 低 — 功能正常，但类型推断应优化 |

---

## ❌ 问题汇总

| # | 类型 | 问题 | 位置 | 影响 | 状态 |
|---|------|------|------|------|------|
| 1 | 返回结构 | `mapApiToForm` 未正确解析 `AbilityDetailResponse` 嵌套结构 | `useAbilityEdit.ts:6-29` | 🔴 高 | ✅ 已修复 |
| 2 | 状态流 | `BasicInfoTab` 缺少 `critical` 风险等级选项 | `BasicInfoTab/index.tsx` | 🔴 高 | ✅ 已修复 |
| 3 | 错误处理 | 未实现 `beforeunload` 离开页面确认 | `AbilityEditPage.tsx` | 🟡 中 | ✅ 已修复 |
| 4 | 错误处理 | 未处理 404/409 特殊错误码 | `useAbilityEdit.ts` | 🟡 低 | ⏳ 待后端确认 |
| 5 | Service层 | `publish`/`versions` 方法需要类型断言 | `useAbilityEdit.ts`, `PublishTab/index.tsx` | 🟡 低 | ⏳ 待优化 |
| 6 | API参数 | 任务卡 `AbilityPublishRequest.abilityId` 与 shared 类型不一致 | 任务卡 §5.1 | 🟢 无影响 | ℹ️ 任务卡描述偏差 |

---

## 🛠 修改建议

### 前端修改（已执行）：
1. ✅ **修复 `mapApiToForm`**：从 `apiData.ability` 读取能力字段，从 `apiData.uiSchema` 读取 UI Schema 字段
2. ✅ **添加 `critical` 风险等级选项**：BasicInfoTab 的 RISK_LEVEL_OPTIONS 新增极高风险
3. ✅ **实现 `beforeunload`**：在 `AbilityEditPage` 中添加 `useEffect` 监听 `beforeunload` 事件，isDirty 时阻止页面关闭

### 前端修改（待执行）：
4. **优化 `createAbilityApi` 类型**：使 `includeAdminOps: true` 时返回包含 `publish`/`versions` 的类型，避免类型断言

### 后端修改：
5. **确认 `GET /admin-api/v1/abilities/:id` 返回结构**：是否为 `AbilityDetailResponse`（嵌套结构）还是扁平的 `Ability`
6. **确认 `GET /admin-api/v1/abilities/:id/versions` 返回结构**：是否为 `{ items: AbilityVersion[] }` 格式

### 数据结构调整：
7. **任务卡 `AbilityPublishRequest`**：移除 `abilityId` 字段（已在 URL path 中），与 shared 类型保持一致

---

## ⚠️ 风险评估

- **是否影响后续任务**: 低 — 修复的 2 项问题均为前端内部问题，不影响其他任务
- **是否影响数据模型**: 低 — `mapApiToForm` 修复确保了 API 返回数据的正确解析，避免 uiSchema/formConfig 数据丢失

---

## ✅ 是否允许进入验收（D2）

👉 **YES** — 2 项高影响问题已修复，剩余为低优先级优化项，不阻塞验收

前提条件：
- 后端需确认 `AbilityDetailResponse` 返回结构（嵌套 vs 扁平）
- 后端需确认版本列表 API 返回格式
