# 认证与注册模块（Auth System）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 认证与注册模块 |
| 任务编号 | T000 |
| 所属模块 ⭐ | 公共模块 — 认证体系 |
| 优先级 | P0 |
| 指派给 | @frontend-dev / @backend-dev |
| 预计工期 | 2-3 人日 |
| 创建日期 | 2026-04-27 |
| 目标上线日期 | MVP 前置完成 |

## 2. 需求背景

### 2.1 问题描述

当前系统已具备用户登录、注册、退出登录、认证状态维护等基础能力，但在新的“能力平台 + 任务平台 + 商业化平台”架构中，认证模块需要作为所有用户端功能的前置基础能力重新纳入任务体系。

本任务不是从零开发认证系统，而是对已有注册/登录能力进行标准化整理、补齐缺口，并确认其能支撑后续能力平台、任务系统、计费系统、API Key 管理等模块。

### 2.2 目标用户

- 终端用户 / 开发者
- 企业用户
- 平台管理员
- 后续能力平台使用者

### 2.3 预期效果

- 用户可以正常注册、登录、退出
- 登录后可访问受保护页面
- 未登录用户访问受保护页面时自动跳转登录页
- 用户身份可被能力、任务、计费、API Key 等模块复用
- AuthStore / Token / 用户信息在前端状态中稳定可用

## 3. 页面位置 ⭐

### 3.1 公共页面

| 页面 | 路由 | 入口 |
|------|------|------|
| 用户登录页 | `/login` | 未登录访问系统 / 顶部入口 |
| 用户注册页 | `/register` | 登录页「去注册」按钮 |
| 忘记密码 | 待补充 | 登录页「忘记密码」按钮，目前为占位 |

### 3.2 用户端关联页面

| 页面 | 路由 | 关系 |
|------|------|------|
| 控制台 | `/` | 登录成功后默认跳转 |
| 账户设置 | `/account` | 使用当前登录用户信息 |
| API Key | `/api-keys` | 依赖登录用户身份 |
| 任务中心 | `/tasks` 或 `/jobs` | 任务归属当前用户 |
| 计费中心 | `/billing` | 账单归属当前用户 |

## 4. 功能范围

### 4.1 包含功能

- [x] 用户登录
- [x] 用户注册
- [x] 邮箱/手机号作为登录凭证
- [x] 密码输入与校验
- [x] 注册确认密码校验
- [x] 邀请码输入，可选
- [x] 登录成功后跳转控制台
- [x] 注册成功后跳转登录页
- [x] 退出登录
- [x] 认证状态存储
- [x] 顶部头像菜单退出登录
- [x] 未登录访问受保护页面时拦截
- [ ] 忘记密码流程，目前为占位
- [ ] 注册时区分个人 / 企业类型，后续补充
- [ ] 第三方登录，后续补充
- [ ] 2FA 双因素认证，后续补充

### 4.2 不包含功能

- 企业认证审核
- 实名认证 / 资质认证
- API Key 创建
- 计费账户初始化
- 第三方 OAuth 登录
- 密码找回完整流程

## 5. 数据结构 ⭐

### 5.1 用户登录请求

