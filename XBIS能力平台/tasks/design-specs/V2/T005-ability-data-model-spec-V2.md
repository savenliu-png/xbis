# T005 ability 数据模型 — 修正版设计方案（V2）

> 原设计方案: [T005-ability-data-model-spec.md](../T005-ability-data-model-spec.md)
> 评审报告: [T005-ability-data-model-Reviewer.md](../T005-ability-data-model-Reviewer.md)
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

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 补充 Ability 与 Executor 绑定关系 | 第 4 节状态管理 |
| 2 | 细化 AbilityUiSchema 类型 | 第 4 节状态管理 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为数据模型定义。

---

### 2. 组件拆分

无组件拆分，本任务为数据模型定义。

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

#### Ability 类型定义（V2 补充）【已修复】

```typescript
interface Ability {
  id: string;
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  icon?: string;
  status: 'published' | 'draft' | 'deprecated';
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  
  // Schema 定义
  requestSchema: Record<string, unknown>;
  responseSchema: Record<string, unknown>;
  uiSchema: AbilityUiSchema; // 【已修复：细化类型】
  
  // 执行配置
  executorId?: string; // 【新增：关联执行器】
  bindings?: AbilityExecutorBinding[]; // 【新增：执行器绑定关系】
  timeout: number;
  retryCount: number;
  
  // 计费配置
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  
  // 元数据
  version: string;
  createdAt: string;
  updatedAt: string;
  createdBy: string;
}

// 【新增】Ability 与 Executor 绑定关系
interface AbilityExecutorBinding {
  executorId: string;
  executorName: string;
  priority: number;
  enabled: boolean;
}

// 【已修复】细化 UI Schema 类型
interface UiSchemaField {
  type: 'string' | 'number' | 'boolean' | 'select' | 'textarea' | 'date' | 'file';
  label: string;
  required?: boolean;
  default?: unknown;
  options?: { label: string; value: string }[];
  validation?: {
    min?: number;
    max?: number;
    pattern?: string;
    message?: string;
  };
}

type AbilityUiSchema = Record<string, UiSchemaField>;
```

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

无边界与异常。

---

### 8. 性能优化

无性能优化。

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 类型演进 | 低 | 后续可能新增字段 | 使用可选属性 |

---

### 10. 开发步骤拆分

#### Step 1: 核心类型定义（0.5 人日）
- [ ] Ability 接口
- [ ] AbilityExecutorBinding 接口 【新增】
- [ ] AbilityUiSchema 细化 【已修复】

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **Executor 绑定** | 未定义 | 新增 `executorId` 和 `AbilityExecutorBinding` |
| **AbilityUiSchema** | `Record<string, any>` | 细化为 `Record<string, UiSchemaField>` |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 原设计已通过评审
2. 建议修改项已补充（Executor 绑定关系、UiSchema 细化）
3. 风险等级：低
