# 管理端能力编辑页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端能力编辑页 |
| 任务编号 | T027 |
| 所属模块 ⭐ | M10 能力管理 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 5 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-22 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏管理端能力编辑页面，管理员需要：
- 编辑能力基础信息
- 配置输入输出 Schema
- 配置 UI 表单
- 配置执行参数
- 配置计费规则
- 发布能力

### 2.2 目标用户
- 管理员：编辑能力配置
- 运营人员：发布能力

### 2.3 预期效果
- 提供 6 Tab 编辑页面
- 支持完整的能力配置
- 支持版本管理

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力编辑 | `/admin/abilities/:id/edit` | 能力管理列表「编辑」按钮 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/AbilityEditPage`

## 4. 功能范围

### 4.1 包含功能
- [ ] 基础信息 Tab（名称/描述/分类/标签）
- [ ] 输入输出 Tab（Schema 编辑）
- [ ] UI 配置 Tab（表单配置）
- [ ] 执行配置 Tab（执行器/超时/重试）
- [ ] 计费配置 Tab（价格/配额）
- [ ] 发布管理 Tab（版本/状态）
- [ ] 保存草稿
- [ ] 预览
- [ ] 发布
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 能力克隆（V2 迭代）
- 能力导入（V2 迭代）
- 能力导出（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 能力编辑表单
export interface AbilityEditForm {
  // 基础信息
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  icon?: string;
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  
  // 输入输出
  requestSchema: Record<string, any>;
  responseSchema: Record<string, any>;
  
  // UI 配置
  uiSchema: Record<string, any>;
  formConfig: Record<string, any>;
  
  // 执行配置
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  timeout: number;
  retryCount: number;
  executorId?: string;
  
  // 计费配置
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  freeQuota?: number;
  overagePrice?: number;
  
  // 发布管理
  status: 'published' | 'draft' | 'deprecated';
  version: string;
  changeLog: string;
}

// Schema 编辑器配置
export interface SchemaEditorConfig {
  type: 'object' | 'array' | 'string' | 'number' | 'boolean';
  title?: string;
  description?: string;
  required?: boolean;
  default?: any;
  enum?: any[];
  properties?: Record<string, SchemaEditorConfig>;
  items?: SchemaEditorConfig;
}

// 发布请求
export interface AbilityPublishRequest {
  abilityId: string;
  version: string;
  changeLog: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AbilityEditForm`, `SchemaEditorConfig`, `AbilityPublishRequest`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入编辑页] ──► [加载能力数据] ──► [编辑各 Tab] ──► [保存草稿] ──► [预览] ──► [发布]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 保存失败 | 数据错误 | 返回错误 | 提示修改 |
| 发布失败 | 校验不通过 | 返回错误 | 提示补充 |
| 版本冲突 | 版本已存在 | 返回错误 | 提示更换版本 |

### 6.3 边界情况
- 复杂 Schema：支持嵌套编辑
- 大表单：支持分 Tab
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 文本输入 | 是（T002） |
| Textarea | 大文本输入 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Tabs | Tab 切换 | 是（T002） |
| Form | 表单容器 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| SchemaEditor | Schema 编辑器 | 否 |
| FormBuilder | 表单构建器 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| BasicInfoTab | 基础信息 Tab | 否 |
| SchemaTab | Schema Tab | 否 |
| UiConfigTab | UI 配置 Tab | 否 |
| ExecutionTab | 执行配置 Tab | 否 |
| BillingTab | 计费配置 Tab | 否 |
| PublishTab | 发布管理 Tab | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| SchemaEditor | business | Schema 编辑器 |
| FormBuilder | business | 表单构建器 |
| BasicInfoTab | blocks | 基础信息 Tab |
| SchemaTab | blocks | Schema Tab |
| UiConfigTab | blocks | UI 配置 Tab |
| ExecutionTab | blocks | 执行配置 Tab |
| BillingTab | blocks | 计费配置 Tab |
| PublishTab | blocks | 发布管理 Tab |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 能力详情查询（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/abilities/:id`
- **响应类型**: `AbilityDetailResponse`
- **权限**: 管理员

#### 能力更新
- **Method**: PUT
- **Path**: `/admin-api/v1/abilities/:id`
- **请求类型**: `AbilityEditForm`
- **响应类型**: `Ability`
- **权限**: 管理员

**请求示例**:
```json
{
  "displayName": "文本生成",
  "description": "基于大语言模型的文本生成能力",
  "category": "ai",
  "tags": ["nlp", "generation"],
  "riskLevel": "low",
  "requestSchema": {
    "type": "object",
    "properties": {
      "text": { "type": "string", "description": "输入文本" }
    }
  },
  "responseSchema": {
    "type": "object",
    "properties": {
      "result": { "type": "string" }
    }
  },
  "executionMode": "sync",
  "timeout": 30000,
  "retryCount": 3,
  "pricingType": "free",
  "status": "draft",
  "version": "v1.2.0",
  "changeLog": "优化性能"
}
```

#### 能力发布
- **Method**: POST
- **Path**: `/admin-api/v1/abilities/:id/publish`
- **请求类型**: `AbilityPublishRequest`
- **响应类型**: `Ability`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.abilities.get`, `adminApi.abilities.update`, `adminApi.abilities.publish`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 6 Tab 编辑正常
- [ ] 基础信息编辑正常
- [ ] Schema 编辑正常
- [ ] UI 配置编辑正常
- [ ] 执行配置编辑正常
- [ ] 计费配置编辑正常
- [ ] 发布管理正常
- [ ] 保存草稿正常
- [ ] 预览功能正常
- [ ] 发布功能正常
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
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 管理端无需移动端设计

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T026` — 管理端能力管理列表 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| Schema 编辑器复杂 | 高 | 高 | 使用开源库或简化实现 |
| 表单数据量大 | 中 | 中 | 分 Tab 保存 |
| 版本管理 | 中 | 中 | 后端校验 |

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

- Schema 编辑器可使用开源库（如 react-jsonschema-form）
- 支持草稿自动保存
- 发布前需校验所有必填项

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
