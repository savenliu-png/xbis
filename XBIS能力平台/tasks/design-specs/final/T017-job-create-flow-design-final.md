# T017 任务创建流程 — 最终可开发设计方案

> 任务卡: [T017-job-create-flow.md](../T017-job-create-flow.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

```
AppShell
└── PageShell
    ├── PageHeader
    │   ├── Title: "创建任务"
    │   └── Breadcrumb: 任务列表 / 创建任务
    └── FormPageShell
        ├── AbilitySelection
        │   └── 能力选择
        ├── JobInputForm
        │   └── 根据能力 Schema 动态生成表单
        └── SubmitButton
            └── [创建任务]
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
| AbilitySelection | 能力选择区 | 是 |
| JobInputForm | 任务输入表单 | 是 |
| SubmitPanel | 提交面板 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页模板 |

---

## 3. 数据流

```
[进入 /jobs/create]
    │
    ▼
[选择能力]
    │
    ▼
[加载能力 Schema]
    │
    ▼
[填写参数]
    │
    ▼
[提交创建]
    │
    ▼
[API: POST /api/v1/jobs]
    │
    ▼
[跳转任务详情]
```

---

## 4. 状态管理

```typescript
interface JobCreateState {
  step: 'select' | 'input' | 'submitting';
  selectedAbility: Ability | null;
  formData: Record<string, unknown>;
  loading: boolean;
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力列表 | GET | `/api/v1/abilities` | 查询可选能力 |
| 能力详情 | GET | `/api/v1/abilities/:id` | 查询能力 Schema |
| 创建任务 | POST | `/api/v1/jobs` | 创建任务 |

---

## 6. 用户交互流程

```
[进入创建页面]
    │
    ▼
[选择能力]
    │
    ▼
[填写参数]
    │
    ▼
[提交创建]
    │
    ▼
[跳转详情]
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无可用能力 | Empty + 引导 |
| 参数错误 | 表单校验 |
| 提交失败 | 提示选择其他能力 |
| 超时 | 提示查看任务列表 |

---

## 8. 性能优化

- 能力列表缓存
- Schema 懒加载
- 首屏加载 < 2s
- 表单生成 < 500ms

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Schema 复杂 | 中 | 复杂 Schema 表单难生成 | 分步表单 |
| 同步调用超时 | 中 | 同步调用可能超时 | 默认异步，同步限制 10s |
| 参数校验复杂 | 低 | 参数校验可能复杂 | 使用 JSON Schema 校验 |

---

## 10. 开发步骤拆分

### Step 1: 能力选择（0.5 天）
- [ ] AbilitySelection

### Step 2: 动态表单（1.5 天）
- [ ] JobInputForm（根据 Schema 动态生成）

### Step 3: 提交流程（0.5 天）
- [ ] API 集成 + 跳转
