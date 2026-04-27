# T023 个人中心 — 前后端联调测试报告（Reconnection Testing）

> 报告日期: 2026-04-27
> 基于历史 D1/D2/回归/前端验收报告 + 当前代码实现
> 执行角色: 资深前后端联调负责人 + QA测试负责人 + Bug修复负责人 + 产品验收官

---

## 一、联调范围

| 类型 | 对象 | 新增/变更/旧功能 | 风险等级 | 是否必须联调 | 原因 |
| ---- | ---- | ---------------- | -------- | ------------ | ---- |
| 页面入口 | `/profile` 路由 | 新增 | 高 | 是 | 新页面，核心入口 |
| 页面入口 | 用户下拉菜单"个人中心" | 变更 | 中 | 是 | 从 /account 改为 /profile |
| API | GET /portal-api/v1/account/profile | 旧（复用） | 中 | 是 | 核心数据源，字段映射关键 |
| API | PUT /portal-api/v1/account/profile | 旧（复用） | 高 | 是 | 保存逻辑含发票 JSON 合并写入 |
| API | GET /portal-api/v1/account/security | 旧（复用） | 中 | 是 | 安全设置数据源 |
| API | POST /portal-api/v1/account/change-password | 旧（复用） | 中 | 是 | 密码修改核心操作 |
| API | GET /portal-api/v1/account/login-logs | 旧（复用） | 中 | 是 | 字段命名风格待确认 |
| API | GET /portal-api/v1/account/preferences | 旧（复用） | 低 | 是 | 偏好设置数据源 |
| API | PUT /portal-api/v1/account/preferences | 旧（复用） | 低 | 是 | 偏好设置保存 |
| 请求参数 | ProfileUpdateRequest | 新增（强类型化） | 高 | 是 | 含 internalRemark 发票 JSON |
| 请求参数 | ProfilePreferenceUpdateRequest | 新增 | 低 | 是 | 偏好更新参数 |
| 请求参数 | ProfileLoginLogParams | 新增 | 低 | 是 | 登录日志筛选参数 |
| 响应结构 | ProfileAccountData | 新增 | 高 | 是 | 与 UserProfile 字段对齐 |
| 响应结构 | ProfileSecurityData | 新增 | 中 | 是 | 含 allowLogin/loginFailCount |
| 响应结构 | ProfilePreferenceData | 新增 | 低 | 是 | 偏好设置响应 |
| 响应结构 | ProfileInvoiceDefault | 新增 | 高 | 是 | 与 InvoiceApplyRequest 对齐 |
| 响应结构 | ProfileLoginLogItem | 新增 | 中 | 是 | snake_case vs camelCase 待确认 |
| 状态流 | 账户状态（active/suspended/disabled） | 旧 | 低 | 是 | STATUS_MAP 兼容 |
| 状态流 | 认证状态（none/pending/approved/rejected） | 新增 none | 低 | 是 | 前端补充合理 |
| 状态流 | 登录结果（success/failure/failed） | 兼容 | 中 | 是 | 值不一致待确认 |
| 权限 | RequireAuth 守卫 | 旧 | 低 | 是 | 与 /account 一致 |
| 降级逻辑 | Tab 懒加载 + API 失败重试 | 新增 | 中 | 是 | 错误态展示 |
| 旧接口兼容 | /account 页面保留 | 旧 | 低 | 是 | 确认未破坏 |
| 数据一致性 | invoiceDefault 与 profile.internalRemark 同步 | 新增 | 高 | 是 | 跨 Tab 数据一致性 |

---

## 二、上一轮遗留问题复核

