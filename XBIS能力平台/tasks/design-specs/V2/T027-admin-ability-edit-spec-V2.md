# T027 管理端能力编辑页 — 修正版设计方案（V2）

> 原设计方案: [T027-admin-ability-edit-spec.md](../T027-admin-ability-edit-spec.md)
> 评审报告: [T027-admin-ability-edit-Reviewer.md](../T027-admin-ability-edit-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | SchemaEditor 采用渐进式实现 | 第 2 节组件拆分 |
| 2 | Tab 内容懒加载 | 第 8 节性能优化 |
| 3 | 明确保存策略 | 第 3 节数据流 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加版本历史查看 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "编辑能力"
    │   └── Actions: [保存草稿] [预览] [发布]
    └── FormPageShell
        └── Tabs
            ├── Tab 1: 基础信息
            ├── Tab 2: 输入输出 Schema
            ├── Tab 3: UI 配置
            ├── Tab 4: 执行配置
            ├── Tab 5: 计费配置
            └── Tab 6: 发布管理
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 文本输入 |
| Tabs | Tab 切换 |
| Form | 表单容器 |
| Modal | 确认弹窗 |

#### business/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| SchemaEditor | Schema 编辑 | **渐进式实现**：第一阶段支持基础类型（string/number/boolean），后续支持对象/数组 |
| FormBuilder | UI 表单构建 | **渐进式实现**：第一阶段支持基础字段配置 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| BasicInfoTab | 基础信息 Tab | **新建** |
| SchemaTab | Schema Tab | **新建** — 含 SchemaEditor |
| UiConfigTab | UI 配置 Tab | **新建** — 含 FormBuilder |
| ExecutionTab | 执行配置 Tab | **新建** |
| BillingTab | 计费配置 Tab | **新建** |
| PublishTab | 发布管理 Tab | **新建** — 含版本历史 |

---

### 3. 数据流（V2 修正）【已修复】

```
[保存草稿]
    │
    ▼
[自动保存（Debounce 3s）]
    │
    ▼
[用户点击保存]
    │
    ▼
[如果自动保存正在进行，等待完成后再执行手动保存] 【已修复】
    │
    ▼
[API: PUT /admin-api/v1/abilities/:id]
    │
    ▼
[保存成功提示]
```

#### 保存策略（V2 明确）【已修复】

```
- 自动保存：仅保存草稿状态，不提示用户
- 手动保存：保存并提示成功
- 冲突处理：用户点击保存时，如果自动保存正在进行，等待自动保存完成后再执行手动保存
```

---

### 4. 状态管理

```typescript
interface AdminAbilityEditState {
  pageState: 'loading' | 'idle' | 'saving' | 'publishing' | 'error';
  abilityId: string;
  formData: AbilityEditForm;
  originalData: AbilityEditForm | null;
  activeTab: string;
  isDirty: boolean;
  autoSaving: boolean; // 【新增】自动保存状态
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力详情 | GET | `/admin-api/v1/abilities/:id` | 查询能力详情 |
| 能力更新 | PUT | `/admin-api/v1/abilities/:id` | 更新能力 |
| 能力发布 | POST | `/admin-api/v1/abilities/:id/publish` | 发布能力 |
| 版本历史 | GET | `/admin-api/v1/abilities/:id/versions` | 【新增】查询版本历史 |

---

### 6. 用户交互流程

```
[进入编辑页]
    │
    ▼
[加载能力数据]
    │
    ▼
[编辑各 Tab]
    │
    ▼
[保存草稿 / 自动保存]
    │
    ▼
[点击发布]
    │
    ▼
[全量校验]
    │
    ▼
[发布成功]
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 版本冲突 | 提示更换版本号 |
| 校验失败 | 定位到错误 Tab |

---

### 8. 性能优化（V2 修正）【已修复】

```
Tab 内容懒加载：
{activeTab === 'schema' && <SchemaTab />}
{activeTab === 'ui' && <UiConfigTab />}
...
```

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| SchemaEditor 复杂度高 | 高 | 完整实现工作量大 | 渐进式实现 |
| 表单数据量大 | 中 | 6 Tab 数据量大 | 懒加载 + 分步保存 |

---

### 10. 开发步骤拆分

#### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + Tabs 结构

#### Step 2: Tab 组件（1.5 天）
- [ ] BasicInfoTab / ExecutionTab / BillingTab / PublishTab

#### Step 3: 复杂编辑器（1.5 天）【已修复】
- [ ] SchemaEditor（第一阶段：基础类型）
- [ ] FormBuilder（第一阶段：基础字段）

#### Step 4: 保存与发布（1 天）【已修复】
- [ ] 自动保存 + 手动保存策略
- [ ] 发布流程

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **SchemaEditor** | 完整实现 | 渐进式实现（先基础类型） |
| **Tab 加载** | 同时挂载 | 懒加载（切换时才渲染） |
| **保存策略** | 未明确 | 明确自动/手动保存优先级 |
| **版本历史** | 未设计 | 新增版本历史查看 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（渐进式实现、懒加载、保存策略）
2. 建议修改项已补充（版本历史）
3. 风险等级：高（需严格控制 SchemaEditor 范围）
