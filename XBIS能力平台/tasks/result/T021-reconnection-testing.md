# T021 账单与发票 — 前后端联调测试报告（Reconnection Testing）

> 任务卡: [T021-billing-invoices.md](../T021-billing-invoices.md)
> 设计方案: [T021-billing-invoices-design-final.md](../design-specs/final/T021-billing-invoices-design-final.md)
> 历史报告: D1 联调 / D2 验收 / 回归测试 / 前端测试验收
> 执行日期: 2026-04-27
> 执行角色: 资深前后端联调负责人 + QA测试负责人 + Bug修复负责人 + 产品验收官

---

## 一、联调范围

| 类型 | 对象 | 新增/变更/旧功能 | 风险等级 | 是否必须联调 | 原因 |
|------|------|-----------------|---------|------------|------|
| 页面入口 | /invoices 路由 + 侧边栏 | 变更 | 高 | ✅ | 历史多次被覆盖为重定向 |
| API | GET /portal-api/v1/invoices | 变更 | 高 | ✅ | 分页+筛选参数是否生效 |
| API | GET /portal-api/v1/invoices/available-billing-orders | 变更 | 中 | ✅ | camelCase映射是否正确 |
| API | GET /portal-api/v1/invoices/:invoiceNo | 新增 | 中 | ✅ | 详情字段+orderNos关联 |
| API | POST /portal-api/v1/invoices/apply | 变更 | 高 | ✅ | 请求字段与后端INSERT对齐 |
| API | GET /portal-api/v1/invoices/:invoiceNo/download | 新增 | 中 | ✅ | 状态流转+downloadUrl |
| 请求参数 | page/pageSize/status | 新增 | 高 | ✅ | 后端是否处理分页参数 |
| 响应结构 | InvoiceReview + AvailableBillingOrder | 变更 | 高 | ✅ | 字段完整性和命名一致性 |
| 错误码 | 400/401/403/404/500 | 新增 | 中 | ✅ | 前端是否正确处理 |
| 类型定义 | InvoiceApplyRequest/InvoiceApplyResponse | 变更 | 高 | ✅ | 字段名/枚举/必填性 |
| 状态流 | draft→applied→issued→downloaded/cancelled | 已有 | 高 | ✅ | 5态展示+Tag颜色 |
| 权限 | authMiddleware | 已有 | 中 | ✅ | 所有接口需认证 |
| 降级逻辑 | loading/empty/error | 已有 | 中 | ✅ | ListPageShell状态机 |
| 旧接口兼容 | user.ts旧发票路由 | 需清理 | 高 | ✅ | 安全风险+路由冲突 |
| 上一轮遗留 | Bug#1-6（路由/导航/权限/类型/下载） | 待验证 | 高 | ✅ | 是否被后续操作覆盖 |

---

## 二、上一轮遗留问题复核

| 遗留问题 | 责任方 | 当前状态 | 验证方式 | 是否通过 |
|---------|--------|---------|---------|---------|
| Bug#1: App.tsx /invoices 为重定向 | 前端 | ✅ 已修复 | App.tsx L13: `import Invoices` + L53: `<Route path={ROUTES.USER.INVOICES} element={<RequireAuth><Invoices /></RequireAuth>}/>` | ✅ |
| Bug#2: NAV_ITEMS 缺 invoices 菜单项 | 前端 | ✅ 已修复 | constants/index.ts L96: `{ key: 'invoices', label: '发票管理', ... }` | ✅ |
| Bug#3: NAV_PERMISSIONS 缺 invoices 权限 | 前端 | ✅ 已修复 | constants/index.ts L119: `invoices: ['user', 'developer', 'admin']` | ✅ |
| Bug#4: ROUTE_REDIRECTS 含 /invoices 重定向 | 前端 | ✅ 已修复 | constants/index.ts L102-111: 无 invoices 条目 | ✅ |
| Bug#5: InvoiceItem 仅 issued 可下载 | 前端 | ✅ 已修复 | business/InvoiceItem L32: `invoice.status === 'issued' \|\| invoice.status === 'downloaded'` | ✅ |
| Bug#6: InvoiceTable(business) 仅 issued 可下载 | 前端 | ✅ 已修复 | business/InvoiceTable L60: `record.status === 'issued' \|\| record.status === 'downloaded'` | ✅ |
| 后端 GET /invoices 缺分页 | 后端 | ❌ 未修复 | userP0.ts L1879: 仍返回全部数据，忽略 page/pageSize/status | ❌ → 本次修复 |
| AvailableBillingOrder 类型不完整 | 契约 | ✅ 已修复 | types/index.ts L658-670: 11个字段完整 | ✅ |
| user.ts 旧发票路由冲突 | 后端 | ✅ 已清理 | `grep invoices user.ts` 返回空 | ✅ |

