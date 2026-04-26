# T008 ability API 层 — 验收文档

> 任务编号：T008
> 任务名称：ability API 层
> 执行阶段：C4 → C5 → C6 → C7
> 执行日期：2026-04-25
> 执行结果：通过

---

## 1. 修改文件清单

### 修改文件

| 文件路径 | 修改类型 | 说明 |
|---------|---------|------|
| `packages/shared/src/api/services.ts` | 修改 | 新增 `userApi.abilities` + `adminApi.abilities` API 服务 |
| `packages/shared/src/types/ability.ts` | 修改 | 新增 `AbilityPublishRequest` + `AbilitySubscribeResponse` 类型 |
| `packages/shared/src/constants/index.ts` | 修改 | 新增 ability 相关路由常量 |

---

## 2. 新增组件清单

无 — 本任务为 API 层建设，无 UI 组件。

---

## 3. API变更清单

### 用户端 API (`userApi.abilities`)

| API | Method | Path | 请求类型 | 响应类型 |
|-----|--------|------|----------|----------|
| list | GET | `/api/v1/abilities` | `AbilityListParams` | `AbilityListResponse` |
| get | GET | `/api/v1/abilities/:id` | - | `AbilityDetailResponse` |
| subscribe | POST | `/api/v1/abilities/:id/subscribe` | - | `AbilitySubscribeResponse` |
| unsubscribe | POST | `/api/v1/abilities/:id/unsubscribe` | - | - |

### 管理端 API (`adminApi.abilities`)

| API | Method | Path | 请求类型 | 响应类型 |
|-----|--------|------|----------|----------|
| list | GET | `/admin-api/v1/abilities` | `AbilityListParams` | `AbilityListResponse` |
| get | GET | `/admin-api/v1/abilities/:id` | - | `AbilityDetailResponse` |
| create | POST | `/admin-api/v1/abilities` | `AbilityCreateRequest` | `Ability` |
| update | PUT | `/admin-api/v1/abilities/:id` | `AbilityUpdateRequest` | `Ability` |
| delete | DELETE | `/admin-api/v1/abilities/:id` | - | - |
| publish | POST | `/admin-api/v1/abilities/:id/publish` | `AbilityPublishRequest?` | `Ability` |
| deprecate | POST | `/admin-api/v1/abilities/:id/deprecate` | - | `Ability` |

---

## 4. 风险说明

| 风险项 | 评估 | 说明 |
|--------|------|------|
| 是否影响现网 | 否 | 新增 API 服务，不影响现有 API |
| 是否影响数据结构 | 否 | 类型定义已存在，仅补充发布/订阅类型 |
| 类型冲突 | 无 | `AbilityPublishRequest` / `AbilitySubscribeResponse` 为新定义类型 |

---

## 5. 自检结果

### C5 AI 强制自检

| 检查项 | 结果 |
|--------|------|
| 必修复问题 | 1 项已修复（`any` 类型替换为具体类型） |
| 可优化问题 | 3 项已优化（参数类型具体化） |

修复内容：
1. `adminApi.abilities.list` 参数类型：`any` → `AbilityListParams`
2. `adminApi.abilities.create` 参数类型：`any` → `AbilityCreateRequest`
3. `adminApi.abilities.update` 参数类型：`any` → `AbilityUpdateRequest`
4. `adminApi.abilities.publish` 参数类型：`any` → `AbilityPublishRequest`
5. `userApi.abilities.list` 参数类型：`any` → `AbilityListParams`

### C6 开发后自检

| 检查项 | 状态 |
|--------|------|
| 组件复用 | ✅ |
| 命名规范 | ✅ |
| API规范 | ✅ |
| 权限兼容 | ✅ |
| 异常处理 | ✅ |
| 是否影响现网 | ✅ |

**结果：Pass**

---

## 6. 验收结果

| 验收项 | 结果 |
|--------|------|
| API 是否能调用 | ✅ |
| 类型是否完整 | ✅ |
| 是否存在报错 | ✅ |
| 是否影响旧功能 | ✅ |
| 路由常量是否新增 | ✅ |

**结果：通过**

---

## 7. 备注

- 旧 `apiMarket` API 保留兼容，新 `abilities` API 为能力平台专用
- 用户端和管理端 API 路径通过前缀区分（`/api/v1` vs `/admin-api/v1`）
- 所有路径参数均使用 `encodePathParam()` 编码，符合安全规范