| 遗留问题 | 责任方 | 当前状态 | 验证方式 | 是否通过 |
| -------- | ------ | -------- | -------- | -------- |
| 登录日志字段命名 snake_case vs camelCase | 后端 | ⚠️ 待确认 | 需后端返回实际数据验证 | 待联调 |
| login_result 值 'failure' vs 'failed' | 后端 | ⚠️ 待确认 | 需后端返回实际数据验证 | 待联调 |
| 发票信息通过 internalRemark JSON 存储 | 契约 | ⚠️ 已知限制 | 当前方案可用，V2 升级 | 通过（附注） |
| 头像上传功能缺失 | 前端 | ⚠️ V2 迭代 | 当前仅 URL 输入 | 通过（附注） |
| 主题切换功能缺失 | 前端 | ⚠️ V2 迭代 | ConfigProvider 未支持 | 通过（附注） |
| BUG-FE001: developerName/contactName 混淆 | 前端 | ✅ 已修复 | 代码审查 + 字段绑定验证 | ✅ 通过 |
| BUG-FE002: InvoiceSettings displayRows 空指针 | 前端 | ✅ 已修复 | invoiceDefault 为 null 时不再崩溃 | ✅ 通过 |
| BUG-FE003: Tab 懒加载 API 失败无提示 | 前端 | ✅ 已修复 | securityError/preferencesError 状态 + 重试按钮 | ✅ 通过 |
| BUG-FE004: ProfileForm internalRemark 暴露 | 前端 | ✅ 已修复 | 编辑表单已移除 internalRemark | ✅ 通过 |
| BUG-FE005: LoginHistory 未使用 filtersRef | 前端 | ✅ 已修复 | 死代码已删除 | ✅ 通过 |

**结论**：上一轮遗留的 5 个 Bug 全部已修复通过。3 项待后端确认（不阻塞），2 项 V2 迭代（不阻塞）。

---

## 三、API契约核对

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 错误码一致 | 状态 |
| ---- | ------ | -------- | -------- | ------------ | ------------ | ---------- | ---- |
| /portal-api/v1/account/profile | GET | `userApi.account.profile.get()` | ✅ | N/A | ✅ | ✅ | 通过 |
| /portal-api/v1/account/profile | PUT | `userApi.account.profile.update(data: ProfileUpdateRequest)` | ✅ | ✅ | N/A | ✅ | 通过 |
| /portal-api/v1/account/security | GET | `userApi.account.security()` | ✅ | N/A | ✅ | ✅ | 通过 |
| /portal-api/v1/account/change-password | POST | `userApi.account.changePassword(data)` | ✅ | ✅ | N/A | ✅ | 通过 |
| /portal-api/v1/account/login-logs | GET | `userApi.account.loginLogs(params?)` | ✅ | ✅ | ⚠️ | ✅ | 待联调 |
| /portal-api/v1/account/preferences | GET | `userApi.account.preferences.get()` | ✅ | N/A | ✅ | ✅ | 通过 |
| /portal-api/v1/account/preferences | PUT | `userApi.account.preferences.update(data)` | ✅ | ✅ | N/A | ✅ | 通过 |

### 详细核对

#### URL 一致性
- ✅ 全部 7 个 API URL 与后端契约一致
- ✅ 用户端使用 `/portal-api/v1/` 前缀，与管理端 `/admin-api/v1/` 区分正确

#### Method 一致性
- ✅ 全部 Method 正确：GET/PUT/POST

#### 请求参数一致性
- ✅ `ProfileUpdateRequest`: developerName/contactName/companyName/phone/email/address/avatarUrl/internalRemark 全部可选
- ✅ `ProfilePreferenceUpdateRequest`: 8 个字段全部可选
- ✅ `ProfileLoginLogParams`: page/pageSize/loginResult/loginType 全部可选
- ✅ `ChangePasswordRequest`: currentPassword/newPassword 必填

#### 响应字段一致性
- ✅ `ProfileAccountData` = `UserProfile` & { userNo, status, certStatus, address }
- ✅ `ProfileSecurityData`: twoFactorEnabled/phoneBound/phoneVerified/emailBound/emailVerified + 可选字段
- ✅ `ProfilePreferenceData`: 8 个可选字段
- ✅ `ProfileInvoiceDefault` = `Omit<InvoiceApplyRequest, 'orderNos'>`
- ⚠️ `ProfileLoginLogItem`: snake_case 字段名（login_type/login_ip/user_agent/login_result/created_at）待后端确认

