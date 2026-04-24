# XBIS 开发规则文档

> 版本：v1.0  
> 适用范围：XBIS 前端项目（用户端 + 管理端）  
> 生效日期：2026-04-23

---

## 1. 命名规范

### 1.1 通用命名原则

- **语义优先**：命名必须准确表达用途，禁止无意义命名（如 `a`, `b`, `data1`, `temp`）
- **英文命名**：所有标识符必须使用英文，禁止拼音或中英混合
- **避免缩写**：除业界通用缩写外（如 `api`, `id`, `url`），禁止自定义缩写

### 1.2 变量与函数命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `userList`, `isLoading`, `currentPage` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_PAGE_SIZE` |
| 布尔变量 | `is` / `has` / `can` / `should` 前缀 | `isAuthenticated`, `hasPermission` |
| 函数 | camelCase，动词开头 | `fetchUserList`, `handleSubmit`, `validateForm` |
| 事件处理函数 | `handle` + 事件名 | `handleClick`, `handleSearch` |
| 异步函数 | 动词用 `fetch` / `load` / `submit` | `fetchData`, `loadConfig`, `submitForm` |
| 回调函数 | `on` + 动作 | `onChange`, `onSubmit`, `onItemSelect` |
| 转换函数 | `to` / `from` / `format` 前缀 | `toCamelCase`, `formatDate`, `fromDTO` |

### 1.3 类型与接口命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Interface | PascalCase，名词或形容词 | `UserProfile`, `ApiKeyConfig`, `Editable` |
| Type | PascalCase，描述性 | `JobStatus`, `PermissionLevel` |
| Enum | PascalCase，单数名词 | `ExecutionMode`, `RiskLevel` |
| 泛型参数 | 单个大写字母或 PascalCase | `T`, `TData`, `TResponse` |
| Props 类型 | `{ComponentName}Props` | `AbilityCardProps`, `TaskItemProps` |
| State 类型 | `{ComponentName}State` | `DashboardState`, `FormState` |

### 1.4 React 组件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 组件名 | PascalCase | `AbilityCard`, `TaskListPage` |
| 组件文件 | PascalCase，与组件名一致 | `AbilityCard.tsx`, `TaskListPage.tsx` |
| 自定义 Hook | `use` + PascalCase | `useAuth`, `useTaskList`, `usePagination` |
| Context | `{Domain}Context` | `AuthContext`, `ThemeContext` |
| Provider | `{Domain}Provider` | `AuthProvider`, `ThemeProvider` |
| HOC | `with` + 能力名 | `withAuth`, `withPermission` |

### 1.5 路由与页面命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 路由常量 | 大写 SNAKE_CASE | `API_KEYS`, `TASK_CENTER` |
| 路由路径 | kebab-case | `/api-keys`, `/task-center` |
| 动态参数 | `:paramName` | `/tasks/:jobId`, `/abilities/:apiId` |
| 页面组件 | `{功能}Page` | `ApiKeysPage`, `TaskCenterPage` |
| 嵌套路由 | 父级前缀保持一致 | `/admin/users`, `/admin/users/:id` |

### 1.6 CSS / 样式命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Design Token | kebab-case | `--color-brand-default`, `--space-4` |
| Tailwind 类 | 遵循 Tailwind 约定 | `flex`, `items-center`, `gap-4` |
| 自定义类名 | `xbis-` 前缀 + kebab-case | `xbis-card-hover`, `xbis-ai-pulse` |
| 动画名 | `xbis-` 前缀 + 动作 | `xbis-fade-in`, `xbis-slide-up` |

---

## 2. 组件使用规范

### 2.1 组件分层使用原则

```
页面层 (pages/) ──► 只能使用 blocks/ + layout/ + business/ + base/
页面块层 (blocks/) ──► 只能使用 business/ + base/
业务层 (business/) ──► 只能使用 base/
布局层 (layout/) ──► 只能使用 base/
基础层 (base/) ──► 只能使用 Ant Design + tokens/
```

**禁止反向依赖**：下层组件禁止引用上层组件。

### 2.2 基础组件使用规则

- **必须使用 `@xbis/components/base`** 进行二次封装后的组件，禁止直接使用裸 Ant Design 组件（除非 base 层未覆盖）
- 所有基础组件必须接入 Design Tokens（`@xbis/tokens`）
- 禁止在业务代码中覆盖基础组件的核心样式（颜色、字体、圆角、阴影）

