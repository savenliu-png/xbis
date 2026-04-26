# T012 能力详情页 — 修正版设计方案（V2）

> 原设计方案: [T012-ability-detail-page-spec.md](../T012-ability-detail-page-spec.md)
> 评审报告: [T012-ability-detail-page-Reviewer.md](../T012-ability-detail-page-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed

---

## 二、必须修改项（Blocking）

无必须修改项。

---

## 三、建议修改项（Optional）

无建议修改项。

---

## 四、修正版设计方案（V2）

### 1. 页面结构

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

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Card | 信息卡片 |
| Tabs | 标签切换 |
| Table | 统计表格 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| AbilityCard | 能力卡片 |
| StatusBadge | 状态标识 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| AbilityInfoCard | 能力信息卡片 | **新建** |
| AbilitySchemaPanel | Schema 展示面板 | **新建** |
| AbilityUsagePanel | 使用统计面板 | **新建** |

---

### 3. 数据流

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

### 4. 状态管理

```typescript
interface AbilityDetailState {
  pageState: 'loading' | 'idle' | 'error';
  ability: Ability | null;
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力详情 | GET | `/api/v1/abilities/:id` | 查询能力详情 |

---

### 6. 用户交互流程

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

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 能力不存在 | 404 页面 |
| 加载失败 | ErrorState + 重试 |

---

### 8. 性能优化

- 详情数据缓存
- Schema 展示懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Schema 复杂 | 低 | 复杂 Schema 渲染慢 | 虚拟滚动 |

---

### 10. 开发步骤拆分

#### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + DetailPageShell

#### Step 2: 详情组件（1 天）
- [ ] AbilityInfoCard + AbilitySchemaPanel + AbilityUsagePanel

#### Step 3: API 集成（0.5 天）
- [ ] API 调用 + 错误处理

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **设计内容** | 无修改 | 保持原设计 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 原设计已通过评审
2. 无阻塞项
3. 风险等级：低
