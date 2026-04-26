# T027 管理端能力编辑页 — 工程级设计方案（Design Spec）

> 任务卡：T027-admin-ability-edit.md  
> 状态：设计确认  
> 输出日期：2026-04-24

---

## 1. 页面结构

### 1.1 层级结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "编辑能力"
    │   ├── Breadcrumb: 概览 / 能力管理 / 编辑能力
    │   └── Actions: [保存草稿] [预览] [发布]
    └── FormPageShell
        └── Tabs (6 Tab)
            ├── Tab 1: 基础信息
            ├── Tab 2: 输入输出 Schema
            ├── Tab 3: UI 配置
            ├── Tab 4: 执行配置
            ├── Tab 5: 计费配置
            └── Tab 6: 发布管理
```

### 1.2 模板使用

| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 外层 | AdminPageShell | 管理端统一骨架，侧边栏 + 顶部栏 |
| 内容区 | FormPageShell | 表单页模板，支持分步/分 Tab |
| 表单 | Form + Tabs | 6 Tab 大表单 |

### 1.3 布局说明

- **Desktop (≥1200px)**: 侧边栏 256px + 主内容区（Tab 内容全宽）
- **管理端无需移动端设计**
- 每个 Tab 内部采用双栏布局：左侧标签 200px + 右侧表单控件

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）

| 组件 | 用途 | 位置 |
|------|------|------|
| Button | 保存/预览/发布按钮 | 全局 |
| Input | 文本输入（名称/版本等） | Tab1, Tab6 |
| Textarea | 大文本输入（描述/变更日志） | Tab1, Tab6 |
| Select | 下拉选择（分类/风险等级/执行模式） | Tab1, Tab4 |
| Tabs | Tab 切换 | 表单顶部 |
| Form | 表单容器 | 每个 Tab |
| Modal | 确认弹窗（发布确认） | 发布流程 |
| Skeleton | 加载骨架 | 初始加载 |
| Empty | 空态（新建能力时） | 初始状态 |

### 2.2 business/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| SchemaEditor | Schema 可视化编辑 | **新建** — 基于 JSON Schema 的编辑器，支持嵌套对象/数组定义 |
| FormBuilder | UI 表单构建器 | **新建** — 拖拽式表单字段配置，生成 uiSchema |

### 2.3 blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| BasicInfoTab | 基础信息 Tab | **新建** — 名称/描述/分类/标签/图标/风险等级 |
| SchemaTab | 输入输出 Schema Tab | **新建** — 包含 SchemaEditor |
| UiConfigTab | UI 配置 Tab | **新建** — 包含 FormBuilder |
| ExecutionTab | 执行配置 Tab | **新建** — 执行器/超时/重试 |
| BillingTab | 计费配置 Tab | **新建** — 价格/配额/阶梯定价 |
| PublishTab | 发布管理 Tab | **新建** — 版本/状态/变更日志 |

### 2.4 组件复用关系

```
SchemaEditor (business)
└── 基于开源库 react-jsonschema-form 或自研简化版
    ├── 支持类型：string/number/boolean/array/object
    ├── 支持属性：title/description/required/default/enum
    └── 嵌套编辑能力

FormBuilder (business)
└── 字段配置列表
    ├── 字段类型选择
    ├── 校验规则配置
    └── 布局配置
```

---

## 3. 数据流

### 3.1 页面加载数据流

```
[管理员进入 /admin/abilities/:id/edit]
    │
    ▼
[路由解析 abilityId]
    │
    ▼
[API: GET /admin-api/v1/abilities/:id]
    │
    ▼
[数据填充到 6 个 Tab 的表单]
    │
    ▼
[表单状态初始化完成 → idle]
```

### 3.2 表单保存数据流

```
[管理员点击「保存草稿」]
    │
    ▼
[收集 6 个 Tab 的表单数据 → AbilityEditForm]
    │
    ▼
[前端校验（必填项/格式）]
    │
    ├── 校验失败 → 显示错误提示 → 定位到错误 Tab
    │
    ▼
[API: PUT /admin-api/v1/abilities/:id]
    │
    ├── 成功 → 显示成功提示 → 更新本地状态
    │
    └── 失败 → 显示错误提示 → 保留表单数据
```

### 3.3 发布数据流

```
[管理员点击「发布」]
    │
    ▼
[Modal 确认弹窗]
    │
    ▼
[全量校验（所有必填项）]
    │
    ▼
[API: POST /admin-api/v1/abilities/:id/publish]
    │
    ├── 成功 → 跳转能力管理列表 → 显示发布成功
    │
    └── 失败 → 显示错误原因（版本冲突/校验失败）
