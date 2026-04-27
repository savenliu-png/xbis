# T030 管理端用户管理 - 验收检查文档

## 1. 修改文件清单

| 文件路径 | 修改类型 | 说明 |
|----------|----------|------|
| `packages/shared/src/types/admin-user.ts` | 新增 | 管理端用户管理类型定义 |
| `packages/shared/src/types/index.ts` | 修改 | 添加 admin-user 类型导出 |
| `packages/shared/src/api/services.ts` | 修改 | 更新 adminApi.users 使用类型化参数，新增 updateLimits/updatePermissions |
| `packages/components/business/UserCard/index.tsx` | 新增 | 用户卡片业务组件 |
| `packages/components/business/index.ts` | 修改 | 注册 UserCard 导出 |
| `packages/components/blocks/UserTable/index.tsx` | 新增 | 用户表格区块组件（含禁用确认弹窗） |
| `packages/components/blocks/UserDetailPanel/index.tsx` | 新增 | 用户详情面板区块组件（含5个Tab） |
| `packages/components/blocks/OrderList/index.tsx` | 新增 | 用户订单列表区块组件 |
| `packages/components/blocks/LimitConfig/index.tsx` | 新增 | 用户限制配置表单区块组件 |
| `packages/components/blocks/PermissionPanel/index.tsx` | 新增 | 用户权限管理区块组件 |
| `packages/components/blocks/index.ts` | 修改 | 注册5个新区块组件导出 |
| `packages/pages/admin/UserManagement/UserManagementPage.tsx` | 新增 | 用户管理页面主组件 |
| `packages/pages/admin/UserManagement/hooks/useUserManagement.ts` | 新增 | 用户管理自定义 Hook |
| `packages/pages/admin/UserManagement/index.ts` | 新增 | 页面导出入口 |
| `packages/pages/admin/index.ts` | 修改 | 添加 UserManagement 导出 |
| `packages/admin/src/App.tsx` | 修改 | 路由切换到 UserManagementPage |

## 2. 新增组件清单

| 组件名称 | 所属层级 | 说明 |
|----------|----------|------|
| UserCard | business | 用户卡片，展示头像/名称/状态/角色/联系信息，含数据脱敏 |
| UserTable | blocks | 用户表格，含脱敏展示、Switch状态切换、禁用确认弹窗（含原因/时长/解禁时间） |
| UserDetailPanel | blocks | 用户详情面板，含5个Tab（基本信息/订单列表/最近任务/限制配置/权限管理） |
| OrderList | blocks | 用户订单列表，展示订单号/套餐/金额/状态/时间 |
| LimitConfig | blocks | 用户限制配置表单，5项配置（每分钟调用/每日调用/并发数/允许能力/IP白名单） |
| PermissionPanel | blocks | 用户权限管理，角色分配/能力访问/API调用权限 |

## 3. API变更清单

| API | Method | Path | 变更类型 | 说明 |
|-----|--------|------|----------|------|
| 用户列表 | GET | `/admin-api/v1/users` | 修改 | 参数类型从 `any` 改为 `AdminUserListParams` |
| 用户详情 | GET | `/admin-api/v1/users/:id` | 修改 | 路径参数使用 `encodePathParam` |
| 状态更新 | PUT | `/admin-api/v1/users/:id/status` | 修改 | Method 从 POST 改为 PUT，参数类型改为 `AdminUserStatusUpdateRequest` |
| 限制配置 | PUT | `/admin-api/v1/users/:id/limits` | 新增 | 参数类型 `AdminUserLimitsUpdateRequest` |
| 权限管理 | PUT | `/admin-api/v1/users/:id/permissions` | 新增 | 参数类型 `AdminUserPermissionsUpdateRequest` |

## 4. 风险说明

| 风险项 | 等级 | 说明 | 应对方案 |
|--------|------|------|----------|
| API 方法变更 | 中 | setStatus 从 POST 改为 PUT，需后端同步 | 联调时确认后端支持 |
| 旧页面保留 | 低 | Users.tsx 和 UserDetail.tsx 保留但不再被路由引用 | 后续版本清理 |
| 数据脱敏 | 低 | 前端脱敏仅为展示层，API 返回完整数据 | 确认后端是否需要脱敏 |

## 5. 自检结果

### C5 AI 强制自检

| 检查项 | 结果 |
|--------|------|
| 类型安全（无 any） | ✅ Pass |
| 无未使用导入 | ✅ Pass |
| Design Tokens 使用 | ✅ Pass |
| 页面模板使用 | ✅ Pass |
| 状态机完整 | ✅ Pass |
| 错误处理 | ✅ Pass |

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

## 6. 验收结果

👉 **通过** — 待联调

- 页面可正常打开（路由已配置）
- 主流程完整：列表→筛选→分页→详情→禁用/恢复→限制配置→权限管理
- API 调用通过 Business Services 层（adminApi）
- 无 TypeScript 类型错误
- UI 使用 Design Tokens，遵循页面模板
- 不影响旧功能（旧文件保留，路由切换）
