# T034 callback 机制 — D1 接口联调检查报告

> 任务编号：T034
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-25
> 检查人：AI 联调审查官（资深后端架构师 + 前端联调负责人 + API契约审查官）

---

## 🧪 联调结果（D1）

**👉 状态：联调通过**

---

## 1️⃣ API 契约一致性

| API | 任务卡 URL | 实现 URL | Method | 状态 | 说明 |
|-----|-----------|----------|--------|------|------|
| 回调记录查询 | `/api/v1/callbacks` | `/api/v1/callbacks` | GET | ✅ | 完全一致 |
| 手动重试 | `/api/v1/callbacks/:id/retry` | `/api/v1/callbacks/:id/retry` | POST | ✅ | 完全一致 |

### 参数检查

| 参数 | 任务卡定义 | 实现定义 | 状态 | 说明 |
|------|-----------|----------|------|------|
| `jobId` | `string?` | `string?` | ✅ | 一致 |
| `status` | `string?` | `CallbackStatus?` | ✅ | 实现更精确（枚举类型） |
| `page` | `number?` | `number?` | ✅ | 一致 |
| `pageSize` | `number?` | `number?` | ✅ | 一致 |

**结论**：URL、Method、参数完全一致，无多余参数。

---

## 2️⃣ 返回结构匹配

| 字段 | 任务卡定义 | 实现定义 | 状态 | 说明 |
|------|-----------|----------|------|------|
| `CallbackRecord.id` | `string` | `string` | ✅ | 一致 |
| `CallbackRecord.jobId` | `string` | `string` | ✅ | 一致 |
| `CallbackRecord.callbackUrl` | `string` | `string` | ✅ | 一致 |
| `CallbackRecord.payload` | `Record<string, any>` | `Record<string, unknown>` | ⚠️ | 实现更安全 |
| `CallbackRecord.status` | `'pending' \| 'success' \| 'failed' \| 'retrying'` | `CallbackStatus` | ✅ | 一致（类型别名） |
| `CallbackRecord.attemptCount` | `number` | `number` | ✅ | 一致 |
| `CallbackRecord.maxAttempts` | `number` | `number` | ✅ | 一致 |
| `CallbackRecord.nextRetryAt` | `string?` | `string?` | ✅ | 一致 |
| `CallbackRecord.lastError` | `string?` | `string?` | ✅ | 一致 |
| `CallbackRecord.createdAt` | `string` | `string` | ✅ | 一致 |
| `CallbackRecord.completedAt` | `string?` | `string?` | ✅ | 一致 |
| `CallbackListResponse.items` | `CallbackRecord[]` | `CallbackRecord[]` | ✅ | 一致 |
| `CallbackListResponse.total` | `number` | `number` | ✅ | 一致 |
| `CallbackListResponse.page` | 未定义 | `number` | ⚠️ | 实现补充 |
| `CallbackListResponse.pageSize` | 未定义 | `number` | ⚠️ | 实现补充 |
| `CallbackRetryResponse.recordId` | `string` | `string` | ✅ | 一致 |
| `CallbackRetryResponse.status` | `string` | `CallbackStatus` | ✅ | 实现更精确 |
| `CallbackRetryResponse.message` | `string` | `string` | ✅ | 一致 |

**差异项说明**：
- `payload` 类型 `any` → `unknown`：TypeScript 类型安全最佳实践，不影响运行时
- `CallbackListResponse` 补充 `page`/`pageSize`：与分页组件对接需要，向后兼容

---

## 3️⃣ 数据结构一致性

| 检查项 | 任务卡定义 | 实现定义 | 状态 |
|--------|-----------|----------|------|
| `CallbackStatus` 枚举 | `'pending' \| 'success' \| 'failed' \| 'retrying'` | 一致 | ✅ |
| `CallbackConfig` | 有 | 有 | ✅ |
| `CallbackSignature` | 有 | 有 | ✅ |
| `CALLBACK_RETRY_INTERVALS` | `[30, 60, 120]` | 一致 | ✅ |
| `CALLBACK_TIMEOUT` | `30000` | 一致 | ✅ |
| `CALLBACK_MAX_RETRIES` | `3` | 一致 | ✅ |
| 字段命名冲突 | 无 | 无 | ✅ |
| 结构嵌套错误 | 无 | 无 | ✅ |

---

## 4️⃣ 状态流一致性