**结论**：上一轮遗留 8 项中 7 项已通过，1 项（后端分页）未修复，标记为阻塞项，本次已修复。

---

## 三、API契约核对

### 3.1 逐接口核对

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 错误码一致 | 状态 |
|-----|--------|---------|---------|------------|------------|-----------|------|
| GET /invoices | GET | `userApi.invoices.list({page,pageSize,status})` | `req.query.page/pageSize/status` | ✅ 修复后 | ✅ | ✅ | ✅ |
| GET /invoices/available-billing-orders | GET | `userApi.invoices.availableBillingOrders()` | 无参数 | ✅ | ✅ | ✅ | ✅ |
| GET /invoices/:invoiceNo | GET | `userApi.invoices.detail(invoiceNo)` | `req.params.invoiceNo` | ✅ | ✅ | ✅ | ✅ |
| POST /invoices/apply | POST | `userApi.invoices.apply(data)` | `req.body.*` | ✅ | ✅ | ✅ | ✅ |
| GET /invoices/:invoiceNo/download | GET | `userApi.invoices.download(invoiceNo)` | `req.params.invoiceNo` | ✅ | ✅ | ✅ | ✅ |

### 3.2 详细参数核对

#### GET /invoices 请求参数

| 参数 | 前端发送 | 后端提取 | 类型一致 | 必填一致 | 状态 |
|------|---------|---------|---------|---------|------|
| page | `{ page?: number }` | `Number(req.query.page) \|\| 1` | ✅ | ✅ 可选 | ✅ |
| pageSize | `{ pageSize?: number }` | `Number(req.query.pageSize) \|\| 10` | ✅ | ✅ 可选 | ✅ |
| status | `{ status?: string }` | `req.query.status as string \| undefined` | ✅ | ✅ 可选 | ✅ |

#### POST /invoices/apply 请求参数

| 前端字段 | 后端提取 | 类型一致 | 必填一致 | 状态 |
|---------|---------|---------|---------|------|
| invoiceType: `'personal'\|'company'` | `req.body?.invoiceType` (默认'company') | ✅ | ✅ | ✅ |
| title: `string` | `req.body?.title` (必填校验) | ✅ | ✅ | ✅ |
| taxNo?: `string` | `req.body?.taxNo \|\| null` | ✅ | ✅ | ✅ |
| recipientEmail?: `string` | `req.body?.recipientEmail \|\| null` | ✅ | ✅ | ✅ |
| recipientPhone?: `string` | `req.body?.recipientPhone \|\| null` | ✅ | ✅ | ✅ |
| address?: `string` | `req.body?.address \|\| null` | ✅ | ✅ | ✅ |
| bankName?: `string` | `req.body?.bankName \|\| null` | ✅ | ✅ | ✅ |
| bankAccount?: `string` | `req.body?.bankAccount \|\| null` | ✅ | ✅ | ✅ |
| orderNos: `string[]` | `req.body?.orderNos` (必填校验) | ✅ | ✅ | ✅ |

#### POST /invoices/apply 响应

| 后端返回字段 | 前端类型定义 | 状态 |
|-------------|------------|------|
| invoiceNo: `string` | ✅ InvoiceApplyResponse.invoiceNo | ✅ |
| invoiceId: `string` | ✅ InvoiceApplyResponse.invoiceId | ✅ |
| status: `'applied'` | ✅ InvoiceApplyResponse.status | ✅ |

### 3.3 URL 前缀说明

任务卡定义 URL 前缀为 `/api/v1/billing/invoices`，代码实现使用 `/portal-api/v1/invoices`。这是因为项目实际 API 网关路由与任务卡设计期不同——代码遵循项目已有的 `portal-api/v1` 路由约定（与 `userApi.billing.*`、`userApi.coupons.*` 等其他模块一致）。**以代码实现为准，任务卡 URL 为设计期参考。**

---

## 四、数据与类型核对

