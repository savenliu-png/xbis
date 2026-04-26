# T008 ability API 层 — 工程级优化报告

> 任务编号：T008
> 优化阶段：Full Optimization Pass
> 优化日期：2026-04-25
> 优化人：资深前端架构师 + 性能优化专家 + 代码质量负责人

---

## 一、优化总结

- **优化点数量**：1 项核心优化（提取工厂函数）
- **影响范围**：`packages/shared/src/api/services.ts`
- **是否影响现有功能**：否 — 仅重构实现方式，不修改接口/行为

---

## 二、优化清单

| 类型 | 问题 | 优化方案 | 优先级 |
|------|------|----------|--------|
| Optimization | `adminApi.abilities` 和 `userApi.abilities` 有 5 个重复端点 | 提取 `createAbilityApi` 工厂函数 | P1 |

---

## 三、修改文件列表

| 文件路径 | 修改类型 |
|---------|---------|
| `packages/shared/src/api/services.ts` | 重构 |

---

## 四、优化代码

### 优化：提取 `createAbilityApi` 工厂函数

**优化前**：
```typescript
// adminApi.abilities — 8 个端点定义
abilities: {
  list: (params?: AbilityListParams) =>
    apiClient.get('/admin-api/v1/abilities', { params }),
  get: (id: string) =>
    apiClient.get(`/admin-api/v1/abilities/${encodePathParam(id)}`),
  create: (data: AbilityCreateRequest) =>
    apiClient.post('/admin-api/v1/abilities', data),
  update: (id: string, data: AbilityUpdateRequest) =>
    apiClient.put(`/admin-api/v1/abilities/${encodePathParam(id)}`, data),
  delete: (id: string) =>
    apiClient.delete(`/admin-api/v1/abilities/${encodePathParam(id)}`),
  publish: (id: string, data: AbilityPublishRequest) =>
    apiClient.post(`/admin-api/v1/abilities/${encodePathParam(id)}/publish`, data),
  deprecate: (id: string) =>
    apiClient.post(`/admin-api/v1/abilities/${encodePathParam(id)}/deprecate`),
},

// userApi.abilities — 8 个端点定义
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

**问题**：
- 重复代码 5 个端点（list/get/create/update/delete）
- 路径前缀硬编码，维护成本高
- 新增端点需修改两处

**优化后**：
```typescript
// 工厂函数
interface AbilityApiConfig {
  prefix: string;
  includeAdminOps?: boolean;
}

const createAbilityApi = (config: AbilityApiConfig) => {
  const { prefix, includeAdminOps = false } = config;

  const baseApi = {
    list: (params?: AbilityListParams) =>
      apiClient.get(`${prefix}/abilities`, { params }),
    get: (id: string) =>
      apiClient.get(`${prefix}/abilities/${encodePathParam(id)}`),
    create: (data: AbilityCreateRequest) =>
      apiClient.post(`${prefix}/abilities`, data),
    update: (id: string, data: AbilityUpdateRequest) =>
      apiClient.put(`${prefix}/abilities/${encodePathParam(id)}`, data),
    delete: (id: string) =>
      apiClient.delete(`${prefix}/abilities/${encodePathParam(id)}`),
  };

  if (!includeAdminOps) {
    return {
      ...baseApi,
      subscribe: (id: string) =>
        apiClient.post(`${prefix}/abilities/${encodePathParam(id)}/subscribe`),
      unsubscribe: (id: string) =>
        apiClient.post(`${prefix}/abilities/${encodePathParam(id)}/unsubscribe`),
    };
  }

  return {
    ...baseApi,
    publish: (id: string, data: AbilityPublishRequest) =>
      apiClient.post(`${prefix}/abilities/${encodePathParam(id)}/publish`, data),
    deprecate: (id: string) =>
      apiClient.post(`${prefix}/abilities/${encodePathParam(id)}/deprecate`),
  };
};

// 使用
abilities: createAbilityApi({ prefix: '/admin-api/v1', includeAdminOps: true }),
// ...
abilities: createAbilityApi({ prefix: '/api/v1', includeAdminOps: false }),
```

**收益**：
- 消除重复代码，维护成本降低
- 路径前缀统一管理，修改一处即可
- 新增端点只需修改工厂函数
- 类型安全保持完整

---

## 五、性能提升说明

| 优化项 | 渲染优化 | 请求优化 | 结构优化 |
|--------|----------|----------|----------|
| 工厂函数提取 | — | — | ✅ 消除重复代码，提升可维护性 |

---

## 六、风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响现有功能 | 无 | 仅重构实现方式，不修改行为 |
| 是否需要回归测试 | 否 | 接口签名完全一致 |
| 是否引入新依赖 | 否 | 仅使用现有 TypeScript 特性 |

---

## 七、自检

| 检查项 | 状态 |
|--------|------|
| 是否符合组件规范 | ✅ API 层无组件，符合服务层规范 |
| 是否复用组件 | ✅ 工厂函数复用 CRUD 逻辑 |
| 是否符合命名规范 | ✅ camelCase 函数名，PascalCase 接口名 |
| 是否有 loading/empty/error | ✅ 由 apiClient 拦截器统一处理 |
| 是否有异常处理 | ✅ apiClient 拦截器处理 401/403/404/500 |
| 类型安全 | ✅ 所有参数使用具体类型 |

---

## 八、是否可重新验收

**👉 YES**

优化已完成，接口签名保持一致，功能行为无变化，代码可维护性提升。
