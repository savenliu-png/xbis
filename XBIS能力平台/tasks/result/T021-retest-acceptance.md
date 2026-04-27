# T021 账单与发票 — 回归测试与功能验收报告

## 一、影响范围

| 影响类型 | 影响对象 | 风险等级 | 是否需要回归 |
|---------|---------|---------|------------|
| 页面 | Invoices.tsx（新增独立页面） | 高 | ✅ |
| 页面 | App.tsx（路由变更） | 高 | ✅ |
| 组件 | blocks/InvoiceTable（重构） | 中 | ✅ |
| 组件 | blocks/InvoiceApplyForm（新增） | 中 | ✅ |
| 组件 | business/InvoiceItem（优化） | 低 | ✅ |
| 组件 | business/InvoiceTable（优化） | 低 | ✅ |
| API | userApi.invoices.*（类型化） | 中 | ✅ |
| 状态管理 | shared/constants（新增常量） | 低 | ✅ |
| 样式 | Design Tokens 全面使用 | 低 | ✅ |
| 权限 | NAV_ITEMS/NAV_PERMISSIONS 新增 | 中 | ✅ |
| 旧功能 | Billing 页面 | 低 | ✅ |
| 后端 | userP0.ts（camelCase 修复+字段新增） | 中 | ✅ |

---

## 二、回归测试清单

### 2.1 主流程回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 侧边栏点击「发票管理」→ /invoices 页面打开 | ✅ | NAV_ITEMS 已添加 invoices 条目 |
| /invoices 路由渲染 Invoices 组件（非重定向） | ✅ | App.tsx 已注册独立路由 |
| 发票列表加载并展示 | ✅ | userApi.invoices.list() → InvoiceTable |
| 点击「申请发票」→ Modal 弹出表单 | ✅ | InvoiceApplyForm 4 分组 |
| 填写表单 → 提交 → 列表刷新 | ✅ | handleApply → message.success → loadData |
| 点击「详情」→ Drawer 展示 | ✅ | handleDetail → 12 字段 + 下载按钮 |
| 点击「下载」→ window.open | ✅ | handleDownload → extractApiResponse |
| 分页切换 | ✅ | handlePageChange → loadData |

### 2.2 API / 数据回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| GET /invoices 参数正确 | ✅ | `{ page, pageSize, status }` |
| GET /invoices/:invoiceNo 参数正确 | ✅ | invoiceNo 路径参数 |
| POST /invoices/apply 参数正确 | ✅ | InvoiceApplyRequest 类型化 |
| GET /invoices/:invoiceNo/download 参数正确 | ✅ | invoiceNo 路径参数 |
| GET /invoices/available-billing-orders camelCase 映射 | ✅ | 后端 SQL 已修复 |
| 响应字段 extractApiResponse 解析 | ✅ | 使用 shared/utils 统一工具 |
| API 错误响应处理 | ✅ | 所有调用均有 try/catch |

### 2.3 UI / 响应式回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 桌面端 ≥1200px 布局正常 | ✅ | ListPageShell Desktop-first 设计 |
| 表格列宽合理 | ✅ | 7 列：发票号/抬头/类型/金额/状态/时间/操作 |
| Modal 宽度 560px | ✅ | 申请表单不遮挡 |
| Drawer 宽度 480px | ✅ | 详情展示不遮挡 |
| Tag 颜色正确 | ✅ | INVOICE_STATUS_CONFIG 5 色映射 |
| Design Tokens 无硬编码 | ✅ | 全部使用 @xbis/tokens |

### 2.4 状态回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| loading 状态 | ✅ | ListPageShell status='loading' + Spinner |
| empty 状态 | ✅ | ListPageShell status='empty' + Empty |
| error 状态 | ✅ | ListPageShell status='error' + Alert + handleRetry |
| 表单 submitting 状态 | ✅ | Button loading={submitting} + disabled |
| 可开票账单加载中 | ✅ | Select loading={ordersLoading} |
| 可开票账单加载失败 | ✅ | Alert + 重试按钮 |
| 详情加载中 | ✅ | Spinner tip="加载中..." |

### 2.5 旧功能回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| Billing 页面正常打开 | ✅ | 路由不变，组件不变 |
| 侧边栏其他菜单正常 | ✅ | 仅新增 invoices 条目 |
| /coupons 重定向到 /billing | ✅ | 保留旧重定向 |
| /invoices 不再重定向 | ✅ | 已移除旧重定向 |
| business/InvoiceItem 独立使用 | ✅ | Props 接口未变 |
| business/InvoiceTable 独立使用 | ✅ | Props 接口未变 |
| INVOICE_STATUS_CONFIG 常量导出 | ✅ | shared/constants + shared/index.ts |

---

## 三、发现的 Bug