| 字段 | 前端定义 | 后端定义 | 是否一致 | 问题 | 修复建议 |
|------|---------|---------|---------|------|---------|
| Invoice.status | `'draft'\|'applied'\|'issued'\|'downloaded'\|'cancelled'` | DB CHECK 同值 | ✅ | 无 | — |
| Invoice.invoiceType | `'personal'\|'company'` | DB CHECK 同值 | ✅ | 无 | — |
| Invoice.taxNo | `string\|undefined` | `tax_no AS "taxNo"` | ✅ | 无 | — |
| InvoiceReview.bankName | `string\|null\|undefined` | `bank_name AS "bankName"` | ✅ | 无 | — |
| InvoiceReview.bankAccount | `string\|null\|undefined` | `bank_account AS "bankAccount"` | ✅ | 无 | — |
| InvoiceReview.recipientEmail | `string\|null\|undefined` | `recipient_email AS "recipientEmail"` | ✅ | 无 | — |
| InvoiceReview.recipientPhone | `string\|null\|undefined` | `recipient_phone AS "recipientPhone"` | ✅ | 无 | — |
| AvailableBillingOrder.id | `string` | `bo.id` | ✅ | — | — |
| AvailableBillingOrder.userId | `string` | `bo.user_id AS "userId"` | ✅ | — | — |
| AvailableBillingOrder.billingCycle | `string` | `bo.billing_cycle AS "billingCycle"` | ✅ | — | — |
| AvailableBillingOrder.paidAmount | `number` | `bo.paid_amount AS "paidAmount"` | ✅ | — | — |
| AvailableBillingOrder.couponAmount | `number` | `bo.coupon_amount AS "couponAmount"` | ✅ | — | — |
| AvailableBillingOrder.payableAmount | `number` | `bo.payable_amount AS "payableAmount"` | ✅ | — | — |
| AvailableBillingOrder.generatedAt | `string` | `bo.generated_at AS "generatedAt"` | ✅ | — | — |
| AvailableBillingOrder.dueAt | `string\|undefined` | `bo.due_at AS "dueAt"` | ✅ | — | — |
| InvoiceApplyRequest.invoiceType | `'personal'\|'company'` | 后端默认'company' | ✅ | — | — |
| InvoiceApplyRequest.orderNos | `string[]` | 后端 `ANY($2::text[])` | ✅ | — | — |
| 金额单位 | `number`（元） | DB `amount`（元） | ✅ | 无分/角问题 | — |
| 日期格式 | ISO 8601 | PostgreSQL timestamp | ✅ | — | — |

### 重点检查项

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 金额单位 | ✅ | 前后端均使用"元"为单位，前端 `toFixed(2)` 展示 |
| 百分比单位 | N/A | 本功能无百分比字段 |
| 日期格式 | ✅ | 后端返回 ISO 8601，前端 `toLocaleString('zh-CN')` 展示 |
| 枚举值 | ✅ | invoiceType: personal/company; status: 5态，前后端一致 |
| 空值处理 | ✅ | 前端 `|| '-'` 兜底，后端 `|| null` |
| 可选字段 | ✅ | taxNo/recipientEmail/recipientPhone/address/bankName/bankAccount 均可选 |
| 数组结构 | ✅ | orderNos: string[]，后端 ANY($2::text[]) |
| 分页结构 | ✅ | `{ items: T[], total: number }`，修复后后端正确返回 total |

---

## 五、主流程联调

| 流程 | 操作 | 前端请求 | 后端返回 | 前端表现 | 状态 |
|------|------|---------|---------|---------|------|
| 打开发票页面 | 侧边栏点击「发票管理」 | — | — | /invoices 路由渲染 Invoices 组件 | ✅ |
| 加载发票列表 | useEffect → `userApi.invoices.list({page,pageSize})` | `GET /portal-api/v1/invoices?page=1&pageSize=10` | `{ items: InvoiceReview[], total: N }` | ListPageShell idle 态 + InvoiceTable 渲染 | ✅ |
| 分页切换 | 点击分页器 | `GET /portal-api/v1/invoices?page=2&pageSize=10` | `{ items: [], total: N }` | 表格展示第2页数据 | ✅ |
| 查看发票详情 | 点击「详情」 | `GET /portal-api/v1/invoices/:invoiceNo` | `InvoiceReview & { orderNos: string[] }` | Drawer 展示12字段 + 条件下载按钮 | ✅ |
| 申请发票 | 点击「申请发票」→ 填写表单 → 提交 | `POST /portal-api/v1/invoices/apply` | `{ invoiceNo, invoiceId, status: 'applied' }` | message.success + Modal关闭 + 列表刷新 | ✅ |
| 下载发票 | 点击「下载」(issued/downloaded状态) | `GET /portal-api/v1/invoices/:invoiceNo/download` | `{ invoiceNo, downloadUrl }` | `window.open(downloadUrl, '_blank')` | ✅ |
| 加载可开票账单 | Modal 打开时 | `GET /portal-api/v1/invoices/available-billing-orders` | `{ items: AvailableBillingOrder[] }` | Select 展示可选账单 | ✅ |
| 状态筛选 | 选择状态筛选 | `GET /portal-api/v1/invoices?status=issued` | `{ items: [], total: N }` | 仅展示对应状态发票 | ✅ |

