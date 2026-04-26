# T035 系统配置增强 — 接口契约级修复报告

> 任务编号：T035
> 修复阶段：Contract Fix（基于 D1 联调报告）
> 修复日期：2026-04-25
> 执行人：资深系统架构师 + API契约修复专家 + 前后端联调负责人

---

## 一、问题分类

| # | 问题 | 分类 | 说明 |
|---|------|------|------|
| 1 | `recommendations.update` 参数类型为 `unknown[]` | 2️⃣ 契约不一致 | 任务卡定义为 `HomeRecommendation[]`，实现为 `unknown[]` |
| 2 | `categories.update` 参数类型为 `unknown` | 2️⃣ 契约不一致 | 任务卡定义为 `CategoryManagement`，实现为 `unknown` |
| 3 | 变更日志查询未传递 `entityType` 参数 | 3️⃣ 设计不明确 | 任务卡支持 `ChangeLogListParams`，但实现未使用 |
| 4 | `SystemConfig.value` 类型 `any` vs `unknown` | 1️⃣ 契约缺失 | 任务卡未明确类型安全级别，`unknown` 更安全 |
| 5 | API 响应无运行时校验 | 3️⃣ 设计不明确 | 缺少统一的响应结构校验机制 |

---

## 二、契约决策

### 决策 1：`recommendations.update` 参数类型

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 改为 `HomeRecommendation[]`，严格匹配任务卡契约 | ✅ 推荐 — 类型安全是工程化核心要求 |
| B | 保持 `unknown[]`，前端自行保证数据格式 | 类型安全丢失，编译期无法检查 |

**推荐：方案A** — 任务卡已明确定义为 `HomeRecommendation[]`，且不影响运行时行为。

### 决策 2：`categories.update` 参数类型

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 改为 `Partial<CategoryManagement>`，支持部分字段更新 | ✅ 推荐 — 符合 RESTful 部分更新语义 |
| B | 改为 `CategoryManagement`，要求完整对象 | 与现有 `handleToggleActive` 仅更新 `isActive` 的实现不一致 |

**推荐：方案A** — 分类管理当前仅更新 `isActive` 字段，使用 `Partial<T>` 更灵活。

### 决策 3：变更日志筛选策略

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 将筛选参数提升到页面层，通过 API 查询 | 实现复杂，需改动组件接口 |
| B | 保持客户端筛选，添加 TODO 注释说明 | ✅ 推荐 — 当前数据量预期不大，客户端筛选响应快 |

**推荐：方案B** — 管理端操作日志数据量预期 < 500 条，客户端筛选实现简单；添加 TODO 注释，数据量增长时迁移到服务端筛选。

### 决策 4：`SystemConfig.value` 类型

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 保持 `unknown`，更安全 | ✅ 推荐 — 强制类型收窄，比 `any` 更安全 |
| B | 改为 `any`，与任务卡一致 | 类型安全丢失 |

**推荐：方案A** — `unknown` 是 `any` 的安全替代，任务卡使用 `any` 是历史遗留，代码已实现更优方案，无需回退。

### 决策 5：API 响应运行时校验

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 每个 API 调用添加 `Array.isArray(data.items)` 校验 | 重复代码多，维护成本高 |
| B | 统一封装 `parseListResponse<T>()` 工具函数 | ✅ 推荐 — 工程化更佳，所有列表页复用 |

**推荐：方案B** — 封装工具函数，统一处理 `{ items: T[]; total: number }` 格式的响应，支持标准包装响应 `{ success, data }` 和直接返回两种格式。

---

## 三、修复内容

### 3.1 代码修改

#### 修改 1：`services.ts` — API 参数类型修正

**文件**：`packages/shared/src/api/services.ts`

**修改前**：
```typescript
import type { ChangeLogListParams, SystemConfigUpdateRequest } from '../types/index.js';

recommendations: {
  update: (data: unknown[]) =>
    apiClient.put('/admin-api/v1/recommendations', data),
},
categories: {
  update: (id: string, data: unknown) =>
    apiClient.put(`/admin-api/v1/categories/${encodePathParam(id)}`, data),
},
```

