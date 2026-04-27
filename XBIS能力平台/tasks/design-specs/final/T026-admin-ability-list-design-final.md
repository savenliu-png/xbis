# T026 管理端能力管理列表 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AdminPageShell
├── AdminHeader (管理端头部)
│   ├── Logo
│   ├── AdminNav (管理端导航)
│   └── UserMenu
├── AdminSidebar (管理端侧边栏)
│   └── NavMenu (导航菜单)
└── MainContent (主内容区)
    ├── PageHeader (页面头部)
    │   ├── Title: "能力管理"
    │   └── ActionButtons (操作按钮: [新建能力])
    ├── FilterBar (筛选栏)
    │   ├── SearchInput (搜索框)
    │   ├── CategorySelect (分类选择)
    │   ├── StatusSelect (状态选择)
    │   └── EnabledSwitch (启用状态)
    ├── BatchActionBar (批量操作栏)
    ├── AbilityTable (能力表格)
    │   ├── TableHeader (表头) — 含 Checkbox 全选
    │   ├── TableBody (表格内容)
    │   └── TableFooter (分页)
    └── DeleteConfirmModal (删除确认弹窗)
```

### 1.2 页面模板
- **使用模板**: `AdminPageShell`
- **布局模式**: 侧边栏 + 主内容区（管理端标准布局）
- **响应式**: 管理端无需移动端适配，固定桌面端 ≥1200px

### 1.3 路由与导航
| 路由 | 说明 | 导航入口 |
|------|------|----------|
| `/admin/abilities` | 能力管理列表 | 管理端导航「能力管理」 |

---

## 2. 组件拆分

### 2.1 基础组件 (base/)
| 组件 | 来源 | 用途 | 复用情况 |
|------|------|------|----------|
| Button | T002 | 操作按钮 | 全项目复用 |
| Input | T002 | 搜索框 | 全项目复用 |
| Table | T002 | 能力表格 | 全项目复用 |
| Tag | T002 | 状态标签 | 全项目复用 |
| Badge | T002 | 状态标识 | 全项目复用 |
| Switch | T002 | 状态切换 | 全项目复用 |
| Modal | T002 | 确认弹窗 | 全项目复用 |
| Skeleton | T002 | 加载骨架 | 全项目复用 |
| Empty | T002 | 空态展示 | 全项目复用 |
| Pagination | T002 | 分页 | 全项目复用 |
| Select | T002 | 筛选下拉 | 全项目复用 |
| Checkbox | T002 | 批量选择 | 全项目复用 |

### 2.2 业务组件 (business/)
| 组件 | 来源 | 用途 | 复用情况 |
|------|------|------|----------|
| StatusBadge | T003 | 状态标识 | 全项目复用 |
| AbilityStatusTag | 新建 | 能力状态标签 | 管理端复用 |

### 2.3 页面块组件 (blocks/)
| 组件 | 来源 | 用途 | 复用情况 |
|------|------|------|----------|
| FilterBar | 新建 | 筛选栏 | 管理端复用 |
| AbilityTable | 新建 | 能力表格 | 当前页专用 |
| BatchActionBar | 新建 | 批量操作栏 | 管理端复用 |

### 2.4 布局组件 (layout/)
| 组件 | 来源 | 用途 |
|------|------|------|
| AdminPageShell | T004 | 管理端页面骨架 |
| PageHeader | T004 | 页面头部 |

---

## 3. 数据流

### 3.1 页面加载数据流
```
[管理员进入 /admin/abilities]
    │
    ▼
[检查权限] ──无权限──► [跳转 403]
    │
    ▼
[加载能力列表]
    │
    ▼
[GET /admin-api/v1/abilities] ──► AdminAbilityItem[]
    │
    ▼
[渲染表格]
```

### 3.2 筛选数据流
```
[管理员操作筛选条件]
    │
    ▼
[更新 filter state]
    │
    ▼
[重新请求列表]
    │
    ▼
[GET /admin-api/v1/abilities?filter={filter}]
    │
    ▼
