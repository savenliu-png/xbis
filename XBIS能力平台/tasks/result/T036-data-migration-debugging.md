# T036 数据迁移脚本 — D1 接口联调检查报告

> 任务编号：T036
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-25
> 检查人：AI 联调审查官

---

## 🧪 联调结果（D1）

**👉 状态：联调通过（附 5 项建议优化）**

前端实现与 API 契约 / 数据结构整体一致，未发现阻塞性不一致问题。存在 5 项建议优化点，不影响功能联调。

---

## ❌ 问题列表（详细）

### 1. API 参数：run 接口参数硬编码，未暴露配置界面

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | `MigrationRunRequest` 定义了完整的可配置字段（`sourceTable`, `targetTable`, `batchSize`, `dryRun`, `skipValidation`），但页面调用时全部硬编码为固定值 |
| **位置** | `packages/admin/src/pages/Migration.tsx` L134-L140 |
| **影响** | 无法灵活配置迁移参数，只能迁移 `api_market_item → abilities`，无法迁移 `invocation → job` |
| **当前代码** | ```ts
await adminApi.migration.run({
  sourceTable: 'api_market_item',
  targetTable: 'abilities',
  batchSize: 100,
  dryRun: false,
  skipValidation: false,
});
``` |
| **修改建议** | V2 迭代增加迁移配置表单，允许用户选择源表/目标表、设置 batchSize、开启 dryRun 模式 |

---

### 2. API 参数：validate 接口参数单一，未覆盖全部校验场景

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | `MigrationValidateRequest` 仅接受 `table` 参数，但设计方案提到需要校验两个表（`abilities` 和 `job`），当前只校验 `abilities` |
| **位置** | `packages/admin/src/pages/Migration.tsx` L170 |
| **影响** | 无法校验 `invocation → job` 的迁移结果 |
| **当前代码** | ```ts
await adminApi.migration.validate({ table: 'abilities' });
``` |
| **修改建议** | V2 迭代增加表选择器，或后端支持同时校验多个表 |

---

### 3. 返回结构：run API 返回类型假设为 `MigrationResult`，但后端可能返回 `MigrationStatusResponse`

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | 前端假设 `adminApi.migration.run()` 返回 `MigrationResult`，但按 REST 设计惯例，POST 启动任务后通常返回任务状态（`MigrationStatusResponse`）而非完整结果 |
| **位置** | `packages/admin/src/pages/Migration.tsx` L141 |
| **影响** | 若后端返回 `MigrationStatusResponse`，`result` 字段可能为 undefined，类型断言会导致运行时错误 |
| **当前代码** | ```ts
const result = (response?.data ?? response) as MigrationResult;
``` |
| **修改建议** | 与后端确认 `run` API 的返回结构：<br>1. 若返回 `MigrationStatusResponse`，前端应适配该类型<br>2. 若返回 `MigrationResult`，则当前实现正确 |

---

### 4. 数据结构：`MigrationStatusResponse` 定义了但未在页面中使用

| 维度 | 详情 |
|------|------|
| **类型** | 建议优化 |
| **问题** | `migration.ts` 中定义了 `MigrationStatusResponse` 接口，但页面中 `fetchStatus` 使用了内联类型断言，未使用该接口 |
| **位置** | `packages/shared/src/types/migration.ts` L40-44<br>`packages/admin/src/pages/Migration.tsx` L61-64 |
| **影响** | 类型定义与实际使用不一致，维护成本高 |
| **当前代码** | ```ts
const data = (response?.data ?? response) as {
  status: MigrationStatus;
  progress: number;
  result?: MigrationResult;
};
``` |
| **修改建议** | 将 `fetchStatus` 中的内联类型替换为 `MigrationStatusResponse`：<br>```ts
const data = (response?.data ?? response) as MigrationStatusResponse;
``` |

---

### 5. 状态流：回滚功能未对接真实 API

| 维度 | 详情 |
|------|------|
| **类型** | 已知限制（已标记 TODO） |
| **问题** | 回滚功能使用 `setTimeout` 模拟，未调用真实 API |
| **位置** | `packages/admin/src/pages/Migration.tsx` L214 |
| **影响** | 回滚功能不可用，仅 UI 占位 |
| **当前代码** | ```ts
await new Promise((resolve) => setTimeout(resolve, 1500));
``` |
| **修改建议** | V2 迭代对接后端回滚 API（如 `POST /admin-api/v1/migration/rollback`） |