### 2.3 业务组件使用规则

- 业务组件必须接收类型化的 Props，使用 `@xbis/shared` 中的类型定义
- 业务组件保持纯展示，数据获取逻辑上提到页面层或自定义 Hook
- 同一业务实体在不同页面展示时，必须使用同一业务组件

### 2.4 布局组件使用规则

- 新页面必须使用页面模板骨架（`ListPageShell`, `DetailPageShell`, `FormPageShell`, `TaskPageShell`, `AbilityPageShell`）
- 布局组件通过 props 配置，禁止复制布局代码到页面文件
- 用户端和管理端共享同一套布局组件，通过配置区分表现

### 2.5 组件 Props 规范

```typescript
// 必须导出 Props 类型
export interface ButtonProps extends AntButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger' | 'link';
  size?: 'small' | 'medium' | 'large';
}

// 必须提供默认值（通过解构赋值）
const Button: React.FC<ButtonProps> = ({ 
  variant = 'primary', 
  size = 'medium',
  children,
  ...rest 
}) => {
  // ...
};

// 事件回调必须类型化
interface ListProps {
  items: Item[];
  onItemClick: (item: Item, index: number) => void;
  onRefresh?: () => Promise<void>;
}
```

---

## 3. 状态管理规范

### 3.1 状态分层

| 层级 | 管理范围 | 技术方案 | 示例 |
|------|----------|----------|------|
| 全局状态 | 跨页面共享 | Zustand + persist | 用户信息、认证状态、主题 |
| 页面状态 | 单页面内共享 | useState / useReducer | 列表数据、筛选条件、分页 |
| 组件状态 | 组件内部 | useState | 展开/收起、hover、局部 UI |
| 服务端状态 | 服务端数据缓存 | React Query / SWR（未来引入）| API 数据、缓存、重试 |

### 3.2 Zustand Store 规范

```typescript
// Store 命名：use{Domain}Store
interface AuthState {
  // State
  accessToken: string | null;
  user: User | null;
  isAuthenticated: boolean;
  
  // Actions（必须是同步函数）
  setAuth: (data: AuthData) => void;
  setUser: (user: User) => void;
  logout: () => void;
  
  // Getters（计算属性）
  getAccessToken: () => string | null;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      // 初始状态
      accessToken: null,
      user: null,
      isAuthenticated: false,
      
      // Actions
      setAuth: (data) => set({ 
        accessToken: data.accessToken, 
        user: data.user, 
        isAuthenticated: true 
      }),
      setUser: (user) => set({ user }),
      logout: () => set({ 
        accessToken: null, 
        user: null, 
        isAuthenticated: false 
      }),
      
      // Getters
      getAccessToken: () => get().accessToken,
    }),
    { name: 'xbis-auth-storage' }
  )
);
```

**规则**：
- Store 中禁止直接调用 API（API 调用在页面层或自定义 Hook 中）
- Store 中的 Action 必须是同步函数
- 异步逻辑封装为自定义 Hook：`useAuthActions()`
- 持久化存储只用于认证信息、用户偏好设置

### 3.3 页面状态规范

```typescript
// 使用统一的页面状态机
type PageStatus = 'idle' | 'loading' | 'empty' | 'error';

interface PageState<T> {
  status: PageStatus;
  data: T | null;
  error: Error | null;
}

// 页面组件中
const [pageState, setPageState] = useState<PageState<Task[]>>({
  status: 'loading',
  data: null,
  error: null,
});

// 状态切换必须完整
setPageState({ status: 'idle', data: response.items, error: null });
```

### 3.4 表单状态规范

- 简单表单：使用 `useState` 管理字段值
- 复杂表单：使用 `useReducer` 或 Ant Design `Form` 组件
- 表单校验：优先使用 Ant Design `Form` 的 `rules`，复杂校验提取为独立函数
- 提交状态：使用 `FormPageShell` 的 `submitting` 状态

---

## 4. API 调用规范

### 4.1 API 分层结构

```
packages/shared/src/api/
├── client.ts          # Axios 实例 + 拦截器
├── services.ts        # API 服务定义（按领域分组）
└── hooks/             # React Query hooks（未来）
    ├── useTasks.ts
    ├── useAbilities.ts
    └── useUser.ts
```

