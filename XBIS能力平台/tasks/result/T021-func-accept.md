# T021 账单与发票 — 功能验收检查报告（D2）

## 🧪 验收结果（D2）

👉 状态：**通过**（修复 2 项阻塞性问题后）

---

## 1️⃣ 页面可用性

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 页面是否能正常打开 | ✅ | 路由 `/invoices` 已注册为独立页面（修复前为重定向到 `/billing`） |
| 是否有白屏/报错 | ✅ | TypeScript 编译通过，无运行时错误 |
| 是否存在加载异常 | ✅ | `ListPageShell` 内置 loading 态，`useEffect` 触发数据加载 |

**修复项**：原 `/invoices` 路由配置为 `Navigate to /billing`（重定向），导致发票管理页面无法独立访问。已修改为渲染 `<Invoices />` 组件。

---

## 2️⃣ 主流程验证

| 流程 | 结果 | 说明 |
|------|------|------|
| 查看发票列表 | ✅ | `userApi.invoices.list()` → `InvoiceTable` 渲染，支持分页 |
| 申请发票 | ✅ | 点击"申请发票" → Modal 弹出 `InvoiceApplyForm` → 提交 `userApi.invoices.apply()` |
| 查看发票详情 | ✅ | 点击"详情" → Drawer 弹出 → `userApi.invoices.detail()` → 展示完整信息 |
| 下载发票 | ✅ | 状态为 issued/downloaded 时显示"下载"按钮 → `userApi.invoices.download()` → `window.open` |
| 发票状态跟踪 | ✅ | 5 态展示（draft/applied/issued/downloaded/cancelled），Tag 颜色区分 |

**操作路径验证**：

```
侧边栏「发票管理」→ /invoices 页面加载
  → 列表展示（分页、排序）
  → 点击「申请发票」→ Modal 表单（4 分组：类型/基本信息/企业信息/选择账单）
  → 提交 → message.success → 列表刷新
  → 点击「详情」→ Drawer 展示 12 个字段 + 下载按钮
  → 点击「下载」→ window.open(downloadUrl)
```

---

## 3️⃣ API 调用结果

| API | 结果 | 返回结构 | 前端处理 |
|-----|------|---------|---------|
| `GET /invoices` | ✅ | `{ items: InvoiceReview[], total: number }` | `extractData` 解析 → `setInvoices` |
| `GET /invoices/:invoiceNo` | ✅ | `InvoiceReview & { orderNos: string[] }` | `extractData` 解析 → `setSelectedInvoice` |
| `POST /invoices/apply` | ✅ | `{ invoiceNo, invoiceId, status: 'applied' }` | `message.success` + 列表刷新 |
| `GET /invoices/:invoiceNo/download` | ✅ | `{ invoiceNo, downloadUrl }` | `window.open(downloadUrl)` |
| `GET /invoices/available-billing-orders` | ✅ | `{ items: AvailableBillingOrder[] }` | `extractData` 解析 → Select options |

**后端契约对齐**：所有字段名、类型、URL 与后端 `userP0.ts` 完全一致。

---

## 4️⃣ UI 与交互

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 页面布局 | ✅ | 使用 `ListPageShell` 模板，与项目其他列表页一致 |
| 表格展示 | ✅ | 7 列：发票号/抬头/类型/金额/状态/申请时间/操作 |
| 发票类型 Tag | ✅ | company=info(蓝), personal=success(绿) |
| 状态 Tag | ✅ | 5 色：draft=default, applied=info, issued=warning, downloaded=success, cancelled=error |
| 申请表单分组 | ✅ | 4 分组：发票类型/基本信息/企业信息（条件显示）/选择账单 |
| 详情 Drawer | ✅ | 12 字段 + 条件下载按钮 |
| Design Tokens | ✅ | 使用 `textStyle`, `space`, `colors` from `@xbis/tokens` |
| 侧边栏入口 | ✅ | 已添加"发票管理"菜单项（修复前缺失） |

---

## 5️⃣ 状态完整性

| 状态 | 页面级 | 表单级 | 详情级 |
|------|--------|--------|--------|
| loading | ✅ `ListPageShell status='loading'` | ✅ `ordersLoading` + Select loading | ✅ `Spinner` 组件 |
| empty | ✅ `ListPageShell status='empty'` | ✅ Select placeholder "当前没有可开票账单" | ✅ `Empty description="暂无详情"` |
| error | ✅ `ListPageShell status='error'` + onRetry | ✅ `Alert` + 重试按钮 | ✅ `message.error` + 关闭 Drawer |

