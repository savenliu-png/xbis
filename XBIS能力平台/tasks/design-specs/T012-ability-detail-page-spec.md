# T012 能力详情页 — 工程级设计方案

> 任务卡: [T012-ability-detail-page.md](../T012-ability-detail-page.md)
> 设计输入: docs/page-templates.md, docs/component-architecture.md, docs/dev-rules.md
> 输出日期: 2026-04-24

---

## 1. 页面结构

```
PageShell
├── PageHeader
│   ├── 面包屑导航
│   └── 返回按钮
├── DetailPageShell
│   ├── MainContentArea (70%)
│   │   ├── AbilityInfo (能力基本信息)
│   │   ├── Tabs
│   │   │   ├── TabPane: 介绍 (Markdown)
│   │   │   ├── TabPane: Schema (输入/输出)
│   │   │   ├── TabPane: 示例
│   │   │   └── TabPane: 版本历史
│   │   └── ActionBar (操作栏)
│   │       ├── 测试按钮
│   │       └── 接入按钮
│   └── SideContentArea (30%)
│       ├── AbilityMeta (元信息)
│       ├── PricingInfo (计费信息)
│       └── RelatedAbilities (相关能力)
└── (空态/加载态/错误态)
```

---

## 2. 组件拆分

### base/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| Button | 操作按钮 | 否 |
| Card | 信息卡片 | 否 |
| Tag | 标签 | 否 |
| Badge | 状态标识 | 否 |
| Tabs | 内容切换 | 否 |
| Skeleton | 加载骨架 | 否 |
| Empty | 空态 | 否 |
| Markdown | 文档渲染 | 否 |

### business/ 层

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| StatusBadge | 状态标识 | 否 |
| SchemaViewer | Schema 预览 | 是 |
| CodeBlock | 代码展示 | 是 |

### blocks/ 层（需新建）

| 组件 | 用途 | 是否新建 |
|------|------|----------|
| AbilityInfo | 能力信息区 | 是 |
| SchemaPreview | Schema 预览区 | 是 |
| ExampleGallery | 示例展示区 | 是 |
| VersionHistory | 版本历史区 | 是 |
| AbilityMeta | 元信息区 | 是 |
| PricingInfo | 计费信息区 | 是 |

### layout/ 层

| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页模板 |

---

## 3. 数据流

```
[页面加载]
  └── 调用 GET /api/v1/abilities/:id ──► 获取能力详情

[用户切换 Tab]
  └── 无需重新加载，本地切换

[用户点击测试]
  └── 跳转 /abilities/:id/test

[用户点击接入]
  └── 跳转 /abilities/:id/subscribe
```

---

## 4. 状态管理

### 页面状态

```typescript
interface AbilityDetailState {
  // 数据状态
  ability?: AbilityDetail;
  versions: AbilityVersion[];

  // UI 状态
  activeTab: 'intro' | 'schema' | 'examples' | 'versions';
  status: 'idle' | 'loading' | 'empty' | 'error';
  errorMessage?: string;
}
```

---

## 5. API 调用

### 触发时机

| API | 触发时机 | 参数 | 错误处理 |
|-----|----------|------|----------|
| GET /api/v1/abilities/:id | 页面加载 | abilityId | 显示错误提示，提供重试按钮 |

---

## 6. 用户交互流程

### 主流程

```
用户点击能力卡片
  ├── 页面加载中 ──► 显示 Skeleton
  ├── 加载完成 ──► 显示能力详情
  │   ├── 用户切换 Tab ──► 展示对应内容
  │   ├── 用户点击测试 ──► 跳转测试页面
  │   └── 用户点击接入 ──► 跳转接入页面
  └── 加载失败 ──► 显示错误提示 + 重试按钮
```

### 异常流程

| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 能力不存在 | 无效的 abilityId | 返回 404 | 提示能力不存在 |
| Schema 解析失败 | 无效的 Schema | 显示原始 JSON | 提示格式错误 |
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 长文档 | 支持锚点导航 |
| 复杂 Schema | 支持折叠展开 |
| 暗色模式 | 自动适配 |

---

## 8. 性能优化

- 首屏加载 < 2s
- Schema 渲染 < 500ms
- 无内存泄漏（useEffect 清理）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| Schema 复杂度高 | 中 | 复杂 Schema 可能难以展示 | 分层展示，支持折叠 |
| Markdown 安全 | 低 | Markdown 可能包含恶意内容 | 使用安全渲染库 |
| 移动端适配 | 中 | 详情页在移动端需调整 | 响应式布局 |

---

## 10. 开发步骤拆分

### Step 1: 页面模板搭建（0.5 人日）
- [ ] 使用 DetailPageShell 搭建页面框架
- [ ] 配置 PageHeader + 面包屑

### Step 2: 信息区开发（1 人日）
- [ ] AbilityInfo 组件
- [ ] AbilityMeta 组件
- [ ] PricingInfo 组件

### Step 3: Tab 内容区开发（1 人日）
- [ ] Markdown 渲染（介绍 Tab）
- [ ] SchemaViewer 组件（Schema Tab）
- [ ] ExampleGallery 组件（示例 Tab）
- [ ] VersionHistory 组件（版本 Tab）

### Step 4: 操作栏开发（0.5 人日）
- [ ] 测试按钮
- [ ] 接入按钮

### Step 5: 状态管理 & API 集成（0.5 人日）
- [ ] 实现页面状态管理
- [ ] 集成 API 调用
- [ ] 实现加载/空态/错误态

### Step 6: 性能优化 & 测试（0.5 人日）
- [ ] 优化 Schema 渲染
- [ ] 自测
