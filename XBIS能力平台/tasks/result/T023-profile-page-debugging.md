# T023 个人中心 — 接口联调检查报告（D1）

## 🧪 联调结果（D1）

👉 状态：**前端待修**（已修复，待联调验证）

---

## 1️⃣ API契约一致性

| API端点 | URL | Method | 前端调用 | 状态 |
|---------|-----|--------|----------|------|
| 获取账户信息 | `/portal-api/v1/account/profile` | GET | `userApi.account.profile.get()` | ✅ 一致 |
| 更新账户信息 | `/portal-api/v1/account/profile` | PUT | `userApi.account.profile.update(data)` | ✅ 一致 |
| 获取安全设置 | `/portal-api/v1/account/security` | GET | `userApi.account.security()` | ✅ 一致 |
| 修改密码 | `/portal-api/v1/account/change-password` | POST | `userApi.account.changePassword(data)` | ✅ 一致 |
| 登录日志 | `/portal-api/v1/account/login-logs` | GET | `userApi.account.loginLogs(params)` | ✅ 一致 |
| 获取偏好设置 | `/portal-api/v1/account/preferences` | GET | `userApi.account.preferences.get()` | ✅ 一致 |
| 更新偏好设置 | `/portal-api/v1/account/preferences` | PUT | `userApi.account.preferences.update(data)` | ✅ 一致 |

**结论：URL、Method 全部一致，无问题。**

---

## 2️⃣ 返回结构匹配

### ❌ 已修复：ProfileAccountData 字段名与后端不一致

| 问题 | 原字段名 | 修正为 | 对齐依据 |
|------|----------|--------|----------|
| 开发者类型字段名 | `userType` | `developerType` | 与 `UserProfile.developerType`、`User.developerType`、`AccountInfo.developerType` 一致 |
| 备注字段名 | `remark` | `internalRemark` | 与 `UserProfile.internalRemark` 一致 |
| 缺少开发者名称 | 无 | `developerName` | 与 `UserProfile.developerName` 一致 |
| 缺少标签 | 无 | `tags` | 与 `UserProfile.tags` 一致 |
| 状态类型过宽 | `string` | `'active' \| 'suspended' \| 'disabled' \| string` | 与 `User.status` 一致 |

### ❌ 已修复：ProfileInvoiceDefault 字段名与 InvoiceApplyRequest 不一致

| 问题 | 原字段名 | 修正为 | 对齐依据 |
|------|----------|--------|----------|
| 发票类型字段名 | `type` | `invoiceType` | 与 `InvoiceApplyRequest.invoiceType` 一致 |
| 发票类型值 | `'enterprise'` | `'company'` | 与 `InvoiceApplyRequest.invoiceType: 'personal' \| 'company'` 一致 |
| 税号字段名 | `taxNumber` | `taxNo` | 与 `InvoiceApplyRequest.taxNo` 一致 |
| 邮箱字段名 | `email` | `recipientEmail` | 与 `InvoiceApplyRequest.recipientEmail` 一致 |
| 电话字段名 | `phone` | `recipientPhone` | 与 `InvoiceApplyRequest.recipientPhone` 一致 |

### ✅ ProfileSecurityData 字段补充

新增 `allowLogin`、`loginFailCount`、`lastAbnormalLoginAt` 字段（与 `UserLoginSecurity` 对齐），设为可选。

### ⚠️ 待联调确认：ProfileLoginLogItem 字段命名

| ProfileLoginLogItem（前端） | LoginHistoryRecord（已有类型） | 说明 |
|---------------------------|---------------------------|------|
| `login_type` (snake_case) | — | 用户端 API 返回 snake_case（与 `InvocationRecord`、`ApiKeyListItem` 一致） |
| `login_ip` (snake_case) | `ipAddress` (camelCase) | 管理端 API 可能返回 camelCase |
| `user_agent` (snake_case) | `userAgent` (camelCase) | 同上 |
| `login_result: 'failure'` | `status: 'failed'` | **值不同**：`failure` vs `failed` |
| `created_at` (snake_case) | `loginAt` (camelCase) | 同上 |

