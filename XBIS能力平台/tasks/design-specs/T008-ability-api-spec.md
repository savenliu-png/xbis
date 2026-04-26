# T008 ability API 层 — 工程级设计方案

> 任务卡: [T008-ability-api.md](../T008-ability-api.md)  
> 设计输入: docs/dev-rules.md, docs/component-architecture.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

本任务为**API 层**建设，无页面结构。输出物为 API 服务定义。

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

| 场景 | 处理方案 |
|------|----------|
| 参数错误 | 返回 400，前端提示参数错误 |
| 能力不存在 | 返回 404，前端提示不存在 |
| 权限不足 | 返回 403，前端提示无权限 |
| 状态冲突 | 返回 409，前端提示状态冲突 |

---

## 8. 性能优化

- API 响应支持缓存（Cache-Control）
- 列表查询支持分页
- 支持字段筛选（减少传输数据量）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 旧接口兼容 | 高 | 新接口需兼容旧 invocation 接口 | 保留旧端点，内部转发 |
| 权限控制 | 高 | 能力管理仅限管理员 | 严格权限校验 |
| 并发创建冲突 | 低 | 并发创建可能导致 ID 冲突 | 使用分布式锁 |

---

## 10. 开发步骤拆分

### Step 1: API 客户端配置（0.5 人日）
- [ ] Axios 实例配置
- [ ] 请求拦截器（Token 注入）
- [ ] 响应拦截器（错误处理）

### Step 2: 能力 API 服务（1 人日）
- [ ] 能力列表查询 API
- [ ] 能力详情查询 API
- [ ] 能力创建 API
- [ ] 能力更新 API
- [ ] 能力删除 API
- [ ] 能力发布 API

### Step 3: 类型定义（0.5 人日）
- [ ] API 请求类型
- [ ] API 响应类型
- [ ] 错误类型

### Step 4: 文档 & 测试（0.5 人日）
- [ ] API 文档（Swagger）
- [ ] 单元测试
