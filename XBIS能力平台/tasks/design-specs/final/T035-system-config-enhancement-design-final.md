# T035 系统配置增强 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "系统配置"
    │   └── Breadcrumb: 概览 / 系统配置
    └── FormPageShell
        └── Tabs
            ├── Tab 1: 系统参数
            │   └── 配置项列表（键值对编辑）
            ├── Tab 2: 首页推荐
            │   └── 推荐项拖拽排序
            ├── Tab 3: 分类管理
            │   └── 分类列表 + 编辑
            └── Tab 4: 变更日志
                └── 操作记录表格
```

### 1.2 模板使用
| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 外层 | AdminPageShell | 管理端骨架 |
| 内容 | FormPageShell | 表单页模板 |

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）
| 组件 | 用途 |
|------|------|
| Button, Input, Table, Tag, Switch, Tabs, Form, Select, Skeleton, Empty, Pagination | 标准用法 |

### 2.2 blocks/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| ConfigList | 配置项列表 | 键值对编辑 |
| RecommendationManager | 推荐管理 | 拖拽排序（使用 @dnd-kit） |
| CategoryManager | 分类管理 | 分类 CRUD |
| ChangeLogTable | 变更日志 | 支持筛选和导出 |

### 2.3 拖拽库选择
推荐使用 **@dnd-kit**，理由：
- 支持无障碍访问
- 动画流畅
- 社区活跃

---

## 3. 数据流

### 3.1 配置加载
```
[进入 /admin/settings]
    │
    ▼
[并行加载]
    ├── API: GET /admin-api/v1/settings
    ├── API: GET /admin-api/v1/recommendations
    ├── API: GET /admin-api/v1/categories
    └── API: GET /admin-api/v1/changelogs
```

### 3.2 配置更新
```
[编辑配置项]
    │
    ▼
[API: PUT /admin-api/v1/settings/:id]
    │
    ├── 成功 → 更新列表 → 记录变更日志
    └── 失败 → 显示错误
```

---

## 4. 状态管理

```typescript
interface SystemConfigState {
  pageState: 'loading' | 'idle' | 'error';
  activeTab: string;
  configs: SystemConfig[];
  recommendations: HomeRecommendation[];
  categories: CategoryManagement[];
  changelogs: ChangeLog[];
  error: ApiError | null;
}

// 分类层级策略
const categoryStrategy = {
  v1: '仅支持一级分类（扁平结构）',
  v2: '支持多级层级（树形结构）',
};
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 系统配置查询 | GET | `/admin-api/v1/settings` | 查询所有配置 |
| 系统配置更新 | PUT | `/admin-api/v1/settings/:id` | 更新配置项 |
| 首页推荐查询 | GET | `/admin-api/v1/recommendations` | 查询推荐 |
| 首页推荐更新 | PUT | `/admin-api/v1/recommendations` | 更新推荐 |
| 分类管理查询 | GET | `/admin-api/v1/categories` | 查询分类 |
| 分类管理更新 | PUT | `/admin-api/v1/categories/:id` | 更新分类 |
| 变更日志查询 | GET | `/admin-api/v1/changelogs` | 查询日志 |

---

## 6. 用户交互流程

```
[进入系统配置]
    │
    ▼
[Tab 1: 系统参数]
    │
    ├── 编辑配置值
    ├── 保存（关键配置需审批）
    └── 查看变更日志
    │
    ▼
[Tab 2: 首页推荐]
    │
    ├── 拖拽排序（使用 @dnd-kit）
    ├── 启用/禁用
    └── 保存
    │
    ▼
[Tab 3: 分类管理]
    │
    ├── 编辑分类
    ├── 排序
    └── 保存
    │
    ▼
[Tab 4: 变更日志]
    │
    ├── 筛选（按操作人/类型/时间）
    └── 导出（CSV/Excel）
```

### 关键配置审批机制
```typescript
interface ConfigChangeRequest {
  configId: string;
  oldValue: unknown;
  newValue: unknown;
  reason: string; // 变更原因（必填）
  requireApproval: boolean; // 是否需要审批
}
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 配置冲突 | 后端校验，返回错误 |
| 并发修改 | 乐观锁，提示刷新 |
| 无权限 | 403 页面 |

---

## 8. 性能优化

- 配置数据缓存
- 变更日志分页
- Tab 懒加载

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 配置错误 | 错误配置影响系统运行 | 配置校验；预览确认；关键配置审批机制 |
| 缓存不一致 | 配置更新后缓存未刷新 | 热更新机制；缓存失效 |

---

## 10. 开发步骤

### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + Tabs 结构

### Step 2: 各 Tab 内容（1.5 天）
- [ ] 系统参数 Tab（含审批机制）
- [ ] 首页推荐 Tab（集成 @dnd-kit）
- [ ] 分类管理 Tab
- [ ] 变更日志 Tab（含筛选/导出）

### Step 3: API 与优化（0.5 天）
- [ ] API 集成
- [ ] 拖拽排序
- [ ] 错误处理

### Step 4: 联调（0.5 天）
- [ ] 端到端测试

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/admin/SystemSettings/
├── index.tsx
├── SystemSettingsPage.tsx
└── components/
    ├── ConfigList.tsx
    ├── RecommendationManager.tsx
    ├── CategoryManager.tsx
    └── ChangeLogTable.tsx
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/admin.tsx
```