### 4.2 API 服务命名

```typescript
// 按领域分组，使用对象嵌套
export const userApi = {
  auth: {
    login: (data: LoginRequest) => apiClient.post('/portal-api/v1/auth/login', data),
    me: () => apiClient.get('/portal-api/v1/auth/me'),
  },
  tasks: {
    list: (params?: TaskListParams) => apiClient.get('/portal-api/v1/tasks', { params }),
    get: (jobId: string) => apiClient.get(`/portal-api/v1/tasks/${encodePathParam(jobId)}`),
    create: (data: CreateTaskRequest) => apiClient.post('/portal-api/v1/tasks', data),
    cancel: (jobId: string) => apiClient.post(`/portal-api/v1/tasks/${encodePathParam(jobId)}/cancel`),
  },
};

// 管理端 API
export const adminApi = {
  users: {
    list: (params?: UserListParams) => apiClient.get('/admin-api/v1/users', { params }),
    setStatus: (id: string, data: StatusUpdateRequest) => 
      apiClient.post(`/admin-api/v1/users/${encodePathParam(id)}/status`, data),
  },
};
```

**规则**：
- API 路径使用 kebab-case
- 路径参数必须使用 `encodePathParam()` 编码
- 请求/响应类型定义在 `@xbis/shared` 的 `types/index.ts` 中
- GET 请求参数使用 `params`，POST/PUT 请求体使用 `data`

### 4.3 API 调用位置

- **禁止在组件内部直接调用 `axios`**，必须通过 `apiClient` 或服务对象
- **禁止在 Store 中调用 API**
- API 调用应在以下位置：
  1. 页面组件的 `useEffect` 中
  2. 自定义 Hook 中（推荐）
  3. 事件处理函数中

### 4.4 错误处理

```typescript
// 页面层错误处理
const loadData = async () => {
  setPageState({ status: 'loading', data: null, error: null });
  try {
    const response = await userApi.tasks.list(params);
    const items = response?.data?.items || [];
    setPageState({ 
      status: items.length > 0 ? 'idle' : 'empty', 
      data: items, 
      error: null 
    });
  } catch (error) {
    setPageState({ 
      status: 'error', 
      data: null, 
      error: error as Error 
    });
    // 全局错误已由 apiClient 拦截器处理
  }
};
```

**规则**：
- API 错误由 `apiClient` 拦截器统一处理（401 刷新/登出、全局提示）
- 页面层只需捕获错误并更新状态，无需重复提示
- 业务错误（如参数校验失败）在页面层处理

---

## 5. 页面结构规范

### 5.1 页面文件结构

```tsx
// 1. 导入（按来源分组）
import React, { useState, useEffect } from 'react';           // React 内置
import { useNavigate, useParams } from 'react-router-dom';    // 路由
import { Card, Button } from '@xbis/components/base';         // 基础组件
import { AbilityCard } from '@xbis/components/business';      // 业务组件
import { AbilityPageShell } from '@xbis/components/layout';   // 布局组件
import { AbilityGrid } from '@xbis/components/blocks';        // 页面块
import { userApi } from '@xbis/shared';                       // API
import type { ApiMarketItem } from '@xbis/shared';            // 类型

// 2. 类型定义（如需要）
interface AbilityPageParams {
  apiId: string;
}

// 3. 组件定义
const AbilityCenterPage: React.FC = () => {
  // 3.1 Hooks（按执行顺序）
  const navigate = useNavigate();
  const { apiId } = useParams<AbilityPageParams>();
  
  // 3.2 状态
  const [pageState, setPageState] = useState<PageState<ApiMarketItem[]>>({...});
  
  // 3.3 副作用
  useEffect(() => {
    loadAbilities();
  }, [selectedCategory]);
  
  // 3.4 事件处理
  const handleAbilityClick = (ability: ApiMarketItem) => {
    navigate(`/abilities/${ability.apiId}`);
  };
  
  // 3.5 渲染
  return (
    <AbilityPageShell
      title="能力中心"
      mode="grid"
      status={pageState.status}
      // ...
    >
      <AbilityGrid abilities={pageState.data} onAbilityClick={handleAbilityClick} />
    </AbilityPageShell>
  );
};

// 4. 导出
export default AbilityCenterPage;
```

### 5.2 页面状态机