**说明**：用户端 API（`/portal-api/v1/`）返回 snake_case，管理端 API（`/admin-api/v1/`）返回 camelCase。`ProfileLoginLogItem` 使用 snake_case 与用户端 API 一致，`LoginHistoryRecord` 使用 camelCase 与管理端 API 一致。**待联调确认后端实际返回格式。**

---

## 3️⃣ 数据结构一致性

### 类型复用关系

| 新类型 | 对应已有类型 | 关系 |
|--------|------------|------|
| `ProfileAccountData` | `UserProfile` + `User` | 扩展，字段已对齐 |
| `ProfileSecurityData` | `UserLoginSecurity` | 扩展，补充了 phoneBound/emailBound 等字段 |
| `ProfileInvoiceDefault` | `InvoiceApplyRequest` | 子集（不含 orderNos），字段名已对齐 |
| `ProfileLoginLogItem` | `LoginHistoryRecord` | 不同 API 端点，命名风格不同 |
| `ChangePasswordRequest` | `adminApi.profile.changePassword` 参数 | 完全一致 ✅ |

### 字段命名冲突

无。所有 T023 新类型与已有类型字段名已对齐。

---

## 4️⃣ 状态流一致性

### 账户状态

| 状态值 | 前端 STATUS_MAP | 后端 User.status | 说明 |
|--------|----------------|-----------------|------|
| active | ✅ 正常 | ✅ | 一致 |
| suspended | ✅ 暂停 | ✅ | 一致 |
| disabled | ✅ 禁用 | ✅ | 一致 |
| normal | ✅ 正常 | — | 前端兼容，不影响 |
| restricted | ✅ 限制 | — | 前端兼容，不影响 |
| forbidden | ✅ 禁止 | — | 前端兼容，不影响 |
| frozen | ✅ 冻结 | — | 前端兼容，不影响 |

### 认证状态

| 状态值 | 前端 CERT_STATUS_MAP | 后端 UserCertification.certStatus | 说明 |
|--------|---------------------|----------------------------------|------|
| none | ✅ 未认证 | — | 前端补充，合理 |
| pending | ✅ 审核中 | ✅ | 一致 |
| approved | ✅ 已认证 | ✅ | 一致 |
| rejected | ✅ 已驳回 | ✅ | 一致 |

### 登录结果

| 前端值 | 已有类型值 | 说明 |
|--------|-----------|------|
| `'success'` | `'success'` | 一致 ✅ |
| `'failure'` | `'failed'` | **不一致** — 待联调确认后端实际返回值 |

---

## 5️⃣ 错误处理

| 组件 | loading | empty | error | 超时/失败 |
|------|---------|-------|-------|----------|
| ProfilePage | ✅ Skeleton | — | ✅ Alert + 重试 | ✅ catch |
| ProfileForm | — | — | ✅ Alert | ✅ catch |
| SecuritySettings | ✅ Skeleton | — | ✅ Modal Alert | ✅ catch |
| PreferenceSettings | ✅ Skeleton | — | ✅ Alert | ✅ catch |
| InvoiceSettings | — | ✅ Empty 提示 | ✅ Alert | ✅ catch |
| LoginHistory | ✅ Spinner | ✅ Empty | ✅ catch 返回空 | ✅ catch |

**结论：错误处理完整。**

---

## 6️⃣ Business Services 层检查

| API调用 | 是否通过 Service 层 | 是否绕过 XBIS |
|---------|-------------------|--------------|
| `userApi.account.profile.get()` | ✅ | 否 |
| `userApi.account.profile.update()` | ✅ | 否 |
| `userApi.account.security()` | ✅ | 否 |
| `userApi.account.changePassword()` | ✅ | 否 |
| `userApi.account.loginLogs()` | ✅ | 否 |
| `userApi.account.preferences.get()` | ✅ | 否 |
| `userApi.account.preferences.update()` | ✅ | 否 |

