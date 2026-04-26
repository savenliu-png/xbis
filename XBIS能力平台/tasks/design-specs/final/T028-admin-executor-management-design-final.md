# T028 管理端执行器管理 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "执行器管理"
    │   └── Breadcrumb: 概览 / 执行器管理
    └── ListPageShell
        ├── FilterBar
        │   ├── 搜索框（名称/ID）
        │   ├── 类型筛选（docker/kubernetes/vm/serverless）
        │   └── 状态筛选（active/inactive/degraded/offline）
        ├── DataTable
        │   ├── 执行器列表（可展开查看详情）
        │   └── 操作列（配置/启停/查看）
        └── Pagination
```

### 1.2 模板使用
| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 外层 | AdminPageShell | 管理端统一骨架 |
| 列表区 | ListPageShell | 列表页模板 |
| 详情区 | DetailPageShell（内嵌） | 展开行或抽屉展示详情 |

### 1.3 布局说明
- **Desktop (≥1200px)**: 侧边栏 256px + 主内容区
- **管理端无需移动端设计**
- 表格列：名称 | 类型 | 状态 | 健康 | CPU | 内存 | 活跃任务 | 版本 | 操作

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）
| 组件 | 用途 | 位置 |
|------|------|------|
| Button | 操作按钮 | 表格操作列 |
| Input | 搜索框 | 筛选栏 |
| Table | 执行器表格 | 主内容区 |
| Tag | 类型标签 | 表格类型列 |
| Badge | 状态徽章 | 状态/健康列 |
| Switch | 启停切换 | 操作列 |
| Modal | 确认弹窗 | 启停/配置确认 |
| Skeleton | 加载骨架 | 初始加载 |
| Empty | 空态 | 无执行器时 |
| Pagination | 分页 | 表格底部 |
| Select | 下拉选择 | 绑定能力选择 |

### 2.2 business/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| HealthIndicator | 健康状态指示器 | 颜色+图标表示 healthy/degraded/unhealthy |
| ResourceChart | 资源使用图表 | 迷你 CPU/内存使用率条形图 |

### 2.3 blocks/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| ExecutorTable | 执行器表格 | 基于 Table，包含展开详情 |
| HealthPanel | 健康详情面板 | 展开行/抽屉内展示健康历史 |
| ConfigPanel | 配置编辑面板 | 抽屉内编辑执行器配置（含版本快照） |
| BindingPanel | 能力绑定面板 | 展示/管理执行器绑定的能力 |

### 2.4 BindingPanel 能力绑定/解绑
```
BindingPanel 支持：
- 展示已绑定能力列表（表格形式）
- 解绑能力（点击解绑按钮 → 确认 Modal → API 解绑）
- 绑定新能力（下拉选择能力 → 设置优先级 → API 绑定）
```

---

## 3. 数据流

### 3.1 页面加载数据流
```
[管理员进入 /admin/executors]
    │
    ▼
[初始化筛选条件（默认值）]
    │
    ▼
[API: GET /admin-api/v1/executors]
    │
    ▼
[渲染执行器列表]
    │
    ▼
[idle 状态]
```

### 3.2 启停操作数据流
```
[管理员点击 Switch 启停]
    │
    ▼
[Modal 确认弹窗（含强制停用选项）]
    │
    ├── 「等待任务完成后停用」（默认）
    └── 「强制停用（终止进行中的任务）」
    │
    ▼
[API: PUT /admin-api/v1/executors/:id/status]
    │
    ├── 成功 → 更新列表行状态 → Toast 成功
    │
    └── 失败 → 恢复 Switch 状态 → Toast 错误
```

### 3.3 配置更新数据流
```
[管理员点击「配置」]
    │
    ▼
[Drawer 打开，加载当前配置]
    │
    ▼
[管理员修改配置]
    │
    ▼
[点击保存]
    │
    ▼
[保存配置版本快照]
    │
    ▼
[API: PUT /admin-api/v1/executors/:id/config]
    │
    ├── 成功 → 关闭 Drawer → 更新列表 → Toast 成功
    │
    └── 失败 → 显示错误 → 保留 Drawer
```

### 3.4 能力绑定/解绑数据流
```
[展开 BindingPanel]
    │
    ▼
[展示已绑定能力列表]
    │
    ├── 点击「解绑」→ 确认 Modal → DELETE /bindings/:abilityId → 刷新列表
    │
    └── 点击「绑定新能力」→ 选择能力 + 设置优先级 → POST /bindings → 刷新列表
