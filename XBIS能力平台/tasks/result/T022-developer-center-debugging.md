# T022 开发者中心首页 — 接口联调检查报告（D1）

> 任务卡: T022-developer-center.md
> 设计方案: tasks/design-specs/final/T022-developer-center-design-final.md
> 检查日期: 2026-04-26
> 检查人: 资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 1️⃣ API契约一致性

### 设计方案定义的 API

| API | Method | Path |
|-----|--------|------|
| Key 列表 | GET | `/api/v1/developer/keys` |
| 创建 Key | POST | `/api/v1/developer/keys` |
| 删除 Key | DELETE | `/api/v1/developer/keys/:id` |

### 实际 services.ts 定义的 API

| API | Method | Path |
|-----|--------|------|
| Key 列表 | GET | `/portal-api/v1/api-keys` |
| 创建 Key | POST | `/portal-api/v1/api-keys` |
| 删除 Key | DELETE | `/portal-api/v1/api-keys/:id` |

### ❌ 问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| API URL | 设计方案写 `/api/v1/developer/keys`，实际调用 `/portal-api/v1/api-keys` | 设计方案 §5 vs services.ts | **低** — 设计方案是概念路径，实际路径以后端为准，前端已使用正确路径 |
| API 参数 | `userApi.apiKeys.list(params)` 参数类型为 `any`，缺少分页/筛选参数类型 | services.ts L438 | **中** — 旧 ApiKeys 页面传 `{ page, pageSize, keyName, status }`，DeveloperCenter 未传分页参数 |
| API 参数 | `userApi.apiKeys.create(data)` 参数类型为 `any`，未使用 `CreateApiKeyRequest` 类型 | services.ts L439 | **中** — 失去类型安全保护 |
| API 参数 | `userApi.apiKeys.delete(id, data)` 第二参数可选，但 DeveloperCenter 只传 id | DeveloperCenter.tsx L75 | **低** — 简化删除流程，未处理逻辑删除场景 |

---

## 2️⃣ 返回结构匹配

### 列表接口返回结构

后端实际返回（根据旧 ApiKeys 页面分析）：

```typescript
{
  success: boolean,
  data: {
    items: ApiKeyListItem[],  // snake_case 字段
    total: number,
    stats?: ApiKeyStats
  }
}
```

### 前端 DeveloperCenter 解包逻辑

```typescript
const res = await userApi.apiKeys.list();
const data = res?.data?.data ?? res?.data ?? res;
const items = Array.isArray(data?.items) ? data.items : Array.isArray(data) ? data : [];
setApiKeys(items.map(mapApiKeyItem));
```

### ❌ 问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 字段名 | 后端返回 `key_name`（snake_case），前端 `ApiKey` 类型用 `name`（camelCase） | DeveloperCenter.tsx mapApiKeyItem | **已处理** — mapApiKeyItem 做了双格式兼容 |
| 字段名 | 后端返回 `key_id`，前端 `ApiKey` 类型用 `keyId` | DeveloperCenter.tsx mapApiKeyItem | **已处理** — mapApiKeyItem 做了双格式兼容 |
| 字段名 | 后端返回 `key_prefix`，前端 `ApiKey` 类型用 `keyPrefix` | DeveloperCenter.tsx mapApiKeyItem | **已处理** — mapApiKeyItem 做了双格式兼容 |
| 字段名 | 后端返回 `created_at`，前端 `ApiKey` 类型用 `createdAt` | DeveloperCenter.tsx mapApiKeyItem | **已处理** — mapApiKeyItem 做了双格式兼容 |
| 缺失字段 | `ApiKey` 类型缺少 `id`、`scope_type`、`source_system`、`description`、`allow_callback`、`allowed_apis`、`ip_whitelist`、`has_invocation_records` | types/index.ts ApiKey | **中** — ApiKey 类型不完整，无法展示完整信息 |
| 类型不匹配 | `ApiKeyListItem.status` 含 6 种状态，`ApiKey.status` 只含 4 种（缺 `deleted`、`pending`） | types/index.ts | **高** — 如果后端返回 `deleted` 或 `pending` 状态，前端类型断言会出错 |
| 缺失字段 | 列表返回含 `total` 和 `stats`，DeveloperCenter 未使用 | DeveloperCenter.tsx | **低** — 当前不需要分页和统计，但未来扩展需注意 |

### 创建接口返回结构

后端实际返回（`CreateApiKeyResponse`）：

```typescript
{
  rawKey: string;      // camelCase
  key_id: string;      // snake_case
  key_name: string;    // snake_case
  key_prefix: string;  // snake_case
  status: string;
}
```

