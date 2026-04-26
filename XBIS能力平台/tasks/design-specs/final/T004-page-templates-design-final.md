# T004 页面模板 — 最终可开发设计方案

> 任务卡: [T004-page-templates.md](../T004-page-templates.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为**基础能力层**建设，输出物为 6 个页面模板组件。

```
packages/components/layout/         # 布局组件包
├── index.ts                        # 统一导出入口
├── AppShell/                       # 应用外壳
│   └── index.tsx
├── PageShell/                      # 页面级外壳
│   └── index.tsx
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
├── AdminPageShell/                 # 管理端统一模板
│   └── index.tsx
├── UserSidebar/                    # 用户端侧边栏
│   └── index.tsx
├── AdminSidebar/                   # 管理端侧边栏
│   └── index.tsx
├── TopNav/                         # 顶部导航
│   └── index.tsx
├── PageHeader/                     # 页面标题区
│   └── index.tsx
├── ContentArea/                    # 内容区容器
│   └── index.tsx
├── Footer/                         # 页脚
│   └── index.tsx
└── MobileDrawer/                   # 移动端抽屉菜单
    └── index.tsx
```

### 一级模板（6 个）

1. ListPageShell      # 列表页模板
2. DetailPageShell    # 详情页模板
3. FormPageShell      # 表单页模板
4. TaskPageShell      # 任务页模板（ListPageShell 的特化版本）
5. AbilityPageShell   # 能力页模板（ListPageShell 的特化版本）
6. AdminPageShell     # 管理端统一模板

### AdminPageShell 模板

```typescript
interface AdminPageShellProps {
  title: string;
  breadcrumbs?: BreadcrumbItem[];
  actions?: React.ReactNode;
  children: React.ReactNode;
}

// 使用示例
<AdminPageShell
  title="能力管理"
  breadcrumbs={[{ label: '概览', path: '/admin' }, { label: '能力管理' }]}
  actions={<Button>新建能力</Button>}
>
  <AbilityTable />
</AdminPageShell>
```

### TaskPageShell 和 AbilityPageShell 差异说明

- **TaskPageShell**：内置任务状态筛选器、批量操作栏
- **AbilityPageShell**：内置能力分类筛选器、订阅状态筛选器
- **为什么保留**：这两个模板在项目中使用频率高，保留独立模板可减少页面级重复配置

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

## 2. 组件拆分

### layout/ 层组件

| 组件 | 说明 | 调整 |
|------|------|------|
| AppShell | 应用级外壳 | - |
| PageShell | 页面级外壳 | - |
| AdminPageShell | 管理端统一模板 | 新增 |
| ListPageShell | 列表页模板 | - |
| DetailPageShell | 详情页模板 | 补充完整 Props 定义 |
| FormPageShell | 表单页模板 | 补充完整 Props 定义 |
| TaskPageShell | 任务页模板 | 保留，说明差异点 |
| AbilityPageShell | 能力页模板 | 保留，说明差异点 |
| UserSidebar | 用户端侧边栏 | 白色背景，顶部 Logo |
| AdminSidebar | 管理端侧边栏 | 深色背景，左侧导航 |
| TopNav | 顶部导航 | - |
| PageHeader | 页面头部 | 合并 BreadcrumbNav 为子组件 |
| BreadcrumbNav | 面包屑导航 | 作为 PageHeader 的子组件或配置项 |
| ContentArea | 内容区域 | - |
| Footer | 页脚 | - |

### Props 定义补充

```typescript
// ListPageShell Props
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

// DetailPageShell Props
interface DetailPageShellProps {
  title: string;
  breadcrumbs?: BreadcrumbItem[];
  actions?: React.ReactNode;
  children: React.ReactNode;
  loading?: boolean;
  error?: Error | null;
}

// FormPageShell Props
interface FormPageShellProps {
  title: string;
  breadcrumbs?: BreadcrumbItem[];
  actions?: React.ReactNode;
  children: React.ReactNode;
  onSubmit?: () => void;
  onCancel?: () => void;
}

// TaskPageShell Props
interface TaskPageShellProps {
  title: string;
  breadcrumbs?: BreadcrumbItem[];
  filters?: React.ReactNode;
  batchActions?: React.ReactNode;
  children: React.ReactNode;
}

// AbilityPageShell Props
interface AbilityPageShellProps {
  title: string;
  breadcrumbs?: BreadcrumbItem[];
  categoryFilter?: React.ReactNode;
  children: React.ReactNode;
}
```

---

## 3. 数据流

页面模板为布局容器，数据通过 Props 传入。

```
页面组件 ──► Props ──► Page Template ──► 渲染布局 + 子组件
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
| 模板嵌套 | 禁止模板嵌套使用 |
| Props 缺失 | 使用默认值或 TypeScript 校验 |

---

## 8. 性能优化

- 布局组件使用 `React.memo` 避免不必要的重渲染
- 内容区使用 `children` 渲染，避免不必要的包裹
- 布局组件懒加载

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 模板过于僵化 | 中 | 模板可能无法满足所有页面需求 | 提供灵活的 Props 配置和 `extra` 插槽扩展 |
| 与现有 Layout 冲突 | 高 | 现有用户端/管理端已有 Layout | 渐进替换，保留旧 Layout |
| 响应式适配复杂 | 中 | 多端适配增加复杂度 | 使用响应式断点 + 媒体查询 |

---

## 10. 开发步骤拆分

### Step 1: 基础布局组件（1 人日）
- [ ] AppShell / PageShell
- [ ] AdminPageShell（新增）

### Step 2: 页面模板（1.5 人日）
- [ ] ListPageShell / DetailPageShell / FormPageShell
- [ ] TaskPageShell / AbilityPageShell（保留并说明差异）

### Step 3: 导航组件（0.5 人日）
- [ ] UserSidebar / AdminSidebar（拆分）
- [ ] TopNav / PageHeader（合并 BreadcrumbNav）

### Step 4: 统一导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写模板使用文档
