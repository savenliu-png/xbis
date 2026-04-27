# T023 个人中心 — 回归测试与功能验收报告

## 一、影响范围

| 影响类型 | 影响对象 | 风险等级 | 是否需要回归 |
|----------|----------|----------|-------------|
| 页面 | 新增 `/profile` 页面 | 高 | 是 |
| 页面 | 旧 `/account` 页面 | 低 | 是（确认未破坏） |
| 组件 | 新增 5 个 blocks 组件 | 高 | 是 |
| 组件 | 新增 2 个 business 组件（InfoRow/FormField） | 中 | 是 |
| 组件 | 修改 Layout.tsx 用户下拉菜单 | 中 | 是 |
| API | services.ts 参数类型强化（any → 强类型） | 低 | 是（运行时不变） |
| 类型 | types/index.ts 新增 8+ 类型定义 | 低 | 是（编译时） |
| 常量 | constants/index.ts 新增 PROFILE 路由 | 低 | 是 |
| 导出 | blocks/index.ts 新增 5 个导出 | 低 | 是 |
| 导出 | business/index.ts 新增 2 个导出 | 低 | 是 |
| 样式 | Design Tokens 使用（无硬编码） | 低 | 是 |
| 权限 | RequireAuth 守卫（与 /account 一致） | 低 | 是 |

## 二、回归测试清单

### 2.1 主流程回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 页面是否正常打开 | ✅ | `/profile` 路由已注册，RequireAuth 守卫 |
| Tab 切换是否正常 | ✅ | 5 个 Tab 切换正常，内容按需渲染 |
| 账户信息查看 | ✅ | 头像+标签+基本信息网格 |
| 账户信息编辑保存 | ✅ | 编辑模式切换，邮箱格式校验，保存后刷新 |
| 安全设置查看 | ✅ | 密码管理+二次验证+绑定信息+会话管理 |
| 修改密码 | ✅ | Modal 弹窗，强度校验，确认一致性 |
| 偏好设置查看/保存 | ✅ | 默认配置+通知设置，回调地址校验 |
| 发票信息查看/设置/修改 | ✅ | 个人/企业切换，条件字段，空态引导 |
| 登录历史查看 | ✅ | 筛选+分页+空态+安全提示 |
| 用户下拉菜单入口 | ✅ | "个人中心 → /profile" |

### 2.2 API / 数据回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| API 调用是否通过 Business Services 层 | ✅ | 全部通过 userApi |
| 请求参数类型是否正确 | ✅ | ProfileUpdateRequest/ProfilePreferenceUpdateRequest/ProfileLoginLogParams |
| 响应字段映射是否正确 | ✅ | extractApiData<T> 统一解析 |
| 错误响应是否正确处理 | ✅ | 页面级 error+重试，组件级 Alert |
| 发票信息解析是否正确 | ✅ | parseInvoiceFromRemark 兼容新旧字段名 |
| 保存 profile 时是否保留发票 JSON | ✅ | handleSaveProfile 合并 internalRemark |

### 2.3 UI / 响应式回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 桌面端 ≥1200px | ✅ | maxWidth=1100 + 响应式 grid |
| 移动端核心流程 | ✅ | MobileNav + 响应式布局 |
| 表格/卡片/弹窗是否错位 | ✅ | Design Tokens 统一间距 |
| 暗色模式支持 | ✅ | CSS 变量 + semanticColors |

### 2.4 状态回归

| 组件 | loading | empty | error | disabled |
|------|---------|-------|-------|----------|
| ProfilePage | ✅ Skeleton | — | ✅ Alert+重试 | — |
| ProfileForm | — | — | ✅ Alert | ✅ saving |
| SecuritySettings | ✅ Skeleton | — | ✅ Modal Alert | ✅ changingPassword |
| PreferenceSettings | ✅ Skeleton | — | ✅ Alert | ✅ saving |
| InvoiceSettings | — | ✅ 空态引导 | ✅ Alert | ✅ saving |
| LoginHistory | ✅ Spinner | ✅ Empty | ✅ 错误态+重试 | — |
| Tab 懒加载失败 | ✅ | — | ✅ Alert+重试按钮 | — |

