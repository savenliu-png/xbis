# T026 管理端能力管理列表 - 接口联调检查报告（D1）

## 🧪 联调结果（D1）

👉 状态：**联调通过**

---

## 1️⃣ API契约一致性

### 1.1 能力列表查询

| 检查项 | 任务卡/设计方案 | 代码实现 | 一致性 |
|--------|----------------|---------|--------|
| URL | `GET /admin-api/v1/abilities` | `apiClient.get('/admin-api/v1/abilities', { params })` | ✅ 一致 |
| Method | GET | GET | ✅ 一致 |
| 参数类型 | `AdminAbilityFilter` | `AbilityListParams`（含 `isEnabled`） | ✅ 一致（已修复） |

### 1.2 能力状态切换

| 检查项 | 任务卡/设计方案 | 代码实现 | 一致性 |
|--------|----------------|---------|--------|
| URL | `PUT /admin-api/v1/abilities/:id/status` | `apiClient.put('/admin-api/v1/abilities/${id}/status', data)` | ✅ 一致 |
| Method | PUT | PUT | ✅ 一致 |
| 请求体 | `AbilityStatusToggleRequest { isEnabled }` | `AbilityStatusToggleRequest { isEnabled }` | ✅ 一致 |

### 1.3 能力删除

| 检查项 | 任务卡/设计方案 | 代码实现 | 一致性 |
|--------|----------------|---------|--------|
| URL | `DELETE /admin-api/v1/abilities/:id` | `apiClient.delete('/admin-api/v1/abilities/${id}')` | ✅ 一致 |
| Method | DELETE | DELETE | ✅ 一致 |

### 1.4 批量删除

| 检查项 | 任务卡/设计方案 | 代码实现 | 一致性 |
|--------|----------------|---------|--------|
| URL | `POST /admin-api/v1/abilities/batch-delete` | `apiClient.post('/admin-api/v1/abilities/batch-delete', { ids })` | ✅ 一致 |
| Method | POST | POST | ✅ 一致 |
| 请求体 | `BatchDeleteRequest { ids }` | `{ ids }` | ✅ 一致 |

### 1.5 批量状态更新

| 检查项 | 任务卡/设计方案 | 代码实现 | 一致性 |
|--------|----------------|---------|--------|
| URL | `POST /admin-api/v1/abilities/batch-status` | `apiClient.post('/admin-api/v1/abilities/batch-status', data)` | ✅ 一致 |
| Method | POST | POST | ✅ 一致 |
| 请求体 | `BatchStatusRequest { ids, isEnabled }` | `BatchStatusRequest { ids, isEnabled }` | ✅ 一致 |

---

## 2️⃣ 返回结构匹配

### 2.1 列表接口响应

| 检查项 | 说明 | 一致性 |
|--------|------|--------|
| 外层 `success` | `ApiResponse.success` 由 client.ts 保证 | ✅ |
| 外层 `data` | `ApiResponse.data` 由 client.ts 返回 | ✅ |
| 内层 `items` | `AdminAbilityListResponse.items` | ✅ |
| 内层 `total` | `AdminAbilityListResponse.total` | ✅ |

### 2.2 AdminAbilityItem 字段匹配

| 字段 | 类型 | 一致性 |
|------|------|--------|
| id | string | ✅ |
| abilityId | string | ✅ |
| name | string | ✅ |
| displayName | string | ✅ |
| category | string | ✅ |
| status | AbilityStatus | ✅ |
| isEnabled | boolean | ✅ |
| version | string | ✅ |
| callCount | number | ✅ |
| successRate | number | ✅ |
| averageLatency | number | ✅ |
| subscriberCount | number? | ✅ 新增 |
| activeJobCount | number? | ✅ 新增 |
| totalJobCount | number? | ✅ 新增 |
| createdAt | string | ✅ |
| updatedAt | string | ✅ |

---

## 3️⃣ 数据结构一致性

### 3.1 AbilityStatus 枚举

| 值 | 一致性 |
|----|--------|
| published | ✅ |
| draft | ✅ |
| deprecated | ✅ |

### 3.2 AdminAbilityFilter vs AbilityListParams

| 字段 | AdminAbilityFilter | AbilityListParams | 一致性 |
|------|--------------------|-------------------|--------|
| keyword | ✅ | ✅ | ✅ |
| category | ✅ | ✅ | ✅ |
| status | ✅ | ✅ | ✅ |
| isEnabled | ✅ | ✅ | ✅ 已修复 |
| page | ✅ | ✅ | ✅ |
| pageSize | ✅ | ✅ | ✅ |

### 3.3 AbilityStatusToggleRequest

| 字段 | 一致性 |
|------|--------|
| isEnabled | ✅ |

注：`abilityId` 已从 body 中移除（通过 URL path 传递），设计方案已同步更新。

---

## 4️⃣ 状态流一致性

| 状态转换 | 前端 | 后端 | 一致性 |
|---------|------|------|--------|
| draft → published | ✅ | ✅ | ✅ |
| published → deprecated | ✅ | ✅ | ✅ |
| 启用/禁用切换 | ✅ | ✅ | ✅ |

---

## 5️⃣ 错误处理

| 场景 | 处理方式 | 一致性 |
|------|---------|--------|
| 列表加载失败 | try/catch → error 状态 | ✅ |
| 状态切换失败 | try/catch → 回滚 + error | ✅ |
| 单个删除失败 | try/catch → 弹窗保持打开 | ✅ 已修复 |
| 批量操作失败 | try/catch → errorMessage | ✅ |
| 筛选无结果 | Empty 组件 | ✅ 已修复 |

---

## 6️⃣ Business Services 层检查

| API 调用 | 是否通过 Service 层 | 合规性 |
|---------|-------------------|--------|
| 能力列表 | ✅ adminApi.abilities.list() | ✅ |
| 状态切换 | ✅ adminApi.abilities.toggleStatus() | ✅ |
| 单个删除 | ✅ adminApi.abilities.delete() | ✅ |
| 批量删除 | ✅ adminApi.abilities.batchDelete() | ✅ |
| 批量状态 | ✅ adminApi.abilities.batchStatus() | ✅ |
| 分类列表 | ✅ adminApi.categories.list() | ✅ |

---

## 契约修复记录

| 编号 | 问题 | 修复内容 | 修复文件 |
|------|------|---------|---------|
| CF-001 | AbilityStatusToggleRequest 含多余 abilityId | 移除 abilityId，设计方案同步更新 | ability.ts, design-final.md |
| CF-002 | AdminAbilityFilter 与 AbilityListParams 双类型 | 保留双类型（TypeScript 结构化类型兼容） | 无需修改 |
| CF-003 | toggleStatus 使用内联类型 | 改用 AbilityStatusToggleRequest | services.ts |
| CF-004 | batchStatus 使用内联类型 | 改用 BatchStatusRequest | services.ts |
| CF-005 | 设计方案内部矛盾 | 更新设计方案，移除 abilityId | design-final.md |
| CF-006 | 删除弹窗缺少关联影响展示 | AdminAbilityItem 添加关联统计字段 + 弹窗展示 | ability.ts, AbilityListPage.tsx |
| CF-007 | AdminAbilityListResponse 命名不一致 | 保留 AdminAbilityListResponse，更新设计方案 | design-final.md |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**
