# T011 能力中心列表页 — 工程级设计方案

> 任务卡: [T011-ability-list-page.md](../T011-ability-list-page.md)  
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
AbilityPageShell (一级模板)
├── PageHeader
│   ├── 标题: "能力中心"
│   └── 副标题: "发现、测试、接入 AI 能力"
├── FilterSidebar (左侧边栏)
│   ├── 分类筛选
│   └── 标签筛选
├── ContentArea
│   ├── TagFilterBar (顶部标签栏)
│   ├── SearchBar (搜索栏)
│   ├── SortBar (排序栏)
│   ├── AbilityGrid (能力网格)
│   │   └── AbilityCard[]
│   └── Pagination (分页器)
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Input | 搜索框 | 否 |
| Card | 卡片容器 | 否 |
| Tag | 标签 | 否 |
| Badge | 徽章 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |
| Pagination | 分页器 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| AbilityCard | 能力卡片 | 否 |
| StatusBadge | 状态标识 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| AbilityGrid | 能力网格布局 | 是 |
| FilterSidebar | 筛选侧边栏 | 是 |
| TagFilterBar | 标签过滤栏 | 是 |
| SearchBar | 搜索栏 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| AbilityPageShell | 能力页一级模板 |

---

## 3. 数据流

```
[页面加载]
  ├── 调用 GET /api/v1/abilities ──► 获取能力列表
  ├── 调用 GET /api/v1/categories ──► 获取分类列表
  └── 调用 GET /api/v1/tags ──► 获取标签列表

[用户筛选]
  ├── 选择分类 ──► 更新 filter.category ──► 重新查询
  ├── 选择标签 ──► 更新 filter.tags ──► 重新查询
  ├── 输入关键词 ──► debounce 300ms ──► 重新查询
  └── 选择排序 ──► 更新 sortBy ──► 重新查询

[用户分页]
  └── 切换页码 ──► 更新 pagination.page ──► 重新查询
```

---

## 4. 状态管理

### 页面状态

```typescript
interface AbilityListState {
  // 数据状态
  abilities: AbilityCardItem[];
  categories: CategoryItem[];
  tags: TagItem[];
  total: number;

  // 筛选状态
  filter: {
    category?: string;
    tags: string[];
    keyword: string;
    status: 'published' | 'draft' | 'deprecated' | 'all';
    sortBy: 'popular' | 'latest' | 'name';
    sortOrder: 'asc' | 'desc';
  };

  // 分页状态
  pagination: {
    page: number;
    pageSize: number;
  };

  // UI 状态
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/abilities | 页面加载、筛选变化、分页变化 | AbilityListParams | 显示错误提示，提供重试按钮 |
| GET /api/v1/categories | 页面加载 | - | 静默失败，不阻塞页面 |
| GET /api/v1/tags | 页面加载 | - | 静默失败，不阻塞页面 |

---

## 6. 用户交互流程

### 主流程

```
用户进入能力中心
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 显示能力网格
  │   ├── 用户点击分类 ──► 筛选能力
  │   ├── 用户点击标签 ──► 筛选能力
  │   ├── 用户输入关键词 ──► 搜索能力
  │   └── 用户点击能力卡片 ──► 跳转能力详情页
  └── 加载失败 ──► 显示错误提示 + 重试按钮
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 搜索无结果 | 关键词无匹配 | 显示空态 | 提示更换关键词 |
| 筛选无结果 | 筛选条件无匹配 | 显示空态 | 提示调整筛选 |
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 大量分类 | 支持折叠和展开 |
| 大量标签 | 支持横向滚动 |
| 超长名称 | 截断显示 tooltip |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 搜索使用 debounce（300ms）
- 列表支持虚拟滚动（>100 条数据时）
- 分页加载
- 缓存分类和标签数据

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 大量数据渲染 | 中 | 能力数量可能很大 | 虚拟滚动 + 分页 |
| 筛选条件复杂 | 低 | 多条件组合筛选 | 状态管理清晰 |
| 移动端适配 | 中 | 侧边栏在移动端需调整 | 响应式布局 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 AbilityPageShell 搭建页面框架
- [ ] 配置 PageHeader

### Step 2: 筛选区开发（1 人日）
- [ ] FilterSidebar 组件
- [ ] TagFilterBar 组件
- [ ] SearchBar 组件
- [ ] SortBar 组件

### Step 3: 数据区开发（1 人日）
- [ ] AbilityGrid 组件
- [ ] 集成 AbilityCard 业务组件
- [ ] Pagination 组件

### Step 4: 状态管理 & API 集成（0.5 人日）
- [ ] 实现筛选状态管理
- [ ] 集成 API 调用
- [ ] 实现加载/空态/错误态

### Step 5: 性能优化 & 测试（0.5 人日）
- [ ] 实现 debounce
- [ ] 实现虚拟滚动（可选）
- [ ] 自测