### 2.5 旧功能回归

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 旧 /account 页面是否正常 | ✅ | 路由保留，页面未删除 |
| 旧路由是否可访问 | ✅ | ROUTES.USER.ACCOUNT 仍存在 |
| 侧边栏导航是否正常 | ✅ | NAV_ITEMS 包含 profile 项 |
| 其他页面路由是否正常 | ✅ | 未修改其他路由 |
| API 服务层是否破坏 | ✅ | 仅 any → 强类型，运行时不变 |

## 三、发现的 Bug

| Bug编号 | 严重级别 | 问题描述 | 复现路径 | 影响范围 | 修复建议 |
|---------|----------|----------|----------|----------|----------|
| BUG-001 | High | InvoiceSettings displayRows 在 invoiceDefault 为 null 时使用 `invoiceDefault!` 非空断言，组件顶层执行导致运行时崩溃 | 打开发票信息 Tab，invoiceDefault 为 null | 发票信息 Tab 白屏 | 将 displayRows 移入条件渲染内部 |
| BUG-002 | Medium | LoginHistory 中 filtersRef 声明但未使用 | 代码审查 | 代码质量 | 删除未使用的 ref |
| BUG-003 | Medium | Tab 懒加载时 security/preferences API 失败后显示永久 Skeleton，无错误提示 | 切换到安全设置/偏好设置 Tab，API 失败 | 用户无法知道加载失败 | 添加错误状态和重试按钮 |
| BUG-004 | Medium | ProfileForm 编辑表单包含 internalRemark 字段，用户编辑备注可能覆盖发票 JSON 数据 | 编辑账户信息，修改备注说明，保存 | 发票默认信息丢失 | 从编辑表单中移除 internalRemark 字段 |

## 四、Bug 修复内容

### BUG-001：InvoiceSettings displayRows 空指针

**问题原因**：`displayRows` 数组在组件顶层定义，使用 `invoiceDefault!` 非空断言。当 `invoiceDefault` 为 null 时，访问 `invoiceDefault!.invoiceType` 导致运行时崩溃。

**修复方案**：将 `displayRows` 移入 `invoiceDefault && invoiceDefault.title` 条件渲染内部，使用 IIFE 包裹。

**修改文件**：`packages/components/blocks/InvoiceSettings/index.tsx`

**修复代码**：
```tsx
// 修复前：组件顶层
const displayRows = [
  { label: '发票类型', value: <Tag>{invoiceDefault!.invoiceType}</Tag> }, // 💥 null 时崩溃
  ...
];

// 修复后：条件渲染内部
{invoiceDefault && invoiceDefault.title ? (
  (() => {
    const displayRows = [
      { label: '发票类型', value: <Tag>{invoiceDefault.invoiceType}</Tag> }, // ✅ 安全
      ...
    ];
    const visibleRows = displayRows.filter(r => r.show !== false);
    return (
      <div>
        {visibleRows.map((row, idx) => (
          <InfoRow key={row.label} label={row.label} showBorder={idx < visibleRows.length - 1}>
            {row.value}
          </InfoRow>
        ))}
      </div>
    );
  })()
) : ...}
```

### BUG-002：LoginHistory 未使用的 filtersRef

**问题原因**：`filtersRef` 在优化过程中声明但实际未使用，属于死代码。

**修复方案**：删除 `filtersRef` 声明和赋值，删除 `useRef` 导入。

**修改文件**：`packages/components/blocks/LoginHistory/index.tsx`

### BUG-003：Tab 懒加载 API 失败无错误提示

**问题原因**：`fetchSecurity`/`fetchPreferences` 失败时仅 `console.error`，用户看到永久 Skeleton 无反馈。

**修复方案**：添加 `securityError`/`preferencesError` 状态，API 失败时设置错误消息，tabContent 中显示 Alert + 重试按钮。

**修改文件**：`packages/user/src/pages/ProfilePage.tsx`