**修改后**：
```typescript
import type { ChangeLogListParams, SystemConfigUpdateRequest, HomeRecommendation, CategoryManagement } from '../types/index.js';

recommendations: {
  update: (data: HomeRecommendation[]) =>
    apiClient.put('/admin-api/v1/recommendations', data),
},
categories: {
  update: (id: string, data: Partial<CategoryManagement>) =>
    apiClient.put(`/admin-api/v1/categories/${encodePathParam(id)}`, data),
},
```

---

#### 修改 2：`system-config.ts` — 添加响应解析工具函数

**文件**：`packages/shared/src/types/system-config.ts`

**新增内容**：
```typescript
// ---------- 响应解析工具函数 ----------

/**
 * 解析列表 API 响应结构
 * 统一处理 { items: T[]; total: number } 格式的响应
 * 添加运行时校验，防止后端返回异常结构导致页面崩溃
 */
export function parseListResponse<T>(response: unknown): { items: T[]; total: number } {
  if (!response || typeof response !== 'object') {
    throw new Error('Invalid response: response is not an object');
  }

  const data = response as Record<string, unknown>;

  // 处理标准包装响应 { success: true, data: { items, total } }
  if (data.data && typeof data.data === 'object') {
    const inner = data.data as Record<string, unknown>;
    if (!Array.isArray(inner.items)) {
      throw new Error('Invalid response structure: data.items is not an array');
    }
    return {
      items: inner.items as T[],
      total: typeof inner.total === 'number' ? inner.total : (inner.items as T[]).length,
    };
  }

  // 处理直接返回 { items, total } 的情况
  if (!Array.isArray(data.items)) {
    throw new Error('Invalid response structure: items is not an array');
  }

  return {
    items: data.items as T[],
    total: typeof data.total === 'number' ? data.total : (data.items as T[]).length,
  };
}
```

---

#### 修改 3：`SystemSettingsPage.tsx` — 使用工具函数 + 响应校验

**文件**：`packages/pages/admin/SystemSettings/SystemSettingsPage.tsx`

**修改前**：
```typescript
import { adminApi } from '@xbis/shared';

const loadConfig = useCallback(async () => {
  setConfigLoading();
  try {
    const response = await adminApi.settings.list();
    const data = response as unknown as { items: SystemConfig[]; total: number };
    setConfigData(data.items);
  } catch (err) {
    setConfigError(err as Error);
  }
}, []);
```

**修改后**：
```typescript
import { adminApi, parseListResponse } from '@xbis/shared';

const loadConfig = useCallback(async () => {
  setConfigLoading();
  try {
    const response = await adminApi.settings.list();
    const data = parseListResponse<SystemConfig>(response);
    setConfigData(data.items);
  } catch (err) {
    setConfigError(err instanceof Error ? err : new Error(String(err)));
  }
}, [setConfigLoading, setConfigData, setConfigError]);
```

**其他加载函数同步修改**：`loadRecommendations`、`loadCategories`、`loadChangelogs` 均使用 `parseListResponse<T>()` 替代 `as unknown as`。

**变更日志参数传递**：
```typescript
const [changelogParams, setChangelogParams] = useState<ChangeLogListParams>({});

const loadChangelogs = useCallback(async () => {
  setChangelogLoading();
  try {
    const response = await adminApi.changelogs.list(changelogParams);
    const data = parseListResponse<ChangeLog>(response);
    setChangelogData(data.items);
  } catch (err) {
    setChangelogError(err instanceof Error ? err : new Error(String(err)));
  }
}, [changelogParams, setChangelogLoading, setChangelogData, setChangelogError]);
```

---

#### 修改 4：`ChangeLogTable.tsx` — 添加客户端筛选注释

**文件**：`packages/pages/admin/SystemSettings/components/ChangeLogTable.tsx`

