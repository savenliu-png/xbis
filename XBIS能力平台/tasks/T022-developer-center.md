# 开发者中心首页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 开发者中心首页 |
| 任务编号 | T022 |
| 所属模块 ⭐ | M7 开发者中心 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-18 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏开发者中心，开发者需要：
- 管理 API Key
- 查看文档
- 使用调试工具

### 2.2 目标用户
- 开发者：管理 API Key 和查看文档
- 运维人员：调试接口

### 2.3 预期效果
- 提供开发者中心入口
- 支持 API Key 管理
- 支持文档和调试工具

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 开发者中心 | `/developer` | 主导航「开发者」 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/developer/DeveloperCenter`

## 4. 功能范围

### 4.1 包含功能
- [ ] API Key 管理（创建/查看/删除）
- [ ] 文档入口
- [ ] 调试工具入口
- [ ] SDK 下载
- [ ] 快速开始指南
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 详细文档（独立文档站点）
- 完整调试工具（独立页面）
- 社区论坛（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// API Key
export interface ApiKey {
  id: string;
  keyId: string;
  name: string;
  keyPrefix: string;
  permissions: string[];
  lastUsedAt?: string;
  createdAt: string;
  expiresAt?: string;
  isActive: boolean;
}

// API Key 创建请求
export interface ApiKeyCreateRequest {
  name: string;
  permissions: string[];
  expiresIn?: number;
}

// API Key 创建响应
export interface ApiKeyCreateResponse {
  apiKey: ApiKey;
  fullKey: string;
}

// 文档链接
export interface DocLink {
  title: string;
  description: string;
  url: string;
  icon?: string;
}

// SDK 信息
export interface SdkInfo {
  language: string;
  version: string;
  downloadUrl: string;
  installCommand: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`ApiKey`, `ApiKeyCreateRequest`, `ApiKeyCreateResponse`, `DocLink`, `SdkInfo`

## 6. 交互流程

### 6.1 主流程
```
[用户进入开发者中心] ──► [加载 API Keys] ──► [展示信息] ──► [用户管理 Keys]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无 API Key | 用户未创建 | 显示空态 | 提示创建 |
| 创建失败 | 超过数量限制 | 返回错误 | 提示删除旧 Key |
| 删除失败 | Key 正在使用 | 返回错误 | 提示确认 |

### 6.3 边界情况
- 大量 API Key：支持分页
- Key 泄露：支持紧急撤销
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 信息卡片 | 是（T002） |
| Table | Key 表格 | 是（T002） |
| Modal | 创建弹窗 | 是（T002） |
| Form | 表单容器 | 是（T002） |
| Input | 文本输入 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| ApiKeyCard | API Key 卡片 | 否 |
| CodeBlock | 代码展示 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| ApiKeyManager | Key 管理区 | 否 |
| DocLinks | 文档链接区 | 否 |
| SdkDownloads | SDK 下载区 | 否 |
| QuickStart | 快速开始区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| ApiKeyCard | business | API Key 卡片 |
| CodeBlock | business | 代码展示 |
| ApiKeyManager | blocks | Key 管理区 |
| DocLinks | blocks | 文档链接区 |
| SdkDownloads | blocks | SDK 下载区 |
| QuickStart | blocks | 快速开始区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### API Key 列表查询
- **Method**: GET
- **Path**: `/api/v1/developer/keys`
- **响应类型**: `{ items: ApiKey[]; total: number }`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "key-001",
        "keyId": "ak_xxxxxxxx",
        "name": "生产环境",
        "keyPrefix": "ak_xxxx",
        "permissions": ["read", "write"],
        "lastUsedAt": "2026-04-24T10:00:00Z",
        "createdAt": "2026-04-01T00:00:00Z",
        "isActive": true
      }
    ],
    "total": 3
  }
}
```

#### API Key 创建
- **Method**: POST
- **Path**: `/api/v1/developer/keys`
- **请求类型**: `ApiKeyCreateRequest`
- **响应类型**: `ApiKeyCreateResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "name": "测试环境",
  "permissions": ["read"],
  "expiresIn": 2592000
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "apiKey": {
      "id": "key-002",
      "keyId": "ak_yyyyyyyy",
      "name": "测试环境",
      "keyPrefix": "ak_yyyy",
      "permissions": ["read"],
      "createdAt": "2026-04-24T10:30:00Z",
      "expiresAt": "2026-05-24T10:30:00Z",
      "isActive": true
    },
    "fullKey": "ak_yyyyyyyy.zzzzzzzzzzzzzzzz"
  }
}
```

#### API Key 删除
- **Method**: DELETE
- **Path**: `/api/v1/developer/keys/:id`
- **权限**: 用户（已登录）

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.developer.keys`, `userApi.developer.createKey`, `userApi.developer.deleteKey`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] API Key 列表展示正常
- [ ] API Key 创建正常
- [ ] API Key 删除正常
- [ ] 文档入口可点击
- [ ] 调试工具入口可点击
- [ ] SDK 下载正常
- [ ] 快速开始指南展示正常
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
- [ ] Key 创建 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| Key 泄露 | 低 | 高 | 仅展示一次，支持撤销 |
| 权限控制 | 中 | 中 | 细粒度权限 |
| 大量 Keys | 低 | 低 | 分页展示 |

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

- API Key 仅创建时展示完整 Key
- 支持 Key 权限配置
- 支持 Key 过期时间设置

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
