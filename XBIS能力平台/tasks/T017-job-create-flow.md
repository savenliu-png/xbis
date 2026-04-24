# 任务创建流程

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 任务创建流程 |
| 任务编号 | T017 |
| 所属模块 ⭐ | M5 任务系统 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-12 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏标准化的任务创建流程，需要：
- 选择能力
- 填写参数
- 提交任务
- 返回任务 ID

### 2.2 目标用户
- 终端用户：创建任务
- 开发者：调用能力

### 2.3 预期效果
- 提供标准化的任务创建流程
- 支持参数表单自动生成
- 支持同步和异步模式

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 创建任务 | `/jobs/create` | 任务列表页「创建任务」按钮 |
| 创建任务 | `/abilities/:id/create` | 能力详情页「创建任务」按钮 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/job/JobCreateFlow`

## 4. 功能范围

### 4.1 包含功能
- [ ] 选择能力
- [ ] 参数表单自动生成
- [ ] 手动填写参数
- [ ] 选择执行模式
- [ ] 提交任务
- [ ] 返回任务 ID
- [ ] 跳转任务详情
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 批量创建（V2 迭代）
- 定时任务（V2 迭代）
- 任务模板（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 任务创建表单
export interface JobCreateForm {
  abilityId: string;
  parameters: Record<string, any>;
  executionMode: 'async' | 'sync';
  callbackUrl?: string;
  timeout?: number;
}

// 能力选择项
export interface AbilityOption {
  id: string;
  abilityId: string;
  displayName: string;
  description: string;
  category: string;
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  icon?: string;
}

// 参数字段定义
export interface ParameterField {
  name: string;
  type: string;
  description?: string;
  required: boolean;
  default?: any;
  enum?: any[];
}

// 创建结果
export interface JobCreateResult {
  jobId: string;
  traceId: string;
  status: JobStatus;
  estimatedDuration?: number;
  redirectUrl?: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`JobCreateForm`, `AbilityOption`, `ParameterField`, `JobCreateResult`

## 6. 交互流程

### 6.1 主流程
```
[用户点击创建任务] ──► [选择能力] ──► [加载 Schema] ──► [填写参数] ──► [提交任务] ──► [返回 jobId]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 参数校验失败 | 必填项未填 | 高亮错误字段 | 提示填写 |
| 提交失败 | 能力不可用 | 返回错误 | 提示选择其他能力 |
| 超时 | 同步任务超时 | 标记超时 | 提示查看任务列表 |

### 6.3 边界情况
- 复杂嵌套参数：支持递归表单
- 大文本参数：支持文本域
- 文件参数：支持上传组件
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 文本输入 | 是（T002） |
| Textarea | 大文本输入 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Switch | 布尔切换 | 是（T002） |
| Form | 表单容器 | 是（T002） |
| Steps | 步骤条 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| SchemaForm | Schema 表单 | 是（T013） |
| AbilityCard | 能力卡片 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilitySelector | 能力选择区 | 否 |
| ParameterForm | 参数表单区 | 否 |
| SubmitPanel | 提交面板 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| AbilitySelector | blocks | 能力选择区 |
| ParameterForm | blocks | 参数表单区 |
| SubmitPanel | blocks | 提交面板 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 获取能力列表（简化）
- **Method**: GET
- **Path**: `/api/v1/abilities/options`
- **响应类型**: `AbilityOption[]`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": "ability-001",
      "abilityId": "text-generation",
      "displayName": "文本生成",
      "description": "基于大语言模型的文本生成能力",
      "category": "ai",
      "riskLevel": "low",
      "executionMode": "sync"
    }
  ]
}
```

#### 创建任务
- **Method**: POST
- **Path**: `/api/v1/jobs`
- **请求类型**: `JobCreateRequest`
- **响应类型**: `JobCreateResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "abilityId": "text-generation",
  "payload": {
    "text": "Hello World",
    "language": "zh"
  },
  "executionMode": "async",
  "callbackUrl": "https://example.com/callback",
  "timeout": 30000
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "jobId": "job-20260424-001",
    "traceId": "trace-abc123",
    "status": "pending",
    "estimatedDuration": 5000
  }
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.abilities.options`, `userApi.jobs.create`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 能力选择正常
- [ ] 参数表单基于 Schema 自动生成
- [ ] 支持多种参数类型
- [ ] 同步任务正常，实时返回结果
- [ ] 异步任务正常，返回 jobId
- [ ] 提交后跳转任务详情
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
- [ ] 表单生成 < 500ms
- [ ] 同步任务响应 < 3s
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T012` — 能力详情页 — 待开发
  - [x] `T015` — 任务列表页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| Schema 复杂度高 | 中 | 中 | 分层展示，支持折叠 |
| 同步调用超时 | 中 | 中 | 默认异步，同步限制 10s |
| 参数校验复杂 | 低 | 中 | 使用 JSON Schema 校验 |

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

- 支持从能力详情页直接创建任务
- 同步调用超时限制 10 秒
- 复杂参数支持递归表单

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