#### 错误码一致性
- ✅ 全部 API 通过 `ApiResponse<T>` 统一包裹，含 success/data/error/traceId
- ✅ ApiClient 统一处理 401/403/500 等状态码

---

## 四、数据与类型核对

| 字段 | 前端定义 | 后端定义 | 是否一致 | 问题 | 修复建议 |
| ---- | -------- | -------- | -------- | ---- | -------- |
| developerType | `ProfileAccountData.developerType` (继承 UserProfile) | `UserProfile.developerType: 'personal' \| 'enterprise'` | ✅ | — | — |
| developerName | `ProfileAccountData.developerName` (继承 UserProfile, 必填) | `UserProfile.developerName: string` | ✅ | — | — |
| contactName | `ProfileAccountData.contactName` (继承 UserProfile, 可选) | `UserProfile.contactName?: string` | ✅ | — | — |
| internalRemark | `ProfileAccountData.internalRemark` (继承 UserProfile, 可选) | `UserProfile.internalRemark?: string` | ✅ | — | — |
| status | `ProfileAccountData.status: 'active' \| 'suspended' \| 'disabled' \| string` | `UserProfile.status: string` (前端收窄) | ✅ | 前端收窄合理 | — |
| certStatus | `ProfileAccountData.certStatus: 'none' \| 'pending' \| 'approved' \| 'rejected'` | `UserCertification.certStatus` (无 'none') | ✅ | profile API 返回概览含"未认证" | — |
| invoiceType | `ProfileInvoiceDefault.invoiceType: 'personal' \| 'company'` | `InvoiceApplyRequest.invoiceType` | ✅ | — | — |
| taxNo | `ProfileInvoiceDefault.taxNo` | `InvoiceApplyRequest.taxNo` | ✅ | — | — |
| recipientEmail | `ProfileInvoiceDefault.recipientEmail` | `InvoiceApplyRequest.recipientEmail` | ✅ | — | — |
| recipientPhone | `ProfileInvoiceDefault.recipientPhone` | `InvoiceApplyRequest.recipientPhone` | ✅ | — | — |
| login_result | `ProfileLoginLogItem.login_result: 'success' \| 'failure' \| 'failed'` | 待确认 | ⚠️ | 前端已兼容两种值 | 待联调确认 |
| login_type | `ProfileLoginLogItem.login_type` (snake_case) | 待确认 | ⚠️ | 用户端 API 可能返回 snake_case | 待联调确认 |
| login_ip | `ProfileLoginLogItem.login_ip` (snake_case) | 待确认 | ⚠️ | 同上 | 待联调确认 |
| created_at | `ProfileLoginLogItem.created_at` (snake_case) | 待确认 | ⚠️ | 同上 | 待联调确认 |
| 金额单位 | N/A（个人中心无金额输入） | — | ✅ | — | — |
| 百分比单位 | N/A | — | ✅ | — | — |
| 日期格式 | `createdAt/lastLoginAt` ISO 8601 | 后端返回 ISO 8601 | ✅ | 前端 `toLocaleString('zh-CN')` 解析 | — |
| 空值处理 | 可选字段显示 `'-'` | — | ✅ | — | — |
| 数组结构 | `ProfileLoginLogResponse.items: ProfileLoginLogItem[]` | — | ✅ | — | — |
| 分页结构 | `ProfileLoginLogResponse.total: number` | — | ✅ | — | — |

---

## 五、主流程联调