每个页面必须实现完整的状态机：

```
loading ──► idle ──► error
   │           │
   ▼           ▼
  empty ◄─────┘
```

- `loading`：首次加载、刷新、筛选变化时
- `idle`：数据正常展示
- `empty`：数据为空
- `error`：加载失败

### 5.3 页面模板选择

| 页面类型 | 使用模板 | 示例 |
|----------|----------|------|
| 列表页 | `ListPageShell` | 用户列表、调用记录、发票列表 |
| 详情页 | `DetailPageShell` | 能力详情、任务详情、用户详情 |
| 表单页 | `FormPageShell` | 创建/编辑表单、配置页面 |
| 任务相关 | `TaskPageShell` | 任务中心、任务监控 |
| 能力相关 | `AbilityPageShell` | 能力中心、能力测试 |

---

## 6. 路由命名习惯

### 6.1 路由常量定义

```typescript
// packages/shared/src/constants/index.ts
export const ROUTES = {
  USER: {
    HOME: '/',
    LOGIN: '/login',
    DASHBOARD: '/dashboard',
    API_KEYS: '/api-keys',
    API_MARKET: '/api-market',
    API_MARKET_DETAIL: (apiId: string) => `/api-market/${encodeURIComponent(apiId)}`,
    TASKS: '/tasks',
    TASK_DETAIL: (jobId: string) => `/tasks/${encodeURIComponent(jobId)}`,
    TASK_CREATE: '/tasks/create',
    ABILITIES: '/abilities',
    ABILITY_DETAIL: (apiId: string) => `/abilities/${encodeURIComponent(apiId)}`,
    BILLING: '/billing',
    ACCOUNT: '/account',
  },
  ADMIN: {
    HOME: '/admin',
    LOGIN: '/admin/login',
    DASHBOARD: '/admin/dashboard',
    USERS: '/admin/users',
    USER_DETAIL: (id: string) => `/admin/users/${id}`,
    TASKS: '/admin/tasks',
    ABILITIES: '/admin/abilities',
    SETTINGS: '/admin/settings',
  },
};
```

### 6.2 路由命名规则

- 用户端路由：无前缀，如 `/tasks`, `/abilities`
- 管理端路由：以 `/admin` 前缀，如 `/admin/users`
- 动态参数：使用 `:paramName`，如 `/tasks/:jobId`
- 嵌套路由：保持层级一致，如 `/admin/users/:id/permissions`
- 避免深层嵌套：最多 3 层，如 `/admin/settings/billing` ✓，`/admin/settings/billing/plans/tiers` ✗

### 6.3 路由与文件映射

```
路由路径              页面文件
─────────────────────────────────────────
/abilities           pages/AbilityCenterPage.tsx
/abilities/:apiId    pages/AbilityDetailPage.tsx
/abilities/:apiId/test    pages/AbilityTestPage.tsx
tasks                pages/TaskCenterPage.tsx
tasks/:jobId         pages/TaskDetailPage.tsx
tasks/create         pages/TaskCreatePage.tsx
/admin/users         pages/admin/UserListPage.tsx
/admin/users/:id     pages/admin/UserDetailPage.tsx
```

---

## 7. 文件命名习惯

### 7.1 文件命名规范

| 类型 | 命名规范 | 示例 |
|------|----------|------|
| 页面组件 | PascalCase + `Page` 后缀 | `DashboardPage.tsx`, `TaskCenterPage.tsx` |
| 组件文件 | PascalCase，与组件名一致 | `AbilityCard.tsx`, `TaskItem.tsx` |
| 自定义 Hook | camelCase，`use` 前缀 | `useAuth.ts`, `useTaskList.ts` |
| 工具函数 | camelCase | `formatDate.ts`, `validateEmail.ts` |
| 类型定义 | PascalCase，`.types.ts` 后缀 | `api.types.ts`, `user.types.ts` |
| 常量文件 | UPPER_SNAKE_CASE 导出 | `constants/index.ts` |
| 样式文件 | 与组件同名，`.module.css` | `Button.module.css` |
| 测试文件 | 与被测文件同名，`.test.ts` | `Button.test.tsx` |

### 7.2 文件组织