**修复代码**：
```tsx
const [securityError, setSecurityError] = useState<string | null>(null);
const [preferencesError, setPreferencesError] = useState<string | null>(null);

const fetchSecurity = useCallback(async () => {
  setSecurityError(null);
  try { ... } catch { setSecurityError('加载安全设置失败'); }
}, []);

// tabContent 中
case 'security':
  return securityError ? (
    <Card variant="default" padding="md">
      <Alert variant="error" message={securityError} />
      <div style={{ textAlign: 'center', marginTop: space['4'] }}>
        <Button variant="primary" onClick={fetchSecurity}>重试</Button>
      </div>
    </Card>
  ) : (
    <SecuritySettings security={security} onChangePassword={handleChangePassword} />
  );
```

### BUG-004：ProfileForm 备注字段与发票数据冲突

**问题原因**：`internalRemark` 字段用于存储发票默认信息 JSON，但在编辑表单中暴露为"备注说明"输入框，用户编辑会覆盖发票 JSON。

**修复方案**：从编辑表单和展示模式中移除 `internalRemark` 字段。该字段是内部存储字段，不应由用户直接编辑。

**修改文件**：`packages/components/blocks/ProfileForm/index.tsx`

## 五、复测结果

| 测试项 | 结果 | 说明 |
|--------|------|------|
| BUG-001 复测：InvoiceSettings invoiceDefault 为 null | ✅ | displayRows 在条件内部，不再崩溃 |
| BUG-001 复测：InvoiceSettings invoiceDefault 有值 | ✅ | InfoRow 正确渲染，showBorder 正确 |
| BUG-002 复测：LoginHistory 编译 | ✅ | 无未使用变量警告 |
| BUG-003 复测：安全设置 API 失败 | ✅ | 显示 Alert + 重试按钮 |
| BUG-003 复测：偏好设置 API 失败 | ✅ | 显示 Alert + 重试按钮 |
| BUG-003 复测：重试按钮点击 | ✅ | 重新调用 fetchSecurity/fetchPreferences |
| BUG-004 复测：编辑账户信息 | ✅ | 无 internalRemark 字段，不会覆盖发票数据 |
| BUG-004 复测：保存后发票信息 | ✅ | handleSaveProfile 合并 internalRemark 保留发票 JSON |
| 主流程回归：5 个 Tab 切换 | ✅ | 正常 |
| 旧功能回归：/account 页面 | ✅ | 未受影响 |
| TypeScript 编译 | ✅ | 新代码无错误 |

## 六、功能验收结论

👉 **Accepted with notes**

### 验收标准对照

| 验收项 | 状态 | 说明 |
|--------|------|------|
| 账户信息展示正常 | ✅ | |
| 账户信息编辑正常 | ✅ | |
| 头像上传正常 | ⚠️ | V2 迭代：当前仅 URL 输入 |
| 安全设置展示正常 | ✅ | |
| 偏好设置保存正常 | ✅ | |
| 发票信息管理正常 | ✅ | |
| 登录历史展示正常 | ✅ | |
| 空态/加载态/错误态完整 | ✅ | 含 Tab 懒加载失败态 |
| TypeScript 类型完整 | ✅ | |
| Design Tokens 使用 | ✅ | |
| 页面模板骨架 | ✅ | FormPageShell |
| 组件分层规范 | ✅ | base → business → blocks → layout |
| API 错误处理完整 | ✅ | |
| 旧功能未受影响 | ✅ | |

### Notes

1. **头像上传**：当前仅支持 URL 输入，V2 迭代补充 Upload 组件 + 专用 API
2. **主题切换**：任务卡中列为偏好设置项，V2 迭代补充
3. **发票专用 API**：当前通过 `internalRemark` JSON 序列化，V2 迭代升级
4. **登录日志字段命名**：`login_result: 'failure'` vs `'failed'` 待联调确认

## 七、是否允许合并

👉 **YES**

- ✅ 无 Blocker / High Bug 未修复
- ✅ 所有 Medium Bug 已修复
- ✅ 回归测试全部通过
- ✅ TypeScript 编译通过
- ✅ 旧功能未受影响
- ✅ 允许进入下一任务
- ⚠️ 需后端联调确认：登录日志字段命名风格
