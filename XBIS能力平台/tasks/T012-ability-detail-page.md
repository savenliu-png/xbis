# 能力详情页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 能力详情页 |
| 任务编号 | T012 |
| 所属模块 ⭐ | M4 能力平台 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-10 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏统一的能力详情展示页面，需要：
- 展示能力的完整信息
- 支持 Schema 预览
- 提供任务提交入口

### 2.2 目标用户
- 终端用户：查看能力详情
- 开发者：了解能力接口定义

### 2.3 预期效果
- 提供清晰的能力详情展示
- 支持 Schema 可视化
- 支持快速创建任务

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力详情 | `/abilities/:id` | 能力列表页卡片点击 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/ability/AbilityDetailTemplate`

## 4. 功能范围

### 4.1 包含功能
- [ ] 能力基本信息展示
- [ ] 能力介绍（Markdown）
- [ ] 输入输出 Schema 预览
- [ ] 调用示例展示
- [ ] 任务提交入口
- [ ] 版本历史
- [ ] 计费信息
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 在线测试（由 T013 实现）
- 接入流程（由 T014 实现）
- 能力编辑（管理端功能）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 能力详情
export interface AbilityDetail {
  id: string;
  abilityId: string;
  name: string;
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  status: 'published' | 'draft' | 'deprecated';
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  freeQuota?: number;
  overagePrice?: number;
  requestSchema?: Record<string, any>;
  responseSchema?: Record<string, any>;
  docMarkdown?: string;
  icon?: string;
  version: string;
  versions: AbilityVersion[];
  callCount: number;
  rating: number;
  createdAt: string;
  updatedAt: string;
}

// Schema 字段定义
export interface SchemaField {
  type: string;
  description?: string;
  required?: boolean;
  default?: any;
  enum?: any[];
  properties?: Record<string, SchemaField>;
  items?: SchemaField;
}

// 调用示例
export interface AbilityExample {
  title: string;
  description?: string;
  request: Record<string, any>;
  response: Record<string, any>;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AbilityDetail`, `SchemaField`, `AbilityExample`

## 6. 交互流程

### 6.1 主流程
```
[用户点击能力卡片] ──► [加载能力详情] ──► [展示信息] ──► [用户查看 Schema] ──► [点击创建任务]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 能力不存在 | 无效的 abilityId | 返回 404 | 提示能力不存在 |
| Schema 解析失败 | 无效的 Schema | 显示原始 JSON | 提示格式错误 |

### 6.3 边界情况
- 长文档：支持锚点导航
- 复杂 Schema：支持折叠展开
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 信息卡片 | 是（T002） |
| Tag | 标签展示 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Tabs | 内容切换 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Markdown | 文档渲染 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatusBadge | 状态标识 | 是（T003） |
| SchemaViewer | Schema 预览 | 否 |
| CodeBlock | 代码展示 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityInfo | 能力信息区 | 否 |
| SchemaPreview | Schema 预览区 | 否 |
| ExampleGallery | 示例展示区 | 否 |
| VersionHistory | 版本历史区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| SchemaViewer | business | Schema 可视化组件 |
| CodeBlock | business | 代码高亮展示 |
| AbilityInfo | blocks | 能力信息区 |
| SchemaPreview | blocks | Schema 预览区 |
| ExampleGallery | blocks | 示例展示区 |
| VersionHistory | blocks | 版本历史区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 能力详情查询
- **Method**: GET
- **Path**: `/api/v1/abilities/:id`
- **响应类型**: `AbilityDetailResponse`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "ability": {
      "id": "ability-001",
      "abilityId": "text-generation",
      "name": "text-generation",
      "displayName": "文本生成",
      "description": "基于大语言模型的文本生成能力",
      "category": "ai",
      "tags": ["sync", "free"],
      "riskLevel": "low",
      "executionMode": "sync",
      "status": "published",
      "pricingType": "free",
      "requestSchema": {
        "type": "object",
        "properties": {
          "text": { "type": "string", "description": "输入文本" },
          "language": { "type": "string", "enum": ["zh", "en"] }
        }
      },
      "responseSchema": {
        "type": "object",
        "properties": {
          "result": { "type": "string" },
          "tokens": { "type": "number" }
        }
      },
      "docMarkdown": "# 文本生成\n\n## 简介\n...",
      "version": "v1.2.0",
      "callCount": 12580,
      "rating": 4.8,
      "createdAt": "2026-01-15T00:00:00Z",
      "updatedAt": "2026-04-20T00:00:00Z"
    },
    "versions": [
      { "version": "v1.2.0", "changeLog": "优化性能", "publishedAt": "2026-04-20T00:00:00Z" },
      { "version": "v1.1.0", "changeLog": "新增语言支持", "publishedAt": "2026-03-01T00:00:00Z" }
    ]
  }
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.abilities.get`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 能力基本信息展示完整
- [ ] 能力介绍 Markdown 渲染正常
- [ ] 输入输出 Schema 预览正常
- [ ] 调用示例展示正常
- [ ] 任务提交入口可点击
- [ ] 版本历史展示正常
- [ ] 计费信息展示正常
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
- [ ] Schema 渲染 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T011` — 能力中心列表页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| Schema 复杂度高 | 中 | 中 | 分层展示，支持折叠 |
| Markdown 安全 | 低 | 高 | 使用安全渲染库 |
| 移动端适配 | 中 | 低 | 响应式布局 |

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

- 参考实现：`packages/pages/ability/AbilityDetailTemplate`
- Schema 预览支持 JSON Schema 格式
- Markdown 渲染需做安全过滤

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
