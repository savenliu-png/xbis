# T016 任务详情页 — 修正版设计方案（V2）

> 原设计方案: [T016-job-detail-page-spec.md](../T016-job-detail-page-spec.md)
> 评审报告: [T016-job-detail-page-Reviewer.md](../T016-job-detail-page-Reviewer.md)
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
    │   ├── Title: "任务详情"
    │   └── Actions: [取消] [重试]
    └── DetailPageShell
        ├── JobInfoCard
        │   └── 任务基本信息
        ├── JobStatusPanel
        │   └── 状态时间线
        └── JobResultPanel
            └── 执行结果
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | 操作按钮 |
| Card | 信息卡片 |
| Timeline | 状态时间线 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| TaskStatusBadge | 状态徽章 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| JobInfoCard | 任务信息卡片 | **新建** |
| JobStatusPanel | 状态面板 | **新建** |
| JobResultPanel | 结果面板 | **新建** |

---

### 3. 数据流

```
[进入 /jobs/:id]
    │
    ▼
[API: GET /api/v1/jobs/:id]
    │
    ▼
[渲染任务详情]
```

---

### 4. 状态管理

```typescript
interface JobDetailState {
  pageState: 'loading' | 'idle' | 'error';
  job: Job | null;
  error: ApiError | null;
}
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 任务详情 | GET | `/api/v1/jobs/:id` | 查询任务详情 |
| 取消任务 | POST | `/api/v1/jobs/:id/cancel` | 取消任务 |
| 重试任务 | POST | `/api/v1/jobs/:id/retry` | 重试任务 |

---

### 6. 用户交互流程

```
[进入任务详情]
    │
    ▼
[浏览详情]
    │
    ├── 点击取消
    ├── 点击重试
    └── 查看结果
```

---

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 任务不存在 | 404 页面 |
| 加载失败 | ErrorState + 重试 |

---

### 8. 性能优化

- 详情数据缓存
- 状态轮询（T018）

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 状态同步 | 中 | 状态可能不及时 | 轮询机制 |

---

### 10. 开发步骤拆分

#### Step 1: 基础结构（0.5 天）
- [ ] 页面路由 + DetailPageShell

#### Step 2: 详情组件（1 天）
- [ ] JobInfoCard + JobStatusPanel + JobResultPanel

#### Step 3: API 集成（0.5 天）
- [ ] API 调用 + 错误处理

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
