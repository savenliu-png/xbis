# T024 用户端导航重构 — 修正版设计方案（V2）

> 原设计方案: [T024-user-nav-refactor-spec.md](../T024-user-nav-refactor-spec.md)
> 评审报告: [T024-user-nav-refactor-Reviewer.md](../T024-user-nav-refactor-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确旧路由映射表 | 第 3 节数据流 |
| 2 | 增加导航权限配置 | 第 4 节状态管理 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确移动端交互 | 第 6 节用户交互流程 |
| 2 | 支持导航项徽章 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

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

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Badge | 徽章 |
| Drawer | 移动端抽屉 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| NavMenu | 导航菜单 |
| NotificationBell | 通知铃铛 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| TopNav | 顶部导航 | **新建** |
| UserMenu | 用户菜单 | **新建** |
| MobileNav | 移动端导航 | **新建** — 含交互定义 |

---

### 3. 数据流（V2 修正）【已修复】

```
[访问旧路由]
    │
    ▼
[路由守卫匹配] 【新增】
    │
    ▼
[302 重定向到新路由]
```

#### 旧路由映射表（V2 新增）【已修复】

```typescript
const routeRedirects = [
  { from: '/old-api-market', to: '/abilities', status: 302 },
  { from: '/old-invocations', to: '/jobs', status: 302 },
  // ... 其他映射
];
```

---

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface NavState {
  activeKey: string;
  navItems: NavItem[]; // 【已修复：过滤后的菜单项】
  mobileDrawerOpen: boolean;
}

// 【新增】导航权限配置
interface NavItem {
  key: string;
  label: string;
  path: string;
  requiredRole?: ('user' | 'developer' | 'admin')[];
  badge?: boolean; // 【新增】是否支持徽章
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

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[用户访问应用]
    │
    ▼
[展示导航]
    │
    ├── 点击菜单项 ──► 路由跳转
    ├── 点击通知 ──► 打开通知面板
    ├── 点击用户头像 ──► 打开用户菜单
    └── 移动端点击汉堡菜单 ──► 打开 Drawer 【已修复】
            │
            ▼
        [Drawer 打开/关闭动画 300ms]
        [支持从左侧边缘滑动手势打开]
        [点击遮罩层关闭]
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 访问无权限页面 | 跳转 403 |
| 访问旧路由 | 302 重定向 |

---

### 8. 性能优化

- 导航配置静态化
- 菜单项使用 React.memo

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 旧路由兼容 | 低 | 用户可能收藏旧链接 | 保留 30 天 302 重定向 |

---

### 10. 开发步骤拆分

#### Step 1: 导航组件（0.5 天）【已修复】
- [ ] TopNav + NavMenu（含权限过滤）

#### Step 2: 移动端适配（0.5 天）【已修复】
- [ ] MobileNav（含动画和手势）

#### Step 3: 路由重定向（0.5 天）【已修复】
- [ ] 旧路由映射表 + 路由守卫

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **旧路由映射** | 未明确 | 新增映射表 |
| **导航权限** | 未说明 | 新增角色权限配置 |
| **移动端交互** | 未说明 | 明确动画/手势/关闭方式 |
| **导航徽章** | 未提及 | 新增徽章支持 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（旧路由映射、导航权限）
2. 建议修改项已补充（移动端交互、徽章）
3. 风险等级：低
