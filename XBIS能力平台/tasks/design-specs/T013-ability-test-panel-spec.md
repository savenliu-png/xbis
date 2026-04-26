# T013 能力测试面板 — 工程级设计方案

> 任务卡: [T013-ability-test-panel.md](../T013-ability-test-panel.md)  
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 面包屑导航
│   └── 标题: "能力测试"
├── FormPageShell
│   ├── ParameterPanel (左侧)
│   │   ├── SchemaForm (基于 Schema 生成表单)
│   │   └── ManualInput (手动输入 JSON)
│   └── ResultPanel (右侧)
│       ├── ResultViewer (结果展示)
│       └── TestHistory (测试历史)
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
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| SchemaForm | Schema 表单 | 是 |
| ResultViewer | 结果展示 | 是 |
| TestHistory | 测试历史 | 是 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| ParameterPanel | 参数面板 | 是 |
| ResultPanel | 结果面板 | 是 |
| HistoryPanel | 历史面板 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[页面加载]
  └── 调用 GET /api/v1/abilities/:id ──► 获取能力详情（含 Schema）

[用户填写参数]
  ├── SchemaForm 根据 Schema 生成表单
  └── 用户填写参数 ──► 更新 formData

[用户点击调用]
  ├── 校验参数 ──► 调用 POST /api/v1/abilities/:id/test
  ├── 同步调用 ──► 直接返回结果
  └── 异步调用 ──► 返回 jobId，需轮询状态

[结果展示]
  ├── 成功 ──► 展示结果
  └── 失败 ──► 展示错误信息
```

---

## 4. 状态管理

### 页面状态

```typescript
interface AbilityTestState {
  // 数据状态
  ability?: AbilityDetail;
  schema?: SchemaField;
  result?: AbilityTestResponse;
  history: TestHistoryItem[];

  // 表单状态
  formData: Record<string, any>;
  executionMode: 'sync' | 'async';

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
| GET /api/v1/abilities/:id | 页面加载 | abilityId | 显示错误提示 |
| POST /api/v1/abilities/:id/test | 点击调用 | AbilityTestRequest | 展示错误信息 |

---

## 6. 用户交互流程

### 主流程

```
用户进入测试面板
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 展示参数表单
  │   ├── 用户填写参数
  │   ├── 用户选择执行模式
  │   └── 用户点击调用
  │       ├── 同步调用 ──► 实时展示结果
  │       └── 异步调用 ──► 返回 jobId，展示任务状态
  └── 加载失败 ──► 显示错误提示
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 参数校验失败 | 必填项未填 | 高亮错误字段 | 提示填写 |
| 调用失败 | 能力异常 | 展示错误信息 | 提示重试 |
| 超时 | 异步任务超时 | 标记超时 | 提示查看任务列表 |

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
- 同步调用响应 < 3s
- 测试历史存储在 localStorage，限制 50 条

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

### Step 2: Schema 表单开发（1.5 人日）
- [ ] SchemaForm 组件（支持多种类型）
- [ ] 递归表单实现
- [ ] 参数校验

### Step 3: 结果展示开发（0.5 人日）
- [ ] ResultViewer 组件
- [ ] JSON 高亮展示

### Step 4: 测试历史开发（0.5 人日）
- [ ] TestHistory 组件
- [ ] localStorage 存储

### Step 5: API 集成 & 状态管理（0.5 人日）
- [ ] 集成 API 调用
- [ ] 实现同步/异步调用

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 优化表单生成
- [ ] 自测
