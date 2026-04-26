# T022 开发者中心首页 — 工程级设计方案

> 任务卡: [T022-developer-center.md](../T022-developer-center.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 标题: "开发者中心"
│   └── 副标题: "管理 API Key、查看文档、使用调试工具"
├── DetailPageShell
│   ├── ApiKeyManager (API Key 管理区)
│   │   ├── ApiKeyList
│   │   └── ApiKeyCreateForm
│   ├── DocLinks (文档链接区)
│   ├── SdkDownloads (SDK 下载区)
│   └── QuickStart (快速开始区)
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Table | Key 表格 | 否 |
| Modal | 创建弹窗 | 否 |
| Form | 表单容器 | 否 |
| Input | 文本输入 | 否 |
| Select | 下拉选择 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| ApiKeyCard | API Key 卡片 | 否 |
| CodeBlock | 代码展示 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| ApiKeyManager | Key 管理区 | 是 |
| DocLinks | 文档链接区 | 是 |
| SdkDownloads | SDK 下载区 | 是 |
| QuickStart | 快速开始区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[页面加载]
  └── 调用 GET /api/v1/developer/keys ──► 获取 API Key 列表

[用户创建 Key]
  ├── 填写创建表单
  └── 调用 POST /api/v1/developer/keys

[用户删除 Key]
  ├── 确认删除
  └── 调用 DELETE /api/v1/developer/keys/:id
```

---

## 4. 状态管理

### 页面状态

```typescript
interface DeveloperCenterState {
  // 数据状态
  apiKeys: ApiKey[];
  total: number;

  // UI 状态
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/developer/keys | 页面加载 | - | 显示错误提示 |
| POST /api/v1/developer/keys | 创建 Key | ApiKeyCreateRequest | 提示创建失败 |
| DELETE /api/v1/developer/keys/:id | 删除 Key | keyId | 提示删除失败 |

---

## 6. 用户交互流程

### 主流程

```
用户进入开发者中心
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示 API Key 列表
  │   ├── 用户创建 Key ──► 打开创建弹窗
  │   ├── 用户复制 Key ──► 复制到剪贴板
  │   └── 用户删除 Key ──► 确认删除
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 无 Key | 用户未创建 | 显示空态 | 提示创建 |
| 创建失败 | 超过数量限制 | 返回错误 | 提示删除旧 Key |
| 删除失败 | Key 正在使用 | 返回错误 | 提示确认 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 大量 Key | 支持分页 |
| Key 泄露 | 支持紧急撤销 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- Key 创建 < 500ms
- 无内存泄漏

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Key 泄露 | 低 | Key 可能被截图或泄露 | 仅展示一次，支持撤销 |
| 权限控制 | 中 | Key 权限需细粒度控制 | 细粒度权限 |
| 大量 Keys | 低 | Key 数量可能很多 | 分页展示 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 DetailPageShell 搭建页面框架

### Step 2: API Key 管理开发（1.5 人日）
- [ ] ApiKeyManager 组件
- [ ] ApiKeyList 组件
- [ ] ApiKeyCreateForm 组件

### Step 3: 文档链接区开发（0.5 人日）
- [ ] DocLinks 组件

### Step 4: SDK 下载区开发（0.5 人日）
- [ ] SdkDownloads 组件

### Step 5: 快速开始区开发（0.5 人日）
- [ ] QuickStart 组件

### Step 6: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用

### Step 7: 性能优化 & 测试（0.5 人日）
- [ ] 自测
