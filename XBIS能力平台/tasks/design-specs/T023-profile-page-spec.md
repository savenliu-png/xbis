# T023 个人中心 — 工程级设计方案

> 任务卡: [T023-profile-page.md](../T023-profile-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 标题: "个人中心"
│   └── 返回按钮
├── FormPageShell
│   ├── Tabs
│   │   ├── TabPane: 账户信息
│   │   ├── TabPane: 安全设置
│   │   ├── TabPane: 偏好设置
│   │   └── TabPane: 发票信息
│   ├── TabContent
│   │   ├── ProfileForm (账户信息表单)
│   │   ├── SecuritySettings (安全设置)
│   │   ├── PreferenceSettings (偏好设置)
│   │   └── InvoiceSettings (发票设置)
│   └── ActionBar (保存/取消)
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Card | 信息卡片 | 否 |
| Form | 表单容器 | 否 |
| Input | 文本输入 | 否 |
| Select | 下拉选择 | 否 |
| Switch | 布尔切换 | 否 |
| Avatar | 头像 | 否 |
| Upload | 文件上传 | 否 |
| Tabs | 内容切换 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| ProfileCard | 个人信息卡片 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| ProfileForm | 个人信息表单 | 是 |
| SecuritySettings | 安全设置区 | 是 |
| PreferenceSettings | 偏好设置区 | 是 |
| InvoiceSettings | 发票设置区 | 是 |
| LoginHistory | 登录历史区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[页面加载]
  ├── 调用 GET /api/v1/user/profile ──► 获取用户信息
  └── 调用 GET /api/v1/user/security ──► 获取安全设置

[用户编辑信息]
  ├── 填写表单 ──► 更新 formData
  └── 点击保存 ──► 调用 PUT /api/v1/user/profile

[用户上传头像]
  ├── 选择文件
  └── 调用 POST /api/v1/user/avatar
```

---

## 4. 状态管理

### 页面状态

```typescript
interface ProfileState {
  // 数据状态
  profile?: UserProfile;
  security?: SecuritySettings;

  // 表单状态
  formData: ProfileUpdateRequest;

  // UI 状态
  status: 'idle' | 'loading' | 'saving' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/user/profile | 页面加载 | - | 显示错误提示 |
| GET /api/v1/user/security | 页面加载 | - | 显示错误提示 |
| PUT /api/v1/user/profile | 保存信息 | ProfileUpdateRequest | 提示保存失败 |
| POST /api/v1/user/avatar | 上传头像 | FormData | 提示上传失败 |

---

## 6. 用户交互流程

### 主流程

```
用户进入个人中心
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示用户信息
  │   ├── 用户编辑信息 ──► 填写表单
  │   ├── 用户上传头像 ──► 选择文件
  │   └── 用户保存 ──► 提交表单
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 保存失败 | 网络错误 | 返回错误 | 提示重试 |
| 头像上传失败 | 文件过大 | 返回错误 | 提示限制 |
| 邮箱已存在 | 重复邮箱 | 返回错误 | 提示更换 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 头像上传 | 限制 2MB |
| 表单校验 | 实时校验 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 保存响应 < 500ms
- 头像上传 < 3s

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 数据迁移 | 中 | 需迁移旧账户数据 | 保留旧数据 |
| 功能开关 | 低 | 需灰度发布 | 功能开关控制 |
| 头像存储 | 低 | 头像需 CDN 加速 | CDN 加速 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 FormPageShell 搭建页面框架
- [ ] 配置 Tabs

### Step 2: 账户信息开发（1 人日）
- [ ] ProfileForm 组件
- [ ] 头像上传

### Step 3: 安全设置开发（0.5 人日）
- [ ] SecuritySettings 组件

### Step 4: 偏好设置开发（0.5 人日）
- [ ] PreferenceSettings 组件

### Step 5: 发票设置开发（0.5 人日）
- [ ] InvoiceSettings 组件

### Step 6: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用

### Step 7: 性能优化 & 测试（0.5 人日）
- [ ] 自测
