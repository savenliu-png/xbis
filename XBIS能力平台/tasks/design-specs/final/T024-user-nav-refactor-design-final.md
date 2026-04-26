# T024 用户端导航重构 — 最终可开发设计方案

> 任务卡: [T024-user-nav-refactor.md](../T024-user-nav-refactor.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
├── TopNav
│   ├── Logo
│   ├── NavMenu
│   ├── NotificationBell
│   └── UserMenu
├── Sidebar（可选）
│   └── 二级导航
└── MainContent
    └── 子路由页面
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Badge | 徽章 |
| Drawer | 移动端抽屉 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| NavMenu | 导航菜单 |
| NotificationBell | 通知铃铛 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| TopNav | 顶部导航 | **新建** |
| UserMenu | 用户菜单 | **新建** |
| MobileNav | 移动端导航 | **新建** — 含交互定义 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| AppShell | 应用外壳 |

---

## 3. 数据流

```
[访问旧路由]
    │
    ▼
[路由守卫匹配]
    │
    ▼
[302 重定向到新路由]
```

### 旧路由映射表

```typescript
const routeRedirects = [
  { from: '/old-api-market', to: '/abilities', status: 302 },
  { from: '/old-invocations', to: '/jobs', status: 302 },
  // ... 其他映射
];
```

---

## 4. 状态管理

```typescript
interface NavState {
  activeKey: string;
  navItems: NavItem[]; // 过滤后的菜单项
  mobileDrawerOpen: boolean;
}

// 导航权限配置
interface NavItem {
  key: string;
  label: string;
  path: string;
  requiredRole?: ('user' | 'developer' | 'admin')[];
  badge?: boolean; // 是否支持徽章
}

const navPermissions = {
  abilities: ['user', 'developer', 'admin'],
  jobs: ['user', 'developer', 'admin'],
  billing: ['user', 'developer', 'admin'],
  developer: ['developer', 'admin'],
  admin: ['admin'],
};
```

---

## 5. API 调用

无 API 调用。

---

## 6. 用户交互流程

```
[用户访问应用]
    │
    ▼
[展示导航]
    │
    ├── 点击菜单项 ──► 路由跳转
    ├── 点击通知 ──► 打开通知面板
    ├── 点击用户头像 ──► 打开用户菜单
    └── 移动端点击汉堡菜单 ──► 打开 Drawer
            │
            ▼
        [Drawer 打开/关闭动画 300ms]
        [支持从左侧边缘滑动手势打开]
        [点击遮罩层关闭]
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 访问无权限页面 | 跳转 403 |
| 访问旧路由 | 302 重定向 |

---

## 8. 性能优化

- 导航配置静态化
- 菜单项使用 React.memo

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 旧路由兼容 | 低 | 用户可能收藏旧链接 | 保留 30 天 302 重定向 |

---

## 10. 开发步骤拆分

### Step 1: 导航组件（0.5 天）
- [ ] TopNav + NavMenu（含权限过滤）

### Step 2: 移动端适配（0.5 天）
- [ ] MobileNav（含动画和手势）

### Step 3: 路由重定向（0.5 天）
- [ ] 旧路由映射表 + 路由守卫
