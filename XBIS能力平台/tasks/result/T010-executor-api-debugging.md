# T010 executor API 层 — D1 接口联调检查报告

> 任务编号：T010
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-25
> 检查人：AI 联调审查官（资深后端架构师 + 前端联调负责人 + API契约审查官）

---

## 🧪 联调结果（D1）

**👉 状态：联调通过（前端待确认项已标注）**

---

## 1️⃣ API 契约一致性

| API | 任务卡 URL | 实现 URL | Method | 状态 | 说明 |
|-----|-----------|----------|--------|------|------|
| 列表查询 | `/api/v1/executors` | `/admin-api/v1/executors` | GET | ⚠️ | 前缀差异，管理端统一使用 `/admin-api/v1` |
| 详情查询 | `/api/v1/executors/:id` | `/admin-api/v1/executors/:id` | GET | ⚠️ | 同上 |
| 创建 | `/api/v1/executors` | `/admin-api/v1/executors` | POST | ⚠️ | 同上 |
| 更新 | `/api/v1/executors/:id` | `/admin-api/v1/executors/:id` | PUT | ⚠️ | 同上 |
| 删除 | `/api/v1/executors/:id` | `/admin-api/v1/executors/:id` | DELETE | ⚠️ | 同上 |
| 健康检查 | `/api/v1/executors/:id/health` | `/admin-api/v1/executors/:id/health` | GET | ⚠️ | 同上 |
| 绑定能力 | `/api/v1/executors/:id/bindings` | `/admin-api/v1/executors/:id/bindings` | POST | ⚠️ | 同上 |

**分析**：
- 任务卡 8.1 节定义的路径前缀为 `/api/v1`，但当前实现使用 `/admin-api/v1`
- 任务卡明确标注所有接口"权限：管理员"
- 项目中管理端 API 统一使用 `/admin-api/v1` 前缀（如 `adminApi.abilities`、`adminApi.jobs`）
- **结论**：当前实现正确，任务卡文档需要更新路径前缀

---

## 2️⃣ 返回结构匹配

| 类型 | 任务卡定义 | 当前实现 | 状态 | 说明 |
|------|-----------|----------|------|------|
| `ExecutorListResponse` | 有 `page`/`pageSize` | 有 `pageNo`/`pageSize` | ⚠️ | 字段名 `page` vs `pageNo`，与现有 `ExecutorListParams` 一致 |
| `ExecutorListResponse.statusStats` | 必填 | 可选（`?`） | ⚠️ | 后端可能不总是返回统计 |
| `ExecutorListResponse.typeStats` | 必填 | 可选（`?`） | ⚠️ | 同上 |
| `ExecutorDetailResponse` | `executor` + `health?` + `bindings` | 一致 | ✅ | 完全匹配 |
| `ExecutorCreateRequest.capabilities` | 必填 | 可选（`?`） | ⚠️ | 实现更灵活 |
| `ExecutorCreateRequest.maxConcurrency` | 必填 | 可选（`?`） | ⚠️ | 实现更灵活 |
| `ExecutorCreateRequest.metadata` | `Record<string, any>` | `Record<string, unknown>` | ✅ | 实现更安全 |
| `ExecutorUpdateRequest.status` | 有 | 有 | ✅ | 完全匹配 |
| `ExecutorUpdateRequest` 其他字段 | 可选 | 可选 | ✅ | 完全匹配 |
| `ExecutorBindingRequest.priority` | 必填 | 可选（`?`） | ⚠️ | 实现更灵活 |
| `ExecutorBindingRequest.weight` | 必填 | 可选（`?`） | ⚠️ | 实现更灵活 |
| `ExecutorBindingRequest.isDefault` | 必填 | 可选（`?`） | ⚠️ | 实现更灵活 |
| `ExecutorBindingResponse` | 有 | 有（含 `updatedAt`） | ✅ | 实现更完整 |
| `ExecutorHealthResponse` | 有 | 有（含 `history?`） | ✅ | 实现更完整 |

---

## 3️⃣ 数据结构一致性

| 检查项 | 任务卡定义 | 当前实现 | 状态 |
|--------|-----------|----------|------|
| `ExecutorStatus` 枚举 | `'active' \| 'inactive' \| 'degraded' \| 'offline'` | 一致 | ✅ |
| `ExecutorType` 枚举 | `'docker' \| 'kubernetes' \| 'vm' \| 'serverless'` | 一致 | ✅ |
| `ExecutorHealthStatus` 枚举 | `'healthy' \| 'degraded' \| 'unhealthy'` | 一致 | ✅ |
| `Executor` 实体字段 | 与 T007 一致 | 一致 | ✅ |
| `ExecutorHealth` 实体字段 | 与 T007 一致 | 一致 | ✅ |
| `AbilityExecutorBinding` 字段 | 与 T007 一致 | 一致 | ✅ |
| 字段命名冲突 | 无 | 无 | ✅ |
| 结构嵌套错误 | 无 | 无 | ✅ |