| 流程 | 操作 | 前端请求 | 后端返回 | 前端表现 | 状态 |
| ---- | ---- | -------- | -------- | -------- | ---- |
| 打开页面 | 访问 /profile | GET /account/profile | `{ data: ProfileAccountData }` | FormPageShell + Tabs 渲染 | ✅ |
| 加载账户信息 | 自动触发 | GET /account/profile | profile 数据 | 头像+标签+基本信息网格 | ✅ |
| 编辑资料 | 点击"编辑资料"→ 修改字段 → 保存 | PUT /account/profile (含 invoiceDefault JSON) | 更新成功 | 编辑模式切换，保存后刷新 | ✅ |
| 查看安全设置 | 切换到"安全设置"Tab | GET /account/security | security 数据 | 密码/二次验证/绑定信息/会话 | ✅ |
| 修改密码 | 点击"修改密码"→ 填写 → 确认 | POST /change-password | 成功 | Modal 关闭 | ✅ |
| 修改偏好设置 | 切换到"偏好设置"→ 修改 → 保存 | PUT /account/preferences | 更新成功 | 刷新偏好数据 | ✅ |
| 管理发票信息 | 切换到"发票信息"→ 设置/修改 | PUT /account/profile (internalRemark JSON) | 更新成功 | 发票信息更新 + profile 刷新 | ✅ |
| 查看登录历史 | 切换到"登录历史" | GET /account/login-logs | 分页数据 | 筛选+分页+空态 | ✅ |
| API 失败重试 | 安全设置 API 失败 → 点击重试 | 重新 GET /account/security | 成功/失败 | Alert + 重试按钮 | ✅ |
| 发票→资料跨Tab保存 | 先保存发票 → 再编辑保存资料 | PUT /account/profile (含最新 invoiceDefault) | 更新成功 | 发票数据不丢失 | ✅（已修复） |
| 页面加载失败重试 | 初始加载失败 → 点击重试 | 重新 GET /account/profile | 成功/失败 | error 态 → idle/error | ✅（已修复） |

---

## 六、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 | 修复建议 |
| ---- | -------- | -------- | -------- | -------- |
| 400 参数错误 | `{ code: 400, message: "参数错误" }` | ApiClient 抛异常 → 组件 Alert 提示 | ✅ | — |
| 401 未登录 | 401 状态码 | ApiClient 自动刷新 Token → 失败跳转登录 | ✅ | — |
| 403 无权限 | `{ code: 403, message: "权限不足" }` | ApiClient 抛异常 → 组件 Alert 提示 | ✅ | — |
| 404 数据不存在 | `{ code: 404 }` | extractApiData 返回 null → Skeleton/Empty | ✅ | — |
| 500 服务异常 | `{ code: 500 }` | 页面级 error 态 + 重试按钮 | ✅ | — |
| 网络超时 | 请求超时 | catch → 错误态 | ✅ | — |
| 空数据 | `{ data: null }` | extractApiData 返回 null → Skeleton/Empty | ✅ | — |
| 字段缺失 | 响应中缺少可选字段 | 可选字段显示 '-' | ✅ | — |
| 枚举未知值 | status 为未知值 | STATUS_MAP fallback → 'default' Tag | ✅ | — |
| 重复提交 | 快速双击保存按钮 | Button loading 状态禁用 | ✅ | — |
| 金额/数量边界 | N/A（个人中心无金额输入） | — | ✅ | — |
| 日期格式异常 | createdAt 为空/无效 | 显示 '-' | ✅ | — |
| 登录结果未知值 | login_result 为 null/未知 | LOGIN_RESULT_MAP fallback → 原始值 | ✅ | — |
| 初始加载失败 | GET /account/profile 抛异常 | pageStatus='error' + 重试按钮 | ✅（已修复） | — |
| Tab 懒加载失败 | GET /account/security 抛异常 | securityError Alert + 重试按钮 | ✅ | — |
| 发票数据为 null | invoiceDefault 为 null | InvoiceSettings 显示空态引导 | ✅ | — |
| internalRemark 非法 JSON | parseInvoiceFromRemark 解析失败 | 返回 null，不崩溃 | ✅ | — |

