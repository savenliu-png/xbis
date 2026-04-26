# T004 页面模板 — 接口联调检查报告（D1）

> 任务编号: T004
> 任务名称: 页面模板（layout/ 页面模板骨架）
> 检查日期: 2026-04-25
> 文档版本: v1.0
> 执行人: 资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 🔍 检查维度逐项报告

### 1️⃣ API 契约一致性

| 检查项 | 状态 | 说明 |
|--------|------|------|
| URL 一致性 | ✅ | 页面模板为纯布局组件，无直接 API 调用 |
| Method 正确性 | ✅ | N/A（无 API 调用） |
| 参数完整性 | ✅ | N/A（数据通过 Props 传入） |
| 多余参数 | ✅ | 无 |

**结论**：页面模板层（layout shells）不直接调用 API，API 契约一致性检查通过。

---

### 2️⃣ 返回结构匹配

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 字段名一致性 | ✅ | 模板 Props 与类型定义匹配 |
| 类型匹配 | ✅ | AdminPageShellProps / BreadcrumbItem 类型完整 |
| 缺失字段 | ✅ | 无缺失 |
| 未定义字段使用 | ✅ | 无 |

**AdminPageShell Props 结构**：
```typescript
export interface AdminPageShellProps {
  title: string;
  subtitle?: string;
  breadcrumbs?: BreadcrumbItem[];
  actions?: React.ReactNode;
  children: React.ReactNode;
  status: AdminPageStatus;  // 'idle' | 'loading' | 'empty' | 'error'
  errorMessage?: string;
  onRetry?: () => void;
  showCardWrapper?: boolean;
  cardPadding?: 'none' | 'compact' | 'default';
}
```

---

### 3️⃣ 数据结构一致性

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 符合任务卡定义 | ✅ | 页面模板 Props 与 T004 任务卡要求一致 |
| 字段命名冲突 | ✅ | 无冲突 |
| 结构嵌套错误 | ✅ | 无嵌套错误 |

---

### 4️⃣ 状态流一致性（关键检查）

#### 4.1 前端 JobStatus 定义

```typescript
// packages/shared/src/types/index.ts
export type JobStatus = 
  | 'accepted' 
  | 'queued' 
  | 'routing' 
  | 'running' 
  | 'waiting_review' 
  | 'completed' 
  | 'failed' 
  | 'cancelled' 
  | 'callback_pending';
```

#### 4.2 后端真实状态机（来自 XBIS_EXTERNAL_SYSTEM_INTEGRATION_GUIDE.md）

后端真实支持的状态：
- `accepted`
- `queued`
- `routing`
- `running`
- `waiting_review`
- `completed`
- `failed`
- `cancelled`
- `callback_pending`
- `callback_failed`  ← **后端有，前端缺失**

#### 4.3 状态差异对比

| 状态 | 前端 JobStatus | 后端状态机 | 差异 |
|------|---------------|-----------|------|
| accepted | ✅ | ✅ | 一致 |
| queued | ✅ | ✅ | 一致 |
| routing | ✅ | ✅ | 一致 |
| running | ✅ | ✅ | 一致 |
| waiting_review | ✅ | ✅ | 一致 |
| completed | ✅ | ✅ | 一致 |
| failed | ✅ | ✅ | 一致 |
| cancelled | ✅ | ✅ | 一致 |
| callback_pending | ✅ | ✅ | 一致 |
| callback_failed | ❌ **缺失** | ✅ | **前端缺失** |

#### 4.4 影响分析

- `TaskStatusBadge` 组件在遇到 `callback_failed` 状态时会回退到 `accepted` 的默认配置（第 24 行 `|| statusConfig.accepted`）
- `TaskFilterBar` 组件缺少 `callback_failed` 筛选选项
- 用户将无法正确看到"回调失败"状态的任务

---

### 5️⃣ 错误处理

| 检查项 | 状态 | 说明 |
|--------|------|------|
| API error 处理 | ✅ | 页面模板为纯布局组件，不直接调用 API |
| 空数据处理（empty） | ✅ | AdminPageShell 支持 `empty` 状态 |
| loading 处理 | ✅ | AdminPageShell 支持 `loading` 状态 |
| 超时/失败处理 | ✅ | AdminPageShell 支持 `error` 状态 + `onRetry` |

**AdminPageShell 状态覆盖**：
- `idle` → 正常渲染 children
- `loading` → 显示 Spinner
- `empty` → 显示 Empty 组件
- `error` → 显示 Alert + 重试按钮

---

### 6️⃣ Business Services 层检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 通过 Service 层调用 API | ✅ | 页面模板无 API 调用，纯布局容器 |
| 绕过业务接入层 | ✅ | 无绕过行为 |

**说明**：
- 页面模板（layout shells）的定位是 **纯布局容器**，不负责数据获取
- 数据获取由页面层（pages）通过 `@xbis/shared` 中的 `adminApi` / `userApi` 完成
- 当前 Service 层已统一封装在 `packages/shared/src/api/services.ts`
- API Client 已统一封装在 `packages/shared/src/api/client.ts`

---

## 🧪 联调结果（D1）

👉 **状态：前端待修**