**结论：全部通过 Business Services 层，无绕过。**

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 状态 |
|------|------|------|------|------|
| 字段名 | `userType` 应为 `developerType` | ProfileAccountData / ProfileForm | API 返回字段不匹配，数据无法正确读取 | ✅ 已修复 |
| 字段名 | `remark` 应为 `internalRemark` | ProfileAccountData / ProfileForm / ProfilePage | API 返回字段不匹配 | ✅ 已修复 |
| 字段名 | `type` 应为 `invoiceType`，值 `'enterprise'` 应为 `'company'` | ProfileInvoiceDefault / InvoiceSettings | 与 InvoiceApplyRequest 不一致，发票申请预填会出错 | ✅ 已修复 |
| 字段名 | `taxNumber` 应为 `taxNo` | ProfileInvoiceDefault / InvoiceSettings | 与 InvoiceApplyRequest 不一致 | ✅ 已修复 |
| 字段名 | `email` 应为 `recipientEmail`，`phone` 应为 `recipientPhone` | ProfileInvoiceDefault / InvoiceSettings | 与 InvoiceApplyRequest 不一致 | ✅ 已修复 |
| 缺失字段 | 缺少 `developerName`、`tags` | ProfileAccountData | 后端可能返回但前端未接收 | ✅ 已修复 |
| 缺失字段 | 缺少 `allowLogin`、`loginFailCount` | ProfileSecurityData | 后端可能返回但前端未接收 | ✅ 已修复 |
| 值不一致 | `login_result: 'failure'` vs `LoginHistoryRecord.status: 'failed'` | ProfileLoginLogItem | 待联调确认 | ⚠️ 待确认 |
| 临时方案 | 发票信息通过 `internalRemark` JSON 序列化存储 | ProfilePage | 无专用 API，数据可能被其他操作覆盖 | ⚠️ 已知限制 |

---

## 🛠 修改建议

### 前端修改（已完成）

1. ✅ `ProfileAccountData.userType` → `developerType`
2. ✅ `ProfileAccountData.remark` → `internalRemark`
3. ✅ 补充 `ProfileAccountData.developerName`、`tags` 字段
4. ✅ `ProfileAccountData.status` 收窄为 `'active' | 'suspended' | 'disabled' | string`
5. ✅ `ProfileInvoiceDefault` 字段全部对齐 `InvoiceApplyRequest`
6. ✅ `ProfileSecurityData` 补充 `allowLogin`、`loginFailCount`、`lastAbnormalLoginAt`
7. ✅ `ProfileUpdateRequest.remark` → `internalRemark`，补充 `developerName`
8. ✅ `parseInvoiceFromRemark` 增加新旧字段兼容映射
9. ✅ ProfileForm、InvoiceSettings、ProfilePage 同步更新

### 后端修改

- 无需修改。前端已对齐后端已有类型定义。

### 数据结构调整

- 发票默认信息存储方案：当前通过 `internalRemark` JSON 序列化，建议后端后续提供专用 API `/portal-api/v1/account/invoice-default`（GET/PUT）

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 登录日志字段命名 | 中 | 用户端 API 可能返回 camelCase 而非 snake_case，需联调确认 |
| 登录结果值 | 低 | `'failure'` vs `'failed'`，需联调确认后端实际返回 |
| 发票信息临时存储 | 低 | 通过 `internalRemark` 存储 JSON，可能与其他备注操作冲突 |
| 类型重复 | 低 | `ProfileAccountData` 与 `UserProfile` 存在字段重叠，后续可考虑合并 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

所有 Blocking 问题已修复：
- ✅ 字段名与后端已有类型完全对齐
- ✅ 发票类型字段与 `InvoiceApplyRequest` 一致
- ✅ API 契约（URL/Method/参数）全部一致
- ✅ Business Services 层合规
- ✅ 错误处理完整
- ✅ TypeScript 编译通过

⚠️ 待联调确认项（不阻塞验收）：
1. `ProfileLoginLogItem` 字段命名风格（snake_case vs camelCase）
2. `login_result` 值（`'failure'` vs `'failed'`）
3. 发票信息专用 API（当前临时方案可用）
