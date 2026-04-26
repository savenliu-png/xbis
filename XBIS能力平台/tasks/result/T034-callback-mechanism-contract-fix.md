# T034 callback 机制 — 接口契约级修复报告

> 任务编号：T034
> 修复阶段：Contract Fix（契约修复）
> 修复日期：2026-04-25
> 修复人：AI 契约修复专家（资深系统架构师 + API契约修复专家 + 前后端联调负责人）

---

## 一、问题分类

| 类型 | 问题 | 分类 |
|------|------|------|
| 契约缺失 | `CallbackListResponse` 缺少 `page`/`pageSize` 字段 | Contract Missing |
| 契约不一致 | `CallbackRecord.payload` 任务卡为 `any`，实现为 `unknown` | Contract Mismatch |
| 契约不一致 | `CallbackResponse.status` 任务卡为 `string`，实现为枚举 | Contract Mismatch |
| 设计不明确 | `CallbackRequest` 是否为前端使用类型 | Ambiguous Design |
| 设计不明确 | `CallbackResponse` 与 `CallbackRetryResponse` 命名差异 | Ambiguous Design |

---

## 二、契约决策

| 问题 | 方案A | 方案B | 推荐方案 | 理由 |
|------|-------|-------|----------|------|
| `page`/`pageSize` 缺失 | 从实现中移除 | 补充到任务卡 | **方案B：补充到任务卡** | 分页组件需要 `page`/`pageSize` 信息 |
| `payload` 类型 `any` vs `unknown` | 改为 `any` | 保持 `unknown` | **方案B：保持 `unknown`** | TypeScript 类型安全最佳实践 |
| `status` 类型 `string` vs 枚举 | 改为 `string` | 保持枚举 | **方案B：保持枚举** | 类型精确，防止非法状态值 |
| `CallbackRequest` 未实现 | 添加类型 | 保持现状 | **方案B：保持现状** | 后端内部使用，前端无需构造 |
| `CallbackResponse` 命名差异 | 重命名 | 保持现状 | **方案B：保持现状** | `CallbackRetryResponse` 更语义化 |

---

## 三、修复内容

### 3.1 代码修改

**无需修改代码** — 所有契约决策均确认当前实现正确，仅需同步任务卡文档。

### 3.2 设计方案更新（任务卡同步）

**文件**：`tasks/T034-callback-mechanism.md`

**修改 1**：`CallbackRecord.payload` 类型更新
```typescript
// 修改前
payload: Record<string, any>;

// 修改后
payload: Record<string, unknown>;
```

**修改 2**：`CallbackRequest.payload` 类型更新 + 标注后端内部使用
```typescript
// 修改前
// 回调请求
export interface CallbackRequest {
  jobId: string;
  callbackUrl: string;
  payload: Record<string, any>;
}

// 修改后
// 回调请求（后端内部使用）
export interface CallbackRequest {
  jobId: string;
  callbackUrl: string;
  payload: Record<string, unknown>;
}
```

**修改 3**：`CallbackResponse.status` 类型精确化
```typescript
// 修改前
export interface CallbackResponse {
  recordId: string;
  status: string;
  message: string;
}

// 修改后
export interface CallbackResponse {
  recordId: string;
  status: 'pending' | 'success' | 'failed' | 'retrying';
  message: string;
}
```

**修改 4**：`CallbackListResponse` 补充 `page`/`pageSize`
```typescript
// 修改前
- **响应类型**: `{ items: CallbackRecord[]; total: number }`

// 修改后
- **响应类型**: `{ items: CallbackRecord[]; total: number; page: number; pageSize: number }`
```

**修改 5**：响应示例补充 `page`/`pageSize`
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 10,
    "page": 1,
    "pageSize": 20
  }
}
```

### 3.3 类型定义更新

**无需修改类型定义** — 当前实现已正确：
- `callback.ts` 中 `payload` 为 `Record<string, unknown>` ✅
- `callback.ts` 中 `CallbackRetryResponse.status` 为 `CallbackStatus` ✅
- `callback.ts` 中 `CallbackListResponse` 含 `page`/`pageSize` ✅

---

## 四、影响评估

| 影响项 | 是否影响 | 说明 |
|--------|----------|------|
| 前端页面 | 否 | 无需修改代码 |
| 后端接口 | 否 | 无需修改接口 |
| 后续任务 | 否 | 契约已明确，后续可直接使用 |
| 现有功能 | 否 | 新功能，不影响现有功能 |

---

## 五、是否完成联调修复

**👉 YES**

### 完成理由

1. **契约决策完成**：5 项问题全部完成决策，理由充分
2. **代码无需修改**：所有决策均确认当前实现正确
3. **设计同步完成**：任务卡已更新，与实现一致
4. **类型同步完成**：TypeScript 类型与契约一致
5. **前后端一致**：契约明确，双方按统一规范实现
6. **无破坏性变更**：所有修改均为文档同步，不影响运行时