[更新表格数据]
```

### 3.3 状态切换数据流（含乐观更新回滚）
```
[管理员点击状态开关]
    │
    ▼
[乐观更新：先更新本地状态]
    │
    ▼
[API: PUT /admin-api/v1/abilities/:id/status]
    │
    ├── 成功：保持更新
    └── 失败：恢复原始状态，显示错误提示
```

### 3.4 删除数据流
```
[管理员点击删除按钮]
    │
    ▼
[显示二次确认弹窗（展示关联影响）]
    │
    ▼
[确认后 DELETE /admin-api/v1/abilities/:id]
    │
    ▼
[重新加载列表]
    │
    ▼
[显示成功提示]
```

### 3.5 批量操作数据流
```
[勾选多行]
    │
    ▼
[BatchActionBar 显示]
    │
    ▼
[选择批量操作：启用/禁用/删除]
    │
    ▼
[调用对应批量 API]
    │
    ▼
[刷新列表]
```

---

## 4. 状态管理

### 4.1 页面状态机
```
[LOADING] ──加载完成──► [SUCCESS]
    │                      │
    │                      ├── 无数据 ──► [EMPTY]
    │                      │
    │                      └── 有数据 ──► [IDLE]
    │
    └── 加载失败 ──► [ERROR]
```

### 4.2 状态定义
```typescript
interface AbilityListPageState {
  // 页面状态
  pageState: 'loading' | 'idle' | 'error';

  // 加载状态
  loading: boolean;
  tableLoading: boolean;

  // 数据状态
  abilities: AdminAbilityItem[];
  total: number;

  // 筛选状态
  filter: AdminAbilityFilter;

  // 分页状态
  pagination: {
    page: number;
    pageSize: number;
  };

  // 批量操作状态
  selectedIds: string[];

  // 操作状态
  deleteModalVisible: boolean;
  deletingAbility: AdminAbilityItem | null;
  toggleLoading: boolean;

  // 错误状态
  error: Error | null;
}
```

### 4.3 各状态处理
| 状态 | Loading | Empty | Error | 说明 |
|------|---------|-------|-------|------|
| 初始加载 | Table Skeleton | - | 错误提示 | 首次加载 |
| 筛选加载 | Table Loading | Empty 组件 | 错误提示 | 筛选后加载 |
| 操作加载 | 行内 Loading | - | 错误提示 | 状态切换/删除 |

---

## 5. API 调用

### 5.1 接口清单
| 接口 | Method | Path | 触发时机 | 参数 | 错误处理 |
|------|--------|------|----------|------|----------|
| 能力列表 | GET | `/admin-api/v1/abilities` | 页面加载/筛选/分页 | `AdminAbilityFilter` | 显示错误提示 |
| 状态切换 | PUT | `/admin-api/v1/abilities/:id/status` | 点击开关 | `AbilityStatusToggleRequest` | 显示错误提示，回滚状态 |
| 删除能力 | DELETE | `/admin-api/v1/abilities/:id` | 确认删除 | 无 | 显示错误提示 |
| 批量删除 | POST | `/admin-api/v1/abilities/batch-delete` | 批量操作 | `{ ids: string[] }` | 显示错误提示 |
| 批量更新状态 | POST | `/admin-api/v1/abilities/batch-status` | 批量操作 | `BatchStatusRequest` | 显示错误提示 |

### 5.2 请求参数
```typescript
// 能力列表请求
interface AdminAbilityFilter {
  keyword?: string;
  category?: string;
  status?: 'published' | 'draft' | 'deprecated';
  isEnabled?: boolean;
  page?: number;
  pageSize?: number;
}

// 状态切换请求
interface AbilityStatusToggleRequest {
  isEnabled: boolean;
}

// 批量操作请求
interface BatchDeleteRequest {
  ids: string[];
}

interface BatchStatusRequest {
  ids: string[];
  isEnabled: boolean;
}
```

### 5.3 响应处理
```typescript
// 列表响应
interface AdminAbilityListResponse {
  items: AdminAbilityItem[];
  total: number;
}

