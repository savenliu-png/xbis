# T010 executor API 层 — 最终可开发设计方案

> 任务卡: [T010-executor-api.md](../T010-executor-api.md)
> 设计输入: docs/dev-rules.md, docs/component-architecture.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为 **API 层** 建设，无页面结构。输出物为 API 服务定义。

---

## 2. 组件拆分

无 UI 组件。本任务输出为 API 服务定义。

### API 服务定义

| API | Method | Path | 请求类型 | 响应类型 |
|-----|--------|------|----------|----------|
| 执行器列表查询 | GET | `/api/v1/executors` | ExecutorListParams | ExecutorListResponse |
| 执行器详情查询 | GET | `/api/v1/executors/:id` | - | ExecutorDetailResponse |
| 执行器创建 | POST | `/api/v1/executors` | ExecutorCreateRequest | Executor |
| 执行器更新 | PUT | `/api/v1/executors/:id` | ExecutorUpdateRequest | Executor |
| 执行器删除 | DELETE | `/api/v1/executors/:id` | - | - |
| 执行器健康检查 | GET | `/api/v1/executors/:id/health` | - | ExecutorHealthResponse |
| 执行器绑定能力 | POST | `/api/v1/executors/:id/bindings` | ExecutorBindingRequest | ExecutorBindingResponse |

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

```typescript
export const executorApi = {
  // 查询执行器列表
  list: () => apiClient.get('/api/v1/executors'),

  // 查询执行器详情
  get: (id: string) => apiClient.get(`/api/v1/executors/${id}`),

  // 查询执行器健康状态
  getHealth: (id: string) => apiClient.get(`/api/v1/executors/${id}/health`),
};
```

### 调用位置

- 管理端页面组件的 `useEffect` 中
- 自定义 Hook 中（推荐）

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
| 404 | 执行器不存在 | 返回 404 |
| 409 | 绑定冲突 | 返回 409 |

---

## 8. 性能优化

- API 响应支持缓存（Cache-Control）
- 列表查询支持分页
- 健康检查数据支持缓存（30 秒）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 绑定冲突 | 中 | 同一能力绑定多个默认执行器 | 绑定前校验默认执行器 |
| 健康检查数据膨胀 | 中 | 健康检查数据可能频繁更新 | 自动清理策略 |
| 权限控制 | 低 | 执行器管理仅限管理员 | 严格管理员权限校验 |
| 健康数据实时性 | 低 | 健康数据可能延迟 | 明确缓存策略 |

---

## 10. 开发步骤拆分

### Step 1: API 定义（0.5 人日）
- [ ] list / get / getHealth

### Step 2: 类型定义（0.5 人日）
- [ ] API 请求类型
- [ ] API 响应类型
- [ ] 错误类型

### Step 3: 文档 & 测试（0.5 人日）
- [ ] API 文档（Swagger）
- [ ] 单元测试