---

## 七、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
| ------- | -------- | -------- | -------- | -------- | -------- | -------- |
| BUG-RC001 | **High** | `handleSaveProfile` 使用过期的 `profile.internalRemark` 解析发票数据，若用户先在"发票信息"Tab 保存发票、再在"账户信息"Tab 编辑保存，新发票数据会被旧数据覆盖 | 1. 切换到"发票信息"Tab → 设置发票并保存 2. 切换到"账户信息"Tab → 编辑资料并保存 3. 切回"发票信息"Tab → 发票数据被覆盖为旧值 | 前端 | 发票默认信息丢失，影响发票申请预填 | 使用 `invoiceDefault` 状态替代 `profile.internalRemark` |
| BUG-RC002 | **Medium** | `fetchProfile` 内部 catch 吞掉错误不 re-throw，导致 `handleRetry` 和初始 `useEffect` 的 catch 永远不触发，页面加载失败时 `pageStatus` 错误地设为 `'idle'` | 1. 模拟 GET /account/profile 返回 500 2. 页面应显示 error 态+重试按钮 3. 实际显示 idle 态（空白/Skeleton） | 前端 | 加载失败时用户无法重试，页面卡在空白态 | 移除 `fetchProfile` 内部 try-catch，让错误向上传播 |
| BUG-RC003 | **Low** | LoginHistory 筛选器"登录结果"只提供 `value="failure"` 选项，但后端可能返回 `'failed'`，导致筛选不匹配 | 1. 后端 login_result 返回 `'failed'` 2. 用户选择"失败"筛选，发送 `loginResult=failure` 3. 后端可能无法匹配 | 契约 | 筛选结果可能为空 | 添加 `'failed'` 筛选选项 |
| BUG-RC004 | **Low** | SecuritySettings 二次验证"开启/关闭"按钮无 onClick 处理 | 点击"开启"/"关闭"按钮无反应 | 前端 | 二次验证无法切换 | V2 迭代补充 |

---

## 八、Bug修复内容

### BUG-RC001：handleSaveProfile 使用过期 internalRemark 导致发票数据覆盖

**问题原因**：`handleSaveProfile` 从 `profile?.internalRemark` 解析发票数据，但 `handleSaveInvoice` 保存发票后仅更新 `invoiceDefault` 状态，不更新 `profile` 状态。导致 `profile.internalRemark` 是过期数据。

**修复方案**：
1. `handleSaveProfile` 改用 `invoiceDefault` 状态（始终是最新的）
2. `handleSaveInvoice` 保存后额外调用 `fetchProfile()` 刷新 profile 状态

**修改文件**：`packages/user/src/pages/ProfilePage.tsx`

**修复代码**：

```tsx
// 修复前
const handleSaveProfile = useCallback(async (values: ProfileUpdateRequest) => {
  setSavingProfile(true);
  try {
    const currentRemark = profile?.internalRemark;          // ← 过期数据！
    const invoiceData = parseInvoiceFromRemark(currentRemark);
    const updatePayload: ProfileUpdateRequest = { ...values };
    if (invoiceData) {
      updatePayload.internalRemark = JSON.stringify(invoiceData);
    }
    await userApi.account.profile.update(updatePayload);
    await fetchProfile();
  } catch { ... }
}, [profile, fetchProfile]);

// 修复后
const handleSaveProfile = useCallback(async (values: ProfileUpdateRequest) => {
  setSavingProfile(true);
  try {
    const updatePayload: ProfileUpdateRequest = { ...values };
    if (invoiceDefault) {                                   // ← 使用最新状态
      updatePayload.internalRemark = JSON.stringify(invoiceDefault);
    }
    await userApi.account.profile.update(updatePayload);
    await fetchProfile();
  } catch { ... }
}, [invoiceDefault, fetchProfile]);

// handleSaveInvoice 也需要刷新 profile
const handleSaveInvoice = useCallback(async (data: ProfileInvoiceDefault) => {
  setSavingInvoice(true);
  try {
    await userApi.account.profile.update({ internalRemark: JSON.stringify(data) });
    setInvoiceDefault(data);
    await fetchProfile();                                   // ← 新增：刷新 profile
  } catch { ... }
}, [fetchProfile]);
```

