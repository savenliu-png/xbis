# T022 开发者中心首页 — 修正版设计方案（V2）

> 原设计方案: [T022-developer-center-spec.md](../T022-developer-center-spec.md)
> 评审报告: [T022-developer-center-Reviewer.md](../T022-developer-center-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 增加 Key 复制确认机制 | 第 6 节用户交互流程 |
| 2 | 明确权限选项 | 第 4 节状态管理 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 选择代码高亮库 | 第 2 节组件拆分 |
| 2 | 外部链接在新窗口打开 | 第 6 节用户交互流程 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   └── Title: "开发者中心"
    └── DetailPageShell
        ├── ApiKeyManager
        │   ├── Key 列表
        │   └── 创建 Key 【已修复】
        ├── QuickStart
        │   └── 快速开始代码
        ├── SdkDownloads
        │   └── SDK 下载
        └── DocLinks
            └── 文档链接 【已修复】
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Table | Key 列表 |
| Modal | 创建 Key 弹窗 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| ApiKeyItem | Key 列表项 |
| CodeBlock | 代码展示 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| ApiKeyManager | Key 管理 | **新建** |
| QuickStart | 快速开始 | **新建** |
| SdkDownloads | SDK 下载 | **新建** |
| DocLinks | 文档链接 | **新建** |

---

### 3. 数据流

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

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface DeveloperCenterState {
  pageState: 'loading' | 'idle' | 'error';
  keys: ApiKey[];
  createModalOpen: boolean;
  newKey: string | null; // 【新增】新创建的 Key（仅展示一次）
  keyConfirmed: boolean; // 【新增】用户是否已确认保存
  error: ApiError | null;
}

// 【新增】权限选项
const permissionOptions = [
  { value: 'read', label: '只读', description: '只能查询任务状态' },
  { value: 'write', label: '读写', description: '可以创建和查询任务' },
  { value: 'admin', label: '管理', description: '可以管理 API Key' }
];
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| Key 列表 | GET | `/api/v1/developer/keys` | 查询 API Key 列表 |
| 创建 Key | POST | `/api/v1/developer/keys` | 创建 API Key |
| 删除 Key | DELETE | `/api/v1/developer/keys/:id` | 删除 API Key |

---

### 6. 用户交互流程（V2 修正）【已修复】

```
[进入开发者中心]
    │
    ▼
[浏览 Key 列表]
    │
    ├── 点击创建 Key
    │       │
    │       ▼
    │   [选择权限] 【已修复】
    │       │
    │       ▼
    │   [创建成功]
    │       │
    │       ▼
    │   [展示完整 Key（仅一次）] 【已修复】
    │       │
    │       ▼
    │   [必须点击"我已安全保存"] 【新增】
    │       │
    │       ▼
    │   [关闭 Modal]
    │
    ├── 点击删除 Key
    └── 点击文档链接（新窗口打开）【已修复】
```

#### Key 复制确认机制（V2 新增）【已修复】

```
1. Modal 显示完整 Key（只读输入框）
2. 提供"复制"按钮
3. 必须点击"我已安全保存"才能关闭 Modal
4. 关闭后不再显示完整 Key
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无 Key | Empty + 引导创建 |
| 创建失败 | 提示错误 |

---

### 8. 性能优化

- Key 列表缓存
- 代码高亮懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Key 泄露 | 高 | Key 仅展示一次，用户可能未保存 | 强制确认机制 |

---

### 10. 开发步骤拆分

#### Step 1: Key 管理（1 天）【已修复】
- [ ] ApiKeyManager（含创建确认机制）
- [ ] 权限选项配置

#### Step 2: 文档与 SDK（0.5 天）
- [ ] QuickStart + SdkDownloads + DocLinks

#### Step 3: 代码高亮（0.5 天）【已修复】
- [ ] 集成 react-syntax-highlighter

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **Key 确认** | 未设计 | 新增强制确认机制 |
| **权限选项** | 未明确 | 明确 read/write/admin 选项 |
| **代码高亮** | 未声明 | 明确使用 react-syntax-highlighter |
| **外部链接** | 未说明 | 明确新窗口打开 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（Key 确认机制、权限选项）
2. 建议修改项已补充（代码高亮、外部链接）
3. 风险等级：低
