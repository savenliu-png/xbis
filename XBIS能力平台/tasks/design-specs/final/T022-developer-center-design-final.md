# T022 开发者中心首页 — 最终可开发设计方案

> 任务卡: [T022-developer-center.md](../T022-developer-center.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "开发者中心"
    └── DetailPageShell
        ├── ApiKeyManager
        │   ├── Key 列表
        │   └── 创建 Key
        ├── QuickStart
        │   └── 快速开始代码
        ├── SdkDownloads
        │   └── SDK 下载
        └── DocLinks
            └── 文档链接
```

---

## 2. 组件拆分

### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Table | Key 列表 |
| Modal | 创建 Key 弹窗 |
| Form | 表单容器 |
| Input | 文本输入 |
| Select | 下拉选择 |
| Skeleton | 加载骨架 |
| Empty | 空态 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| ApiKeyItem | Key 列表项 |
| CodeBlock | 代码展示 |

### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| ApiKeyManager | Key 管理 | **新建** |
| QuickStart | 快速开始 | **新建** |
| SdkDownloads | SDK 下载 | **新建** |
| DocLinks | 文档链接 | **新建** |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[进入 /developer]
    │
    ▼
[API: GET /api/v1/developer/keys]
    │
    ▼
[渲染开发者中心]
```

---

## 4. 状态管理

```typescript
interface DeveloperCenterState {
  pageState: 'loading' | 'idle' | 'error';
  keys: ApiKey[];
  createModalOpen: boolean;
  newKey: string | null; // 新创建的 Key（仅展示一次）
  keyConfirmed: boolean; // 用户是否已确认保存
  error: ApiError | null;
}

// 权限选项
const permissionOptions = [
  { value: 'read', label: '只读', description: '只能查询任务状态' },
  { value: 'write', label: '读写', description: '可以创建和查询任务' },
  { value: 'admin', label: '管理', description: '可以管理 API Key' }
];
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| Key 列表 | GET | `/api/v1/developer/keys` | 查询 API Key 列表 |
| 创建 Key | POST | `/api/v1/developer/keys` | 创建 API Key |
| 删除 Key | DELETE | `/api/v1/developer/keys/:id` | 删除 API Key |

---

## 6. 用户交互流程

```
[进入开发者中心]
    │
    ▼
[浏览 Key 列表]
    │
    ├── 点击创建 Key
    │       │
    │       ▼
    │   [选择权限]
    │       │
    │       ▼
    │   [创建成功]
    │       │
    │       ▼
    │   [展示完整 Key（仅一次）]
    │       │
    │       ▼
    │   [必须点击"我已安全保存"]
    │       │
    │       ▼
    │   [关闭 Modal]
    │
    ├── 点击删除 Key
    └── 点击文档链接（新窗口打开）
```

### Key 复制确认机制

```
1. Modal 显示完整 Key（只读输入框）
2. 提供"复制"按钮
3. 必须点击"我已安全保存"才能关闭 Modal
4. 关闭后不再显示完整 Key
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无 Key | Empty + 引导创建 |
| 创建失败 | 提示错误 |

---

## 8. 性能优化

- Key 列表缓存
- 代码高亮懒加载

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Key 泄露 | 高 | Key 仅展示一次，用户可能未保存 | 强制确认机制 |

---

## 10. 开发步骤拆分

### Step 1: Key 管理（1 天）
- [ ] ApiKeyManager（含创建确认机制）
- [ ] 权限选项配置

### Step 2: 文档与 SDK（0.5 天）
- [ ] QuickStart + SdkDownloads + DocLinks

### Step 3: 代码高亮（0.5 天）
- [ ] 集成 react-syntax-highlighter