| 状态 | 任务卡定义 | 实现定义 | 状态 |
|------|-----------|----------|------|
| `pending` | ✅ | ✅ | ✅ |
| `success` | ✅ | ✅ | ✅ |
| `failed` | ✅ | ✅ | ✅ |
| `retrying` | ✅ | ✅ | ✅ |

### 状态转换验证

| 转换 | 是否合理 | 说明 |
|------|----------|------|
| `pending` → `success` | ✅ | 回调成功 |
| `pending` → `failed` | ✅ | 回调失败且重试耗尽 |
| `pending` → `retrying` | ✅ | 首次回调失败，开始重试 |
| `retrying` → `success` | ✅ | 重试成功 |
| `retrying` → `failed` | ✅ | 重试耗尽 |
| `failed` → `retrying` | ✅ | 手动触发重试 |

**结论**：状态定义前后端一致，转换逻辑合理。

---

## 5️⃣ 错误处理

| 场景 | 任务卡定义 | 实现定义 | 状态 |
|------|-----------|----------|------|
| 回调失败 | 重试 | 由后端处理重试 | ✅ |
| 重试耗尽 | 标记失败 | 由后端处理 | ✅ |
| 回调超时 | 重试 | 由后端处理 | ✅ |
| 无效 URL | 标记失败 | 由后端处理 | ✅ |
| 参数错误（400） | — | 由 `apiClient` 拦截器处理 | ✅ |
| 权限不足（403） | — | 由 `apiClient` 拦截器处理 | ✅ |
| 记录不存在（404） | — | 由 `apiClient` 拦截器处理 | ✅ |
| 空数据 | — | `ListPageShell` empty 态 | ✅ |
| loading | — | `ListPageShell` loading 态 | ✅ |
| 重试失败 | Toast 错误 | 页面错误提示 | ✅ |

---

## 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 是否通过 Service 层调用 API | ✅ | 通过 `userApi.callbacks` 调用 |
| 是否绕过业务接入层 | ❌ 否 | 未绕过，符合 XBIS 规范 |
| 路径参数是否编码 | ✅ | 使用 `encodePathParam` 统一编码 |
| 请求/响应类型是否定义 | ✅ | TypeScript 类型完整 |

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 建议 |
|------|------|------|------|------|
| 类型差异 | `CallbackRecord.payload` 任务卡为 `any`，实现为 `unknown` | `callback.ts` L15 | 无 | 实现更安全，建议任务卡同步更新 |
| 响应结构 | `CallbackListResponse` 实现含 `page`/`pageSize`，任务卡未定义 | `callback.ts` L44-L49 | 低 | 实现补充分页字段，向后兼容 |
| 响应结构 | `CallbackRetryResponse.status` 任务卡为 `string`，实现为 `CallbackStatus` | `callback.ts` L51-L55 | 低 | 实现更精确，建议任务卡同步更新 |

---

## 🛠 修改建议

### 前端修改：
1. **无需修改** — 当前实现已正确处理所有场景

### 后端修改：
1. 确认回调重试策略（指数退避，最大 3 次）
2. 确认回调超时时间（30 秒）
3. 确认回调签名验证机制

### 数据结构调整：
1. 任务卡 `CallbackRecord.payload` 类型更新为 `Record<string, unknown>`
2. 任务卡 `CallbackListResponse` 补充 `page`/`pageSize` 字段
3. 任务卡 `CallbackRetryResponse.status` 类型更新为 `CallbackStatus`

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | 低 | API 层已完整定义，后续页面任务可直接调用 |
| 是否影响数据模型 | 无 | 类型定义与任务卡一致 |
| 类型差异风险 | 无 | `unknown` 比 `any` 更安全，不影响运行时 |
| 分页字段补充 | 无 | 向后兼容，不影响现有功能 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

### 通过理由

1. **API 契约一致**：2 个 API 端点全部定义，URL/Method/参数完全一致
2. **返回结构匹配**：核心字段一致，差异项为类型安全优化
3. **数据结构一致**：枚举类型、常量与任务卡完全匹配
4. **状态流一致**：4 个状态定义前后端一致，转换逻辑合理
5. **错误处理规范**：异常场景全覆盖，由 `apiClient` 拦截器统一处理
6. **Business Services 层合规**：通过 `userApi.callbacks` 调用，未绕过业务接入层
7. **无阻塞性问题**：所有问题均为低优先级类型优化项
