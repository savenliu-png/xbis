# T030 管理端用户管理 — 修正版设计方案（V2）

> 原设计方案: [T030-admin-user-management-spec.md](../T030-admin-user-management-spec.md)
> 评审报告: [T030-admin-user-management-Reviewer.md](../T030-admin-user-management-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确限制配置项 | 第 4 节状态管理 |
| 2 | 禁用用户时记录原因 | 第 6 节用户交互流程 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加权限管理功能 | 第 2 节组件拆分 |
| 2 | 增加高级筛选 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "用户管理"
    │   └── Breadcrumb: 概览 / 用户管理
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框（用户名/邮箱）
        │   ├── 状态筛选
        │   ├── 角色筛选
        │   ├── 注册时间范围 【新增】
        │   └── 最后登录时间范围 【新增】
        ├── UserTable
        │   └── 用户列表
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 用户表格 |
| Tag | 状态/角色标签 |
| Switch | 禁用/启用 |
| Modal | 确认弹窗 |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Pagination | 分页 |

#### business/ 层组件（V2 新增）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| UserCard | 用户卡片 | 详情页顶部用户信息卡片 |

#### blocks/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| UserTable | 用户表格 | 含脱敏展示 |
| UserDetailPanel | 用户详情 | 抽屉/页面内详情 |
| OrderList | 订单列表 | 用户订单子表格 |
| LimitConfig | 限制配置 | **已修复** — 表单编辑用户限制 |

#### 限制配置项（V2 新增）【已修复】

```typescript
interface UserLimitConfig {
  maxCallsPerMinute: number;
  maxCallsPerDay: number;
  maxConcurrentJobs: number;
  allowedAbilities: string[];
  ipWhitelist?: string[];
}
```

---

### 3. 数据流

```
[进入 /admin/users]
    │
    ▼
[API: GET /admin-api/v1/users]
    │
    ▼
[渲染用户列表（脱敏展示）]
```

---

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface AdminUserState {
  pageState: 'loading' | 'idle' | 'error';
  users: AdminUserItem[];
  total: number;
  filters: { keyword?: string; status?: string; role?: string; registerStart?: string; registerEnd?: string; lastLoginStart?: string; lastLoginEnd?: string; page: number; pageSize: number };
  detailDrawerOpen: boolean;
  selectedUser: AdminUserItem | null;
  userDetail: UserDetailAdmin | null;
  activeDetailTab: string;
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 用户列表 | GET | `/admin-api/v1/users` | 支持筛选分页 |
| 用户详情 | GET | `/admin-api/v1/users/:id` | 含订单/任务/配置 |
| 状态更新 | PUT | `/admin-api/v1/users/:id/status` | 启用/禁用/封禁 |
| 限制配置 | PUT | `/admin-api/v1/users/:id/limits` | 更新用户限制 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入用户管理]
    │
    ▼
[浏览用户列表]
    │
    ├── 搜索/筛选（含高级筛选）【已修复】
    ├── 点击 Switch 禁用/启用
    │       │
    │       ▼
    │   [Modal 要求填写禁用原因] 【新增】
    │       │
    │       ├── 禁用原因（必填）
    │       ├── 禁用时长（永久/临时）
    │       └── 预计解禁时间（如临时禁用）
    │
    ├── 点击「查看详情」
    │       │
    │       ▼
    │   [Drawer/新页面打开]
    │       │
    │       ├── 查看基本信息
    │       ├── 查看订单列表
    │       ├── 查看最近任务
    │       ├── 编辑限制配置 【已修复】
    │       └── 权限管理 【新增】
    │               │
    │               ├── 角色分配
    │               ├── 能力访问权限
    │               └── API 调用权限
    │
    └── 点击「配置限制」
            │
            ▼
        [Drawer 编辑限制]
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 敏感信息 | 手机号/邮箱脱敏展示（138****8000） |
| 禁用失败 | 提示「用户有进行中的任务，请先处理」 |
| 无用户 | Empty 组件 |

---

### 8. 性能优化

- 列表虚拟滚动（>100 条）
- 详情懒加载（打开 Drawer 时才请求详情）
- 筛选 Debounce 300ms

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 敏感信息泄露 | 高 | 用户隐私数据 | 数据脱敏 + 权限控制 |
| 误禁用用户 | 中 | 影响用户正常使用 | 二次确认 + 原因记录 |

---

### 10. 开发步骤拆分

#### Step 1: 列表页（1 天）
- [ ] 用户列表 + 筛选 + 分页
- [ ] 用户禁用功能（含原因记录）

#### Step 2: 详情页（1 天）【已修复】
- [ ] 用户详情 Drawer
- [ ] 订单/任务/限制 Tab
- [ ] 权限管理功能

#### Step 3: 配置功能（0.5 天）
- [ ] 限制配置表单
- [ ] API 联调

#### Step 4: 优化（0.5 天）
- [ ] 数据脱敏
- [ ] 错误处理

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **限制配置** | 未细化 | 明确 5 项配置 |
| **禁用原因** | 未记录 | 新增原因/时长/解禁时间 |
| **权限管理** | 未设计 | 新增角色/能力/API 权限 |
| **高级筛选** | 未设计 | 新增注册时间/最后登录筛选 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（限制配置细化、禁用原因记录）
2. 建议修改项已补充（权限管理、高级筛选）
3. 风险等级：低