```

### 3.5 数据流图
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Router    │────▶│  ListParams │────▶│  API GET    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                                │
                       ┌────────────────────────┘
                       ▼
              ┌─────────────────┐
              │  ExecutorList    │
              │  (Admin 视角)    │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │  查看详情 │   │  启停   │   │  配置   │
   │  GET    │   │  PUT    │   │  PUT    │
   └─────────┘   └─────────┘   └─────────┘
```

---

## 4. 状态管理

### 4.1 页面级状态机
```typescript
type PageState =
  | 'loading'      // 初始加载
  | 'idle'         // 正常浏览
  | 'updating'     // 更新状态中
  | 'error'        // 加载错误
  | 'configuring'  // 配置编辑中
```

### 4.2 状态定义
```typescript
interface AdminExecutorState {
  // 页面状态
  pageState: PageState;

  // 列表数据
  executors: AdminExecutorItem[];
  total: number;

  // 筛选条件
  filters: {
    keyword?: string;
    type?: string;
    status?: string;
    page: number;
    pageSize: number;
  };

  // 选中/展开
  expandedRowId: string | null;
  selectedExecutor: AdminExecutorItem | null;

  // 配置编辑
  configDrawerOpen: boolean;
  configData: ExecutorConfig | null;

  // 错误信息
  error: ApiError | null;
}
```

### 4.3 各状态 UI 表现
| 状态 | UI 表现 |
|------|---------|
| loading | Table Skeleton |
| idle | 正常表格展示 |
| updating | 操作按钮 loading，行状态更新中 |
| error | ErrorState 组件 |
| configuring | Drawer 打开，配置表单可编辑 |

---

## 5. API 调用

### 5.1 API 列表
| API | Method | Path | 触发时机 | 错误处理 |
|-----|--------|------|----------|----------|
| 执行器列表 | GET | `/admin-api/v1/executors` | 页面加载/筛选变化 | 显示错误态 |
| 执行器详情 | GET | `/admin-api/v1/executors/:id` | 展开行/查看详情 | 显示错误提示 |
| 配置更新 | PUT | `/admin-api/v1/executors/:id/config` | 保存配置 | Drawer 内显示错误 |
| 启停 | PUT | `/admin-api/v1/executors/:id/status` | 点击 Switch | 恢复状态，Toast 错误 |
| 绑定能力 | POST | `/admin-api/v1/executors/:id/bindings` | 绑定新能力 | Toast 错误 |
| 解绑能力 | DELETE | `/admin-api/v1/executors/:id/bindings/:abilityId` | 解绑能力 | Toast 错误 |

### 5.2 请求参数
```typescript
// GET /admin-api/v1/executors
interface ExecutorListParams {
  keyword?: string;
  type?: 'docker' | 'kubernetes' | 'vm' | 'serverless';
  status?: 'active' | 'inactive' | 'degraded' | 'offline';
  page?: number;
  pageSize?: number;
}

// PUT /admin-api/v1/executors/:id/config
interface ExecutorConfig {
  maxConcurrency: number;
  timeout: number;
  retryPolicy: {
    maxRetries: number;
    retryInterval: number;
  };
  resources: {
    cpu: string;
    memory: string;
  };
}

// PUT /admin-api/v1/executors/:id/status
interface ExecutorStatusRequest {
  status: 'active' | 'inactive';
  force?: boolean; // 强制停用
}

// POST /admin-api/v1/executors/:id/bindings
interface BindingRequest {
  abilityId: string;
  priority: number;
}
```

### 5.3 错误处理策略
| 错误码 | 场景 | 处理方式 |
|--------|------|----------|
| 400 | 配置参数错误 | Drawer 内显示校验错误 |
| 403 | 权限不足 | 跳转 403 |
| 409 | 执行器正在运行任务 | Modal 提示「有进行中的任务，是否强制停用？」 |
| 500 | 服务器错误 | Toast 提示，可重试 |

---

## 6. 用户交互流程

### 6.1 主流程：查看并管理执行器
```
[进入执行器管理页]
    │
    ▼
[加载执行器列表]
    │
    ▼
[浏览列表] ──► [筛选/搜索] ──► [重新加载列表]
    │
    ├── 点击「展开」──► 查看健康历史/绑定能力
    │
    ├── 点击 Switch ──► Modal 确认（含强制停用选项）──► 启停执行器
    │
    └── 点击「配置」──► Drawer 打开 ──► 编辑配置 ──► 保存（含版本快照）
```

