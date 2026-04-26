# T035 系统配置增强 — 接口联调检查报告（D1）

> 任务编号：T035
> 检查阶段：D1 — 接口联调检查
> 检查日期：2026-04-25
> 执行人：资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 🧪 联调结果（D1）

👉 **状态：前端待修**

说明：发现 5 项前端实现与 API 契约不一致的问题，需修复后方可进入 D2 验收。

---

## ❌ 问题列表

### 问题 1：API 参数类型不匹配 — 推荐更新接口

| 属性 | 内容 |
|------|------|
| **类型** | API参数 |
| **问题** | `adminApi.recommendations.update` 请求类型为 `unknown[]`，但任务卡定义为 `HomeRecommendation[]` |
| **位置** | `services.ts:360` |
| **影响** | 类型安全丢失，编译期无法检查请求数据合法性 |

**契约定义（任务卡 §8.1）：**
```
Method: PUT
Path: /admin-api/v1/recommendations
请求类型: HomeRecommendation[]
```

**当前实现：**
```typescript
recommendations: {
  update: (data: unknown[]) =>
    apiClient.put('/admin-api/v1/recommendations', data),
}
```

**修改建议（前端）：**
```typescript
recommendations: {
  update: (data: HomeRecommendation[]) =>
    apiClient.put('/admin-api/v1/recommendations', data),
}
```

---

### 问题 2：API 参数类型不匹配 — 分类更新接口

| 属性 | 内容 |
|------|------|
| **类型** | API参数 |
| **问题** | `adminApi.categories.update` 请求类型为 `unknown`，但任务卡定义为 `CategoryManagement` |
| **位置** | `services.ts:366` |
| **影响** | 类型安全丢失，编译期无法检查请求数据合法性 |

**契约定义（任务卡 §8.1）：**
```
Method: PUT
Path: /admin-api/v1/categories/:id
请求类型: CategoryManagement
响应类型: CategoryManagement
```

**当前实现：**
```typescript
categories: {
  update: (id: string, data: unknown) =>
    apiClient.put(`/admin-api/v1/categories/${encodePathParam(id)}`, data),
}
```

**修改建议（前端）：**
```typescript
categories: {
  update: (id: string, data: Partial<CategoryManagement>) =>
    apiClient.put(`/admin-api/v1/categories/${encodePathParam(id)}`, data),
}
```

---

### 问题 3：API 请求参数缺失 — 变更日志查询

| 属性 | 内容 |
|------|------|
| **类型** | API参数 |
| **问题** | `ChangeLogTable` 组件仅做客户端筛选，未将 `entityType` 参数传递给 API |
| **位置** | `SystemSettingsPage.tsx:87`, `ChangeLogTable.tsx:29-40` |
| **影响** | 后端返回全量数据，前端内存筛选；大数据量时性能差，且无法利用后端分页 |

**契约定义（任务卡 §8.1）：**
```
Method: GET
Path: /admin-api/v1/changelogs
请求类型: { entityType?: string; page?: number; pageSize?: number }
```

**当前实现（页面层）：**
```typescript
const loadChangelogs = useCallback(async () => {
  const response = await adminApi.changelogs.list(); // 未传参数
}, []);
```

**当前实现（组件层）：**
```typescript
const ChangeLogTable: React.FC<ChangeLogTableProps> = ({ data, loading }) => {
  const [selectedType, setSelectedType] = useState('all');
  const filteredData = selectedType === 'all'
    ? data
    : data.filter((item) => item.entityType === selectedType); // 仅客户端筛选
};
```

**修改建议（前端）：**

方案 A（推荐）：将筛选参数提升到页面层，通过 API 查询
```typescript
// SystemSettingsPage.tsx
const [changelogParams, setChangelogParams] = useState<ChangeLogListParams>({});

const loadChangelogs = useCallback(async () => {
  setChangelogLoading();
  try {
    const response = await adminApi.changelogs.list(changelogParams);
    const data = response as unknown as ChangeLogListResponse;
    setChangelogData(data.items);
  } catch (err) {
    setChangelogError(err as Error);
  }
}, [changelogParams, setChangelogLoading, setChangelogData, setChangelogError]);

// ChangeLogTable.tsx
interface ChangeLogTableProps {
  data: ChangeLog[];
  loading: boolean;
  params: ChangeLogListParams;
  onParamsChange: (params: ChangeLogListParams) => void;
}
```

方案 B（保持客户端筛选，但添加注释说明）：
在 `ChangeLogTable` 中添加注释，说明当前为客户端筛选，后续大数据量时迁移到服务端筛选。

---

