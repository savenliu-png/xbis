# T013 能力测试面板 — 修正版设计方案（V2）

> 原设计方案: [T013-ability-test-panel-spec.md](../T013-ability-test-panel-spec.md)
> 评审报告: [T013-ability-test-panel-Reviewer.md](../T013-ability-test-panel-Reviewer.md)
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
TestPanel
├── InputForm
│   └── 根据 Schema 动态生成表单
├── ExecuteButton
│   └── [执行测试]
└── ResultPanel
    ├── 响应结果
    └── 执行日志
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 执行按钮 |
| Form | 表单容器 |
| Input | 输入框 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| TestPanel | 测试面板 | **新建** |
| ResultPanel | 结果展示 | **新建** |

---

### 3. 数据流

```
[填写测试参数]
    │
    ▼
[点击执行]
    │
    ▼
[API: POST /api/v1/abilities/:id/test]
    │
    ▼
[展示结果]
```

---

### 4. 状态管理

```typescript
interface TestPanelState {
  pageState: 'idle' | 'executing' | 'success' | 'error';
  result: TestResult | null;
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 能力测试 | POST | `/api/v1/abilities/:id/test` | 执行能力测试 |

---

### 6. 用户交互流程

```
[进入测试面板]
    │
    ▼
[填写参数]
    │
    ▼
[点击执行]
    │
    ▼
[查看结果]
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 参数错误 | 表单校验提示 |
| 执行失败 | 展示错误信息 |

---

### 8. 性能优化

- 测试请求超时处理
- 结果缓存

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 测试安全风险 | 中 | 测试可能执行危险操作 | 沙箱环境 |

---

### 10. 开发步骤拆分

#### Step 1: 测试面板（1 天）
- [ ] InputForm + ResultPanel

#### Step 2: API 集成（0.5 天）
- [ ] 测试 API + 错误处理

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