```

### 3.4 数据流图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Router    │────▶│  abilityId  │────▶│  API GET    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                                │
                       ┌────────────────────────┘
                       ▼
              ┌─────────────────┐
              │  AbilityDetail   │
              │  (Admin 视角)    │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │ Tab 1-6 │   │ 草稿保存 │   │  发布   │
   │ 表单状态 │   │  PUT    │   │  POST   │
   └─────────┘   └─────────┘   └─────────┘
```

---

## 4. 状态管理

### 4.1 页面级状态机

```typescript
type PageState =
  | 'loading'      // 初始加载能力数据
  | 'idle'         // 正常编辑状态
  | 'saving'       // 保存中
  | 'publishing'   // 发布中
  | 'error'        // 加载/保存错误
  | 'validating'   // 校验中
```

### 4.2 状态定义

```typescript
interface AdminAbilityEditState {
  // 页面状态
  pageState: PageState;

  // 数据
  abilityId: string;
  formData: AbilityEditForm;
  originalData: AbilityEditForm | null; // 用于 dirty check

  // Tab 状态
  activeTab: string; // 'basic' | 'schema' | 'ui' | 'execution' | 'billing' | 'publish'

  // 校验状态
  validationErrors: Record<string, string[]>; // tabKey -> errors

  // 错误信息
  error: ApiError | null;

  // 草稿自动保存
  lastAutoSaveAt: string | null;
  isDirty: boolean;
}
```

### 4.3 各状态 UI 表现

| 状态 | UI 表现 | 说明 |
|------|---------|------|
| loading | Skeleton 骨架屏 | 全页加载 |
| idle | 正常表单展示 | 可编辑 |
| saving | 保存按钮 loading | 禁用其他操作 |
| publishing | 发布按钮 loading + Modal | 全量校验中 |
| error | 错误提示 + 重试按钮 | 可重试 |
| validating | Tab 错误高亮 | 显示校验错误 |

### 4.4 空态/错误态

| 场景 | 触发条件 | 展示内容 |
|------|----------|----------|
| 空态 | 新建能力（无 id） | 空白表单，所有字段为空 |
| 加载错误 | API 失败 | ErrorState 组件 + 重试按钮 |
| 保存错误 | PUT 失败 | Toast 错误提示 |
| 发布错误 | POST 失败 | Modal 显示详细错误 |
| 权限不足 | 非管理员 | 403 页面 |

---

## 5. API 调用

### 5.1 API 列表

| API | Method | Path | 触发时机 | 请求体 | 错误处理 |
|-----|--------|------|----------|--------|----------|
| 能力详情 | GET | `/admin-api/v1/abilities/:id` | 页面加载 | — | 显示错误态，可重试 |
| 能力更新 | PUT | `/admin-api/v1/abilities/:id` | 保存草稿/自动保存 | `AbilityEditForm` | Toast 错误提示 |
| 能力发布 | POST | `/admin-api/v1/abilities/:id/publish` | 点击发布 | `AbilityPublishRequest` | Modal 显示错误详情 |

### 5.2 请求参数

```typescript
// GET /admin-api/v1/abilities/:id
// Path: { id: string }

// PUT /admin-api/v1/abilities/:id
interface AbilityEditForm {
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  icon?: string;
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  requestSchema: Record<string, any>;
  responseSchema: Record<string, any>;
  uiSchema: Record<string, any>;
  formConfig: Record<string, any>;
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  timeout: number;
  retryCount: number;
  executorId?: string;
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  freeQuota?: number;
  overagePrice?: number;
  status: 'published' | 'draft' | 'deprecated';
  version: string;
  changeLog: string;
}

// POST /admin-api/v1/abilities/:id/publish
interface AbilityPublishRequest {
  abilityId: string;
  version: string;
  changeLog: string;
}
```

### 5.3 错误处理策略

| 错误码 | 场景 | 处理方式 |
|--------|------|----------|
| 400 | 参数错误/校验失败 | 显示校验错误，定位到对应 Tab |
| 403 | 权限不足 | 跳转 403 页面 |
| 404 | 能力不存在 | 显示错误态，提供返回列表按钮 |
| 409 | 版本冲突 | Modal 提示更换版本号 |
| 500 | 服务器错误 | Toast 提示，保留表单数据 |

---

## 6. 用户交互流程

### 6.1 主流程：编辑并发布能力

