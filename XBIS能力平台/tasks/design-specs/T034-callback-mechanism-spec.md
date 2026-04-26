# T034 Callback 机制 — 工程级设计方案（Design Spec）

> 任务卡：T034-callback-mechanism.md  
> 状态：设计确认  
> 输出日期：2026-04-24

---

## 1. 页面结构

### 1.1 层级结构

本任务主要为后端机制，前端仅提供回调记录查询页面。

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "回调记录"
    └── ListPageShell
        ├── FilterBar
        │   ├── 任务 ID 搜索
        │   └── 状态筛选
        ├── DataTable
        │   └── 回调记录列表
        └── Pagination
```

### 1.2 模板使用

| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 列表 | ListPageShell | 列表页模板 |

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button, Input, Table, Tag, Badge, Skeleton, Empty, Pagination | 标准用法 |

### 2.2 blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| CallbackTable | 回调记录表格 | **新建** — 含重试按钮 |

---

## 3. 数据流

### 3.1 回调记录查询

```
[进入 /callbacks]
    │
    ▼
[API: GET /api/v1/callbacks]
    │
    ▼
[渲染回调记录列表]
```

### 3.2 手动重试

```
[点击「重试」]
    │
    ▼
[API: POST /api/v1/callbacks/:id/retry]
    │
    ├── 成功 → 更新记录状态
    └── 失败 → Toast 错误
```

---

## 4. 状态管理

```typescript
interface CallbackPageState {
  pageState: 'loading' | 'idle' | 'error';
  records: CallbackRecord[];
  total: number;
  filters: { jobId?: string; status?: string; page: number; pageSize: number };
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 回调记录查询 | GET | `/api/v1/callbacks` | 查询回调记录 |
| 手动重试 | POST | `/api/v1/callbacks/:id/retry` | 手动触发重试 |

---

## 6. 用户交互流程

```
[进入回调记录页]
    │
    ▼
[浏览回调记录]
    │
    ├── 筛选状态/任务ID
    │
    └── 点击「重试」（失败记录）
            │
            ▼
        [触发重试]
            │
            ▼
        [更新状态]
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无回调记录 | Empty 组件 |
| 重试失败 | Toast 错误提示 |
| 回调 URL 无效 | 标记失败，记录错误 |

---

## 8. 性能优化

- 回调记录分页加载
- 状态筛选 Debounce

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| **回调风暴** | 大量任务同时完成触发回调 | 后端限流 + 队列 |
| **重试风暴** | 失败回调频繁重试 | 指数退避；最大 3 次 |
| **回调安全** | 回调 URL 可能被恶意利用 | 签名验证；URL 白名单 |

---

## 10. 开发步骤拆分

### Step 1: 后端机制（2 天）
- [ ] 回调触发逻辑
- [ ] 重试机制（指数退避）
- [ ] 回调记录存储

### Step 2: 前端页面（0.5 天）
- [ ] 回调记录列表页
- [ ] 手动重试功能

### Step 3: 联调（0.5 天）
- [ ] 端到端测试
- [ ] 异常场景测试

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/user/Callbacks/
├── index.tsx
├── CallbacksPage.tsx
└── components/
    └── CallbackTable.tsx
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/index.tsx
```
