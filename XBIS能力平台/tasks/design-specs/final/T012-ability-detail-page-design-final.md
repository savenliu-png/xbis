# T012 能力详情页 — 最终可开发设计方案

> 任务卡: [T012-ability-detail-page.md](../T012-ability-detail-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   ├── Title: "能力详情"
    │   └── Actions: [编辑] [删除]
    └── DetailPageShell
        ├── AbilityInfoCard
        │   ├── 能力基本信息
        │   └── 操作按钮
        ├── AbilitySchemaPanel
        │   ├── 输入 Schema
        │   └── 输出 Schema
        └── AbilityUsagePanel
            └── 使用统计
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Card | 信息卡片 |
| Tabs | 标签切换 |
| Table | 统计表格 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| AbilityCard | 能力卡片 |
| StatusBadge | 状态标识 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| AbilityInfoCard | 能力信息卡片 | **新建** |
| AbilitySchemaPanel | Schema 展示面板 | **新建** |
| AbilityUsagePanel | 使用统计面板 | **新建** |

---

## 3. 数据流

```
[进入 /abilities/:id]
    │
    ▼
[API: GET /api/v1/abilities/:id]
    │
    ▼
[渲染能力详情]
```

---

## 4. 状态管理

```typescript
interface AbilityDetailState {
  pageState: 'loading' | 'idle' | 'error';
  ability: Ability | null;
  error: ApiError | null;
}
```

> **设计补充说明**：`versions` 作为 `AbilityDetailResponse` 的独立字段返回，与 `ability` 分离，避免详情对象过于臃肿。

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力详情 | GET | `/api/v1/abilities/:id` | 查询能力详情 |

---

## 6. 用户交互流程

```
[进入能力详情]
    │
    ▼
[浏览详情]
    │
    ├── 点击编辑
    ├── 点击删除
    └── 点击测试
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 能力不存在 | 404 页面 |
| 加载失败 | ErrorState + 重试 |
| Schema 解析失败 | 显示原始 JSON |

---

## 8. 性能优化

- 详情数据缓存
- Schema 展示懒加载
- 首屏加载 < 2s

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Schema 复杂 | 低 | 复杂 Schema 渲染慢 | 虚拟滚动 |
| Markdown 安全 | 低 | Markdown 可能包含恶意内容 | 使用安全渲染库 |

> **设计补充说明**：调用示例展示（`AbilityExample`）为增强功能，当前版本暂不实现，后续迭代添加。

---

## 10. 开发步骤拆分

### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + DetailPageShell

### Step 2: 详情组件（1 天）
- [ ] AbilityInfoCard + AbilitySchemaPanel + AbilityUsagePanel

### Step 3: API 集成（0.5 天）
- [ ] API 调用 + 错误处理