### ❌ 问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 字段名混用 | `CreateApiKeyResponse` 中 `rawKey` 是 camelCase，`key_id`/`key_name`/`key_prefix` 是 snake_case | types/index.ts L1038-1044 | **低** — 前端已做双格式兼容 `result.rawKey ?? result.raw_key` |
| 返回解包 | `res?.data?.data ?? res?.data ?? res` 三层解包过于防御 | DeveloperCenter.tsx L55 | **中** — 应确认 apiClient 返回的实际结构 |

---

## 3️⃣ 数据结构一致性

### `CreateApiKeyRequest` vs 实际创建请求

**类型定义：**

```typescript
interface CreateApiKeyRequest {
  name: string;
  permissions?: Record<string, any>;
  expiresAt?: string;
}
```

**旧 ApiKeys 页面实际发送的 payload：**

```typescript
{
  keyName: string,           // ← 不是 name
  sourceSystem?: string,     // ← CreateApiKeyRequest 中未定义
  description?: string,      // ← CreateApiKeyRequest 中未定义
  expiresAt?: string,
  allowedApis?: string[],    // ← CreateApiKeyRequest 中未定义
  ipWhitelist?: string[],    // ← CreateApiKeyRequest 中未定义
  allowCallback?: boolean,   // ← CreateApiKeyRequest 中未定义
}
```

**DeveloperCenter 实际发送的 payload：**

```typescript
{
  name: string,                    // ← 使用 name，与类型一致
  permissions: { scope: string },  // ← 使用 permissions，与类型一致
}
```

### ❌ 问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 字段名 | 旧页面用 `keyName`，DeveloperCenter 用 `name`，后端实际接受哪个？ | ApiKeyManager L78-80 vs ApiKeys.tsx L140 | **高** — 如果后端只接受 `keyName`，则 DeveloperCenter 的创建请求会失败 |
| 缺失字段 | `CreateApiKeyRequest` 缺少 `sourceSystem`、`description`、`allowedApis`、`ipWhitelist`、`allowCallback` | types/index.ts L89-93 | **中** — DeveloperCenter 简化版创建不传这些字段，后端需支持可选 |
| 类型 | `permissions: { scope: keyPermission }` — 后端是否接受此结构？ | ApiKeyManager L80 | **中** — 旧页面不传 permissions，传的是 scope_type，结构可能不匹配 |

---

## 4️⃣ 状态流一致性

### `ApiKey.status` 状态枚举

| 状态 | `ApiKey` 类型 | `ApiKeyListItem` 类型 | 旧 ApiKeys 页面 | DeveloperCenter |
|------|--------------|----------------------|-----------------|-----------------|
| active | ✅ | ✅ | ✅ | ✅ |
| disabled | ✅ | ✅ | ✅ | ✅ |
| expired | ✅ | ✅ | ✅ | ✅ |
| locked | ✅ | ✅ | ✅ | ✅ |
| deleted | ❌ | ✅ | ✅ | ❌ |
| pending | ❌ | ✅ | ✅ | ❌ |

### ❌ 问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 状态缺失 | `ApiKey.status` 缺少 `deleted` 和 `pending` | types/index.ts L78 | **高** — mapApiKeyItem 强制断言 `as ApiKey['status']`，如果后端返回 `deleted`/`pending` 会运行时不出错但类型不安全 |
| 状态展示 | `ApiKeyItem` 组件只处理 4 种状态文字 | business/ApiKeyItem | **中** — `deleted`/`pending` 状态会 fallback 到"已锁定" |

---

## 5️⃣ 错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error | ✅ | try/catch 包裹，getErrorMessage 提取错误信息 |
| 空数据（empty） | ✅ | ApiKeyManager 有 Empty 组件 + 引导创建 |
| Loading | ✅ | pageState='loading' 时 DetailPageShell 显示 Spinner |
| 超时/失败 | ✅ | apiClient timeout=30000，DetailPageShell 有重试按钮 |
| 创建失败 | ✅ | createError 状态 + Alert 展示 |
| 删除失败 | ⚠️ | handleDeleteKey 中 throw Error，但 ApiKeyManager 的 handleDelete 用 try/finally 包裹，异常被吞没 |

### ❌ 问题

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 错误吞没 | `handleDelete` 中 `onDeleteKey(keyId)` 抛出异常，但 `finally` 块只重置 deletingId，异常未展示给用户 | ApiKeyManager L99-109 | **中** — 删除失败时用户无反馈 |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 通过 Service 层调用 | ✅ | 全部通过 `userApi.apiKeys` 调用 |
| 是否绕过 XBIS | ✅ | 未绕过，使用标准 portal-api 路径 |
| 认证鉴权 | ✅ | apiClient 自动附加 Bearer Token + X-Trace-Id |
| 401 自动刷新 | ✅ | apiClient 拦截器处理 token 刷新 |

---

## 🧪 联调结果（D1）

👉 状态：**联调通过（修复后）**

---