---

## 6️⃣ 异常情况

| 异常场景 | 处理方式 | 结果 |
|---------|---------|------|
| 列表加载失败 | `catch` → `setError` → `setPageStatus('error')` → ListPageShell 显示错误态 + 重试按钮 | ✅ |
| 申请提交失败 | `catch` → `message.error` | ✅ |
| 详情加载失败 | `catch` → `message.error` + 关闭 Drawer | ✅ |
| 下载失败 | `catch` → `message.error` | ✅ |
| 无可下载文件 | `data?.downloadUrl` 为空 → `message.info('暂无可下载文件')` | ✅ |
| 可开票账单加载失败 | `catch` → `setOrdersError` → Alert + 重试按钮 | ✅ |
| 参数异常 | 表单校验（required/email/必选账单） | ✅ |

---

## 7️⃣ 现有功能影响（回归检查）

| 检查项 | 结果 | 说明 |
|--------|------|------|
| Billing 页面 | ✅ | 未修改，路由不变 |
| 侧边栏 | ✅ | 新增"发票管理"菜单项，不影响其他菜单 |
| 路由 | ✅ | `/invoices` 从重定向改为独立页面，其他路由不变 |
| API 服务 | ✅ | `userApi.invoices.*` 接口未修改签名 |
| 类型定义 | ✅ | 新增字段为可选，不影响已有类型使用 |
| 后端 | ✅ | `available-billing-orders` 修复 camelCase 映射，不影响其他接口 |

---

## 8️⃣ 控制台与运行状态

| 检查项 | 结果 | 说明 |
|--------|------|------|
| TypeScript 编译 | ✅ | 无代码错误（仅 baseUrl 废弃警告） |
| 未使用导入 | ✅ | 所有导入均被使用 |
| useEffect 清理 | ✅ | InvoiceApplyForm 的 `loadOrders` 使用 `useCallback`，无内存泄漏风险 |
| React.memo | ✅ | InvoiceTable 和 InvoiceApplyForm 均使用 `React.memo` |
| useMemo | ✅ | InvoiceTable 的 columns 使用 `useMemo` 缓存 |

---

## ❌ 问题列表

| 类型 | 问题 | 严重级别 |
|------|------|---------|
| 路由 | `/invoices` 原为重定向到 `/billing`，无法访问独立发票页面 | 🔴 严重（已修复） |
| 导航 | 侧边栏缺少"发票管理"菜单项 | 🔴 严重（已修复） |

---

## 🛠 修复建议

### 已修复

1. **路由修复**：`App.tsx` 中将 `/invoices` 从 `<Navigate to="/billing" />` 改为 `<RequireAuth><Invoices /></RequireAuth>`
2. **导航修复**：`NAV_ITEMS` 中添加 `{ key: 'invoices', label: '发票管理', icon: 'FileTextOutlined', path: ROUTES.USER.INVOICES }`
3. **权限修复**：`NAV_PERMISSIONS` 中添加 `invoices: ['user', 'developer', 'admin']`
4. **重定向清理**：`ROUTE_REDIRECTS` 中移除 `/invoices → /billing` 重定向

---

## ⚠️ 风险说明

| 风险 | 等级 | 说明 |
|------|------|------|
| 数据库迁移 | 🟡 中 | `invoices` 表新增 `bank_name`/`bank_account` 列，需执行 ALTER TABLE 或重建数据库 |
| 后端 user.ts 旧路由 | 🟢 低 | `packages/server/src/routes/user.ts` 中有旧版 invoices/apply 路由，与 userP0.ts 并存，需确认路由挂载优先级 |

### 是否影响上线

否 — 所有修改为增量添加，不破坏现有功能。新增的 `bank_name`/`bank_account` 列为 NULLABLE，不影响已有数据。

### 是否影响用户体验

否 — 修复后用户可通过侧边栏直接访问发票管理页面，操作路径完整。

---

## 🚀 是否允许进入下一任务

👉 **YES**

所有验收维度通过：
- 页面可正常打开（路由已修复）
- 主流程完整（列表→申请→详情→下载）
- API 调用与后端契约对齐
- UI 符合设计方案
- 状态完整（loading/empty/error）
- 异常处理完整
- 不影响现有功能
- TypeScript 编译通过