### 6.2 停用确认流程
```
[点击停用 Switch]
    │
    ▼
[Modal 确认弹窗]
    │
    ├── 「等待任务完成后停用」（默认选项）
    └── 「强制停用（终止进行中的任务）」
    │
    ▼
[显示进行中任务数]
    │
    ▼
[确认后调用 API]
```

### 6.3 异常流程
| 场景 | 用户行为 | 系统响应 |
|------|----------|----------|
| 停用有任务的执行器 | 点击 Switch | Modal 提示「有 N 个进行中的任务」，提供强制停用选项 |
| 配置值非法 | 输入负数 | 表单校验阻止提交 |
| 网络中断 | 保存配置 | 检测网络恢复后提示重试 |
| 绑定失败 | 点击绑定 | Toast 提示失败原因 |

---

## 7. 边界与异常

### 7.1 空态处理
| 场景 | 处理方式 |
|------|----------|
| 无执行器 | Empty 组件 + 「注册执行器」引导 |
| 筛选无结果 | Empty 组件 + 「清除筛选」按钮 |

### 7.2 错误处理
| 场景 | 处理方式 |
|------|----------|
| 加载失败 | ErrorState + 重试 |
| 启停失败 | 恢复 Switch 状态 + Toast 错误 |
| 配置保存失败 | Drawer 保持打开 + 显示错误 |
| 绑定/解绑失败 | Toast 错误提示 |

### 7.3 权限控制
| 角色 | 权限 |
|------|------|
| 管理员 | 可启停/配置/绑定所有执行器 |
| 运维人员 | 可查看/配置/绑定，不可启停 |

---

## 8. 性能优化

### 8.1 加载优化
- 列表分页加载，默认 20 条/页
- 展开行详情懒加载（点击展开时才请求详情）

### 8.2 运行时优化
- 健康数据定时刷新（30s），使用轮询
- 筛选条件 Debounce 300ms

### 8.3 渲染优化
- Table 行使用 React.memo
- 健康指示器使用 CSS 动画而非 JS

### 8.4 健康数据保留策略
- 保留最近 7 天的健康数据
- 每页显示 20 条，支持分页
- 支持按日期筛选

---

## 9. 风险点

### 9.1 高风险
| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 误操作停用执行器 | 停用正在运行任务的执行器会导致任务失败 | 二次确认 Modal + 强制停用选项 + 显示进行中的任务数 + 后端校验 |

### 9.2 中风险
| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 健康数据量大 | 历史健康数据可能很大 | 保留最近 7 天，每页 20 条，支持分页 |
| 配置错误 | 错误的配置可能导致执行器异常 | 前端校验 + 后端校验 + 配置版本快照 |
| 绑定冲突 | 同一能力绑定多个执行器 | 后端校验优先级 |

---

## 10. 开发步骤

### Step 1: 基础结构（0.5 天）
- [ ] 创建页面路由 `/admin/executors`
- [ ] 搭建 AdminPageShell + ListPageShell
- [ ] 实现筛选栏

### Step 2: 表格组件（1 天）
- [ ] 实现 ExecutorTable（含展开行）
- [ ] 实现 HealthIndicator 组件
- [ ] 实现 ResourceChart 组件
- [ ] 集成启停 Switch

### Step 3: 详情与配置（1 天）
- [ ] 实现展开行详情（HealthPanel + BindingPanel）
- [ ] 实现 ConfigPanel（Drawer 形式，含版本快照）
- [ ] 实现配置保存逻辑
- [ ] 实现能力绑定/解绑功能

### Step 4: API 与状态（0.5 天）
- [ ] 实现 API 调用（含绑定/解绑 API）
- [ ] 实现状态管理
- [ ] 实现错误处理

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/admin/ExecutorManagement/
├── index.tsx
├── ExecutorManagementPage.tsx
├── components/
│   ├── ExecutorTable.tsx
│   ├── HealthPanel.tsx
│   ├── ConfigPanel.tsx
│   └── BindingPanel.tsx
└── hooks/
    └── useExecutorManagement.ts

packages/components/business/
├── HealthIndicator/
│   └── index.tsx
└── ResourceChart/
    └── index.tsx
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/admin.tsx
```
