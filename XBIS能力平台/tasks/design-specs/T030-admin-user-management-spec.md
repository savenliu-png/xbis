# T030 管理端用户管理 — 工程级设计方案（Design Spec）

> 任务卡：T030-admin-user-management.md  
> 状态：设计确认  
> 输出日期：2026-04-24

---

## 1. 页面结构

### 1.1 层级结构

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
        │   └── 角色筛选
        ├── DataTable
        │   ├── 用户列表
        │   └── 操作列（查看/禁用/配置）
        └── Pagination
```

### 1.2 用户详情（抽屉/新页面）

```
UserDetailDrawer / UserDetailPage
├── UserProfileCard
│   ├── 头像/名称/邮箱
│   ├── 状态/角色
│   └── 注册时间/最后登录
├── Tabs
│   ├── Tab 1: 基本信息
│   ├── Tab 2: 订单列表
│   ├── Tab 3: 最近任务
│   └── Tab 4: 限制配置
```

### 1.3 模板使用

| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 列表 | ListPageShell | 列表页模板 |
| 详情 | DetailPageShell | 详情页模板（抽屉或独立页面） |

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button, Input, Table, Tag, Badge, Switch, Modal, Tabs, Skeleton, Empty, Pagination | 标准用法 |

### 2.2 business/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| UserCard | 用户卡片 | **新建** — 详情页顶部用户信息卡片 |

### 2.3 blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| UserTable | 用户表格 | **新建** — 含脱敏展示 |
| UserDetailPanel | 用户详情 | **新建** — 抽屉/页面内详情 |
| OrderList | 订单列表 | **新建** — 用户订单子表格 |
| LimitConfig | 限制配置 | **新建** — 表单编辑用户限制 |

---

## 3. 数据流

### 3.1 列表加载

```
[进入 /admin/users]
    │
    ▼
[API: GET /admin-api/v1/users]
    │
    ▼
[渲染用户列表（脱敏展示）]
```

### 3.2 用户禁用

```
[点击 Switch]
    │
    ▼
[Modal 确认]
    │
    ▼
[API: PUT /admin-api/v1/users/:id/status]
    │
    ├── 成功 → 更新列表
    └── 失败 → 提示原因（如「有进行中的任务」）
```

### 3.3 限制配置

```
[点击「配置限制」]
    │
    ▼
[Drawer 打开]
    │
    ▼
[加载当前限制配置]
    │
    ▼
[编辑表单]
    │
    ▼
[API: PUT /admin-api/v1/users/:id/limits]
```

---

## 4. 状态管理

```typescript
interface AdminUserState {
  pageState: 'loading' | 'idle' | 'error';
  users: AdminUserItem[];
  total: number;
  filters: { keyword?: string; status?: string; role?: string; page: number; pageSize: number };
  detailDrawerOpen: boolean;
  selectedUser: AdminUserItem | null;
  userDetail: UserDetailAdmin | null;
  activeDetailTab: string;
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 用户列表 | GET | `/admin-api/v1/users` | 支持筛选分页 |
| 用户详情 | GET | `/admin-api/v1/users/:id` | 含订单/任务/配置 |
| 状态更新 | PUT | `/admin-api/v1/users/:id/status` | 启用/禁用/封禁 |
| 限制配置 | PUT | `/admin-api/v1/users/:id/limits` | 更新用户限制 |

---

## 6. 用户交互流程

```
[进入用户管理]
    │
    ▼
[浏览用户列表]
    │
    ├── 搜索/筛选用户
    ├── 点击 Switch 禁用/启用
    ├── 点击「查看详情」
    │       │
    │       ▼
    │   [Drawer/新页面打开]
    │       │
    │       ├── 查看基本信息
    │       ├── 查看订单列表
    │       ├── 查看最近任务
    │       └── 编辑限制配置
    │
    └── 点击「配置限制」
            │
            ▼
        [Drawer 编辑限制]
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 敏感信息 | 手机号/邮箱脱敏展示（138****8000） |
| 禁用失败 | 提示「用户有进行中的任务，请先处理」 |
| 无用户 | Empty 组件 |

---

## 8. 性能优化

- 列表虚拟滚动（>100 条）
- 详情懒加载（打开 Drawer 时才请求详情）
- 筛选 Debounce 300ms

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| **敏感信息泄露** | 用户隐私数据 | 数据脱敏；权限控制 |
| **误禁用用户** | 影响用户正常使用 | 二次确认；提示进行中的任务 |

---

## 10. 开发步骤拆分

### Step 1: 列表页（1 天）
- [ ] 用户列表 + 筛选 + 分页
- [ ] 用户禁用功能

### Step 2: 详情页（1 天）
- [ ] 用户详情 Drawer
- [ ] 订单/任务/限制 Tab

### Step 3: 配置功能（0.5 天）
- [ ] 限制配置表单
- [ ] API 联调

### Step 4: 优化（0.5 天）
- [ ] 数据脱敏
- [ ] 错误处理

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/admin/UserManagement/
├── index.tsx
├── UserManagementPage.tsx
├── components/
│   ├── UserTable.tsx
│   ├── UserDetailPanel.tsx
│   ├── OrderList.tsx
│   └── LimitConfig.tsx
└── hooks/
    └── useUserManagement.ts

packages/components/business/UserCard/
└── index.tsx
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/admin.tsx
```
