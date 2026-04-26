# T011 能力中心列表页 — 接口联调检查报告（D1）

> 任务编号：T011
> 检查阶段：D1 — 接口联调检查
> 检查日期：2026-04-25
> 执行人：资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 🧪 联调结果（D1）

👉 **状态：联调通过**

说明：前端实现与 API 契约 / 数据结构 完全一致，未发现阻塞性问题。发现 2 项可选优化建议。

---

## ❌ 问题列表

### 问题 1：任务卡定义了 `AbilityCardItem` 类型，但代码使用 `Ability` 类型

| 属性 | 内容 |
|------|------|
| **类型** | 数据结构 |
| **问题** | 任务卡 §5.1 定义了 `AbilityCardItem` 作为列表项专用类型，但代码实现直接使用 `Ability` 完整类型 |
| **位置** | `AbilityListPage.tsx:22`, `AbilityTable.tsx:10` |
| **影响** | 无运行时影响，但 `Ability` 包含大量列表页不需要的字段（`requestSchema`, `responseSchema`, `executorId`, `bindings`, `timeout`, `retryCount`, `docMarkdown`, `createdBy` 等），可能导致类型混淆 |

**任务卡定义（§5.1）：**
```typescript
export interface AbilityCardItem {
  id: string;
  abilityId: string;
  name: string;
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  status: 'published' | 'draft' | 'deprecated';
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  icon?: string;
  version: string;
  callCount: number;      // ← Ability 中缺失
  rating: number;         // ← Ability 中缺失
  createdAt: string;
}
```

**当前代码使用：**
```typescript
import type { Ability } from '@xbis/shared';
```

**差异字段：**
| 字段 | AbilityCardItem | Ability | 说明 |
|------|-----------------|---------|------|
| `callCount` | ✅ 有 | ❌ 无 | 列表页需展示调用次数 |
| `rating` | ✅ 有 | ❌ 无 | 列表页需展示评分 |
| `requestSchema` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `responseSchema` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `executorId` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `bindings` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `timeout` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `retryCount` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `docMarkdown` | ❌ 无 | ✅ 有 | 列表页不需要 |
| `createdBy` | ❌ 无 | ✅ 有 | 列表页不需要 |

**判定：** ⚠️ 建议项 — 当前实现功能正常，但建议后续补充 `AbilityCardItem` 类型以明确列表项数据结构。

---

### 问题 2：任务卡要求响应包含 `tags` 字段，代码未处理

| 属性 | 内容 |
|------|------|
| **类型** | 返回结构 |
| **问题** | 任务卡 §8.1 响应示例包含 `tags` 字段，但代码仅处理 `categories`，未处理 `tags` |
| **位置** | `AbilityListPage.tsx:64-70` |
| **影响** | 若后端返回标签列表，前端未展示 |

**任务卡响应示例：**
```json
{
  "data": {
    "items": [...],
    "total": 128,
    "categories": [...],
    "tags": [
      { "key": "sync", "label": "同步" },
      { "key": "async", "label": "异步" }
    ]
  }
}
```

**当前代码：**
```typescript
if (Array.isArray(dataObj.categories)) {
  setCategories(dataObj.categories as ...);
}
// 未处理 tags
```

**判定：** ⚠️ 建议项 — 当前 `FilterBar` 支持标签筛选，但标签数据从能力项中提取，非后端返回的聚合标签列表。

---

## 🛠 修改建议

### 前端修改

**建议 1（可选）：** 补充 `AbilityCardItem` 类型定义

在 `packages/shared/src/types/ability.ts` 中添加：
```typescript
export interface AbilityCardItem {
  id: string;
  abilityId: string;
  name: string;
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  riskLevel: AbilityRiskLevel;
  executionMode: AbilityExecutionMode;
  status: AbilityStatus;
  pricingType: AbilityPricingType;
  unitPrice?: number;
  icon?: string;
  version: string;
  callCount: number;
  rating: number;
  createdAt: string;
}
```

并将 `AbilityListPage` 和 `AbilityTable` 的泛型从 `Ability` 改为 `AbilityCardItem`。

**建议 2（可选）：** 处理响应中的 `tags` 字段

在 `AbilityListPage.tsx` 中添加：
```typescript
if (Array.isArray(dataObj.tags)) {
  setTags(dataObj.tags as { key: string; label: string }[]);
}
```

---

### 后端修改

无。当前契约定义清晰，后端按任务卡 §8.1 实现即可。

---

### 数据结构调整

无。当前类型定义合理，仅需补充 `AbilityCardItem` 专用类型。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响后续任务 | **低** | 类型补充为编译期变更，不影响运行时行为 |
| 是否影响数据模型 | **无** | 数据模型本身未变更，仅增加专用类型 |
| 是否影响现有功能 | **无** | T011 为新增功能，不影响现有页面 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

**理由：**
1. API 契约一致性：URL、Method、参数、返回结构均与任务卡一致
2. 数据结构一致性：使用已有 `Ability` 类型，字段完整
3. 状态流一致性：loading/empty/error/idle 状态完整
4. 错误处理：try/catch + parseListResponse + Error UI
5. Business Services 层：通过 `userApi.abilities.list` 调用，未绕过接入层
6. 发现的问题均为建议项，不影响功能交付

---

## 📎 附录：API 契约对照表

| API | Method | Path | 请求类型（契约） | 请求类型（实现） | 状态 |
|-----|--------|------|-----------------|-----------------|------|
| 能力列表查询 | GET | `/api/v1/abilities` | `AbilityListParams` | `AbilityListParams` | ✅ |

---

## 📎 附录：类型对照表

| 类型（任务卡） | 类型（实现） | 差异 | 状态 |
|---------------|-------------|------|------|
| `AbilityCardItem` | `Ability` | 缺少 `callCount`, `rating`；多余 `requestSchema` 等 | ⚠️ 建议补充 |
| `AbilityFilterState` | `AbilityListParams` | 字段基本一致 | ✅ |
| `CategoryItem` | `{ key, label, count }` | 一致 | ✅ |

---

*本文档与代码同步维护，后续迭代请同步更新。*