```
src/
├── pages/                    # 页面组件
│   ├── DashboardPage.tsx
│   ├── TaskCenterPage.tsx
│   └── admin/
│       └── UserListPage.tsx
├── components/               # 页面级组件（逐步迁移到 @xbis/components）
├── hooks/                    # 自定义 Hooks
│   ├── useAuth.ts
│   └── useTaskList.ts
├── utils/                    # 工具函数
│   ├── formatDate.ts
│   └── validators.ts
├── types/                    # 页面级类型（优先使用 @xbis/shared）
└── styles/                   # 页面级样式
```

---

## 8. 目录组织规则

### 8.1 Monorepo 目录结构

```
xbis-front/
├── packages/
│   ├── user/                 # 用户端应用
│   │   ├── src/
│   │   │   ├── pages/        # 页面组件
│   │   │   ├── components/   # 页面级组件（逐步迁移）
│   │   │   ├── hooks/        # 自定义 Hooks
│   │   │   ├── utils/        # 工具函数
│   │   │   ├── App.tsx       # 路由配置
│   │   │   └── main.tsx      # 入口文件
│   │   ├── index.html
│   │   └── package.json
│   │
│   ├── admin/                # 管理端应用
│   │   └── src/
│   │       └── ...           # 与用户端相同结构
│   │
│   ├── shared/               # 共享代码
│   │   ├── src/
│   │   │   ├── api/          # API 客户端 + 服务
│   │   │   │   ├── client.ts
│   │   │   │   └── services.ts
│   │   │   ├── types/        # TypeScript 类型
│   │   │   │   └── index.ts
│   │   │   ├── constants/    # 常量
│   │   │   │   └── index.ts
│   │   │   ├── store/        # Zustand Store
│   │   │   │   └── auth.ts
│   │   │   └── utils/        # 共享工具函数
│   │   └── package.json
│   │
│   ├── components/           # 组件库（新）
│   │   ├── base/             # 基础组件
│   │   ├── business/         # 业务组件
│   │   ├── layout/           # 布局组件 + 页面模板
│   │   ├── blocks/           # 页面块组件
│   │   ├── index.ts
│   │   └── package.json
│   │
│   ├── server/               # 后端服务
│   │   └── src/
│   │       ├── routes/
│   │       ├── middleware/
│   │       ├── utils/
│   │       └── main.ts
│   │
│   └── tokens/               # Design Tokens（已存在）
│       ├── colors.ts
│       ├── spacing.ts
│       ├── typography.ts
│       ├── radius.ts
│       ├── shadows.ts
│       └── index.ts
│
├── docs/                     # 项目文档
│   ├── dev-rules.md          # 本文件
│   ├── design-tokens.md
│   ├── component-architecture.md
│   ├── page-templates.md
│   └── ...
│
├── tokens/                   # Design Tokens（根目录引用）
├── package.json              # 根 package.json
├── pnpm-workspace.yaml       # pnpm workspace 配置
└── AI_RULES.md               # AI 开发助手必读
```

### 8.2 导入顺序

```typescript
// 1. React 内置
import React, { useState, useEffect } from 'react';

// 2. 第三方库
import { useNavigate } from 'react-router-dom';
import { Button } from 'antd';

// 3. 内部共享（@xbis/*）
import { userApi } from '@xbis/shared';
import { Card, Tag } from '@xbis/components/base';
import { AbilityCard } from '@xbis/components/business';
import { AbilityPageShell } from '@xbis/components/layout';
import { AbilityGrid } from '@xbis/components/blocks';

// 4. 类型导入
import type { ApiMarketItem } from '@xbis/shared';

// 5. 本地导入
import { useTaskList } from './hooks/useTaskList';
import { formatDate } from './utils/formatDate';
```

---

## 9. 禁止事项

### 9.1 样式相关（严格禁止）

| 禁止项 | 说明 | 正确做法 |
|--------|------|----------|
| **禁止直接写散落样式** | 禁止在组件中使用 `style={{ margin: 10, color: 'red' }}` 等硬编码样式 | 使用 Design Tokens：`style={{ margin: space['4'], color: colors.text.primary }}` |
| **禁止覆盖 Ant Design 全局样式** | 禁止直接修改 `.ant-*` 类名样式 | 使用 `ConfigProvider` 或基础组件二次封装 |
| **禁止内联样式写死颜色/尺寸** | 禁止 `#1890ff`、`14px`、`8px` 等硬编码 | 使用 `@xbis/tokens` 中的语义化 Token |
| **禁止 CSS 魔法数字** | 禁止 `margin: 23px`、`width: 137px` 等无意义数字 | 使用 spacing scale：`space['6']` = 24px |
| **禁止 !important** | 禁止使用 `!important` 覆盖样式 | 提高选择器优先级或重构组件 |

