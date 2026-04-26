# T031 管理端套餐配置 — 修正版设计方案（V2）

> 原设计方案: [T031-admin-billing-config-spec.md](../T031-admin-billing-config-spec.md)
> 评审报告: [T031-admin-billing-config-Reviewer.md](../T031-admin-billing-config-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 简化 BillingRuleEditor 实现 | 第 2 节组件拆分 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 上下架时提示影响范围 | 第 6 节用户交互流程 |
| 2 | 增加套餐排序功能 | 第 6 节用户交互流程 |
| 3 | 增加套餐预览功能 | 第 6 节用户交互流程 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "套餐配置"
    │   ├── Breadcrumb: 概览 / 套餐配置
    │   └── Actions: [新建套餐]
    └── ListPageShell
        ├── PlanTable
        │   └── 套餐列表
        └── Pagination
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Input | 输入框 |
| Table | 套餐表格 |
| Tag | 状态标签 |
| Switch | 上下架切换 |
| Modal | 确认弹窗 |
| Form | 表单容器 |
| Select | 下拉选择 |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Pagination | 分页 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| PlanCard | 套餐卡片 |

#### blocks/ 层组件（V2 修正）【已修复】

| 组件 | 用途 | 说明 |
|------|------|------|
| PlanTable | 套餐表格 | 含状态/操作列 |
| PlanForm | 套餐表单 | Modal/Drawer 内表单 |
| BillingRuleEditor | 计费规则编辑器 | **已修复** — 表格形式配置阶梯定价 |

#### BillingRuleEditor（V2 简化）【已修复】

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

### 3. 数据流

```
[进入 /admin/billing/plans]
    │
    ▼
[API: GET /admin-api/v1/billing/plans]
    │
    ▼
[渲染套餐列表]
```

---

### 4. 状态管理

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

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 套餐列表 | GET | `/admin-api/v1/billing/plans` | 查询所有套餐 |
| 套餐创建 | POST | `/admin-api/v1/billing/plans` | 新建套餐 |
| 套餐更新 | PUT | `/admin-api/v1/billing/plans/:id` | 编辑套餐 |
| 套餐删除 | DELETE | `/admin-api/v1/billing/plans/:id` | 删除套餐 |

---

### 6. 用户交互流程（V2 修正）【已修复】

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
    │   [填写信息 + 配置计费规则（表格形式）] 【已修复】
    │       │
    │       ▼
    │   [点击预览（可选）] 【新增】
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
    └── 点击「删除」
            │
            ▼
        [确认 Modal]
            │
            ▼
        [删除/取消]
    │
    └── 点击「上下架」
            │
            ▼
        [确认弹窗（展示影响范围）] 【新增】
            │
            ▼
        [确认/取消]
```

#### 上下架影响提示（V2 新增）【已修复】

```
确认弹窗中显示：
- 当前订阅用户数
- 影响说明（"下架后新用户无法购买，已订阅用户不受影响"）
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 删除有用户的套餐 | 提示「套餐有用户订阅，请先下架」 |
| 价格错误 | 前端校验（价格 ≥ 0） |
| 阶梯定价冲突 | 前端校验区间连续性 |
| 无套餐 | Empty + 引导新建 |

---

### 8. 性能优化

- 套餐数量通常较少（< 50），无需虚拟滚动
- 表单校验实时反馈

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 误删除套餐 | 中 | 影响已订阅用户 | 二次确认 + 有用户时禁止删除 |
| 价格配置错误 | 中 | 影响计费 | 预览确认 + 前端校验 |
| 计费规则复杂 | 中 | 阶梯定价容易配错 | 表格形式 + 实时校验 |

---

### 10. 开发步骤拆分

#### Step 1: 列表页（1 天）
- [ ] 套餐列表展示
- [ ] 上下架 Switch（含影响提示）
- [ ] 删除功能

#### Step 2: 表单（1 天）【已修复】
- [ ] PlanForm 组件
- [ ] BillingRuleEditor 组件（表格形式）
- [ ] 创建/编辑 API

#### Step 3: 优化（0.5 天）
- [ ] 表单校验
- [ ] 错误处理
- [ ] 预览功能

#### Step 4: 联调（0.5 天）
- [ ] API 联调
- [ ] 验收检查

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **BillingRuleEditor** | 未明确实现方式 | 明确为表格形式配置 |
| **上下架影响** | 未提示 | 新增影响范围提示 |
| **套餐排序** | 未设计 | 新增排序功能 |
| **套餐预览** | 未设计 | 新增预览功能 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（BillingRuleEditor 简化实现）
2. 建议修改项已补充（影响提示、排序、预览）
3. 风险等级：中（可控）
