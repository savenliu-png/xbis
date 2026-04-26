# T008 ability API 层 — 最终可开发设计方案

> 任务卡: [T008-ability-api.md](../T008-ability-api.md)
> 设计输入: docs/dev-rules.md, docs/component-architecture.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为 **API 层** 建设，无页面结构。输出物为 API 服务定义。

```
packages/shared/src/api/
├── client.ts                       # Axios 实例 + 拦截器
├── services.ts                     # API 服务定义
└── hooks/                          # React Query hooks（未来）
    ├── useAbilities.ts
    ├── useJobs.ts
    └── useUser.ts
```

---

## 2. 组件拆分

无 UI 组件。本任务输出为 API 服务定义。

### API 服务定义

| API | Method | Path | 请求类型 | 响应类型 |
|-----|--------|------|----------|----------|
| 能力列表查询 | GET | `/api/v1/abilities` | AbilityListParams | AbilityListResponse |
| 能力详情查询 | GET | `/api/v1/abilities/:id` | - | AbilityDetailResponse |
| 能力创建 | POST | `/api/v1/abilities` | AbilityCreateRequest | Ability |
| 能力更新 | PUT | `/api/v1/abilities/:id` | AbilityUpdateRequest | Ability |
| 能力删除 | DELETE | `/api/v1/abilities/:id` | - | - |
| 能力发布 | POST | `/api/v1/abilities/:id/publish` | AbilityPublishRequest | Ability |

---

## 3. 数据流

```
前端页面 ──► API 服务 ──► Axios ──► 后端服务
```

---

## 4. 状态管理

无状态管理。API 服务为纯函数。

---

## 5. API 调用

### 用户端 API

```typescript
// 能力列表（带筛选）
interface AbilityListParams {
  keyword?: string;
  category?: string;
  status?: string;
  page?: number;
  pageSize?: number;
}

export const abilityApi = {
  // 查询能力列表
  list: (params?: AbilityListParams) =>
    apiClient.get('/api/v1/abilities', { params }),

  // 查询能力详情
  get: (id: string) =>
    apiClient.get(`/api/v1/abilities/${id}`),

  // 订阅能力
  subscribe: (id: string) =>
    apiClient.post(`/api/v1/abilities/${id}/subscribe`),

  // 取消订阅
  unsubscribe: (id: string) =>
    apiClient.post(`/api/v1/abilities/${id}/unsubscribe`),
};
```

### 管理端 API

```typescript
export const adminAbilityApi = {
  // 查询能力列表（管理端）
  list: (params?: AdminAbilityListParams) =>
    apiClient.get('/admin-api/v1/abilities', { params }),

  // 更新能力
  update: (id: string, data: AbilityUpdateRequest) =>
    apiClient.put(`/admin-api/v1/abilities/${id}`, data),

  // 删除能力
  delete: (id: string) =>
    apiClient.delete(`/admin-api/v1/abilities/${id}`),

  // 发布能力
  publish: (id: string) =>
    apiClient.post(`/admin-api/v1/abilities/${id}/publish`),

  // 下架能力
  deprecate: (id: string) =>
    apiClient.post(`/admin-api/v1/abilities/${id}/deprecate`),
};
```

### 调用位置

- 页面组件的 `useEffect` 中
- 自定义 Hook 中（推荐）
- 事件处理函数中

### 错误处理

- API 错误由 `apiClient` 拦截器统一处理
- 页面层只需捕获错误并更新状态

---

## 6. 用户交互流程

无直接用户交互。API 由页面层调用。

---

## 7. 边界与异常

| 错误码 | 场景 | 处理方式 |
|--------|------|----------|
| 400 | 参数错误 | 返回校验错误详情 |
| 403 | 权限不足 | 返回 403 |
| 404 | 能力不存在 | 返回 404 |

---

## 8. 性能优化

- API 响应缓存（5 分钟）
- 列表分页加载
- 支持字段筛选（减少传输数据量）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 旧接口兼容 | 高 | 新接口需兼容旧 invocation 接口 | 保留旧端点，内部转发 |
| 权限控制 | 高 | 能力管理仅限管理员 | 严格权限校验 |
| 并发创建冲突 | 低 | 并发创建可能导致 ID 冲突 | 使用分布式锁 |
| API 版本冲突 | 低 | 用户端/管理端 API 路径冲突 | 使用前缀区分 |

---

## 10. 开发步骤拆分

### Step 1: API 客户端配置（0.5 人日）
- [ ] Axios 实例配置
- [ ] 请求拦截器（Token 注入）
- [ ] 响应拦截器（错误处理）

### Step 2: 用户端 API（0.5 人日）
- [ ] list / get
- [ ] subscribe / unsubscribe
- [ ] 筛选参数

### Step 3: 管理端 API（0.5 人日）
- [ ] admin list / update / delete
- [ ] publish / deprecate

### Step 4: 类型定义（0.5 人日）
- [ ] API 请求类型
- [ ] API 响应类型
- [ ] 错误类型

### Step 5: 文档 & 测试（0.5 人日）
- [ ] API 文档（Swagger）
- [ ] 单元测试