### 9.2 组件相关（严格禁止）

| 禁止项 | 说明 | 正确做法 |
|--------|------|----------|
| **禁止重复封装功能等价组件** | 禁止创建与已有组件功能完全相同的组件 | 复用已有组件，通过 props 扩展 |
| **禁止在页面中直接写基础 UI** | 禁止在页面文件中直接写 `<div style={{...}}>` 作为按钮/卡片 | 使用 `@xbis/components/base` |
| **禁止组件文件超过 300 行** | 单组件文件行数超过 300 行必须拆分 | 提取子组件或业务逻辑到 Hook |
| **禁止 Props 穿透超过 3 层** | 禁止通过多层传递 props（prop drilling） | 使用 Context 或状态管理 |
| **禁止在组件内部定义组件** | 禁止在函数组件内部定义其他组件 | 提取为独立组件文件 |

### 9.3 API 相关（严格禁止）

| 禁止项 | 说明 | 正确做法 |
|--------|------|----------|
| **禁止在 Store 中调用 API** | Zustand Store 中禁止异步请求 | 在页面层或自定义 Hook 中调用 |
| **禁止在组件中直接调用 axios** | 禁止 `axios.get()` 裸调用 | 使用 `apiClient` 或服务对象 |
| **禁止硬编码 API 路径** | 禁止在页面中写 `/api/v1/...` | 使用 `services.ts` 中定义的服务 |
| **禁止忽略 API 错误** | 禁止 `.catch(() => {})` 空捕获 | 至少记录错误并更新页面状态 |
| **禁止在循环中串行调用 API** | 禁止 `for (...) { await api.call() }` | 使用 `Promise.all()` 并行调用 |

### 9.4 状态管理相关（严格禁止）

| 禁止项 | 说明 | 正确做法 |
|--------|------|----------|
| **禁止一个页面使用多个状态管理方式** | 禁止同时使用 Zustand + useState + useReducer 管理同类数据 | 统一使用一种方式 |
| **禁止在 useEffect 中直接 setState 循环** | 禁止导致无限循环的依赖设置 | 检查依赖数组，使用函数式更新 |
| **禁止滥用 Context** | 禁止用 Context 传递频繁变化的数据 | 使用 Zustand 或状态提升 |
| **禁止在 render 中调用 API** | 禁止在组件 render 阶段发起请求 | 在 useEffect 或事件处理中调用 |

### 9.5 类型相关（严格禁止）

| 禁止项 | 说明 | 正确做法 |
|--------|------|----------|
| **禁止滥用 any** | 禁止 `any` 类型（除第三方库无类型定义外） | 使用 `unknown` 或定义具体类型 |
| **禁止隐式 any** | 禁止 TypeScript 隐式推断为 any | 开启 `noImplicitAny: true` |
| **禁止在类型定义中使用 any[]** | 禁止无类型数组 | 使用 `Item[]` 或 `Array<Item>` |
| **禁止导出未使用的类型** | 禁止死代码 | 定期清理未使用的类型定义 |

### 9.6 文件组织相关（严格禁止）

| 禁止项 | 说明 | 正确做法 |
|--------|------|----------|
| **禁止循环依赖** | 禁止 A 导入 B，B 又导入 A | 提取公共部分到独立文件 |
| **禁止相对路径超过 2 层** | 禁止 `../../../components/Button` | 使用路径别名或提取到更近的位置 |
| **禁止一个文件导出多个不相关组件** | 禁止一个文件导出 2 个以上独立组件 | 拆分为独立文件 |
| **禁止提交 console.log** | 禁止生产代码中包含调试日志 | 使用 `console.error` 或日志库 |

---

## 10. 旧 API 平台页面向能力/任务平台迁移命名规则

### 10.1 迁移背景

项目正在从"API 平台"向"能力平台 + 任务平台"架构迁移。旧页面使用 `api-*` 命名，新页面使用 `ability-*` / `task-*` 命名。

### 10.2 命名映射表

