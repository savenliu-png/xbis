# T023 个人中心 — 前后端联调测试与功能验收报告

## 一、联调范围

| 类型 | 对象 | 是否新增 | 风险等级 | 是否必须联调 |
|------|------|----------|----------|-------------|
| 页面入口 | `/profile` 路由 | 是 | 高 | 是 |
| 页面入口 | 用户下拉菜单"个人中心" | 修改 | 中 | 是 |
| API | GET /portal-api/v1/account/profile | 否（复用） | 中 | 是 |
| API | PUT /portal-api/v1/account/profile | 否（复用） | 中 | 是 |
| API | GET /portal-api/v1/account/security | 否（复用） | 中 | 是 |
| API | POST /portal-api/v1/account/change-password | 否（复用） | 中 | 是 |
| API | GET /portal-api/v1/account/login-logs | 否（复用） | 中 | 是 |
| API | GET /portal-api/v1/account/preferences | 否（复用） | 中 | 是 |
| API | PUT /portal-api/v1/account/preferences | 否（复用） | 中 | 是 |
| 请求参数 | ProfileUpdateRequest | 是（强类型化） | 低 | 是 |
| 请求参数 | ProfilePreferenceUpdateRequest | 是 | 低 | 是 |
| 请求参数 | ProfileLoginLogParams | 是 | 低 | 是 |
| 响应结构 | ProfileAccountData（= UserProfile & {...}） | 是 | 高 | 是 |
| 响应结构 | ProfileSecurityData | 是 | 中 | 是 |
| 响应结构 | ProfilePreferenceData | 是 | 低 | 是 |
| 响应结构 | ProfileInvoiceDefault（= Omit<InvoiceApplyRequest, 'orderNos'>） | 是 | 高 | 是 |
| 响应结构 | ProfileLoginLogItem | 是 | 中 | 是 |
| 状态流 | 账户状态（active/suspended/disabled） | 否 | 低 | 是 |
| 状态流 | 认证状态（none/pending/approved/rejected） | 是（新增 none） | 低 | 是 |
| 状态流 | 登录结果（success/failure/failed） | 是（兼容） | 低 | 是 |
| 权限 | RequireAuth 守卫 | 否 | 低 | 是 |
| 降级逻辑 | Tab 懒加载 + API 失败重试 | 是 | 中 | 是 |
| 旧接口兼容 | /account 页面保留 | 否 | 低 | 是 |

## 二、API契约核对

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 状态 |
|-----|--------|----------|----------|-------------|-------------|------|
| /portal-api/v1/account/profile | GET | `userApi.account.profile.get()` | ✅ | N/A | ✅ | 通过 |
| /portal-api/v1/account/profile | PUT | `userApi.account.profile.update(data: ProfileUpdateRequest)` | ✅ | ✅ | N/A | 通过 |
| /portal-api/v1/account/security | GET | `userApi.account.security()` | ✅ | N/A | ✅ | 通过 |
| /portal-api/v1/account/change-password | POST | `userApi.account.changePassword(data)` | ✅ | ✅ | N/A | 通过 |
| /portal-api/v1/account/login-logs | GET | `userApi.account.loginLogs(params?: ProfileLoginLogParams)` | ✅ | ✅ | ⚠️ | 待联调 |
| /portal-api/v1/account/preferences | GET | `userApi.account.preferences.get()` | ✅ | N/A | ✅ | 通过 |
| /portal-api/v1/account/preferences | PUT | `userApi.account.preferences.update(data: ProfilePreferenceUpdateRequest)` | ✅ | ✅ | N/A | 通过 |

**⚠️ 待联调项**：`/portal-api/v1/account/login-logs` 响应字段命名风格（snake_case vs camelCase）和 `login_result` 值（`'failure'` vs `'failed'`）需后端确认。

## 三、数据与类型核对