### 问题 4：数据类型不匹配 — `SystemConfig.value` 类型

| 属性 | 内容 |
|------|------|
| **类型** | 数据结构 |
| **问题** | 任务卡定义 `value: any`，代码实现为 `value: unknown` |
| **位置** | `system-config.ts:18` |
| **影响** | `unknown` 比 `any` 更安全，但需确认与后端返回类型一致 |

**契约定义（任务卡 §5.1）：**
```typescript
export interface SystemConfig {
  value: any;
}
```

**当前实现：**
```typescript
export interface SystemConfig {
  value: unknown;
}
```

**判定：** ✅ 此为合理优化，`unknown` 比 `any` 更安全，建议保留当前实现。但需确认后端实际返回类型支持 `string | number | boolean | object`。

---

### 问题 5：响应结构假设 — 缺少运行时校验

| 属性 | 内容 |
|------|------|
| **类型** | 返回结构 |
| **问题** | 所有 API 响应均使用 `as unknown as { items: T[]; total: number }` 强制类型转换，无运行时校验 |
| **位置** | `SystemSettingsPage.tsx:51-53, 64-66, 76-78, 88-90` |
| **影响** | 后端返回结构与预期不一致时，可能导致运行时错误（如 `data.items` 为 `undefined`） |

**当前实现：**
```typescript
const response = await adminApi.settings.list();
const data = response as unknown as { items: SystemConfig[]; total: number };
setConfigData(data.items); // 若 data.items 为 undefined，会传递 undefined
```

**修改建议（前端）：**
添加防御性校验：
```typescript
const response = await adminApi.settings.list();
const data = response as unknown as { items: SystemConfig[]; total: number };
if (!data || !Array.isArray(data.items)) {
  throw new Error('Invalid response structure: items is not an array');
}
setConfigData(data.items);
```

---

## 🛠 修改建议汇总

### 前端修改

1. **`services.ts`**
   - `recommendations.update` 参数类型改为 `HomeRecommendation[]`
   - `categories.update` 参数类型改为 `Partial<CategoryManagement>`

2. **`SystemSettingsPage.tsx`**
   - `loadChangelogs` 添加 `changelogParams` 参数传递
   - 所有 API 响应添加运行时结构校验

3. **`ChangeLogTable.tsx`**
   - 支持 `params` / `onParamsChange` props，或添加注释说明当前为客户端筛选

### 后端修改

无。当前契约定义清晰，后端按任务卡 §8.1 实现即可。

### 数据结构调整

无。当前类型定义合理，仅需将 `unknown[]` / `unknown` 替换为具体业务类型。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | **低** | 类型修正为编译期变更，不影响运行时行为 |
| 是否影响数据模型 | **无** | 数据模型本身未变更，仅修正类型标注 |
| 是否影响现有功能 | **无** | T035 为新增功能，不影响现有页面 |
| 变更日志筛选性能 | **中** | 当前客户端筛选在数据量大时存在性能风险，建议后续迁移到服务端筛选 |

---

## ✅ 是否允许进入验收（D2）

👉 **NO（需先修复上述问题）**

**前置条件：**
1. [ ] `services.ts` 中 `recommendations.update` 和 `categories.update` 参数类型修正
2. [ ] `SystemSettingsPage.tsx` 中 API 响应添加运行时结构校验
3. [ ] 确认 `ChangeLogTable` 筛选策略（客户端筛选需添加注释，或迁移到服务端筛选）

以上问题修复后，可重新执行 D1 检查并进入 D2 验收。

---

## 📎 附录：API 契约对照表

| API | Method | Path | 请求类型（契约） | 请求类型（实现） | 状态 |
|-----|--------|------|-----------------|-----------------|------|
| 系统配置查询 | GET | `/admin-api/v1/settings` | — | — | ✅ |
| 系统配置更新 | PUT | `/admin-api/v1/settings/:id` | `{ value: any }` | `SystemConfigUpdateRequest` | ✅ |
| 变更日志查询 | GET | `/admin-api/v1/changelogs` | `ChangeLogListParams` | `ChangeLogListParams` | ⚠️ 参数未传递 |
| 首页推荐查询 | GET | `/admin-api/v1/recommendations` | — | — | ✅ |
| 首页推荐更新 | PUT | `/admin-api/v1/recommendations` | `HomeRecommendation[]` | `unknown[]` | ❌ |
| 分类管理查询 | GET | `/admin-api/v1/categories` | — | — | ✅ |
| 分类管理更新 | PUT | `/admin-api/v1/categories/:id` | `CategoryManagement` | `unknown` | ❌ |

---

*本文档与代码同步维护，修复后请同步更新。*
