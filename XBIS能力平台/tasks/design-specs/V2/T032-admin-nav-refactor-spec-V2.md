# T032 管理端导航重构 — 修正版设计方案（V2）

> 原设计方案: [T032-admin-nav-refactor-spec.md](../T032-admin-nav-refactor-spec.md)
> 评审报告: [T032-admin-nav-refactor-Reviewer.md](../T032-admin-nav-refactor-Reviewer.md)
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
| 2 | 细化导航权限配置 | 第 4 节状态管理 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加导航徽章支持 | 第 2 节组件拆分 |
| 2 | 明确 V1 搜索方案 | 第 1 节页面结构 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
├── AdminSidebar (左侧导航)
│   ├── Logo 区
│   ├── NavMenu (9 项主导航) 【已修复：含权限配置】
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
│   ├── 全局搜索（V2 占位）【已修复】
│   └── 用户信息
└── MainContent (主内容区)
    └── 子路由页面
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 折叠按钮 |
| Icon | 导航图标 |
| Badge | 消息/告警徽章 |

#### business/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| AdminNavMenu | 管理导航菜单 | 9 项菜单，支持权限控制 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| AdminSidebar | 管理侧边栏 | 包含 Logo + NavMenu + 折叠 |

---

### 3. 数据流（V2 修正）【已修复】

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
    │
    ▼
[访问旧路由 /admin/reviews]
    │
    ▼
[路由守卫匹配] 【新增】
    │
    ▼
[302 重定向到 /admin/operations]
```

#### 旧路由映射表（V2 新增）【已修复】

```typescript
const adminRouteRedirects = [
  { from: '/admin/reviews', to: '/admin/operations', status: 302 },
  { from: '/admin/settings/old', to: '/admin/settings', status: 302 },
  // ... 其他映射
];
```

---

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface AdminNavState {
  collapsed: boolean;
  activeKey: string;
  navItems: AdminNavItem[]; // 【已修复：过滤后的菜单项】
}

// 【新增】导航权限配置
interface AdminNavItem {
  key: string;
  label: string;
  icon: string;
  path: string;
  requiredRole: ('admin' | 'superadmin' | 'operator')[];
  badge?: 'alerts' | 'pendingTasks'; // 【新增】徽章数据源
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

### 5. API 调用

无新增 API。导航配置为前端静态配置，权限由后端接口返回的用户角色决定。

---

### 6. 用户交互流程（V2 修正）【已修复】

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

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 访问无权限页面 | 跳转 403 |
| 访问不存在的路由 | 跳转 404 |
| 旧路由访问 | 302 重定向（保留 30 天） |

---

### 8. 性能优化

- 导航配置静态化，无需 API 请求
- 菜单项使用 React.memo
- 路由懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 旧路由兼容 | 中 | 管理员可能收藏旧链接 | 保留 30 天 302 重定向 |
| 用户体验变化 | 中 | 导航结构变化 | 灰度发布 + 引导提示 |
| 权限控制 | 低 | 不同角色看到不同菜单 | 后端校验 + 前端过滤 |

---

### 10. 开发步骤拆分

#### Step 1: 导航组件（0.5 天）【已修复】
- [ ] AdminNavMenu 组件（含权限过滤）
- [ ] AdminSidebar 组件
- [ ] 9 项菜单配置

#### Step 2: 布局集成（0.5 天）
- [ ] AdminLayout 重构
- [ ] 侧边栏折叠功能
- [ ] 当前项高亮

#### Step 3: 路由重定向（0.5 天）【已修复】
- [ ] 旧路由重定向配置
- [ ] 路由守卫权限校验

#### Step 4: 优化（0.5 天）
- [ ] 响应式布局
- [ ] 暗色模式适配
- [ ] 性能优化

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **旧路由映射** | 未明确 | 新增映射表 |
| **导航权限** | 未细化 | 明确角色权限配置 |
| **导航徽章** | 未提及 | 新增徽章支持 |
| **V1 搜索** | 未说明 | 明确隐藏占位符 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（旧路由映射、导航权限）
2. 建议修改项已补充（徽章、V1 搜索）
3. 风险等级：低