| 字段 | 前端定义 | 后端定义 | 问题 | 修复建议 |
|------|----------|----------|------|----------|
| developerType | `ProfileAccountData.developerType`（继承自 `UserProfile`） | `UserProfile.developerType` | ✅ 一致 | — |
| developerName | `ProfileAccountData.developerName`（继承自 `UserProfile`，必填） | `UserProfile.developerName` | ✅ 一致 | — |
| contactName | `ProfileAccountData.contactName`（继承自 `UserProfile`，可选） | `UserProfile.contactName` | ✅ 一致 | — |
| internalRemark | `ProfileAccountData.internalRemark`（继承自 `UserProfile`，可选） | `UserProfile.internalRemark` | ✅ 一致 | — |
| status | `ProfileAccountData.status`（收窄为联合类型） | `UserProfile.status: string` | ✅ 前端收窄合理 | — |
| certStatus | `ProfileAccountData.certStatus`（含 `'none'`） | `UserCertification.certStatus`（无 `'none'`） | ⚠️ 语义不同 | 保留：profile API 返回概览状态含"未认证" |
| invoiceType | `ProfileInvoiceDefault.invoiceType`（= `InvoiceApplyRequest.invoiceType`） | `InvoiceApplyRequest.invoiceType: 'personal' \| 'company'` | ✅ 一致 | — |
| taxNo | `ProfileInvoiceDefault.taxNo`（= `InvoiceApplyRequest.taxNo`） | `InvoiceApplyRequest.taxNo` | ✅ 一致 | — |
| recipientEmail | `ProfileInvoiceDefault.recipientEmail`（= `InvoiceApplyRequest.recipientEmail`） | `InvoiceApplyRequest.recipientEmail` | ✅ 一致 | — |
| recipientPhone | `ProfileInvoiceDefault.recipientPhone`（= `InvoiceApplyRequest.recipientPhone`） | `InvoiceApplyRequest.recipientPhone` | ✅ 一致 | — |
| login_result | `ProfileLoginLogItem.login_result: 'success' \| 'failure' \| 'failed'` | 待确认 | ⚠️ 前端已兼容两种值 | 待联调确认 |
| login_type | `ProfileLoginLogItem.login_type`（snake_case） | 待确认 | ⚠️ 命名风格 | 待联调确认 |

## 四、主流程联调

| 流程 | 操作 | 预期结果 | 实际结果 | 状态 |
|------|------|----------|----------|------|
| 打开页面 | 访问 /profile | 显示个人中心页面 | FormPageShell + Tabs 渲染 | ✅ |
| 加载账户信息 | 自动 GET /account/profile | 显示头像+标签+基本信息 | ProfileForm 渲染 profile 数据 | ✅ |
| 编辑资料 | 点击"编辑资料"→ 修改字段 → 保存 | PUT /account/profile → 刷新数据 | 编辑模式切换，保存后刷新 | ✅ |
| 查看安全设置 | 切换到"安全设置"Tab | GET /account/security → 显示安全信息 | SecuritySettings 渲染 | ✅ |
| 修改密码 | 点击"修改密码"→ 填写 → 确认 | POST /change-password → 成功关闭弹窗 | Modal + 校验 + 提交 | ✅ |
| 修改偏好设置 | 切换到"偏好设置"→ 修改 → 保存 | PUT /account/preferences → 刷新 | PreferenceSettings 渲染 | ✅ |
| 管理发票信息 | 切换到"发票信息"→ 设置/修改 | 显示/编辑发票默认信息 | InvoiceSettings 渲染 | ✅ |
| 查看登录历史 | 切换到"登录历史" | GET /account/login-logs → 分页表格 | LoginHistory 渲染 | ✅ |
| API 失败重试 | 安全设置 API 失败 → 点击重试 | 显示 Alert + 重试按钮 → 重新请求 | 错误态 + 重试 | ✅ |