// AdminAbilityItem 包含关联统计字段（可选）：
// subscriberCount?: number  — 当前订阅用户数
// activeJobCount?: number   — 进行中任务数
// totalJobCount?: number    — 历史任务总数

// 错误处理策略
// - 401: 跳转登录页
// - 403: 跳转 403 页
// - 409: 能力正在使用，提示先下架
// - 500: 显示错误提示，支持重试
```

### 5.4 防抖处理
- 搜索框输入: 300ms debounce
- 筛选条件变更: 立即触发

---

## 6. 用户交互流程

### 6.1 主交互流程
```
[管理员进入能力管理]
    │
    ▼
[页面加载中] ──显示 Skeleton──► [数据加载完成]
    │
    ▼
[输入搜索关键词] ──300ms debounce──► [筛选结果]
    │
    ▼
[选择分类/状态] ──立即筛选──► [筛选结果]
    │
    ▼
[点击状态开关] ──乐观更新──► [切换状态]
    │
    ▼
[勾选多行] ──► [BatchActionBar 显示]
    │
    ▼
[批量启用/禁用/删除]
    │
    ▼
[点击编辑] ──跳转──► [/admin/abilities/:id/edit]
    │
    ▼
[点击删除] ──二次确认（展示关联影响）──► [删除能力]
```

### 6.2 删除前关联影响展示
确认弹窗中显示：
- 当前订阅用户数
- 进行中任务数
- 历史任务总数
- 提示"删除后这些任务将无法执行"

### 6.3 异常交互流程
| 场景 | 用户行为 | 系统响应 |
|------|----------|----------|
| 删除失败 | 点击"确定" | 显示错误原因，弹窗不关闭 |
| 切换失败 | 点击开关 | 回滚开关状态，显示错误 |
| 筛选无结果 | 输入关键词 | 显示 Empty 组件 |

### 6.4 交互细节
- **搜索框**: 支持回车搜索，带清除按钮
- **筛选条件**: 支持多条件组合，带重置按钮
- **状态开关**: 乐观更新，失败回滚
- **删除按钮**: 必须二次确认，显示能力名称及关联影响
- **分页**: 支持页码跳转，每页条数选择
- **表格排序**: 支持按名称/时间/状态排序

---

## 7. 边界与异常

### 7.1 空态处理
| 场景 | 空态表现 | 用户引导 |
|------|----------|----------|
| 无能力数据 | Empty 组件 | 提示"暂无能力，请先创建" |
| 筛选无结果 | Empty 组件 | 提示"无匹配结果，请调整筛选条件" |
| 无分类数据 | 分类下拉为空 | 禁用分类筛选 |

### 7.2 错误处理
| 错误类型 | 处理方式 | 用户反馈 |
|----------|----------|----------|
| 网络错误 | 显示错误提示 + 重试按钮 | "网络异常，请重试" |
| 删除失败 | 弹窗不关闭，显示错误 | "删除失败：能力正在使用" |
| 切换失败 | 回滚状态，显示错误 | "切换失败：请重试" |
| 权限错误 | 跳转 403 页面 | "无权限访问" |

### 7.3 权限控制
| 权限 | 控制点 | 行为 |
|------|--------|------|
| admin | 路由守卫 | 非管理员跳转 403 |
| ability.read | 页面访问 | 无权限隐藏入口 |
| ability.write | 编辑/删除 | 无权限禁用操作 |

---

## 8. 性能优化

### 8.1 加载优化
| 优化点 | 策略 | 说明 |
|--------|------|------|
| 骨架屏 | Table Skeleton | 减少白屏时间 |
| 分页加载 | 服务端分页 | 每页默认 20 条 |
| 搜索防抖 | 300ms debounce | 减少请求频率 |
| 数据缓存 | SWR / React Query | 缓存列表数据 |

### 8.2 渲染优化
| 优化点 | 策略 | 说明 |
|--------|------|------|
| 虚拟滚动 | >100 条数据 | 大数据量时启用 |
| 行内更新 | 局部刷新 | 状态切换只更新当前行 |
| 条件渲染 | 按需加载 | 弹窗懒加载 |

### 8.3 内存优化
- useEffect 清理定时器
- 组件卸载取消请求
- 弹窗关闭重置状态

---

## 9. 风险点

### 9.1 技术风险
| 风险 | 等级 | 说明 | 应对措施 |
|------|------|------|----------|
| 误删除 | 高 | 删除能力影响大 | 二次确认 + 关联影响展示 + 后端校验 |
| 状态冲突 | 中 | 能力状态可能冲突 | 后端校验 + 乐观更新 + 失败回滚 |
| 数据量大 | 中 | 能力数量可能很多 | 分页 + 虚拟滚动 |

### 9.2 业务风险
| 风险 | 等级 | 说明 | 应对措施 |
|------|------|------|----------|
| 删除影响 | 高 | 删除能力影响已订阅用户 | 提示关联任务数 |
| 状态切换影响 | 中 | 禁用能力影响新任务 | 提示影响范围 |

### 9.3 依赖风险
| 依赖 | 风险 | 应对措施 |
|------|------|----------|
| T003 业务组件 | 中 | 需先完成业务组件库 |
| T004 页面模板 | 中 | 需先完成 AdminPageShell |
| T008 ability API | 中 | 需先完成 ability API 层 |

---

## 10. 开发步骤

### Step 1: 基础准备 (0.5d)
- [ ] 确认 AdminPageShell 可用
- [ ] 确认基础组件可用
- [ ] 准备 mock 数据

### Step 2: 组件开发 (1d)
- [ ] 开发 FilterBar 页面块
- [ ] 开发 AbilityTable 页面块（含 Checkbox 选择）
- [ ] 开发 AbilityStatusTag 业务组件
- [ ] 开发 BatchActionBar 页面块

### Step 3: 页面开发 (0.5d)
- [ ] 开发能力管理列表页
- [ ] 集成筛选和表格
- [ ] 实现分页逻辑
- [ ] 实现表格排序

### Step 4: API 对接 (0.5d)
- [ ] 对接列表 API
- [ ] 对接状态切换 API
- [ ] 对接删除 API
- [ ] 对接批量操作 API

### Step 5: 交互与状态 (0.5d)
- [ ] 实现搜索防抖
- [ ] 实现状态切换确认（乐观更新 + 回滚）
- [ ] 实现删除二次确认（含关联影响展示）
- [ ] 实现批量操作
- [ ] 实现 loading/empty/error 状态

### Step 6: 联调与验收 (0.5d)
- [ ] 联调测试
- [ ] 验收标准检查
- [ ] 代码审查

---

## 附录

### 修改文件清单
| 文件 | 操作 | 说明 |
|------|------|------|
| `packages/pages/admin/AbilityListPage.tsx` | 新增 | 能力管理列表页 |
| `packages/components/business/AbilityStatusTag.tsx` | 新增 | 能力状态标签 |
| `packages/components/blocks/FilterBar.tsx` | 新增 | 筛选栏 |
| `packages/components/blocks/AbilityTable.tsx` | 新增 | 能力表格 |
| `packages/components/blocks/BatchActionBar.tsx` | 新增 | 批量操作栏 |
| `packages/shared/src/types/index.ts` | 修改 | 添加类型定义 |
| `packages/shared/src/api/services.ts` | 修改 | 添加 API 定义 |

### 新增组件清单
- `AbilityStatusTag` - business 层
- `FilterBar` - blocks 层
- `AbilityTable` - blocks 层
- `BatchActionBar` - blocks 层

### 数据流汇总
- 页面加载: 列表 API 请求
- 筛选: 条件变更触发列表刷新
- 状态切换: PUT 请求 + 乐观更新 + 失败回滚
- 删除: DELETE 请求 + 关联影响展示 + 列表刷新
- 批量操作: 批量 API 请求 + 列表刷新

### API 调用点汇总
- `GET /admin-api/v1/abilities`
- `PUT /admin-api/v1/abilities/:id/status`
- `DELETE /admin-api/v1/abilities/:id`
- `POST /admin-api/v1/abilities/batch-delete`
- `POST /admin-api/v1/abilities/batch-status`
