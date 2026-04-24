# 个人中心

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 个人中心 |
| 任务编号 | T023 |
| 所属模块 ⭐ | M8 个人中心 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目有个人中心页面，但需要改造以支持：
- 账户信息管理
- 安全设置
- 偏好设置
- 发票管理

### 2.2 目标用户
- 终端用户：管理个人信息
- 管理员：管理用户

### 2.3 预期效果
- 提供统一的个人中心
- 支持信息修改
- 支持安全设置

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 个人中心 | `/profile` | 主导航「个人中心」 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：现有 `/account` 页面

## 4. 功能范围

### 4.1 包含功能
- [ ] 账户信息展示和编辑
- [ ] 头像上传
- [ ] 安全设置（密码/手机/邮箱）
- [ ] 偏好设置（主题/语言/通知）
- [ ] 发票信息管理
- [ ] 登录历史
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 实名认证（V2 迭代）
- 第三方账号绑定（V2 迭代）
- 账号注销（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 用户信息
export interface UserProfile {
  id: string;
  username: string;
  email: string;
  phone?: string;
  avatar?: string;
  displayName: string;
  company?: string;
  title?: string;
  bio?: string;
  language: 'zh' | 'en';
  theme: 'light' | 'dark' | 'system';
  notificationSettings: NotificationSettings;
  createdAt: string;
  updatedAt: string;
}

// 通知设置
export interface NotificationSettings {
  email: boolean;
  sms: boolean;
  push: boolean;
  marketing: boolean;
  taskCompletion: boolean;
  billingAlert: boolean;
}

// 安全设置
export interface SecuritySettings {
  passwordLastChanged: string;
  twoFactorEnabled: boolean;
  loginHistory: LoginHistoryItem[];
}

// 登录历史
export interface LoginHistoryItem {
  id: string;
  ip: string;
  device: string;
  browser: string;
  location?: string;
  loggedInAt: string;
}

// 发票信息
export interface InvoiceInfo {
  type: 'personal' | 'enterprise';
  title: string;
  taxNumber?: string;
  address?: string;
  phone?: string;
  bankName?: string;
  bankAccount?: string;
}

// 更新请求
export interface ProfileUpdateRequest {
  displayName?: string;
  company?: string;
  title?: string;
  bio?: string;
  language?: 'zh' | 'en';
  theme?: 'light' | 'dark' | 'system';
  notificationSettings?: Partial<NotificationSettings>;
  invoiceInfo?: InvoiceInfo;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`UserProfile`, `NotificationSettings`, `SecuritySettings`, `LoginHistoryItem`, `InvoiceInfo`, `ProfileUpdateRequest`

## 6. 交互流程

### 6.1 主流程
```
[用户进入个人中心] ──► [加载用户信息] ──► [展示信息] ──► [用户编辑] ──► [保存修改]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 保存失败 | 网络错误 | 返回错误 | 提示重试 |
| 头像上传失败 | 文件过大 | 返回错误 | 提示限制 |
| 邮箱已存在 | 重复邮箱 | 返回错误 | 提示更换 |

### 6.3 边界情况
- 头像上传：限制 2MB
- 表单校验：实时校验
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 信息卡片 | 是（T002） |
| Form | 表单容器 | 是（T002） |
| Input | 文本输入 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Switch | 布尔切换 | 是（T002） |
| Avatar | 头像 | 是（T002） |
| Upload | 文件上传 | 是（T002） |
| Tabs | 内容切换 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| ProfileCard | 个人信息卡片 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| ProfileForm | 个人信息表单 | 否 |
| SecuritySettings | 安全设置区 | 否 |
| PreferenceSettings | 偏好设置区 | 否 |
| InvoiceSettings | 发票设置区 | 否 |
| LoginHistory | 登录历史区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| ProfileCard | business | 个人信息卡片 |
| ProfileForm | blocks | 个人信息表单 |
| SecuritySettings | blocks | 安全设置区 |
| PreferenceSettings | blocks | 偏好设置区 |
| InvoiceSettings | blocks | 发票设置区 |
| LoginHistory | blocks | 登录历史区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 用户信息查询
- **Method**: GET
- **Path**: `/api/v1/user/profile`
- **响应类型**: `UserProfile`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": "user-001",
    "username": "developer",
    "email": "dev@example.com",
    "phone": "13800138000",
    "avatar": "https://cdn.example.com/avatar.jpg",
    "displayName": "开发者",
    "company": "某某科技",
    "title": "高级工程师",
    "bio": "热爱技术",
    "language": "zh",
    "theme": "system",
    "notificationSettings": {
      "email": true,
      "sms": false,
      "push": true,
      "marketing": false,
      "taskCompletion": true,
      "billingAlert": true
    },
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-04-24T10:00:00Z"
  }
}
```

#### 用户信息更新
- **Method**: PUT
- **Path**: `/api/v1/user/profile`
- **请求类型**: `ProfileUpdateRequest`
- **响应类型**: `UserProfile`
- **权限**: 用户（已登录）

#### 头像上传
- **Method**: POST
- **Path**: `/api/v1/user/avatar`
- **请求类型**: `FormData`
- **响应类型**: `{ avatarUrl: string }`
- **权限**: 用户（已登录）

#### 安全设置查询
- **Method**: GET
- **Path**: `/api/v1/user/security`
- **响应类型**: `SecuritySettings`
- **权限**: 用户（已登录）

#### 登录历史查询
- **Method**: GET
- **Path**: `/api/v1/user/login-history`
- **响应类型**: `{ items: LoginHistoryItem[]; total: number }`
- **权限**: 用户（已登录）

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.user.profile`, `userApi.user.updateProfile`, `userApi.user.uploadAvatar`, `userApi.user.security`, `userApi.user.loginHistory`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 账户信息展示正常
- [ ] 账户信息编辑正常
- [ ] 头像上传正常
- [ ] 安全设置展示正常
- [ ] 偏好设置保存正常
- [ ] 发票信息管理正常
- [ ] 登录历史展示正常
- [ ] 空态/加载态/错误态完整

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用页面模板骨架
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 首屏加载 < 2s
- [ ] 保存响应 < 500ms
- [ ] 头像上传 < 3s
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **是** — 影响范围：原 `/account` 页面改造
  - 影响页面：`/account`
  - 影响用户：全部
  - 回滚方案：保留旧页面，功能开关控制

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 数据迁移 | 中 | 中 | 保留旧数据 |
| 功能开关 | 低 | 高 | 灰度发布 |
| 头像存储 | 低 | 中 | CDN 加速 |

## 11. 开发检查清单

### 11.1 开发前
- [ ] 需求已评审并确认
- [ ] 设计稿已确认
- [ ] 数据结构和 API 已评审
- [ ] 依赖任务已完成

### 11.2 开发中
- [ ] 遵循 `docs/dev-rules.md` 规范
- [ ] 遵循 `AI_RULES.md` 规范
- [ ] 自测覆盖所有验收标准

### 11.3 开发后
- [ ] 代码已提交 PR
- [ ] PR 已通过 Code Review
- [ ] 已联调通过
- [ ] 已更新文档（如需要）

## 12. 备注

- 改造现有 `/account` 页面
- 保留旧页面，功能开关控制
- 支持主题切换（light/dark/system）

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