```
[进入编辑页]
    │
    ▼
[加载能力数据] ──► [展示 6 Tab 表单]
    │
    ▼
[管理员编辑各 Tab]
    │
    ├── Tab1: 基础信息 ──► 填写名称/描述/分类/标签
    ├── Tab2: Schema ──► 使用 SchemaEditor 定义输入输出
    ├── Tab3: UI 配置 ──► 使用 FormBuilder 配置表单
    ├── Tab4: 执行配置 ──► 选择执行器/设置超时
    ├── Tab5: 计费配置 ──► 设置价格/配额
    └── Tab6: 发布管理 ──► 设置版本/变更日志
    │
    ▼
[点击「保存草稿」] ──► [校验当前 Tab] ──► [API 保存]
    │
    ▼
[点击「预览」] ──► [新窗口打开预览页]
    │
    ▼
[点击「发布」] ──► [Modal 确认] ──► [全量校验]
    │
    ▼
[API 发布] ──► [成功跳转能力列表]
```

### 6.2 异常流程

| 场景 | 用户行为 | 系统响应 |
|------|----------|----------|
| 切换 Tab 有未保存更改 | 点击其他 Tab | 提示「有未保存的更改，是否保存？」 |
| 离开页面有未保存更改 | 关闭/刷新页面 | beforeunload 提示确认 |
| 发布校验失败 | 点击发布 | Modal 显示失败原因，定位到错误 Tab |
| 版本冲突 | 发布时 | Modal 提示「版本已存在，请更换版本号」 |
| 自动保存失败 | 定时触发 | 静默失败，下次用户操作时重试 |

### 6.3 快捷键

| 快捷键 | 功能 |
|--------|------|
| Ctrl/Cmd + S | 保存草稿 |
| Ctrl/Cmd + P | 预览（如已保存） |

---

## 7. 边界与异常

### 7.1 空态处理

| 场景 | 处理方式 |
|------|----------|
| 新建能力 | 所有字段为空，显示占位提示 |
| Schema 为空 | SchemaEditor 显示空对象 `{}` |
| UI 配置为空 | FormBuilder 显示空列表 |

### 7.2 错误处理

| 场景 | 处理方式 |
|------|----------|
| 加载失败 | ErrorState 组件 + 重试按钮 |
| 保存失败 | Toast 错误 + 保留表单数据 |
| 发布失败 | Modal 错误详情 + 返回编辑 |
| 网络中断 | 检测网络恢复后自动重试保存 |

### 7.3 权限控制

| 角色 | 权限 |
|------|------|
| 管理员 | 可编辑所有字段 |
| 运营人员 | 可编辑基础信息/计费配置，不可编辑 Schema/执行配置 |
| 普通用户 | 无权限访问 |

### 7.4 边界情况

- **复杂 Schema**: 支持嵌套 5 层以内，超过时提示简化
- **大表单数据**: 单个 Tab 表单数据 > 100KB 时提示优化
- **版本号格式**: 强制 `v{major}.{minor}.{patch}` 格式
- **并发编辑**: 保存时检测版本号，冲突时提示合并

---

## 8. 性能优化

### 8.1 加载优化

| 优化点 | 策略 |
|--------|------|
| 首屏加载 | 并行加载能力数据 + 执行器列表 |
| Tab 切换 | 懒加载 SchemaEditor/FormBuilder（首屏只加载当前 Tab） |
| 代码分割 | SchemaEditor/FormBuilder 动态导入 |

### 8.2 运行时优化

| 优化点 | 策略 |
|--------|------|
| 自动保存 | Debounce 3s，避免频繁请求 |
| 表单校验 | 当前 Tab 实时校验，其他 Tab 保存时校验 |
| 内存管理 | 离开页面清理 SchemaEditor 实例 |

### 8.3 渲染优化

| 优化点 | 策略 |
|--------|------|
| Tab 内容 | 非活跃 Tab 保持挂载但暂停响应式更新 |
| SchemaEditor | 大数据量 Schema 使用虚拟滚动 |
| 表单控件 | 使用 React.memo 减少重渲染 |

---

## 9. 风险点

### 9.1 高风险

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| **Schema 编辑器复杂度高** | 需要支持 JSON Schema 完整语法，开发工作量大 | 使用开源库 react-jsonschema-form 作为基础，二次封装为 SchemaEditor；如时间不足，先支持基础类型（string/number/boolean/object/array） |
| **表单数据量大** | 6 Tab 大表单，状态管理复杂 | 每个 Tab 独立管理表单状态，提交时聚合；使用 Form 组件的嵌套 Form 能力 |

### 9.2 中风险

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 版本冲突 | 多人同时编辑同一能力 | 乐观锁机制，保存时校验版本号 |
| 数据一致性 | 6 Tab 数据分散，可能不一致 | 统一 AbilityEditForm 类型，提交时全量校验 |
| 自动保存冲突 | 用户操作和自动保存同时触发 | Debounce + 队列机制，确保顺序执行 |

