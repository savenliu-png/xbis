# T031 管理端套餐配置 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "套餐配置"
    │   ├── Breadcrumb: 概览 / 套餐配置
    │   └── Actions: [新建套餐]
    └── ListPageShell
        ├── DataTable
        │   ├── 套餐列表
        │   └── 操作列（编辑/删除/上下架）
        └── Pagination
```

### 1.2 套餐编辑（Modal/Drawer）
```
PlanFormModal / PlanFormDrawer
├── 基础信息
│   ├── 名称 / 描述
│   ├── 价格 / 原价 / 币种
│   └── 排序 / 上下架状态
├── 配额配置
│   └── 调用次数配额
├── 功能特性
│   └── 特性列表（可增删）
└── 计费规则
    └── BillingRuleEditor（表格形式）
```

### 1.3 模板使用
| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 列表 | ListPageShell | 列表页模板 |
| 表单 | FormPageShell（内嵌） | Modal/Drawer 内表单 |

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）
| 组件 | 用途 |
|------|------|
| Button, Input, Table, Tag, Badge, Switch, Modal, Form, Select, Skeleton, Empty, Pagination | 标准用法 |

### 2.2 business/ 层组件（已存在）
| 组件 | 用途 | 说明 |
|------|------|------|
| PlanCard | 套餐卡片 | T014 已开发，预览用 |

### 2.3 blocks/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| PlanTable | 套餐表格 | 含状态/操作列 |
| PlanForm | 套餐表单 | Modal/Drawer 内表单 |
| BillingRuleEditor | 计费规则编辑器 | 表格形式配置阶梯定价 |

### 2.4 BillingRuleEditor（表格形式）
```typescript
// 表格形式配置阶梯定价
interface BillingTier {
  minUsage: number;
  maxUsage: number;
  unitPrice: number;
}

// 表格列：起始用量 | 结束用量 | 单价
// 自动校验区间连续性
```

---

## 3. 数据流

### 3.1 列表加载
```
[进入 /admin/billing/plans]
    │
    ▼
[API: GET /admin-api/v1/billing/plans]
    │
    ▼
[渲染套餐列表]
```

### 3.2 创建/编辑套餐
```
[点击「新建」或「编辑」]
    │
    ▼
[Modal/Drawer 打开]
    │
    ▼
[编辑表单（含计费规则表格配置）]
    │
    ▼
[点击预览（可选）]
    │
    ▼
[点击保存]
    │
    ▼
[API: POST/PUT /admin-api/v1/billing/plans/:id]
    │
    ├── 成功 → 关闭 Modal → 刷新列表
    └── 失败 → 显示校验错误
```

### 3.3 删除套餐
```
[点击「删除」]
    │
    ▼
[Modal 确认]
    │
    ▼
[API: DELETE /admin-api/v1/billing/plans/:id]
    │
    ├── 成功 → 刷新列表
    └── 失败 → 提示「套餐有用户，请先下架」
```

### 3.4 上下架套餐
```
[点击「上下架」Switch]
    │
    ▼
[确认弹窗（展示影响范围）]
    │
    ├── 当前订阅用户数
    └── 影响说明（"下架后新用户无法购买，已订阅用户不受影响"）
    │
    ▼
[确认/取消]
```

---

## 4. 状态管理

```typescript
interface AdminBillingConfigState {
  pageState: 'loading' | 'idle' | 'error';
  plans: AdminPlanItem[];
  total: number;
  formModalOpen: boolean;
  editingPlan: AdminPlanItem | null;
  formData: PlanForm;
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 套餐列表 | GET | `/admin-api/v1/billing/plans` | 查询所有套餐 |
| 套餐创建 | POST | `/admin-api/v1/billing/plans` | 新建套餐 |
| 套餐更新 | PUT | `/admin-api/v1/billing/plans/:id` | 编辑套餐 |
| 套餐删除 | DELETE | `/admin-api/v1/billing/plans/:id` | 删除套餐 |

---

## 6. 用户交互流程

```
[进入套餐配置]
    │
    ▼
[浏览套餐列表]
    │
    ├── 点击「新建套餐」
    │       │
    │       ▼
    │   [Modal 打开空白表单]
    │       │
    │       ▼
    │   [填写信息 + 配置计费规则（表格形式）]
    │       │
    │       ▼
    │   [点击预览（可选）]
    │       │
    │       ▼
    │   [保存]
    │
    ├── 点击「编辑」
    │       │
    │       ▼
    │   [Modal 打开预填充表单]
    │       │
    │       ▼
    │   [修改 + 保存]
    │
    ├── 点击「删除」
    │       │
    │       ▼
    │   [确认 Modal]
    │       │
    │       ▼
    │   [删除/取消]
    │
    └── 点击「上下架」
            │
            ▼
        [确认弹窗（展示影响范围）]
            │
            ▼
        [确认/取消]
```

### 上下架影响提示
确认弹窗中显示：
- 当前订阅用户数
- 影响说明（"下架后新用户无法购买，已订阅用户不受影响"）

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 删除有用户的套餐 | 提示「套餐有用户订阅，请先下架」 |
| 价格错误 | 前端校验（价格 ≥ 0） |
| 阶梯定价冲突 | 前端校验区间连续性 |
| 无套餐 | Empty + 引导新建 |

---

## 8. 性能优化

- 套餐数量通常较少（< 50），无需虚拟滚动
- 表单校验实时反馈

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 误删除套餐 | 影响已订阅用户 | 二次确认；有用户时禁止删除 |
| 价格配置错误 | 影响计费 | 预览确认；前端校验 |
| 计费规则复杂 | 阶梯定价容易配错 | 表格形式编辑器；实时校验；实时预览 |

---

## 10. 开发步骤

### Step 1: 列表页（1 天）
- [ ] 套餐列表展示
- [ ] 上下架 Switch（含影响提示）
- [ ] 删除功能

### Step 2: 表单（1 天）
- [ ] PlanForm 组件
- [ ] BillingRuleEditor 组件（表格形式阶梯定价）
- [ ] 创建/编辑 API
- [ ] 预览功能

### Step 3: 优化（0.5 天）
- [ ] 表单校验
- [ ] 错误处理

### Step 4: 联调（0.5 天）
- [ ] API 联调
- [ ] 验收检查

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/admin/BillingConfig/
├── index.tsx
├── BillingConfigPage.tsx
├── components/
│   ├── PlanTable.tsx
│   ├── PlanForm.tsx
│   └── BillingRuleEditor.tsx
└── hooks/
    └── useBillingConfig.ts
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/admin.tsx
```
