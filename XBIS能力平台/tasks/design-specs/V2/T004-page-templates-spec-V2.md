# T004 页面模板 — 修正版设计方案（V2）

> 原设计方案: [T004-page-templates-spec.md](../T004-page-templates-spec.md)
> 评审报告: [T004-page-templates-Reviewer.md](../T004-page-templates-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 评估 TaskPageShell 和 AbilityPageShell 必要性 | 第 1 节页面结构 |
| 2 | 增加 AdminPageShell 模板 | 第 1 节页面结构 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 合并细粒度布局组件（PageHeader + BreadcrumbNav） | 第 2 节组件拆分 |
| 2 | 拆分 Sidebar 为用户端/管理端两个组件 | 第 2 节组件拆分 |
| 3 | 补充所有模板的完整 Props 定义 | 第 1 节页面结构 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

#### 一级模板（5 个）【已修复】

```
1. ListPageShell      # 列表页模板
2. DetailPageShell    # 详情页模板
3. FormPageShell      # 表单页模板
4. TaskPageShell      # 任务页模板（评估后保留，作为 ListPageShell 的特化版本）
5. AbilityPageShell   # 能力页模板（评估后保留，作为 ListPageShell 的特化版本）
6. AdminPageShell     # 【新增】管理端统一模板
```

#### AdminPageShell 模板（V2 新增）【已修复】

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

#### TaskPageShell 和 AbilityPageShell 保留说明【已修复】

- **与 ListPageShell 的差异**：
  - TaskPageShell：内置任务状态筛选器、批量操作栏
  - AbilityPageShell：内置能力分类筛选器、订阅状态筛选器
- **为什么保留**：这两个模板在项目中使用频率高，保留独立模板可减少页面级重复配置
- **如果不独立**：可通过 ListPageShell 的 `extraFilters` 和 `batchActions` 配置实现

---

### 2. 组件拆分

#### layout/ 层组件（V2 调整）【已修复】

| 组件 | 说明 | 调整 |
|------|------|------|
| AppShell | 应用级外壳 | - |
| PageShell | 页面级外壳 | - |
| AdminPageShell | 管理端统一模板 | 【新增】 |
| ListPageShell | 列表页模板 | - |
| DetailPageShell | 详情页模板 | 补充完整 Props 定义 |
| FormPageShell | 表单页模板 | 补充完整 Props 定义 |
| TaskPageShell | 任务页模板 | 保留，说明差异点 |
| AbilityPageShell | 能力页模板 | 保留，说明差异点 |
| Sidebar | 侧边栏 | 【已修复：拆分为 UserSidebar 和 AdminSidebar】 |
| UserSidebar | 用户端侧边栏 | 【新增】白色背景，顶部 Logo |
| AdminSidebar | 管理端侧边栏 | 【新增】深色背景，左侧导航 |
| TopNav | 顶部导航 | - |
| PageHeader | 页面头部 | 【已修复：合并 BreadcrumbNav 为子组件】 |
| BreadcrumbNav | 面包屑导航 | 【已修复：作为 PageHeader 的子组件或配置项】 |
| ContentArea | 内容区域 | - |
| Footer | 页脚 | - |

#### Props 定义补充【已修复】

```typescript
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

### 3. 数据流

无运行时数据流。模板为 Props 驱动。

---

### 4. 状态管理

无状态管理需求。

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 模板嵌套 | 禁止模板嵌套使用 |
| Props 缺失 | 使用默认值或 TypeScript 校验 |

---

### 8. 性能优化

- 模板组件使用 React.memo
- 布局组件懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 模板过于僵化 | 中 | 模板限制页面灵活性 | 提供 `extra` 插槽扩展 |

---

### 10. 开发步骤拆分

#### Step 1: 基础布局组件（1 人日）
- [ ] AppShell / PageShell
- [ ] AdminPageShell 【新增】

#### Step 2: 页面模板（1.5 人日）
- [ ] ListPageShell / DetailPageShell / FormPageShell
- [ ] TaskPageShell / AbilityPageShell（保留并说明差异）

#### Step 3: 导航组件（0.5 人日）【已修复】
- [ ] UserSidebar / AdminSidebar（拆分）
- [ ] TopNav / PageHeader（合并 BreadcrumbNav）

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **一级模板** | 5 个（无 AdminPageShell） | 6 个（新增 AdminPageShell） |
| **TaskPageShell/AbilityPageShell** | 未说明差异点 | 明确差异点并保留 |
| **Sidebar** | 1 个组件 | 拆分为 UserSidebar 和 AdminSidebar |
| **PageHeader/BreadcrumbNav** | 独立组件 | 合并为 PageHeader（面包屑作为子组件） |
| **Props 定义** | ListPageShell 有，其他未展示 | 补充所有模板完整 Props 定义 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（评估 TaskPageShell/AbilityPageShell 必要性、新增 AdminPageShell）
2. Sidebar 已拆分为用户端/管理端两个组件
3. 所有模板 Props 已补充
4. 风险等级：中（可控）