```typescript
export interface LoginRequest {
  account: string; // 邮箱或手机号
  password: string;
}
~~~

### 5.2 用户注册请求

```typescript
export interface RegisterRequest {
  email: string;
  phone?: string;
  password: string;
  confirmPassword: string;
  inviteCode?: string;
}
```

### 5.3 登录响应

```typescript
export interface LoginResponse {
  token: string;
  user: CurrentUser;
}
```

### 5.4 当前用户信息

```typescript
export interface CurrentUser {
  id: string;
  tenantId?: string;
  email: string;
  phone?: string;
  contactName?: string;
  userType?: 'personal' | 'enterprise';
  role: 'user' | 'admin';
  status: 'active' | 'disabled' | 'pending';
  certStatus?: 'none' | 'pending' | 'approved' | 'rejected';
  avatarUrl?: string;
  lastLoginAt?: string;
}
```

### 5.5 前端状态

```typescript
export interface AuthState {
  token: string | null;
  user: CurrentUser | null;
  isAuthenticated: boolean;
  login: (payload: LoginResponse) => void;
  logout: () => void;
  setUser: (user: CurrentUser) => void;
}
```

## 6. API 接口 ⭐

### 6.1 用户登录

```http
POST /api/v1/auth/login
```

请求：

```json
{
  "account": "user@example.com",
  "password": "123456"
}
```

响应：

```json
{
  "data": {
    "token": "jwt-token",
    "user": {
      "id": "user_001",
      "email": "user@example.com",
      "role": "user",
      "status": "active"
    }
  }
}
```

### 6.2 用户注册

```http
POST /api/v1/auth/register
```

请求：

```json
{
  "email": "user@example.com",
  "phone": "13800000000",
  "password": "123456",
  "confirmPassword": "123456",
  "inviteCode": "INVITE001"
}
```

### 6.3 获取当前用户

```http
GET /api/v1/auth/me
```

### 6.4 退出登录

```http
POST /api/v1/auth/logout
```

## 7. 交互流程

### 7.1 登录流程

```text
用户进入 /login
→ 输入邮箱/手机号和密码
→ 点击登录
→ 前端表单校验
→ 调用登录 API
→ 成功：保存 token 和 user
→ 跳转控制台 /
→ 失败：展示错误提示
```

### 7.2 注册流程

```text
用户进入 /register
→ 输入邮箱、手机号、密码、确认密码、邀请码
→ 前端校验
→ 调用注册 API
→ 成功：提示注册成功并跳转 /login
→ 失败：展示错误提示
```

### 7.3 退出流程

```text
用户点击头像菜单「退出登录」
→ 清除 AuthStore / token
→ 跳转 /login
```

### 7.4 未登录拦截

```text
用户访问受保护页面
→ 检查 isAuthenticated
→ 未登录：跳转 /login
→ 已登录：正常进入页面
```

## 8. 异常流程

| 场景           | 触发条件        | 系统行为     | 用户反馈       |
| -------------- | --------------- | ------------ | -------------- |
| 登录失败       | 密码错误        | API 返回错误 | 显示错误提示   |
| 用户禁用       | status=disabled | 拒绝登录     | 显示账户不可用 |
| Token 过期     | API 返回 401    | 清除登录态   | 跳转登录页     |
| 注册邮箱已存在 | 后端校验失败    | 注册失败     | 显示邮箱已存在 |
| 两次密码不一致 | 前端校验失败    | 阻止提交     | 显示校验错误   |
| 邀请码无效     | 后端校验失败    | 注册失败     | 显示邀请码无效 |

## 9. 验收标准 ⭐

### 9.1 功能验收

-  `/login` 页面可以正常打开
-  `/register` 页面可以正常打开
-  用户可以使用邮箱/手机号 + 密码登录
-  登录成功后进入控制台
-  登录失败时有明确错误提示
-  用户可以完成注册
-  注册成功后跳转登录页
-  退出登录后清除登录状态
-  未登录访问受保护页面会跳转 `/login`
-  登录后可访问控制台、API Key、计费、任务等页面

### 9.2 技术验收

-  AuthStore 状态正确
-  token 持久化逻辑正确
-  API 请求自动携带 token
-  401 响应可触发退出登录
-  TypeScript 类型完整
-  不影响已有 API Key / Billing / Dashboard 页面
-  不出现明文密码存储
-  不在前端日志中输出 token

### 9.3 安全验收

-  密码输入框使用 password 类型
-  登录失败不暴露敏感错误
-  token 不出现在 URL 中
-  退出登录后无法访问受保护页面
-  管理端登录与用户端登录状态隔离

## 10. 依赖关系

| 依赖项            | 状态            | 说明                         |
| ----------------- | --------------- | ---------------------------- |
| 用户表            | 已存在 / 待确认 | 需确认字段完整               |
| 登录 API          | 已存在 / 待确认 | 需联调确认                   |
| 注册 API          | 已存在 / 待确认 | 需联调确认                   |
| AuthStore         | 已存在          | 需检查是否支持新任务平台     |
| 路由守卫          | 已存在 / 待确认 | 需验证受保护路由             |
| API Client 拦截器 | 已存在 / 待确认 | 需验证 token 注入与 401 处理 |

## 11. 是否影响现网

| 项目         | 说明                                                     |
| ------------ | -------------------------------------------------------- |
| 是否影响现网 | 是                                                       |
| 影响范围     | 登录、注册、受保护路由、API 请求鉴权                     |
| 风险等级     | 高                                                       |
| 风险说明     | 认证模块是全站入口，修改不当会导致用户无法登录或访问页面 |
| 应对措施     | 必须做回归测试、登录态测试、401 测试、路由守卫测试       |

## 12. 后续优化项

- 忘记密码流程
- 邮箱验证码注册
- 手机验证码登录
- 企业 / 个人注册类型选择
- 第三方登录
- 2FA 双因素认证
- 登录安全日志增强
- 登录异常风控
- Session 管理