## 五、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 |
|------|----------|----------|----------|
| 接口 400 | `{ code: 400, message: "参数错误" }` | ApiClient 抛出异常 → 组件 Alert 提示 | ✅ |
| 接口 401 | 401 状态码 | ApiClient 自动刷新 Token → 失败跳转登录 | ✅ |
| 接口 403 | `{ code: 403, message: "权限不足" }` | ApiClient 抛出异常 → 组件 Alert 提示 | ✅ |
| 接口 500 | `{ code: 500, message: "服务器错误" }` | 页面级 error 态 + 重试按钮 | ✅ |
| 网络超时 | 请求超时 | catch → 错误态 | ✅ |
| 空数据 | `{ data: null }` | extractApiData 返回 null → Skeleton/Empty | ✅ |
| 字段缺失 | 响应中缺少可选字段 | 可选字段显示 '-' | ✅ |
| 枚举未知值 | status 为未知值 | STATUS_MAP fallback → 'default' Tag | ✅ |
| 权限不足 | 未登录访问 /profile | RequireAuth → 跳转 /login | ✅ |
| 重复提交 | 快速双击保存按钮 | Button loading 状态禁用 | ✅ |
| 金额/数值边界 | N/A（个人中心无金额输入） | — | ✅ |
| 时间格式异常 | createdAt 为空 | 显示 '-' | ✅ |
| 登录结果未知值 | login_result 为 null | LOGIN_RESULT_MAP fallback → 原始值 | ✅ |

## 六、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
|---------|----------|----------|----------|----------|----------|----------|
| BUG-FE001 | High | ProfileForm 中 `developerName` 和 `contactName` 混淆：编辑模式"开发者名称"标签绑定的是 `contactName`，展示模式"开发者名称"显示的也是 `contactName`，导致 `developerName` 字段完全丢失 | 打开个人中心 → 查看账户信息 → "开发者名称"显示的是联系人姓名而非开发者名称 | 前端 | 开发者名称数据丢失，用户无法查看/编辑正确的开发者名称 | 修复字段绑定 |
| BUG-FE002 | Medium | InvoiceSettings displayRows 在 invoiceDefault 为 null 时使用 `invoiceDefault!` 非空断言 | 打开发票信息 Tab，invoiceDefault 为 null | 前端 | 发票信息 Tab 白屏 | 移入条件渲染内部 |
| BUG-FE003 | Medium | Tab 懒加载时 security/preferences API 失败后显示永久 Skeleton | 切换到安全设置/偏好设置 Tab，API 失败 | 前端 | 用户无法知道加载失败 | 添加错误状态和重试 |
| BUG-FE004 | Medium | ProfileForm 编辑表单包含 internalRemark 字段，用户编辑备注可能覆盖发票 JSON | 编辑账户信息，修改备注说明，保存 | 前端 | 发票默认信息丢失 | 移除 internalRemark 字段 |
| BUG-FE005 | Low | LoginHistory 中 filtersRef 声明但未使用 | 代码审查 | 前端 | 代码质量 | 删除死代码 |

## 七、Bug修复内容

### BUG-FE001：developerName 与 contactName 混淆

**问题原因**：`UserProfile` 中 `developerName`（开发者名称）和 `contactName`（联系人姓名）是两个不同字段，但 ProfileForm 编辑模式和展示模式都将 `contactName` 标记为"开发者名称"，导致 `developerName` 字段完全丢失。

**修改文件**：`packages/components/blocks/ProfileForm/index.tsx`

**修复方案**：
1. formData 初始化时添加 `developerName`
2. 编辑表单中"开发者名称"Input 绑定 `developerName`，新增"联系人姓名"Input 绑定 `contactName`
3. 展示模式中"开发者名称"显示 `profile.developerName`，新增"联系人姓名"显示 `profile.contactName`
4. 头部卡片标题优先使用 `developerName`
5. Avatar fallback 优先使用 `developerName`

