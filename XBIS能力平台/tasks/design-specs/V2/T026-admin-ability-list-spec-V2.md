# T026 管理端能力管理列表 — 修正版设计方案（V2）

> 原设计方案: [T026-admin-ability-list-spec.md](../T026-admin-ability-list-spec.md)
> 评审报告: [T026-admin-ability-list-Reviewer.md](../T026-admin-ability-list-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 删除前展示关联影响 | 第 6 节用户交互流程 |
| 2 | 明确乐观更新回滚机制 | 第 3 节数据流 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加批量操作功能 | 第 2 节组件拆分 |
| 2 | 明确表格排序能力 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "能力管理"
    │   └── Actions: [新建能力]
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框
        │   ├── 分类筛选
        │   └── 状态筛选
        ├── AbilityTable
        │   └── 能力列表（含批量操作）【已修复】
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 能力表格 |
| Tag | 状态标签 |
| Switch | 状态切换 |
| Pagination | 分页 |
| Checkbox | 批量选择 【新增】 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| StatusBadge | 状态标识 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| AbilityTable | 能力表格 | **新建** — 含批量操作 |
| FilterBar | 筛选栏 | **新建** |
| BatchActionBar | 批量操作栏 | **新建** |

---

### 3. 数据流（V2 修正）【已修复】

```
[进入 /admin/abilities]
    │
    ▼
[API: GET /admin-api/v1/abilities]
    │
    ▼
[渲染能力列表]
    │
    ▼
[点击 Switch 切换状态]
    │
    ▼
[乐观更新：先更新本地状态] 【已修复】
    │
    ▼
[API: PUT /admin-api/v1/abilities/:id/status]
    │
    ├── 成功：保持更新
    └── 失败：恢复原始状态，显示错误提示 【已修复】
```

---

### 4. 状态管理

```typescript
interface AdminAbilityListState {
  pageState: 'loading' | 'idle' | 'error';
  abilities: Ability[];
  total: number;
  selectedIds: string[]; // 【新增】批量选中
  filters: { keyword?: string; category?: string; status?: string; page: number; pageSize: number };
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力列表 | GET | `/admin-api/v1/abilities` | 支持筛选分页 |
| 更新状态 | PUT | `/admin-api/v1/abilities/:id/status` | 更新能力状态 |
| 删除能力 | DELETE | `/admin-api/v1/abilities/:id` | 删除能力 |
| 批量删除 | POST | `/admin-api/v1/abilities/batch-delete` | 【新增】批量删除 |
| 批量更新状态 | POST | `/admin-api/v1/abilities/batch-status` | 【新增】批量更新状态 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入能力管理]
    │
    ▼
[浏览列表]
    │
    ├── 搜索/筛选
    ├── 点击 Switch 切换状态（乐观更新）
    ├── 勾选多行 ──► 批量操作栏显示 【新增】
    │       │
    │       ▼
    │   [批量启用/禁用/删除]
    │
    └── 点击删除
            │
            ▼
        [确认弹窗（展示关联影响）] 【已修复】
            │
            ▼
        [确认删除/取消]
```

#### 删除前关联影响展示（V2 新增）【已修复】

```
确认弹窗中显示：
- 当前订阅用户数
- 进行中任务数
- 历史任务总数
- 提示"删除后这些任务将无法执行"
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无能力 | Empty 组件 |
| 删除失败 | 提示原因（如"有进行中的任务"） |

---

### 8. 性能优化

- 列表虚拟滚动（>100 条）
- 筛选 Debounce 300ms

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 误删除 | 高 | 删除影响大 | 展示关联影响 + 二次确认 |

---

### 10. 开发步骤拆分

#### Step 1: 列表组件（1 天）
- [ ] AbilityTable + FilterBar

#### Step 2: 批量操作（0.5 天）【已修复】
- [ ] BatchActionBar（批量启用/禁用/删除）

#### Step 3: 删除确认（0.5 天）【已修复】
- [ ] 删除确认弹窗（含关联影响展示）

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **批量操作** | 未设计 | 新增批量启用/禁用/删除 |
| **删除确认** | 仅二次确认 | 新增关联影响展示 |
| **乐观更新** | 未明确回滚 | 明确失败恢复机制 |
| **表格排序** | 未说明 | 支持按名称/时间/状态排序 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（删除关联影响、乐观更新回滚）
2. 建议修改项已补充（批量操作、排序）
3. 风险等级：中（可控）
