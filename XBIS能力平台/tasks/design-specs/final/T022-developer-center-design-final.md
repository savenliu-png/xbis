# T022 开发者中心首页 — 最终可开发设计方案

> 任务卡: [T022-developer-center.md](../T022-developer-center.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final v2（契约修复后同步更新）
> 契约修复日期: 2026-04-26

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
        │   └── 创建 Key（含接口范围选择 + availableApis）
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
| Modal | 创建 Key 弹窗 |
| Input | 文本输入 |
| Select | 下拉选择（含多选） |
| Skeleton | 加载骨架 |
| Empty | 空态 |
| Alert | 错误/警告提示 |
| CopyButton | 复制按钮 |

### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| ApiKeyItem | Key 列表项（6种状态展示） |

### blocks/ 层组件（新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| ApiKeyManager | Key 管理 | **新建**（含创建确认机制 + availableApis + 删除错误反馈） |
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
[API: GET /portal-api/v1/api-keys]
    │
    ├── 同时可获取 availableApis: GET /portal-api/v1/api-keys/available-apis
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
  error: string | null;
}

// 接口范围选项（与后端 scope_type 对齐）
const scopeTypeOptions = [
  { value: 'full', label: '全部接口', description: '可调用所有已授权接口' },
  { value: 'custom', label: '自定义接口', description: '仅可调用指定接口' },
];
```

### 设计补充说明

- 原设计 `permissionOptions`（read/write/admin）已废弃，后端不接受此结构
- 后端使用 `scope_type: 'full' | 'custom'` + `allowedApis: string[]` 模型
- `keyConfirmed` 字段已移除，由 `rawKeyModalOpen + closable=false` 隐式保证

---

## 5. API 调用（契约修复后）

| API | Method | Path | 说明 | 参数类型 |
|-----|--------|------|------|----------|
| Key 列表 | GET | `/portal-api/v1/api-keys` | 查询 API Key 列表 | `{ page?, pageSize?, keyName?, keyword?, status? }` |
| 创建 Key | POST | `/portal-api/v1/api-keys` | 创建 API Key | `CreateApiKeyRequest` |
| 删除 Key | DELETE | `/portal-api/v1/api-keys/:id` | 删除 API Key | `{ forceLogicalDelete?, confirmHasRecords? }` |
| 可用接口 | GET | `/portal-api/v1/api-keys/available-apis` | 获取可绑定接口列表 | 无 |

### CreateApiKeyRequest 契约

```typescript
interface CreateApiKeyRequest {
  keyName: string;           // 必填，至少2字符
  allowedApis: string[];     // 必填，至少1个 apiId
  sourceSystem?: string;     // 可选
  description?: string;      // 可选
  expiresAt?: string;        // 可选，默认1年
  ipWhitelist?: string[];    // 可选
  allowCallback?: boolean;   // 可选，默认 false
}
```

### CreateApiKeyResponse 契约（camelCase 统一）

```typescript
interface CreateApiKeyResponse {
  rawKey: string;            // 完整密钥，仅创建时返回一次
  keyId: string;
  keyName: string;
  keyPrefix: string;
  status: string;
  allowedApis: string[];
  scopeType: string;
}
```

### 删除 Key 多步流程

```
1. 无调用记录 → 物理删除
2. 有调用记录 + 未禁用 → 拒绝，要求先禁用
3. 有调用记录 + 已禁用 + 未确认 → 拒绝，要求二次确认
4. 有调用记录 + 已禁用 + 已确认 → 逻辑删除
```

当前开发者中心简化处理：直接调用删除，后端返回错误时展示给用户。

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
    │   [输入密钥名称]
    │       │
    │       ▼
    │   [选择接口范围：全部/自定义]
    │       │
    │       ▼
    │   [自定义模式下选择接口]
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
1. Modal 显示完整 Key（只读）
2. 提供"复制"按钮
3. 必须点击"我已安全保存"才能关闭 Modal
4. 关闭后不再显示完整 Key
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无 Key | Empty + 引导创建 |
| 创建失败 | Alert 展示错误信息 |
| 删除失败 | Alert 展示错误信息 |
| availableApis 加载失败 | 空列表，创建时提示 |
| Key 有调用记录无法直接删除 | 展示后端返回的错误信息 |

---

## 8. 性能优化

- Key 列表缓存
- React.memo 包裹所有 blocks 组件
- useMemo 缓存 displayCode / apiOptions / firstKeyPrefix

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Key 泄露 | 高 | Key 仅展示一次，用户可能未保存 | 强制确认机制 |
| 删除多步确认 | 中 | 有调用记录的 Key 需先禁用再逻辑删除 | 当前简化处理，展示后端错误 |

---

## 10. 开发步骤拆分

### Step 1: Key 管理（1 天）
- [x] ApiKeyManager（含创建确认机制 + availableApis + 接口范围选择）
- [x] 删除错误反馈

### Step 2: 文档与 SDK（0.5 天）
- [x] QuickStart + SdkDownloads + DocLinks

### Step 3: 契约修复（0.5 天）
- [x] services.ts 类型安全
- [x] CreateApiKeyRequest 对齐后端
- [x] CreateApiKeyResponse camelCase 统一
- [x] ApiKeyListResponse 类型定义
- [x] 设计方案同步更新