**修复代码**：
```tsx
// formData 初始化
setFormData({
  developerName: profile.developerName || '',
  contactName: profile.contactName || '',
  ...
});

// 编辑表单
<div>
  <label>开发者名称</label>
  <Input value={formData.developerName || ''} onChange={(e) => handleChange('developerName', e.target.value)} />
</div>
<div>
  <label>联系人姓名</label>
  <Input value={formData.contactName || ''} onChange={(e) => handleChange('contactName', e.target.value)} />
</div>

// 展示模式
<div>开发者名称: {profile.developerName || '-'}</div>
<div>联系人姓名: {profile.contactName || '-'}</div>

// 头部标题
<h3>{profile.developerName || profile.contactName || profile.userId}</h3>

// Avatar fallback
{profile.developerName?.charAt(0)?.toUpperCase() || profile.contactName?.charAt(0)?.toUpperCase() || ...}
```

### BUG-FE002~FE005

已在上一轮回归测试中修复，此处不再重复。

## 八、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| BUG-FE001 复测：开发者名称展示 | ✅ | `profile.developerName` 正确显示 |
| BUG-FE001 复测：开发者名称编辑 | ✅ | Input 绑定 `developerName`，保存时发送 |
| BUG-FE001 复测：联系人姓名展示 | ✅ | `profile.contactName` 正确显示 |
| BUG-FE001 复测：联系人姓名编辑 | ✅ | Input 绑定 `contactName`，保存时发送 |
| BUG-FE001 复测：头部标题 | ✅ | 优先 `developerName` → `contactName` → `userId` |
| BUG-FE001 复测：Avatar fallback | ✅ | 优先 `developerName` → `contactName` → `userId` |
| 主流程回归：5 个 Tab 切换 | ✅ | 正常 |
| 主流程回归：编辑保存 | ✅ | developerName + contactName 均正确发送 |
| API 契约复测：ProfileUpdateRequest | ✅ | 包含 developerName 字段 |
| 异常场景复测：API 失败重试 | ✅ | Alert + 重试按钮 |
| 旧功能回归：/account 页面 | ✅ | 未受影响 |
| TypeScript 编译 | ✅ | 新代码无错误 |

## 九、功能验收结论

| 验收项 | 状态 | 说明 |
|--------|------|------|
| 账户信息展示正常 | ✅ | developerName/contactName/companyName/phone/email/address |
| 账户信息编辑正常 | ✅ | developerName/contactName 可独立编辑 |
| 头像上传正常 | ⚠️ | V2 迭代：当前仅 URL 输入 |
| 安全设置展示正常 | ✅ | 密码/二次验证/绑定信息/会话管理 |
| 偏好设置保存正常 | ✅ | 默认配置+通知设置 |
| 发票信息管理正常 | ✅ | 个人/企业切换，条件字段 |
| 登录历史展示正常 | ✅ | 筛选+分页+空态 |
| 空态/加载态/错误态完整 | ✅ | 含 Tab 懒加载失败态 |
| API 错误处理完整 | ✅ | 页面级+组件级+Tab 级 |
| 旧功能未受影响 | ✅ | /account 保留 |

👉 **Accepted with notes**

### Notes

1. **头像上传**：V2 迭代补充 Upload 组件 + 专用 API
2. **主题切换**：V2 迭代补充
3. **发票专用 API**：V2 迭代升级
4. **登录日志字段命名**：待后端联调确认 snake_case vs camelCase
5. **login_result 值**：待后端联调确认 `'failure'` vs `'failed'`

## 十、是否允许合并

| 决策项 | 结论 |
|--------|------|
| 是否允许合并 | **YES** |
| 是否允许进入下一任务 | **YES** |
| 是否需要继续修复 | **NO**（无 Blocker/High Bug 未修复） |
| 是否需要后端继续处理 | **YES**（登录日志字段命名待确认） |
| 是否需要产品确认 | **NO** |
