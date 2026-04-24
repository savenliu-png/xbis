# 能力测试面板

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 能力测试面板 |
| 任务编号 | T013 |
| 所属模块 ⭐ | M4 能力平台 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏在线能力测试功能，开发者需要：
- 在线填写参数并调用能力
- 实时查看调用结果
- 快速验证能力接口

### 2.2 目标用户
- 开发者：在线测试能力接口
- 运维人员：验证能力可用性

### 2.3 预期效果
- 提供在线测试面板
- 支持参数表单自动生成
- 支持结果可视化展示

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力测试 | `/abilities/:id/test` | 能力详情页「测试」按钮 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/ability/AbilityTestPanel`

## 4. 功能范围

### 4.1 包含功能
- [ ] 参数表单自动生成（基于 Schema）
- [ ] 手动填写参数
- [ ] 调用能力
- [ ] 结果展示
- [ ] 错误展示
- [ ] 调用历史
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 批量测试（V2 迭代）
- 性能测试（V2 迭代）
- 自动化测试（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 测试参数
export interface TestParameter {
  name: string;
  type: string;
  description?: string;
  required: boolean;
  default?: any;
  value?: any;
}

// 测试请求
export interface AbilityTestRequest {
  abilityId: string;
  parameters: Record<string, any>;
  executionMode?: 'sync' | 'async';
}

// 测试响应
export interface AbilityTestResponse {
  jobId?: string;
  status: JobStatus;
  result?: Record<string, any>;
  error?: {
    code: string;
    message: string;
    details?: Record<string, any>;
  };
  duration: number;
  executedAt: string;
}

// 测试历史
export interface TestHistoryItem {
  id: string;
  abilityId: string;
  parameters: Record<string, any>;
  status: JobStatus;
  result?: Record<string, any>;
  error?: string;
  duration: number;
  createdAt: string;
}
```

### 5.2 数据库变更（如适用）
无（测试历史存储在本地存储）

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`TestParameter`, `AbilityTestRequest`, `AbilityTestResponse`, `TestHistoryItem`

## 6. 交互流程

### 6.1 主流程
```
[用户进入测试面板] ──► [加载 Schema] ──► [生成参数表单] ──► [用户填写参数] ──► [点击调用] ──► [展示结果]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| Schema 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 参数校验失败 | 必填项未填 | 高亮错误字段 | 提示填写 |
| 调用失败 | 能力异常 | 展示错误信息 | 提示重试 |
| 超时 | 异步任务超时 | 标记超时 | 提示查看任务列表 |

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
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| SchemaForm | Schema 表单 | 否 |
| ResultViewer | 结果展示 | 否 |
| TestHistory | 测试历史 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| ParameterPanel | 参数面板 | 否 |
| ResultPanel | 结果面板 | 否 |
| HistoryPanel | 历史面板 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| SchemaForm | business | 基于 Schema 生成表单 |
| ResultViewer | business | 结果可视化展示 |
| TestHistory | business | 测试历史列表 |
| ParameterPanel | blocks | 参数填写区 |
| ResultPanel | blocks | 结果展示区 |
| HistoryPanel | blocks | 历史记录区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 能力测试（同步）
- **Method**: POST
- **Path**: `/api/v1/abilities/:id/test`
- **请求类型**: `AbilityTestRequest`
- **响应类型**: `AbilityTestResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "abilityId": "text-generation",
  "parameters": {
    "text": "Hello World",
    "language": "zh"
  },
  "executionMode": "sync"
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "status": "completed",
    "result": {
      "result": "你好，世界",
      "tokens": 5
    },
    "duration": 1200,
    "executedAt": "2026-04-24T10:30:00Z"
  }
}
```

#### 能力测试（异步）
- **Method**: POST
- **Path**: `/api/v1/abilities/:id/test`
- **请求类型**: `AbilityTestRequest`
- **响应类型**: `JobCreateResponse`
- **权限**: 用户（已登录）

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
- 命名：`userApi.abilities.test`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 参数表单基于 Schema 自动生成
- [ ] 支持多种参数类型（字符串/数字/布尔/枚举/对象/数组）
- [ ] 同步调用正常，实时展示结果
- [ ] 异步调用正常，返回 jobId
- [ ] 结果展示支持 JSON 高亮
- [ ] 错误展示清晰，包含错误码和描述
- [ ] 测试历史记录正常（本地存储）
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
- [ ] 同步调用响应 < 3s
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

- 测试历史存储在 localStorage，限制 50 条
- 同步调用超时限制 10 秒
- 复杂参数支持递归表单

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