| 旧概念（API 平台） | 新概念（能力/任务平台） | 说明 |
|-------------------|------------------------|------|
| API / 接口 | Ability / 能力 | 核心概念替换 |
| API Market | Ability Center / 能力中心 | 页面名称替换 |
| API Keys | Access Keys / 访问密钥 | 概念扩展 |
| Invocation / 调用 | Task / 任务 | 执行单元替换 |
| Invocation Record | Task History / 任务历史 | 记录概念替换 |
| ApiMarketItem | Ability / Capability | 数据模型替换 |
| apiId | abilityId | 字段名替换 |
| apiName | abilityName | 字段名替换 |
| targetAgent + targetSkill | executor + ability | 执行模型替换 |

### 10.3 文件迁移命名规则

```
旧文件                          新文件
─────────────────────────────────────────────────────────
ApiMarket.tsx              →   AbilityCenterPage.tsx
ApiMarketDetail.tsx        →   AbilityDetailPage.tsx
Invocations.tsx            →   TaskCenterPage.tsx
ApiKeys.tsx                →   AccessKeyPage.tsx
Dashboard.tsx（旧）         →   DashboardPage.tsx（新架构）
```

### 10.4 路由迁移命名规则

```
旧路由              新路由              状态
─────────────────────────────────────────────────
/api-market        /abilities         迁移完成
/api-market/:id    /abilities/:id     迁移完成
/invocations       /tasks             迁移完成
/invocations/:id   /tasks/:jobId      迁移完成
/api-keys          /access-keys       迁移完成
```

### 10.5 组件迁移命名规则

```
旧组件名                新组件名
─────────────────────────────────────────────────
ApiMarketCard           AbilityCard
ApiMarketGrid           AbilityGrid
ApiMarketFilter         AbilityFilterBar
InvocationItem          TaskItem
InvocationList          TaskList
InvocationDetail        TaskDetailPanel
ApiKeyItem              AccessKeyItem
```

### 10.6 API 迁移命名规则

```
旧 API 路径                  新 API 路径
─────────────────────────────────────────────────
/portal-api/v1/apis/*       /portal-api/v1/abilities/*
/portal-api/v1/invocations/* /portal-api/v1/tasks/*
/admin-api/v1/apis/*        /admin-api/v1/abilities/*
```

### 10.7 迁移阶段标记

在代码中使用注释标记迁移状态：

```typescript
// TODO-MIGRATION-P0: 旧 API 平台代码，需迁移到能力平台
// 迁移目标: AbilityCenterPage.tsx
// 预计完成: 2026-05-15
const ApiMarket: React.FC = () => {
  // ...
};

// MIGRATION-DONE: 已从 ApiMarketDetail.tsx 迁移
// 原文件: packages/user/src/pages/ApiMarketDetail.tsx
const AbilityDetailPage: React.FC = () => {
  // ...
};
```

### 10.8 迁移优先级

| 优先级 | 范围 | 说明 |
|--------|------|------|
| P0 | 核心概念替换 | `api` → `ability`, `invocation` → `task` |
| P1 | 页面级迁移 | 旧页面文件迁移到新命名 |
| P2 | 组件级迁移 | 旧组件迁移到 `@xbis/components` |
| P3 | API 路径迁移 | 后端 API 路径更新 |

---

## 11. 代码审查清单

### 11.1 提交前自检

- [ ] 所有变量/函数/组件命名符合规范
- [ ] 没有直接写散落样式（使用 Design Tokens）
- [ ] 没有重复封装功能等价组件
- [ ] API 调用通过 `apiClient` 或服务对象
- [ ] 页面使用正确的页面模板骨架
- [ ] 状态管理符合分层规范
- [ ] TypeScript 类型完整，没有隐式 any
- [ ] 没有 `console.log` 调试代码
- [ ] 没有循环依赖
- [ ] 文件行数不超过 300 行（页面文件不超过 500 行）

### 11.2 CR 重点检查项

- [ ] 是否使用了已有的业务组件（避免重复实现）
- [ ] 是否正确使用了页面模板
- [ ] API 错误处理是否完整
- [ ] 状态机是否完整（loading/empty/error/idle）
- [ ] 是否有内存泄漏（useEffect 清理、事件监听）
- [ ] 移动端适配是否考虑（用户端）

---

*本文档由开发团队共同维护，新增规则需经团队评审后更新。*
