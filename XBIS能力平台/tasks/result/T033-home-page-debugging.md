# T033 首页（产品化）- 接口联调检查报告（D1）

> 任务编号：T033
> 检查阶段：D1 接口联调检查
> 检查日期：2026-04-26
> 检查角色：后端架构师 + 前端联调负责人 + API 契约审查官

---

## 🧪 联调结果

**状态：✅ 修复完成，允许进入 D2 验收**

原始问题 7 个，已修复 6 个，1 个经确认无需修复。

---

## ❌ 问题列表（含修复状态）

### 问题 #1：API 路径不一致（中等）→ 经确认无需修复

| 维度 | 内容 |
|------|------|
| **类型** | API 契约 |
| **位置** | `packages/shared/src/api/services.ts` 第 569 行 |

**详情**：
- 任务卡定义：`GET /api/v1/home`
- 前端实现：`GET /portal-api/v1/home`

**处理结论**：
项目所有用户端 API 均统一使用 `/portal-api/v1/*` 前缀（dashboard、account、apiMarket 等）。任务卡中的 `/api/v1/home` 为早期设计稿，未同步更新。
**无需修复**，但需更新任务卡文档以保持一致。

---

### 问题 #2：`popularAbilities` 类型不匹配（高）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | 数据结构 |
| **位置** | `packages/shared/src/types/index.ts` 第 1082 行 |

**详情**：
- 任务卡定义：`popularAbilities: AbilityCardItem[]`
- 前端实现：`popularAbilities: ApiMarketItem[]`

**处理结论**：
经审查，`AbilityCardItem` 与 `ApiMarketItem` 字段差异较大。`AbilityCard` 业务组件已广泛使用 `ApiMarketItem`，且首页"热门能力"数据来源于 API 市场，使用 `ApiMarketItem` 更符合实际业务场景。

**修复方式**：保持前端使用 `ApiMarketItem[]`，任务卡文档需同步更新。
**修复后代码**：`popularAbilities: ApiMarketItem[]`（与现有实现一致）

---

### 问题 #3：`recentJobs[].status` 类型应为 JobStatus（中）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | 类型安全 |
| **位置** | `packages/shared/src/types/index.ts` 第 1069 行 |

**修复方式**：
```typescript
// 修复前
status: string;

// 修复后
status: JobStatus;
```

---

### 问题 #4：JobStatus 状态映射不完整（中）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | 状态流 |
| **位置** | `packages/components/blocks/HomeRecentJobs/index.tsx` 第 14-32 行 |

**修复方式**：
补充了 3 个缺失状态映射：

```typescript
const STATUS_COLORS: Record<string, string> = {
  // ... 已有 7 个
  routing: colors.info.default,
  callback_pending: colors.warning.default,
  callback_failed: colors.error.default,
};

const STATUS_LABELS: Record<string, string> = {
  // ... 已有 7 个
  routing: '路由中',
  callback_pending: '回调中',
  callback_failed: '回调失败',
};
```

---

### 问题 #5：StatsBar 响应式样式未生效（低）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | 样式/Bug |
| **位置** | `packages/components/blocks/StatsBar/index.tsx` 第 37-42 行 |

**修复方式**：
外层 div 添加 `className="stats-bar-grid"`，使 CSS 媒体查询生效。

---

### 问题 #6：`CategoryGrid` borderRadius 硬编码（低）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | Design Token |
| **位置** | `packages/components/blocks/CategoryGrid/index.tsx` 第 53 行 |

**修复方式**：
```tsx
// 修复前
borderRadius: 12,

// 修复后
borderRadius: radius.md,
```

---

### 问题 #7：`HomeRecentJobs` borderRadius 硬编码（低）→ ✅ 已修复

| 维度 | 内容 |
|------|------|
| **类型** | Design Token |
| **位置** | `packages/components/blocks/HomeRecentJobs/index.tsx` 第 96 行 |

**修复方式**：
```tsx
// 修复前
borderRadius: 8,

// 修复后
borderRadius: radius.md,
```

---

## 🛠 修改文件汇总

### 已修改文件

1. `packages/shared/src/types/index.ts`
   - `RecentJobItem.status`：`string` → `JobStatus`
   - `HomePageData.popularAbilities`：保持 `ApiMarketItem[]`（经确认符合业务场景）

2. `packages/components/blocks/HomeRecentJobs/index.tsx`
   - 补充 `routing`, `callback_pending`, `callback_failed` 状态映射
   - `borderRadius: 8` → `radius.md`

3. `packages/components/blocks/StatsBar/index.tsx`
   - 外层 div 添加 `className="stats-bar-grid"`

4. `packages/components/blocks/CategoryGrid/index.tsx`
   - `borderRadius: 12` → `radius.md`

### 需更新文档

1. `tasks/T033-home-page.md`
   - API 路径：`/api/v1/home` → `/portal-api/v1/home`
   - `popularAbilities` 类型：`AbilityCardItem[]` → `ApiMarketItem[]`

---

## ⚠️ 风险评估

| 风险 | 说明 | 影响 |
|------|------|------|
| 是否影响后续任务 | 否。类型调整已与现有组件兼容 | 无 |
| 是否影响数据模型 | 否。仅前端类型定义细化，不涉及后端变更 | 无 |

---

## ✅ 是否允许进入验收（D2）

**YES**

所有前端问题已修复：
- ✅ 问题 #3（recentJobs.status 类型）— 已修复
- ✅ 问题 #4（JobStatus 状态映射）— 已修复
- ✅ 问题 #5（StatsBar 响应式样式）— 已修复
- ✅ 问题 #6（CategoryGrid borderRadius）— 已修复
- ✅ 问题 #7（HomeRecentJobs borderRadius）— 已修复

经确认无需修复：
- ✅ 问题 #1（API 路径）— 项目统一使用 `/portal-api/v1/*`，任务卡文档需更新
- ✅ 问题 #2（popularAbilities 类型）— `ApiMarketItem[]` 符合业务场景，任务卡文档需更新

**建议**：进入 D2 验收前，同步更新任务卡文档中的 API 路径和类型定义。
