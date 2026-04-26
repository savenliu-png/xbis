# T011 能力中心列表页 — 最终可开发设计方案

> 任务卡: [T011-ability-list-page.md](../T011-ability-list-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   ├── Title: "能力列表"
    │   └── Actions: [创建能力]
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框
        │   ├── 分类筛选
        │   └── 状态筛选
        ├── AbilityTable
        │   └── 能力列表
        └── Pagination
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 能力表格 |
| Tag | 状态标签 |
| Pagination | 分页 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| AbilityCard | 能力卡片 |
| StatusBadge | 状态标识 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| AbilityTable | 能力表格 | **新建** |
| FilterBar | 筛选栏 | **新建** |

---

## 3. 数据流

```
[进入 /abilities]
    │
    ▼
[API: GET /api/v1/abilities]
    │
    ▼
[渲染能力列表]
```

---

## 4. 状态管理

```typescript
interface AbilityListState {
  pageState: 'loading' | 'idle' | 'error';
  abilities: Ability[];
  total: number;
  filters: { keyword?: string; category?: string; status?: string; page: number; pageSize: number };
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力列表 | GET | `/api/v1/abilities` | 支持筛选分页 |

---

## 6. 用户交互流程

```
[进入能力列表]
    │
    ▼
[浏览列表]
    │
    ├── 搜索/筛选
    ├── 点击能力查看详情
    └── 点击创建能力
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无能力 | Empty 组件 |
| 筛选无结果 | Empty + 清除筛选 |
| 加载失败 | ErrorState + 重试 |

---

## 8. 性能优化

- 列表虚拟滚动（>100 条）
- 筛选 Debounce 300ms
- 分页加载

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 数据量大 | 低 | 能力数量多 | 分页加载 |
| 筛选条件复杂 | 低 | 多条件组合筛选 | 状态管理清晰 |

---

## 10. 开发步骤拆分

### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + ListPageShell

### Step 2: 列表组件（1 天）
- [ ] AbilityTable + FilterBar

### Step 3: API 与优化（0.5 天）
- [ ] API 集成 + 虚拟滚动