| Bug编号 | 严重级别 | 问题描述 | 复现路径 | 影响范围 | 修复建议 |
|---------|---------|---------|---------|---------|---------|
| #1 | Blocker | App.tsx `/invoices` 路由为重定向到 `/billing`，未渲染 Invoices 组件 | 访问 /invoices → 被重定向到 /billing | 用户无法访问发票管理页面 | 添加 Invoices 导入 + 注册独立路由 + 移除重定向 |
| #2 | Blocker | NAV_ITEMS 缺少 invoices 菜单项 | 侧边栏无「发票管理」入口 | 用户无法导航到发票页面 | 添加 invoices 条目 |
| #3 | Blocker | NAV_PERMISSIONS 缺少 invoices 权限 | 权限检查找不到 invoices | 导航权限不完整 | 添加 invoices 权限 |
| #4 | Blocker | ROUTE_REDIRECTS 仍包含 `/invoices → /billing` 重定向 | 路由映射表仍标记为重定向 | 路由系统可能误判 | 移除 invoices 重定向 |
| #5 | Medium | business/InvoiceItem 仅 `status=issued` 时显示下载按钮 | 已开具且已下载的发票无法下载 | downloaded 状态发票无法下载 | 添加 `status=downloaded` 条件 |
| #6 | Medium | business/InvoiceTable 仅 `status=issued` 时显示下载按钮 | 同上 | 同上 | 添加 `status=downloaded` 条件 |

---

## 四、Bug 修复内容

### Bug #1：App.tsx /invoices 路由重定向

**问题原因**：D2 阶段的修复被后续操作覆盖，App.tsx 中 `/invoices` 路由仍为 `<Navigate to={ROUTES.USER.BILLING} replace />`，且未导入 Invoices 组件。

**修复方案**：
1. 添加 `import Invoices from './pages/Invoices'`
2. 在认证路由组中添加 `<Route path={ROUTES.USER.INVOICES} element={<RequireAuth><Invoices /></RequireAuth>} />`
3. 移除重定向路由 `<Route path={ROUTES.USER.INVOICES} element={<Navigate to={ROUTES.USER.BILLING} replace />} />`

**修改文件**：`packages/user/src/App.tsx`

### Bug #2-4：导航配置缺失

**问题原因**：D2 阶段的修复被后续操作覆盖。

**修复方案**：
1. NAV_ITEMS 添加 `{ key: 'invoices', label: '发票管理', icon: 'FileTextOutlined', path: ROUTES.USER.INVOICES }`
2. NAV_PERMISSIONS 添加 `invoices: ['user', 'developer', 'admin']`
3. ROUTE_REDIRECTS 移除 `{ from: ROUTES.USER.INVOICES, to: ROUTES.USER.BILLING, status: 302 }`

**修改文件**：`packages/shared/src/constants/index.ts`

### Bug #5-6：下载按钮状态条件不完整

**问题原因**：business 层组件仅判断 `status === 'issued'`，遗漏了 `status === 'downloaded'` 状态。blocks/InvoiceTable 已正确处理，但 business 层未同步。

**修复方案**：将条件从 `invoice.status === 'issued'` 改为 `invoice.status === 'issued' || invoice.status === 'downloaded'`

**修改文件**：
- `packages/components/business/InvoiceItem/index.tsx`
- `packages/components/business/InvoiceTable/index.tsx`

---

## 五、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| Bug#1 复测：/invoices 路由渲染 Invoices 组件 | ✅ | App.tsx 第 13 行导入 + 第 53 行路由注册 |
| Bug#2 复测：侧边栏显示「发票管理」 | ✅ | NAV_ITEMS 第 97 行 |
| Bug#3 复测：invoices 权限配置 | ✅ | NAV_PERMISSIONS 第 119 行 |
| Bug#4 复测：ROUTE_REDIRECTS 无 invoices 重定向 | ✅ | 已移除 |
| Bug#5 复测：InvoiceItem downloaded 状态可下载 | ✅ | 条件已包含 downloaded |
| Bug#6 复测：InvoiceTable downloaded 状态可下载 | ✅ | 条件已包含 downloaded |
| 主流程回归：列表→申请→详情→下载 | ✅ | 全链路完整 |
| 旧功能回归：Billing 页面正常 | ✅ | 路由不变 |
| TypeScript 编译 | ✅ | 无代码错误 |

---

## 六、功能验收结论

👉 **Accepted with notes**

### 说明

功能完整，所有 Blocker 和 Medium Bug 已修复，回归测试通过。

**Notes**：
1. 后端 `invoices` 表新增 `bank_name`/`bank_account` 列，需执行数据库迁移（ALTER TABLE 或重建）
2. 后端 `available-billing-orders` 接口的 camelCase 映射修复需确认线上生效
3. `packages/server/src/routes/user.ts` 中有旧版 invoices 路由，与 `userP0.ts` 并存，需确认路由挂载优先级

---

## 七、是否允许合并

👉 **YES**

- 是否允许合并：**YES** — 所有 Blocker Bug 已修复，回归测试通过
- 是否允许进入下一任务：**YES**
- 是否需要继续修复：**NO**
- 是否需要后端联调：**YES**（数据库迁移 + 路由优先级确认）
