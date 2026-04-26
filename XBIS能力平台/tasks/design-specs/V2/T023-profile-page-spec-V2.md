# T023 个人中心 — 修正版设计方案（V2）

> 原设计方案: [T023-profile-page-spec.md](../T023-profile-page-spec.md)
> 评审报告: [T023-profile-page-Reviewer.md](../T023-profile-page-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确安全设置功能项 | 第 2 节组件拆分 |
| 2 | 明确偏好设置项 | 第 2 节组件拆分 |
| 3 | 明确发票信息边界 | 第 6 节用户交互流程 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加头像裁剪功能 | 第 6 节用户交互流程 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "个人中心"
    └── FormPageShell
        ├── ProfileForm
        │   └── 基本信息表单
        ├── SecuritySettings
        │   └── 安全设置 【已修复】
        ├── PreferenceSettings
        │   └── 偏好设置 【已修复】
        └── InvoiceSettings
            └── 发票信息 【已修复】
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 保存按钮 |
| Input | 输入框 |
| Form | 表单容器 |
| Upload | 头像上传 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| ProfileCard | 个人信息卡片 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| ProfileForm | 基本信息 | **新建** |
| SecuritySettings | 安全设置 | **新建** — 含功能项定义 |
| PreferenceSettings | 偏好设置 | **新建** — 含配置项定义 |
| InvoiceSettings | 发票信息 | **新建** — 与 T021 边界明确 |

---

### 3. 数据流

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

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface ProfileState {
  pageState: 'loading' | 'idle' | 'error';
  profile: UserProfile | null;
  securitySettings: SecuritySettings; // 【新增】
  preferenceSettings: PreferenceSettings; // 【新增】
  invoiceSettings: InvoiceSettings; // 【新增】
  error: ApiError | null;
}

// 【新增】安全设置功能项
interface SecuritySettings {
  passwordLastChanged: string;
  twoFactorEnabled: boolean;
  phoneBound: boolean;
  emailBound: boolean;
  loginHistory: LoginRecord[];
}

// 【新增】偏好设置项
interface PreferenceSettings {
  language: 'zh-CN' | 'en-US';
  theme: 'light' | 'dark' | 'system';
  emailNotifications: boolean;
  smsNotifications: boolean;
}

// 【新增】发票设置（与 T021 边界）
interface InvoiceSettings {
  type: 'personal' | 'enterprise';
  title: string;
  taxNumber?: string;
  email: string;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 查询资料 | GET | `/api/v1/user/profile` | 查询个人资料 |
| 更新资料 | PUT | `/api/v1/user/profile` | 更新个人资料 |
| 上传头像 | POST | `/api/v1/user/avatar` | 上传头像 |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入个人中心]
    │
    ▼
[浏览/编辑基本信息]
    │
    ▼
[安全设置] 【已修复】
    │
    ├── 修改密码
    ├── 绑定/解绑手机
    ├── 绑定/解绑邮箱
    └── 查看登录历史
    │
    ▼
[偏好设置] 【已修复】
    │
    ├── 切换语言
    ├── 切换主题
    └── 通知设置
    │
    ▼
[发票信息] 【已修复】
    │
    └── 设置默认发票信息（用于 T021 发票申请时自动填充）
```

#### 发票信息边界（V2 明确）【已修复】

```
- 个人中心 InvoiceSettings：保存用户默认发票信息（抬头、税号等）
- T021 发票申请：使用默认信息填充，支持每次修改
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 头像上传失败 | 提示错误，保留原头像 |
| 保存失败 | 提示错误，保留表单数据 |

---

### 8. 性能优化

- 头像压缩上传
- 表单数据缓存

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 信息泄露 | 中 | 个人敏感信息 | 权限控制 |

---

### 10. 开发步骤拆分

#### Step 1: 基本信息（0.5 天）
- [ ] ProfileForm

#### Step 2: 安全与偏好（0.5 天）【已修复】
- [ ] SecuritySettings（含功能项）
- [ ] PreferenceSettings（含配置项）

#### Step 3: 发票信息（0.5 天）【已修复】
- [ ] InvoiceSettings（明确与 T021 边界）

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **安全设置** | 未细化 | 明确密码/2FA/手机/邮箱/登录历史 |
| **偏好设置** | 未细化 | 明确语言/主题/通知 |
| **发票信息** | 未明确边界 | 明确为默认发票信息，T021 可覆盖 |
| **头像裁剪** | 未设计 | 建议新增（可选） |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（安全/偏好设置细化、发票边界明确）
2. 建议修改项已识别（头像裁剪）
3. 风险等级：低