### 9.3 低风险

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 浏览器兼容性 | 复杂表单控件 | 仅支持现代浏览器 |
| 暗色模式适配 | 编辑器主题 | 使用 Design Tokens 动态主题 |

---

## 10. 开发步骤拆分

### Step 1: 基础结构（0.5 天）

- [ ] 创建页面路由 `/admin/abilities/:id/edit`
- [ ] 搭建 AdminPageShell + FormPageShell + Tabs 结构
- [ ] 实现页面状态机（loading/idle/error）

### Step 2: Tab 组件（1 天）

- [ ] 实现 BasicInfoTab（基础信息）
- [ ] 实现 ExecutionTab（执行配置）
- [ ] 实现 BillingTab（计费配置）
- [ ] 实现 PublishTab（发布管理）

### Step 3: 复杂编辑器（1.5 天）

- [ ] 集成/开发 SchemaEditor（输入输出 Tab）
- [ ] 集成/开发 FormBuilder（UI 配置 Tab）
- [ ] 实现编辑器与表单数据的双向绑定

### Step 4: 数据流与 API（1 天）

- [ ] 实现 API 调用（GET/PUT/POST）
- [ ] 实现表单校验逻辑
- [ ] 实现自动保存（Debounce 3s）
- [ ] 实现 dirty check

### Step 5: 交互完善（0.5 天）

- [ ] 实现发布确认 Modal
- [ ] 实现 Tab 切换未保存提示
- [ ] 实现离开页面确认（beforeunload）
- [ ] 实现错误处理与重试

### Step 6: 联调与优化（0.5 天）

- [ ] API 联调
- [ ] 性能优化（懒加载/代码分割）
- [ ] 暗色模式适配
- [ ] 验收标准检查

---

## 附录：修改文件清单

### 新增文件

```
packages/pages/admin/AbilityEditPage/
├── index.tsx                 # 页面入口
├── AbilityEditPage.tsx       # 主组件
├── hooks/
│   ├── useAbilityEdit.ts     # 编辑逻辑 Hook
│   └── useAutoSave.ts        # 自动保存 Hook
├── components/
│   ├── BasicInfoTab.tsx      # 基础信息 Tab
│   ├── SchemaTab.tsx         # Schema Tab
│   ├── UiConfigTab.tsx       # UI 配置 Tab
│   ├── ExecutionTab.tsx      # 执行配置 Tab
│   ├── BillingTab.tsx        # 计费配置 Tab
│   └── PublishTab.tsx        # 发布管理 Tab
└── types.ts                  # 页面级类型

packages/components/business/
├── SchemaEditor/
│   ├── index.tsx
│   ├── SchemaNode.tsx
│   └── types.ts
└── FormBuilder/
    ├── index.tsx
    ├── FieldConfig.tsx
    └── types.ts
```

### 修改文件

```
packages/shared/src/types/index.ts          # 添加 AbilityEditForm, SchemaEditorConfig, AbilityPublishRequest
packages/shared/src/api/services.ts         # 添加 adminApi.abilities.get, update, publish
packages/router/admin.tsx                   # 添加 /admin/abilities/:id/edit 路由
```

---

## 附录：数据流图示

```
┌─────────────────────────────────────────────────────────────┐
│                      AbilityEditPage                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  PageHeader  │  │   Tabs      │  │   TabContent        │  │
│  │ [保存][预览] │  │ ┌─┬─┬─┬─┬─┐ │  │ ┌─────────────────┐ │  │
│  │ [发布]      │  │ │1│2│3│4│5│6│ │  │ │ 当前 Tab 表单    │ │  │
│  └─────────────┘  │ └─┴─┴─┴─┴─┘ │  │ └─────────────────┘ │  │
│                   └─────────────┘  └─────────────────────┘  │
│                              │                               │
│                              ▼                               │
│                   ┌─────────────────────┐                    │
│                   │   useAbilityEdit     │                    │
│                   │  ┌───────────────┐  │                    │
│                   │  │  formData     │  │                    │
│                   │  │  originalData │  │                    │
│                   │  │  isDirty      │  │                    │
│                   │  │  validation   │  │                    │
│                   │  └───────────────┘  │                    │
│                   └──────────┬──────────┘                    │
│                              │                               │
│              ┌───────────────┼───────────────┐               │
│              ▼               ▼               ▼               │
│        ┌─────────┐    ┌─────────┐    ┌─────────┐            │
│        │ API GET │    │ API PUT │    │ API POST│            │
│        │ (load)  │    │ (save)  │    │(publish)│            │
│        └─────────┘    └─────────┘    └─────────┘            │
└─────────────────────────────────────────────────────────────┘
```