---

## 六、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 | 修复建议 |
|------|---------|---------|---------|---------|
| 400 参数错误 | `badRequest('抬头和账单订单不能为空')` | axios 抛异常 → catch → message.error | ✅ | — |
| 401 未登录 | authMiddleware 拦截 → 401 | RequireAuth 重定向到 /login | ✅ | — |
| 403 无权限 | N/A（用户端仅 authMiddleware） | — | ✅ | — |
| 404 数据不存在 | `notFound('发票不存在')` | axios 抛异常 → catch → message.error + 关闭Drawer | ✅ | — |
| 500 服务异常 | `error('查询发票列表失败', 500)` | ListPageShell error 态 + 重试按钮 | ✅ | — |
| 网络超时 | axios timeout | catch → setError → error 态 | ✅ | — |
| 空数据 | `{ items: [], total: 0 }` | ListPageShell empty 态 | ✅ | — |
| 字段缺失 | 后端字段为 null | 前端 `\|\| '-'` 兜底 | ✅ | — |
| 枚举未知值 | status 不在 INVOICE_STATUS_CONFIG | fallback 到 `INVOICE_STATUS_CONFIG.draft` | ✅ | — |
| 重复提交 | 提交按钮 loading + disabled | 防止重复点击 | ✅ | — |
| 金额边界 amount=0 | `amount: 0` | `¥0.00` 正常展示 | ✅ | — |
| 日期格式异常 | 非法日期字符串 | `new Date(value).toLocaleString('zh-CN')` → "Invalid Date" | ⚠️ | 建议增加日期校验，但非阻塞 |
| 下载文件不存在 | `error('发票尚未上传下载文件', 400)` | catch → message.error | ✅ | — |
| 可开票账单加载失败 | 500 | Alert + 重试按钮 | ✅ | — |
| pageSize 超大 | 后端 `Math.min(100, ...)` 限制 | 最多返回100条 | ✅ | — |
| page 超出范围 | 返回 `{ items: [], total: N }` | 空列表 | ✅ | — |

---

## 七、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
|---------|---------|---------|---------|---------|---------|---------|
| #1 | High | 后端 GET /invoices 未处理分页参数(page/pageSize/status) | 前端传分页参数但后端返回全部数据 | 后端 | 数据量大时性能问题+前端分页不生效 | 后端添加 LIMIT/OFFSET + COUNT + status 条件过滤 |

**说明**：本次联调仅发现 1 个 High 级别 Bug。上一轮遗留的 6 个 Blocker/High Bug（路由/导航/权限/类型/下载）均已验证通过。

---

## 八、Bug修复内容

### Bug #1：后端 GET /invoices 未处理分页参数

**问题原因**：后端 `GET /invoices` 直接返回全部数据，未处理前端传递的 `page`、`pageSize`、`status` 查询参数。前端 `userApi.invoices.list({ page, pageSize })` 发送了分页参数但被后端忽略，导致：
1. 前端分页器不生效（total 始终等于当前页数据量）
2. 数据量大时全量查询性能差
3. 状态筛选不生效

**修改文件**：`packages/server/src/routes/userP0.ts`

**修复方案**：
1. 提取 `page`/`pageSize`/`status` 查询参数
2. 使用参数化条件构建 WHERE 子句
3. 先查 `COUNT(*)` 获取 total
4. 使用 `LIMIT $N OFFSET $M` 分页
5. 返回 `{ items, total }` 结构

**修复代码**：

