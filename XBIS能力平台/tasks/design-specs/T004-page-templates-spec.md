# T004 页面模板 — 工程级设计方案

> 任务卡: [T004-page-templates.md](../T004-page-templates.md)  
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

本任务为**基础能力层**建设，输出物为 5 个页面模板组件。

```
packages/components/layout/         # 布局组件包
├── index.ts                        # 统一导出入口
├── ListPageShell/                  # 列表页模板
│   └── index.tsx
├── DetailPageShell/                # 详情页模板
│   └── index.tsx
├── FormPageShell/                  # 表单页模板
│   └── index.tsx
├── TaskPageShell/                  # 任务页模板（一级）
│   └── index.tsx
├── AbilityPageShell/               # 能力页模板（一级）
│   └── index.tsx
├── AppShell/                       # 应用外壳
│   └── index.tsx
├── Sidebar/                        # 侧边栏
│   └── index.tsx
├── TopNav/                         # 顶部导航
│   └── index.tsx
├── PageHeader/                     # 页面标题区
│   └── index.tsx
├── ContentArea/                    # 内容区容器
│   └── index.tsx
├── BreadcrumbNav/                  # 面包屑导航
│   └── index.tsx
├── Footer/                         # 页脚
│   └── index.tsx
├── MobileDrawer/                   # 移动端抽屉菜单
│   └── index.tsx
├── UserDropdown/                   # 用户下拉菜单
│   └── index.tsx
└── NotificationBell/               # 通知铃铛
    └── index.tsx
```

---

## 2. 组件拆分

### layout/ 层（14 个组件）

| 组件 | 说明 | 用户端 | 管理端 |
|------|------|--------|--------|
| AppShell | 应用外壳（Header + Sider + Content） | 使用 | 使用 |
| Sidebar | 侧边栏导航 | 白色背景 | 深色背景 |
| TopNav | 顶部导航栏 | 使用 | 使用 |
| PageHeader | 页面标题区（含面包屑/操作） | 使用 | 使用 |
| ContentArea | 内容区容器 | 使用 | 使用 |
| BreadcrumbNav | 面包屑导航 | 使用 | 使用 |
| Footer | 页脚 | 可选 | 可选 |
| MobileDrawer | 移动端抽屉菜单 | 使用 | 使用 |
| UserDropdown | 用户下拉菜单 | 使用 | 使用 |
| NotificationBell | 通知铃铛 | 使用 | 使用 |
| ListPageShell | 列表页模板 | 使用 | 使用 |
| DetailPageShell | 详情页模板 | 使用 | 使用 |
| FormPageShell | 表单页模板 | 使用 | 使用 |
| TaskPageShell | 任务页模板（一级） | 使用 | 使用 |
| AbilityPageShell | 能力页模板（一级） | 使用 | 使用 |

### 页面模板结构

#### ListPageShell

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
│  ├── 标题 + 副标题                                          │
│  ├── 面包屑导航                                             │
│  └── 操作按钮（新建/导出/刷新）                              │
├─────────────────────────────────────────────────────────────┤
│  Filter Bar（筛选栏）                                       │
│  ├── 搜索框                                                 │
│  ├── 状态筛选标签                                           │
│  └── 高级筛选器（可选）                                      │
├─────────────────────────────────────────────────────────────┤
│  Batch Action Bar（批量操作栏，选中时显示）                   │
├─────────────────────────────────────────────────────────────┤
│  Data Area（数据区）                                        │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: empty → Empty                                    │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → Table / Card List                         │
├─────────────────────────────────────────────────────────────┤
│  Pagination（分页器）                                       │
└─────────────────────────────────────────────────────────────┘
```

#### DetailPageShell

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
├─────────────────────────────────────────────────────────────┤
│  Tabs（可选，多标签详情）                                    │
├─────────────────────────────────────────────────────────────┤
│  Main Content Area                                          │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: notFound → Empty "内容不存在"                     │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → 多栏布局                                   │
│       ├── 主信息区（70%）                                    │
│       └── 侧边信息区（30%）                                  │
└─────────────────────────────────────────────────────────────┘
```

#### FormPageShell

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
├─────────────────────────────────────────────────────────────┤
│  Form Content Area                                          │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → 表单布局                                   │
│       ├── 表单主区（70%）                                    │
│       └── 辅助信息区（30%，可选）                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 数据流

页面模板为布局容器，数据通过 Props 传入。

```
页面组件 ──► Props ──► Page Template ──► 渲染布局 + 子组件
```

### ListPageShell 数据流

```typescript
interface ListPageShellProps {
  title: string;
  subtitle?: string;
  breadcrumbs?: BreadcrumbItem[];
  primaryAction?: ActionButton;
  filterBar?: React.ReactNode;
  batchActionBar?: React.ReactNode;
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
  onRetry?: () => void;
  pagination?: React.ReactNode;
  children: React.ReactNode;
}
```

---

## 4. 状态管理

- 页面模板自身无状态管理
- 状态由父组件（页面层）管理
- 模板通过 Props 接收状态并渲染对应 UI

---

## 5. API 调用

无 API 调用。数据通过 Props 传入。

---

## 6. 用户交互流程

| 组件 | 交互 | 行为 | 结果 |
|------|------|------|------|
| ListPageShell | 点击新建按钮 | onPrimaryAction 回调 | 打开新建弹窗/页面 |
| ListPageShell | 点击重试 | onRetry 回调 | 重新加载数据 |
| DetailPageShell | 点击返回 | 返回上一页 | 返回列表页 |
| FormPageShell | 点击保存 | onSubmit 回调 | 提交表单 |
| FormPageShell | 点击取消 | onCancel 回调 | 返回上一页 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 缺少必填 Props | TypeScript 编译时检查 |
| 子组件为空 | 渲染 Empty 组件 |
| 状态不匹配 | 使用 `never` 类型确保 exhaustive check |

---

## 8. 性能优化

- 布局组件使用 `React.memo` 避免不必要的重渲染
- 内容区使用 `children` 渲染，避免不必要的包裹
- 支持懒加载（未来）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 模板过于僵化 | 中 | 模板可能无法满足所有页面需求 | 提供灵活的 Props 配置 |
| 与现有 Layout 冲突 | 高 | 现有用户端/管理端已有 Layout | 渐进替换，保留旧 Layout |
| 响应式适配复杂 | 中 | 多端适配增加复杂度 | 使用响应式断点 + 媒体查询 |

---

## 10. 开发步骤拆分

### Step 1: 布局基础组件（AppShell, Sidebar, TopNav, ContentArea）（2 人日）
- [ ] AppShell 组件
- [ ] Sidebar 组件（支持用户端/管理端配置）
- [ ] TopNav 组件
- [ ] ContentArea 组件

### Step 2: 页面辅助组件（PageHeader, BreadcrumbNav, Footer, MobileDrawer）（1 人日）
- [ ] PageHeader 组件
- [ ] BreadcrumbNav 组件
- [ ] Footer 组件
- [ ] MobileDrawer 组件

### Step 3: 用户交互组件（UserDropdown, NotificationBell）（0.5 人日）
- [ ] UserDropdown 组件
- [ ] NotificationBell 组件

### Step 4: 页面模板（ListPageShell, DetailPageShell, FormPageShell）（1.5 人日）
- [ ] ListPageShell 组件
- [ ] DetailPageShell 组件
- [ ] FormPageShell 组件

### Step 5: 一级模板（TaskPageShell, AbilityPageShell）（1 人日）
- [ ] TaskPageShell 组件
- [ ] AbilityPageShell 组件

### Step 6: 统一导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写模板使用文档
