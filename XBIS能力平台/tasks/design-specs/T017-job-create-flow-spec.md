# T017 任务创建流程 — 工程级设计方案

> 任务卡: [T017-job-create-flow.md](../T017-job-create-flow.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 面包屑导航
│   └── 标题: "创建任务"
├── FormPageShell
│   ├── Steps (步骤条)
│   │   ├── Step 1: 选择能力
│   │   └── Step 2: 填写参数
│   ├── StepContent
│   │   ├── AbilitySelector (能力选择)
│   │   └── ParameterForm (参数表单)
│   └── SubmitPanel (提交面板)
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Input | 文本输入 | 否 |
| Textarea | 大文本输入 | 否 |
| Select | 下拉选择 | 否 |
| Switch | 布尔切换 | 否 |
| Form | 表单容器 | 否 |
| Steps | 步骤条 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| SchemaForm | Schema 表单 | 否 |
| AbilityCard | 能力卡片 | 否 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| AbilitySelector | 能力选择区 | 是 |
| ParameterForm | 参数表单区 | 是 |
| SubmitPanel | 提交面板 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[页面加载]
  └── 调用 GET /api/v1/abilities/options ──► 获取能力列表

[用户选择能力]
  └── 调用 GET /api/v1/abilities/:id ──► 获取能力详情（含 Schema）

[用户填写参数]
  ├── SchemaForm 根据 Schema 生成表单
  └── 用户填写参数 ──► 更新 formData

[用户提交任务]
  ├── 校验参数 ──► 调用 POST /api/v1/jobs
  ├── 同步任务 ──► 直接返回结果
  └── 异步任务 ──► 返回 jobId，跳转任务详情
```

---

## 4. 状态管理

### 页面状态

```typescript
interface JobCreateState {
  // 数据状态
  abilities: AbilityOption[];
  selectedAbility?: AbilityDetail;
  result?: JobCreateResult;

  // 表单状态
  formData: Record<string, any>;
  executionMode: 'async' | 'sync';
  callbackUrl?: string;
  timeout?: number;

  // 步骤状态
  currentStep: number;

  // UI 状态
  status: 'idle' | 'loading' | 'success' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/abilities/options | 页面加载 | - | 显示错误提示 |
| GET /api/v1/abilities/:id | 选择能力 | abilityId | 显示错误提示 |
| POST /api/v1/jobs | 提交任务 | JobCreateRequest | 展示错误信息 |

---

## 6. 用户交互流程

### 主流程

```
用户点击创建任务
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示能力选择
  │   ├── 用户选择能力
  │   ├── 加载能力 Schema
  │   ├── 用户填写参数
  │   ├── 用户选择执行模式
  │   └── 用户提交任务
  │       ├── 同步任务 ──► 展示结果
  │       └── 异步任务 ──► 跳转任务详情
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 参数校验失败 | 必填项未填 | 高亮错误字段 | 提示填写 |
| 提交失败 | 能力不可用 | 返回错误 | 提示选择其他能力 |
| 超时 | 同步任务超时 | 标记超时 | 提示查看任务列表 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 复杂嵌套参数 | 支持递归表单 |
| 大文本参数 | 支持文本域 |
| 文件参数 | 支持上传组件 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- 表单生成 < 500ms
- 同步任务响应 < 3s

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Schema 复杂度高 | 中 | 复杂 Schema 可能难以生成表单 | 分层展示，支持折叠 |
| 同步调用超时 | 中 | 同步调用可能超时 | 默认异步，同步限制 10s |
| 参数校验复杂 | 低 | 参数校验可能复杂 | 使用 JSON Schema 校验 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 FormPageShell 搭建页面框架
- [ ] 配置 Steps 步骤条

### Step 2: 能力选择开发（1 人日）
- [ ] AbilitySelector 组件
- [ ] 能力卡片展示

### Step 3: 参数表单开发（1 人日）
- [ ] ParameterForm 组件
- [ ] SchemaForm 集成
- [ ] 参数校验

### Step 4: 提交面板开发（0.5 人日）
- [ ] SubmitPanel 组件
- [ ] 执行模式选择

### Step 5: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现同步/异步处理

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 优化表单生成
- [ ] 自测