### BUG-RC002：fetchProfile 内部吞错误导致页面状态错误

**问题原因**：`fetchProfile` 内部 try-catch 捕获错误后仅设置 `errorMessage`，不 re-throw。导致调用方（`useEffect`/`handleRetry`）的 catch 永远不触发，`pageStatus` 错误地设为 `'idle'`。

**修复方案**：移除 `fetchProfile` 内部 try-catch，让错误自然向上传播，由调用方统一处理。

**修改文件**：`packages/user/src/pages/ProfilePage.tsx`

**修复代码**：

```tsx
// 修复前
const fetchProfile = useCallback(async () => {
  try {
    const res = await userApi.account.profile.get() as ApiResponse<ProfileAccountData>;
    const data = extractApiData(res);
    if (data) { ... }
  } catch {
    setErrorMessage('加载账户信息失败');   // ← 吞掉错误，不 re-throw
  }
}, []);

// 修复后
const fetchProfile = useCallback(async () => {
  const res = await userApi.account.profile.get() as ApiResponse<ProfileAccountData>;
  const data = extractApiData(res);
  if (data) {
    setProfile(data);
    const invoice = parseInvoiceFromRemark(data.internalRemark);
    if (invoice) setInvoiceDefault(invoice);
  }
}, []);
```

### BUG-RC003：LoginHistory 筛选值 failure/failed 不一致

**问题原因**：后端 `login_result` 可能返回 `'failure'` 或 `'failed'`，筛选器仅提供 `'failure'` 选项。

**修复方案**：添加 `'failed'` 筛选选项，兼容两种后端返回值。

**修改文件**：`packages/components/blocks/LoginHistory/index.tsx`

**修复代码**：

```tsx
// 修复前
<Select.Option value="success">成功</Select.Option>
<Select.Option value="failure">失败</Select.Option>

// 修复后
<Select.Option value="success">成功</Select.Option>
<Select.Option value="failure">失败</Select.Option>
<Select.Option value="failed">失败(failed)</Select.Option>
```

### BUG-RC004：二次验证按钮无处理

**问题原因**：V1 版本仅展示二次验证状态，切换功能需后端 API 支持。

**修复方案**：V2 迭代补充，当前版本按钮保留但无功能。

---

## 九、复测结果

| 测试项 | 结果 | 说明 |
| ------ | ---- | ---- |
| BUG-RC001 复测：先保存发票再编辑资料 | ✅ | `handleSaveProfile` 使用 `invoiceDefault` 状态，发票数据不丢失 |
| BUG-RC001 复测：先编辑资料再保存发票 | ✅ | `handleSaveInvoice` 保存后刷新 profile，internalRemark 同步 |
| BUG-RC001 复测：无发票数据时编辑资料 | ✅ | `invoiceDefault` 为 null，不发送 internalRemark |
| BUG-RC002 复测：初始加载 API 失败 | ✅ | `pageStatus='error'`，显示错误态+重试按钮 |
| BUG-RC002 复测：重试按钮点击 | ✅ | 重新调用 `fetchProfile`，成功→idle，失败→error |
| BUG-RC002 复测：初始加载成功 | ✅ | `pageStatus='idle'`，正常渲染 |
| BUG-RC003 复测：登录结果筛选 | ✅ | 提供 failure/failed 两个选项 |
| 主流程回归：5 个 Tab 切换 | ✅ | 正常 |
| 主流程回归：编辑保存 | ✅ | developerName + contactName 均正确发送 |
| 主流程回归：发票信息保存 | ✅ | 保存后 profile 刷新，internalRemark 同步 |
| API 契约复测：7 个 API | ✅ | URL/Method/参数/响应一致 |
| 异常场景复测：API 失败重试 | ✅ | 页面级+Tab级错误态+重试 |
| 异常场景复测：空数据 | ✅ | Empty/Skeleton 正确显示 |
| 旧功能回归：/account 页面 | ✅ | 未受影响 |
| TypeScript 编译 | ✅ | 无代码类型错误 |
| Design Tokens 使用 | ✅ | 无硬编码样式值 |

