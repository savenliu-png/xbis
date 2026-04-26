# T023 个人中心 — 最终可开发设计方案

> 任务卡: [T023-profile-page.md](../T023-profile-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "个人中心"
    └── FormPageShell
        ├── ProfileForm
        │   └── 基本信息表单
        ├── SecuritySettings
        │   └── 安全设置
        ├── PreferenceSettings
        │   └── 偏好设置
        └── InvoiceSettings
            └── 发票信息
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 保存按钮 |
| Input | 输入框 |
| Form | 表单容器 |
| Upload | 头像上传 |
| Avatar | 头像 |
| Switch | 布尔切换 |
| Select | 下拉选择 |
| Tabs | 内容切换 |
| Skeleton | 加载骨架 |
| Empty | 空态 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| ProfileCard | 个人信息卡片 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| ProfileForm | 基本信息 | **新建** |
| SecuritySettings | 安全设置 | **新建** — 含功能项定义 |
| PreferenceSettings | 偏好设置 | **新建** — 含配置项定义 |
| InvoiceSettings | 发票信息 | **新建** — 与 T021 边界明确 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[进入 /profile]
    │
    ▼
[API: GET /api/v1/user/profile]
    │
    ▼
[渲染个人中心]
```

---

## 4. 状态管理

```typescript
interface ProfileState {
  pageState: 'loading' | 'idle' | 'error';
  profile: UserProfile | null;
  securitySettings: SecuritySettings;
  preferenceSettings: PreferenceSettings;
  invoiceSettings: InvoiceSettings;
  error: ApiError | null;
}

// 安全设置功能项
interface SecuritySettings {
  passwordLastChanged: string;
  twoFactorEnabled: boolean;
  phoneBound: boolean;
  emailBound: boolean;
  loginHistory: LoginRecord[];
}

// 偏好设置项
interface PreferenceSettings {
  language: 'zh-CN' | 'en-US';
  theme: 'light' | 'dark' | 'system';
  emailNotifications: boolean;
  smsNotifications: boolean;
}

// 发票设置（与 T021 边界）
interface InvoiceSettings {
  type: 'personal' | 'enterprise';
  title: string;
  taxNumber?: string;
  email: string;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 查询资料 | GET | `/api/v1/user/profile` | 查询个人资料 |
| 更新资料 | PUT | `/api/v1/user/profile` | 更新个人资料 |
| 上传头像 | POST | `/api/v1/user/avatar` | 上传头像 |

---

## 6. 用户交互流程

```
[进入个人中心]
    │
    ▼
[浏览/编辑基本信息]
    │
    ▼
[安全设置]
    │
    ├── 修改密码
    ├── 绑定/解绑手机
    ├── 绑定/解绑邮箱
    └── 查看登录历史
    │
    ▼
[偏好设置]
    │
    ├── 切换语言
    ├── 切换主题
    └── 通知设置
    │
    ▼
[发票信息]
    │
    └── 设置默认发票信息（用于 T021 发票申请时自动填充）
```

### 发票信息边界

```
- 个人中心 InvoiceSettings：保存用户默认发票信息（抬头、税号等）
- T021 发票申请：使用默认信息填充，支持每次修改
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 头像上传失败 | 提示错误，保留原头像 |
| 保存失败 | 提示错误，保留表单数据 |

---

## 8. 性能优化

- 头像压缩上传
- 表单数据缓存

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 信息泄露 | 中 | 个人敏感信息 | 权限控制 |

---

## 10. 开发步骤拆分

### Step 1: 基本信息（0.5 天）
- [ ] ProfileForm

### Step 2: 安全与偏好（0.5 天）
- [ ] SecuritySettings（含功能项）
- [ ] PreferenceSettings（含配置项）

### Step 3: 发票信息（0.5 天）
- [ ] InvoiceSettings（明确与 T021 边界）
