# T034 Callback 机制 — 修正版设计方案（V2）

> 原设计方案: [T034-callback-mechanism-spec.md](../T034-callback-mechanism-spec.md)
> 评审报告: [T034-callback-mechanism-Reviewer.md](../T034-callback-mechanism-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 详细设计回调安全机制 | 第 5 节 API 调用 |
| 2 | 明确退避策略 | 第 5 节 API 调用 |
| 3 | 设置回调超时 | 第 5 节 API 调用 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加数据清理策略 | 第 8 节性能优化 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "回调记录"
    └── ListPageShell
        ├── FilterBar
        │   ├── 任务 ID 搜索
        │   └── 状态筛选
        ├── CallbackTable
        │   └── 回调记录列表
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 重试按钮 |
| Input | 搜索框 |
| Table | 回调记录表格 |
| Tag | 状态标签 |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Pagination | 分页 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| CallbackTable | 回调记录表格 | **新建** — 含重试按钮 |

---

### 3. 数据流

```
[进入 /callbacks]
    │
    ▼
[API: GET /api/v1/callbacks]
    │
    ▼
[渲染回调记录列表]
```

---

### 4. 状态管理

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

### 5. API 调用（V2 修正）【已修复】

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 回调记录查询 | GET | `/api/v1/callbacks` | 查询回调记录 |
| 手动重试 | POST | `/api/v1/callbacks/:id/retry` | 手动触发重试 |

#### 回调安全机制（V2 新增）【已修复】

```typescript
// 签名验证
interface CallbackSignature {
  timestamp: string;
  nonce: string;
  signature: string; // HMAC-SHA256(secret, timestamp + nonce + body)
}

// URL 白名单校验
const allowedCallbackDomains = [
  '*.example.com',
  'partner.com'
];
```

#### 退避策略（V2 新增）【已修复】

```typescript
const retryIntervals = [30, 60, 120]; // 秒
// 第1次重试：30秒后
// 第2次重试：60秒后
// 第3次重试：120秒后
```

#### 回调超时（V2 新增）【已修复】

```typescript
const CALLBACK_TIMEOUT = 30000; // 30 秒
// 超过 30 秒未响应视为失败，触发重试
```

---

### 6. 用户交互流程

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

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无回调记录 | Empty 组件 |
| 重试失败 | Toast 错误提示 |
| 回调 URL 无效 | 标记失败，记录错误 |

---

### 8. 性能优化（V2 修正）【已修复】

```
- 回调记录分页加载
- 状态筛选 Debounce
- 【新增】数据清理策略：
  - 回调记录保留 30 天
  - 成功回调记录保留 7 天
  - 定期清理任务（每天凌晨执行）
```

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 回调风暴 | 高 | 大量任务同时完成触发回调 | 后端限流 + 队列 |
| 重试风暴 | 中 | 失败回调频繁重试 | 指数退避 + 最大 3 次 |
| 回调安全 | 高 | 回调 URL 可能被恶意利用 | 签名验证 + URL 白名单 |

---

### 10. 开发步骤拆分

#### Step 1: 后端机制（2 天）
- [ ] 回调触发逻辑
- [ ] 重试机制（指数退避）【已修复】
- [ ] 回调记录存储
- [ ] 安全机制（签名 + 白名单）【已修复】

#### Step 2: 前端页面（0.5 天）
- [ ] 回调记录列表页
- [ ] 手动重试功能

#### Step 3: 联调（0.5 天）
- [ ] 端到端测试
- [ ] 异常场景测试

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **回调安全** | 未细化 | 新增签名验证 + URL 白名单 |
| **退避策略** | 未明确 | 明确 30/60/120 秒间隔 |
| **回调超时** | 未设置 | 明确 30 秒超时 |
| **数据清理** | 未设计 | 新增保留策略 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（安全机制、退避策略、超时设置）
2. 建议修改项已补充（数据清理）
3. 风险等级：高（需严格测试安全机制）