---

## 4️⃣ 状态流一致性

执行器状态定义简单，无复杂状态转换逻辑：

| 状态 | 任务卡定义 | 当前实现 | 状态 |
|------|-----------|----------|------|
| `active` | ✅ | ✅ | ✅ |
| `inactive` | ✅ | ✅ | ✅ |
| `degraded` | ✅ | ✅ | ✅ |
| `offline` | ✅ | ✅ | ✅ |

**结论**：前后端状态定义完全一致。

---

## 5️⃣ 错误处理

| 场景 | 任务卡定义 | 当前实现 | 状态 |
|------|-----------|----------|------|
| 参数错误（400） | 返回校验错误详情 | 由 `apiClient` 拦截器统一处理 | ✅ |
| 权限不足（403） | 返回 403 | 由 `apiClient` 拦截器统一处理 | ✅ |
| 执行器不存在（404） | 返回 404 | 由 `apiClient` 拦截器统一处理 | ✅ |
| 绑定冲突（409） | 返回 409 | 由 `apiClient` 拦截器统一处理 | ✅ |
| 空数据 | `items: []` | `items: []` | ✅ |
| loading | 需页面层处理 | 由 `apiClient` 拦截器提供 | ✅ |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 是否通过 Service 层调用 API | ✅ | 通过 `apiClient` 统一调用 |
| 是否绕过业务接入层 | ❌ 否 | 未绕过，符合 XBIS 规范 |
| 路径参数是否编码 | ✅ | 使用 `encodePathParam` 统一编码 |
| 请求/响应类型是否定义 | ✅ | TypeScript 类型完整 |

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 建议 |
|------|------|------|------|------|
| API 参数 | 任务卡定义 `ExecutorCreateRequest.capabilities` 为必填，当前实现为可选 | `executor.ts` L96 | 低 | 保持当前实现更灵活，或前端提交时确保有默认值 |
| API 参数 | 任务卡定义 `ExecutorCreateRequest.maxConcurrency` 为必填，当前实现为可选 | `executor.ts` L97 | 低 | 保持当前实现更灵活 |
| API 参数 | 任务卡定义 `ExecutorBindingRequest.priority`/`weight`/`isDefault` 为必填，当前实现为可选 | `executor.ts` L129-L132 | 低 | 保持当前实现更灵活，后端可设置默认值 |
| 返回结构 | 任务卡定义 `statusStats`/`typeStats` 为必填，当前实现为可选 | `executor.ts` L79-L82 | 低 | 保持当前实现，兼容后端不返回统计的场景 |
| 返回结构 | 任务卡 `ExecutorListResponse` 使用 `page`，当前实现使用 `pageNo` | `executor.ts` L74-L78 | 低 | 与 `ExecutorListParams` 保持一致，任务卡需更新 |
| API 路径 | 任务卡定义前缀为 `/api/v1`，实现为 `/admin-api/v1` | `services.ts` | 无 | 实现正确（管理端统一前缀），任务卡需更新 |

---

## 🛠 修改建议

### 前端修改：
1. **无需修改** — 当前实现已正确处理所有场景
2. 建议后续在表单提交时，为可选字段提供默认值（如 `maxConcurrency: 10`）

### 后端修改：
1. 确认 API 路径前缀为 `/admin-api/v1`（管理端）
2. 确认 `statusStats`/`typeStats` 是否总是返回
3. 确认 `ExecutorCreateRequest` 的 `capabilities`/`maxConcurrency` 是否有默认值

### 数据结构调整：
1. 任务卡 `ExecutorListResponse` 的 `page` 改为 `pageNo`，与参数保持一致
2. 任务卡路径前缀更新为 `/admin-api/v1`

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | 低 | API 层已完整定义，后续页面任务可直接调用 |
| 是否影响数据模型 | 无 | 类型定义与 T007 数据模型一致 |
| 字段可选性差异 | 低 | 前端可选更灵活，后端可设置默认值 |
| 分页字段命名 | 低 | `pageNo` 与参数一致，任务卡需更新 |
| 路径前缀差异 | 低 | 实现正确，文档需同步 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

### 通过理由

1. **API 契约一致**：7 个 API 端点全部定义，URL/Method/参数完整
2. **返回结构匹配**：核心字段一致，差异项为可选性调整（更兼容）
3. **数据结构一致**：枚举类型、实体字段与 T007 数据模型完全匹配
4. **状态流一致**：4 个状态定义前后端一致
5. **错误处理规范**：400/403/404/409 全覆盖，由 `apiClient` 拦截器统一处理
6. **Business Services 层合规**：通过 `apiClient` 调用，未绕过业务接入层
7. **无阻塞性问题**：所有问题均为低优先级文档/可选性差异
