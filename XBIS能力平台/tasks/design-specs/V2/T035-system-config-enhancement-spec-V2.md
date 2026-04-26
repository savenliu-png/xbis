# T035 系统配置增强 — 修正版设计方案（V2）

> 原设计方案: [T035-system-config-enhancement-spec.md](../T035-system-config-enhancement-spec.md)
> 评审报告: [T035-system-config-enhancement-Reviewer.md](../T035-system-config-enhancement-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确拖拽排序实现方案 | 第 2 节组件拆分 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加关键配置审批机制 | 第 6 节用户交互流程 |
| 2 | 明确分类层级策略 | 第 4 节状态管理 |
| 3 | 增加变更日志筛选和导出 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

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
            │   └── 推荐项拖拽排序 【已修复】
            ├── Tab 3: 分类管理
            │   └── 分类列表 + 编辑
            └── Tab 4: 变更日志
                └── 操作记录表格
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 输入框 |
| Table | 表格 |
| Tag | 标签 |
| Switch | 开关 |
| Tabs | Tab 切换 |
| Form | 表单容器 |
| Select | 下拉选择 |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Pagination | 分页 |

#### blocks/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| ConfigList | 配置项列表 | 键值对编辑 |
| RecommendationManager | 推荐管理 | **已修复** — 拖拽排序（使用 @dnd-kit） |
| CategoryManager | 分类管理 | 分类 CRUD |
| ChangeLogTable | 变更日志 | **已修复** — 支持筛选和导出 |

#### 拖拽库选择（V2 新增）【已修复】

推荐使用 **@dnd-kit**，理由：
- 支持无障碍访问
- 动画流畅
- 社区活跃

---

### 3. 数据流

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

---

### 4. 状态管理（V2 修正）【已修复】

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

// 【新增】分类层级策略
const categoryStrategy = {
  v1: '仅支持一级分类（扁平结构）',
  v2: '支持多级层级（树形结构）',
};
```

---

### 5. API 调用

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

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入系统配置]
    │
    ▼
[Tab 1: 系统参数]
    │
    ├── 编辑配置值
    ├── 保存
    └── 查看变更日志
    │
    ▼
[Tab 2: 首页推荐]
    │
    ├── 拖拽排序（使用 @dnd-kit）【已修复】
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
    ├── 筛选（按操作人/类型/时间）【新增】
    └── 导出（CSV/Excel）【新增】
```

#### 关键配置审批机制（V2 新增）【已修复】

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

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 配置冲突 | 后端校验，返回错误 |
| 并发修改 | 乐观锁，提示刷新 |
| 无权限 | 403 页面 |

---

### 8. 性能优化

- 配置数据缓存
- 变更日志分页
- Tab 懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 配置错误 | 中 | 错误配置影响系统运行 | 配置校验 + 预览确认 |
| 缓存不一致 | 低 | 配置更新后缓存未刷新 | 热更新机制 |

---

### 10. 开发步骤拆分

#### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + Tabs 结构

#### Step 2: 各 Tab 内容（1.5 天）【已修复】
- [ ] 系统参数 Tab
- [ ] 首页推荐 Tab（集成 @dnd-kit）
- [ ] 分类管理 Tab
- [ ] 变更日志 Tab（含筛选/导出）

#### Step 3: API 与优化（0.5 天）
- [ ] API 集成
- [ ] 拖拽排序
- [ ] 错误处理

#### Step 4: 联调（0.5 天）
- [ ] 端到端测试

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **拖拽库** | 未声明 | 明确使用 @dnd-kit |
| **配置审批** | 未设计 | 新增关键配置审批机制 |
| **分类层级** | 未明确 | 明确 V1 扁平/V2 树形 |
| **变更日志** | 基础表格 | 新增筛选和导出 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（拖拽库选择）
2. 建议修改项已补充（审批机制、分类层级、日志筛选）
3. 风险等级：低