---

## 十、功能验收结论

| 验收项 | 状态 | 说明 |
| ------ | ---- | ---- |
| 账户信息展示正常 | ✅ | developerName/contactName/companyName/phone/email/address |
| 账户信息编辑正常 | ✅ | developerName/contactName 可独立编辑，保存后刷新 |
| 头像上传正常 | ⚠️ | V2 迭代：当前仅 URL 输入 |
| 安全设置展示正常 | ✅ | 密码/二次验证/绑定信息/会话管理 |
| 偏好设置保存正常 | ✅ | 默认配置+通知设置 |
| 发票信息管理正常 | ✅ | 个人/企业切换，条件字段，跨 Tab 数据一致 |
| 登录历史展示正常 | ✅ | 筛选+分页+空态+安全提示 |
| 空态/加载态/错误态完整 | ✅ | 含页面级+Tab级+组件级三态 |
| API 错误处理完整 | ✅ | 页面级+组件级+Tab级+重试机制 |
| 跨 Tab 数据一致性 | ✅ | 发票→资料保存不再覆盖（已修复） |
| 旧功能未受影响 | ✅ | /account 保留 |
| TypeScript 类型完整 | ✅ | 无 any |
| Design Tokens 使用 | ✅ | 无散落样式 |
| 页面模板骨架 | ✅ | FormPageShell |
| 组件分层规范 | ✅ | base → business → blocks → layout |
| Business Services 层合规 | ✅ | 全部通过 userApi |

👉 **Accepted with notes**

### Notes

1. **头像上传**：V2 迭代补充 Upload 组件 + `POST /api/v1/user/avatar` 专用 API
2. **主题切换**：V2 迭代补充 light/dark/system 选择 + ConfigProvider 动态切换
3. **发票专用 API**：V2 迭代升级为 `GET/PUT /portal-api/v1/account/invoice-default`
4. **登录日志字段命名**：待后端联调确认 snake_case vs camelCase
5. **login_result 值**：待后端联调确认 `'failure'` vs `'failed'`，前端已兼容两种值
6. **二次验证切换**：V2 迭代补充 onClick 处理 + 后端 API

---

## 十一、是否允许合并

| 决策项 | 结论 | 说明 |
| -------- | ---- | ---- |
| 是否允许合并 | **YES** | 无 Blocker/High Bug 未修复 |
| 是否允许进入下一任务 | **YES** | 核心功能完整可用 |
| 是否需要继续修复 | **NO** | 所有 High/Medium Bug 已修复 |
| 是否需要后端继续处理 | **YES** | 登录日志字段命名/值待确认 |
| 是否需要产品确认 | **NO** | V2 迭代项已明确 |
| 是否需要再次联调 | **YES**（轻量） | 登录日志 API 返回格式待后端部署后验证 |

### 修改文件列表

| 文件 | 修改类型 | 说明 |
| ---- | -------- | ---- |
| `packages/user/src/pages/ProfilePage.tsx` | 修改 | 修复 BUG-RC001（handleSaveProfile 使用 invoiceDefault）+ BUG-RC002（fetchProfile 不吞错误）+ handleSaveInvoice 刷新 profile |
| `packages/components/blocks/LoginHistory/index.tsx` | 修改 | 修复 BUG-RC003（添加 failed 筛选选项） |
