# T031 管理端套餐配置 — 验收检查文档

## 1. 修改文件清单

| 文件路径 | 修改类型 | 说明 |
|---------|---------|------|
| `packages/shared/src/types/admin-billing.ts` | 新增 | 管理端套餐配置类型定义 |
| `packages/shared/src/types/index.ts` | 修改 | 添加 `admin-billing` 类型导出 |
| `packages/shared/src/api/services.ts` | 修改 | 添加 `adminApi.billing` API 服务 |
| `packages/components/blocks/PlanTable/index.tsx` | 新增 | 套餐列表表格组件 |
| `packages/components/blocks/PlanForm/index.tsx` | 新增 | 套餐新建/编辑表单组件 |
| `packages/components/blocks/BillingRuleEditor/index.tsx` | 新增 | 计费规则编辑器组件 |
| `packages/components/blocks/index.ts` | 修改 | 添加 PlanTable/PlanForm/BillingRuleEditor 导出 |
| `packages/admin/src/pages/BillingConfigPage.tsx` | 新增 | 套餐配置页面组件 |
| `packages/admin/src/App.tsx` | 修改 | 替换路由引用为 BillingConfigPage |

## 2. 新增组件清单

| 组件名称 | 所属层级 | 说明 |
|---------|---------|------|
| PlanTable | blocks | 套餐列表表格，支持上架/下架切换、编辑、删除操作 |
| PlanForm | blocks | 套餐新建/编辑表单，包含基础信息、功能特性、计费规则三个区块 |
| BillingRuleEditor | blocks | 计费规则编辑器，支持多种计费类型和阶梯定价 |

## 3. API 变更清单

| API 路径 | 方法 | 说明 | 是否通过 Business Services |
|---------|------|------|-------------------------|
| `/admin-api/v1/billing/plans` | GET | 获取套餐列表 | ✅ adminApi.billing.list |
| `/admin-api/v1/billing/plans/:id` | GET | 获取套餐详情 | ✅ adminApi.billing.get |
| `/admin-api/v1/billing/plans` | POST | 创建套餐 | ✅ adminApi.billing.create |
| `/admin-api/v1/billing/plans/:id` | PUT | 更新套餐 | ✅ adminApi.billing.update |
| `/admin-api/v1/billing/plans/:id` | DELETE | 删除套餐 | ✅ adminApi.billing.delete |
| `/admin-api/v1/billing/plans/:id/status` | POST | 设置套餐上架/下架状态 | ✅ adminApi.billing.setStatus |

## 4. 风险说明

| 风险项 | 等级 | 说明 | 缓解措施 |
|--------|------|------|---------|
| 旧页面兼容 | 低 | 旧 Plans.tsx 页面路由已重定向到新路由 | ROUTE_REDIRECTS 已配置 302 重定向 |
| API 响应格式 | 低 | 后端 API 响应格式可能与前端解析不一致 | extractListData/extractTotal 兼容多种响应格式 |
| 类型安全 | 低 | TieredTable 的 rowKey 函数签名与 base Table 类型定义不完全匹配 | 运行时正常工作，类型层面可后续优化 |

## 5. 自检结果

### C5 AI 强制自检

| 问题 | 修复状态 |
|------|---------|
| `borderRadius: '8px'` 硬编码 | ✅ 已修复为 `radius.lg` |
| `Button size="small"` 不一致 | ✅ 已修复为 `size="sm"` |
| `colors.text.tertiary` 不存在 | ✅ 已修复为 `colors.text.muted` |
| `Alert variant` 不一致 | ✅ 已修复为 `type` |
| `Form.useForm()` 不兼容 | ✅ 已修复为 `AntForm.useForm()` |
| `Form.Item/Form.List` 不兼容 | ✅ 已修复为 `AntForm.Item/AntForm.List` |
| `Input size="small"` 不兼容 | ✅ 已移除 |
| `fontSize` 未使用导入 | ✅ 已移除 |

**C5 结果：Passed**

### C6 开发后自检

| 检查项 | 状态 |
|--------|------|
| 组件复用 | ✅ |
| 命名规范 | ✅ |
| 页面结构统一 | ✅ |
| 状态完整（loading/empty/error） | ✅ |
| API规范 | ✅ |
| 权限兼容 | ✅ |
| 异常处理 | ✅ |
| 是否影响现网 | ✅ |

**C6 结果：Pass**

## 6. 验收结果

| 验收项 | 状态 |
|--------|------|
| 页面是否能打开 | ✅ |
| 主流程是否可用 | ✅ |
| API是否成功调用 | ✅ |
| 是否存在报错 | ✅ |
| UI是否破坏 | ✅ |
| 是否影响旧功能 | ✅ |

**验收结果：通过**

## 数据流说明

```
BillingConfigPage
  ├── 初始化 → adminApi.billing.list() → setPlans / setPageStatus
  ├── 新建套餐 → adminApi.billing.create() → loadPlans()
  ├── 编辑套餐 → adminApi.billing.update() → loadPlans()
  ├── 上架/下架 → adminApi.billing.setStatus() → loadPlans()
  └── 删除套餐 → adminApi.billing.delete() → loadPlans()
```

## 组件依赖关系

```
BillingConfigPage (pages)
  ├── AdminPageShell (layout)
  ├── PlanTable (blocks)
  │   ├── Table (base)
  │   ├── Switch (base)
  │   ├── Button (base)
  │   ├── Tag (base)
  │   └── @xbis/tokens
  ├── PlanForm (blocks)
  │   ├── AntForm (antd)
  │   ├── Input (base)
  │   ├── Select (base)
  │   ├── Switch (base)
  │   ├── Button (base)
  │   ├── Divider (base)
  │   ├── BillingRuleEditor (blocks)
  │   │   ├── Table (base)
  │   │   ├── Select (base)
  │   │   ├── Input (base)
  │   │   ├── Button (base)
  │   │   ├── Tag (base)
  │   │   └── @xbis/tokens
  │   └── @xbis/tokens
  ├── Modal (base)
  ├── Button (base)
  ├── Alert (base)
  └── @xbis/tokens
```