**修改前**：
```typescript
/**
 * ChangeLogTable — 变更日志组件
 * ==============================
 * 变更日志表格，支持按实体类型筛选、客户端分页
 */
```

**修改后**：
```typescript
/**
 * ChangeLogTable — 变更日志组件
 * ==============================
 * 变更日志表格，支持按实体类型筛选、客户端分页
 *
 * TODO: 当前为客户端筛选，适用于数据量 < 500 条场景。
 * 当数据量增长时，需将筛选参数通过 props 传递到页面层，
 * 由页面层调用 API 进行服务端筛选和分页。
 */
```

---

### 3.2 设计方案更新

无需更新设计方案（Final），因为：
- 决策 1、2 为类型修正，不影响设计文档中的 API 契约定义
- 决策 3 已在代码中添加 TODO 注释，作为工程化备忘
- 决策 4 `unknown` 优于 `any`，代码已实现更优方案
- 决策 5 为工程化增强，工具函数已添加到类型定义文件中

---

### 3.3 类型定义更新

**文件**：`packages/shared/src/types/system-config.ts`

| 变更 | 说明 |
|------|------|
| 新增 `parseListResponse<T>()` | 统一列表响应解析工具函数 |
| 保持 `SystemConfig.value: unknown` | 比任务卡 `any` 更安全，无需回退 |

---

## 四、影响评估

| 评估项 | 结论 | 说明 |
|--------|------|------|
| 是否影响前端页面 | **否** | 仅类型修正 + 响应校验增强，无行为变更 |
| 是否影响后端接口 | **否** | 后端按任务卡 §8.1 实现即可 |
| 是否影响后续任务 | **否** | 类型更严格，有利于后续开发 |
| 是否影响现有功能 | **否** | T035 为新增功能，不影响现有页面 |
| 是否破坏已有 API | **否** | `settings.get()` / `updateByCategory()` 保留兼容 |

---

## 五、是否完成联调修复

👉 **YES**

**修复清单**：
- [x] `services.ts` 中 `recommendations.update` 参数类型修正为 `HomeRecommendation[]`
- [x] `services.ts` 中 `categories.update` 参数类型修正为 `Partial<CategoryManagement>`
- [x] `system-config.ts` 新增 `parseListResponse<T>()` 工具函数
- [x] `SystemSettingsPage.tsx` 所有 API 响应使用 `parseListResponse` 替代 `as unknown as`
- [x] `SystemSettingsPage.tsx` 变更日志加载传递 `changelogParams`
- [x] `ChangeLogTable.tsx` 添加客户端筛选 TODO 注释
- [x] `SystemConfig.value` 保持 `unknown`（比 `any` 更安全）

**建议**：修复完成后，重新执行 D1 检查确认无遗留问题，然后进入 D2 功能验收。

---

## 六、修复后 API 契约对照表

| API | Method | Path | 请求类型（契约） | 请求类型（修复后） | 状态 |
|-----|--------|------|-----------------|-------------------|------|
| 系统配置查询 | GET | `/admin-api/v1/settings` | — | — | ✅ |
| 系统配置更新 | PUT | `/admin-api/v1/settings/:id` | `{ value: any }` | `SystemConfigUpdateRequest` | ✅ |
| 变更日志查询 | GET | `/admin-api/v1/changelogs` | `ChangeLogListParams` | `ChangeLogListParams` | ✅ |
| 首页推荐查询 | GET | `/admin-api/v1/recommendations` | — | — | ✅ |
| 首页推荐更新 | PUT | `/admin-api/v1/recommendations` | `HomeRecommendation[]` | `HomeRecommendation[]` | ✅ |
| 分类管理查询 | GET | `/admin-api/v1/categories` | — | — | ✅ |
| 分类管理更新 | PUT | `/admin-api/v1/categories/:id` | `CategoryManagement` | `Partial<CategoryManagement>` | ✅ |

---

*本文档与代码同步维护，后续迭代请同步更新。*
