# T028 管理端执行器管理 — 修正版设计方案（V2）

> 原设计方案: [T028-admin-executor-management-spec.md](../T028-admin-executor-management-spec.md)
> 评审报告: [T028-admin-executor-management-Reviewer.md](../T028-admin-executor-management-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确能力绑定/解绑操作 | 第 2 节组件拆分 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加强制停用选项 | 第 6 节用户交互流程 |
| 2 | 明确健康数据保留策略 | 第 8 节性能优化 |
| 3 | 增加配置版本快照 | 第 6 节用户交互流程 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "执行器管理"
    │   └── Breadcrumb: 概览 / 执行器管理
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框（名称/ID）
        │   ├── 类型筛选
        │   └── 状态筛选
        ├── ExecutorTable
        │   └── 执行器列表（含展开详情）
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 搜索框 |
| Table | 执行器表格 |
| Tag | 类型标签 |
| Badge | 状态徽章 |
| Switch | 启停切换 |
| Modal | 确认弹窗 |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Pagination | 分页 |

#### business/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| HealthIndicator | 健康状态 | 颜色+图标表示 healthy/degraded/unhealthy |
| ResourceChart | 资源图表 | 迷你 CPU/内存使用率条形图 |

#### blocks/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| ExecutorTable | 执行器表格 | 含展开详情 |
| HealthPanel | 健康详情 | 展开行/抽屉内展示 |
| ConfigPanel | 配置编辑 | 抽屉内编辑执行器配置 |
| BindingPanel | 能力绑定 | **已修复** — 展示/管理执行器绑定的能力 |

#### BindingPanel 能力绑定/解绑（V2 新增）【已修复】

```
BindingPanel 支持：
- 展示已绑定能力列表（表格形式）
- 解绑能力（点击解绑按钮 → 确认 Modal → API 解绑）
- 绑定新能力（下拉选择能力 → 设置优先级 → API 绑定）
```

---

### 3. 数据流

```
[进入 /admin/executors]
    │
    ▼
[初始化筛选条件]
    │
    ▼
[API: GET /admin-api/v1/executors]
    │
    ▼
[渲染执行器列表]
```

---

### 4. 状态管理

```typescript
interface AdminExecutorState {
  pageState: 'loading' | 'idle' | 'updating' | 'error' | 'configuring';
  executors: AdminExecutorItem[];
  total: number;
  filters: { keyword?: string; type?: string; status?: string; page: number; pageSize: number };
  expandedRowId: string | null;
  selectedExecutor: AdminExecutorItem | null;
  configDrawerOpen: boolean;
  configData: ExecutorConfig | null;
  error: ApiError | null;
}
```

---

### 5. API 调用（V2 修正）【已修复】

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 执行器列表 | GET | `/admin-api/v1/executors` | 查询执行器列表 |
| 执行器详情 | GET | `/admin-api/v1/executors/:id` | 查询执行器详情 |
| 配置更新 | PUT | `/admin-api/v1/executors/:id/config` | 更新执行器配置 |
| 启停 | PUT | `/admin-api/v1/executors/:id/status` | 启停执行器 |
| 绑定能力 | POST | `/admin-api/v1/executors/:id/bindings` | 【新增】绑定能力 |
| 解绑能力 | DELETE | `/admin-api/v1/executors/:id/bindings/:abilityId` | 【新增】解绑能力 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入执行器管理]
    │
    ▼
[浏览列表]
    │
    ├── 点击展开 ──► 查看健康历史/绑定能力
    │
    ├── 点击 Switch ──► 启停执行器
    │       │
    │       ▼
    │   [Modal 确认（含强制停用选项）] 【已修复】
    │       │
    │       ├── 「等待任务完成后停用」（默认）
    │       └── 「强制停用（终止进行中的任务）」
    │
    └── 点击「配置」──► Drawer 打开 ──► 编辑配置 ──► 保存
            │
            ▼
        [配置版本快照] 【新增】
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无执行器 | Empty + 引导注册 |
| 启停失败 | 恢复 Switch 状态 + Toast 错误 |
| 绑定失败 | 提示原因 |

---

### 8. 性能优化（V2 修正）【已修复】

```
健康数据保留策略：
- 保留最近 7 天的健康数据
- 每页显示 20 条，支持分页
- 支持按日期筛选
```

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 误操作停用 | 高 | 停用正在运行任务的执行器 | 二次确认 + 强制停用选项 |
| 配置错误 | 中 | 错误配置导致执行器异常 | 前端校验 + 版本快照 |

---

### 10. 开发步骤拆分

#### Step 1: 列表组件（1 天）
- [ ] ExecutorTable（含展开行）
- [ ] HealthIndicator + ResourceChart

#### Step 2: 详情与配置（1 天）【已修复】
- [ ] HealthPanel + BindingPanel（含绑定/解绑）
- [ ] ConfigPanel（含版本快照）

#### Step 3: API 与状态（0.5 天）
- [ ] API 调用 + 状态管理

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **能力绑定** | 未明确操作 | 明确绑定/解绑流程和 API |
| **强制停用** | 未设计 | 新增强制停用选项 |
| **健康数据** | 未说明保留策略 | 明确保留 7 天 |
| **配置版本** | 未设计 | 新增版本快照 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（能力绑定/解绑操作）
2. 建议修改项已补充（强制停用、健康数据、配置版本）
3. 风险等级：低
