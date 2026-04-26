# T017 任务创建流程 — 修正版设计方案（V2）

> 原设计方案: [T017-job-create-flow-spec.md](../T017-job-create-flow-spec.md)
> 评审报告: [T017-job-create-flow-Reviewer.md](../T017-job-create-flow-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed

---

## 二、必须修改项（Blocking）

无必须修改项。

---

## 三、建议修改项（Optional）

无建议修改项。

---

## 四、修正版设计方案（V2）

### 1. 页面结构

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

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 提交按钮 |
| Form | 表单容器 |
| Select | 能力选择 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| AbilityCard | 能力卡片 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| AbilitySelection | 能力选择 | **新建** |
| JobInputForm | 任务输入表单 | **新建** |

---

### 3. 数据流

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

### 4. 状态管理

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

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力列表 | GET | `/api/v1/abilities` | 查询可选能力 |
| 能力详情 | GET | `/api/v1/abilities/:id` | 查询能力 Schema |
| 创建任务 | POST | `/api/v1/jobs` | 创建任务 |

---

### 6. 用户交互流程

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

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 无可用能力 | Empty + 引导 |
| 参数错误 | 表单校验 |

---

### 8. 性能优化

- 能力列表缓存
- Schema 懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Schema 复杂 | 中 | 复杂 Schema 表单难生成 | 分步表单 |

---

### 10. 开发步骤拆分

#### Step 1: 能力选择（0.5 天）
- [ ] AbilitySelection

#### Step 2: 动态表单（1.5 天）
- [ ] JobInputForm（根据 Schema 动态生成）

#### Step 3: 提交流程（0.5 天）
- [ ] API 集成 + 跳转

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **设计内容** | 无修改 | 保持原设计 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 原设计已通过评审
2. 无阻塞项
3. 风险等级：低