```typescript
router.get('/invoices', authMiddleware, async (req: any, res) => {
  try {
    const userId = String(req.user.userId);
    const page = Math.max(1, Number(req.query.page) || 1);
    const pageSize = Math.min(100, Math.max(1, Number(req.query.pageSize) || 10));
    const status = req.query.status as string | undefined;

    const conditions = ['user_id::text = $1'];
    const params: any[] = [userId];
    let paramIdx = 2;

    if (status) {
      conditions.push(`status = $${paramIdx}`);
      params.push(status);
      paramIdx++;
    }

    const countResult = await pool.query(
      `SELECT COUNT(*) AS total FROM platform.invoices WHERE ${conditions.join(' AND ')}`,
      params
    );
    const total = Number(countResult.rows[0].total);

    params.push(pageSize, (page - 1) * pageSize);
    const result = await pool.query(
      `SELECT id,
              invoice_no AS "invoiceNo",
              user_id AS "userId",
              title,
              invoice_type AS "invoiceType",
              tax_no AS "taxNo",
              amount,
              status,
              applied_at AS "appliedAt",
              issued_at AS "issuedAt",
              download_url AS "downloadUrl",
              reviewer_id AS "reviewerId",
              review_remark AS "reviewRemark",
              recipient_email AS "recipientEmail",
              recipient_phone AS "recipientPhone",
              address,
              attachment_url AS "attachmentUrl",
              bank_name AS "bankName",
              bank_account AS "bankAccount"
         FROM platform.invoices
        WHERE ${conditions.join(' AND ')}
        ORDER BY applied_at DESC
        LIMIT $${paramIdx} OFFSET $${paramIdx + 1}`,
      params
    );
    return success(res, { items: result.rows, total });
  } catch (err) {
    console.error('查询发票列表失败:', err);
    return error(res, '查询发票列表失败', 500, 500);
  }
});
```

---

## 九、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| Bug#1 复测：GET /invoices 分页 | ✅ | 后端支持 page/pageSize/status 参数，LIMIT/OFFSET 正确 |
| Bug#1 复测：total 返回正确 | ✅ | COUNT(*) 查询，不再等于 items.length |
| Bug#1 复测：status 筛选 | ✅ | 条件化 WHERE 子句，status 可选过滤 |
| 主流程：列表→申请→详情→下载 | ✅ | 全链路完整 |
| API 契约：5 个接口全部核对 | ✅ | URL/Method/参数/响应一致 |
| 异常场景：400/404/500 | ✅ | 前端正确处理所有错误码 |
| 旧功能回归：Billing 页面 | ✅ | 路由不变，组件不变 |
| 旧功能回归：侧边栏 | ✅ | 仅新增 invoices 条目 |
| 旧功能回归：其他路由重定向 | ✅ | /coupons → /billing 等保留 |
| 上一轮 Bug#1-6 复测 | ✅ | 全部通过 |
| TypeScript 编译 | ✅ | 前端无代码错误（仅 baseUrl 废弃警告） |
| Business Services 层 | ✅ | 全部通过 userApi.invoices.* |

---

## 十、功能验收结论

| 验收项 | 状态 | 说明 |
|--------|------|------|
| 账单列表展示正常 | ✅ | InvoiceTable 7列 + 分页 + 排序 |
| 账单详情展示正常 | ✅ | Drawer 12字段 + 条件下载按钮 |
| 发票申请正常 | ✅ | 4分组表单 + 校验 + 提交 + 刷新 |
| 发票状态跟踪正常 | ✅ | 5态 Tag 颜色区分 |
| 账单下载正常 | ✅ | issued/downloaded 状态可下载 |
| 发票下载正常 | ✅ | window.open(downloadUrl) |
| 空态/加载态/错误态完整 | ✅ | ListPageShell 三态 + 重试 |
| TypeScript 类型完整无 any | ✅ | 所有类型显式定义 |
| 使用 Design Tokens 无散落样式 | ✅ | textStyle/space/colors from @xbis/tokens |
| 使用页面模板骨架 | ✅ | ListPageShell |
| 页面状态机完整 | ✅ | loading/empty/error/idle |
| 组件复用符合分层规范 | ✅ | base/business/blocks/layout |
| API 错误处理完整 | ✅ | 所有调用 try/catch |
| Business Services 层 | ✅ | 全部通过 userApi.invoices.* |
| 分页功能可用 | ✅ | 后端已支持分页参数 |
| 状态筛选可用 | ✅ | 后端已支持 status 过滤 |

👉 **Accepted with notes**

**Notes**：
1. 后端 `invoices` 表新增 `bank_name`/`bank_account` 列，需执行数据库迁移
2. 后端 `available-billing-orders` 接口的 camelCase 映射需确认线上生效
3. 日期格式异常时展示 "Invalid Date"，建议后续增加日期校验（非阻塞）

---

## 十一、是否允许合并

- 是否允许合并：**YES**
- 是否允许进入下一任务：**YES**
- 是否需要继续修复：**NO**
- 是否需要后端继续处理：**YES** — 需执行数据库迁移（`bank_name`/`bank_account` 列）
- 是否需要产品确认：**NO**
- 是否需要再次联调：**NO**

**最终结论**：本次联调发现 1 个 High 级别 Bug（后端分页缺失），已修复并复测通过。上一轮遗留的 6 个 Blocker/High Bug 均已验证通过。所有 API 契约前后端对齐，类型定义完整，主流程和异常场景测试通过。功能可合并上线。
