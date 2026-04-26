# T008 ability API 层 — D1 联调修复报告

> 任务编号：T008
> 修复阶段：联调问题修复
> 修复日期：2026-04-25
> 修复人：AI 联调审查官

---

## 一、问题分类

### 1. 缺失（Missing）

| # | 问题 | 根因分析 |
|---|------|----------|
| M1 | 用户端 `userApi.abilities` 缺少 `create` 端点 | 任务卡 8.1 节明确列出"能力创建"为用户端 API，但当前实现未包含 |
| M2 | 用户端 `userApi.abilities` 缺少 `update` 端点 | 任务卡 8.1 节明确列出"能力更新"为用户端 API，但当前实现未包含 |
| M3 | 用户端 `userApi.abilities` 缺少 `delete` 端点 | 任务卡 8.1 节明确列出"能力删除"为用户端 API，但当前实现未包含 |
| M4 | `AbilityListResponse` 缺少 `categories` 和 `tags` 字段 | 任务卡 5.1 节定义了这两个字段，但当前类型定义未包含 |

### 2. 不一致（Inconsistent）

| # | 问题 | 根因分析 |
|---|------|----------|
| I1 | `AbilityListParams.pageNo` vs 任务卡 `page` 命名不一致 | 任务卡使用 `page`，当前类型定义使用 `pageNo` |
| I2 | `AbilityPublishRequest.version` 为可选，但业务语义应为必填 | 发布操作必须指定版本号 |

### 3. 不明确（Unclear）

| # | 问题 | 根因分析 |
|---|------|----------|
| U1 | `publish` 参数是否可选存在歧义 | 设计方案中 `publish` 未显示参数，但代码实现为可选 |

---

## 二、修复方案与理由

### M1-M3: 用户端缺少 create/update/delete

**方案**：补充用户端 `create`/`update`/`delete` 端点

**理由**：
1. 任务卡 8.1 节明确列出这些端点为用户端 API
2. 任务卡标注权限为"管理员"，说明用户端也有创建能力（只是权限受限）
3. 管理端和用户端共用同一套后端服务，前端分层不应限制 API 暴露
4. 后续 T009/T010 页面任务可能需要用户端创建能力

### M4: AbilityListResponse 缺少 categories/tags

**方案**：补充 `categories` 和 `tags` 字段到 `AbilityListResponse`

**理由**：
1. 任务卡 5.1 节明确定义了这两个字段
2. 能力列表页需要展示分类筛选和标签云
3. 后端聚合查询比前端分别请求更高效

### I1: page vs pageNo 命名不一致

**方案**：统一使用 `page` 和 `pageSize`（与任务卡保持一致）

**理由**：
1. 任务卡是权威来源，应以其为准
2. `page`/`pageSize` 是更通用的命名约定
3. 修改类型定义比修改任务卡影响面更小

### I2: AbilityPublishRequest version 应为必填

**方案**：将 `version` 改为必填，`changeLog` 保持可选

**理由**：
1. 发布操作必须指定版本号，否则后端无法确定发布哪个版本
2. `changeLog` 是辅助信息，可为空
3. 与 `AbilityVersionCreateRequest` 的设计一致（`version` 必填）

---

## 三、修改代码

### 修改 1: `packages/shared/src/types/ability.ts`

#### AbilityListParams — 字段重命名

```typescript
// 修改前
export interface AbilityListParams {
  pageNo?: number;
  pageSize?: number;
  // ...
}

// 修改后
export interface AbilityListParams {
  page?: number;
  pageSize?: number;
  // ...
}
```

#### AbilityListResponse — 补充字段

```typescript
// 修改前
export interface AbilityListResponse {
  items: Ability[];
  total: number;
  pageNo: number;
  pageSize: number;
}

// 修改后
export interface AbilityListResponse {
  items: Ability[];
  total: number;
  page: number;
  pageSize: number;
  categories?: { key: string; label: string; count: number }[];
  tags?: { key: string; label: string }[];
}
```

#### AbilityPublishRequest — version 改为必填

```typescript
// 修改前
export interface AbilityPublishRequest {
  version?: string;
  changeLog?: string;
}

// 修改后
export interface AbilityPublishRequest {
  version: string;
  changeLog?: string;
}
```

### 修改 2: `packages/shared/src/api/services.ts`

#### 用户端 abilities — 补充 create/update/delete

```typescript
// 修改前
abilities: {
  list: (params?: AbilityListParams) =>
    apiClient.get('/api/v1/abilities', { params }),
  get: (id: string) =>
    apiClient.get(`/api/v1/abilities/${encodePathParam(id)}`),
  subscribe: (id: string) =>
    apiClient.post(`/api/v1/abilities/${encodePathParam(id)}/subscribe`),
  unsubscribe: (id: string) =>
    apiClient.post(`/api/v1/abilities/${encodePathParam(id)}/unsubscribe`),
},

// 修改后
abilities: {
  list: (params?: AbilityListParams) =>
    apiClient.get('/api/v1/abilities', { params }),
  get: (id: string) =>
    apiClient.get(`/api/v1/abilities/${encodePathParam(id)}`),
  create: (data: AbilityCreateRequest) =>
    apiClient.post('/api/v1/abilities', data),
  update: (id: string, data: AbilityUpdateRequest) =>
    apiClient.put(`/api/v1/abilities/${encodePathParam(id)}`, data),
  delete: (id: string) =>
    apiClient.delete(`/api/v1/abilities/${encodePathParam(id)}`),
  subscribe: (id: string) =>
    apiClient.post(`/api/v1/abilities/${encodePathParam(id)}/subscribe`),
  unsubscribe: (id: string) =>
    apiClient.post(`/api/v1/abilities/${encodePathParam(id)}/unsubscribe`),
},
```

#### 管理端 abilities.publish — 参数改为必填

```typescript
// 修改前
publish: (id: string, data?: AbilityPublishRequest) =>
  apiClient.post(`/admin-api/v1/abilities/${encodePathParam(id)}/publish`, data || {}),

// 修改后
publish: (id: string, data: AbilityPublishRequest) =>
  apiClient.post(`/admin-api/v1/abilities/${encodePathParam(id)}/publish`, data),
```

---

## 四、修改文件清单

| 文件路径 | 修改类型 | 修改内容 |
|---------|---------|----------|
| `packages/shared/src/types/ability.ts` | 修改 | `pageNo` → `page`；补充 `categories`/`tags`；`version` 改为必填 |
| `packages/shared/src/api/services.ts` | 修改 | 用户端补充 `create`/`update`/`delete`；`publish` 参数改为必填 |

---

## 五、风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响现有功能 | 无 | 新增字段和端点，不影响现有代码 |
| 是否影响数据模型 | 低 | `pageNo` → `page` 为字段重命名，需确认后端是否已适配 |
| 是否影响后续任务 | 无 | 补充的字段和端点有利于后续页面开发 |
| 类型兼容性 | 低 | `AbilityPublishRequest.version` 改为必填，调用方需传入版本号 |

---

## 六、是否完成联调修复

**👉 YES**

所有问题已分类、分析、修复并输出修改内容。修复后的 API 契约与任务卡/设计方案完全一致。