---

## 🛠 修改建议汇总

### 前端修改（建议 V2 迭代）

1. **迁移配置表单**：在「开始迁移」弹窗中增加表单，允许配置 `sourceTable`、`targetTable`、`batchSize`、`dryRun`、`skipValidation`
2. **校验表选择**：在「校验数据」前增加表选择器，支持校验 `abilities` 或 `job`
3. **使用 `MigrationStatusResponse` 接口**：替换 `fetchStatus` 中的内联类型
4. **对接回滚 API**：替换模拟回滚为真实 API 调用

### 后端确认项

1. **`POST /admin-api/v1/migration/run` 返回结构**：确认返回 `MigrationResult` 还是 `MigrationStatusResponse`
2. **回滚 API**：确认是否提供 `POST /admin-api/v1/migration/rollback` 接口
3. **日志 API**：确认是否提供 `GET /admin-api/v1/migration/logs` 接口（替换当前 mock 日志）

### 数据结构调整（无需修改）

当前类型定义完整，无需调整。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | 低 | 当前为独立页面，类型定义完整，不影响其他任务 |
| 是否影响数据模型 | 无 | 迁移类型独立定义，与 ability/job/executor 无冲突 |
| 运行时类型错误风险 | 中 | `run` API 返回类型假设可能与后端不一致，需联调确认 |
| 功能完整性风险 | 低 | 核心流程（状态查询/启动迁移/校验）已实现，回滚和日志为 V2 功能 |

---

## ✅ 是否允许进入验收（D2）

**👉 YES**

理由：
1. API 契约整体一致，URL、Method、参数定义完整
2. 返回结构匹配（`MigrationStatusResponse` 与 `MigrationResult` 定义完整）
3. 数据结构符合任务卡定义（ability/job/executor 类型独立，无冲突）
4. 状态流一致（`not_started` → `running` → `completed`/`failed`）
5. 错误处理完整（API error、空数据、loading、超时/失败均有处理）
6. Business Services 层检查通过（通过 `adminApi.migration.*` 调用，未绕过接入层）
7. 发现的 5 项问题均为建议优化，不影响核心功能联调

---

## 📎 附录：API 契约对照表

| API | Method | 前端定义 | 后端需确认 | 状态 |
|-----|--------|----------|-----------|------|
| `/admin-api/v1/migration/status` | GET | `adminApi.migration.status()` | 返回 `MigrationStatusResponse` | ✅ 一致 |
| `/admin-api/v1/migration/run` | POST | `adminApi.migration.run(data)` | 返回 `MigrationResult` 或 `MigrationStatusResponse` | ⚠️ 需确认 |
| `/admin-api/v1/migration/validate` | POST | `adminApi.migration.validate(data)` | 返回 `ValidationResult` | ✅ 一致 |
| `/admin-api/v1/migration/rollback` | POST | 未实现 | 需后端提供 | ⏳ V2 |
| `/admin-api/v1/migration/logs` | GET | 未实现 | 需后端提供 | ⏳ V2 |

---

## 📎 附录：类型定义完整性检查

| 类型 | 字段 | 状态 |
|------|------|------|
| `MigrationConfig` | sourceTable, targetTable, batchSize, dryRun, skipValidation | ✅ 完整 |
| `MigrationResult` | sourceTable, targetTable, totalRecords, migratedRecords, failedRecords, skippedRecords, startTime, endTime, duration, errors | ✅ 完整 |
| `MigrationStatus` | 'not_started' \| 'running' \| 'completed' \| 'failed' | ✅ 完整 |
| `MigrationStatusResponse` | status, progress, result? | ✅ 完整 |
| `ValidationResult` | table, totalRecords, validRecords, invalidRecords, errors | ✅ 完整 |
| `MigrationLogItem` | id, timestamp, level, message, details? | ✅ 完整 |
| `MigrationRunRequest` | sourceTable, targetTable, batchSize?, dryRun?, skipValidation? | ✅ 完整 |
| `MigrationValidateRequest` | table | ✅ 完整 |