原因：前端 `JobStatus` 类型和 `TaskStatusBadge` / `TaskFilterBar` 组件缺少 `callback_failed` 状态支持。

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 |
|------|------|------|------|
| 状态缺失 | `callback_failed` 未包含在 `JobStatus` 类型中 | `packages/shared/src/types/index.ts:96-105` | 类型系统无法识别回调失败状态 |
| 状态缺失 | `callback_failed` 未在 `TaskStatusBadge` 中配置 | `packages/components/business/TaskStatusBadge/index.tsx:10-20` | 回调失败任务显示为"已接受" |
| 状态缺失 | `callback_failed` 未在 `TaskFilterBar` 中配置 | `packages/components/business/TaskFilterBar/index.tsx:18-29` | 无法筛选回调失败任务 |

---

## 🛠 修改建议

### 前端修改：

#### 1. 更新 `JobStatus` 类型定义

**文件**: `packages/shared/src/types/index.ts`

```typescript
export type JobStatus = 
  | 'accepted' 
  | 'queued' 
  | 'routing' 
  | 'running' 
  | 'waiting_review' 
  | 'completed' 
  | 'failed' 
  | 'cancelled' 
  | 'callback_pending'
  | 'callback_failed';  // ← 新增
```

#### 2. 更新 `TaskStatusBadge` 配置

**文件**: `packages/components/business/TaskStatusBadge/index.tsx`

```typescript
const statusConfig: Record<string, { label: string; variant: 'success' | 'warning' | 'error' | 'info' | 'default' | 'brand'; dot: 'success' | 'warning' | 'error' | 'info' | 'default' | 'brand' }> = {
  accepted: { label: '已接受', variant: 'default', dot: 'default' },
  queued: { label: '队列中', variant: 'info', dot: 'info' },
  routing: { label: '路由中', variant: 'info', dot: 'info' },
  running: { label: '执行中', variant: 'brand', dot: 'brand' },
  waiting_review: { label: '待审核', variant: 'warning', dot: 'warning' },
  completed: { label: '已完成', variant: 'success', dot: 'success' },
  failed: { label: '失败', variant: 'error', dot: 'error' },
  cancelled: { label: '已取消', variant: 'default', dot: 'default' },
  callback_pending: { label: '回调中', variant: 'info', dot: 'info' },
  callback_failed: { label: '回调失败', variant: 'error', dot: 'error' },  // ← 新增
};
```

#### 3. 更新 `TaskFilterBar` 筛选选项

**文件**: `packages/components/business/TaskFilterBar/index.tsx`

```typescript
const statusOptions: { value: JobStatus | 'all'; label: string }[] = [
  { value: 'all', label: '全部' },
  { value: 'accepted', label: '已接受' },
  { value: 'queued', label: '队列中' },
  { value: 'routing', label: '路由中' },
  { value: 'running', label: '执行中' },
  { value: 'waiting_review', label: '待审核' },
  { value: 'completed', label: '已完成' },
  { value: 'failed', label: '失败' },
  { value: 'cancelled', label: '已取消' },
  { value: 'callback_pending', label: '回调中' },
  { value: 'callback_failed', label: '回调失败' },  // ← 新增
];
```

### 后端修改：

无需修改。后端已支持 `callback_failed` 状态。

### 数据结构调整：

无需调整。

---

## ⚠️ 风险评估

| 风险项 | 影响 | 说明 |
|--------|------|------|
| 是否影响后续任务 | 低 | 状态缺失为边界情况，不影响主流程 |
| 是否影响数据模型 | 否 | 仅前端类型和展示配置缺失 |
| 是否影响用户体验 | 中 | 用户无法正确识别回调失败的任务 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES（附条件）**

条件：建议在进入 T005 前修复 `callback_failed` 状态缺失问题，但当前问题不影响页面模板核心功能，可并行处理。

理由：
1. 页面模板核心功能（idle/loading/empty/error）完整且正确
2. `callback_failed` 为边界状态，主流程不受影响
3. 修复成本极低（3 处简单添加）
4. 不涉及 API 契约变更或数据结构变更

---

## 📋 附加说明

### 页面模板与 API 的调用关系

```
┌─────────────────────────────────────────────────────────────┐
│  页面层 (Pages)                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  通过 adminApi / userApi 调用 API                     │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │   │
│  │  │ /portal-api  │  │ /admin-api   │  │ /api/v1    │  │   │
│  │  └──────────────┘  └──────────────┘  └────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  数据通过 Props 传入                                   │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │   │
│  │  │ ListPageShell│  │DetailPageShell│  │AdminPageShell│  │   │
│  │  └──────────────┘  └──────────────┘  └────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  渲染 UI（PageHeader + ContentArea + Card + 状态）     │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### API 路由对照表

| 前端 Service | 路由前缀 | 后端对应 |
|-------------|----------|----------|
| `adminApi.*` | `/admin-api/v1/*` | 管理后台 API |
| `userApi.*` | `/portal-api/v1/*` | 用户门户 API |
| `apiClient` | `/api/v1/*` | XBIS 核心 API |

### 关键结论

1. **页面模板层不直接调用 API** — 符合架构设计，数据通过 Props 注入
2. **Service 层已统一封装** — `packages/shared/src/api/services.ts` 覆盖 admin/portal 双端
3. **唯一问题为状态缺失** — `callback_failed` 在前端类型和组件中未定义
4. **API 契约整体一致** — 无 URL/Method/参数/结构不匹配问题

---

*报告生成时间: 2026-04-25*
*检查依据: XBIS_EXTERNAL_SYSTEM_INTEGRATION_GUIDE.md, packages/shared/src/types/index.ts, packages/shared/src/api/services.ts, packages/components/business/TaskStatusBadge/index.tsx, packages/components/business/TaskFilterBar/index.tsx*