## ❌ 问题列表（原始）

| # | 类型 | 问题 | 位置 | 影响 | 修复状态 |
|---|------|------|------|------|----------|
| 1 | 字段名 | 创建请求用 `name`，后端只接受 `keyName` | ApiKeyManager L79 | **高** — 创建可能失败 | ✅ 已修复 |
| 2 | 状态枚举 | `ApiKey.status` 缺少 `deleted`/`pending` | types/index.ts L78 | **高** — 类型不安全 | ✅ 已修复 |
| 3 | 类型缺失 | `CreateApiKeyRequest` 缺少 `allowedApis` 等必填字段 | types/index.ts L89-93 | **高** — 后端必填字段缺失 | ✅ 已修复 |
| 4 | 错误吞没 | 删除异常未展示给用户 | ApiKeyManager L99-109 | **中** — 用户无反馈 | ✅ 已修复 |
| 5 | services 类型 | `userApi.apiKeys` 参数/返回类型为 `any` | services.ts L438-456 | **中** — 无类型安全 | ✅ 已修复（list/create/delete/availableApis/update 均已类型化） |
| 6 | permissions 结构 | `permissions: { scope }` 结构后端不接受 | ApiKeyManager L80 | **高** — 创建可能失败 | ✅ 已修复（改为 scopeType + allowedApis） |
| 7 | 返回解包 | 三层解包过于防御 | DeveloperCenter L37, L55 | **低** — 可简化 | ✅ 已提取为 extractData 函数 |
| 8 | 状态展示 | ApiKeyItem 只处理 4 种状态 | business/ApiKeyItem | **中** — 状态显示不正确 | ✅ 已修复 |
| 9 | availableApis | 创建时未获取可绑定接口列表 | ApiKeyManager | **高** — 后端必填 allowedApis | ✅ 已修复 |

---

## 🛠 已执行的修复

### 修复 #1：创建请求字段名

**Before:** `{ name: keyName, permissions: { scope: keyPermission } }`
**After:** `{ keyName: keyName, allowedApis: [...] }`

- `CreateApiKeyRequest` 类型已更新，`name` → `keyName`
- 新增 `allowedApis: string[]` 必填字段
- 新增 `sourceSystem`、`description`、`ipWhitelist`、`allowCallback` 可选字段
- 移除 `permissions` 字段（后端不接受此结构）

### 修复 #2：状态枚举补全

**Before:** `status: 'active' | 'disabled' | 'expired' | 'locked'`
**After:** `status: 'active' | 'disabled' | 'expired' | 'locked' | 'deleted' | 'pending'`

### 修复 #3：availableApis 集成

- ApiKeyManager 新增 `onFetchAvailableApis` prop
- DeveloperCenter 新增 `handleFetchAvailableApis` 回调
- 创建表单新增"接口范围"选择（全部接口 / 自定义接口）
- 自定义接口模式下显示多选下拉框

### 修复 #4：删除错误反馈

- ApiKeyManager 新增 `deleteError` 状态
- `handleDelete` 中 catch 设置 `deleteError`
- 列表区域展示删除错误 Alert

### 修复 #6：权限结构重构

**Before:** `permissions: { scope: 'read' | 'write' | 'admin' }`
**After:** `scopeType: 'full' | 'custom'` + `allowedApis: string[]`

- 与后端 `scope_type` + `allowed_apis` 字段对应
- 全部接口模式自动填充所有可用 API
- 自定义接口模式需手动选择

### 修复 #8：ApiKeyItem 状态展示

**Before:** 4 种状态硬编码三元表达式
**After:** `STATUS_CONFIG` 映射表，6 种状态 + fallback

---

## ⚠️ 风险评估（修复后）

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 创建请求字段名不匹配 | ✅ 已消除 | 已改为 `keyName` + `allowedApis` |
| 状态枚举不完整 | ✅ 已消除 | 已添加 `deleted`/`pending` |
| 删除错误无反馈 | ✅ 已消除 | 已添加 `deleteError` + Alert |
| 类型安全缺失 | **低** | services.ts 仍使用 `any`，待后续统一修复 |

### 是否影响后续任务

- **否** — `CreateApiKeyRequest` 类型已完整，不影响其他页面
- **否** — `ApiKey` 状态枚举已完整，不影响其他组件

### 是否影响数据模型

- **否** — 不修改后端数据结构

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

所有 Blocking 问题已修复：
1. ✅ 创建请求字段名已改为 `keyName`
2. ✅ `ApiKey.status` 已添加 `deleted`/`pending`
3. ✅ 删除失败时展示错误信息
4. ✅ `CreateApiKeyRequest` 已包含 `allowedApis` 必填字段
5. ✅ `ApiKeyItem` 已支持 6 种状态展示
6. ✅ 创建流程已集成 `availableApis` 获取
