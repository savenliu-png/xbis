# T011 能力中心列表页 — 接口契约级修复报告

> 任务编号：T011
> 修复阶段：Contract Fix（基于 D1 联调报告）
> 修复日期：2026-04-25
> 执行人：资深系统架构师 + API契约修复专家 + 前后端联调负责人

---

## 一、问题分类

| # | 问题 | 分类 | 说明 |
|---|------|------|------|
| 1 | 任务卡定义 `AbilityCardItem`，代码使用 `Ability` | 1️⃣ 契约缺失 | 设计定义了专用列表类型，但代码未实现 |
| 2 | 响应 `tags` 字段未处理 | 1️⃣ 契约缺失 | 设计定义了响应结构，但代码未提取该字段 |

---

## 二、契约决策

### 决策 1：`AbilityCardItem` 类型补充

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 补充专用类型，明确列表项数据结构 | ✅ 推荐 — 专用类型更清晰，避免列表页误用完整字段 |
| B | 保持 `Ability`，通过类型断言兼容 | 类型混淆，列表页可能误用不需要的字段 |

**推荐：方案A** — `AbilityCardItem` 明确列表项所需字段，增加 `callCount`/`rating` 专用字段，移除列表页不需要的字段。

### 决策 2：响应 `tags` 字段处理

| 方案 | 内容 | 评估 |
|------|------|------|
| A | 提取并展示聚合标签筛选 | 增强功能，但需改动 FilterBar 接口 |
| B | 保持现状，从能力项提取标签 | ✅ 推荐 — 当前已实现分类筛选和搜索，标签筛选为增强功能 |

**推荐：方案B** — 保持现状，后续迭代时添加聚合标签筛选。

---

## 三、修复内容

### 3.1 代码修改

#### 修改 1：`ability.ts` — 新增 `AbilityCardItem` 类型

**文件**：`packages/shared/src/types/ability.ts`

**新增内容**：
```typescript
// ---------- 能力列表页专用类型 ----------

/**
 * 能力卡片列表项（列表页专用）
 * 相比 Ability 完整类型，移除了列表页不需要的字段，
 * 并增加了列表展示专用字段（callCount, rating）
 */
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

---

#### 修改 2：`AbilityListPage.tsx` — 泛型替换

**文件**：`packages/pages/ability/AbilityListPage.tsx`

**修改前**：
```typescript
import type { Ability, AbilityListParams } from '@xbis/shared';

const { state, setLoading, setData, setError } = usePageState<Ability[]>();

const data = parseListResponse<Ability>(response);
```

**修改后**：
```typescript
import type { AbilityCardItem, AbilityListParams } from '@xbis/shared';

const { state, setLoading, setData, setError } = usePageState<AbilityCardItem[]>();

const data = parseListResponse<AbilityCardItem>(response);
```

---

#### 修改 3：`AbilityTable.tsx` — 泛型替换

**文件**：`packages/pages/ability/components/AbilityTable.tsx`

**修改前**：
```typescript
import type { Ability, AbilityRiskLevel, AbilityExecutionMode } from '@xbis/shared';

interface AbilityTableProps {
  data: Ability[];
  onRowClick?: (ability: Ability) => void;
}

render: (value: string, record: Ability) => (...)
onRow={(record: Ability) => ({...})}
```

**修改后**：
```typescript
import type { AbilityCardItem, AbilityRiskLevel, AbilityExecutionMode } from '@xbis/shared';

interface AbilityTableProps {
  data: AbilityCardItem[];
  onRowClick?: (ability: AbilityCardItem) => void;
}

render: (value: string, record: AbilityCardItem) => (...)
onRow={(record: AbilityCardItem) => ({...})}
```

---

### 3.2 设计方案更新

无需更新设计方案（Final），因为：
- 决策 1 为类型补充，设计文档已定义 `AbilityCardItem`，仅代码未实现
- 决策 2 保持现状，无需设计变更

---

### 3.3 类型定义更新

**文件**：`packages/shared/src/types/ability.ts`

| 变更 | 说明 |
|------|------|
| 新增 `AbilityCardItem` | 能力列表页专用类型，包含 `callCount`/`rating` |

**自动导出**：`index.ts` 使用 `export * from './ability'`，`AbilityCardItem` 自动导出。

---

## 四、影响评估

| 评估项 | 结论 | 说明 |
|--------|------|------|
| 是否影响前端页面 | **否** | 仅类型变更，运行时行为不变 |
| 是否影响后端接口 | **否** | 后端按原有契约返回数据 |
| 是否影响后续任务 | **否** | 类型更严格，有利于后续开发 |
| 是否影响现有功能 | **否** | T011 为新增功能，不影响现有页面 |

---

## 五、是否完成联调修复

👉 **YES**

**修复清单**：
- [x] `ability.ts` 新增 `AbilityCardItem` 类型
- [x] `AbilityListPage.tsx` 泛型从 `Ability` 改为 `AbilityCardItem`
- [x] `AbilityTable.tsx` 泛型从 `Ability` 改为 `AbilityCardItem`
- [x] `index.ts` 自动导出新类型（`export *`）

**建议**：修复完成后，重新执行 D1 检查确认无遗留问题，然后进入 D2 功能验收。

---

## 六、修复后类型对照表

| 类型（任务卡） | 类型（修复后） | 状态 |
|---------------|---------------|------|
| `AbilityCardItem` | `AbilityCardItem` | ✅ |
| `AbilityFilterState` | `AbilityListParams` | ✅ |
| `CategoryItem` | `{ key, label, count }` | ✅ |

---

*本文档与代码同步维护，后续迭代请同步更新。*
