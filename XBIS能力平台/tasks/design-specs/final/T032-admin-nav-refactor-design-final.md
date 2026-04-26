# T032 管理端导航重构 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AdminLayout
├── AdminSidebar (左侧导航)
│   ├── Logo 区
│   ├── NavMenu (9 项主导航)
│   │   ├── 概览 (/admin/overview)
│   │   ├── 能力管理 (/admin/abilities)
│   │   ├── 执行器管理 (/admin/executors)
│   │   ├── 任务运营 (/admin/operations)
│   │   ├── 用户管理 (/admin/users)
│   │   ├── 套餐配置 (/admin/billing/plans)
│   │   ├── 系统配置 (/admin/settings)
│   │   ├── 日志 (/admin/logs)
│   │   └── 设置 (/admin/profile)
│   └── 底部折叠按钮
├── AdminHeader (顶部栏)
│   ├── 面包屑
│   ├── 全局搜索（占位）
│   └── 用户信息
└── MainContent (主内容区)
    └── 子路由页面
```

### 1.2 模板使用
| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 整体 | AdminLayout | 管理端布局模板 |
| 导航 | AdminSidebar | 侧边栏导航 |

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）
| 组件 | 用途 |
|------|------|
| Button | 折叠按钮 |
| Icon | 导航图标 |
| Badge | 消息/告警徽章 |

### 2.2 business/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| AdminNavMenu | 管理导航菜单 | 9 项菜单，支持权限控制 |

### 2.3 blocks/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| AdminSidebar | 管理侧边栏 | 包含 Logo + NavMenu + 折叠 |

---

## 3. 数据流

### 3.1 导航初始化
```
[管理员访问管理端]
    │
    ▼
[加载 AdminLayout]
    │
    ▼
[渲染 AdminNavMenu]
    │
    ▼
[根据当前路由高亮对应菜单项]
```

### 3.2 权限过滤
```
[获取当前用户角色]
    │
    ▼
[过滤导航项（根据 requiredRole）]
    │
    ▼
[渲染可见菜单]
```

### 3.3 旧路由重定向
```
[访问旧路由 /admin/reviews]
    │
    ▼
[路由守卫匹配]
    │
    ▼
[302 重定向到 /admin/operations]
```

### 3.4 旧路由映射表
```typescript
const adminRouteRedirects = [
  { from: '/admin/reviews', to: '/admin/operations', status: 302 },
  { from: '/admin/settings/old', to: '/admin/settings', status: 302 },
  // ... 其他映射
];
```

---

## 4. 状态管理

```typescript
interface AdminNavState {
  collapsed: boolean;           // 侧边栏折叠状态
  activeKey: string;            // 当前激活菜单
  navItems: AdminNavItem[];     // 过滤后的菜单项
}

interface AdminNavItem {
  key: string;
  label: string;
  icon: string;
  path: string;
  requiredRole: ('admin' | 'superadmin' | 'operator')[];
  badge?: 'alerts' | 'pendingTasks'; // 徽章数据源
}

const navPermissions = {
  overview: ['admin', 'superadmin', 'operator'],
  abilities: ['admin', 'superadmin'],
  executors: ['admin', 'superadmin'],
  operations: ['admin', 'superadmin', 'operator'],
  users: ['admin', 'superadmin'],
  billing: ['admin', 'superadmin'],
  settings: ['admin', 'superadmin'],
  logs: ['admin', 'superadmin'],
  profile: ['admin', 'superadmin', 'operator']
};
```

---

## 5. API 调用

无新增 API。导航配置为前端静态配置，权限由后端接口返回的用户角色决定。

---

## 6. 用户交互流程

```
[管理员访问管理端]
    │
    ▼
[展示侧边栏导航]
    │
    ├── 点击菜单项 ──► 路由跳转 ──► 高亮当前项
    ├── 点击折叠按钮 ──► 侧边栏收起/展开
    └── 访问旧路由 ──► 302 重定向到新路由
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 访问无权限页面 | 跳转 403 |
| 访问不存在的路由 | 跳转 404 |
| 旧路由访问 | 302 重定向（保留 30 天） |

---

## 8. 性能优化

- 导航配置静态化，无需 API 请求
- 菜单项使用 React.memo
- 路由懒加载

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 旧路由兼容 | 管理员可能收藏了旧链接 | 保留 30 天 302 重定向；监控旧路由访问 |
| 用户体验变化 | 导航结构变化需要适应 | 灰度发布；提供引导提示 |
| 权限控制 | 不同角色看到不同菜单 | 后端校验 + 前端过滤 |

---

## 10. 开发步骤

### Step 1: 导航组件（0.5 天）
- [ ] AdminNavMenu 组件（含权限过滤）
- [ ] AdminSidebar 组件
- [ ] 9 项菜单配置

### Step 2: 布局集成（0.5 天）
- [ ] AdminLayout 重构
- [ ] 侧边栏折叠功能
- [ ] 当前项高亮

### Step 3: 路由重定向（0.5 天）
- [ ] 旧路由重定向配置
- [ ] 路由守卫权限校验

### Step 4: 优化（0.5 天）
- [ ] 响应式布局
- [ ] 暗色模式适配
- [ ] 性能优化

---

## 附录：修改文件清单

### 新增文件
```
packages/components/business/AdminNavMenu/
└── index.tsx

packages/components/blocks/AdminSidebar/
└── index.tsx
```

### 修改文件
```
packages/layout/AdminLayout.tsx              # 重构管理端布局
packages/router/admin.tsx                    # 添加重定向路由
packages/shared/src/types/index.ts           # 添加 AdminNavItem 类型
```
